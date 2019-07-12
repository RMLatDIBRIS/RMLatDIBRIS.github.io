---
layout: default
title: RML home
show_downloads: false
---
# Welcome to the Runtime Monitoring Language!

* * *

![Logo](logoBW.png)

## A brief introduction
**RML** is a rewriting-based and system agnostic **Domain Specific Language** for **Runtime Verification**,
which decouples monitoring from instrumentation by allowing users to write specifications and
to synthesize monitors from them, independently of the **System Under Scrutiny** and its instrumentation. 

**RML** is more expressive than **Context-Free** grammars, for instance the following specification allows
monitoring of **FIFO** properties:

```
// FIFO queues

enq(val) matches {event:'func_pre',name:'enqueue',args:[val]};
deq(val) matches  {event:'func_post',name:'dequeue',res:val};
deq matches deq(_);

Main = {let val; enq(val) ((deq | Main) /\ (deq >> deq(val) all))}!;
```


