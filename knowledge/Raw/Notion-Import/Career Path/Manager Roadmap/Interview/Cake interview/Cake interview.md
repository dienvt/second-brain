## Round 1

Test in Hacker rank, 20 questions in 120 min

- Data struct:
    - Binary tree
- Algorithm:
    - bigO notation
    - search algorithm
- Design system:
    - flash sale and message queue
- SQL
- Golang:
    - call http to get something

## Round 2

1h30’ : Agenda

- 15’ giới thiệu bản thân
    - giới thiệu sơ về bản thân họ tên, năm sinh
    - Title hiện tại, kinh nghiệm trong ngôn ngữ nào, công nghệ nào?
    - Có share về công việc hiện tại, những project đã làm?
    - Share về task cảm thấy painful nhất và cách vượt qua nó
- 60’ làm bài tập: Đề bài thiết kế hệ thống portal cho phép user import một file excel vào và thực hiện top-up đúng số tiền vào từng account (câu hỏi không rõ ràng, lúc đầu còn ko clear file excel chứa cái gì) **kinh nghiệm nên áp dụng framework như trong bytebytego** thực hiện đủ các step.
    
    - Solution:
    
    ![[Untitled_Diagram.jpg]]
    
    - Trong lúc mình thiết kế, mình có show được việc tạo table schema như thế nào, đánh index như thế nào (**bị động vì ko rõ requirement**).
    - Vì là thiết kế theo event driven, nên việc chọn message queue có hơi khó khăn và cuối cùng là chọn kafka. Sau đó anh PV có hỏi khi tạo một topic thì cần những lưu ý gì? trả lời: replicate, partition và retention. Tiếp theo debate về cách commit offset(chỗ này hơi dở vì mình nên show cho họ thấy là performance của mình, có nhiều lúc cãi nhau ko hợp lý)
    - Tiếp tục cãi nhau về order status (lại một cái dở nữa là mình đã assume họ có knowledge như mình)
    - **Kinh nghiệm rút ra nên làm rõ requirement và quick estimate(cần tính toàn về lượng traffic)**
    - Về mặc người phỏng vấn: Hơi khó chịu vì interviewer đặt suy nghĩ của mình vào interviewee thay vì hỏi xoáy vào vấn đề.
- 15’ Q&A
    - Có hỏi về bức tranh mình mong muốn và lý do rời đi
    - T hỏi về tech-stack, nhiệm vụ trong team
        - Nghiệp vụ bank là chủ yếu
    - Đang áp dụng sprint, 1 tuần đầu đa số là họp vào QC viết test case, dev implement. tuần sau QC test và dev fix technical debt.