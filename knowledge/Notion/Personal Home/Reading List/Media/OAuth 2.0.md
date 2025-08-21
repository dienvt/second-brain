---
Status: Finished
Author:
  - Ryan Boyd
Link: https://www.amazon.com/Getting-Started-OAuth-Authorization-Authentication/dp/1449311601
Score /5: ⭐️⭐️⭐️
Type: Book
"\bGenre":
  - IT
Start date: 2023-03-01
---
# Introduce

Giới thiệu về OAuth 2.0.

Các mô hình và mô tả chi tiết cách implement

- Web server
- Client
- Password
- Native app
- …

  

# Cảm nhận

Sách đọc ok, giới thiệu chi tiết

  

## Chapter 2. Server-Side Web Application Flow

`state` - A unique value used by your application in order to prevent  
cross-site request forgery (CSRF) attacks on your implementation.  
The value should be a random unique string for this particular  
request, unguessable and kept secret in the client (perhaps in a  
server-side session)  

  

## Chapter 3. Client-Side Web Applications Flow

Same as server-side but access-token will be sent in URI. For example

“[http://photoviewer.saasyapp.com/pv/oauth2callback.html#access_token=ya29.AHES6ZSzX&token_type=Bearer&expires_in=3600”](http://photoviewer.saasyapp.com/pv/oauth2callback.html#access_token=ya29.AHES6ZSzX&token_type=Bearer&expires_in=3600%E2%80%9D)

  

## Chapter 4: Resource Owner password

Request:

`client_id` and `client_secret` & `grant_type="client_credential"`

```Bash
curl -d "grant_type=password" \
-d "client_id=3MVG9QDx8IKCsXTFM0o9aE3KfEwsZLvRt" \
-d "client_secret=4826278391389087694" \
-d "username=ryan%40ryguy.com" \
-d "password=_userspassword__userssecuritytoken_" \
https://login.salesforce.com/services/oauth2/token
```

## Chapter 5: Client Credentials flow

“There is another representative case for the Client Credentials  
flow—when a resource owner has granted an application access to their  
resources out of band, without using a typical OAuth flow. Google provides  
a concrete use case in the Google Apps Marketplace. When an application is listed  
on the Marketplace, vendors get credentials that represent their  
application and also register the scopes of data they need access to. When  
the application is later installed by an organization’s IT administrator,  
Google asks the administrator whether it’s OK to grant the application  
access to his organization’s data. When access is approved, Google stores  
that organization “Acme Corp” has granted access to “Google Calendar and  
Google Contacts” for application “Task Manager Pro.” Google does not issue  
any tokens to the application. When the application tries to access data  
in the future, Google simply looks up whether the application is allowed  
access to data for the particular organization.”  
Like mod post bài và admin có thể sửa  

`client_id` and `client_secret` & `grant_type="client_credential"`

  

## Chapter 6: Getting Access to User Data from Mobile Apps

  

  

  

  

“The Authorization header is the preferred  
mechanism becauseThe header is rarely logged by proxy servers and web server  
access logs.The header is almost never cached.The header doesn’t get stored in the browser cache when making  
requests from the client.”  

Excerpt From  
Getting Started with OAuth 2.0  
Ryan Boyd  
This material may be protected by copyright.