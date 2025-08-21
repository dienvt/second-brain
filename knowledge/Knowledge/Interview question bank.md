Redis

Chuyện gì sẽ xảy ra khi redis đầy

Có 2 khả năng: trả về error đối với write command (read-only vẫn ok) hoặc evict key (dùng nếu như xem redis là cache)

Hỏi thêm về eviction policy

noeviction: trả về lỗi

allkeys(volatile)-lru: Xoá cách key ít được sử dụng (thời gian)

allkeys(volatile)-lfu: Xoá key ít dùng (tần suất)

allkeys(volatile)-random: randome

volatile-ttl: xoá key có ttl ngắn nhất

Hỏi thêm khi nào dùng loại eviction nào?

allkeys(volatile)-lru: dùng cho data có độ hot giảm dần theo thời gian (ví dụ như báo chí, trend)

allkeys(volatile)-lfu: Dùng cho data có độ hot giảm dần theo loại (ví dụ như nhạc của sơn tùng sẽ được nghe nhiều hơn nhạc của chi pu)

Hỏi xem có dùng redis cluster hay không?

Nếu có thì hỏi về failover

Cách node nói chuyện với nhau thông qua một protocol ngầm (khác với client). Khi 2 node ko ping được tới một node thì sẽ mark node đó down và tiến hành promote slave lên làm master. Tiến trình có thể kéo dài do phải check data slave có đảm bảo đủ hay không

Golang

slice truyền vào function là tham chiếu hay tham trị?

Trả lời là tham trị thì chỉ hiểu được bề nổi.

function trong go luôn luôn tham trị (by value)? vì slice lưu point tới underline array cho nên khi thay đổi giá trị ở slice chính là thay đổi giá trị ở underline array. Điều này dẫn tới việc khi ra khỏi function thì gía trị slice cũng sẽ thay đổi. Tuy nhiên nếu như thay đổi độ dài của slice (append) thì một underline array mới sẽ được allocate dẫn tới khi ra khỏi function giá trị của slice sẽ ko đổi.

string vs []byte?

[]byte là slice nên nó mang đầy đủ tính chất của slice. string thì immutable

type string struct {

data uintptr

len int

}

Duck typing?

có thể truyền int vào func(v interface{}) nhưng không thể truyền []int và func(v []interface{})

embed struct và inherit?

Câu hỏi nâng cao: Khi nào biến được allocate trên heap ? khi nào trên stack?

ans: Khi biến ko escapes khỏi function thì sẽ được allocate trên stack

ex: cả x,y đều được khai báo trên stack, z được khai báo trên heap

func f(){

y := new(int)

- y = 1

```Plain
		x := 1
	}
	var global *int
	func g(){
		z := 1
		global = &z
	}
```

Data struct

array khác gì linklist?

ans: array random access, linklist iterate

UTF-8 là gì không?

Develop Order Processing System

Hỏi xem có dùng cache hay không

Giải quyết vấn đề cache invalidation thế nào?

Write-Around: Write DB first, cache async: regular, but when using cache-aside, data might be wrong until cache updated

Write-Through: Write DB and cache in the same time: slow but data fresh in both db and cache

Write-behind: Write cache first, db async : quick but data maybe wrong

Dùng cách nào để giải quyết vấn đề trên?

Làm cách nào để update data từ nhiều nguồn?

Hứng event thể nào : Dùng callback, pub/sub hay query status?

Sẽ làm gì nếu ko có callback, hoặc pubsub

Khi nhiều trạng thái cùng đến thì lock thế nào?

Ví dụ như khi order được update trạng thái thì có nhiều phụ việc phải thực hiện (như gửi mail, cập nhật database,…) thì phải làm thế nào?