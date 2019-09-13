---
layout: default
title: Queues
show_downloads: false
---
# Solution
Extend specification [*queue5*](fifo#fifo-queues-with-enqueue-and-dequeue) to verify `peek`.

```js
// queue6: single FIFO queue with enqueue, dequeue and peek

deq matches deq(_);
peek matches peek(_);
peek_deq matches peek | deq;

Main = Queue!; 
Queue = {let val; enq(val) ((peek* deq|Queue)/\(peek_deq>>peek(val)* deq(val) all))};
```

