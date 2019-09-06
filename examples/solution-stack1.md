---
layout: default
title: Stacks
show_downloads: false
---
# Solution

Show that the following variation of `stack1` is not correct. 
```js
Main = Stack!;
Stack = { let val; (push(val) Stack pop(val))* };
```
The monitor generated from the specification fails to emit the **True** verdict for the trace
with events matching in the corresponding order   `push(1)` `pop(1)` `push(2)` `pop(2)`. 

The problem is that variable `val` is instantiated with a specific value *v* when the first event of type `push` is matched, therefore
all `push` events occurring when the stack is empty will have to match that value *v*.  