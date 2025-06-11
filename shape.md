# 🛡️ Shape Security Fingerprint – Deep Analysis Report

This document provides a **complete breakdown** of everything Shape Security is checking from what we can see in this JavaScript reverse-engineered environment. It includes browser fingerprinting, device signals, behavioral telemetry, timing analysis, runtime integrity checks, and spoof detection methods.

---

## 📌 What Is Shape Security Doing?

Shape injects heavily obfuscated JavaScript on protected pages (e.g., Target, Starbucks). This JavaScript:

1. Gathers device and environment fingerprint data.
2. Tracks real-time behavioral input from users (mouse, keyboard, orientation).
3. Simulates and manipulates events using `initCustomEvent`.
4. Measures timing, frame rates, and latency.
5. Sends the `signals` and `positionalSignals` to a backend API.
6. Receives a validation token (like a custom header) used in follow-up requests.

The full spectrum of checks is grouped and detailed below.

---

## 🔍 Full Category Breakdown

### 1. 🧭 Browser Fingerprint

Shape builds a high-entropy browser fingerprint using:

| Field | Description |
|-------|-------------|
| `navigator.userAgent` | Browser identity string |
| `navigator.platform` | OS and architecture |
| `navigator.appVersion`, `vendor`, `appName` | Browser and vendor consistency |
| `navigator.plugins`, `mimeTypes` | Plugin presence and MIME associations |
| `navigator.hardwareConcurrency` | Number of logical cores |
| `navigator.maxTouchPoints` | Touch support |
| `navigator.webdriver` | Selenium detection flag |
| `navigator.mediaDevices.enumerateDevices()` | Audio input devices |
| `navigator.productSub`, `doNotTrack`, `oscpu` | Secondary fingerprinting details |
| `cookies.length`, `cookie age` | Age and structure of existing cookies |
| `localStorage` and `sessionStorage` | Key presence, age, value size |
| `indexedDB` existence | Fake profiles often lack indexedDB or extensions don't populate it |


---

### 2. 🖼️ Graphics Fingerprint

Graphics card and rendering path detection:

| Field | Description |
|-------|-------------|
| `canvas.toDataURL()` | Rendered pixel hash for canvas fingerprint |
| `getContext('webgl')` | WebGL context probing |
| `webgl.getParameter()` | Queries vendor, version, shader precision, etc. |
| `shaderPrecisions` | High/medium/low float precision |
| `getExtension()` | Supported GL extensions list |
| `getContextAttributes()` | Antialias, alpha, depth, etc. |
| `getSupportedExtensions()` | Full feature list from GPU |
| GPU Labels | e.g. `ANGLE (NVIDIA GeForce GTX ...)` or `Intel HD ...` |

---

### 3. 🖱️ Behavioral Signals

Emulated signals to imitate or detect real human input:

| Signal Type | Source |
|-------------|--------|
| `mouseMoveEvents.recent` | Real-time mouse tracking over short intervals |
| `mouseMoveEvents.throttled` | Throttled historical motion |
| `mouseButtonEvents` | Mouse down/up sequences with delay |
| `keyup event` | Key code, location, modifier state |
| `deviceorientation` | Orientation sensor values (alpha, beta, gamma) |
| `numOrientationEvents` | Count and timing of orientation events |
| `DOMContentLoaded` | Fires `initCustomEvent` with telemetry block |
| `requestAnimationFrame` | Timed simulation frame callback |
| `localStorage.getItem/setItem` | Script tests data storage capability |
| `document.visibilityState` | Checks tab state (e.g., `hidden`/`visible`) |
| `document.querySelectorAll()` | Used to validate document parsing works |

---

### 4. 🧪 JavaScript Runtime Integrity

Used to detect sandboxes, bots, or fake execution contexts:

