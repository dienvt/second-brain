- Email 
- Envelopes
- Attachment

Email have status draft scheduler, send. The envelop only have sent status
Attachment won't count to the email sent usage
Envelops sent will 

For storage we count all envelop content and file size

attachment will attached to the envelop via attachment.ResourceId

Envelop create along?

Why we need attachment create?

worker
validateAndCreateSentEnvelope>???

for storage_count we will sum all the file? even though there are temporary file NOPE because the creation update temporary to fileID

## Storage size là accumulate

## Delete by threadID
- Delete batch envelops
- Single delete attachments and async
- 
