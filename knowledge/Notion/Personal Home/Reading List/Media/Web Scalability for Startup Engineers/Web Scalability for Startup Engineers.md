---
Status: Finished
Author:
  - Atur Ejsmont
Type: Book
"\bGenre":
  - IT
  - Nonfiction
---
# Chapter 7: Asynchronous Processing

Knowledge about synchronous and asynchronous procession

Concepts about producer, consumer and message queue/broker

Event-Driven Architecture

## Compare Request/Response interaction vs Direct Worker Queue Interaction vs Event-Based interaction

### Request/Response interaction

synchonous

high coupling than others

### Direct Worker Queue Interaction

asynchronous

The publisher publish to and queue that publisher know exactly how message will be treated

publish an event in to queue like OrderProcessingQueue

Have opportunities for closer coupling

### Event-Based interaction

asynchronous

The publisher has no idea about consumer

publish an event like NewOrderCreated

very low coupling

# Chapter 9 Other Dimension

## Scale Yourself

### 80/20 principle

- 80% of features can be build in 20% of the overall time
- 80% of code coverage can be achieve in 20% of the overall time
- 80% of user only use 20% of the feature
- 80% of documentation value is in 20% of its text
- 80% of the bugs come from 20% of the code
- 80% of the code changes are made in 20% of the codebase.

## Scale Agile Team