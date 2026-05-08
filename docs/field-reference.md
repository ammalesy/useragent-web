# User Agent Inspector — Field Reference Document

> เอกสารนี้อธิบายรายละเอียดของทุก field ที่เว็บไซต์ User Agent Inspector ทำการตรวจจับและแสดงผล  
> แบ่งเป็น 2 กลุ่มหลัก คือ **UA-String Values** (ข้อมูลจากสตริง User-Agent) และ **Browser Environment** (ข้อมูลจาก Browser APIs)

---

## 1. UA-String Values (ข้อมูลจากสตริง User-Agent)

ค่าเหล่านี้ถูก **parse จากข้อความ `navigator.userAgent`** โดยตรง ซึ่งเป็น string ที่เบราว์เซอร์ส่งไปใน HTTP header `User-Agent` ทุกครั้งที่ request เกิดขึ้น

---

### 1.1 Browser (เบราว์เซอร์)

| รายการ | รายละเอียด |
|--------|------------|
| **ความหมาย** | ชื่อและเวอร์ชันของเบราว์เซอร์ที่ผู้ใช้กำลังใช้งาน |
| **ตัวอย่างค่า** | Chrome 125.x, Firefox 126.x, Edge 125.x, Safari 17.x |

**วิธีการดึงค่า:**

```javascript
const ua = navigator.userAgent;

// กำหนด regex ลำดับจากเฉพาะเจาะจง → ทั่วไป (เพราะ Chrome token ปรากฏใน Edge/Opera ด้วย)
const browsers = [
  { name: "Edge",            rx: /Edg(?:e|A|iOS)?\/([0-9.]+)/ },
  { name: "Samsung Browser", rx: /SamsungBrowser\/([0-9.]+)/ },
  { name: "Opera",           rx: /OPR\/([0-9.]+)|Opera\/([0-9.]+)/ },
  { name: "Chrome",          rx: /Chrome\/([0-9.]+)/ },
  { name: "Firefox",         rx: /Firefox\/([0-9.]+)/ },
  { name: "Safari",          rx: /Version\/([0-9.]+).*Safari/ },
  { name: "IE",              rx: /MSIE ([0-9.]+)|Trident\/.*rv:([0-9.]+)/ },
];

let browser = { name: "Unknown", version: "—" };
for (const b of browsers) {
  const m = ua.match(b.rx);
  if (m) {
    browser = { name: b.name, version: m[1] || m[2] };
    break;
  }
}
```

**หลักการ:** UA string จะประกอบด้วย product token หลายตัว เช่น `Mozilla/5.0 ... Chrome/125.0.6422.77 Safari/537.36` — ต้อง match ตัวที่เฉพาะเจาะจงก่อน เพราะ Edge จะมีทั้ง `Chrome/` และ `Edg/` อยู่ในสตริงเดียวกัน

---

### 1.2 Engine (เอนจินเรนเดอร์)

| รายการ | รายละเอียด |
|--------|------------|
| **ความหมาย** | Layout/rendering engine ที่เบราว์เซอร์ใช้แสดงผล HTML/CSS |
| **ค่าที่เป็นไปได้** | WebKit / Blink, Gecko, Trident, Presto |

**วิธีการดึงค่า:**

```javascript
let engine = "Unknown";
if (/Gecko\/[0-9]/.test(ua) && /Firefox/.test(ua)) engine = "Gecko";
else if (/AppleWebKit\/[0-9]/.test(ua))            engine = "WebKit / Blink";
else if (/Trident\//.test(ua))                     engine = "Trident";
else if (/Presto\//.test(ua))                      engine = "Presto";

// ดึงเวอร์ชัน
const engVer = (ua.match(/(?:AppleWebKit|Gecko|Trident)\/([0-9.]+)/) || [])[1] || "—";
```

**หลักการ:**
- `AppleWebKit/xxx` → Chrome, Safari, Edge (Chromium-based) ใช้ Blink ซึ่งแยกมาจาก WebKit
- `Gecko/xxx` + `Firefox` → Firefox ใช้ Gecko engine
- `Trident/xxx` → Internet Explorer
- `Presto/xxx` → Opera รุ่นเก่า (ก่อน v15)

---

### 1.3 Operating System (ระบบปฏิบัติการ)

