---
layout: default
title: Object-oriented interfaces
show_downloads: false
---
Modify [*iterators*](ooi#multiple-iterators) above to define yet another weaker version which
verifies only that events of type `next` occurs after events of type `hasNext(true)`, while multiple consecutive occurrences of events of type `hasNext` are allowed, as long as they are coherent.

# Solution

```js 
Main = {let id; iterator(id)(It<id> free(id)|Main)}?;
It<id> = (hasNext(id,true)+ next(id))* hasNext(id,false)+;
```