---
layout: default
title: Resource management
show_downloads: false
---
# Resource management

Incorrect resource management in software systems is a typical source of subtle bugs 
that are usually hard to detect and locate, especially in concurrent and distributed applications.

## Non-exclusive access to resources

### Simplified specification

The following specification provides a pattern that can be easily adapted when resources can be accessed
in a non-exclusive way:

```js
// non-exclusive1
Main = {let rid; acquire(rid) (Main | use(rid)* release(rid))}?;
```

The pattern is based on the following event types with the corresponding meanings:
* `acquire(rid)`: resource `rid` has been acquired;
* `use(rid)`: resource `rid` has been used;
* `release(rid)`: resource `rid` has been released.

As expected, the specification is parametric in the resource identifier `rid`; as soon as a resource `rid` is
acquired, the specification is rewritten into a new one where the main specification is interleaved with `use(rid)* release(rid)` to allow the resource to
be used more times (zero included) before it is released.

The specification accepts non-exclusive access to the same resource; for instance, a trace where events match
in the corresponding order `acquire(42)`, `acquire(42)`, `release(42)` and `release(42)`, is accepted.

### Exercise

Which of the following traces is correct (that is, verdict **True** is returned by the monitor) according to the specification `non-exclusive1` above? (see the [solution](solution-non-exclusive-resource.md))

1. trace with events matching in the corresponding order   `acquire(42)` `acquire(42)` `use(42)` `release(42)` `use(42)` `release(42)` 
2. trace with events matching in the corresponding order   `acquire(42)` `acquire(42)` `release(42)` `use(42)` `use(42)` `release(42)` 
3. trace with events matching in the corresponding order   `acquire(42)` `acquire(43)` `use(43)` `release(42)` `use(42)` `release(43)` 
4. trace with events matching in the corresponding order   `acquire(42)` `acquire(42)` `release(42)` `use(42)`  `release(42)` `use(42)` 

### A more precise specification

The specification for non-exclusive access to resources defined above does not
track the entities that access the resources; the more precise specification below uses event types
`acquire(eid,rid)`, `use(eid,rid)` and `release(eid,rid)`, where `eid` is the identity of the entity.

```js
// non-exclusive2
Main = {let eid, rid; acquire(eid,rid) (Main /\ (event(eid,rid) >> use(eid,rid)* release(eid,rid) all))}?;
```
The derived event type `event(eid,rid)` matches any event involving both `eid` and `rid`:

```js
event(eid,rid) matches acquire(eid,rid) | use(eid,rid) | release(eid,rid);
```

Thanks to the combination of the filter and intersection operator, whenever an entity `eid` acquire
a new resource `rid`, the subsequent events involving both `eid` and `rid` have to
meet the specification `use(eid,rid)* release(eid,rid) all`. The `all` operator after `release(eid,rid)`
is used to stop all checks after that `eid` has released `rid`.

### Exercise

Which of the following traces is correct (that is, verdict **True** is returned by the monitor) according to the specification `non-exclusive2` above? (see the [solution](solution-non-exclusive2-resource.md))

1. trace with events matching in the corresponding order   `acquire(0,42)` `acquire(1,42)` `use(1,42)` `release(1,42)` `use(0,42)` `release(0,42)` 
2. trace with events matching in the corresponding order   `acquire(0,42)` `acquire(1,42)` `release(0,42)` `use(1,42)` `use(1,42)` `release(1,42)` 
3. trace with events matching in the corresponding order   `acquire(0,42)` `acquire(0,43)` `use(0,43)` `release(0,42)` `use(0,42)` `release(0,43)` 
4. trace with events matching in the corresponding order   `acquire(0,42)` `acquire(1,42)` `use(1,42)` `use(0,42)`  `release(1,42)` `acquire(0,42)` `release(0,42)`  
