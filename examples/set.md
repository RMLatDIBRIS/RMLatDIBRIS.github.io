---
layout: default
title: Verification of sets
show_downloads: false
---
# Verification of sets

The specifications are based on the following event types:
* `add(el,res)`: function `add` has been called with argment `el` and boolean result `res`
* `rm(el,res)`: function `remove` has been called with argment `el` and boolean result `res`

## Single set with add and remove

```js
rm_false matches rm(_,false); 
not_add_true_rm(el) not matches add(el,true) | rm(el,_);

Main = rm_false* {let el; add(el,true) ((Main | add(el,false)* rm(el,true)) /\ not_add_true_rm(el)* rm(el,true) all)}?;
```