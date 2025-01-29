## web-sdk-integration-guide

The Sign3 WEB SDK is an javascript based fraud prevention toolkit designed to assess browser security, detecting potential risks such as Proxy, VPN connections, USER AGENT spoofing, TOR connection and much more. Providing insights into the browser's safety, it enhances security measures against fraudulent activities and ensures a robust protection system.

# Adding Sign3 WEB SDK to Your Project
1. Download the JavaScript Agent from cdn link provided to you and include it in your code base. You can also import dynamically upon specific actions (e.g. clicking on Login, Payment, and Registration buttons, before calling the API).
2. Call the `sign3.initialize(config)` to initialize javascript client for signal collection.
3. `sign3.initialize(config)` will return a promise that will resolve to an Object containting get method.
4. Call `result.get().then(response)` this will again return a promise which will resolve to response object which contains browser information or will reject giving error if something went wrong.
5. From response object fields like device ID, request Id, browserIntelligenceData or ipIntelligenceData as per your use case.

# Bundel Into your code
  lets assume you have sdk code inside a file names sign3-web-sdk.js
  ```
import sdk from './sign3-web-sdk.js';

sdk.initialize({
  env: env,
  sessionId: sessionId,
  apiKey: apiKey, // client -id - salt
  apiSecret: apiSecret, // sectret - password
}).then((result) => {
  result.get().then((response) => {
    console.log(response);
  }, (error) => {
    console.log('get errr', error.message);
  });
}, (error) => {
  console.log('err: ', error.message);
});
```
