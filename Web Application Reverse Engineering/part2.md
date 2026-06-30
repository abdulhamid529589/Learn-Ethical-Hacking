## Phase 5: WebAssembly & Advanced Topics

### Week 9: WebAssembly (WASM) Reverse Engineering

```
WHY THIS MATTERS: WASM usage is growing specifically in places where
developers want to make logic HARDER to reverse engineer than regular
JS — anti-cheat systems, DRM/license checks, crypto/wallet logic,
obfuscated bot-detection, and performance-critical code ported from
C/C++/Rust/Go. If you only know JS reversing, WASM-protected logic is
completely opaque to you — this is genuinely the most valuable new
skill in this guide for modern web targets.
```

**Locating and Extracting WASM Modules**

```javascript
// Find WASM modules loaded by the page:
performance
  .getEntriesByType('resource')
  .filter((r) => r.name.endsWith('.wasm'))
  .forEach((r) => console.log('WASM module:', r.name))

// Hook WebAssembly instantiation to capture modules as they load,
// including ones loaded dynamically/lazily after initial page load:
const originalInstantiate = WebAssembly.instantiate
WebAssembly.instantiate = function (...args) {
  console.log('WASM instantiate called:', args)
  return originalInstantiate.apply(this, args)
}

const originalInstantiateStreaming = WebAssembly.instantiateStreaming
WebAssembly.instantiateStreaming = function (source, importObject) {
  console.log('WASM instantiateStreaming:', source)
  return originalInstantiateStreaming.call(this, source, importObject)
}

// Download a WASM file directly once you have its URL:
// curl -O https://target.com/static/module.wasm
```

**Disassembling WASM — wasm2wat (Human-Readable Text Format)**

```bash
# Install the WebAssembly Binary Toolkit (WABT) — the standard
# foundational toolset:
sudo apt install -y wabt
# Or build from source: https://github.com/WebAssembly/wabt

# Convert binary .wasm to readable WAT (WebAssembly Text format):
wasm2wat module.wasm -o module.wat
less module.wat

# WAT looks like a stack-based assembly language:
#   (func $add (param $a i32) (param $b i32) (result i32)
#     local.get $a
#     local.get $b
#     i32.add)
# This is genuinely readable once you learn ~15 core instructions
# (local.get/set, i32.add/sub/mul, call, br_if, etc.) — far more
# approachable than x86 disassembly.

# Round-trip back to binary after editing WAT (useful for patching):
wat2wasm module.wat -o module_patched.wasm

# Validate a module without fully disassembling (sanity check):
wasm-validate module.wasm

# Dump just the imports/exports — usually your FIRST step, since this
# tells you exactly what JS functions the WASM calls OUT to (imports)
# and what functions JS can call INTO the WASM module (exports) —
# this is your map of the JS<->WASM boundary, which is where the
# interesting logic usually crosses:
wasm-objdump -x module.wasm
```

**Decompiling WASM to (Pseudo) C — Higher-Level Analysis**

```bash
# wasm2c (also part of WABT) — generates actual C code, useful when
# you want to compile/modify the logic rather than just read it:
wasm2c module.wasm -o module.c

# Ghidra has a WASM processor module (community-maintained, may need
# manual install depending on Ghidra version) that gives you the same
# decompiler-driven workflow you'd use for a native binary — function
# graphs, cross-references, renaming — applied to WASM bytecode:
#   https://github.com/nneonneo/ghidra-wasm-plugin

# For quick interactive exploration without a full Ghidra setup,
# wasm-decompile (part of WABT, newer versions) produces a more
# readable pseudo-C-like output directly:
wasm-decompile module.wasm -o module_decompiled.c
```

**Tracing WASM Execution from JavaScript (Dynamic Analysis)**

```javascript
// The most practical real-world technique: you usually don't need to
// fully reverse the WASM logic statically if you can intercept the
// imports/exports at the JS boundary and watch VALUES flow through:

WebAssembly.instantiateStreaming(fetch('module.wasm'), {
  env: {
    // If the original import object had a function called
    // 'js_log_value', wrap it to see every value the WASM module
    // sends back out to JS:
    js_log_value: function (val) {
      console.log('WASM -> JS value:', val)
    },
  },
}).then(({ instance }) => {
  // Wrap EVERY exported function to log calls and results:
  const wrapped = {}
  for (const [name, fn] of Object.entries(instance.exports)) {
    if (typeof fn === 'function') {
      wrapped[name] = function (...args) {
        console.log(`CALL ${name}(${args.join(', ')})`)
        const result = fn(...args)
        console.log(`  -> ${result}`)
        return result
      }
    }
  }
  window.__wasmExports = wrapped
})

// Reading/writing WASM linear memory directly — most interesting
// data (strings, buffers, structs) lives in a flat ArrayBuffer you
// can inspect at any point during execution:
function dumpWasmMemory(instance, offset, length) {
  const mem = new Uint8Array(instance.exports.memory.buffer, offset, length)
  return new TextDecoder().decode(mem) // if it's a UTF-8 string
}

// Set a "breakpoint" by wrapping a suspected validation function and
// inspecting its arguments right before the real call — e.g. for a
// license-check or anti-cheat WASM function, this reveals exactly
// what input format it expects and what it returns on success/failure.
```

