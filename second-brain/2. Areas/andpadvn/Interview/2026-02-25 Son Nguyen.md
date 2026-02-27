- [ ] 1st technical interview (60m).
    - [ ] Greeting and ask for the introduction.
	    - [ ] Introduce interviewee.
	    - [ ] Backend, được 
	    - [ ] 6-7 năm code golang
    - [ ] Hỏi về định hướng khi chọn andpad
	    - [ ] Định hướng:
		    - [ ] Code cải thiện chiều sau về technical
		    - [ ] Có kinh nghiệm quản lý
			    - [ ] Quản lý 4-5 
			    - [ ] Core member: 15 người
    - [ ]  Hỏi về kinh nghiệm làm việc (15’)
	    - [ ] Daily tasks
        - [ ] Khó khăn vào thách thức
		- [ ] Hỏi về kinh nghiệm quản lý
			- [ ] Thêm vấn đề về con người: Không trả lời rõ ràng được.
			- [ ] Cty startup có rót vốn.
			- [ ] Host daily meetings
			- [ ] Lý do rời đi:
				- [ ] Doanh thu có vấn đề nên bán
		- [ ] Làm về smart contract
		- [ ] Làm về trading và làm sàn
		- [ ] Bạn làm về BE
			- [ ] Có team devops
				- [ ] K8s tự động scale, ci/cd đều có. ArgoCD
				- [ ] Grafana:
					- [ ] Error log
					- [ ] Test
					- [ ] Metrics ram, database.
			- [ ] verify requirement
		- [ ] Trả lời khá chán, Không đi vào ví dụ cụ thể
		- [ ] Dùng worker pool để giải quyết 
    - [ ] Verifify wide then narrow the scope
	    - [ ] What is the protocol use to communicate between your services
		    - [ ] 
	    - [ ] How do you deploy your service, to mange deplyment state?
			- [ ] Ask do I familiar with CI/CD
				- [ ] Gitlab ci, gitlab runner.
			- [ ] Ask do I familiar with k8s.
				- [ ] Build dockerfile.
	    - [ ] How do you observer or telemety your services? Do you apply tracing
		    - [ ]  Yếu 
	    - [ ] How to you write test and manage test coverage.
		    - [ ] 
	    - [ ] Which one your code need unit test and what is the perfect percent of coverage
	    - [ ] Do you apply any system design, Design pattern to your services. What is pro and cons of that design
	- [ ] OAuth
		- [ ] 
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

[Optimistic Locking](http://en.wikipedia.org/wiki/Optimistic_locking) is a strategy where you read a record, take note of a version number (other methods to do this involve dates, timestamps or checksums/hashes) and check that the version hasn't changed before you write the record back. When you write the record back you filter the update on the version to make sure it's atomic. (i.e. hasn't been updated between when you check the version and write the record to the disk) and update the version in one hit.

If the record is dirty (i.e. different version to yours) you abort the transaction and the user can re-start it.

This strategy is most applicable to high-volume systems and three-tier architectures where you do not necessarily maintain a connection to the database for your session. In this situation the client cannot actually maintain database locks as the connections are taken from a pool and you may not be using the same connection from one access to the next.

[Pessimistic Locking](http://en.wikipedia.org/wiki/Lock_\(database\)) is when you lock the record for your exclusive use until you have finished with it. It has much better integrity than optimistic locking but requires you to be careful with your application design to avoid [Deadlocks](https://en.wikipedia.org/wiki/Deadlock_\(computer_science\)). To use pessimistic locking you need either a direct connection to the database (as would typically be the case in a [two tier client server](http://en.wikipedia.org/wiki/Client-server) application) or an externally available transaction ID that can be used independently of the connection.

## 3. **Distributed Transactions**
- **Saga Pattern:**
  - **Question:** "Can you explain the Saga pattern and how it works in distributed systems?"
  - **Answer:** The Saga pattern indeed involves a series of local transactions, each with a corresponding compensation transaction. The orchestrator is responsible for managing the overall workflow and ensuring that if any step fails, it triggers the compensating transactions to roll back the previous steps.

The two main approaches are:

1. **Orchestration:** Here, an orchestrator service coordinates the saga by calling each microservice in sequence and managing the compensations if something fails. 
2. **Choreography:** In this approach, each microservice knows when to perform its local transaction and how to handle compensation, and they communicate through events.

* Two phase commit:

| Aspect           | 2PC            | 3PC              | Saga                  |
| ---------------- | -------------- | ---------------- | --------------------- |
| Consistency      | Strong (ACID)  | Strong           | Eventual              |
| Blocking         | Yes            | No (theoretical) | No                    |
| Availability     | Low            | Medium           | High                  |
| Scalability      | Poor           | Poor             | Excellent             |
| Complexity       | Medium         | High             | High (business logic) |
| Real-world usage | Legacy systems | Rare             | Very common           |


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
