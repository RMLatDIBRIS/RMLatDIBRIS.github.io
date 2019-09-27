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
// iterator: single iterator
// strong version, forces best practice, hasNext can only be called once per try
// iterator must be fully consumed

Main = (hasNext(true) next)* hasNext(false);
```
### Exercise
Modify *iterator* above to define a weaker version which does not require the iterator to be fully consumed. (see the [solution](solution-iter1))

### Exercise
Modify *iterator* above to define yet another weaker version which
verifies only that events of type `next` occurs after events of type `hasNext(true)`, while multiple consecutive occurrences of events of type `hasNext` are allowed, as long as they are coherent.
(see the [solution](solution-iter2))

## Multiple iterators

To verify multiple iterators, the following additional event type needs to be defined:
* `iterator`: a new iterator has been created;

```js
// iterators: multiple iterators
// strong version, forces best practice, hasNext can only be called once per try
// iterators must be fully consumed

Main = {let id; iterator(id)(Iterator<id>|Main)}?;
Iterator<id> = (hasNext(id,true) next(id))* hasNext(id,false);
```

### Exercise
Modify *iterators* above to define a weaker version which
verifies only that events of type `next` occurs after events of type `hasNext(true)`, while multiple consecutive occurrences of events of type `hasNext` are allowed, as long as they are coherent.
(see the [solution](solution-iter3))

*Hint*: a new event type `free(id)` is needed to ensure that an iterator has been deallocated and can no longer be used, otherwise the generated
monitor cannot stop verifying its behavior. 