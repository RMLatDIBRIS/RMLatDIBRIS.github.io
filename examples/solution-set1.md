---
layout: default
title: Sets
show_downloads: false
---
# Solution
For convenience the two specifications are copied below:

```js
// exclusive
notAcqRel(eid,rid) not matches acquire(_,rid) | release(eid,rid);

Main = {let eid,rid; acquire(eid,rid) ((Main | use(eid,rid)* release(eid,rid)) /\ notAcqRel(eid,rid)* release(eid,rid) all)}?;
```

```js
// set1: single set with add and delete
del_false matches del(_,false); 
not_add_true_del(el) not matches add(el,true) | del(el,_);

Main = Set!;
Set = del_false* {let el; add(el,true) ((Set | add(el,false)* del(el,true)) /\ not_add_true_del(el)* del(el,true) all)}?;
```

The similarity between the two specifications can be outlined in terms of event types:
* `add(el,true)` corresponds to `acquire(_,el)`
* `add(el,false)` corresponds to `use(_,el)`
* `del(el,true)` corresponds to `release(_,el)`
* `not_add_true_del(el)` corresponds to `notAcqRel(_,el)`

There are, however, two differences:

* specification *set1* uses the `!` operator, since in general a program is considered correct if it terminates with a non-empty set, but not
if it has not released some acquired resource;
* specification *set1* verifies also events of type `del_false`; in terms of resource management, this could correspond in verifying
also the events of type 'acquisition of the resource has been negated'.
