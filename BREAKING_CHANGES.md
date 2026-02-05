# Breaking Changes

This document lists some breaking changes introduced

## Node.js Version Requirement

Node.js versions with `require(esm)` support are now required. This is because some updated dependencies are now ESM-only. 

Minimum supported versions:
- Node.js ^20.19.0
- Node.js ^22.12.0
- Node.js >= 23.0.0

## Express 5.x Required

The minimum Express version has been updated from 4.x to 5.x.

## `httpAgent` Configuration Option Removed

The `httpAgent` configuration option has been removed. Use `customFetch` instead for proxy configuration or custom HTTP behavior.

**Old code:**
```js
const { HttpsProxyAgent } = require('https-proxy-agent');

app.use(auth({
  httpAgent: new HttpsProxyAgent('http://proxy:8080'),
  // ... other options
}));
```

**New code:**
```js
const { ProxyAgent, fetch: undiciFetch } = require('undici');
const dispatcher = new ProxyAgent('http://proxy.example.com:8080');

app.use(auth({
  customFetch: (url, options) => undiciFetch(url, { ...options, dispatcher }),
  // ... other options
}));
```

## Error Message Changes

Error messages from the underlying OpenID Connect library may differ from previous versions.

## Token Type Response Format

The `token_type` in responses now returns lowercased values (e.g., `bearer` instead of `Bearer`).


## `allowInsecureRequests` Required for HTTP Issuers

If you're running against a local HTTP issuer for development, you must now explicitly enable insecure requests:

```js
app.use(auth({
  allowInsecureRequests: true, // Only for development!
  // ... other options
}));
```

Or via environment variable:
```bash
ALLOW_INSECURE_REQUESTS=true
```

**Do not enable this in production environments.**

## `clientAssertionSigningKey` Changes

### Removed Key Types

The following key types are no longer supported for `clientAssertionSigningKey`:

- **Removed:** `KeyObject`, `Buffer`
- **Supported:** PKCS#8 PEM string, JWK object, or `CryptoKey`

If you were using a `KeyObject` or `Buffer`, convert to a PEM string or JWK.

## `clientAssertionSigningAlg` Changes

### Algorithm Support Changes

The supported algorithms for `clientAssertionSigningAlg` have changed:

- **Removed:** `ES256K`, `EdDSA`
- **Added:** `Ed25519` (as a replacement for `EdDSA`)

### Now Required for PEM Keys and JWKs Without `alg`

`clientAssertionSigningAlg` is now required when `clientAssertionSigningKey` is:
- A PKCS#8 PEM string
- A JWK object without an `alg` property

**Old code (worked without specifying alg):**
```js
app.use(auth({
  clientAssertionSigningKey: '-----BEGIN PRIVATE KEY-----\n...',
  // clientAssertionSigningAlg was optional
}));
```

**New code:**
```js
app.use(auth({
  clientAssertionSigningKey: '-----BEGIN PRIVATE KEY-----\n...',
  clientAssertionSigningAlg: 'RS256', // Now required for PEM keys
}));
```

If using a JWK with an `alg` property, `clientAssertionSigningAlg` remains optional.
