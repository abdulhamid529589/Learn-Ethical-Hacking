# Complete Web Application Reverse Engineering Guide
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
const emailRegex = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/;
const phoneRegex = /\+?1?\d{9,15}/;

// 2. Advanced Browser APIs
console.log(window.performance.getEntries());
console.log(navigator.userAgent);
console.log(document.cookie);

// 3. Network Protocols
// HTTP/HTTPS, WebSocket, Server-Sent Events (SSE)
// TCP/IP basics, DNS resolution

// 4. Cryptography Basics
const crypto = require('crypto');
const hash = crypto.createHash('sha256').update('data').digest('hex');

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
$$('[data-testid]').forEach(el => {
    console.log(el.getAttribute('data-testid'), el);
});

// 2. Find React/Vue component instance
$0.__reactFiber$  // React component (select element first)
$0.__vue__        // Vue component

// 3. Monitor element changes
const observer = new MutationObserver(mutations => {
    mutations.forEach(mutation => {
        console.log('Changed:', mutation);
    });
});
observer.observe(document.body, {
    childList: true,
    subtree: true,
    attributes: true
});

// 4. Find hidden elements
$$('*').filter(el => {
    const style = window.getComputedStyle(el);
    return style.display === 'none' || style.visibility === 'hidden';
});

// 5. Extract all links
Array.from(document.links).map(a => ({
    text: a.textContent,
    href: a.href
}));

// 6. Find all forms and inputs
$$('form').forEach(form => {
    console.log('Form:', form.action);
    $$('input, textarea', form).forEach(input => {
        console.log('  Input:', input.name, input.type);
    });
});
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
const fs = require('fs');
const har = JSON.parse(fs.readFileSync('network.har', 'utf8'));

har.log.entries.forEach(entry => {
    const request = entry.request;
    const response = entry.response;
    
    console.log(`${request.method} ${request.url}`);
    console.log(`Status: ${response.status}`);
    console.log(`Size: ${response.content.size} bytes`);
    console.log('---');
});
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
Object.keys(localStorage).forEach(key => {
    console.log(key, localStorage.getItem(key));
});

// Extract all localStorage
const allStorage = {};
for (let i = 0; i < localStorage.length; i++) {
    const key = localStorage.key(i);
    allStorage[key] = localStorage.getItem(key);
}
console.log(JSON.stringify(allStorage, null, 2));

// 2. Session Storage
Object.keys(sessionStorage).forEach(key => {
    console.log(key, sessionStorage.getItem(key));
});

// 3. Cookies
document.cookie.split(';').forEach(cookie => {
    console.log(cookie.trim());
});

// Parse cookies
function parseCookies() {
    return document.cookie.split(';').reduce((cookies, cookie) => {
        const [name, value] = cookie.split('=').map(c => c.trim());
        cookies[name] = decodeURIComponent(value);
        return cookies;
    }, {});
}

// 4. IndexedDB
// Application → IndexedDB → Inspect databases

// Access IndexedDB programmatically
const request = indexedDB.open('myDatabase');
request.onsuccess = (event) => {
    const db = event.target.result;
    const transaction = db.transaction(['myStore'], 'readonly');
    const store = transaction.objectStore('myStore');
    const getRequest = store.getAll();
    getRequest.onsuccess = () => {
        console.log('All data:', getRequest.result);
    };
};

// 5. Cache Storage
caches.keys().then(names => {
    names.forEach(name => {
        caches.open(name).then(cache => {
            cache.keys().then(requests => {
                console.log(`Cache: ${name}`);
                requests.forEach(req => {
                    console.log('  ', req.url);
                });
            });
        });
    });
});

// 6. Service Workers
navigator.serviceWorker.getRegistrations().then(registrations => {
    registrations.forEach(registration => {
        console.log('SW:', registration.active.scriptURL);
    });
});
```

**Console Tab - Advanced Techniques:**
```javascript
// 1. Monitor function calls
monitor(functionName);  // Logs when function is called
unmonitor(functionName);

