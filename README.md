# Sign3 Web SDK Integration Guide

The **Sign3 Web SDK** is a JavaScript-based fraud prevention toolkit designed to assess browser security and detect potential risks such as proxy/VPN usage, user agent spoofing, TOR connections, and more.

By providing real-time browser intelligence, it enhances fraud detection capabilities and strengthens protection against malicious activity.

---

## Integration Overview

The Sign3 Web SDK is loaded into the browser via a CDN `<script>` tag provided by Sign3 and is exposed on the global `window.hydraSdk`. The end-to-end flow is:

1. **Browser** — Load the SDK and call `window.hydraSdk.initialize(config)`. This returns a Promise that resolves to an SDK instance with a `get` method.
2. **Browser** — Call `sdk.get(params)` before critical user actions (e.g., login, payment, registration API calls). It resolves with `{ requestId }`.
3. **Browser → Your Backend** — Forward the `requestId` to your backend along with the action being performed.
4. **Your Backend → Sign3** — Exchange the `requestId` for the full intelligence response using the [Backend Hook](#backend-hook) API.

It is recommended to load and initialize the SDK once at application startup (or as early as possible in the page lifecycle) so all signals are captured correctly.

> The actual SDK URL will be provided separately by Sign3. The URL used in the examples below (`https://cdn.example.com/sdk/fingerprint.min.js`) is for demonstration purposes only.

Two implementation examples are provided below:

* [Vanilla JavaScript Integration](#vanilla-javascript-integration) — for plain HTML/JS pages.
* [React / Next.js — Provider Approach](#react--nextjs--provider-approach) — for component-based apps.

---

# Vanilla JavaScript Integration

Load the SDK using a `<script>` tag and use the global `window.hydraSdk`:

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
        const { requestId } = await sdk.get({
          userId: "abc123",
          loginIdentifier: "abcd-12erf-rtyv3-pfrtec",
        });

        // Forward `requestId` to your backend; your backend calls the
        // Sign3 Backend Hook to fetch the full intelligence response.
        console.log("requestId:", requestId);
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
      const { requestId } = await getFingerprint({
        userId: "abc123",
        loginIdentifier: "abcd-12erf-rtyv3-pfrtec",
      });

      // Forward `requestId` to your backend; your backend calls the
      // Sign3 Backend Hook to fetch the full intelligence response.
      console.log("requestId:", requestId);
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

After successful initialization, call `get()` to collect the fingerprint:

```javascript
sdk.get({
  phoneNumber: '967****766',
  userId: 'user******'
}).then(({ requestId }) => {
  // Send `requestId` to your backend along with the user action.
  console.log('requestId:', requestId);
}).catch((error) => {
  console.log('Get error:', error.message);
});
```

**SDK response — only `requestId` is returned in the browser:**

```json
{ "requestId": "f60f48bf-890f-4274-a050-2cfa9d3ad33d" }
```

Use this `requestId` from your backend to fetch the full intelligence response via the [Backend Hook](#backend-hook).

### Optional Parameters for `get()`

| Parameter     | Type   | Required | Description                                       |
| ------------- | ------ | -------- | ------------------------------------------------- |
| `phoneNumber` | string | No       | Used for triangulation with Digital Footprint API |
| `userId`      | string | No       | User identifier                                   |

> The `get()` method accepts a JavaScript object. You may also pass additional fields such as `email`, `loginIdentifier`, etc.

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

# Backend Hook

The browser SDK returns only a `requestId`. Your backend exchanges that `requestId` for the full intelligence response using the server-to-server endpoint below.

> ⚠️ This endpoint must be called **only from your backend**. It uses your `tenantSecret`, which must never be exposed to the browser.

## Request

```bash
curl --location 'https://intelligence-b.sign3.in/v1/request?requestId=<REQUEST_ID>' \
  --header 'x-channel: web' \
  --header 'Authorization: Basic <BASE64(tenantId:tenantSecret)>'
```

## Query Parameters

| Parameter   | Type   | Required | Description                                            |
| ----------- | ------ | -------- | ------------------------------------------------------ |
| `requestId` | string | Yes      | The `requestId` returned by `sdk.get()` in the browser |

## Headers

| Header          | Required | Description                                                                                       |
| --------------- | -------- | ------------------------------------------------------------------------------------------------- |
| `x-channel`     | Yes      | Must be set to `web`                                                                              |
| `Authorization` | Yes      | `Basic ` followed by Base64-encoded `tenantId:tenantSecret`. Credentials are provided by Sign3.   |

## Building the `Authorization` Header

```bash
# Example: tenantId=tenant-1, tenantSecret=secret-tenant-1
echo -n 'tenant-1:secret-tenant-1' | base64
# → dGVuYW50LTE6c2VjcmV0LXRlbmFudC0x
```

```javascript
// Node.js
const auth = Buffer.from(`${tenantId}:${tenantSecret}`).toString("base64");
// Use as: `Basic ${auth}`
```

## Response

The endpoint returns the intelligence payload described below.

---

# Intelligence Response Structure

Example payload:

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

## Version 1.0.2

* `sdk.get()` now resolves with `{ requestId }`; full intelligence response is fetched via the Backend Hook
* Added server-to-server `Backend Hook` API reference
* API responses are now encrypted in transit and decrypted in-browser by the SDK; `get()` shape unchanged
