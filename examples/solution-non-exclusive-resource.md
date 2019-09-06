---
layout: default
title: Resource management
show_downloads: false
---
# Solution
Which of the following traces is correct (that is, verdict **True** is returned by the monitor) according to the specification `non-exclusive1`? 

```js
// non-exclusive1
Main = {let id; acquire(id) (Main | use(id)* release(id))}?;
```

1. trace with events matching in the corresponding order   `acquire(42)` `acquire(42)` `use(42)` `release(42)` `use(42)` `release(42)` 
2. trace with events matching in the corresponding order   `acquire(42)` `acquire(42)` `release(42)` `use(42)` `use(42)` `release(42)` 
3. trace with events matching in the corresponding order   `acquire(42)` `acquire(43)` `use(43)` `release(42)` `use(42)` `release(43)` 
4. trace with events matching in the corresponding order   `acquire(42)` `acquire(42)` `release(42)` `use(42)`  `release(42)` `use(42)` 

Traces 1 and 2 are correct, whereas traces 3 and 4 are not because of the last event 
matching `use(42)`: resource 42 is not currently acquired.
