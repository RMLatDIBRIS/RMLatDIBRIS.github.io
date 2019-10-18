---
layout: default
title: Object-oriented interfaces
show_downloads: false
---
Modify [*iterator*](ooi#single-iterator) above to define a weak version which
verifies only that events of type `next` occur immediately after events of type `hasNext(true)` and
that the result of 'hasNext' can change only after `next`;
multiple consecutive occurrences of events of type `hasNext` are allowed, as long as they are coherent and
the iterator is not required to be fully consumed.

# Solution

```js 
// iterator2: single iterator, weak version

// verifies only that next occurs immediately after hasNext(true) and that the result of hasNext can change only after next
// multiple consecutive occurrences of hasNext are allowed, as long as they are coherent
// the iterator is not required to be fully consumed

Main = Iterator!;
Iterator = (hasNext(true)+ next)* hasNext(false)+;
```
The definition exploits the **prefix closure operator** `!` for readability; an equivalent, but more involved, specification
can be given with the union operator.

```js 
Main = hasNext(true)+ (next Main)? \/ hasNext(false)*;
```