| รายการ | รายละเอียด |
|--------|------------|
| **ความหมาย** | ระบบปฏิบัติการและเวอร์ชันที่อุปกรณ์ใช้งานอยู่ |
| **ค่าที่เป็นไปได้** | Windows 11, Windows 10, macOS, iPadOS, iOS, Android, Linux, ChromeOS |

**วิธีการดึงค่า:**

```javascript
const osList = [
  { name: "Windows 11",  rx: /Windows NT 10\.0.*Win64/ },
  { name: "Windows 10",  rx: /Windows NT 10\.0/ },
  { name: "Windows 8.1", rx: /Windows NT 6\.3/ },
  { name: "macOS",       rx: /Mac OS X ([0-9_]+)/ },
  { name: "iOS",         rx: /iPhone OS ([0-9_]+)/ },
  { name: "iPadOS",      rx: /iPad.*OS ([0-9_]+)/ },
  { name: "Android",     rx: /Android ([0-9.]+)/ },
  { name: "Linux",       rx: /Linux/ },
  { name: "ChromeOS",    rx: /CrOS/ },
];

let os = { name: "Unknown", version: "—" };
for (const o of osList) {
  const m = ua.match(o.rx);
  if (m) {
    os = { name: o.name, version: m[1] ? m[1].replace(/_/g, ".") : "—" };
    break;
  }
}

// กรณีพิเศษ: iPad ที่เปิด Request Desktop Website
if (os.name === "macOS" && navigator.maxTouchPoints > 1) {
  os.name = "iPadOS";
  os.version = "(Desktop mode — version hidden)";
}
```

**หลักการ:** ข้อมูล OS จะอยู่ในวงเล็บของ UA string เช่น `(Windows NT 10.0; Win64; x64)` หรือ `(Macintosh; Intel Mac OS X 10_15_7)` — ใช้ regex จับ pattern ดังกล่าว โดย Windows NT version number mapping เป็น:
- `10.0` = Windows 10/11
- `6.3` = Windows 8.1
- `6.2` = Windows 8
- `6.1` = Windows 7

**กรณีพิเศษ iPad:** เมื่อ iPad เปิด "Request Desktop Website" (ค่าเริ่มต้นตั้งแต่ iPadOS 13) UA จะส่งเป็น `Macintosh` เหมือน Mac จริง — ใช้ `navigator.maxTouchPoints > 1` เพื่อแยกแยะ (Mac จริง = 0, iPad = 5)

---

### 1.4 Device Type (ประเภทอุปกรณ์)

| รายการ | รายละเอียด |
|--------|------------|
| **ความหมาย** | ประเภทของอุปกรณ์ที่กำลังเข้าถึงเว็บไซต์ |
| **ค่าที่เป็นไปได้** | Desktop, Mobile Phone, Tablet (iPad), Smart TV, Bot / Crawler |

**วิธีการดึงค่า:**

```javascript
const isFakeDesktopIPad = /Macintosh/.test(ua) && navigator.maxTouchPoints > 1;

let deviceType = "Desktop";
if (isFakeDesktopIPad)                               deviceType = "Tablet (iPad)";
else if (/Mobile|Android.*Mobile/.test(ua))          deviceType = "Mobile Phone";
else if (/iPad|Android(?!.*Mobile)|Tablet/.test(ua)) deviceType = "Tablet";
else if (/TV|SmartTV|HbbTV/.test(ua))                deviceType = "Smart TV";
else if (/bot|crawl|spider/i.test(ua))               deviceType = "Bot / Crawler";
```

**หลักการ:**
- `Mobile` keyword → มือถือ
- `iPad` หรือ Android ที่ไม่มี `Mobile` → แท็บเล็ต
- `Macintosh` + touch points > 1 → iPad ในโหมด Desktop
- ไม่มี keyword ใดเลย → Desktop

---

### 1.5 Architecture (สถาปัตยกรรม CPU)

| รายการ | รายละเอียด |
|--------|------------|
| **ความหมาย** | สถาปัตยกรรมของ CPU ที่อุปกรณ์ใช้ |
| **ค่าที่เป็นไปได้** | x86-64 (64-bit), x86 (32-bit), ARM, Unknown |

**วิธีการดึงค่า:**

```javascript
let arch = "Unknown";
if (/Win64|x64|x86_64|WOW64/.test(ua))  arch = "x86-64 (64-bit)";
else if (/Win32|i686|i386/.test(ua))     arch = "x86 (32-bit)";
else if (/arm|aarch64/i.test(ua))        arch = "ARM";
```