| Check | Purpose |
|-------|---------|
| `hasGlobal`, `hasProcess` | Detects Node or Puppeteer |
| `hasArguments`, `argumentsValue`, `argumentsHasCycle` | Detects scope behavior |
| `toSource()` | Detects native code tampering |
| `window.Error`, `TypeError`, `Function`, etc. | Ensures globals are present |
| `Object`, `Array`, `Math`, `parseInt`, etc. | JS built-ins validation |
| `document.createElement('a')` | URL parsing simulation |
| `navigator.mediaDevices` | Ensures device access |
| `indexedDB.open()` | Triggers exception if not supported |
| `HTMLFormElement` | HTML object presence |
| `fetch`, `XMLHttpRequest.send()` | Ensures APIs exist for XHR/fetch |
| `console.log`, `console.debug` | Anti-debugging or console usage checks |

---

### 5. ⏱️ Timing Analysis

Used to detect automation via unnatural response time patterns:

| Check | Description |
|-------|-------------|
| `requestAnimationFrame timing` | Frame time between calls |
| `initCustomEvent timestamp` | Marks challenge start time |
| `timeToDelay calculation` | Difference from expected event timing |
| `event timestamp normalization` | Mouse/keyboard delays adjusted by coefficient |
| `setTimeout(fn, delay)` | May be overwritten to fake responsiveness |
| `avgInterval`, `stdDevInterval` | Mean/std of sensor events |

---

### 6. 🔍 Static JavaScript Metrics

Used to detect replayed payloads or tampering with challenge logic:

| Metric | Description |
|--------|-------------|
| `function length` | Length of the decoded JS string |
| `whitespace count` | Counts spaces to detect reformatting |
| `punctuator count` | Measures how many `{}[]();` exist |
| `bodyAttribute`, `scriptPresent` | DOM flag spoofing detection |

---

## 📦 JSON Representation of All Checks

```json
[
  {
    "category": "Browser Fingerprint",
    "checks": [
      "navigator.userAgent",
      "navigator.platform",
      "navigator.appVersion",
      "navigator.vendor",
      "navigator.plugins",
      "navigator.mimeTypes",
      "navigator.hardwareConcurrency",
      "navigator.maxTouchPoints",
      "navigator.webdriver",
      "navigator.mediaDevices.enumerateDevices",
      "navigator.productSub",
      "navigator.doNotTrack",
      "navigator.oscpu"
    ]
  },
  {
    "category": "Graphics Fingerprint",
    "checks": [
      "canvas.toDataURL",
      "webgl.getParameter",
      "webgl.getExtension",
      "webgl.getContextAttributes",
      "webgl.getSupportedExtensions",
      "shaderPrecisions",
      "GPU identification (ANGLE/Intel/NVIDIA)"
    ]
  },
  {
    "category": "Behavioral Signals",
    "checks": [
      "mouseMoveEvents.recent",
      "mouseMoveEvents.throttled",
      "mouseButtonEvents",
      "keyup event",
      "deviceorientation",
      "numOrientationEvents",
      "DOMContentLoaded event",
      "requestAnimationFrame",
      "document.visibilityState",
      "localStorage access",
      "querySelectorAll",
      "document.createElement('canvas', 'a')"
    ]
  },
  {
    "category": "JS Runtime Integrity",
    "checks": [
      "hasGlobal",
      "hasProcess",
      "hasArguments",
      "argumentsValue",
      "argumentsHasCycle",
      "toSource presence",
      "window globals (Function, Object, Array, Math, etc)",
      "console logging",
      "HTMLFormElement",
      "fetch and XMLHttpRequest",
      "indexedDB",
      "navigator.mediaDevices",
      "Error/TypeError objects"
    ]
  },
  {
    "category": "Timing Analysis",
    "checks": [
      "requestAnimationFrame timing",
      "initCustomEvent timestamp",
      "event timing normalization",
      "timeToDelay calculation",
      "avgInterval",
      "stdDevInterval",
      "setTimeout behavior"
    ]
  },
  {
    "category": "Static JS Metrics",
    "checks": [
      "function length",
      "whitespace count",
      "punctuator count",
      "scriptPresent",
      "bodyAttribute spoofing"
    ]
  }
]
```


