---
layout: default
title: Resource management
show_downloads: false
---
# Solution

```js
// exclusive
notAcqRel(eid,rid) not matches acquire(_,rid) | release(eid,rid);

Main = {let rid; acquire(eid,rid) ((Main | use(eid,rid)* release(eid,rid)) /\ notAcqRel(rid)* release(eid,rid) all)}?;
```

1. trace with events matching in the corresponding order   `acquire(0,42)` `acquire(1,42)` `use(1,42)` `use(0,42)`  `release(1,42)` `acquire(0,42)` `release(0,42)`  
2. trace with events matching in the corresponding order   `acquire(0,42)` `use(0,42)` `acquire(1,43)` `release(0,42)` `use(1,42)` `release(1,43)` 
3. trace with events matching in the corresponding order   `acquire(0,42)` `use(0,42)` `acquire(0,43)` `use(0,43)` `release(0,42)`  `release(0,43)`
4. trace with events matching in the corresponding order   `acquire(0,42)` `use(0,42)` `release(0,42)` `acquire(1,43)` `use(1,43)` `release(1,43)` 

Traces 3 and 4 are correct, whereas traces 1 and 2 are not:
* in trace 1 entity 1 tries to acquire resource 42 already held by entity 0;
* in trace 2 entity 1 tries to use resource 42 without having acquired it. 