**หลักการ:** UA string มักบรรจุข้อมูลสถาปัตยกรรมไว้ในส่วน comment เช่น `Win64; x64` หรือ `Linux aarch64` — ใช้ regex จับ keywords ที่บ่งบอกประเภท CPU

---

## 2. Browser Environment (ข้อมูลจาก Browser APIs)

ค่าเหล่านี้ **ไม่ได้มาจาก UA string** แต่ได้จากการเรียก JavaScript APIs ต่างๆ ที่เบราว์เซอร์เปิดให้ใช้งาน

---

### 2.1 Language (ภาษา)

| รายการ | รายละเอียด |
|--------|------------|
| **ความหมาย** | ภาษาที่ผู้ใช้ตั้งค่าไว้ในเบราว์เซอร์ (ส่งเป็น Accept-Language header ด้วย) |
| **ตัวอย่างค่า** | th, en-US, ja |

**วิธีการดึงค่า:**

```javascript
const lang = navigator.language;                   // ภาษาหลัก เช่น "th"
const allLangs = navigator.languages;              // ลำดับภาษาทั้งหมด เช่น ["th", "en-US", "en"]
```

**อธิบาย:** `navigator.language` คืนค่าภาษาหลักของเบราว์เซอร์ ส่วน `navigator.languages` เป็น array ลำดับความชอบของภาษา (ตรงกับ Accept-Language header)

---

### 2.2 Platform (แพลตฟอร์ม)

| รายการ | รายละเอียด |
|--------|------------|
| **ความหมาย** | สตริงระบุ OS/CPU platform ที่เบราว์เซอร์ compile ขึ้นมา |
| **ตัวอย่างค่า** | MacIntel, Win32, Linux x86_64, iPhone |

**วิธีการดึงค่า:**

```javascript
const platform = navigator.platform;
```

**อธิบาย:** `navigator.platform` เป็น API เก่า (deprecated แต่ยังใช้ได้) ที่คืนค่าสตริงบอกว่าเบราว์เซอร์ทำงานบน platform อะไร — ค่านี้อาจไม่ตรงกับ OS จริงเสมอไป (เช่น iPad desktop mode จะคืน `MacIntel`)

---

### 2.3 Cookies (คุกกี้)

| รายการ | รายละเอียด |
|--------|------------|
| **ความหมาย** | ระบุว่าเบราว์เซอร์รับ cookies หรือไม่ |
| **ค่าที่เป็นไปได้** | Enabled, Disabled |

**วิธีการดึงค่า:**

```javascript
const cookiesEnabled = navigator.cookieEnabled;    // boolean
```

**อธิบาย:** เบราว์เซอร์ส่วนใหญ่ enable cookies ตามค่าเริ่มต้น แต่ผู้ใช้หรือ extension อาจ disable ได้ — ค่านี้สำคัญสำหรับการเก็บ session/token

---

### 2.4 CPU Cores (จำนวนคอร์ CPU)

| รายการ | รายละเอียด |
|--------|------------|
| **ความหมาย** | จำนวน logical CPU threads ที่เบราว์เซอร์เข้าถึงได้ |
| **ตัวอย่างค่า** | 4, 8, 10, 16 logical cores |

**วิธีการดึงค่า:**

```javascript
const cores = navigator.hardwareConcurrency;       // number
```

**อธิบาย:** คืนจำนวน logical processors (รวม hyper-threading) ที่ใช้ได้ — มีประโยชน์สำหรับ Web Workers และการทำ parallel processing ฝั่ง client

---

### 2.5 Screen Resolution (ความละเอียดหน้าจอ)

| รายการ | รายละเอียด |
|--------|------------|
| **ความหมาย** | ขนาดหน้าจอทั้งหมดของอุปกรณ์ (หน่วย CSS pixels) และ Device Pixel Ratio |
| **ตัวอย่างค่า** | 1920 × 1080, 2x DPR |

**วิธีการดึงค่า:**

```javascript
const width = screen.width;                        // ความกว้าง CSS pixels
const height = screen.height;                      // ความสูง CSS pixels
const dpr = window.devicePixelRatio;               // อัตราส่วน physical/CSS pixels
```

