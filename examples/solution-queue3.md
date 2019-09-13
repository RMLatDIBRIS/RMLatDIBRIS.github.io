---
layout: default
title: Queues
show_downloads: false
---
# Solution
Extend [specification *queue3*](fifo#randomized-queues-with-no-repetitions-enqueue-and-dequeue) to verify `peek`.

```js
// queue4: single random queue with no repetitions, enqueue, dequeue and peek

peek_deq matches peek(_) | deq(_);

Main = (Queue/\peek_deq>>Seq)!; 
Queue = {let val; enq(val) ((enq(val)\/peek(val))* deq(val) | Queue)}; 
Seq = {let val; peek(val)*deq(val)}*;
```

