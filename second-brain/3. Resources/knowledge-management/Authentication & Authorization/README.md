## OAuth2 and SSO Comparation
- OAuth2 **is not an SSO** solution, but it **can be a part of SSO when combined with OpenID Connect (OIDC)**.
- If you need authentication (who you are) + SSO, **use OIDC**.
- If you just need authorization (what you can access), **use OAuth2**.

## OAuth2

### Server-Side Web Application Flow

`state` - A unique value used by your application in order to prevent cross-site request forgery (CSRF) attacks on your implementation. The value should be a random unique string for this particular request, unguessable and kept secret in the client (perhaps in a server-side session)

Get Code → exchange for token → refresh token

```bash
$queryParams = array(
  'client_id' => '240195362.apps.googleusercontent.com',
  'redirect_uri' => (isset($_SERVER['HTTPS'])?'https://':'http://') .
                   $_SERVER['HTTP_HOST'] . $redirectUriPath,
  'scope' => '<https://www.googleapis.com/auth/tasks>',
  'response_type' => 'code',
  'state' => $_SESSION['state'],
  'approval_prompt' => 'force', // always request user consent
  'access_type' => 'offline' // obtain a refresh token
);
```

### Client-Side Web Applications Flow

Same as server-side but access-token will be sent in URI. For example [`http://photoviewer.saasyapp.com/pv/oauth2callback.html#access_token=ya29.AHES6ZSzX&token_type=Bearer&expires_in=3600`](http://photoviewer.saasyapp.com/pv/oauth2callback.html#access_token=ya29.AHES6ZSzX&token_type=Bearer&expires_in=3600%E2%80%9D)

### Resource Owner password

```bash
curl -d "grant_type=password" \\
-d "client_id=3MVG9QDx8IKCsXTFM0o9aE3KfEwsZLvRt" \\
-d "client_secret=4826278391389087694" \\
-d "username=ryan%40ryguy.com" \\
-d "password=_userspassword__userssecuritytoken_" \\
<https://login.salesforce.com/services/oauth2/token>
```

### Client Credentials flow

“There is another representative case for the Client Credentials flow—when a resource owner has granted an application access to their resources out of band, without using a typical OAuth flow. Google provides a concrete use case in the Google Apps Marketplace. When an application is listed on the Marketplace, vendors get credentials that represent their application and also register the scopes of data they need access to. When the application is later installed by an organization’s IT administrator, Google asks the administrator whether it’s OK to grant the application access to his organization’s data. When access is approved, Google stores that organization “Acme Corp” has granted access to “Google Calendar and Google Contacts” for application “Task Manager Pro.” Google does not issue any tokens to the application. When the application tries to access data in the future, Google simply looks up whether the application is allowed access to data for the particular organization.” Like mod post bài và admin có thể sửa

`client_id` and `client_secret` & `grant_type="client_credential"`

### Getting Access to User Data from Mobile Apps

## PCKE

Là một technique để tăng bảo mật khi kết nối dùng OAuth 2

**Trong Step Authorization Request:**

PCKE định nghĩa thêm 2 param: `code_challenge` và `code_challenge_method`

2 thông tin này được gắn vào request

**Trong Step Authorization Code Exchange:**

Client cầ truyền thêm thông tin `code_verifier` thông tin này là thông tin bí mật và client đã dùng để tính ra được `code_challenge` bằng `code_challenge_method` ở step trước. Nôm na có thể hiểu `code_challenge_method`(xxxx, `code_verifier`) = `code_challenge`

If the `code_challenge_method` is `plain`, then the authorization server needs only to check that the provided `code_verifier` matches the expected `code_challenge` string. If the method is `S256`, then the authorization server should take the provided `code_verifier`and transform it using the same hash method, then comparing it to the stored `code_challenge` string.

## Refrences

[](https://www.oauth.com/)[https://www.oauth.com](https://www.oauth.com)

[https://www.youtube.com/watch?v=ZV5yTm4pT8g](https://www.youtube.com/watch?v=ZV5yTm4pT8g)

# SSO

## references

[https://www.youtube.com/watch?v=fyTxwIa-1U0](https://www.youtube.com/watch?v=fyTxwIa-1U0)

[https://www.youtube.com/watch?v=O1cRJWYF-g4](https://www.youtube.com/watch?v=O1cRJWYF-g4)