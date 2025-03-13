# Sign3 Web SDK Integration Guide

The Sign3 WEB SDK is a JavaScript-based fraud prevention toolkit designed to assess browser security, detecting potential risks such as Proxy, VPN connections, USER AGENT spoofing, TOR connections, and more. Providing insights into the browser's safety, it enhances security measures against fraudulent activities and ensures robust protection.

---

## Adding Sign3 WEB SDK to Your Project

1. Download the JavaScript Agent from the CDN link and include it in your codebase with file name `sign3-web-sdk.js` (provided seperately).
2. Call `sign3.initialize(config)` to initialize the JavaScript client for signal collection.
3. `sign3.initialize(config)` returns a promise that resolves to an object containing the `get` method.
4. After initializing, you can get browser intelligence upon specific actions (e.g., clicking on Login, Payment, and Registration buttons before calling the API).
5. Call `result.get().then(response)`, which returns a promise resolving to a response object containing browser information or rejecting with an error if something went wrong.
6. The response object contains fields like fingerprint, request ID, browser intelligence data, or IP intelligence data.

---

## Bundling Into Your Code

Assuming you have the SDK inside a file named `sign3-web-sdk.js`:

### Initialization

To use the SDK, initialize it with the required parameters.

```javascript
import sdk from './sign3-web-sdk.js';

sdk.initialize({
  env: 'PROD', // required: The environment ('PROD', 'STAGE').
  sessionId: 'your-unique-session-id', // required: A unique session identifier to track the user session.
  apiKey: 'your-api-key', // required: API key used for authentication (shared separately by Sign3).
  apiSecret: 'your-api-secret', // required: Secret key used for authentication (shared separately by Sign3).
}).then((result) => {
  console.log('Initialization successful');
}, (error) => {
  console.log('Initialization error:', error.message);
});
```

### Parameters

| Parameter   | Type     | Required | Description                                             |
| ----------- | -------- | -------- | ------------------------------------------------------- |
| `env`       | `string` | Yes      | Specifies the environment ('PROD', 'STAGE').     |
| `sessionId` | `string` | Yes      | A unique session identifier to track the user session.  |
| `apiKey`    | `string` | Yes      | API key used for authentication (shared separately).    |
| `apiSecret` | `string` | Yes      | Secret key used for authentication (shared separately). |

---

## Fetching Browser Data

Once initialized successfully, use the `get` method to retrieve browser information.

```javascript
result.get({
  phoneNumber: '967****766',
  userId: 'user******'
}).then((response) => {
  console.log(response);
}, (error) => {
  console.log('Get error:', error.message);
});
```

### Parameters for `get` Method

| Parameter     | Type     | Required | Description                                                                                 |
| ------------- | -------- | -------- | ------------------------------------------------------------------------------------------- |
| `phoneNumber` | `string` | No      | The phone number of the user to triangulate this data later with the Digital Footprint API. |
| `userId` | `string` | No      | The user ID of the user. |

**Note:** The `get` call accepts a dictionary where you can also send email, userId, etc.

---

## Error Handling

If there is any error during initialization or a `get` call, it is handled using `.then()` and `.catch()` blocks.

### Example Error Handling

```javascript
sdk.initialize({ ... }).then((result) => {
  result.get({ phoneNumber: '967****766' })
    .then((response) => console.log(response))
    .catch((error) => console.log('Get error:', error.message));
}).catch((error) => {
  console.log('Initialization error:', error.message);
});
```

---

## Browser Fingerprint Response Structure

This section provides a detailed explanation of the fields included in the browser fingerprint response JSON.

### JSON Structure

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

### Field Descriptions

### General Information
- **requestId** *(string)*: A unique identifier for this fingerprint request.
- **newDevice** *(boolean)*: Indicates whether this is a newly detected device (`true`) or a previously seen device (`false`).
- **fingerprint** *(string)*: A unique identifier generated for the browser and device combination based on various fingerprinting techniques.
- **sessionId** *(string)*: A unique session identifier for tracking user activity during a session.
- **createdAt** *(integer)*: Timestamp (Unix format) representing when this fingerprint was generated.
- **riskScore** *(string)*: Indicates the risk level of the detected fingerprint (e.g., "Low", "Medium", "High") triangulated with ip data and other malicious signals 
- **firstSeenDays** *(integer)*: The number of days since this device was first seen.

### IP Intelligence
- **city** *(string)*: The city where the detected IP is located.
- **region** *(string)*: The administrative region associated with the IP.
- **country** *(string)*: The country code (ISO 3166-1 alpha-2) where the IP originates.
- **latitude** *(float)*: Approximate latitude coordinate of the detected IP.
- **longitude** *(float)*: Approximate longitude coordinate of the detected IP.
- **isVPN** *(boolean)*: Indicates whether the detected IP is associated with a VPN service.
- **isTor** *(boolean)*: Indicates whether the detected IP is part of the Tor network.
- **isProxy** *(boolean)*: Indicates whether the detected IP is using a proxy server.
- **ip** *(string)*: The IP address of the detected user.

### Browser Detections
- **isIncognito** *(boolean)*: Indicates whether the browser is in incognito or private browsing mode.
- **isDevToolsOpen** *(boolean)*: Detects if the browser's developer tools are open.
- **isBotDetected** *(boolean)*: Identifies if the user is a bot or an automated script.
- **isAdBlockerEnabled** *(boolean)*: Determines if an ad blocker is active in the browser.
- **isUserAgentSpoofed** *(boolean)*: Detects if the user agent string has been modified to disguise the browser's identity.

**Note:** All signals are collected in real-time when the `get` call is made.

---

## Change Log

### Version 1.0.0

- Initial release of Sign3 Web SDK integration guide.
- Added `initialize` and `get` method details.
- Included browser fingerprint response structure and field explanations.
- Provided error handling examples.