**อธิบาย:** `screen.width/height` คือขนาดจอทั้งหมด ส่วน `devicePixelRatio` บอกว่า 1 CSS pixel = กี่ physical pixels (เช่น Retina display = 2x หรือ 3x)

---

### 2.6 Viewport (ขนาดพื้นที่แสดงผล)

| รายการ | รายละเอียด |
|--------|------------|
| **ความหมาย** | ขนาดพื้นที่ที่มองเห็นได้ของหน้าเว็บ (ไม่รวม toolbar, scrollbar) |
| **ตัวอย่างค่า** | 1440 × 820 px |

**วิธีการดึงค่า:**

```javascript
const vpWidth = window.innerWidth;                 // ความกว้าง viewport
const vpHeight = window.innerHeight;               // ความสูง viewport
```

**อธิบาย:** ต่างจาก `screen.width/height` ตรงที่เป็นขนาด "หน้าต่างเว็บ" จริงๆ — มีประโยชน์สำหรับ responsive design และการตรวจสอบว่าผู้ใช้เปิดหน้าต่างขนาดเท่าใด

---

### 2.7 Touch Support (รองรับระบบสัมผัส)

| รายการ | รายละเอียด |
|--------|------------|
| **ความหมาย** | จำนวน touch points ที่อุปกรณ์รองรับพร้อมกัน |
| **ตัวอย่างค่า** | Yes (5 points), No |

**วิธีการดึงค่า:**

```javascript
const touchPoints = navigator.maxTouchPoints;      // number (0 = ไม่รองรับ)
const hasTouch = touchPoints > 0;
```

**อธิบาย:** บอกว่าอุปกรณ์เป็น touch screen หรือไม่ และรองรับกี่นิ้วพร้อมกัน — ใช้ตรวจจับ iPad ที่ปลอมตัวเป็น Mac ได้ด้วย (Mac = 0, iPad = 5)

---

### 2.8 Timezone (เขตเวลา)

| รายการ | รายละเอียด |
|--------|------------|
| **ความหมาย** | เขตเวลาของเบราว์เซอร์ตามการตั้งค่า OS |
| **ตัวอย่างค่า** | Asia/Bangkok, UTC+7:00 |

**วิธีการดึงค่า:**

```javascript
const tz = Intl.DateTimeFormat().resolvedOptions().timeZone;  // เช่น "Asia/Bangkok"

// คำนวณ UTC offset
const offsetMinutes = -new Date().getTimezoneOffset();        // minutes from UTC
const offsetStr = (offsetMinutes >= 0 ? "UTC+" : "UTC") 
  + Math.floor(offsetMinutes / 60) + ":" 
  + String(Math.abs(offsetMinutes % 60)).padStart(2, "0");    // เช่น "UTC+7:00"
```

**อธิบาย:** `Intl.DateTimeFormat` ให้ชื่อ timezone แบบ IANA (เช่น Asia/Bangkok) ส่วน `getTimezoneOffset()` ให้ค่า offset เป็นนาที — มีประโยชน์สำหรับ localization

---

### 2.9 Device Memory (หน่วยความจำอุปกรณ์)

| รายการ | รายละเอียด |
|--------|------------|
| **ความหมาย** | ปริมาณ RAM โดยประมาณ (ปัดเป็นเลขยกกำลัง 2 เพื่อความเป็นส่วนตัว) |
| **ตัวอย่างค่า** | 8 GB (approx.) |
| **ข้อจำกัด** | ใช้ได้เฉพาะ Chromium-based browsers |

**วิธีการดึงค่า:**

```javascript
const memory = navigator.deviceMemory;             // number: 0.25, 0.5, 1, 2, 4, 8
```

**อธิบาย:** คืนค่าประมาณ RAM ในหน่วย GB — ค่าถูกปัดเป็น power of 2 ใกล้ที่สุดเพื่อป้องกัน fingerprinting ที่แม่นยำเกินไป (เช่น RAM 6 GB จะถูกรายงานว่า 8)

---

### 2.10 Color Depth (ความลึกสี)

| รายการ | รายละเอียด |
|--------|------------|
| **ความหมาย** | จำนวน bits ต่อ pixel ที่จอแสดงผลใช้ |
| **ตัวอย่างค่า** | 24-bit (True Color) |

**วิธีการดึงค่า:**

