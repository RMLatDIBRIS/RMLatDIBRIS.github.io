---
layout: default
title: Resource management
show_downloads: false
---
# Solution

Which of the following traces is correct (that is, verdict **True** is returned by the monitor) according to the specification `non-exclusive2`?

```js
// non-exclusive2
notAcqRel(eid,rid) not matches acquire(eid,rid) | release(eid,rid);

Main = {let eid,rid; acquire(eid,rid) ((Main | use(eid,rid)* release(eid,rid)) /\ notAcqRel(eid,rid)* release(eid,rid) all)}?;
```

1. trace with events matching in the corresponding order   `acquire(0,42)` `acquire(1,42)` `use(1,42)` `release(1,42)` `use(0,42)` `release(0,42)` 
2. trace with events matching in the corresponding order   `acquire(0,42)` `acquire(1,42)` `release(0,42)` `use(1,42)` `use(1,42)` `release(1,42)` 
3. trace with events matching in the corresponding order   `acquire(0,42)` `acquire(0,43)` `use(0,43)` `release(0,42)` `use(0,42)` `release(0,43)` 
4. trace with events matching in the corresponding order   `acquire(0,42)` `acquire(1,42)` `use(1,42)` `use(0,42)`  `release(1,42)` `acquire(0,42)` `release(0,42)`  

Traces 1 and 2 are correct, whereas traces 3 and 4 are not:
* in trace 3 entity 0 tries to use resource 42 after it has released it;
* in trace 4 entity 0 tries to acquire the already acquired resource 42. 
