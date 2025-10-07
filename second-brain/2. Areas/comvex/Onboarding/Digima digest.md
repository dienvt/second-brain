---
tags:
  - comvex-onboarding
---
# Models

```mermaid
classDiagram

direction TB

class User {

id

email

first_name

last_name

...

}

  

class Contact {

id

first_name

last_name

email

company_id

...

}

  

class Email {

id

subject

body_html

author_id

...

}

  

class OutgoingCall {

id

user_id

contact_id

to_number

...

}

  

class Notification {

id

type

owner_id

...

}

  

class NotificationSubscription {

id

subscriber_id

resource_type

resource_id

...

}

  

class Company {

id

name

...

}

  

class Group {

id

name

...

}

  

User "1" --o "many" Contact : manages

User "1" --o "many" Email : sends

User "1" --o "many" OutgoingCall : makes

User "1" --o "many" Notification : receives

User "1" --o "many" NotificationSubscription : subscribes

Contact "many" --o "1" Company : works_at

Contact "many" --o "many" Email : receives

Contact "many" --o "many" OutgoingCall : called

Contact "many" --o "many" Group : member_of

Company "1" --o "many" Contact : employs

Email "many" --o "1" User : author

Email "many" --o "many" Contact : recipients

OutgoingCall "many" --o "1" User : by

OutgoingCall "many" --o "1" Contact : to

Notification "many" --o "1" User : owner

NotificationSubscription "many" --o "1" User : subscriber
```
## Models relation

- User manages Contact, sends Email, makes OutgoingCall, receives Notification, and subscribes to NotificationSubscription.

- Contact works at Company, receives Email, is called via OutgoingCall, and can be a member of Group.

- Company employs Contact.

- Email is authored by User and sent to Contact.

- OutgoingCall is made by User to Contact.

- Notification and NotificationSubscription are linked to User.






```mermaid
graph TD

A[User & Account Management]

B[Contact & Company Management]

C[Communication Channels]

D[Workflow & Automation]

E[Import/Export]

F[Admin & Internal Tools]

G[Integration & API]

H[Billing & Invoicing]

I[Security & Access Control]

J[Other Features]

  

C1[Email]

C2[SMS]

C3[Phone/Call]

C4[Notifications]

G1[OAuth2]

G2[External Integrations]

G3[URL Shortener]

G4[Web Tracking & Forms]

I1[RBAC]

I2[Device Management]

I3[Password Reset]

J1[Activity Logging]

J2[Feature Flags]

J3[Exclusion Lists]

J4[Reminders & Alerts]

J5[Localization]

  

C --> C1

C --> C2

C --> C3

C --> C4

G --> G1

G --> G2

G --> G3

G --> G4

I --> I1

I --> I2

I --> I3

J --> J1

J --> J2

J --> J3

J --> J4

J --> J5

  

A -->|Manages| B

A -->|Uses| C

B -->|Uses| C

D -->|Automates| C

D -->|Automates| E

F -->|Controls| A

F -->|Controls| B

F -->|Controls| D

G -->|Connects| C

G -->|Connects| F

H -->|Invoices| A

H -->|Invoices| B

I -->|Secures| A

I -->|Secures| B

I -->|Secures| F

J -->|Supports| All

  

subgraph Features

A

C

D

E

F

G

H

I

J

end
```



# Homepage
My Account / Account dashboard: 
- What is the difference?

We have contact/user ??? what is differences?


Contact có thể tạo? như mà properties đâu ??? real estate need an apartment right.

**IP Restriction** ? how?? and what restricted??

Why upload has to have UID?

Usser submit form then they become a lead then we convert them to contact(customer)
Create form will have the api key for this
fter create a Form, digima return the form authcode, then they using rest to call to digima to create lead.

Contact form 7
Emails to leads
- > create email add, user using email add to set as form.
- sendgrid receive email, then sendgrid send to digima as a weebhook
Portal: search house -> s connect with digima, then when customer connect to summo, digima will receive this.

## UIPAth
RPA (Robotic Process Automation) tools are software solutions that automate repetitive, rule-based tasks typically performed by humans in business processes . replace by `browser-worker`  to craw info mation.
Login and grab information from summoo

## webtracking
webtracking using for steel cookie of user then tracking them name and email only.





# Bug:
- Error when click drop down when upload file fails
- Back from upload field don't work correctly (it always back to the import page)
* layout broken when Page have headline: https://app.digima.local:8080/settings/profile/inbox/connected
* Connect imbox really using credentials.
* https://app.digima.local:8080/import/9/successes: Show 17 successes ??? should be Inprogress.

Local: in my machine
STG: Corellion, link with bookmark 

Boosted == started
Customer subscribe


**I can assigned to any one?**


# Features
- Contact
	- Create single contact
	- Import list contacts
	- Update contact fields
	- Export contact **But why export in zip files**
	- Auto update contact to "Approached" when set activities
	- Set meeting with contact
- Contact group:
	- list of contact, can be dynamic or static
	- 
- Company:
	- auto sync contact have email match with company 
	- Import/export company

* Activites:
	* Base on both members and company: created day, sms send, ...

* Campaign:


* Mail:
	* Send single receive mails
	* Send bulk emails
	* Schedule
* Web tracking: ??? using script attach to user web
* Workflows: 
	* Instead of create campaign and manually approach contacts, we have workflows
	* Only can set sms, email in flow
	* 
* 
* File: all attended email 

* Connect with anpadL
	* after change will sync to andpad-order vise versa
	* Like ready to build house this data will send to andpad for create ordder.


# Data

1. Saas so each data will save in to  dgm_account_\<\<clientID\>\>
2. Invoice system didn't have

