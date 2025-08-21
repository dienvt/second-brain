# JWT
```go
type RegisteredClaims struct {  
    // the `iss` (Issuer) claim. See https://datatracker.ietf.org/doc/html/rfc7519#section-4.1.1  
    Issuer string `json:"iss,omitempty"`  
  
    // the `sub` (Subject) claim. See https://datatracker.ietf.org/doc/html/rfc7519#section-4.1.2  
    Subject string `json:"sub,omitempty"`  
  
    // the `aud` (Audience) claim. See https://datatracker.ietf.org/doc/html/rfc7519#section-4.1.3  
    Audience ClaimStrings `json:"aud,omitempty"`  
  
    // the `exp` (Expiration Time) claim. See https://datatracker.ietf.org/doc/html/rfc7519#section-4.1.4  
    ExpiresAt *NumericDate `json:"exp,omitempty"`  
  
    // the `nbf` (Not Before) claim. See https://datatracker.ietf.org/doc/html/rfc7519#section-4.1.5  
    NotBefore *NumericDate `json:"nbf,omitempty"`  
  
    // the `iat` (Issued At) claim. See https://datatracker.ietf.org/doc/html/rfc7519#section-4.1.6  
    IssuedAt *NumericDate `json:"iat,omitempty"`  
  
    // the `jti` (JWT ID) claim. See https://datatracker.ietf.org/doc/html/rfc7519#section-4.1.7  
    ID string `json:"jti,omitempty"`  
}
```
# Claim Mapping
auth-api
```go
// AuthToken defines access token instance structure to be passed around in contexttype AuthToken struct {  
    Id string `json:"id"`  
    // Deprecated: Use UserId instead to get AccountId from users resource    AccountId *uint   `json:"account_id"`  
    UserId    *string `json:"user_id"`  
    ClientId  string  `json:"client_id"`  
    GrantType string  `json:"grant_type"`  
    // Depend on the grantType
    // GrantTypeClientCredentials: ClientId
    // GrantTypePassword or GrantTypeRefresh: UserID
    Subject   string  `json:"subject"`  
    Scopes    string  `json:"scope"`  
}
```
