# Sign3 Web SDK Integration Guide

The **Sign3 Web SDK** is a JavaScript-based fraud prevention toolkit designed to assess browser security and detect potential risks such as proxy/VPN usage, user agent spoofing, TOR connections, and more.

By providing real-time browser intelligence, it enhances fraud detection capabilities and strengthens protection against malicious activity.

---

## Adding the Sign3 Web SDK to Your Project

1. Download the JavaScript agent from the provided CDN link and include it in your codebase as `sign3-web-sdk.js`.
2. Call `sign3.initialize(config)` to initialize the SDK.
3. `sign3.initialize(config)` returns a Promise that resolves to an object containing the `get` method.
4. After initialization, call `get()` before critical user actions (e.g., login, payment, registration API calls).
5. Call `result.get().then(response)` to retrieve browser intelligence data.
6. The response object includes fields such as fingerprint, request ID, browser intelligence, and IP intelligence data.

---

# Option A: Bundling Into Your Code

If the SDK file is saved locally as `sign3-web-sdk.js`:

## Initialization

```javascript
import sdk from './sign3-web-sdk.js';

sdk.initialize({
  env: 'PROD', // Required: 'PROD' or 'STAGE'
  sessionId: 'your-unique-session-id', // Required: Unique session identifier
  apiKey: 'your-api-key', // Required: Provided by Sign3
  apiSecret: 'your-api-secret', // Required: Provided by Sign3
}).then((result) => {
  console.log('Initialization successful');
}, (error) => {
  console.log('Initialization error:', error.message);
});
```

---

# Option B: Dynamic Script Loading (Recommended)

> The actual SDK URL will be provided separately by Sign3. The URL shown below is for demonstration purposes only.

---

## Vanilla JavaScript Integration

Load the SDK using a script tag. The global object will be available as `window.hydraSdk`.

It is recommended to initialize the SDK once at application startup (or as early as possible in the page lifecycle) to ensure all signals are captured correctly.

Assume that the CDN url shared by Sign3 is:

```
https://cdn.example.com/sdk/fingerprint.min.js
```

### Initialization

```html
<!-- Load the SDK -->
<script src="https://cdn.example.com/sdk/fingerprint.min.js"></script>

<script>
  (async function () {
    try {
      const sdk = await window.hydraSdk.initialize({
        env: "PROD",
        sessionId: "unique-session-id",
        apiKey: "your-api-key",
        apiSecret: "your-api-secret",
      });

      try {
        const result = await sdk.get({
          userId: "abc123",
          loginIdentifier: "abcd-12erf-rtyv3-pfrtec",
        });

        console.log("Fingerprint:", result);
      } catch (err) {
        console.error("Fingerprint collection failed:", err.message);
      }
    } catch (err) {
      console.error("SDK initialization failed:", err.message);
    }
  })();
</script>
```

---

# React / Next.js — Provider Approach

Load the SDK once in your root layout and expose it via a provider.

---

## Step 1: Load the Script

```javascript
// app/layout.js
import Script from "next/script";

export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <head>
        <link rel="preconnect" href="https://cdn.example.com" />
      </head>
      <body>
        {children}
        <Script
          src="https://cdn.example.com/sdk/fingerprint.min.js"
          strategy="beforeInteractive"
        />
      </body>
    </html>
  );
}
```

---

## Step 2: Create the Provider

```javascript
// providers/fingerprint-provider.jsx
"use client";

import { createContext, useCallback, useContext, useEffect, useRef, useState } from "react";

const FingerprintContext = createContext(null);

export function FingerprintProvider({ sessionId, apiKey, apiSecret, env = "PROD", children }) {
  const [ready, setReady] = useState(false);
  const [error, setError] = useState(null);
  const sdkRef = useRef(null);
  const initPromiseRef = useRef(null);

  useEffect(() => {
    if (!window.hydraSdk) {
      setError(new Error("SDK script not loaded"));
      return;
    }

    if (sdkRef.current) {
      setReady(true);
      return;
    }

    if (!initPromiseRef.current) {
      initPromiseRef.current = window.hydraSdk.initialize({
        env,
        sessionId,
        apiKey,
        apiSecret,
      });
    }

    let mounted = true;

    initPromiseRef.current
      .then((instance) => {
        sdkRef.current = instance;
        if (mounted) setReady(true);
      })
      .catch((err) => {
        initPromiseRef.current = null;
        if (mounted) setError(err);
      });

    return () => { mounted = false; };
  }, [env, sessionId, apiKey, apiSecret]);

  const getFingerprint = useCallback(
    async (additionalParams) => {
      if (!sdkRef.current) throw new Error("SDK not initialized yet");
      return sdkRef.current.get(additionalParams);
    },
    []
  );

  return (
    <FingerprintContext.Provider value={{ getFingerprint, ready, error }}>
      {children}
    </FingerprintContext.Provider>
  );
}

export function useFingerprint() {
  const ctx = useContext(FingerprintContext);
  if (!ctx) throw new Error("useFingerprint must be used within FingerprintProvider");
  return ctx;
}
```

---

## Step 3: Wrap Your Application

```javascript
// app/page.js
import { FingerprintProvider } from "@/providers/fingerprint-provider";
import MyComponent from "@/components/my-component";

export default function Page() {
  return (
    <FingerprintProvider
      env="PROD"
      sessionId="unique-session-id"
      apiKey="your-api-key"
      apiSecret="your-api-secret"
    >
      <MyComponent />
    </FingerprintProvider>
  );
}
```

