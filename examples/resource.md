---
layout: default
title: Resource management
show_downloads: false
---
# Resource management

Correct resource management is a typical issue of software systems and failures can lead to subtle bugs 
hard to detect, especially in concurrent and distributed applications.

## Non-exclusive access to resources

The following specification provides a pattern that can be easily adapted when resources can be accessed
in a non-exclusive way:

```js
Main = {let id; acquire(id) (Main | use(id)* release(id))}?;
```

The pattern is based on the following event types with the corresponding meanings:
* `acquire(id)`: resource identified by `id` has been acquired;
* `use(id)`: resource identified by 'id' has been used;
* `release(id)`: resource identified by 'id' has been released.

As expected, the specification is parametric in the identifier `id` of the resource; as long as a recourse `id` is
acquired, the main specification is recursively interleaved with `use(id)* release(id)` to allow the resource to
be used more times (zero included) before releasing it.

The specification accepts non-exclusive access to the same resource; for instance, a trace where events match
`acquire(42)`, `acquire(42)`, `release(42)` and `release(42)`, respectively, is accepted.

### Exercise

Which of the following traces is correct (that is, verdict **True** is returned by the monitor) according to the specification above? (see the [solution](solution-non-exclusive-resource.md))

1. six events respectively matching  `acquire(42)` `acquire(42)` `use(42)` `release(42)` `use(42)` `release(42)` 
2. six events respectively matching  `acquire(42)` `acquire(42)` `release(42)` `use(42)` `use(42)` `release(42)` 
3. six events respectively matching  `acquire(42)` `acquire(43)` `use(43)` `release(42)` `use(42)` `release(43)` 
4. six events respectively matching  `acquire(42)` `acquire(42)` `release(42)` `use(42)`  `release(42)` `use(42)`

