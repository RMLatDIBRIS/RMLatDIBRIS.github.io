---
layout: default
title: Verification of queues 
show_downloads: false
---
# Verification of queues and FIFO properties

The specifications are based on the following event types:
* `enq(val)`: value `val` has been inserted in the queue;
* `deq(val)`: value `val` has been removed from the queue;
* `peek(val)`: value `val` is retrieved but not removed from the head of the queue.

## Randomized queues with enqueue and dequeue

```js
// queue1: single random queue with enqueue and dequeue 
Main = Queue!; 
Queue = {let val; enq(val) (deq(val) | Queue)}; 
```

## Randomized queues with enqueue, dequeue and peek

### Exercise
Show that the following extension of *queue1*, that should verify also `peek`, is not correct. (see the [solution](solution-queue1.md))

```js
// queue2: incorrect specification!
Main = Queue!; 
Queue = {let val; enq(val) (peek(val)*deq(val) | Queue)}; 
 ```

### Exercise
Fix the incorrect version of *queue2* by following the approach 'by decomposition', that is, by imposing a constraint with the intersection operator.
(see the [solution](solution-queue2.md))


## Randomized queues with no repetitions, enqueue and dequeue 

```js
// queue3: single random queue with no repetitions, enqueue and dequeue
Main = Queue!; 
Queue = {let val; enq(val) (enq(val)* deq(val) | Queue)}; 
```

## Randomized queues with no repetitions, enqueue, dequeue and peek 

### Exercise
Extend specification *queue3* to verify `peek`.
(see the [solution](solution-queue3.md))

## FIFO queues with enqueue and dequeue 

```js
deq matches deq(_);

Main = Queue!; 
Queue={let val; enq(val) ((deq|Queue)/\(deq>>deq(val) all))};
```

## FIFO queues with no repetitions, enqueue and dequeue 

```js
deq matches deq(_);

Main = Queue!; 
Queue={let val; enq(val) ((enq(val)* deq|Queue)/\(deq>>deq(val) all))};
```

