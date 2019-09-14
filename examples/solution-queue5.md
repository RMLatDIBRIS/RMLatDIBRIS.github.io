---
layout: default
title: Queues
show_downloads: false
---
# Solution

Extend specification [*queue3*](fifo#randomized-queues-with-no-repetitions-enqueue-and-dequeue) to verify FIFO queues with no repetitions, `enqueue` and `dequeue`.

```js
// queue7
// single FIFO queue with no repetitions, enqueue and dequeue

deq matches deq(_);

Main = Queue!; 
Queue={let val; enq(val) ((enq(val)* deq|Queue)/\(deq>>deq(val) all))};
```

