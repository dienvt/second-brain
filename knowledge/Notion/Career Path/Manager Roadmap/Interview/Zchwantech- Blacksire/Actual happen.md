Agenda: 1h30’

Section 1: introduce and share current position:

- Introduce ok
- Share about current system bill
- Question:
    - How to deal with many request come : using rate limit (ko chịu), async request(ko chịu) → suggest auto scale
    - Ask about roll back through micro services: Like when A call B, B call C fail how to recover
    - As about transaction: Làm sao mà 2 request 1 cái tăng, 1 cái giảm có thể xử lý
        - Dùng trasaction ở DB
        - Dùng ditributed lock
- Talk about event drive nhưng có vẻ interviewer ko hứng thú.
- Hỏi trả lời thôi chứ cũng ko có đánh giá gì

  

## Section 2: Hỏi về ngôn ngữ

- Phủ đầu hỏi về heap với stack: Đưa một đoạn code hỏi cái nào nằm trên stack cái nào nằm trên heap

```Java
main(){
	functionA(1,2);
}

void functionA(int a, int b){
	functionB("abc","cde");
}
```

- Không hỏi về GC
- Hỏi về data type nhiều. Enum, list, hash hỏi cả về cách implement.
- Hỏi về String, StringBuilder cách implement: 2 thread có share String pool ko?
- Rào luôn ko dùng Spring
- Không hỏi về OOP, SOLID, ACID vì không đủ thời gian.
- COUNT() vs SUM():
    - Sum cần check bên trong
    - count không cần check

## Section 3: Cultrule fit

- Khi có vấn đề gì giải quyết làm sao
- dùng công nghệ cũ thì có vấn đề gì ko

  

## My question

ask about team culture → trả lời: 8 giá trị cốt lõi con mẹ gì đó, lên website đọc

## HR question

Có cấp laptop cho ko?

- Có cấp laptop dùng window
- Có monitor nếu yêu cầu