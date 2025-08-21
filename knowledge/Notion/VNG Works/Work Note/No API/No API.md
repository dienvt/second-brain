## **Step 1 - Understand the Problem and Establish Design Scope**

### Concept design

No API là một kho nợ gồm 2 phần portal để import, export thông tin nợ và thông tin thanh toán. API để cho các bên tích hợp query nợ và gạch nợ từ hệ thống bill. Hằng tháng đối tác sẽ import file vào hệ thống thông qua portal, Client sẽ query và gạch nợ thông qua API và đối tác có thể xuất báo cáo thông quan portal.

Lượng request ntn ? format ntn? tính phí ntn? có liên quan tới thứ 7 hoặc chủ nhật hay không? một số nhà cung cấp ko làm việc thứ 7, cn nên nếu fix ngày phí thì nó hơi dở

  

Sao kê:

- Ngày có file Excel?
- Due-date: Ngày 05 (Ngày 06 được hiểu là trễ). Cái này là sao? due date ko có trong file hả?
- Phí dịch vụ: (ACS tự chia ra 2 luồng khi import data lên portal) theo em hiểu sẽ có phí hay sao? Portal có cần tính TH này ko, trong trường hợp portal team mình tự dựng
    - HĐ giữa ACS & KH phát sinh từ 01/08/2023: Thu phí user 9k (-VAT)
    - HĐ giữa ACS & KH phát sinh trước 01/08/2023: ACS trả phí dịch vụ.

Gạch nợ:

- **Hàng tháng**, download file và gạch nợ vào ngày T+1 (không có chấm nợ ngày T0). Sao hàng tháng nhưng download file gạch nợ vào T + 1? nếu download rồi thì có upload lại hay không?
- Phí phạt theo Due-date: Từ ngày 06 trở đi thì sẽ bị áp phí phạt với mức phí là 5.000 đ/ ngày, tối đa 100.000đ /tháng.

### System design

**Functional requirements**

**Non-functional requirements**

- Khách hàng có thể import File Excel (mẫu MM đính kèm)
- Download file gạch nợ vào ngày T+1 (không có chấm nợ ngày T0). Tức là không có gạch nợ realtime. Thông tin gạch nợ phải tính tới TH user muốn đóng nhiều hơn thì sao?
- Sao kê?

**Back-of-the-envelope estimation**

## **Step 2 - Propose High-Level Design and Get Buy-In**

**Business Knowledge 101**

- Có nên áp dụng nghiệp vụ kế toán : kiểu sẽ lưu 2 table khác nhau, tách bạch tiền nợ và gạch nợ. Lúc query sẽ compile với nhau.

**High-level design**

live diagram: [https://app.diagrams.net/#G1ZhSdnAQskdeOp6OLkYTp7XqNq2mi6cOL](https://app.diagrams.net/#G1ZhSdnAQskdeOp6OLkYTp7XqNq2mi6cOL)

![[no-api.drawio.png]]

  

Phí merchant import: Dựa vào data import, nhờ đối tác thêm thẳng vào file excel

Phí phạt (theo ngày 5k/1 ngày trễ): 5k một ngày và tối đa 100K

  

## **Step 3 - Design Deep Dive**

At this step, you and your interviewer should have already achieved the following objectives:

- Agreed on the overall goals and feature scope
- Sketched out a high-level blueprint for the overall design
- Obtained feedback from your interviewer on the high-level design
- Had some initial ideas about areas to focus on in deep dive based on her feedback

You shall work with the interviewer to identify and prioritize components in the architecture. Try not to get into unnecessary details.

## **Step 4 - Wrap up**

To wrap up, we summarize a list of the Dos and Don’ts.

Dos

- Always ask for clarification. Do not assume your assumption is correct.
- Understand the requirements of the problem.
- There is neither the right answer nor the best answer. A solution designed to solve the problems of a young startup is different from that of an established company with millions of users. Make sure you understand the requirements.
- Let the interviewer know what you are thinking. Communicate with your interview.
- Suggest multiple approaches if possible.
- Once you agree with your interviewer on the blueprint, go into details on each component. Design the most critical components first.
- Bounce ideas off the interviewer. A good interviewer works with you as a teammate.
- Never give up.

Don’ts

- Don't be unprepared for typical interview questions.
- Don’t jump into a solution without clarifying the requirements and assumptions.
- Don’t go into too much detail on a single component in the beginning. Give the high-level design first then drills down.
- If you get stuck, don't hesitate to ask for hints.
- Again, communicate. Don't think in silence.
- Don’t think your interview is done once you give the design. You are not done until your interviewer says you are done. Ask for feedback early and often.

  

Due date ; sẽ là ngày 6  
Nợ quá hạn theo tháng: ko trả trong range 6 -> 20 ngày  
Số tiền quá hạn thì đã được cộng: BILLAMOUNTofCURRENTMONTH  

Data mới nhất -> file hiện tại.

Phí

- > File xuất về cần những thông tin gì?
- > Số tiền thanh toán tiếp theo, ko chặn số lần thanh toán. Thanh toán chia từng kì, ko được nhập số tiền. tiền quá hạn sẽ là một kì khác.

C : chỉ cần thành toán tháng hiện tại, muốn đóng thêm vẫn ok

Dx -> phải thanh toán tiền nợ quá hạn + tiền nợ hiện tại. Lúc đó sẽ là status C và user có thể thanh toán mỗi BILLAMOUNTofNEXTMONTH.

File giao dịch: Payment: Thuyết phục reuse .  
Trạng thái thanh toán và trạng thái gạch nợ, -> gạch nợ T + 1 và File thì vẫn là một tháng mới import 1 lần.  

ACS: sẽ cập nhật lại. ACS: thanh toán dư, chủ động. User nhớ tổng số tiền thanh toán, 13  
Quy trình xử lý: Có đội CSKH liên hệ để xử lý khiếu nại khi user thanh toán.  

Trích nợ tự động. Chỉ Sacom, Samsum,

Lưu và tự động thanh toán. ACS vẫn ok ko vấn đề gì nếu ví muốn làm,  
Có API nhưng mà chưa đồng bộ. làm offline là thời gian ngắn.  

Gửi 1 file sang sang sftp. quá là ok?  
File ACS ->  

Portal phải có logic tính month để hiển thị đúng. Tính nợ của ACS như thế nào?  
Note lại case lúc ACS đang chuẩn bị file và gửi file thì user thanh toán, lúc đó phải làm như thế nào?