## 🔍 What Shape Checks — In Detail

| **Category**                     | **Checked Directly** | **How/Why Shape Cares**                                                                 |
|----------------------------------|----------------------|------------------------------------------------------------------------------------------|
| `localStorage`                  | ✅ Yes               | Looks for specific keys, values, and size. Detects fresh, empty, or scripted profiles.   |
| `sessionStorage`                | ✅ Yes               | Tracks session continuity and key-value size/frequency across requests.                 |
| `indexedDB`                     | ✅ Yes               | Used by many real-world extensions and sites. Absence can suggest automation.            |
| `cookies`                       | ✅ Yes               | Evaluates age, domain, and scope of cookies to infer history depth and reuse.            |
| `bookmarks`                     | ❌ No (but inferred) | JS cannot access bookmarks, but may infer user history via cookie age, autofill, etc.    |
| `autofill` / form memory        | ❌ No (but inferred) | Browser hints like filled inputs or saved addresses may indicate user profile depth.     |
| `WebGL / Canvas / Fonts`        | ✅ Yes               | Full GPU fingerprint via rendering, shader precision, and font probing.                  |
| `Plugins / MimeTypes`           | ✅ Yes               | Real plugins/mime types suggest legit usage (e.g. PDF Viewer, Widevine).                 |
| `Mouse/Keyboard/Scroll events`  | ✅ Yes               | Observes timing, density, and sequence of interactions for human-likeness.               |
| `deviceorientation events`      | ✅ Yes               | Orientation changes or motion sensor presence on mobile/tablet.                         |
| `navigator.mediaDevices`        | ✅ Yes               | Probes for audio/video device IDs — spoofing or lack of these is suspicious. (PLUG A MONITOR & SPEAKERS INTO YOUR DEVICE)             |
| `navigator.* properties`        | ✅ Yes               | Collects userAgent, platform, vendor, concurrency, webdriver, etc.                       |
| `requestAnimationFrame` timing | ✅ Yes               | Measures event timing jitter and responsiveness.                                         |
| `function length`, `whitespace` | ✅ Yes               | Code fingerprinting — detects script modification or replay.                             |
| `eval`/`Function` integrity     | ✅ Yes               | Confirms JS environment is native and not wrapped/emulated.                             |
| `console`, `Error`, `TypeError` | ✅ Yes               | Detects debugging tools, headless environments, or polyfills.                            |
| `indexedDB.open()` exception    | ✅ Yes               | Validates feature support — absence triggers red flags.                                 |

---

## 📌 How Should You Treat Storage, Cookies, and History

| **Signal**                      | **Risk if Absent**            | **Spoofing Strategy**                                                                   |
|--------------------------------|-------------------------------|------------------------------------------------------------------------------------------|
| `localStorage` empty           | 🚩 High suspicion              | Populate with benign keys (e.g. settings, feature flags, site preferences).              |
| `sessionStorage` missing       | 🚩 High suspicion              | Include short-lived session data like form progress or click tracking.                   |
| `indexedDB` missing            | 🚩 Medium suspicion            | Use popular extensions that auto-create it, or simulate known DB names/keys.             |
| `cookies` always fresh         | 🚩 High suspicion              | Reuse cookies across sessions, mimic realistic expiration and `Set-Cookie` history.      |
| `bookmarks` missing            | ✅ OK (not directly checked)   | No action needed unless you're crafting a deeply "seasoned" profile.                     |
| `autofill`, form history       | 🚩 Medium suspicion            | Pre-fill forms or allow the browser to suggest autofill where possible.                  |
| `navigator.plugins` empty      | 🚩 High suspicion              | Install common plugins/extensions with detectable fingerprints (e.g. PDF Viewer).         |
| `mediaDevices` empty or stub   | 🚩 High suspicion              | Expose 1–3 fake audio input devices with plausible IDs.                                  |
| `requestAnimationFrame` too perfect | 🚩 High suspicion        | Add random jitter to render times or simulate real frame drops.                          |