**Practical Notes on WASM Reversing Difficulty**

```
- WASM compiled from Rust/C++ tends to be HARDER to reverse than
  hand-written WASM or compiler output from simpler languages,
  because of aggressive inlining, monomorphization, and lack of any
  meaningful symbol names by default (release builds strip these).
- Check for a matching .wasm.map source map FIRST — same idea as JS
  source maps, and equally often mistakenly shipped in production.
- emscripten-built modules (common for C/C++ → WASM) often pair the
  .wasm with a "glue" .js file that's NOT obfuscated and clearly shows
  how exports are called — always check the accompanying glue JS
  before diving into the WASM disassembly itself.
- This is a genuinely deep, specialized rabbit hole — treat the above
  as a strong starting toolkit, not full mastery. The same
  disassembly/dynamic-analysis mindset from your reverse engineering
  background transfers directly; WASM is just a different ISA to learn.
```

### Week 10: Advanced Topics — PWAs, Service Workers, WebRTC, CSP Bypass

**Service Worker Deep Analysis**

```javascript
// Service workers can intercept ALL network requests, cache
// responses, and run even when the tab is closed — understanding
// what one does is essential for fully understanding an app's
// offline/caching/push-notification behavior:

navigator.serviceWorker.getRegistrations().then((regs) => {
  regs.forEach((reg) => {
    console.log('Scope:', reg.scope)
    console.log('Script URL:', reg.active?.scriptURL)
  })
})

// Fetch and read the service worker's own source directly:
fetch(navigator.serviceWorker.controller?.scriptURL || '/sw.js')
  .then((r) => r.text())
  .then((code) => console.log(code))

// Inspect everything currently cached by the service worker (often
// reveals the FULL set of API responses/assets the app prefetches,
// including endpoints never triggered during your normal browsing):
caches.keys().then(async (names) => {
  for (const name of names) {
    const cache = await caches.open(name)
    const requests = await cache.keys()
    console.log(
      `Cache "${name}":`,
      requests.map((r) => r.url),
    )
  }
})

// Force-unregister a service worker during testing (useful when its
// caching behavior is interfering with seeing fresh server responses):
navigator.serviceWorker.getRegistrations().then((regs) => {
  regs.forEach((reg) => reg.unregister())
})
```

**WebRTC IP Leak Analysis**

```javascript
// WebRTC can leak a user's real local/public IP even behind a VPN if
// the app uses it (video chat, P2P features) — relevant both for
// privacy auditing your own app and for OSINT/forensics work
// investigating what a target page might reveal about a user:

function getWebRTCIPs() {
  return new Promise((resolve) => {
    const ips = new Set()
    const pc = new RTCPeerConnection({ iceServers: [{ urls: 'stun:stun.l.google.com:19302' }] })
    pc.createDataChannel('')
    pc.onicecandidate = (e) => {
      if (!e.candidate) {
        resolve([...ips])
        pc.close()
        return
      }
      const match = e.candidate.candidate.match(/([0-9]{1,3}\.){3}[0-9]{1,3}/)
      if (match) ips.add(match[0])
    }
    pc.createOffer().then((offer) => pc.setLocalDescription(offer))
  })
}
getWebRTCIPs().then((ips) => console.log('WebRTC-exposed IPs:', ips))
```

**Content Security Policy (CSP) Analysis and Common Bypass Patterns**

