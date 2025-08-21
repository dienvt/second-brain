# Pattern

## 1. Event Notification

don't have value

## 2. Event-carried state transfer

have state

## 3. Event sourcing

Every time we got a change, push it to a **change queue** or something.

When state of system disappear, we can rebuild our system state by using **Change queue**

## 4. CQRS (Command query responsibility segregation)

tách read flow với write flow ra