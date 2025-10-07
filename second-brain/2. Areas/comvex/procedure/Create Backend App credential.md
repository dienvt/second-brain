
* [ ] Create the application manually 
Using function `assignApiKey` (`digima-backend-app/app/Models/Core/Application.php`) to generate the api_key

```sql 
INSERT INTO `dgm_core`.`applications` (`created_at`, `updated_at`, `name`, `code`, `api_key`, `description`, `is_enabled`, `is_native`) VALUES (NOW(), NOW(), 'Digima Billing', 'digima_billing', 'bsVFdelQA8f4eBKxeu2L8H2XwxUN4vTGIif8k4Hjp3XCqtWuAYo4MTa4kg3F', NULL, 1, 0); 
``` 

* [ ] Create the client ID / Secret 
```bash
digima:oauth-client-create --force --application-id={id} --name=\\\"Digima Billing\\\" --code=digima_billing --platform=web --redirect-uri=https://auth.digima.com/auth --is-first-party=true --enabled=true --image-url=https://www.example.com/image.png --background-color='#FFFFFF' --foreground-color='#A0A0A0' 
``` 

* [ ] Store in vault and run the synchronization job
https://jenkins.mgmt.digima.com/job/Jenkins%20Vault%20Synchronization/


