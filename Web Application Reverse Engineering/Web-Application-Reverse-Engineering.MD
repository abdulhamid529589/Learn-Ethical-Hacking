# 🔬 Complete Web Application Reverse Engineering Guide

## For MERN/Full Stack Developers

> 🎯 **Mission**: Master reverse engineering web applications, APIs, JavaScript, and modern web technologies from a developer's perspective.

---

## Table of Contents

1. [Why Web Reverse Engineering?](#why-web-reverse-engineering)
2. [Prerequisites for Web Developers](#prerequisites)
3. [Phase 1: Browser DevTools Mastery (Week 1-2)](#phase-1-browser-devtools)
4. [Phase 2: JavaScript Reverse Engineering (Week 3-4)](#phase-2-javascript-reverse-engineering)
5. [Phase 3: API Reverse Engineering (Week 5-6)](#phase-3-api-reverse-engineering)
6. [Phase 4: React/Vue/Angular Analysis (Week 7-8)](#phase-4-framework-analysis)
7. [Phase 5: WebAssembly & Advanced Topics (Week 9-10)](#phase-5-advanced-topics)
8. [Phase 6: Mobile Web Apps (Week 11-12)](#phase-6-mobile-apps)
9. [Real-World Projects](#real-world-projects)
10. [Tools & Automation](#tools--automation)
11. [Legal & Ethical Considerations](#legal--ethical)

---

## Why Web Reverse Engineering?

### Applications for Web Developers:

- **Competitive Analysis** - Understand how competitors built features
- **API Discovery** - Find undocumented endpoints and features
- **Learning** - Study production code from successful apps
- **Security Research** - Find vulnerabilities in web apps
- **Integration** - Create unofficial APIs for services
- **Automation** - Scrape data, automate tasks
- **Legacy System Recovery** - Understand old codebases without docs
- **Bug Bounty Hunting** - Find security issues for profit
- **Digital Forensics** - Reconstruct what a web app did client-side
  during an incident (relevant given your forensics track — browser
  artifacts + JS analysis often answer "what did this page actually
  send/store" when server logs alone don't show it)

### Real-World Examples:

```javascript
// Discovered: Twitter's internal GraphQL API
// Result: Built third-party Twitter clients

// Discovered: Spotify's web player protocol
// Result: Created custom music players

// Discovered: Netflix's content delivery system
// Result: Optimized streaming for research

// Discovered: Instagram's private API
// Result: Built analytics tools
```

---

## Prerequisites

### Your Existing Knowledge (as MERN Developer):

- ✅ JavaScript/TypeScript
- ✅ Node.js & Express
- ✅ React/Frontend frameworks
- ✅ MongoDB/Database queries
- ✅ REST APIs & HTTP
- ✅ npm/yarn packages
- ✅ Git version control
- ✅ Basic security concepts

### Additional Skills to Learn:

```javascript
// 1. Regular Expressions
const emailRegex = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/
const phoneRegex = /\+?1?\d{9,15}/

// 2. Advanced Browser APIs
console.log(window.performance.getEntries())
console.log(navigator.userAgent)
console.log(document.cookie)

// 3. Network Protocols
// HTTP/HTTPS, WebSocket, Server-Sent Events (SSE)
// TCP/IP basics, DNS resolution

// 4. Cryptography Basics
const crypto = require('crypto')
const hash = crypto.createHash('sha256').update('data').digest('hex')

// 5. Obfuscation & Minification
// Understanding bundled/minified code
// Source maps
```

---

## Phase 1: Browser DevTools Mastery

### Week 1: Chrome DevTools Deep Dive

**Elements Tab - DOM Analysis:**

```javascript
// Console tricks for DOM manipulation

// 1. Find all elements with specific attribute
$$('[data-testid]').forEach((el) => {
  console.log(el.getAttribute('data-testid'), el)
})

// 2. Find React/Vue component instance
$0.__reactFiber$ // React component (select element first)
$0.__vue__ // Vue component

// 3. Monitor element changes
const observer = new MutationObserver((mutations) => {
  mutations.forEach((mutation) => {
    console.log('Changed:', mutation)
  })
})
observer.observe(document.body, {
  childList: true,
  subtree: true,
  attributes: true,
})

// 4. Find hidden elements
$$('*').filter((el) => {
  const style = window.getComputedStyle(el)
  return style.display === 'none' || style.visibility === 'hidden'
})

// 5. Extract all links
Array.from(document.links).map((a) => ({
  text: a.textContent,
  href: a.href,
}))

// 6. Find all forms and inputs
$$('form').forEach((form) => {
  console.log('Form:', form.action)
  $$('input, textarea', form).forEach((input) => {
    console.log('  Input:', input.name, input.type)
  })
})
```

**Network Tab - Traffic Analysis:**

```javascript
// 1. Monitor all requests
// Filter by:
// - XHR/Fetch (API calls)
// - JS (JavaScript files)
// - CSS (Stylesheets)
// - Img (Images)
// - WS (WebSocket)

// 2. Copy as cURL
// Right-click request → Copy → Copy as cURL

// 3. Copy as fetch
// Right-click request → Copy → Copy as fetch

// 4. Block requests
// Right-click request → Block request URL
// Useful for testing fallbacks

// 5. Throttle network
// Throttling: Fast 3G, Slow 3G, Offline
// Test how app handles slow connections

// 6. Preserve log
// Keep network log across page navigations

// 7. Export HAR file
// Right-click → Save all as HAR with content
// Analyze with tools or scripts

// Analyze HAR file with Node.js
const fs = require('fs')
const har = JSON.parse(fs.readFileSync('network.har', 'utf8'))

har.log.entries.forEach((entry) => {
  const request = entry.request
  const response = entry.response

  console.log(`${request.method} ${request.url}`)
  console.log(`Status: ${response.status}`)
  console.log(`Size: ${response.content.size} bytes`)
  console.log('---')
})
```

**Sources Tab - JavaScript Debugging:**

```javascript
// 1. Pretty-print minified code
// Click {} button at bottom

// 2. Set breakpoints
// - Line breakpoints (click line number)
// - Conditional breakpoints (right-click line)
// - XHR/Fetch breakpoints
// - Event listener breakpoints
// - Exception breakpoints

// 3. Watch expressions
// Add expressions to watch
watch('user.id')
watch('localStorage.getItem("token")')

// 4. Call stack
// See function call hierarchy

// 5. Scope variables
// View local, closure, and global variables

// 6. Blackbox scripts
// Ignore third-party code during debugging
// Right-click → Blackbox script

// 7. Override files
// Sources → Overrides → Select folder
// Edit and save files locally

// 8. Workspace
// Map local files to network resources
```

**Application Tab - Storage Analysis:**

```javascript
// 1. Local Storage
Object.keys(localStorage).forEach((key) => {
  console.log(key, localStorage.getItem(key))
})

// Extract all localStorage
const allStorage = {}
for (let i = 0; i < localStorage.length; i++) {
  const key = localStorage.key(i)
  allStorage[key] = localStorage.getItem(key)
}
console.log(JSON.stringify(allStorage, null, 2))

// 2. Session Storage
Object.keys(sessionStorage).forEach((key) => {
  console.log(key, sessionStorage.getItem(key))
})

// 3. Cookies
document.cookie.split(';').forEach((cookie) => {
  console.log(cookie.trim())
})

// Parse cookies
function parseCookies() {
  return document.cookie.split(';').reduce((cookies, cookie) => {
    const [name, value] = cookie.split('=').map((c) => c.trim())
    cookies[name] = decodeURIComponent(value)
    return cookies
  }, {})
}

// 4. IndexedDB
// Application → IndexedDB → Inspect databases

// Access IndexedDB programmatically
const request = indexedDB.open('myDatabase')
request.onsuccess = (event) => {
  const db = event.target.result
  const transaction = db.transaction(['myStore'], 'readonly')
  const store = transaction.objectStore('myStore')
  const getRequest = store.getAll()
  getRequest.onsuccess = () => {
    console.log('All data:', getRequest.result)
  }
}

// 5. Cache Storage
caches.keys().then((names) => {
  names.forEach((name) => {
    caches.open(name).then((cache) => {
      cache.keys().then((requests) => {
        console.log(`Cache: ${name}`)
        requests.forEach((req) => {
          console.log('  ', req.url)
        })
      })
    })
  })
})

// 6. Service Workers
navigator.serviceWorker.getRegistrations().then((registrations) => {
  registrations.forEach((registration) => {
    console.log('SW:', registration.active.scriptURL)
  })
})
```

**Console Tab - Advanced Techniques:**

```javascript
// 1. Monitor function calls
monitor(functionName) // Logs when function is called
unmonitor(functionName)

// 2. Monitor events
monitorEvents(window, 'click') // Log all click events
monitorEvents(document.body) // Log all events
unmonitorEvents(window)

// 3. Get event listeners
getEventListeners(document.body)

// 4. Copy to clipboard
copy(complexObject) // Copies JSON to clipboard

// 5. Console API
console.time('operation')
// ... code ...
console.timeEnd('operation')

console.table([
  { name: 'John', age: 30 },
  { name: 'Jane', age: 25 },
])

console.trace() // Print stack trace

// 6. Command Line API
$0 // Currently selected element
$_ // Previously evaluated expression
$$() // querySelectorAll
$x() // XPath query
keys() // Object.keys
values() // Object.values
dir() // console.dir

// 7. Preserve log
// Keep console log across page reloads

// 8. Filter logs
// Filter by: Errors, Warnings, Info, Verbose, User Messages
```

### Week 2: Advanced Browser Tools

**Burp Suite for Web Apps:**

```
# Setup
1. Download Burp Suite Community
2. Configure browser proxy (127.0.0.1:8080)
3. Install Burp CA certificate

# Features for Web Analysis
- Intercept HTTP/HTTPS traffic
- Modify requests/responses
- Replay requests (Repeater)
- Automated scanning (Pro)
- Session handling
- Macro recording

# Workflow
1. Browse app with proxy on
2. Analyze traffic in HTTP history
3. Send interesting requests to Repeater
4. Modify and replay
5. Test different payloads
```

**Fiddler Classic/Fiddler Everywhere:**

```javascript
// Auto-responder rules
// Redirect requests to local files

# Rule examples:
REGEX:.*\.js$           → C:\local\app.js
REGEX:.*api/user.*      → C:\local\user.json
*.example.com/*         → http://localhost:3000/$1

// Composer
// Create custom requests

POST https://api.example.com/login
Headers:
  Content-Type: application/json
Body:
  {"username":"test","password":"test"}
```

**Charles Proxy:**

```
# Features
- SSL proxying
- Throttling
- Breakpoints
- Rewrite rules
- Map local
- Reverse proxy

# Map Local (replace remote with local)
Tools → Map Local
Host: api.example.com
Path: /api/
Local Path: /Users/you/mock-api/
```

**mitmproxy — Scriptable CLI Alternative:**

```bash
# Installed easily on Parrot OS, scriptable in Python, and the
# strongest free option for AUTOMATING traffic manipulation rather
# than clicking through a GUI for every test:
pip3 install mitmproxy --break-system-packages

mitmproxy                          # interactive TUI
mitmweb                            # web-based UI on localhost:8081
mitmdump -w capture.flow           # headless capture to a file

# Scripted interception — e.g. auto-modify every response containing
# a feature flag, to test hidden/disabled features:
cat > flip_flags.py << 'EOF'
from mitmproxy import http
import json

def response(flow: http.HTTPFlow) -> None:
    if "config" in flow.request.pretty_url and flow.response.headers.get("content-type", "").startswith("application/json"):
        try:
            data = json.loads(flow.response.text)
            if "featureFlags" in data:
                for k in data["featureFlags"]:
                    data["featureFlags"][k] = True
                flow.response.text = json.dumps(data)
        except Exception:
            pass
EOF
mitmdump -s flip_flags.py
```

---

## Phase 2: JavaScript Reverse Engineering

### Week 3: Deobfuscation & Analysis

**Understanding Minified Code:**

```javascript
// Original code
function calculateTotal(items) {
  return items.reduce((sum, item) => sum + item.price, 0)
}

// Minified
function a(b) {
  return b.reduce((c, d) => c + d.e, 0)
}

// Steps to understand:
// 1. Use DevTools pretty-print ({} button)
// 2. Rename variables meaningfully
// 3. Add breakpoints to trace execution
// 4. Watch expressions to see values
```

**Common Obfuscation Techniques:**

```javascript
// 1. String Array Obfuscation
const _0x1234 = ['hello', 'world', 'test']
const _0x5678 = function (index) {
  return _0x1234[index]
}
console.log(_0x5678(0)) // 'hello'

// Deobfuscate: Replace calls with actual strings

// 2. Control Flow Flattening
const state = 0
while (true) {
  switch (state) {
    case 0:
      console.log('Step 1')
      state = 1
      break
    case 1:
      console.log('Step 2')
      state = 2
      break
    case 2:
      return
  }
}

// Deobfuscate: Follow control flow manually

// 3. Dead Code Injection
if (false) {
  // This never executes but adds confusion
  console.log('junk')
}
const x = true ? 'value' : 'never_used'

// 4. Proxy Objects
const obj = new Proxy(
  {},
  {
    get: function (target, prop) {
      return target[prop]
    },
  },
)

// 5. Encoding
const encoded = 'aGVsbG8=' // Base64
const decoded = atob(encoded) // 'hello'

// Hex encoding
const hex = '68656c6c6f'
const text = Buffer.from(hex, 'hex').toString() // 'hello'
```

**Recognizing javascript-obfuscator.io Output (The Most Common Tool in the Wild)**

```javascript
// The vast majority of obfuscated production JS you'll encounter was
// run through javascript-obfuscator (npm: javascript-obfuscator) —
// recognizing its signature patterns saves huge amounts of time versus
// treating every obfuscated file as a unique puzzle:

// SIGNATURE 1: a single big string array + a "decoder" function near
// the top of the file, referenced via short hex-named functions:
var _0x4f2a = ['log', 'value', 'split', ...];
function _0x1a2b(_0x.., _0x..) { /* index math into _0x4f2a */ }

// SIGNATURE 2: self-defending code (an IIFE that detects if the
// function's source has been modified/reformatted, and crashes/loops
// if so — this is WHY pretty-printing alone sometimes breaks
// obfuscated code; you may need to strip self-defense first)

// SIGNATURE 3: control-flow flattening via a dispatch array:
var _0x.. = ['1', '0', '3', '2'];  // execution order, scrambled
(function() {
    var _order = _0x..;
    while (true) {
        switch (_order[index++]) {
            case '0': /* ... */ continue;
            case '1': /* ... */ continue;
        }
        break;
    }
})();

// PRACTICAL DEOBFUSCATION WORKFLOW:
// 1. Try a dedicated tool FIRST before manual AST work — purpose-built
//    deobfuscators handle the common cases automatically:
//      - de4js (web, lelinhtinh.github.io/de4js) — handles many
//        common packers/obfuscators including eval-based packing
//      - restringer (npm: restringer) — actively maintained, designed
//        specifically to reverse javascript-obfuscator.io output via
//        AST-based constant folding and string-array resolution
//      - webcrack (npm: webcrack) — unpacks webpack/browserify bundles
//        AND deobfuscates javascript-obfuscator output in one pass
//
// 2. If those don't fully resolve it, fall back to manual Babel AST
//    work (below) targeting the SPECIFIC pattern you're seeing.
```

**Automated Deobfuscation:**

```javascript
// Using de4js (https://lelinhtinh.github.io/de4js/)
// Or use Node.js tools

const { parse } = require('@babel/parser')
const traverse = require('@babel/traverse').default
const generate = require('@babel/generator').default
const t = require('@babel/types')

const code = `function _0x1234(){return"hello";}`

// Parse code
const ast = parse(code)

// Transform AST
traverse(ast, {
  FunctionDeclaration(path) {
    // Rename function
    if (path.node.id.name.startsWith('_0x')) {
      path.node.id.name = 'deobfuscated_function'
    }
  },
})

// Generate code
const output = generate(ast, {}, code)
console.log(output.code)
```

**Resolving String-Array Obfuscation with a Babel Pass (Full Working Example)**

```javascript
// This is the single most common deobfuscation task you'll face.
// Given a string array + decoder function pattern, statically evaluate
// every call to the decoder and replace it with the literal string —
// turning unreadable `_0x1a2b(0x3)` calls back into `"username"`.

const { parse } = require('@babel/parser')
const traverse = require('@babel/traverse').default
const generate = require('@babel/generator').default
const t = require('@babel/types')
const fs = require('fs')

const code = fs.readFileSync('obfuscated.js', 'utf8')
const ast = parse(code)

// Step 1: locate the string array and the decoder function name
let stringArray = null
let decoderName = null

traverse(ast, {
  VariableDeclarator(path) {
    if (
      t.isArrayExpression(path.node.init) &&
      path.node.init.elements.every((el) => t.isStringLiteral(el))
    ) {
      stringArray = path.node.init.elements.map((el) => el.value)
    }
  },
  FunctionDeclaration(path) {
    // heuristic: a function that takes one numeric arg and
    // indexes into the array we just found
    if (path.node.params.length <= 2 && /_0x/.test(path.node.id.name)) {
      decoderName = path.node.id.name
    }
  },
})

console.log('Found string array of length:', stringArray?.length)
console.log('Found likely decoder function:', decoderName)

// Step 2: replace every call to the decoder with the literal string
traverse(ast, {
  CallExpression(path) {
    if (t.isIdentifier(path.node.callee, { name: decoderName })) {
      const indexArg = path.node.arguments[0]
      if (t.isNumericLiteral(indexArg) && stringArray) {
        const value = stringArray[indexArg.value]
        if (value !== undefined) {
          path.replaceWith(t.stringLiteral(value))
        }
      }
    }
  },
})

const output = generate(ast, { comments: true }, code)
fs.writeFileSync('deobfuscated.js', output.code)
console.log('Deobfuscation pass complete -> deobfuscated.js')

// NOTE: real obfuscator output often adds an extra rotation/offset
// step to the array (a self-invoking function that .push()/.shift()s
// the array before use specifically to defeat naive static
// resolution like this). If indices don't line up, execute the
// rotation logic itself in a sandboxed Node `vm` context first to
// get the array in its FINAL runtime order before indexing into it.
```

**JavaScript AST Analysis:**

```javascript
// Abstract Syntax Tree manipulation

const esprima = require('esprima')
const escodegen = require('escodegen')

const code = `
function secret() {
    const key = "hidden";
    return key;
}
`

// Parse to AST
const ast = esprima.parseScript(code)

// Explore AST
function walk(node, callback) {
  callback(node)
  for (let key in node) {
    if (node[key] && typeof node[key] === 'object') {
      walk(node[key], callback)
    }
  }
}

walk(ast, (node) => {
  if (node.type === 'Literal' && typeof node.value === 'string') {
    console.log('Found string:', node.value)
  }
})

// Modify AST
ast.body[0].id.name = 'revealed'

// Generate code
const modified = escodegen.generate(ast)
console.log(modified)
```

**Extracting Embedded Data:**

```javascript
// Find all strings in JavaScript
const jsCode = document.scripts[0].textContent
const strings = jsCode.match(/"([^"\\]|\\.)*"|'([^'\\]|\\.)*'/g)
console.log(strings)

// Find all URLs
const urls = jsCode.match(/https?:\/\/[^\s"']+/g)
console.log(urls)

// Find API endpoints
const endpoints = jsCode.match(/\/api\/[^\s"']+/g)
console.log(endpoints)

// Find keys/secrets (careful!)
const possibleKeys = jsCode.match(/[a-zA-Z0-9]{32,}/g)
console.log(possibleKeys)

// Extract configuration objects
const configMatch = jsCode.match(/config\s*=\s*({[\s\S]*?});/)
if (configMatch) {
  const config = eval('(' + configMatch[1] + ')')
  console.log(config)
}
```

> ⚠️ **Never use `eval()` on untrusted/unknown code outside a sandboxed
> environment.** A page's own bundled config object is usually safe to
> `eval`, but if you're analyzing a sample you don't fully trust
> (e.g. while investigating a malicious page during a forensics case),
> use Node's `vm` module with a fresh, isolated context instead —
> `eval` in your real browser console runs with your real cookies,
> session, and origin privileges.

**Secret/Key Scanning at Scale (TruffleHog-Style Regexes)**

```javascript
// Beyond a generic 32+ char regex, real secret-scanning tools match
// SPECIFIC known key formats, which dramatically cuts false positives:

const secretPatterns = {
  awsAccessKey: /AKIA[0-9A-Z]{16}/g,
  awsSecretKey: /(?:[A-Za-z0-9/+=]{40})/g, // pair with context check
  googleApiKey: /AIza[0-9A-Za-z\-_]{35}/g,
  stripeKey: /sk_live_[0-9a-zA-Z]{24,}/g,
  stripePublic: /pk_live_[0-9a-zA-Z]{24,}/g,
  githubToken: /gh[pousr]_[A-Za-z0-9]{36,}/g,
  slackToken: /xox[baprs]-[0-9A-Za-z-]{10,}/g,
  jwtToken: /eyJ[A-Za-z0-9_-]+\.[A-Za-z0-9_-]+\.[A-Za-z0-9_-]+/g,
  privateKeyBlock: /-----BEGIN (RSA |EC )?PRIVATE KEY-----/g,
  genericApiKey: /['"]?api[_-]?key['"]?\s*[:=]\s*['"][0-9a-zA-Z\-_]{16,}['"]/gi,
}

function scanForSecrets(code) {
  const findings = {}
  for (const [name, pattern] of Object.entries(secretPatterns)) {
    const matches = code.match(pattern)
    if (matches) findings[name] = [...new Set(matches)]
  }
  return findings
}

// Run against every loaded script:
Promise.all(
  Array.from(document.scripts)
    .filter((s) => s.src)
    .map((s) => fetch(s.src).then((r) => r.text())),
).then((allCode) => {
  allCode.forEach((code, i) => {
    const found = scanForSecrets(code)
    if (Object.keys(found).length) console.log(`Script ${i}:`, found)
  })
})

// For real auditing at repo/build scale, prefer the dedicated CLI
// tools over ad-hoc regex — they maintain pattern databases and
// entropy-based detection that catches what fixed regexes miss:
//   trufflehog filesystem ./dist --only-verified
//   gitleaks detect --source . -v
```

### Week 4: Webpack/Bundler Analysis

**Analyzing Webpack Bundles:**

```javascript
// Webpack bundle structure
;(function (modules) {
  var installedModules = {}

  function __webpack_require__(moduleId) {
    // Module caching
    if (installedModules[moduleId]) {
      return installedModules[moduleId].exports
    }
    // Create new module
    var module = (installedModules[moduleId] = {
      i: moduleId,
      l: false,
      exports: {},
    })
    // Execute module
    modules[moduleId].call(module.exports, module, module.exports, __webpack_require__)
    module.l = true
    return module.exports
  }

  // Entry point
  return __webpack_require__(0)
})([
  // Module 0
  function (module, exports, __webpack_require__) {
    // Your code here
  },
  // Module 1
  function (module, exports) {
    // More code
  },
])

// Extract all modules
const modules = []
const bundleCode = document.scripts[0].textContent
const moduleRegex = /\/\*\*\*\/ \(function\(module, exports.*?\}\),?/gs
const matches = bundleCode.matchAll(moduleRegex)

for (const match of matches) {
  modules.push(match[0])
}

console.log(`Found ${modules.length} modules`)
```

**Modern Webpack 5 Module IDs and webpack-unpack-style Recovery**

```javascript
// Modern Webpack 5 bundles often use a numeric/hashed module ID
// scheme (rather than the older readable array-index style), loaded
// via webpackJsonp / self["webpackChunk..."] arrays. Identify the
// runtime first:

// Look for the chunk-loading global, name varies per app but the
// pattern is consistent:
Object.keys(window).filter((k) => k.startsWith('webpackChunk'))
// e.g. "webpackChunkmy_app" → window.webpackChunkmy_app

// Each chunk push is: [[chunkIds], {moduleId: moduleFn, ...}, runtimeFn]
// You can hook the push to capture EVERY module function as it loads,
// even ones loaded lazily via code-splitting AFTER initial page load:
;(function () {
  const chunkName = Object.keys(window).find((k) => k.startsWith('webpackChunk'))
  const original = window[chunkName].push.bind(window[chunkName])
  window.__capturedModules = {}
  window[chunkName].push = function (chunkData) {
    const [, modules] = chunkData
    Object.assign(window.__capturedModules, modules)
    return original(chunkData)
  }
})()
// After the app has run for a while (clicked through several routes
// to trigger lazy-loaded chunks), inspect window.__capturedModules —
// this captures modules a static download of the initial bundle alone
// would MISS entirely.

// For OFFLINE bundle analysis (you have the .js files on disk, not a
// live page), webcrack (npm) is the modern, actively maintained
// successor to the older webpack-unpack-style tools — it detects the
// bundler, splits modules back into separate files with import/export
// statements reconstructed, AND runs deobfuscation in the same pass:
//   npx webcrack bundle.js -o ./unpacked/
```

**Webpack Source Maps:**

```javascript
// If source maps available (.map files)
// They reveal original source code!

// Find source maps
$$('script').forEach((script) => {
  if (script.src && script.src.includes('.js')) {
    fetch(script.src + '.map')
      .then((r) => r.json())
      .then((map) => {
        console.log('Source map found:', script.src)
        console.log('Sources:', map.sources)
        console.log('Source content:', map.sourcesContent)
      })
      .catch(() => {})
  }
})

// Parse source map
const sourceMap = require('source-map')
const fs = require('fs')

const rawSourceMap = JSON.parse(fs.readFileSync('bundle.js.map', 'utf8'))

sourceMap.SourceMapConsumer.with(rawSourceMap, null, (consumer) => {
  // Get original position
  const original = consumer.originalPositionFor({
    line: 1,
    column: 50,
  })
  console.log('Original:', original)

  // Get all sources
  consumer.sources.forEach((source) => {
    console.log('Source:', source)
    const content = consumer.sourceContentFor(source)
    console.log('Content:', content)
  })
})
```

**Reconstructing a Full Source Tree from a Source Map**

```javascript
// When sourcesContent is embedded in the map (common in dev builds
// accidentally shipped to prod — a surprisingly frequent real-world
// finding), you can dump the ENTIRE original source tree, not just
// look up single positions:

const fs = require('fs')
const path = require('path')

const map = JSON.parse(fs.readFileSync('bundle.js.map', 'utf8'))

if (!map.sourcesContent) {
  console.log('No embedded source content — map only gives positions, not full files.')
} else {
  map.sources.forEach((source, i) => {
    const content = map.sourcesContent[i]
    if (!content) return
    // sources are often webpack:// URIs — sanitize into a real path
    const safePath = source
      .replace(/^webpack:\/\/\/?/, '')
      .replace(/^\.\//, '')
      .replace(/[?].*$/, '')
    const outPath = path.join('./recovered_source', safePath)
    fs.mkdirSync(path.dirname(outPath), { recursive: true })
    fs.writeFileSync(outPath, content)
  })
  console.log(`Recovered ${map.sources.length} source files -> ./recovered_source/`)
}

// SEARCH TIP: before doing any of this manually, just check if a
// source map exists and is reachable — this is a frequent, embarrassingly
// simple finding in bug bounty/competitive-analysis work:
//   curl -s https://target.com/static/js/main.abc123.js.map | head -c 200
// A 200 OK with real JSON here means the WHOLE original source tree,
// including comments and variable names, may be one curl away.
```

**Extracting React Component Tree:**

```javascript
// Get React Fiber tree
function getReactFiber(element) {
  const key = Object.keys(element).find(
    (key) => key.startsWith('__reactInternalInstance') || key.startsWith('__reactFiber'),
  )
  return element[key]
}

// Walk component tree
function walkReactTree(fiber, callback) {
  if (!fiber) return

  callback(fiber)

  if (fiber.child) walkReactTree(fiber.child, callback)
  if (fiber.sibling) walkReactTree(fiber.sibling, callback)
}

// Find all components
const rootElement = document.getElementById('root')
const fiber = getReactFiber(rootElement)

const components = []
walkReactTree(fiber, (node) => {
  if (node.type && typeof node.type === 'function') {
    components.push({
      name: node.type.name,
      props: node.memoizedProps,
      state: node.memoizedState,
    })
  }
})

console.log('Components:', components)

// Access React DevTools data
window.__REACT_DEVTOOLS_GLOBAL_HOOK__.renderers.forEach((renderer, id) => {
  console.log('React Renderer:', id)
})
```

---

## Phase 3: API Reverse Engineering

### Week 5: REST API Discovery

**Finding Hidden Endpoints:**

```javascript
// 1. Extract from JavaScript
const scripts = Array.from(document.scripts).map((s) => s.src)
const allCode = await Promise.all(scripts.map((src) => fetch(src).then((r) => r.text())))

const endpoints = new Set()
allCode.forEach((code) => {
  // Find API paths
  const paths = code.match(/['"`](\/api\/[^'"`\s]+)['"`]/g)
  if (paths) {
    paths.forEach((p) => endpoints.add(p.replace(/['"`]/g, '')))
  }
})

console.log('Found endpoints:', Array.from(endpoints))

// 2. Monitor network traffic
const originalFetch = window.fetch
window.fetch = function (...args) {
  console.log('Fetch:', args[0])
  return originalFetch.apply(this, args)
}

const originalXHR = window.XMLHttpRequest.prototype.open
window.XMLHttpRequest.prototype.open = function (method, url) {
  console.log('XHR:', method, url)
  return originalXHR.apply(this, arguments)
}

// 3. Brute force common endpoints
const commonPaths = [
  '/api/users',
  '/api/user',
  '/api/profile',
  '/api/settings',
  '/api/auth',
  '/api/login',
  '/api/logout',
  '/api/posts',
  '/api/comments',
  '/api/admin',
  '/api/dashboard',
  '/api/data',
  '/api/export',
  '/api/import',
]

for (const path of commonPaths) {
  fetch(path)
    .then((r) => {
      if (r.ok) console.log('Found:', path, r.status)
    })
    .catch(() => {})
}
```

**Faster Endpoint Brute-Forcing with a Real Wordlist (ffuf)**

```bash
# A hardcoded 14-item array finds almost nothing on a real target.
# ffuf with SecLists' real-world API wordlists is the practical
# upgrade — fast, threaded, and easy to filter/process:

sudo apt install -y ffuf
git clone https://github.com/danielmiessler/SecLists /opt/SecLists

ffuf -u "https://target.com/api/FUZZ" \
     -w /opt/SecLists/Discovery/Web-Content/api/api-endpoints.txt \
     -mc 200,201,204,301,302,401,403 \
     -t 40 -o api_results.json -of json

# Filter by response SIZE to weed out a generic "not found" page that
# returns 200 for everything (a common false-positive trap):
ffuf -u "https://target.com/api/FUZZ" \
     -w wordlist.txt -fs 1234   # -fs = filter by exact size to exclude

# Recursive discovery once you find a base path with sub-resources:
ffuf -u "https://target.com/api/users/FUZZ" \
     -w /opt/SecLists/Discovery/Web-Content/common.txt -mc 200
```

**API Parameter Discovery:**

```javascript
// Find parameters used in requests
const requests = performance
  .getEntries()
  .filter((e) => e.initiatorType === 'fetch' || e.initiatorType === 'xmlhttprequest')

requests.forEach((req) => {
  const url = new URL(req.name)
  console.log('URL:', url.pathname)
  console.log('Params:', Object.fromEntries(url.searchParams))
})

// Fuzz parameters
const baseUrl = 'https://api.example.com/users'
const commonParams = [
  'id',
  'userId',
  'user_id',
  'limit',
  'offset',
  'page',
  'sort',
  'order',
  'filter',
  'search',
  'query',
  'q',
  'include',
  'expand',
  'fields',
  'token',
  'apiKey',
  'api_key',
]

for (const param of commonParams) {
  fetch(`${baseUrl}?${param}=test`)
    .then((r) => r.json())
    .then((data) => {
      if (!data.error) {
        console.log(`Parameter ${param} works!`)
      }
    })
    .catch(() => {})
}
```

**Arjun — Dedicated Parameter Discovery Tool**

```bash
# Arjun automates exactly the parameter-fuzzing pattern above, but
# with a much larger curated wordlist and smarter diffing logic
# (it compares response behavior, not just status code, to detect
# when a parameter is actually being processed vs silently ignored):
pip3 install --break-system-packages arjun

arjun -u "https://target.com/api/users" -m GET
arjun -u "https://target.com/api/login" -m POST --headers "Content-Type: application/json"
arjun -u "https://target.com/api/search" -w custom_params.txt
```

**Authentication Analysis:**

```javascript
// 1. Extract tokens
const token =
  localStorage.getItem('token') || sessionStorage.getItem('token') || parseCookies().token

console.log('Token:', token)

// 2. Decode JWT
function parseJWT(token) {
  const base64Url = token.split('.')[1]
  const base64 = base64Url.replace(/-/g, '+').replace(/_/g, '/')
  const jsonPayload = decodeURIComponent(
    atob(base64)
      .split('')
      .map((c) => {
        return '%' + ('00' + c.charCodeAt(0).toString(16)).slice(-2)
      })
      .join(''),
  )
  return JSON.parse(jsonPayload)
}

if (token) {
  const decoded = parseJWT(token)
  console.log('JWT Payload:', decoded)
  console.log('Expires:', new Date(decoded.exp * 1000))
}

// 3. Test token in requests
fetch('https://api.example.com/protected', {
  headers: {
    Authorization: `Bearer ${token}`,
  },
})
  .then((r) => r.json())
  .then((data) => console.log('Protected data:', data))

// 4. Check for weak tokens
if (token && token.length < 20) {
  console.warn('Token might be weak!')
}

// 5. Monitor auth flow
const authFlow = []
const originalFetch = window.fetch
window.fetch = function (url, options) {
  if (url.includes('login') || url.includes('auth')) {
    authFlow.push({
      url,
      method: options?.method || 'GET',
      headers: options?.headers,
      body: options?.body,
    })
  }
  return originalFetch.apply(this, arguments)
}
```

**Testing JWT Implementation Weaknesses (Authorized Scope Only)**

```bash
# Once you've extracted a token, dedicated JWT tooling tests
# implementation-level flaws far more thoroughly than manual checks:

pip3 install --break-system-packages jwt-tool 2>/dev/null || \
  git clone https://github.com/ticarpi/jwt_tool

python3 jwt_tool.py <token>                    # parse + basic checks
python3 jwt_tool.py <token> -X a                # test alg:none attack
python3 jwt_tool.py <token> -C -d /path/wordlist.txt  # crack HS256
                                                          # secret offline
python3 jwt_tool.py <token> -X k -pk public.pem -S hs256
                                                  # test RS256->HS256
                                                  # algorithm confusion,
                                                  # signing with the
                                                  # public key as an
                                                  # HMAC secret
```

### Week 6: GraphQL & WebSocket Analysis

**GraphQL Introspection:**

```javascript
// GraphQL introspection query
const introspectionQuery = `
query IntrospectionQuery {
  __schema {
    queryType { name }
    mutationType { name }
    subscriptionType { name }
    types {
      ...FullType
    }
  }
}

fragment FullType on __Type {
  kind
  name
  description
  fields(includeDeprecated: true) {
    name
    description
    args {
      ...InputValue
    }
    type {
      ...TypeRef
    }
  }
}

fragment InputValue on __InputValue {
  name
  description
  type { ...TypeRef }
  defaultValue
}

fragment TypeRef on __Type {
  kind
  name
  ofType {
    kind
    name
    ofType {
      kind
      name
    }
  }
}
`

// Execute introspection
fetch('https://api.example.com/graphql', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({ query: introspectionQuery }),
})
  .then((r) => r.json())
  .then((schema) => {
    console.log('GraphQL Schema:', schema)

    // Extract all queries
    const queries = schema.data.__schema.types.find(
      (t) => t.name === schema.data.__schema.queryType.name,
    ).fields

    console.log(
      'Available Queries:',
      queries.map((q) => q.name),
    )

    // Extract all mutations
    const mutations = schema.data.__schema.types.find(
      (t) => t.name === schema.data.__schema.mutationType.name,
    ).fields

    console.log(
      'Available Mutations:',
      mutations.map((m) => m.name),
    )
  })

// Test discovered queries
queries.forEach((query) => {
  const testQuery = `query { ${query.name} { id } }`
  fetch('https://api.example.com/graphql', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ query: testQuery }),
  })
    .then((r) => r.json())
    .then((result) => {
      if (!result.errors) {
        console.log(`Query ${query.name} works!`, result)
      }
    })
})
```

**When Introspection Is Disabled — GraphQL Schema Reconstruction**

```bash
# Production APIs increasingly disable introspection (it's an obvious
# information-disclosure risk). Two practical fallbacks:

# 1. clairvoyance — brute-forces field/type names against a real
#    wordlist, using GraphQL's own error messages ("did you mean X?")
#    to incrementally reconstruct the schema even with introspection off:
pip3 install --break-system-packages clairvoyance
clairvoyance -o recovered_schema.graphql https://target.com/graphql

# 2. graphql-cop — checks for common GraphQL-specific misconfigurations
#    beyond just introspection (batching abuse, field suggestion
#    leakage, CSRF via GET-based queries):
pip3 install --break-system-packages graphql-cop
graphql-cop -t https://target.com/graphql

# 3. InQL (Burp Suite extension) — once you DO have a schema (from
#    introspection or clairvoyance), InQL auto-generates a query
#    library covering every discovered query/mutation, ready to send
#    through Repeater for testing each one's authorization individually.
```

**GraphQL-Specific Vulnerability Classes**

```javascript
// BATCHING / ALIAS-BASED DOS & RATE-LIMIT BYPASS
// GraphQL lets you send many "queries" in a single HTTP request via
// aliases — this can bypass naive per-request rate limiting entirely:
const batchedAttack = `
query {
  a1: login(username: "admin", password: "guess1") { token }
  a2: login(username: "admin", password: "guess2") { token }
  a3: login(username: "admin", password: "guess3") { token }
  # ... hundreds more in ONE request, ONE rate-limit hit
}`

// DEEP/NESTED QUERY DOS — exploiting unrestricted relational nesting:
const nestedDOS = `
query {
  user(id: 1) {
    friends { friends { friends { friends { friends { id } } } } }
  }
}`
// A naive resolver fans this out exponentially — properly configured
// APIs should enforce query depth/complexity limits server-side.

// FIELD-LEVEL AUTHORIZATION GAPS — the most common real-world GraphQL
// finding: an object-level check exists on the TOP query, but a
// nested field on the SAME type skips the check:
const authzBypass = `
query {
  publicProduct(id: 5) {
    name
    owner { email, ssn, internalNotes }  # nested field forgot to
  }                                       # re-check authorization
}`
```

**WebSocket Analysis:**

```javascript
// Intercept WebSocket connections
const originalWebSocket = window.WebSocket
window.WebSocket = function (url, protocols) {
  console.log('WebSocket created:', url)

  const ws = new originalWebSocket(url, protocols)

  // Log sent messages
  const originalSend = ws.send
  ws.send = function (data) {
    console.log('WS Send:', data)
    return originalSend.call(this, data)
  }

  // Log received messages
  ws.addEventListener('message', (event) => {
    console.log('WS Receive:', event.data)
  })

  ws.addEventListener('open', () => {
    console.log('WS Connected')
  })

  ws.addEventListener('close', () => {
    console.log('WS Disconnected')
  })

  return ws
}

// Manually connect to WebSocket
const ws = new WebSocket('wss://api.example.com/socket')

ws.onopen = () => {
  // Try different message formats
  ws.send(JSON.stringify({ type: 'subscribe', channel: 'updates' }))
  ws.send(JSON.stringify({ action: 'ping' }))
  ws.send('{"event":"join","data":{"room":"general"}}')
}

ws.onmessage = (event) => {
  console.log('Received:', event.data)
  try {
    const data = JSON.parse(event.data)
    console.log('Parsed:', data)
  } catch (e) {
    console.log('Not JSON:', event.data)
  }
}

// Replay recorded messages
const recordedMessages = ['{"type":"auth","token":"..."}', '{"type":"getData","id":123}']

recordedMessages.forEach((msg, i) => {
  setTimeout(() => {
    ws.send(msg)
  }, i * 1000)
})
```

**Inspecting WebSocket Traffic Outside the Browser (Burp / mitmproxy)**

```
Chrome DevTools' WS tab is fine for casual inspection but lacks replay/
fuzzing. For serious WebSocket analysis:

BURP SUITE: the Proxy > WebSockets history tab (Burp 2020.9+) captures
  every frame bidirectionally and lets you send individual frames to
  Repeater for manual replay/modification — by far the most practical
  GUI workflow for WS testing.

mitmproxy: also captures WebSocket frames (shown inline in the flow
  view) and lets you write a Python addon to programmatically modify
  frames in transit:
    def websocket_message(flow):
        msg = flow.websocket.messages[-1]
        print(f"{'<-' if msg.from_client else '->'} {msg.content}")
```

---

## Phase 4: Framework Analysis

### Week 7: React Application Analysis

**React Component Extraction:**

```javascript
// Find all React components in bundle
const componentRegex =
  /function\s+([A-Z][a-zA-Z0-9]*)\s*\([^)]*\)\s*{[\s\S]*?return\s+React\.createElement/g
const bundleCode = Array.from(document.scripts)
  .map((s) => s.textContent)
  .join('\n')

const components = []
let match
while ((match = componentRegex.exec(bundleCode)) !== null) {
  components.push(match[1])
}

console.log('React Components:', components)

// Access component props and state
function getComponentData(element) {
  const fiber = getReactFiber(element)
  if (!fiber) return null

  return {
    type: fiber.type?.name || fiber.type,
    props: fiber.memoizedProps,
    state: fiber.memoizedState,
    key: fiber.key,
    ref: fiber.ref,
  }
}

// Get all component instances
const allComponents = []
document.querySelectorAll('*').forEach((el) => {
  const data = getComponentData(el)
  if (data) allComponents.push(data)
})

console.log('Component Instances:', allComponents)

// Hook into component lifecycle
const originalCreateElement = React.createElement
React.createElement = function (type, props, ...children) {
  console.log('Creating element:', type, props)
  return originalCreateElement.apply(this, arguments)
}
```

**Using React DevTools Programmatically (More Reliable Than Manual Fiber Walking)**

```javascript
// Manually walking __reactFiber$ keys is fragile — the exact key
// suffix is a random hash that changes between React versions/builds.
// The React DevTools global hook exposes a stable, version-agnostic
// API for exactly this purpose, which is what the actual DevTools
// extension itself uses internally:

const hook = window.__REACT_DEVTOOLS_GLOBAL_HOOK__
if (hook) {
  hook.renderers.forEach((renderer, id) => {
    console.log(`Renderer ${id}:`, renderer.version)
    // Walk the fiber roots this renderer is tracking:
    renderer.findFiberByHostInstance // can resolve a DOM node -> fiber
  })

  // Hook into every commit (re-render) as it happens — extremely
  // useful for understanding WHEN and WHY a component re-renders,
  // which is both a debugging technique and a way to reverse-engineer
  // an app's actual state-update triggers without reading minified
  // source at all:
  const originalOnCommitFiberRoot = hook.onCommitFiberRoot
  hook.onCommitFiberRoot = function (id, root, ...args) {
    console.log('React commit:', root.current)
    return originalOnCommitFiberRoot?.apply(this, [id, root, ...args])
  }
}

// Finding a fiber from a DOM node reliably (works across React
// versions without guessing the exact internal key name):
function findReactFiber(domNode) {
  const key = Object.keys(domNode).find(
    (k) => k.startsWith('__reactFiber$') || k.startsWith('__reactInternalInstance$'),
  )
  return key ? domNode[key] : null
}

// Extracting Redux/Context state if exposed via DevTools hooks:
if (window.__REDUX_DEVTOOLS_EXTENSION__) {
  console.log(
    'Redux DevTools detected — store state is inspectable via the extension UI/its own message-passing API.',
  )
}
```

### Week 8: Vue & Angular Analysis

**Vue.js Reverse Engineering**

```javascript
// VUE 2 — instance is attached directly as __vue__
const vueInstance = document.querySelector('#app').__vue__
console.log('Data:', vueInstance.$data)
console.log('Props:', vueInstance.$props)
console.log('Computed:', vueInstance.$options.computed)
console.log('Methods:', Object.keys(vueInstance.$options.methods || {}))

// Walk the FULL Vue 2 component tree from the root:
function walkVue2Tree(instance, callback, depth = 0) {
  callback(instance, depth)
  if (instance.$children) {
    instance.$children.forEach((child) => walkVue2Tree(child, callback, depth + 1))
  }
}
walkVue2Tree(vueInstance, (inst, depth) => {
  console.log(
    '  '.repeat(depth) +
      (inst.$options.name || inst.$options._componentTag || 'AnonymousComponent'),
  )
})

// VUE 3 — the composition API changes the introspection approach;
// instances are exposed via __vueParentComponent / __vnode on the DOM
// node, and the public instance is reached via .proxy:
function getVue3Instance(el) {
  const key = Object.keys(el).find((k) => k.startsWith('__vueParentComponent'))
  return key ? el[key] : null
}
const root = document.querySelector('#app')
const vue3Instance = getVue3Instance(root)
console.log('Vue 3 setup state:', vue3Instance?.setupState)
console.log('Vue 3 props:', vue3Instance?.props)

// Vuex store extraction (Vue 2's standard state management):
if (vueInstance.$store) {
  console.log('Vuex state:', vueInstance.$store.state)
  console.log('Vuex getters:', vueInstance.$store.getters)
  // List registered mutations/actions (useful to find hidden/
  // undocumented state-mutating actions):
  console.log('Mutations:', Object.keys(vueInstance.$store._mutations))
  console.log('Actions:', Object.keys(vueInstance.$store._actions))
}

// Pinia store extraction (Vue 3's modern standard, replacing Vuex):
// Pinia stores register themselves on a global devtools plugin API
// when devtools mode is active:
if (window.__PINIA_DEVTOOLS_HOOK__) {
  console.log('Pinia stores detected — inspect via Vue DevTools panel.')
}
```

**Angular Reverse Engineering**

```javascript
// Angular (2+) exposes a global debugging API when NOT in production
// mode (ng.* helpers are stripped/disabled in a properly built prod
// bundle — their PRESENCE is itself a finding: it means the app was
// shipped with devMode enabled, an info-disclosure misconfiguration):

// Get the component instance attached to a DOM element:
const el = document.querySelector('app-root')
const component = ng.getComponent(el)
console.log('Component:', component)

// Get the full dependency-injected context for an element:
const context = ng.getContext(el)
console.log('Context:', context)

// List all directives applied to an element:
console.log('Directives:', ng.getDirectives(el))

// Get a component's injector to resolve its dependencies (e.g. pull
// out an injected HttpClient or a service holding auth state):
const injector = ng.getInjector(el)
// (specific service retrieval requires knowing the service's class
// reference, typically found by reading the bundle's chunk for the
// module defining it)

// Profile change detection — useful to understand what triggers a
// given component to re-render, similarly to the React commit hook above:
ng.profiler.timeChangeDetection({ record: true })

// Find ALL Angular components currently mounted on the page:
function findAllNgComponents(root = document.body) {
  const found = []
  function walk(el) {
    try {
      const comp = ng.getComponent(el)
      if (comp) found.push({ el, comp, name: comp.constructor.name })
    } catch (e) {}
    el.childNodes.forEach((child) => {
      if (child.nodeType === 1) walk(child)
    })
  }
  walk(root)
  return found
}
console.log(findAllNgComponents())

// Angular's Zone.js wraps async APIs (setTimeout, addEventListener,
// Promises) for change detection — this means you can also hook
// Zone.js itself to log EVERY async operation the app schedules,
// which is a powerful way to trace hidden polling/background
// behavior without reading the source:
if (window.Zone) {
  console.log('Zone.js detected — app uses zone-based change detection (standard Angular).')
}
```

**Framework Fingerprinting — Identify What You're Dealing With First**

```javascript
// Before diving into framework-specific techniques, quickly identify
// the stack from the DOM/global scope alone:

function fingerprintFramework() {
  const findings = []
  if (window.React || document.querySelector('[data-reactroot], #__next'))
    findings.push('React' + (window.next ? ' (Next.js)' : ''))
  if (window.Vue || document.querySelector('[data-v-app]')) findings.push('Vue')
  if (window.ng || document.querySelector('[ng-version]'))
    findings.push(
      `Angular ${document.querySelector('[ng-version]')?.getAttribute('ng-version') || ''}`,
    )
  if (window.__SVELTE__ || document.querySelector('[class*="svelte-"]')) findings.push('Svelte')
  if (document.querySelector('script[src*="nuxt"]') || window.__NUXT__) findings.push('Nuxt.js')
  if (window.__remixContext) findings.push('Remix')
  return findings.length ? findings : ['Unknown / vanilla JS / SSR-only']
}
console.log('Detected framework(s):', fingerprintFramework())

// For a more authoritative, regularly-updated fingerprint database,
// Wappalyzer's underlying technology database (also usable as a CLI/
// library, not just the browser extension) covers far more than
// frameworks — CDNs, analytics, payment processors, server tech:
//   npm install -g wappalyzer
//   wappalyzer https://target.com
```

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
