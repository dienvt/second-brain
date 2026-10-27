---
Round: Technical Interview
Date: tp.date.now("YYYY-MM-DD")
Candidate:
Decision:
Evaluation:
---
## Agenda: 1st technical interview (60m)
- [ ] Greeting and ask for the introduction. Ice breaking
	- [ ] Introduce interviewee (english is optional)
		- [ ] 8 nam kinh nghiem
- [ ] Hỏi về định hướng khi chọn andpad
	- [ ] Match voi he thong cua minh
		- [ ] long term: Hien tai IC
		- [ ] dinh huong leader team
		- [ ] 
- [ ] Kinh nghiệm làm việc (15’)
	- [ ] Daily tasks
		- [ ] Build data pipe line
		- [ ] remote
	- [ ] Lam ve y te
	- [ ] Dang lam data pipeline
		- [ ] Lay data cua khach hang ban du lieu sach
		- [ ] Microservice giao tiep nhu the nao
		- [ ] Architect
			- [ ] Multiple service
				- [ ] Phuc vu 1 business
			- [ ] ko can realim nen noi chuyen bang api
			- [ ] doi tac va cloud:
				- [ ] Queue gui request
		- [ ] Optimize pipeline
			- [ ] Nhan du lieu cua khach hang:
				- [ ] Anh Y te: MRI, x-quang
				- [ ] Data partner truyen data nhu the nao
					- [ ] report dung mot con service stream qua
					- [ ] report gui ngay nhung anh thi gui lau
			- [ ] Khach hang:
				- [ ] Search ben an, chon dataset.
				- [ ] Luc do thi moi di lay anh.
				- [ ] Detail implement
					- [ ] User co the chon nhieu user partner
					- [ ] Pick 
					- [ ] User request update status
					- [ ] send kafka: topic 
					- [ ] VM: co 1 service event save vao db sau do se co cron de craw data.
						- [ ] Truoc version hien tai nhan event va di lay luon.
						- [ ] Nhan event thi process phai success: ko trace dc status processing
						- [ ] Nhan event goi sang nhan anh, nhan anh van dang xu ly. can cron job hoi con xu ly anh return lai result.
						- [ ] Over queue: Listen lai request la thang can gi, can xu ly an thoi.
						- [ ] co hai mode, base vao con image service. chay sync va async.
						- [ ] 
					- [ ] pulish ko dc thi ko update db?
					- [ ] `Safe to reproduce` 
						- [ ] gateway va con listener
						- [ ] da xu ly roi ko xu ly lai. Chi xu ly lai khi co co force
						- [ ] phia con nhan message co cache message xu ly roi.
						- [ ] Cache trong db: 1 thang toi 90 ngay. 
						- [ ] cache: File path on the cloud
						- [ ] 
					- [ ] 
			- [ ] Memory leak xay ra luc nao va vi du:
				- [ ] Nhan anh: leak 
				- [ ] Xu ly tren cloud: 
				- [ ] bi OOM, co the reproduce. Dung pprof
				- [ ] VM: dung ban metrics len prometheus va grafana de visualize.
				- [ ] Infra: mem va redis
				- [ ] Khong phai million user
				- [ ] Nhieu data dc build 
			- [ ] Request nhieu data benh an:
				- [ ] Chieu sau toi uu 
				- [ ] Toi uu chieu sau: 
					- [ ] turning bang cach stream line image
				- [ ] toi uu chieu ngang:
					- [ ] Khi co da ta roi 
	- [ ] Khó khăn vào thách thức
		- [ ] Thach thuc:
			- [ ] Touch, de 1 table 900 rows
			- [ ] Ichident:
				- [ ] Overflow ID: Split id ve so am
				- [ ] tao mot column bigint, migrate va update primary key moi
			- [ ] Slow query:
				- [ ] Check plan
				- [ ] Doi plan
				- [ ] Prune data dinh ki
			- [ ] Pogress
				- [ ] Random doi plan
	- [ ] Dung context nhu the nao
		- [ ] Context truyen vao 
		- [ ] Shutdown.
		- [ ] truyen vao goroutine
		- [ ] Can xu ly song song khong, gioi han so luong 
		- [ ] Spawn goroutine, share data bang channel waitgroup hoac leafe.
		- [ ] 
		- [ ] 