```javascript
// Read the actual enforced policy:
fetch(location.href).then((r) => console.log(r.headers.get('content-security-policy')))
// Also check the meta-tag variant:
document.querySelector('meta[http-equiv="Content-Security-Policy"]')?.content

// Common CSP weaknesses worth checking for (authorized testing only):
//   - 'unsafe-inline' in script-src → inline <script> still executes,
//     defeats most of CSP's XSS-mitigation value
//   - A whitelisted CDN domain that also hosts user-uploadable/
//     attacker-controllable content (e.g. *.googleusercontent.com,
//     some JSONP-capable API domains) → can be abused to load
//     attacker JS from an "allowed" origin
//   - 'unsafe-eval' present → defeats protection against eval-based
//     injection sinks
//   - Missing object-src 'none' → allows Flash/legacy plugin-based
//     bypasses on older browsers
//   - A report-only policy (Content-Security-Policy-Report-Only)
//     mistakenly left in place instead of enforcing — it LOGS
//     violations but doesn't actually BLOCK anything

// CSP Evaluator (Google's tool) automates this analysis far more
// thoroughly than manual inspection:
//   https://csp-evaluator.withgoogle.com/ (paste the policy string)
```

---

## Phase 6: Mobile Web Apps

### Week 11: Mobile-Specific Web Reversing

**Responsive/Mobile-Detection Logic Extraction**

```javascript
// Many apps serve DIFFERENT behavior/API responses based on detected
// device type — understanding the detection logic itself often
// reveals hidden mobile-only API endpoints or feature flags:

// Find user-agent-based branching in bundled JS:
fetch('/static/js/main.js')
  .then((r) => r.text())
  .then((code) => {
    const uaChecks = code.match(/navigator\.userAgent[^;]{0,200}/g)
    console.log('UA-based branches found:', uaChecks)
  })

// Test how the SAME endpoint responds to different User-Agent values
// — APIs sometimes have a parallel, less-secured mobile API surface:
const userAgents = {
  desktop: 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) Chrome/124.0',
  iosApp: 'MyApp/3.2.1 (iPhone; iOS 17.4; Scale/3.00)',
  androidApp: 'MyApp/3.2.1 (Linux; Android 14; Pixel 8)',
}
for (const [label, ua] of Object.entries(userAgents)) {
  fetch('https://api.target.com/data', { headers: { 'User-Agent': ua } }).then((r) =>
    console.log(label, r.status, r.headers.get('content-type')),
  )
}
```

**Hybrid App WebViews — Bridging JS to Native**

```javascript
// React Native WebView / Cordova / Capacitor apps expose a JS<->native
// bridge — finding and understanding this bridge is the core of
// hybrid-app web reversing, since it's where "web" logic gets the
// power to do native-app things (camera, filesystem, push tokens):

// Cordova: look for window.cordova and the exec bridge
if (window.cordova) {
  console.log('Cordova bridge present. Plugins:', Object.keys(window.cordova.plugins || {}))
}

// React Native WebView: messages typically flow through
// window.ReactNativeWebView.postMessage — find every call site:
fetch('bundle.js')
  .then((r) => r.text())
  .then((code) => {
    const bridgeCalls = code.match(/ReactNativeWebView\.postMessage\([^)]*\)/g)
    console.log('RN bridge calls:', bridgeCalls)
  })

// Capacitor: window.Capacitor exposes registered native plugins directly
if (window.Capacitor) {
  console.log('Capacitor plugins:', Object.keys(window.Capacitor.Plugins || {}))
}

// PRACTICAL VALUE: the bridge is frequently where security checks are
// WEAKEST, because developers trust "their own" WebView content
// implicitly — messages sent across this bridge often skip
// validation that the equivalent native-to-native or web-to-server
// call would enforce. This is a well-known real-world vulnerability
// class in hybrid apps (overly trusting bridge input).
```

**Mobile Network Traffic Interception (Device-Level, Not Just Browser)**

```bash
# To see a NATIVE mobile app's traffic (not a mobile browser tab),
# you need device-level proxying, since there's no DevTools to attach to:

# 1. Configure the phone's WiFi to use your laptop's IP as an HTTP
#    proxy (Settings > WiFi > network details > Configure Proxy)
#    pointing at mitmproxy/Burp running on your machine.

# 2. Install the proxy tool's CA certificate ON THE DEVICE so HTTPS
#    traffic can be decrypted (mitmproxy serves this automatically at
#    http://mitm.it when the device is correctly proxied):
#    mitmproxy → visit mitm.it from the device browser → install cert

# 3. Many apps now implement CERTIFICATE PINNING specifically to
#    defeat this — the app refuses to trust ANY certificate except a
#    hardcoded one, even a validly-installed CA. Defeating pinning
#    for AUTHORIZED testing on a device you control typically requires:
#      - Frida (dynamic instrumentation) with a pinning-bypass script,
#        injected into the running app process:
frida -U -f com.target.app -l ssl-pinning-bypass.js --no-pause
#      - objection (built on Frida, much friendlier CLI for exactly
#        this class of task):
pip3 install --break-system-packages objection
objection -g com.target.app explore
# > android sslpinning disable

# This is meaningfully more involved than browser-based web reversing
# — flag it honestly as its own skill track (mobile app instrumentation)
# rather than something covered by browser DevTools knowledge alone.
```