// 2. Monitor events
monitorEvents(window, 'click');  // Log all click events
monitorEvents(document.body);    // Log all events
unmonitorEvents(window);

// 3. Get event listeners
getEventListeners(document.body);

// 4. Copy to clipboard
copy(complexObject);  // Copies JSON to clipboard

// 5. Console API
console.time('operation');
// ... code ...
console.timeEnd('operation');

console.table([
    {name: 'John', age: 30},
    {name: 'Jane', age: 25}
]);

console.trace();  // Print stack trace

// 6. Command Line API
$0       // Currently selected element
$_       // Previously evaluated expression
$$()     // querySelectorAll
$x()     // XPath query
keys()   // Object.keys
values() // Object.values
dir()    // console.dir

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

---

## Phase 2: JavaScript Reverse Engineering

### Week 3: Deobfuscation & Analysis

**Understanding Minified Code:**
```javascript
// Original code
function calculateTotal(items) {
    return items.reduce((sum, item) => sum + item.price, 0);
}

// Minified
function a(b){return b.reduce((c,d)=>c+d.e,0)}

// Steps to understand:
// 1. Use DevTools pretty-print ({} button)
// 2. Rename variables meaningfully
// 3. Add breakpoints to trace execution
// 4. Watch expressions to see values
```

**Common Obfuscation Techniques:**
```javascript
// 1. String Array Obfuscation
const _0x1234 = ['hello', 'world', 'test'];
const _0x5678 = function(index) { return _0x1234[index]; };
console.log(_0x5678(0)); // 'hello'

// Deobfuscate: Replace calls with actual strings

// 2. Control Flow Flattening
const state = 0;
while (true) {
    switch (state) {
        case 0: console.log('Step 1'); state = 1; break;
        case 1: console.log('Step 2'); state = 2; break;
        case 2: return;
    }
}

// Deobfuscate: Follow control flow manually

// 3. Dead Code Injection
if (false) {
    // This never executes but adds confusion
    console.log('junk');
}
const x = true ? 'value' : 'never_used';

// 4. Proxy Objects
const obj = new Proxy({}, {
    get: function(target, prop) {
        return target[prop];
    }
});

// 5. Encoding
const encoded = 'aGVsbG8='; // Base64
const decoded = atob(encoded); // 'hello'

// Hex encoding
const hex = '68656c6c6f';
const text = Buffer.from(hex, 'hex').toString(); // 'hello'
```

**Automated Deobfuscation:**
```javascript
// Using de4js (https://lelinhtinh.github.io/de4js/)
// Or use Node.js tools

const { parse } = require('@babel/parser');
const traverse = require('@babel/traverse').default;
const generate = require('@babel/generator').default;
const t = require('@babel/types');

const code = `function _0x1234(){return"hello";}`;

// Parse code
const ast = parse(code);

// Transform AST
traverse(ast, {
    FunctionDeclaration(path) {
        // Rename function
        if (path.node.id.name.startsWith('_0x')) {
            path.node.id.name = 'deobfuscated_function';
        }
    }
});

// Generate code
const output = generate(ast, {}, code);
console.log(output.code);
```

**JavaScript AST Analysis:**
```javascript
// Abstract Syntax Tree manipulation

const esprima = require('esprima');
const escodegen = require('escodegen');

const code = `
function secret() {
    const key = "hidden";
    return key;
}
`;

// Parse to AST
const ast = esprima.parseScript(code);

// Explore AST
function walk(node, callback) {
    callback(node);
    for (let key in node) {
        if (node[key] && typeof node[key] === 'object') {
            walk(node[key], callback);
        }
    }
}

walk(ast, (node) => {
    if (node.type === 'Literal' && typeof node.value === 'string') {
        console.log('Found string:', node.value);
    }
});

// Modify AST
ast.body[0].id.name = 'revealed';

// Generate code
const modified = escodegen.generate(ast);
console.log(modified);
```

