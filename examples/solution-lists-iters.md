---
layout: default
title: Object-oriented interfaces
show_downloads: false
---
Modify [*list_iterators*](ooi#multiple-iterators-over-a-single-list) above to specify multiple iterators over multiple lists (weak version).
(see the [solution](solution-lists-iters))

# Solution

```js 
// lists_iterators: multiple iterators over multiple lists, weak version

newIter(id) matches newIter(_,id);
hasNext_or_next(id) matches hasNext(id,_) | next(id);
iterator matches newIter(_) | hasNext_or_next(_) | freeIter(_);
not_newIter not matches newIter(_);
list_or_iter(lsid,itid) matches hasNext_or_next(itid) | freeIter(itid) | list(lsid);

Main = ListSafe /\ iterator >> Iterators;

ListSafe = not_newIter* {let lsid,itid; newIter(lsid,itid)(ListSafeIter<lsid,itid> /\ ListSafe)}?;
ListSafeIter<lsid,itid> = list_or_iter(lsid,itid) >> hasNext_or_next(itid)* list(lsid)* freeIter(itid) all;

// specification already defined
Iterators = {let id; newIter(id)(Iterator<id> freeIter(id)|Iterators)}?;
Iterator<id> = ((hasNext(id,true)+ next(id))* hasNext(id,false)+)!;
```