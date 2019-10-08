---
layout: default
title: Object-oriented interfaces
show_downloads: false
---
Modify [*iterators*](ooi#multiple-iterators) above to define a weak version which
verifies only that events of type `next(id)` occur after events of type `hasNext(id,true)` and
that the result of `hasNext(id,res)` can change only after `next(id)`;
multiple consecutive occurrences of events of type `hasNext` are allowed, as long as they are coherent, and
iterators are not required to be fully consumed.

# Solution

```js 
// iterators2: multiple iterators, weak version

// verifies only that next(id) occurs after hasNext(id,true) and that the result of hasNext(id,res) can change only after next(id)
// multiple consecutive occurrences of hasNext are allowed, as long as they are coherent
// iterators are not required to be fully consumed

Main = {let id; newIter(id)(Iterator<id> freeIter(id)|Main)}?;
Iterator<id> = ((hasNext(id,true)+ next(id))* hasNext(id,false)+)!;
```