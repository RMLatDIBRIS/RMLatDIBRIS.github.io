---
layout: default
title: Verification of sets
show_downloads: false
---
# Verification of sets

The specifications are based on the following event types:
* `add(el,res)`: element `el` has been added to the set, with computed boolean result `res`: `true` if `el` was not in the set, `false` otherwise;
* `del(el,res)`: element `el` has been deleted from the set, with computed boolean result `res`: `true` if `el` was in the set, `false` otherwise;
* `size(s)`: `s` has been computed as the size of the set.	

## Single set with add and delete

```js
// set1: single set with add and delete
del_false matches del(_,false); 
not_add_true_del(el) not matches add(el,true) | del(el,_);

Main = Set!;
Set = del_false* {let el; add(el,true) ((Set | add(el,false)* del(el,true)) /\ not_add_true_del(el)* del(el,true) all)}?;
```
### Exercise
Show that specification *set1* is in fact an extension of the pattern for [exclusive access to resources](resource#exclusive-access-to-resources),
restricted to the case 'single entity', since it verifies a single set. (see the [solution](solution-set1.md))

## Single set with add, delete and size
To verify also the `size` operation we need to introduce the state variable `s` with a generic specification, to track the size of the set,
as also done for [stacks](lifo#single-stack-with-push-pop-and-size).

### Exercise
Extend version *set1* of the specification of sets, to verify also `size` by following the approach 'by decomposition' as done
for [stacks](lifo#by-decomposition-approach). (see the [solution](solution-set2.md))
