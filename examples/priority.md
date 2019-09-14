---
layout: default
title: Verification of priority queues 
show_downloads: false
---
# Verification of priority queues

The specifications are based on the following event types:
* `enq(val)`: value `val` has been inserted in the queue;
* `deq(val)`: value `val` has been removed from the queue;
* `peek(val)`: value `val` is retrieved but not removed from the head of the queue.

## Priority queues with no repetitions, enqueue and dequeue 

```js
// priority-queue1: single priority queue with no repetitions, enqueue and dequeue 

deq_geq(val) matches deq(val2) with val2 >= val;
deq matches deq(_);

Main = (Queue/\Sorted)!;
Queue = {let val; enq(val) (enq(val)* deq(val) | Queue)}; 
Sorted = {let val; enq(val) ((deq_geq(val) >> deq(val) all) /\ Sorted)} \/ (deq Sorted);
```

## Priority queues with repetitions, enqueue and dequeue 

```js
// priority-queue2: single priority queue with repetitions, enqueue and dequeue 

deq_geq(val) matches deq(val2) with val2 >= val;
enq_deq_geq(val) matches enq(val) | deq_geq(val); 
deq matches deq(_);


Main = (Queue/\Sorted)!;
Queue = {let val; enq(val) (deq(val) | Queue)}; 
Sorted = {let val; enq(val) ((enq_deq_geq(val) >> Cons<val> all) /\ Sorted)} \/ (deq Sorted);
Cons<val> = deq(val) \/ (enq(val) (deq(val) | Cons<val>));
```