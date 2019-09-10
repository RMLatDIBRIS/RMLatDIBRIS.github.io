---
layout: default
title: Queues
show_downloads: false
---
# Solution

Show that the following extension of *queue1*, that should verify also `peek`, is not correct. 

```js
// queue2: incorrect specification!
Main = Queue!; 
Queue = {let val; enq(val) ( peek(val)*deq(val) | Queue)}; 
```

The monitor generated from the specification emits the **True** verdict for the incorrect trace
with events matching in the corresponding order  `enq(1)` `enq(2)` `peek(1)` `peek(2)`. 
