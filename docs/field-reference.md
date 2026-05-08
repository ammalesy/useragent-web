# User Agent Inspector — Field Reference

**Version:** 2.0 | **Date:** May 2026 | **Project:** useragent-web

---

## 1. User-Agent String

| Attribute | Detail |
|-----------|--------|
| **Definition** | A text string sent by the browser in every HTTP request identifying the application, OS, and platform. |
| **API** | `navigator.userAgent` |

**Extraction:**

```javascript
const ua = navigator.userAgent;
```

**Example output:**

```
Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/125.0.0.0 Safari/537.36
```

**Platform Variations:**

| Platform | Typical UA Pattern |
|----------|--------------------|
| Desktop (macOS) | `Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/...` |
| Desktop (Windows) | `Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/...` |
| iOS (Safari) | `Mozilla/5.0 (iPhone; CPU iPhone OS 17_5 like Mac OS X) AppleWebKit/...` |
| iOS (Chrome) | Same as Safari (WebKit required on iOS) + `CriOS/125.0` |
| Android (Chrome) | `Mozilla/5.0 (Linux; Android 14; Pixel 8) AppleWebKit/... Chrome/125.0 Mobile Safari/...` |
| Android (Samsung) | Same structure + `SamsungBrowser/25.0` |
| iPad (Desktop mode) | Identical to macOS — use `maxTouchPoints` to differentiate |

**Key note:** On iOS, all browsers (Chrome, Firefox, etc.) are forced to use WebKit. Their UA strings differ only by a version token (e.g., `CriOS`, `FxiOS`), not by engine.

---

## 2. Browser Environment

These values come from JavaScript APIs, not the UA string.

---

### 2.1 Touch Support

| Attribute | Detail |
|-----------|--------|
| **What it is** | How many fingers the screen can detect simultaneously. `0` means no touch screen. |
| **API** | `navigator.maxTouchPoints` |

```javascript
const points = navigator.maxTouchPoints; // 0 = no touch, 5 = typical tablet/phone
```

| Platform | Typical Value |
|----------|---------------|
| Desktop (no touch) | `0` |
| iPad / iPhone | `5` |
| Android phone/tablet | `5` or `10` |
| Windows touch laptop | `10` |
| Desktop + external touch | `1–10` |

**Use case:** Distinguishes iPad-in-desktop-mode (`Macintosh` UA + `maxTouchPoints > 1`) from a real Mac (`maxTouchPoints === 0`).

---

### 2.2 Platform

| Attribute | Detail |
|-----------|--------|
| **What it is** | A short string identifying the OS/CPU the browser runs on. |
| **API** | `navigator.platform` (deprecated but functional) |

```javascript
const platform = navigator.platform;
```

| Platform | Value |
|----------|-------|
| macOS (Intel/Apple Silicon) | `MacIntel` |
| iPad (desktop mode) | `MacIntel` |
| Windows | `Win32` |
| Linux | `Linux x86_64` |
| iPhone | `iPhone` |
| Android | `Linux armv81` or `Linux aarch64` |

---

### 2.3 Cookies

| Attribute | Detail |
|-----------|--------|
| **What it is** | Whether the browser accepts first-party cookies. |
| **API** | `navigator.cookieEnabled` |

```javascript
const enabled = navigator.cookieEnabled; // boolean
```

| Platform | Default |
|----------|---------|
| All modern browsers | `true` |
| Private/incognito mode | `true` (still accepts session cookies) |
| Explicitly disabled by user | `false` |

---

### 2.4 CPU Cores

| Attribute | Detail |
|-----------|--------|
| **What it is** | Number of logical CPU threads available to the browser. |
| **API** | `navigator.hardwareConcurrency` |

```javascript
const cores = navigator.hardwareConcurrency; // e.g., 8
```

| Platform | Typical Values |
|----------|---------------|
| Desktop | 4–16+ |
| iPhone / iPad | 6 (capped by Safari) |
| Android mid-range | 4–8 |
| Android flagship | 8 |

**Note:** Safari on iOS may report a lower value than the physical core count for privacy.

---

### 2.5 Screen Resolution

| Attribute | Detail |
|-----------|--------|
| **What it is** | Display size in CSS pixels + device pixel ratio (DPR). |
| **API** | `screen.width`, `screen.height`, `window.devicePixelRatio` |

```javascript
const w = screen.width;          // CSS px
const h = screen.height;         // CSS px
const dpr = devicePixelRatio;    // physical px per CSS px
```

| Platform | Resolution | DPR |
|----------|-----------|-----|
| MacBook Pro 14" | 1512 × 982 | 2 |
| iPhone 15 Pro | 393 × 852 | 3 |
| iPad Pro 12.9" | 1024 × 1366 | 2 |
| Android 1080p phone | 412 × 915 | 2.625 |
| Full HD desktop | 1920 × 1080 | 1 |

---

### 2.6 Viewport

| Attribute | Detail |
|-----------|--------|
| **What it is** | The visible page area (excludes browser chrome, address bar, toolbars). |
| **API** | `window.innerWidth`, `window.innerHeight` |

