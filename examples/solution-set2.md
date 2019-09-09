---
layout: default
title: Sets
show_downloads: false
---
# Solution

```js
// set2: single set with add, delete and size

not_add_true_del(el) not matches add(el,true) | del(el,_);

// event types needed for the 'divide et impera' approach 
add(res) matches add(_,res); 
del(res) matches del(_,res); 
not_size not matches size(_);

Main = ((not_size>>Set)/\Size<0>)!;
Set = del(false)* {let el; add(el,true) ((Set | add(el,false)* del(el,true)) /\ not_add_true_del(el)* del(el,true) all)}?;
Size<s> = ((size(s)\/add(false)\/del(false))Size<s>\/add(true)Size<s+1>\/del(true)Size<s-1>)?;
```
