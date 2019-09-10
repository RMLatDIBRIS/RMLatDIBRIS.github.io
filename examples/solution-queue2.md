---
layout: default
title: Queues
show_downloads: false
---
# Solution

Fix the incorrect version of *queue2* by following the approach 'by decomposition', that is, by imposing a constraint with the intersection operator.

```js
// queue2: incorrect specification!
Main = Queue!; 
Queue = {let val; enq(val) (peek(val)*deq(val) | Queue)}; 
```

The following one is a correct specification:

```js
// queue2: single random queue with enqueue, dequeue and peek
peek_deq matches peek(_) | deq(_);

Main = (Queue/\peek_deq>>Seq)!; 
Queue = {let val; enq(val) (peek(val)*deq(val) | Queue)}; 
Seq = {let val; peek(val)*deq(val)}*;
```