- [ ] #technical_knowledge 
	- [ ] What is the protocol use to communicate between your services
	- [ ] How do you deploy your service, to mange deplyment state?
		- [ ] Ask do I familiar with CI/CD.
			- [ ] Gitlab ci, gitlab runner.
		- [ ] Ask do I familiar with k8s.
			- [ ] Build docker file.
	- [ ] How do you observer or telemety your services? Do you apply tracing
		- [ ] 
	- [ ] How to you write test and manage test coverage.
		- [ ] 
	- [ ] Which one your code need unit test and what is the perfect percent of coverage
	- [ ] Do you apply any system design, Design pattern to your services. What is pro and cons of that design
- [ ] Problem Solving
- [ ] Logical Thinking
	- [ ] Kubenet: 
	- [ ] request
- [ ] OAuth
- [ ] OOP
	- [ ] What is OOP ?
	- [ ] What is SOLID ?
- [ ] Golang
	- [ ] What is goroutines
	- [ ] How to avoid rate condition
	- [ ] How you resolve consensure problem indo
- [ ] Ask about SA: How service talking with each other
- [ ] Draw about current project structure

## Interview Preparation Summary

### 1. **Golang-Specific Topics**
### 1. Goroutine leak

**Đáp án tốt:** Goroutine bị block vĩnh viễn nên không bao giờ được thu hồi. Nguyên nhân phổ biến:

- Gửi/nhận trên channel không có phía đối ứng (điển hình: worker `ch <- result` nhưng caller đã `return` vì timeout).
- Không truyền/không lắng nghe `ctx.Done()` trong vòng lặp.
- `for { select { case <-time.After(d): ... } }` — mỗi vòng tạo timer mới (từ Go 1.23 GC đã xử lý tốt hơn, nhưng vẫn nên dùng `time.NewTimer` + `Stop`).
- Quên `defer resp.Body.Close()` → rò connection.

**Cách phát hiện:** đẩy `runtime.NumGoroutine()` lên Prometheus và alert khi đường biểu đồ chỉ đi lên không đi xuống; xác nhận bằng `/debug/pprof/goroutine?debug=2` để xem stack trace nhóm goroutine đang kẹt ở đâu.

> **Cờ đỏ:** chỉ nói "goroutine chạy mãi không kết thúc" mà không biết pprof, hoặc nói "restart pod là hết".  
> **Điểm cộng:** nhắc pattern gửi vào **buffered channel size 1** để worker không bị block khi caller đã bỏ đi.

---

### 2. Context propagation

**Đáp án tốt:** `ctx` là tham số đầu tiên của mọi hàm I/O, không lưu trong struct. Dùng `QueryContext`/`ExecContext` thay vì `Query`/`Exec`, RabbitMQ dùng `PublishWithContext`, HTTP client dùng `http.NewRequestWithContext`. Khi client hủy, driver Postgres sẽ gửi cancel request lên server.

**Điểm cộng (rất mạnh):**

- Nhận ra rằng **hủy context không rollback được thứ đã ghi** — nên với write critical phải tách: nhận request → ghi vào outbox/queue → xử lý bằng context riêng.
- Biết `context.WithoutCancel` (Go 1.21+) hoặc tạo context mới cho background job không được chết theo request.
- Không dùng `context.Value` để truyền business data (chỉ request-id, trace-id, auth).

> **Cờ đỏ:** "em truyền `context.TODO()` cho nhanh" hoặc không phân biệt được context của request và của service.

---

### 3. Typed nil interface

**Đáp án đúng: `err != nil` là TRUE.**

Interface trong Go là cặp `(type, value)`. Khi gán `*MyError` nil vào biến `error`, phần _type_ là `*MyError` (khác nil), nên bản thân interface **không** nil — dù value bên trong là nil.

go

```go
func do() error {
    var e *MyError = nil
    return e          // BUG: caller thấy err != nil
}
// Đúng:
func do() error {
    if failed { return &MyError{} }
    return nil        // trả nil literal
}
```

> Đây là câu lọc mạnh nhất. Người dùng Go thật sự 9 năm hầu như đã từng dính bug này và sẽ kể được ngữ cảnh cụ thể (thường gặp khi hàm trả `*CustomError` rồi gán vào biến `error`).  
> **Cờ đỏ:** trả lời "false" và không giải thích được cấu trúc interface.

---

### 4. Channel vs Mutex vs sync.Map

**Đáp án tốt:**