### Week 12: Putting It Together — A Full Mobile-Web Workflow

```
1. Identify surface: is this a responsive website, a hybrid WebView
   app, or a native app with an embedded web component?
2. If browser-accessible: use Chrome's remote device debugging
   (chrome://inspect on desktop, connect device via USB with USB
   debugging enabled) to get FULL DevTools access to a page actually
   rendering inside a real mobile browser or in-app WebView.
3. If hybrid: locate and analyze the JS<->native bridge as above.
4. If fully native: this guide's web techniques no longer directly
   apply — that's the boundary into native mobile reverse engineering
   (APK/IPA decompilation, Frida instrumentation, Jadx/Ghidra for
   native code) which is a related but distinct discipline from web
   reversing, worth treating as its own follow-up study track given
   your existing reverse engineering background.
```

---

## Real-World Projects

```
PROJECT 1 — Build a Source Map Scanner
  Write a Node.js CLI tool that takes a domain, finds every loaded JS
  bundle, checks for an accompanying .map file, and if found, dumps
  the full reconstructed source tree. Combine the techniques from
  Phase 2 (Week 4) into one reusable tool — genuinely useful for real
  bug bounty recon, not just an exercise.

PROJECT 2 — API Surface Mapper
  Given a target's main bundle, automatically extract every API path
  literal, every parameter name referenced, and every header the app
  sends, producing a structured JSON "map" of the discovered API
  surface — a lightweight, JS-aware alternative to ffuf-only discovery
  that catches paths a generic wordlist would miss.

PROJECT 3 — GraphQL Schema Diff Tool
  Run introspection (or clairvoyance) against the SAME API at two
  points in time (e.g. before/after a deploy you're tracking), diff
  the schemas, and flag newly added/removed queries and mutations —
  useful for monitoring competitor feature rollouts or your own API's
  unintended surface growth.

PROJECT 4 — WASM Call Tracer Extension
  Build a small browser extension that auto-injects the WASM
  import/export wrapping technique from Phase 5 into every page,
  logging all WASM boundary calls to a panel — turns a one-off console
  snippet into a reusable analysis tool.

PROJECT 5 — Forensics Crossover: Browser Artifact + JS Correlation Tool
  Given a browser profile directory (History, Local Storage leveldb
  files, IndexedDB) extracted during a forensic acquisition, AND the
  site's own JS bundle, write a tool that maps storage KEYS found in
  the artifacts back to the bundle code that WROTE them — directly
  combining this guide with your digital forensics knowledge base to
  answer "what app logic produced this stored value" during an
  investigation.
```

---

## Tools & Automation — Consolidated Reference

```bash
# ═══════════════ BROWSER / PROXY ═══════════════
Chrome DevTools                  # built-in, primary daily-driver tool
Burp Suite Community/Pro          # intercepting proxy, Repeater, WS history
mitmproxy / mitmweb / mitmdump     # scriptable CLI proxy, Python addons
Charles Proxy / Fiddler              # GUI alternatives, Map Local feature

# ═══════════════ JS DEOBFUSCATION ═══════════════
de4js (web)                        # quick interactive deobfuscation
restringer (npm)                    # AST-based, targets javascript-obfuscator
webcrack (npm)                       # unpacks bundles AND deobfuscates
@babel/parser + traverse + generator  # manual AST work, full control
prettier / js-beautify                 # pretty-printing minified code

# ═══════════════ SECRET SCANNING ═══════════════
trufflehog filesystem ./dist          # entropy + pattern-based secret scan
gitleaks detect --source .             # git-history-aware secret scan

# ═══════════════ API DISCOVERY ═══════════════
ffuf -u URL/FUZZ -w wordlist.txt         # fast endpoint brute-forcing
arjun -u URL -m GET                       # parameter discovery
SecLists (github.com/danielmiessler/SecLists)  # the wordlists ffuf/arjun need

# ═══════════════ GRAPHQL ═══════════════
clairvoyance                              # schema recon w/o introspection
graphql-cop                                # misconfiguration scanner
InQL (Burp extension)                       # query library generation

# ═══════════════ JWT ═══════════════
jwt_tool                                     # alg:none, cracking, alg confusion
jwt.io (web, offline-capable)                 # quick decode/inspect

# ═══════════════ WEBASSEMBLY ═══════════════
wabt (wasm2wat/wat2wasm/wasm-objdump)          # disassembly/reassembly toolkit
ghidra-wasm-plugin                              # full decompiler workflow for WASM

# ═══════════════ FRAMEWORK FINGERPRINTING ═══════════════
wappalyzer (CLI/lib)                              # tech stack identification
React/Vue/Angular DevTools extensions               # framework-native inspection

# ═══════════════ MOBILE / HYBRID ═══════════════
Frida                                                # dynamic instrumentation,
                                                       # SSL pinning bypass
objection                                              # Frida-based CLI wrapper,
                                                       # much friendlier UX
chrome://inspect                                        # remote-debug a real
                                                       # mobile browser/WebView

# ═══════════════ HAR / TRAFFIC ANALYSIS ═══════════════
Node.js + fs (parse exported HAR files programmatically, as shown in Phase 1)
```