---

## Step 4: Use Anywhere in Your App

```javascript
// components/my-component.jsx
"use client";

import { useFingerprint } from "@/providers/fingerprint-provider";

export default function MyComponent() {
  const { getFingerprint, ready, error } = useFingerprint();

  if (error) return <p>SDK failed to load.</p>;
  if (!ready) return <p>Loading...</p>;

  const handleClick = async () => {
    try {
      const result = await getFingerprint({
        userId: "abc123",
        loginIdentifier: "abcd-12erf-rtyv3-pfrtec",
      });

      console.log("Fingerprint:", result);
    } catch (err) {
      console.error("Fingerprint collection failed:", err.message);
    }
  };

  return <button onClick={handleClick}>Get Fingerprint</button>;
}
```

---

# Minimum Requirements

| Parameter      | Minimum Requirement                                        |
| -------------- | ---------------------------------------------------------- |
| **Browser**    | Chrome 70+, Firefox 65+, Safari 12.1+, Edge 80+, Opera 70+ |
| **JavaScript** | ES6 or later                                               |
| **Network**    | Stable internet connection                                 |

---

# Initialization Parameters

| Parameter   | Type   | Required | Description                        |
| ----------- | ------ | -------- | ---------------------------------- |
| `env`       | string | Yes      | Environment: `'PROD'` or `'STAGE'` |
| `sessionId` | string | Yes      | Unique session identifier          |
| `apiKey`    | string | Yes      | API key provided by Sign3          |
| `apiSecret` | string | Yes      | API secret provided by Sign3       |

---

# Fetching Browser Data

After successful initialization:

```javascript
result.get({
  phoneNumber: '967****766',
  userId: 'user******'
}).then((response) => {
  console.log(response);
}).catch((error) => {
  console.log('Get error:', error.message);
});
```

### Optional Parameters for `get()`

| Parameter     | Type   | Required | Description                                       |
| ------------- | ------ | -------- | ------------------------------------------------- |
| `phoneNumber` | string | No       | Used for triangulation with Digital Footprint API |
| `userId`      | string | No       | User identifier                                   |

> The `get()` method accepts a flexible dictionary. You may also send additional fields such as email, loginIdentifier, etc.

---

# Error Handling

Errors during initialization or fingerprint retrieval are handled using `.catch()`.

```javascript
sdk.initialize({ ... })
  .then((result) => {
    return result.get({ phoneNumber: '967****766' });
  })
  .then((response) => console.log(response))
  .catch((error) => {
    console.log('Error:', error.message);
  });
```

---

# Browser Fingerprint Response Structure

Example response:

```json
{
  "requestId": "f60f48bf-890f-4274-a050-2cfa9d3ad33d",
  "newDevice": false,
  "fingerprint": "018fff6b-d4a3-4c67-ba53-b8bdce2cac3d",
  "sessionId": "17dnu81f-890f-adff-a050-2cfa9d3ad33d",
  "createdAt": 1738141661,
  "riskScore": "Medium",
  "firstSeenDays": 10,
  "ipIntelligence": {
    "city": "New Delhi",
    "region": "National Capital Territory of Delhi",
    "country": "IN",
    "latitude": 28.60000038,
    "longitude": 77.19999695,
    "isVPN": false,
    "isTor": false,
    "isProxy": false,
    "ip": "14.140.38.186"
  },
  "browserDetections": {
    "isIncognito": false,
    "isDevToolsOpen": true,
    "isBotDetected": false,
    "isAdBlockerEnabled": false,
    "isUserAgentSpoofed": true
  }
}
```

---

## Field Descriptions

### General Information

* **requestId** *(string)* — Unique identifier for the fingerprint request
* **newDevice** *(boolean)* — Whether the device is newly detected
* **fingerprint** *(string)* — Unique identifier for the browser/device
* **sessionId** *(string)* — Session identifier
* **createdAt** *(integer)* — Unix timestamp of fingerprint generation
* **riskScore** *(string)* — Risk level (Low / Medium / High), derived from device and IP intelligence
* **firstSeenDays** *(integer)* — Days since the device was first seen

---

### IP Intelligence

* **city** — IP location city
* **region** — Administrative region
* **country** — ISO 3166-1 alpha-2 country code
* **latitude / longitude** — Approximate geolocation
* **isVPN** — VPN detected
* **isTor** — Tor network detected
* **isProxy** — Proxy detected
* **ip** — User IP address

---

### Browser Detections

* **isIncognito** — Private browsing detected
* **isDevToolsOpen** — Developer tools open
* **isBotDetected** — Bot behavior detected
* **isAdBlockerEnabled** — Ad blocker active
* **isUserAgentSpoofed** — User agent tampering detected

> All signals are collected in real time when `get()` is called.

---

# Change Log

## Version 1.0.0

* Initial release of Sign3 Web SDK
* Added `initialize()` and `get()` documentation
* Included browser fingerprint response structure
* Added error handling examples

---

If you would like, I can also provide:

* A **more enterprise-polished public documentation version**
* A **shortened Quick Start version**
* Or a **developer portal–ready Markdown version** optimized for tools like GitBook / Docusaurus**
