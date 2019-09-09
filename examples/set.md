---
layout: default
title: Verification of sets
show_downloads: false
---
# Verification of sets

The specifications are based on the following event types:
* `add(el,res)`: function `add` has been called with argment `el` and boolean result `res`
* `del(el,res)`: function `delete` has been called with argment `el` and boolean result `res`

## Single set with add and delete

```js
del_false matches del(_,false); 
not_add_true_del(el) not matches add(el,true) | del(el,_);

Main = Set!;
Set = del_false* {let el; add(el,true) ((Set | add(el,false)* del(el,true)) /\ not_add_true_del(el)* del(el,true) all)}?;
```