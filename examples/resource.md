---
layout: default
title: Resource management verification
show_downloads: false
---
# Resource management verification

Incorrect resource management in software systems is a typical source of subtle bugs 
that are usually hard to detect and locate, especially in concurrent and distributed applications.
This is a typical control-oriented verification problem that can be managed with **RV**.

## Non-exclusive access to resources

### Simplified specification

The following specification provides a pattern that can be easily adapted when resources can be accessed
in a non-exclusive way:

```rml
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

Which of the following traces is correct (that is, verdict **True** is returned by the monitor) according to the specification *non-exclusive1* [above](#simplified-specification)? (see the [solution](solution-non-exclusive-resource.md))

1. trace with events matching in the corresponding order   `acquire(42)` `acquire(42)` `use(42)` `release(42)` `use(42)` `release(42)` 
2. trace with events matching in the corresponding order   `acquire(42)` `acquire(42)` `release(42)` `use(42)` `use(42)` `release(42)` 
3. trace with events matching in the corresponding order   `acquire(42)` `acquire(43)` `use(43)` `release(42)` `use(42)` `release(43)` 
4. trace with events matching in the corresponding order   `acquire(42)` `acquire(42)` `release(42)` `use(42)`  `release(42)` `use(42)` 

### A more precise specification 

The specification for non-exclusive access to resources defined above does not
track the entities that access the resources; the more precise specification below uses event types
`acquire(eid,rid)`, `use(eid,rid)` and `release(eid,rid)`, where `eid` is the identity of the entity interacting with
resource `rid`.

```js
// non-exclusive2
notAcqRel(eid,rid) not matches acquire(eid,rid) | release(eid,rid);

Main = {let eid,rid; acquire(eid,rid) ((Main | use(eid,rid)* release(eid,rid)) /\ notAcqRel(eid,rid)* release(eid,rid) all)}?;
```
The derived event type `notAcqRel(eid,rid)` matches any event which does not match `acquire(eid,rid)` or `release(eid,rid)`.

The intersection operator imposes the further constraint that entity `eid` can acquire resource `rid` only if
it does not hold it already; this implies that an entity can reacquire a resource only after it has
released it. This is possible thanks to the `all` operator following `release(eid,rid)`.

### Exercise

Which of the following traces is correct (that is, verdict **True** is returned by the monitor) according to the specification *non-exclusive2* [above](#a-more-precise-specification)? (see the [solution](solution-non-exclusive2-resource.md))

1. trace with events matching in the corresponding order   `acquire(0,42)` `acquire(1,42)` `use(1,42)` `release(1,42)` `use(0,42)` `release(0,42)` 
2. trace with events matching in the corresponding order   `acquire(0,42)` `acquire(1,42)` `release(0,42)` `use(1,42)` `use(1,42)` `release(1,42)` 
3. trace with events matching in the corresponding order   `acquire(0,42)` `acquire(0,43)` `use(0,43)` `release(0,42)` `use(0,42)` `release(0,43)` 
4. trace with events matching in the corresponding order   `acquire(0,42)` `acquire(1,42)` `use(1,42)` `use(0,42)`  `release(1,42)` `acquire(0,42)` `release(0,42)`  

## Exclusive access to resources

Mutually exclusive access to resources can be imposed by slightly changing the specification *non-exclusive2* [above](#a-more-precise-specification);
we just need to modify the definition of event type `notAcqRel(eid,rid)`, to forbid acquisition of resource `rid` by any entity,
and not just `eid`. In this way the specification on the right-hand-side of intersection ensures that acquisition of `rid` is allowed
only after `rid` has been released by `eid`. This is possible thanks to the `all` operator following `release(eid,rid)`.


```js
// exclusive
notAcqRel(eid,rid) not matches acquire(_,rid) | release(eid,rid);

Main = {let eid,rid; acquire(eid,rid) ((Main | use(eid,rid)* release(eid,rid)) /\ notAcqRel(eid,rid)* release(eid,rid) all)}?;
```

### Exercise

Which of the following traces is correct (that is, verdict **True** is returned by the monitor) according to the specification *exclusive* [above](#exclusive-access-to-resources)? (see the [solution](solution-exclusive-resource.md))

1. trace with events matching in the corresponding order   `acquire(0,42)` `acquire(1,42)` `use(1,42)` `use(0,42)`  `release(1,42)` `acquire(0,42)` `release(0,42)`  
2. trace with events matching in the corresponding order   `acquire(0,42)` `use(0,42)` `acquire(1,43)` `release(0,42)` `use(1,42)` `release(1,43)` 
3. trace with events matching in the corresponding order   `acquire(0,42)` `use(0,42)` `acquire(0,43)` `use(0,43)` `release(0,42)`  `release(0,43)`
4. trace with events matching in the corresponding order   `acquire(0,42)` `use(0,42)` `release(0,42)` `acquire(1,43)` `use(1,43)` `release(1,43)` 
