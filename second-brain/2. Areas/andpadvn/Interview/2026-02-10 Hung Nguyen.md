- [ ] 1st technical interview (60m).
    - [ ] Greeting and ask for the introduction.
	    - [ ] Introduce interviewee.
    - [ ] Hỏi về định hướng khi chọn andpad
    - [ ]  Hỏi về kinh nghiệm làm việc (15’)
	    - [ ] Daily tasks
	    - [ ] Platform tạo form để nộp thuế
	    - [ ] Form 40 session 
		    - [ ] INP giảm từ 20s về 0.2s
		    - [ ] Có backend lưu stage
		    - [ ] Có websocket để lưu trạng thái của aws. 
		    - [ ] Allow 2 người  cũng edit một form.
		    - [ ] Navigation...
		    - [ ] Metric, P.999
        - [ ] Khó khăn vào thách thức
	- [ ] Project logistic
		- [ ] Kho muốn nhập hàng: số lượng, khối lượng, inventory, lưu trữ metrics lại.
		- [ ] lưu unit theo gram, lit. Lưu theo tree direct.
		- [ ] Tại sao lại phải lưu theo tree
			- [ ] User có thể Customize unit và tạo relation tới 
			- [ ] Kg
		- [ ] 
    - [ ] Verifify wide then narrow the scope
	    - [ ] What is the protocol use to communicate between your services
		    - [ ] 
	    - [ ] How do you deploy your service, to mange deplyment state?
			- [ ] Ask do I familiar with CI/CD
				- [ ] Gitlab ci, gitlab runner.
				- [ ] 
			- [ ] Ask do I familiar with k8s.
				- [ ] Build docker file.
	    - [ ] How do you observer or telemety your services? Do you apply tracing
		    - [ ]  
	    - [ ] How to you write test and manage test coverage.
		    - [ ] 
	    - [ ] Which one your code need unit test and what is the perfect percent of coverage
	    - [ ] Do you apply any system design, Design pattern to your services. What is pro and cons of that design
	- [ ] OAuth
	- [ ] OOP
		- [ ] What is OOP ?
		- [ ] What is SOLID
	- [ ] Golang
		- [ ] What is goroutines
		- [ ] How to avoid rate condition
		- [ ] How you resolve consensure problem indo
	- [ ] Ask about SA: How service talking with each other
	- [ ] Draw about current project structure
	- [ ] 
	- [ ] 
# Interview Preparation Summary

## 1. **Golang-Specific Topics**
- **Concurrency:** 
  - **Question:** "Can you tell me about how you typically handle concurrency in Go?"
  - **Answer:** Discussed using goroutines and channels, best practices like `sync.Mutex`, `sync.WaitGroup`, and avoiding race conditions.
- **Context**:
	- How you use context in your 

## 2. **Database Integration**
- **Query Optimization:** Techniques like indexing, avoiding `SELECT *`, using proper joins, and analyzing query execution plans.

## 3. **Distributed Transactions**
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


## 4. **Web Architecture**
- **Communication:** Using GraphQL for public APIs and gRPC for internal service communication.
- **Scalability:** Deployment in Kubernetes for auto-scaling, fault tolerance, and using caching layers to reduce load.

## 5. **System Design & Scaling**
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
