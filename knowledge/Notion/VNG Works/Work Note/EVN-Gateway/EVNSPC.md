# PRO env cfg

endpoint:

- old: [http://10.50.32.32:8812/PaymentEVNSPC/PaymentService.svc](http://10.50.32.32:8812/PaymentEVNSPC/PaymentService.svc)
- new: [http://10.50.32.32:8812/Gateway/TGTT/PaymentService.svc](http://10.50.32.32:8812/Gateway/TGTT/PaymentService.svc)

  

bankID: 970435

# Develop

## Question

- gen transaction magic function
- how to settle when code dis match

EVN-provider

- User config ở file config mang giá trị là "ZaloPay"
- Billist là listBillID

EVN-Gateway

- Serve Esale và cả Zalopay. cho nên User là định nghĩa client gọi vào
- Không có gì đặc biệt

## Gen soap file

Step 1: ssh dev86 for dowload .wsdl file

`curl http://10.50.32.8:8811/PaymentEVNSPC/PaymentService.svc\?wsdl > /tmp/evnspc.wsdl`

Step 2: copy that file to local

`scp dev86:/tmp/evnspc.wsdl .`

Step 3: generate code

  

## Changelist

- remove bill `InvoiceBookId`
- remove bill `Description`
- remove `billCodes` in `PayBillFeesByCustomerCode`
- File generate by wsdl2go and soap from that lib, but i got some problem
    - namespace can't inject <ns:XXXX>, and don't have default namespace xmlns="XXX"
        - Can't fix by change NSAttr to `NSAttr string xml:"xmlns,attr"`, this approach require change soap client
    - Fix issue above i have another, call post man ok but by source return err. Cause by header SOAPAction didn't be set
        - Set header `SOAPAction` by method call, this approach require change soap client too.
    - Switch to gowsdl can't fix above issue but we got another issue can't call pay because of array long tag.
        - Solution add `arr` to xml, result be like `` Long []*int64 `xml:"arr:long"` ``
    - Both wsdl2go and gowsdl don't have `Schema` type. I don't know what that type is and can't find solution
        - Just remove source error
- From above solution and review source of Mr.Vinh and Mr.Hung, I have find the solution:
    - Define soap client from env-server
    - I generate by wsdl2go, after that I edit file since created:
        - wsdl2go < [http://10.50.32.8:8811/PaymentEVNSPC/PaymentService.svc\\?wsdl](http://10.50.32.8:8811/PaymentEVNSPC/PaymentService.svc%5C%5C?wsdl) > evnspcws.go
        - Using soap client i had copied substitute for wsdl2go soap client
        - New soap client provide function has different signature with wsdl2go soap client so i have to change in generated file too
    - Another issue is when provider response in Pay and Quota, the `result` is *string but that data have XML struct ( like `<Result><item></item></Result>`) so that XMLDecoder can't recognize and will return empty string.
        - Solution using soap client function to retrieve truth body and return follow by generated signature

## Docker run

```Bash
docker run -d --name=evn-providers \
--env-file /home/hungnv4/evn-providers/conf/.env \
-v /home/hungnv4/evn-providers:/app \
-p 8997:8080 \
--restart=always \
go.cps.provider:0.1 ./srv 

# without env
docker run -d --name=evn-providers \
-v /home/hungnv4/evn-providers:/app \
-p 8997:8080 \
--restart=always \
go.cps.provider:0.1
```