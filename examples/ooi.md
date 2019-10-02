---
layout: default
title: Object-oriented interfaces
show_downloads: false
---
# Iterators

The specifications are based on the following event types:
* `hasNext(b)`: boolean `b` has been returned by `hasNext`;
* `next`: `next` has been called;

## Single iterator

```js
// iterator: single iterator, strong version

// forces best practice:
//   hasNext can only be called once per try
//   hasNext(true) requires next to be called
//   the iterator must be fully consumed

Main = (hasNext(true) next)* hasNext(false);
```
### Exercise
Modify *iterator* above to define a weak version which
verifies only that events of type `next` occur after events of type `hasNext(true)` and
that the result of 'hasNext' can change only after `next`;
multiple consecutive occurrences of events of type `hasNext` are allowed, as long as they are coherent, and
the iterator is not required to be fully consumed.
(see the [solution](solution-iter))

## Multiple iterators

To verify multiple iterators, the following additional event type needs to be defined:
* `newIter(id)`: a new iterator `id` has been created;

```js
// iterators: multiple iterators, strong version

// works with traces generated from iterators.js
// forces best practice: hasNext can only be called once per try, hasNext(id,true) requires next(id) to be called,
// iterators must be fully consumed

Main = {let id; newIter(id)(Iterator<id>|Main)}?;
Iterator<id> = (hasNext(id,true) next(id))* hasNext(id,false);
```

### Exercise
Modify *iterators* above to define a weak version which
verifies only that events of type `next(id)` occur after events of type `hasNext(id,true)` and
that the result of `hasNext(id,res)` can change only after `next(id)`;
multiple consecutive occurrences of events of type `hasNext` are allowed, as long as they are coherent, and
iterators are not required to be fully consumed.
(see the [solution](solution-iters))

*Hint*: a new event type `freeIter(id)` is needed to ensure that iterator `id` has been deallocated and can no longer be used, otherwise the generated
monitor cannot stop verifying the behavior of iterator `id`.

## Multiple iterators over a single list

To verify multiple iterators, the following additional event type needs to be defined:
* `list`: the list has been structurally modified (its lenght has been changed);

```js
// list_iterators: multiple iterators over a single list, weak version

// verifies only that next(id) occurs after hasNext(id,true) and that the result of hasNext(id,res) can change only after next(id)
// multiple consecutive occurrences of hasNext are allowed, as long as they are coherent
// the iterator is not required to be fully consumed

hasNext_or_next(id) matches hasNext(id,_) | next(id);
iterator matches newIter(_) | hasNext_or_next(_) | freeIter(_);
not_newIter not matches newIter(_);
list_or_iter(id) matches hasNext_or_next(id) | freeIter(id) | list;

Main = ListSafe /\ iterator >> Iterators;

ListSafe = not_newIter* {let id; newIter(id)(ListSafeIter<id> /\ ListSafe)}?;
ListSafeIter<id> = list_or_iter(id) >> hasNext_or_next(id)* list* freeIter(id) all;

// iterators specification
Iterators = {let id; newIter(id)(Iterator<id> freeIter(id)|Iterators)}?;
Iterator<id> = ((hasNext(id,true)+ next(id))* hasNext(id,false)+)!;
```
The intersection and filter operators allow a compositional definition:
`Iterators` corresponds to `Main` as defined in [iterators](solution-iters#solution).
