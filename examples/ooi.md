---
layout: default
title: Object-oriented interfaces
show_downloads: false
---
# Iterators

The specifications are based on the following event types:
* `hasNext()`: boolean `b` has been returned by `hasNext`;
* `next`: `next` has been called;

```js
// iterator: single iterator
// strong version, forces best practice, hasNext can only be called once per try
// iterator must be fully consumed

Main = (hasNext(true) next)* hasNext(false);
```
### Exercise
Modify `iterator`above to define a weaker vesion which does not require the iterator to be fully consumed. (see the [solution](solution-iter1))