```javascript
const colorDepth = screen.colorDepth;              // 24 (8 bits × RGB) หรือ 30 (HDR)
const pixelDepth = screen.pixelDepth;              // ส่วนใหญ่เท่า colorDepth
```

**อธิบาย:** 24-bit = 16.7 ล้านสี (8 bit per channel × 3 channels) ซึ่งเป็นมาตรฐาน True Color — จอ HDR อาจรายงาน 30-bit (10 bit per channel)

---

### 2.11 Color Scheme (โทนสีที่ผู้ใช้เลือก)

| รายการ | รายละเอียด |
|--------|------------|
| **ความหมาย** | ผู้ใช้ตั้งค่า OS เป็น Dark mode หรือ Light mode |
| **ค่าที่เป็นไปได้** | Dark, Light |

**วิธีการดึงค่า:**

```javascript
const prefersDark = window.matchMedia("(prefers-color-scheme: dark)").matches;  // boolean
```

**อธิบาย:** CSS media query `prefers-color-scheme` สะท้อนการตั้งค่า dark/light mode ระดับ OS (เช่น macOS Dark Mode, Windows Dark theme) — ใช้สำหรับ auto-theming เว็บไซต์

---

### 2.12 Reduced Motion (ลดการเคลื่อนไหว)

| รายการ | รายละเอียด |
|--------|------------|
| **ความหมาย** | ผู้ใช้เปิดการตั้งค่า accessibility เพื่อลด animation/motion |
| **ค่าที่เป็นไปได้** | Requested, No preference |

**วิธีการดึงค่า:**

```javascript
const prefersReduced = window.matchMedia("(prefers-reduced-motion: reduce)").matches;
```

**อธิบาย:** ผู้ที่มีอาการ motion sickness หรือ vestibular disorders สามารถเปิดตัวเลือกนี้ใน OS — เว็บที่ดีควร respect ค่านี้โดยปิด animation เมื่อ `reduce` ถูก request

---

### 2.13 Network (ข้อมูลเครือข่าย)

| รายการ | รายละเอียด |
|--------|------------|
| **ความหมาย** | ประเภทการเชื่อมต่อที่ประเมินได้ (effective connection type) และความเร็ว |
| **ตัวอย่างค่า** | 4G, 10 Mbps |
| **ข้อจำกัด** | ใช้ได้เฉพาะ Chromium-based browsers (Network Information API) |

**วิธีการดึงค่า:**

```javascript
const conn = navigator.connection || navigator.mozConnection || navigator.webkitConnection;
if (conn) {
  const type = conn.effectiveType;   // "slow-2g", "2g", "3g", "4g"
  const downlink = conn.downlink;    // Mbps (estimated bandwidth)
  const rtt = conn.rtt;             // round-trip time in ms
}
```

**อธิบาย:** `effectiveType` ไม่ใช่ประเภทการเชื่อมต่อจริง (WiFi/Cellular) แต่เป็นการประเมินจาก bandwidth + latency — เช่น WiFi ที่ช้ามากอาจถูกจัดเป็น "3g"

---

### 2.14 Online Status (สถานะออนไลน์)

| รายการ | รายละเอียด |
|--------|------------|
| **ความหมาย** | เบราว์เซอร์สามารถเข้าถึงเครือข่ายได้หรือไม่ |
| **ค่าที่เป็นไปได้** | Online, Offline |

**วิธีการดึงค่า:**

```javascript
const isOnline = navigator.onLine;                 // boolean

// สามารถ listen event ได้
window.addEventListener("online", () => { /* reconnected */ });
window.addEventListener("offline", () => { /* disconnected */ });
```

**อธิบาย:** `navigator.onLine` คืน `true` เมื่อเบราว์เซอร์มี network connectivity — **หมายเหตุ:** `true` ไม่ได้รับประกันว่า internet ใช้งานได้จริง เพียงแค่ network interface ยัง active อยู่

---

### 2.15 Do Not Track (ห้ามติดตาม)

| รายการ | รายละเอียด |
|--------|------------|
| **ความหมาย** | ผู้ใช้แสดงความประสงค์ไม่ให้เว็บไซต์ติดตามพฤติกรรม |
| **ค่าที่เป็นไปได้** | Enabled ("1"), Disabled ("0"), Not set (null/unspecified) |

**วิธีการดึงค่า:**

