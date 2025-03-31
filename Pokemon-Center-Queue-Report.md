
# 📝 Technical Analysis Report  
## Pokémon Center Online Waiting Room System  
**By Zpixels**

---

## 📌 Objective  
To analyze the Pokémon Center’s virtual waiting room system in-depth, documenting its mechanisms, structure, and client-side behavior for high-traffic queuing, with a focus on functionality, persistence, user interaction, and anti-bot enforcement.

---

## 📦 Contents  

1. [System Purpose & Behavior](#1-system-purpose--behavior)  
2. [Architecture Overview](#2-architecture-overview)  
3. [Key Components](#3-key-components)  
4. [Web Worker Usage: HackTimer.js](#4-web-worker-usage-hacktimerjs)  
5. [Queue State Management](#5-queue-state-management)  
6. [Polling & Queue Logic](#6-polling--queue-logic)  
7. [Anti-Bot & Security Integration](#7-anti-bot--security-integration)  
8. [UI Behavior](#8-ui-behavior)  
9. [Queue Lifecycle & Persistence](#9-queue-lifecycle--persistence)  
10. [Summary of Observations](#10-summary-of-observations)  

---

## 1. 🎯 System Purpose & Behavior  
The system is designed to manage **heavy traffic surges** (e.g., during special product drops) by:
- Preventing server overload  
- Fairly pacing user access  
- Creating a **virtual queue** where users wait their turn  
- **Automatically redirecting** users once they’re allowed entry  

---

## 2. 🏗️ Architecture Overview  
The waiting room functions as a **self-contained HTML page** with embedded JavaScript that:
- **Polls** the server for position updates  
- Shows queue info (position, time-to-wait)  
- Uses **Web Workers** to maintain timer accuracy  
- Communicates with the server via **Incapsula** (Imperva’s Web Application Firewall)  
- Uses `localStorage`/`sessionStorage` to maintain session across reloads

---

## 3. 🔧 Key Components  

### HTML Elements
- `<div id="loader">`: Placeholder while queue data is loading  
- `<div class="wrapper">`: Main UI shown when user is in queue  
- `<span id="ttw">`: Estimated time to wait  
- `<span id="position">`: Position in queue  
- Branding elements: Pikachu SVG, Pokémon Center logo  

### JavaScript Behavior  
- Loads scripts from obscured or security-protected endpoints (e.g., `/vice-come-Soldenyson...`)  
- Uses a custom `HackTimer.js` implementation to **override standard `setInterval`/`setTimeout`**

---

## 4. 🧵 Web Worker Usage: HackTimer.js  
Browsers throttle `setTimeout`/`setInterval` when tabs are backgrounded. This system **bypasses** that by:
- Creating a **Blob URL** to define a Web Worker script in memory  
- Replacing `setTimeout`/`setInterval` with messages to the worker  
- Ensuring timers execute **regardless of tab visibility**  
- Allows for precise polling intervals even if tab is inactive

#### Example Worker Message:
```js
postMessage({ name: 'setTimeout', fakeId: 1234, time: 1000 });
```

---

## 5. 💾 Queue State Management  

### Storage Keys
- `localStorage["868_ttw"]`: Time-to-wait, in `HH:MM:SS` format  
- `localStorage["868_position"]`: User's queue position  
- `sessionStorage["isReloading"]`: Flag to differentiate reload vs tab close

### Behavior
- On **refresh**: stored position/time is reused  
- On **tab close**: data is cleared (unless user fakes a refresh)

---

## 6. 🔄 Polling & Queue Logic  

### Polling Endpoint
```
/_Incapsula_Resource?SWWRGTS=868
```

### Polling Logic
- Polls every 1 second  
- On success:
  - If `pending === 1`: update TTW and position  
  - If `pending === 0 || rld === 1`: reload the page  
- On error:
  - Store last known position
  - Reload the page

---

## 7. 🛡️ Anti-Bot & Security Integration  

Powered by **Imperva Incapsula**:
- Routes traffic through `/vice-...` and `/_Incapsula_Resource?...`
- Validates human behavior through:
  - IP fingerprinting  
  - Mouse movement, focus detection  
  - Timing characteristics  
  - JavaScript challenges  

This makes it **very difficult** for headless browsers or basic bots to enter or remain in the queue.

---

## 8. 💻 UI Behavior

- Users see a **themed message** with Pikachu and brand imagery  
- Once queue data is available, the UI does not display it.

  
### Mobile Considerations
- Responsive design adapts layout and font size for screens `< 768px`

---

## 9. ♻️ Queue Lifecycle & Persistence

| Event              | Behavior                              |
|--------------------|----------------------------------------|
| Refresh (F5)       | Position/time reused from `localStorage` |
| Tab Close          | Position lost unless session faked     |
| Queue Completion   | Redirect to site, clear all data       |
| Idle Timeout       | Reload every 10 minutes to prevent staleness |

---

## 10. 📋 Summary of Observations  

| Feature | Details |
|--------|---------|
| **Queue Mechanism** | Polls every 1s via Incapsula endpoint |
| **Persistence** | Time/position saved in localStorage |
| **Timer Accuracy** | Maintained via `HackTimer.js` Web Worker |
| **UI/UX** | Branded, friendly messaging with responsive layout |
| **Security** | Protected by Imperva Incapsula WAF |
| **Bot Resistance** | Strong — includes detection scripts, challenge pages |
| **Data Retention** | Queue state cleared on success or full reload |
| **Scalability** | Client-side queue updates reduce server strain |

---

## 🔚 Final Notes  

This system is **robust, secure, and user-friendly**, with good defenses against abuse and automation.  
By outsourcing timing and state tracking to the client, it offloads pressure from the server, while Incapsula handles authentication, validation, and security.

---