- **Channel** khi cần _chuyển quyền sở hữu dữ liệu_ hoặc điều phối luồng: pipeline, fan-out/fan-in, signaling, giới hạn concurrency (semaphore bằng buffered channel).
- **Mutex** khi chỉ cần _bảo vệ state dùng chung_ với critical section ngắn: counter, cache in-memory, registry connection. Đơn giản và nhanh hơn channel cho việc này.
- **`sync.Map`** chỉ hợp 2 trường hợp theo doc: (a) key ghi một lần đọc rất nhiều lần, (b) các goroutine thao tác trên tập key rời nhau. Ngoài ra `map` + `RWMutex` thường nhanh hơn và type-safe hơn.

> **Điểm cộng:** nhắc "Don't communicate by sharing memory; share memory by communicating" nhưng _đồng thời_ nói rõ đây không phải luật cứng — lạm dụng channel cho state đơn giản làm code khó debug. Hoặc nhắc **sharded map** khi contention cao.  
> Với dự án WebSocket của anh (SDB/WeeIO), hỏi thêm: "registry giữ hàng nghìn connection anh dùng gì?" — câu trả lời hợp lý là map + RWMutex có shard.

---

### 5. Slice gotcha khi append

**Đáp án tốt:** Slice cắt ra vẫn dùng chung backing array, và **cap kéo dài đến hết array gốc**, nên `append` sẽ ghi đè dữ liệu của slice cha:

go

```go
a := []int{1,2,3,4,5}
b := a[1:3]          // len=2, cap=4
b = append(b, 99)    // a giờ là [1,2,3,99,5] — a[3] bị ghi đè
```

**Cách tránh:** three-index slice `a[1:3:3]` để ép `cap == len` (append sẽ buộc phải cấp phát mới), hoặc `slices.Clone(b)`.

> **Điểm cộng:** nhắc thêm biến thể memory leak — giữ một slice nhỏ cắt từ buffer 10MB sẽ giữ nguyên cả 10MB không cho GC thu hồi; phải copy ra.  
> Và: truyền slice vào hàm rồi append — caller _có thể_ thấy hoặc không thấy thay đổi tùy cap, đây là nguồn bug rất khó lần.

---

### 6. Performance / giảm allocation

**Đáp án tốt (phải kể được một case thật, có số liệu trước–sau):**

- Đo trước: `go test -bench -benchmem`, heap profile qua pprof, xem `alloc_objects`/`inuse_space`.
- Xem escape analysis: `go build -gcflags='-m -m'`.
- Các khoản dễ ăn nhất: `make([]T, 0, n)` khi biết trước size; `strings.Builder` thay vì `+=`; tránh convert `[]byte` ↔ `string` trong hot path; tránh `fmt.Sprintf` trong vòng lặp.
- `sync.Pool` cho buffer lớn tái sử dụng (ví dụ encode/decode JSON, đọc UDP datagram) — nhưng biết rõ pool bị dọn sau mỗi chu kỳ GC nên **không** dùng cho object nhỏ.
- Từ Go 1.19: `GOMEMLIMIT` để kiểm soát GC trong container thay vì chỉ chỉnh `GOGC`.

> **Điểm cộng lớn:** nói rõ "em đo trước rồi mới tối ưu" và thừa nhận có lần tối ưu xong không cải thiện gì.  
> **Cờ đỏ:** liệt kê tip tối ưu thuộc lòng nhưng không kể được lần nào tự profile; hoặc nói "em dùng sync.Pool cho mọi struct".  
> Dự án **Waternet** (3.000 IoT gửi UDP) là chỗ hoàn hảo để hỏi tiếp — đọc UDP mà cấp buffer mới mỗi packet là điểm tối ưu kinh điển.

---

### 7. `errors.Is` vs `errors.As`

**Đáp án tốt:**

- `errors.Is(err, target)` — so sánh **danh tính**, đi dọc chuỗi wrap để tìm đúng sentinel error. Dùng cho `sql.ErrNoRows`, `context.DeadlineExceeded`, `io.EOF`.
- `errors.As(err, &target)` — tìm error đầu tiên trong chuỗi **khớp kiểu**, gán vào target để đọc field. Dùng cho custom error có mã lỗi, HTTP status, retryable flag.
- Wrap bằng `fmt.Errorf("...: %w", err)` để giữ chuỗi; dùng `%v` là **cắt đứt** chuỗi (đôi khi cố ý, để không lộ lỗi tầng dưới ra API).

