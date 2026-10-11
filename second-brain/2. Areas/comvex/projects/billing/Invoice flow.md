
## Overview
- Check the BI to find the mismatch
- BI already migrated: So billing missing this
	- Feature usages
	- Billing Segments
	- Perform query 
		- Verify there is always equal or grate 23 billing segments
		- Check how many account generated billling segments
		- The billing segment 
	- If there is mismatch usage, you have to run billing-data-recalculate account--period 
	- 
	- Verify feature usage:
		- Check if all daily usage was created within account
	- Feature usage is run upsert-daily-usage-data-rec
- If everythin ok let send messsage to feat-digima-admin
- We're feature , billing BI redshift. Reason: 
- New billing system
- Send invoice: Where can I Don't have feature. Download report from tablue, Upload csv to third party (Robot payment app) Then this app send invoice to our customer.
- Next task is automate send to Robot payment app or we do own platfom.

### Questions
1. Do we need authen middleware in worker? Because I found that we don't have auth when trigger from Event Bridge? me miss a lot of context authen token?
2. How OneTime cost calculate?
3. How sms_message_inbound was related to?



### TODOs
1. [x] Digima infa/local add cache config (already added before)
2. Test local with scheduler

Event billing.create -> trigger 

this one will get all message then 
is this ok to use just one job???
generate token???? 
How to know when all token are ok?
cron job test????
create event all segment created but how to do that? using a dispatch
![[Drawing 2025-10-01 15.51.59.excalidraw]]