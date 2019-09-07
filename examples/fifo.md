---
layout: default
title: Verification of queues 
show_downloads: false
---
# Verification of queues and FIFO properties

The specifications are based on the following event types:
* `enq(val)`: value `val` has been inserted in the queue;
* `deq(val)`: value `val` has been removed from the queue.

## Randomized queues

```js
Main = Queue!; 
Queue = {let val; enq(val) ( deq(val) | Queue)}; 
```

## Randomized queues with no repetitions

```js
Main = Queue!; 
Queue = {let val; enq(val) (enq(val)* deq(val) | Queue)}; 
```

## FIFO queues

```js
deq matches deq(_);

Main = Queue!; 
Queue={let val; enq(val) ((deq|Queue)/\(deq>>deq(val) all))};
```

## FIFO queues with no repetitions

```js
deq matches deq(_);

Main = Queue!; 
Queue={let val; enq(val) ((enq(val)* deq|Queue)/\(deq>>deq(val) all))};
```

