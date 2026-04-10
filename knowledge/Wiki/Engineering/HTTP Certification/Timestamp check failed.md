---
title: "Solution"
date: 2026-01-14
tags:
  - engineering
  - http
  - networking
---

Full error:

```Plain
verify otp exception: javax.net.ssl.SSLHandshakeException: sun.security.validator.ValidatorException: PKIX path validation failed: java.security.cert.CertPathValidatorException: timestamp check failed
```

Theo như System thì đã gia hạn cert nhưng source java không detect cert mới.

Thử check với `openssl`  
không có có  
`-servername`

```Plain
> openssl s_client -connect security.vng.com.vn:443  2>/dev/null | openssl x509 -noout -dates

notBefore=Feb 26 00:00:00 2021 GMT
notAfter=Feb 12 23:59:59 2022 GMT
```

có `-servername`

```Plain
> echo -n | openssl s_client -connect security.vng.com.vn:443 -servername security.vng.com.vn 2>/dev/null | openssl x509 -noout -dates

notBefore=Jan 18 00:00:00 2022 GMT
notAfter=Jan 18 23:59:59 2023 GMT
```

- --> có thể SRE đã không update toàn bộ cert

đã enable TLS Server Name Indication (SNI) extension nhưng vẫn không fix được lỗi

```Plain
-Djsse.enableSNIExtension=true
```

# Solution

Change $JAVA_HOME

```Plain
JAVA_HOME="/zserver/jdk1.8.0_181"
```

# Investigation

1. Enable SSL debug

```Plain
-Djavax.net.debug=all
```

1. Check trust store log

```Plain
ICT|TrustStoreManager.java:112|trustStore is: /Library/Java/JavaVirtualMachines/jdk1.8.0_271.jdk/Contents/Home/jre/lib/security/cacerts
```

`certificate`, `vng.com.vn`  
3. I think the problem happen when CA in jre not correct