**Quy ước team tốt:** wrap kèm ngữ cảnh ở ranh giới tầng (repo → service → handler), **không** vừa log vừa return cùng một error (gây log trùng lặp), chỉ log ở tầng ngoài cùng.

> **Cờ đỏ:** vẫn dùng `err == ErrNotFound` hoặc `strings.Contains(err.Error(), "not found")`.

---

### 8. Graceful shutdown trên Kubernetes

**Đáp án tốt — quan trọng là _thứ tự_:**

1. `signal.NotifyContext(ctx, syscall.SIGTERM, os.Interrupt)`.
2. **Fail readiness probe trước**, rồi _chờ vài giây_ — vì kube-proxy/ingress cần thời gian để gỡ pod khỏi endpoint list; nếu shutdown ngay sẽ vẫn có request mới bay vào và bị từ chối. (Cách khác: `preStop` hook `sleep 5`.)
3. `srv.Shutdown(ctx)` với timeout — ngừng nhận connection mới, chờ request đang chạy xong.
4. Dừng consumer: hủy consumer trên RabbitMQ (`Cancel`) để không nhận message mới, chờ message đang xử lý hoàn tất rồi mới `Ack`; message chưa xử lý sẽ tự requeue.
5. Đóng DB pool, flush trace/metrics cuối cùng.
6. `terminationGracePeriodSeconds` trong K8s phải **lớn hơn** tổng thời gian trên, nếu không sẽ ăn SIGKILL giữa chừng.

> **Điểm cộng rất mạnh (rất hợp CV của anh):** biết rằng **`http.Server.Shutdown` không đóng connection đã hijack — tức là WebSocket không nằm trong diện chờ**. Phải tự quản lý registry connection, gửi close frame và chờ client reconnect sang instance khác. Anh làm SDB và WeeIO đều dùng WebSocket, nếu không biết điều này thì mức độ "architect, deploy, maintain production" cần đặt dấu hỏi.

---

### Cách quy đổi điểm nhanh

|Số câu trả lời có chiều sâu|Kết luận|
|---|---|
|7–8|Senior/Lead thật, đi tiếp phần system design|
|5–6|Solid senior, thiếu vài mảng — hỏi bù bằng code review|
|3–4|Mid-level dùng Go nhiều hơn là hiểu Go|
|≤ 2|Không khớp với claim 9 năm trên CV|

Lưu ý khi chấm: đừng trừ điểm vì anh quên cú pháp chính xác. Cái cần nghe là **anh có kể được ngữ cảnh thật đi kèm không** — "em từng dính cái này ở dự án Facepay, lúc đó..." đáng giá hơn nhiều so với một định nghĩa trơn tru.

Mình có thể viết luôn đoạn code Go ~30 dòng cài sẵn 3 lỗi cho phần code review, kèm đáp án — nếu bạn muốn dùng trong buổi phỏng vấn.

- **Concurrency:** 
  - **Question:** "Can you tell me about how you typically handle concurrency in Go?"
  - **Answer:** Discussed using goroutines and channels, best practices like `sync.Mutex`, `sync.WaitGroup`, and avoiding race conditions.
- **Context**:
	- How you use context in your 
