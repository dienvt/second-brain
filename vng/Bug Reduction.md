# Bug Reduction:
* Resource ko load được dẫn tới crash app.
* Loop màng hình bill home.
	* Bị xui
	* Test ko hết được toàn bộ device trên thị trường.
	Root cause:
	* Load nhiều quá. 
	Solution:
	* Dùng dashboard để monitor traffic load static (monitor throughtput) , notify ZMS, mail,.... Khắc phục khi lỗi không chặn được lỗi. Đơn giản dễ thực hiện. (anh Vũ)
	* Dùng automation, hiện tại đã thất lạc ở đâu đó mà ko ai biết ở đâu.
	* Phải có QC verified sau khi lên production.
	* Rút kinh nghiệm (note lại)
	* Unit test FE.

2 bug này sẽ được grooming bug

* Test thiếu, có test case nhưng mà vẫn test thiếu.
	Root cause:
	* QC Thiếu thời gian test, lúc deploy bị hối.
	Solution:
	* Thêm thời gian
	* Note lại thông tin test case trên môi trường production, thêm test execution.
	* Team tập sống chậm, có thời gian suy nghĩ biến những thức phức tạp về đơn giản.
	* Nên có task gối đầu sprint 


# Revamp missing trigger voucher