```javascript
const vw = window.innerWidth;
const vh = window.innerHeight;
```

**Note:** On mobile, viewport height changes when the virtual keyboard appears or the address bar collapses.

---

### 2.7 Device Memory

| Attribute | Detail |
|-----------|--------|
| **What it is** | Approximate device RAM in GB (rounded to power of 2 for privacy). |
| **API** | `navigator.deviceMemory` |

```javascript
const ram = navigator.deviceMemory; // 0.25, 0.5, 1, 2, 4, or 8
```

| Platform | Support |
|----------|---------|
| Chrome / Edge / Opera | Yes |
| Firefox | No (`undefined`) |
| Safari (iOS/macOS) | No (`undefined`) |
| Samsung Browser | Yes |

---

### 2.8 Color Depth

| Attribute | Detail |
|-----------|--------|
| **What it is** | Bits per pixel the display uses for color. |
| **API** | `screen.colorDepth` |

```javascript
const bits = screen.colorDepth; // 24 or 30
```

| Value | Meaning |
|-------|---------|
| 24 | Standard True Color (8 bits × RGB) — 16.7M colors |
| 30 | HDR / Deep Color (10 bits × RGB) — 1B colors |

Consistent across all platforms; value depends on hardware, not OS.

---

### 2.9 Reduced Motion Preference

| Attribute | Detail |
|-----------|--------|
| **What it is** | User has enabled "Reduce Motion" in OS accessibility settings to minimize animations. |
| **API** | `window.matchMedia("(prefers-reduced-motion: reduce)")` |

```javascript
const reduced = matchMedia("(prefers-reduced-motion: reduce)").matches; // boolean
```

| Platform | Where to enable |
|----------|----------------|
| iOS | Settings → Accessibility → Motion → Reduce Motion |
| Android | Settings → Accessibility → Remove animations |
| macOS | System Settings → Accessibility → Display → Reduce motion |
| Windows | Settings → Accessibility → Visual effects → Animation effects OFF |

---

### 2.10 Network Information

| Attribute | Detail |
|-----------|--------|
| **What it is** | Estimated connection quality (not actual WiFi/cellular type). |
| **API** | `navigator.connection` (Network Information API) |

```javascript
const conn = navigator.connection;
if (conn) {
  conn.effectiveType; // "slow-2g" | "2g" | "3g" | "4g"
  conn.downlink;      // Mbps (estimated)
  conn.rtt;           // ms (round-trip)
}
```

| Platform | Support |
|----------|---------|
| Chrome / Edge / Opera / Samsung | Yes |
| Firefox | No |
| Safari (iOS/macOS) | No |

| Effective Type | Meaning |
|---------------|---------|
| `4g` | ≥ 700 Kbps, ≤ 270 ms RTT |
| `3g` | < 700 Kbps, > 270 ms RTT |
| `2g` | < 70 Kbps, > 1400 ms RTT |
| `slow-2g` | < 50 Kbps, > 2000 ms RTT |

---

### 2.11 JavaScript Status

| Attribute | Detail |
|-----------|--------|
| **What it is** | Confirms JS is running. If this code executes, it's enabled. |
| **API** | Implicit |

```javascript
// Always true if code reaches this point
const jsEnabled = true;
```

For fallback when JS is off, use `<noscript>` in HTML.

---

## 3. Summary

| # | Field | API | iOS | Android | Desktop |
|---|-------|-----|-----|---------|---------|
| 1 | User-Agent | `navigator.userAgent` | ✅ | ✅ | ✅ |
| 2 | Touch Support | `navigator.maxTouchPoints` | ✅ (5) | ✅ (5–10) | ✅ (0 or 1–10) |
| 3 | Platform | `navigator.platform` | ✅ | ✅ | ✅ |
| 4 | Cookies | `navigator.cookieEnabled` | ✅ | ✅ | ✅ |
| 5 | CPU Cores | `navigator.hardwareConcurrency` | ✅ (capped) | ✅ | ✅ |
| 6 | Screen Resolution | `screen.width/height` + `devicePixelRatio` | ✅ | ✅ | ✅ |
| 7 | Viewport | `window.innerWidth/innerHeight` | ✅ | ✅ | ✅ |
| 8 | Device Memory | `navigator.deviceMemory` | ❌ | ✅ (Chrome) | ✅ (Chrome/Edge) |
| 9 | Color Depth | `screen.colorDepth` | ✅ | ✅ | ✅ |
| 10 | Reduced Motion | `matchMedia("prefers-reduced-motion")` | ✅ | ✅ | ✅ |
| 11 | Network Info | `navigator.connection` | ❌ | ✅ (Chrome) | ✅ (Chrome/Edge) |
| 12 | JavaScript | Implicit | ✅ | ✅ | ✅ |

---

## 4. References

1. MDN — Navigator API: https://developer.mozilla.org/en-US/docs/Web/API/Navigator
2. MDN — User-Agent: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/User-Agent
3. W3C — Network Information API: https://wicg.github.io/netinfo/
4. W3C — Device Memory: https://www.w3.org/TR/device-memory/

---

*useragent-web Project — May 2026*
