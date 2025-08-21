Change the routing  
# 1. Diagram:![[Diagram 3.svg]]  
# 2. HTTP  
## External 
>Routing read to V2 (bill storage): change FE call to zalopay.vn (tracking new on k8s), switch from frontend  
### Billing
* [x] getcustomergroup  
* [x] getcustomer  
* [x] removecustomer  
* [x] getallbill  
* [x] getnumallbill  
* [x] getstorageinfo  
### Postpaid
* [ ] getcustomergroup  
* [ ] getcustomer  
* [ ] removecustomer  
* [ ] getallbill  
* [ ] getnumallbill  
* [ ] getstorageinfo  

## Internal
* [x] Bill Core  
	* [x] Update multibill
		* [x] kafka chưa consumer, physical sẽ consume.
* [ ] Postpaid 

## Kafka
* [ ] Update multibill Listener
* [x] Auto query Listener
	* [x] call 2 con core (network policy) nhưng đã đi proxy -> ko vấn đề.