```javascript
const dnt = navigator.doNotTrack || window.doNotTrack;
// "1" = ผู้ใช้ไม่ต้องการถูกติดตาม
// "0" = ผู้ใช้ยินยอม
// null/undefined = ไม่ได้ตั้งค่า
```

**อธิบาย:** DNT เป็น signal ที่ผู้ใช้ส่งออกมา แต่ไม่มีผล enforce ทางกฎหมาย (เป็นเพียง request) — ปัจจุบันเบราว์เซอร์บางตัว (เช่น Safari) ได้ยกเลิก support แล้ว เนื่องจากกลายเป็น fingerprinting vector แทน

---

### 2.16 PDF Viewer (ตัวแสดง PDF)

| รายการ | รายละเอียด |
|--------|------------|
| **ความหมาย** | เบราว์เซอร์สามารถแสดง PDF inline ได้หรือไม่ |
| **ค่าที่เป็นไปได้** | Supported, Not supported |

**วิธีการดึงค่า:**

```javascript
const hasPdf = navigator.pdfViewerEnabled;         // boolean (undefined ถ้า browser ไม่รองรับ API นี้)
```

**อธิบาย:** เบราว์เซอร์สมัยใหม่ส่วนใหญ่มี built-in PDF viewer — API นี้ถูกเพิ่มเข้ามาเพื่อทดแทนการตรวจสอบผ่าน `navigator.plugins` ที่ถูก deprecated แล้ว

---

### 2.17 JavaScript (สถานะ JavaScript)

| รายการ | รายละเอียด |
|--------|------------|
| **ความหมาย** | JavaScript ถูก enable ในเบราว์เซอร์หรือไม่ |
| **ค่าที่เป็นไปได้** | Enabled (เสมอ — เพราะถ้า disabled จะไม่สามารถ run code นี้ได้) |

**วิธีการดึงค่า:**

```javascript
// ไม่ต้อง detect — ถ้า code นี้ run ได้ แปลว่า JS enabled
const jsEnabled = true;
```

**อธิบาย:** เป็น field ยืนยันว่า JavaScript ทำงานอยู่ — ถ้าต้องการรองรับกรณี disabled ควรใช้ `<noscript>` ใน HTML แทน

---

## สรุปตารางแหล่งที่มา

| # | Field | แหล่งที่มา (API / Property) | กลุ่ม |
|---|-------|----------------------------|-------|
| 1 | Browser | `navigator.userAgent` regex match | UA-String |
| 2 | Engine | `navigator.userAgent` regex match | UA-String |
| 3 | Operating System | `navigator.userAgent` regex + `maxTouchPoints` | UA-String |
| 4 | Device Type | `navigator.userAgent` regex + `maxTouchPoints` | UA-String |
| 5 | Architecture | `navigator.userAgent` regex match | UA-String |
| 6 | Language | `navigator.language` / `navigator.languages` | Browser Env |
| 7 | Platform | `navigator.platform` | Browser Env |
| 8 | Cookies | `navigator.cookieEnabled` | Browser Env |
| 9 | CPU Cores | `navigator.hardwareConcurrency` | Browser Env |
| 10 | Screen Resolution | `screen.width` / `screen.height` / `devicePixelRatio` | Browser Env |
| 11 | Viewport | `window.innerWidth` / `window.innerHeight` | Browser Env |
| 12 | Touch Support | `navigator.maxTouchPoints` | Browser Env |
| 13 | Timezone | `Intl.DateTimeFormat().resolvedOptions().timeZone` | Browser Env |
| 14 | Device Memory | `navigator.deviceMemory` | Browser Env |
| 15 | Color Depth | `screen.colorDepth` / `screen.pixelDepth` | Browser Env |
| 16 | Color Scheme | `matchMedia("prefers-color-scheme")` | Browser Env |
| 17 | Reduced Motion | `matchMedia("prefers-reduced-motion")` | Browser Env |
| 18 | Network | `navigator.connection` (Network Information API) | Browser Env |
| 19 | Online Status | `navigator.onLine` | Browser Env |
| 20 | Do Not Track | `navigator.doNotTrack` | Browser Env |
| 21 | PDF Viewer | `navigator.pdfViewerEnabled` | Browser Env |
| 22 | JavaScript | Implicit (code execution) | Browser Env |

---

> **จัดทำโดย:** User Agent Inspector Project  
> **อัปเดตล่าสุด:** May 2026  
> **Repository:** github.com/ammalesy/useragent-web
