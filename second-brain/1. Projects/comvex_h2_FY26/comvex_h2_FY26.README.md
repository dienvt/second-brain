---
tags:
  - comvex_h2_FY26
aliases: "#comvex_h2_FY26"
---
%%Set the project deadline and its result description%%
- [ ] deadline 📅 2026-12-30
- key result
Em alignment lại goal của Huy giúp anh nhé:
- 

Cái plan cho Huy đến cuối năm sẽ support deliver những tính năng nào nhé:
-

Now : roald map H2 Sep
* [ ] New Billing feature Clear requirement Sep, Oct : **Anh Huy sẽ support phần này**
	* [ ] Missing requirement (there is no PRD yet)
	* [ ] Deal, mobile segment: Type? subscription? usage volume?...
	* [ ] Support multiple billing scenario
	* [ ] No billing mismatch
* [ ] SA & mass email sending: 6 Sprints **Anh Huy sẽ support phần này**
	* [ ] Mass email sending -> migrate to email-microservices (email-api)
		* [ ] Timeline (1 month - 2 sprints)
		* [ ] Design and approval (2 sprints) easy to test? but how to know?
			* [ ] 1 weeks
			* [ ] implement 3 weeks
	* [ ] SA (workflow) Create new services (by golang)
		* [ ] Brainstorming assessment: Don't need QA?
		* [ ] Create a new service often take 1 Sprints
			* [ ] new sevice: pipe line staging 1 week
			* [ ] models definition
			* [ ] port Copy logic
			* [ ] Design
			* [ ] set Stg env
		* [ ] Move logic backend-app to automation (2 sprints)
		* [ ] assessment & test (1 sprints)
* [ ] Monitoring: 2 sprints: **Optional**
	* [ ] Tuan sau kick off ()
	* [ ] Research infra (otel -> AMP (aws Prometheus) -> hosted Grafana, cloudwatch) 1 sprint
		* [ ] Muc tieu van la define metrics
	* [ ] Go-frame implement instrument (otel) 1 sprint
	* [ ] Create Board (depend infra) 
	* [ ] Set SLI / SLO 




## Task
%%Query tasks based on the tags field of the [Properties](https://help.obsidian.md/Editing+and+formatting/Properties) of the current file, extracted from all the notes%%
```LifeOS
TaskListByTag
```

## Bullet
%%Query bullets based on the tags field of the [Properties](https://help.obsidian.md/Editing+and+formatting/Properties) of the current file, extracted from all the notes%%
```LifeOS
BulletListByTag
```

## File
%%Query files based on the tags field of the [Properties](https://help.obsidian.md/Editing+and+formatting/Properties) of the current file, extracted from all the notes%%
```LifeOS
FileListByTag
```