**Extracting Embedded Data:**
```javascript
// Find all strings in JavaScript
const jsCode = document.scripts[0].textContent;
const strings = jsCode.match(/"([^"\\]|\\.)*"|'([^'\\]|\\.)*'/g);
console.log(strings);

// Find all URLs
const urls = jsCode.match(/https?:\/\/[^\s"']+/g);
console.log(urls);

// Find API endpoints
const endpoints = jsCode.match(/\/api\/[^\s"']+/g);
console.log(endpoints);

// Find keys/secrets (careful!)
const possibleKeys = jsCode.match(/[a-zA-Z0-9]{32,}/g);
console.log(possibleKeys);

// Extract configuration objects
const configMatch = jsCode.match(/config\s*=\s*({[\s\S]*?});/);
if (configMatch) {
    const config = eval('(' + configMatch[1] + ')');
    console.log(config);
}
```

### Week 4: Webpack/Bundler Analysis

**Analyzing Webpack Bundles:**
```javascript
// Webpack bundle structure
(function(modules) {
    var installedModules = {};
    
    function __webpack_require__(moduleId) {
        // Module caching
        if (installedModules[moduleId]) {
            return installedModules[moduleId].exports;
        }
        // Create new module
        var module = installedModules[moduleId] = {
            i: moduleId,
            l: false,
            exports: {}
        };
        // Execute module
        modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
        module.l = true;
        return module.exports;
    }
    
    // Entry point
    return __webpack_require__(0);
})([
    // Module 0
    function(module, exports, __webpack_require__) {
        // Your code here
    },
    // Module 1
    function(module, exports) {
        // More code
    }
]);

// Extract all modules
const modules = [];
const bundleCode = document.scripts[0].textContent;
const moduleRegex = /\/\*\*\*\/ \(function\(module, exports.*?\}\),?/gs;
const matches = bundleCode.matchAll(moduleRegex);

for (const match of matches) {
    modules.push(match[0]);
}

console.log(`Found ${modules.length} modules`);
```

**Webpack Source Maps:**
```javascript
// If source maps available (.map files)
// They reveal original source code!

// Find source maps
$$('script').forEach(script => {
    if (script.src && script.src.includes('.js')) {
        fetch(script.src + '.map')
            .then(r => r.json())
            .then(map => {
                console.log('Source map found:', script.src);
                console.log('Sources:', map.sources);
                console.log('Source content:', map.sourcesContent);
            })
            .catch(() => {});
    }
});

// Parse source map
const sourceMap = require('source-map');
const fs = require('fs');

const rawSourceMap = JSON.parse(fs.readFileSync('bundle.js.map', 'utf8'));

sourceMap.SourceMapConsumer.with(rawSourceMap, null, consumer => {
    // Get original position
    const original = consumer.originalPositionFor({
        line: 1,
        column: 50
    });
    console.log('Original:', original);
    
    // Get all sources
    consumer.sources.forEach(source => {
        console.log('Source:', source);
        const content = consumer.sourceContentFor(source);
        console.log('Content:', content);
    });
});
```

**Extracting React Component Tree:**
```javascript
// Get React Fiber tree
function getReactFiber(element) {
    const key = Object.keys(element).find(key => 
        key.startsWith('__reactInternalInstance') || 
        key.startsWith('__reactFiber')
    );
    return element[key];
}

// Walk component tree
function walkReactTree(fiber, callback) {
    if (!fiber) return;
    
    callback(fiber);
    
    if (fiber.child) walkReactTree(fiber.child, callback);
    if (fiber.sibling) walkReactTree(fiber.sibling, callback);
}

// Find all components
const rootElement = document.getElementById('root');
const fiber = getReactFiber(rootElement);

const components = [];
walkReactTree(fiber, (node) => {
    if (node.type && typeof node.type === 'function') {
        components.push({
            name: node.type.name,
            props: node.memoizedProps,
            state: node.memoizedState
        });
    }
});

console.log('Components:', components);

// Access React DevTools data
window.__REACT_DEVTOOLS_GLOBAL_HOOK__.renderers.forEach((renderer, id) => {
    console.log('React Renderer:', id);
});
```

