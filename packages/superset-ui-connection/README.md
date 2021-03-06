## @superset-ui/connection

[![Version](https://img.shields.io/npm/v/@superset-ui/connection.svg?style=flat)](https://img.shields.io/npm/v/@superset-ui/connection.svg?style=flat-square)
[![David (path)](https://img.shields.io/david/apache-superset/superset-ui.svg?path=packages%2Fsuperset-ui-connection&style=flat-square)](https://david-dm.org/apache-superset/superset-ui?path=packages/superset-ui-connection)

Connection modules for Superset:

- `SupersetClient` requests and authentication
- (future) `i18n` locales and translation

### SupersetClient

The `SupersetClient` handles all client-side requests to the Superset backend. It can be configured
for use within the Superset application, or used to issue `CORS` requests in other applications. At
a high-level it supports:

- `CSRF` token authentication
  - a token may be passed at configuration time, else the client will handle fetching and passing
    the token in all subsequent requests.
  - queues requests in the case that another request is made before the token is received.
  - it checks for a token before every request, and will fail if no token was received or if it has
    expired. In either case the user should be directed to re-authenticate.
- supports `GET` and `POST` requests (no `PUT` or `DELETE`)
- timeouts
- query aborts through the `AbortController` API

#### Example usage

```javascript
// appSetup.js
import { SupersetClient } from `@superset-ui/connection`;
// or import SupersetClient from `@superset-ui/connection/lib|esm/SupersetClient`;

SupersetClient.configure(...clientConfig);
SupersetClient.init(); // CSRF auth, can also chain `.configure().init();

// anotherFile.js
import { SupersetClient } from `@superset-ui/connection`;

SupersetClient.post(...requestConfig)
  .then(({ request, json }) => ...)
  .catch((error) => ...);
```

#### API

##### Client Configuration

The following flags can be passed in the client config call
`SupersetClient.configure(...clientConfig);`

- `protocol = 'http:'`
- `host`
- `headers`
- `credentials = 'same-origin'` (set to `include` for non-Superset apps)
- `mode = 'same-origin'` (set to `cors` for non-Superset apps)
- `timeout`
- `csrfToken` you can configure the client with a CSRF token at configuration time, else the client
  will attempt to fetch this before any other requests are issued

##### Per-request Configuration

The following flags can be passed on a per-request call `SupersetClient.get/post(...requestConfig);`

- `url` or `endpoint`
- `headers`
- `body`
- `timeout`
- `signal` (for aborting, from `const { signal } = (new AbortController())`)
- for `POST` requests
  - `postPayload` (key values are added to a `new FormData()`)
  - `stringify` whether to call `JSON.stringify` on `postPayload` values

##### Request aborting

Per-request aborting is implemented through the `AbortController` API:

```javascript
import { SupersetClient } from '@superset-ui/connection';
import AbortController from 'abortcontroller-polyfill';

const controller = new AbortController();
const { signal } = controller;

SupersetClient.get({ ..., signal }).then(...).catch(...);

if (IWantToCancelForSomeReason) {
  signal.abort(); // Promise is rejected, request `catch` is invoked
}
```

### Development

`@data-ui/build-config` is used to manage the build configuration for this package including babel
builds, jest testing, eslint, and prettier.