- Live coding:
	* Q1: https://goplay.tools/snippet/CtzeYviqY6u
		* **Scenario:** We are building the "Profile View" feature for a system like LinkedIn.
			* Actor**:** A Viewer (User A).
			* Target**:** A Profile Owner (User B).
			* Action**:** User A views User B's profile.
			* Requirements**:**
				* Return User B's profile data to User A
				* User B should receive a notification: *"User A viewed your profile."*
		* **Purpose**:
			* Check biz logic
			* Check testing skills
		* **Follow up questions**
			* The notification service is slow (takes 2 seconds). We don't want the Viewer to wait 2 seconds to see the profile. How do we fix this?
				* Expect use the goroutine with override the context to avoid parent context cancel and traceable
				* It's great to control the goroutine by worker pool
    * Q2: https://goplay.tools/snippet/b4_znTpKqmI
        * **Scenario:** Walk me through what happens when this code runs. Specifically, what will be printed to the console?
        * **Purpose:**
            * Check Channel Fundamentals: Do they understand that unbuffered channels block until both sender and receiver are ready?
            * Check Concurrency Awareness: Do they recognize that after the channel exchange, the order of execution is non-deterministic?
        * **Follow up questions:**
            * How does the behavior change if we change the line to `ch := make(chan int, 1)`?
                * The goroutine sends `1` into the buffer and immediately continues to `fmt.Println("Sent 1")`.
                * "Sent 1" is extremely likely to print *before* "Received 1" 
    * Q3:https://goplay.tools/snippet/ZZG6-TLf7E0
        * **Scenario:** Implement the `canConstruct` function. You must account for the frequency of letters (e.g., if the target needs two 'a's, the source must provide at least two 'a's).
        * **Purpose:**
            * Check Data Structure Selection
            * Check Algorithm Logic
            * Check Optimization Awareness: Since the constraint specifies "lowercase alphabet only," do they realize they can optimize space?
        * **Follow up questions:**
            * Is there a simple check we can do at the very beginning to make this function faster?
                * Check length. If `len(target) > len(source)`, return `false` immediately. 
	* Q4: https://goplay.tools/snippet/_d8W_1Zpm40
		- **Scenario**: Implement `Run` function. 
		- **Purpose:**
			- Check channel or sync package usage
			- Check graceful shutdown
			- check the goroutine in practice
		- **Follow up questions:**

### 2. **Database Integration**
- **Query Optimization:** Techniques like indexing, avoiding `SELECT *`, using proper joins, and analyzing query execution plans.

### 3. **Distributed Transactions**
- **Saga Pattern:**
  - **Question:** "Can you explain the Saga pattern and how it works in distributed systems?"
  - **Answer:** The Saga pattern indeed involves a series of local transactions, each with a corresponding compensation transaction. The orchestrator is responsible for managing the overall workflow and ensuring that if any step fails, it triggers the compensating transactions to roll back the previous steps.

The two main approaches are:

1. **Orchestration:** Here, an orchestrator service coordinates the saga by calling each microservice in sequence and managing the compensations if something fails.
    
2. **Choreography:** In this approach, each microservice knows when to perform its local transaction and how to handle compensation, and they communicate through events.

* Two phase commit:

|Aspect|2PC|3PC|Saga|
|---|---|---|---|
|Consistency|Strong (ACID)|Strong|Eventual|
|Blocking|Yes|No (theoretical)|No|
|Availability|Low|Medium|High|
|Scalability|Poor|Poor|Excellent|
|Complexity|Medium|High|High (business logic)|
|Real-world usage|Legacy systems|Rare|Very common|


### 4. **Web Architecture**
- **Communication:** Using GraphQL for public APIs and gRPC for internal service communication.
- **Scalability:** Deployment in Kubernetes for auto-scaling, fault tolerance, and using caching layers to reduce load.

### 5. **System Design & Scaling**
- **Load Balancing:** Distributing traffic to prevent overload.
- **Horizontal Scaling:** Adding more instances to manage increased load.
- **Caching:** Using Redis or Memcached to reduce database load.
- **Database Sharding:** Partitioning data to improve performance.
- **Auto-Scaling:** Automatically adjusting resources based on traffic.

## **Sample Questions Practiced:**
### **Scaling Example Question:**
- *"Imagine you’re designing an e-commerce platform that expects a large number of users during peak seasons. How would you design the system to handle this high load?"*

### **System Design Example Questions:**
- *"Can you describe a microservice you previously worked on? What was its main functionality, and how did you design it to handle high traffic and ensure fault tolerance?"*

- *"What were some of the key challenges you faced during the development of that service, and how did you overcome them?"*

### **Additional Questions Practiced:**

- **Concurrency in Go:**
  - *"How do you use goroutines and channels to manage parallel tasks in Go?"*

- **Saga Pattern in Distributed Transactions:**
  - *"Can you explain how the Saga pattern works and how you handle compensating transactions?"*

- **Web Architecture and Communication:**
  - *"Why do you choose GraphQL for public APIs and gRPC for internal communication?"*

---
The two main approaches are:

1. **Orchestration:** Here, an orchestrator service coordinates the saga by calling each microservice in sequence and managing the compensations if something fails.
    
2. **Choreography:** In this approach, each microservice knows when to perform its local transaction and how to handle compensation, and they communicate through events.

### Account and member
For example Multi Tenant:
- User belong to the account?
- How to search by name?