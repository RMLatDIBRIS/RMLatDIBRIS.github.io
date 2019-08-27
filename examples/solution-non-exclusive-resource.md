---
layout: default
title: Resource management
show_downloads: false
---
# Solution

```js
Main = {let id; acquire(id) (Main | use(id)* release(id))}?;
```

1. six events respectively matching  `acquire(42)` `acquire(42)` `use(42)` `release(42)` `use(42)` `release(42)` 
2. six events respectively matching  `acquire(42)` `acquire(42)` `release(42)` `use(42)` `use(42)` `release(42)` 
3. six events respectively matching  `acquire(42)` `acquire(43)` `use(43)` `release(42)` `use(42)` `release(43)` 
4. six events respectively matching  `acquire(42)` `acquire(42)` `release(42)` `use(42)`  `release(42)` `use(42)`

Traces 1 and 2 are correct, whereas traces 3 and 4 are not because of the last event 
matching `use(42)`: resource 42 is not currently acquired.
