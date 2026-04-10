mirage: [https://confluence.zalopay.vn/display/ZA2/APIs+for+merchant#APIsformerchant-Path:/v2/agreement/query_pin](https://confluence.zalopay.vn/display/ZA2/APIs+for+merchant#APIsformerchant-Path:/v2/agreement/query_pin)

  

token cũ: 33 ký tự  
mới: 49  

  

Hiện tại

  

  

  

- [ ] Lên v2
    
    - [ ] Worker v2
    - [ ] Test flow ĐK bằng autodebit-management
    - [ ] Test paid by agreement v2.1
    
      
    
- [ ] Migrate: test làm sao?
    - [ ] chìa một endpoint ra để chọn user nào sẽ được migrate, và hệ thống sẽ migrate user đó, business phải implement.
        - [ ] Gọi sang phía autodebit-management để binding? Cần check lại có notify đến user hay không?
        - [ ] Remove binding phía core-api và agreement v1
    - [ ] Lúc test chỉ cần verify user đó được paid bằng token v2 (đi qua luồng mới)
        - [ ] Show được trên tab quản lý của bill
        - [ ] Không còn show trên tab cá nhân

# Import into DB:

- auto debit worker : insert metadata : status = 0 để ko được trigger
- auto debit manager: insert new token

Test

- QC đăng kí một agreement v1
- Action migrate
- disable cũ
- call api for trigger pay in v2

# Remove agreement

- [ ] Agreement
- [ ] Metadata phía worker

  

  

  

app,suppli,cus

createorder(ctx, req interface{})