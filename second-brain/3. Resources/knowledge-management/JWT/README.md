Structure

- Header
- Payload
- Signature

Token: **Base64(Header)**.Base64(Payload).(Header.Alg(base64UrlEncode(header)+"."+ base64UrlEncode(payload))

ex:

```json
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

## How to rework token

using JTI payload [https://datatracker.ietf.org/doc/html/rfc7519#section-4.1.7](https://datatracker.ietf.org/doc/html/rfc7519#section-4.1.7)

define unique id for `jti` payload (often UUID)

then add to blacklist until the ttl of the token reach