---

## Legal & Ethical Considerations

```
This guide assumes one of the following contexts for everything above:

  1. You own the application/API being analyzed, OR
  2. You have EXPLICIT, WRITTEN authorization (a bug bounty program's
     published scope, a signed pentest engagement, an employer's
     internal app you're authorized to test), OR
  3. You're working entirely against your OWN deployed test
     environment / a deliberately-vulnerable practice app (PortSwigger
     Web Security Academy, OWASP Juice Shop, your own local builds), OR
  4. You're conducting purely passive analysis explicitly permitted by
     a site's Terms of Service (e.g. reading publicly-served JS that
     loads in any visitor's browser is generally different, legally,
     from actively probing authentication/authorization boundaries —
     but ToS and applicable law vary by jurisdiction and by exactly
     what you do, so don't assume "it's public in my browser" blankly
     covers active testing).

KEY DISTINCTIONS THAT MATTER LEGALLY (general principles, not legal
advice — consult an actual lawyer for a specific situation):

  - Reading/analyzing JS that your OWN browser legitimately downloaded
    while visiting a page normally is meaningfully different from
    actively probing endpoints, brute-forcing parameters, or bypassing
    auth — the former is closer to "reading what was sent to you," the
    latter starts to look like unauthorized access attempts in many
    jurisdictions (e.g. under frameworks like the US CFAA, UK Computer
    Misuse Act, or equivalent local laws).
  - Bug bounty programs define SCOPE precisely — testing outside
    documented scope, even on a domain owned by the same company that
    runs the program, can be a violation of the program's own terms
    and may not be authorized at all.
  - Discovering an unprotected/secret API endpoint does NOT mean you're
    authorized to use it beyond what's needed to REPORT its existence
    responsibly — pulling large amounts of data through a found
    endpoint goes well beyond "I found a misconfiguration" into
    potentially unauthorized data access.
  - WASM/anti-tampering bypass techniques (Phase 5) specifically can
    intersect with anti-circumvention laws (e.g. DMCA Section 1201 in
    the US) when applied to DRM-protected content — this is a distinct
    legal risk category from general web app testing, worth being
    aware of explicitly if you ever apply these techniques to
    DRM/license-check logic specifically.

RESPONSIBLE DISCLOSURE, IF YOU FIND A REAL VULNERABILITY:
  1. Stop testing once you've confirmed the issue — don't escalate
     further than needed to demonstrate impact.
  2. Check for a security.txt file (/.well-known/security.txt) or a
     published bug bounty/vulnerability disclosure policy FIRST.
  3. Report privately, with clear reproduction steps, BEFORE any
     public disclosure.
  4. Respect any stated disclosure timeline the organization provides.
  5. Document everything you did, the same disciplined way described
     in your forensics report-writing notes — "what did I do, when,
     and what exactly did I observe" protects you as much as it
     protects the organization.

Given your specific goal of supporting police digital forensics work:
the techniques in this guide are equally applicable in REVERSE — when
investigating a malicious or fraudulent website as part of a case
(phishing pages, fraudulent e-commerce sites, scam web apps), the same
DevTools/JS-analysis/network-analysis skills let you reconstruct what
the page actually did, what data it exfiltrated and to where, and what
backend infrastructure it talked to — directly complementing the
email/malware/network forensics sections of your other notes.
```

---

_This guide extends the original web reverse engineering notes with deeper deobfuscation
workflows, modern Webpack 5/source-map recovery, GraphQL-specific attack classes, a full
WebAssembly reversing section, mobile/hybrid-app bridge analysis, and an explicit legal/ethical
framework tied to your forensics-support goals._

_The browser is the most observable runtime you will ever reverse engineer — almost everything
is inspectable by design. Use that transparency deliberately and within scope._ 🔬