---

## Phase 3: API Reverse Engineering

### Week 5: REST API Discovery

**Finding Hidden Endpoints:**
```javascript
// 1. Extract from JavaScript
const scripts = Array.from(document.scripts).map(s => s.src);
const allCode = await Promise.all(
    scripts.map(src => fetch(src).then(r => r.text()))
);

const endpoints = new Set();
allCode.forEach(code => {
    // Find API paths
    const paths = code.match(/['"`](\/api\/[^'"`\s]+)['"`]/g);
    if (paths) {
        paths.forEach(p => endpoints.add(p.replace(/['"`]/g, '')));
    }
});

console.log('Found endpoints:', Array.from(endpoints));

// 2. Monitor network traffic
const originalFetch = window.fetch;
window.fetch = function(...args) {
    console.log('Fetch:', args[0]);
    return originalFetch.apply(this, args);
};

const originalXHR = window.XMLHttpRequest.prototype.open;
window.XMLHttpRequest.prototype.open = function(method, url) {
    console.log('XHR:', method, url);
    return originalXHR.apply(this, arguments);
};

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
    '/api/import'
];

for (const path of commonPaths) {
    fetch(path)
        .then(r => {
            if (r.ok) console.log('Found:', path, r.status);
        })
        .catch(() => {});
}
```

**API Parameter Discovery:**
```javascript
// Find parameters used in requests
const requests = performance.getEntries()
    .filter(e => e.initiatorType === 'fetch' || e.initiatorType === 'xmlhttprequest');

requests.forEach(req => {
    const url = new URL(req.name);
    console.log('URL:', url.pathname);
    console.log('Params:', Object.fromEntries(url.searchParams));
});

// Fuzz parameters
const baseUrl = 'https://api.example.com/users';
const commonParams = [
    'id', 'userId', 'user_id',
    'limit', 'offset', 'page',
    'sort', 'order', 'filter',
    'search', 'query', 'q',
    'include', 'expand', 'fields',
    'token', 'apiKey', 'api_key'
];

for (const param of commonParams) {
    fetch(`${baseUrl}?${param}=test`)
        .then(r => r.json())
        .then(data => {
            if (!data.error) {
                console.log(`Parameter ${param} works!`);
            }
        })
        .catch(() => {});
}
```

**Authentication Analysis:**
```javascript
// 1. Extract tokens
const token = localStorage.getItem('token') || 
              sessionStorage.getItem('token') ||
              parseCookies().token;

console.log('Token:', token);

// 2. Decode JWT
function parseJWT(token) {
    const base64Url = token.split('.')[1];
    const base64 = base64Url.replace(/-/g, '+').replace(/_/g, '/');
    const jsonPayload = decodeURIComponent(atob(base64).split('').map(c => {
        return '%' + ('00' + c.charCodeAt(0).toString(16)).slice(-2);
    }).join(''));
    return JSON.parse(jsonPayload);
}

if (token) {
    const decoded = parseJWT(token);
    console.log('JWT Payload:', decoded);
    console.log('Expires:', new Date(decoded.exp * 1000));
}

// 3. Test token in requests
fetch('https://api.example.com/protected', {
    headers: {
        'Authorization': `Bearer ${token}`
    }
})
.then(r => r.json())
.then(data => console.log('Protected data:', data));

// 4. Check for weak tokens
if (token && token.length < 20) {
    console.warn('Token might be weak!');
}

// 5. Monitor auth flow
const authFlow = [];
const originalFetch = window.fetch;
window.fetch = function(url, options) {
    if (url.includes('login') || url.includes('auth')) {
        authFlow.push({
            url,
            method: options?.method || 'GET',
            headers: options?.headers,
            body: options?.body
        });
    }
    return originalFetch.apply(this, arguments);
};
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
`;

// Execute introspection
fetch('https://api.example.com/graphql', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json'
    },
    body: JSON.stringify({ query: introspectionQuery })
})
.then(r => r.json())
.then(schema => {
    console.log('GraphQL Schema:', schema);
    
    // Extract all queries
    const queries = schema.data.__schema.types
        .find(t => t.name === schema.data.__schema.queryType.name)
        .fields;
    
    console.log('Available Queries:', queries.map(q => q.name));
    
    // Extract all mutations
    const mutations = schema.data.__schema.types
        .find(t => t.name === schema.data.__schema.mutationType.name)
        .fields;
    
    console.log('Available Mutations:', mutations.map(m => m.name));
});

// Test discovered queries
queries.forEach(query => {
    const testQuery = `query { ${query.name} { id } }`;
    fetch('https://api.example.com/graphql', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ query: testQuery })
    })
    .then(r => r.json())
    .then(result => {
        if (!result.errors) {
            console.log(`Query ${query.name} works!`, result);
        }
    });
});
```

**WebSocket Analysis:**
```javascript
// Intercept WebSocket connections
const originalWebSocket = window.WebSocket;
window.WebSocket = function(url, protocols) {
    console.log('WebSocket created:', url);
    
    const ws = new originalWebSocket(url, protocols);
    
    // Log sent messages
    const originalSend = ws.send;
    ws.send = function(data) {
        console.log('WS Send:', data);
        return originalSend.call(this, data);
    };
    
    // Log received messages
    ws.addEventListener('message', (event) => {
        console.log('WS Receive:', event.data);
    });
    
    ws.addEventListener('open', () => {
        console.log('WS Connected');
    });
    
    ws.addEventListener('close', () => {
        console.log('WS Disconnected');
    });
    
    return ws;
};

// Manually connect to WebSocket
const ws = new WebSocket('wss://api.example.com/socket');

ws.onopen = () => {
    // Try different message formats
    ws.send(JSON.stringify({ type: 'subscribe', channel: 'updates' }));
    ws.send(JSON.stringify({ action: 'ping' }));
    ws.send('{"event":"join","data":{"room":"general"}}');
};

ws.onmessage = (event) => {
    console.log('Received:', event.data);
    try {
        const data = JSON.parse(event.data);
        console.log('Parsed:', data);
    } catch (e) {
        console.log('Not JSON:', event.data);
    }
};

// Replay recorded messages
const recordedMessages = [
    '{"type":"auth","token":"..."}',
    '{"type":"getData","id":123}'
];

recordedMessages.forEach((msg, i) => {
    setTimeout(() => {
        ws.send(msg);
    }, i * 1000);
});
```

---

## Phase 4: Framework Analysis

### Week 7: React Application Analysis

**React Component Extraction:**
```javascript
// Find all React components in bundle
const componentRegex = /function\s+([A-Z][a-zA-Z0-9]*)\s*\([^)]*\)\s*{[\s\S]*?return\s+React\.createElement/g;
const bundleCode = Array.from(document.scripts)
    .map(s => s.textContent)
    .join('\n');

const components = [];
let match;
while ((match = componentRegex.exec(bundleCode)) !== null) {
    components.push(match[1]);
}

console.log('React Components:', components);

// Access component props and state
function getComponentData(element) {
    const fiber = getReactFiber(element);
    if (!fiber) return null;
    
    return {
        type: fiber.type?.name || fiber.type,
        props: fiber.memoizedProps,
        state: fiber.memoizedState,
        key: fiber.key,
        ref: fiber.ref
    };
}

// Get all component instances
const allComponents = [];
document.querySelectorAll('*').forEach(el => {
    const data = getComponentData(el);
    if (data) allComponents.push(data);
});

console.log('Component Instances:', allComponents);

// Hook into component lifecycle
const originalCreateElement = React.createElement;
React.createElement = function(type, props, ...children) {
    console.log('Creating