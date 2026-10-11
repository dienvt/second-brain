---
tags:
  - comvex
---
# references
https://comvex.atlassian.net/wiki/spaces/SE/pages/2421227545/Meeting+Schedules#Sprint-Review

# Details Agenda
## Rule
- Only one person host
- There was a template that using to represent: For example [Sprint 141](https://comvex.atlassian.net/wiki/spaces/SE/pages/2667184129/Sprint+141)
## Preparation
1. Switch to [PdM Sprint Management](https://comvex.myjetbrains.com/youtrack/agiles/107-133/current?query=%23Platform%20) - Where all story was listed.
	- Add new Swim lane and named it `[Effort Spent Total] This card summarize all effort spent on Sprint 137 that cards are moved to next Sprint`. Refs: DGM2-23389
		- Type Engineer Story
		- Content: `## Cards under these efforts`
		- 
	- Put our effor which we spent to that card
	- Spend 5 minutes for everyone put remaining effort to the comments section
## In the Meetings
### 1. Review and present each cards:

Take Screen shot

Switch to [PdM Sprint Management](https://comvex.myjetbrains.com/youtrack/agiles/107-133/current?query=%23Platform%20) - Where all story was listed.
* **Finish Cards**: 
	* Add tag "To be deployed" or "QA Validation"
	* Note to the [Repositories to be deployed](https://comvex.atlassian.net/wiki/spaces/SE/pages/2667184129/Sprint+141#Repositories-to-be-deployed)
* **Not Finish Cards**:
	* Move card to the next sprint
	* Negotiate the story point

- click the refresh button in top left burn down chart.
- Then check management board (Digima-management)
- Check Sprint Goals
- Check the completion
- Update the BE-Release Task Board

- Clone the new card from the STG cards
* Why don't we just let it there and finish the sprint?
- When deploy we need check the infra change or it will be long night
- Promote??? Digima-infra.

### 3. Release tasks

**How we test release tasks**



## 4. Review Grantt Chart
Moi truong khong toi uwu.


## 5. Update Bug Management Sheet
https://docs.google.com/spreadsheets/d/1cQNbpHZeAJDXQweu7Mgje0QfO5JJdJNuA1OFnsKIT3E/edit?gid=2018229120#gid=2018229120

```
Platformチーム記載しました
```
## 6. Confirm release list

https://comvex.slack.com/archives/C02K1GDDNMP
[@Dien.Vo](https://comvex.slack.com/team/U091Z4WUQ48) (cc: [@eng-platforms](https://comvex.slack.com/admin/user_groups)  and maybe [@eng-communications](https://comvex.slack.com/admin/user_groups))  
As part of our Postmortem meeting last week regarding lack of communications to PdM for what cards will be released at the end of the Sprint.  
Reply on the bug sheet
```
Platform記載しました
```
As discussed, could you guys report here your Sprint Review results so PdM will be aware what cards will be released tomorrow's deployment.

```
[@Ryuta Ichiyanagi / 一柳 竜太](https://comvex.slack.com/team/U064CP33YN4)  
cc: [@Cliff Richard Anfone / アンフォネ クリフリチャード](https://comvex.slack.com/team/UCPRJC5C4)  
Platform team will release those service  
• digima-backend-file-api  
[DGM2-31251](https://comvex.myjetbrains.com/youtrack/issue/DGM2-31251) [Sentry] infra.cloudinary.UploadPrivateFile  
  
Those card are under testing. I'll release if the testing done today  
• digima-backend-adminweb-bff  
[DGM2-31349](https://comvex.myjetbrains.com/youtrack/issue/DGM2-31349) : Using permission policies to hide the reality capture section  
• digima-adminweb-app  
[DGM2-31349](https://comvex.myjetbrains.com/youtrack/issue/DGM2-31349) : Using permission policies to hide the reality capture section  
• digima-backend-auth-api  
[DGM2-30805](https://comvex.myjetbrains.com/youtrack/issue/DGM2-30805) 本番環境でログイン時に5回パスワード入力を失敗したさいにアカウントロックされない / Accounts are not locked after five failed password attempts during login in the production environment  
• digima-web-app, digima-secure-web-app  
[DGM2-30805](https://comvex.myjetbrains.com/youtrack/issue/DGM2-30805) 本番環境でログイン時に5回パスワード入力を失敗したさいにアカウントロックされない / Accounts are not locked after five failed password attempts during login in the production environment
```