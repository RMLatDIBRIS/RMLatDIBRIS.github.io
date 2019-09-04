---
layout: default
title: Verification of stacks
show_downloads: false
---
# Verification of stacks and LIFO properties

Stacks are a quite common data type, and system correctness may often depends on LIFO properties;  
for instance, absence of data races in multi-threaded programs can be guaranteed by nested locks, whose implementation follows
the LIFO strategy.

We use the following event types:
* `push(val)`: value `val` has been pushed on the stack;
* `pop(val)`: value `val` has been popped from the stack;
* `size(s)`: `s` has been computed as the size of the stack.	.

## Single stack with push and pop

We start by considering the simple problem of verifying a single stack with just the `push` and `pop` operations.
```js
// stack1: single stack with push and pop

Main = Stack!;
Stack = { let val; push(val) Stack pop(val) }*;
```
The specification assumes that the stack is initially empty; In  `Stack`,  for every event matching `push(val)` a corresponding subsequent event
matching `pop(val)` is expected; the prefix closure operator in `Main` allows the generated monitor to emit the **True** verdict even when
the stack is not empty after the last event of the trace.

## Single stack with push, pop and size


```js
// stack2: single stack with push, pop and size

Main = Stack<0>!; 
Stack<s> = size(s)* { let val; push(val) Stack<s+1> pop(val) Stack<s> }?;
```

## Multiple stacks with push, pop and size

```js
// stacks: multiple stacks with push, pop and size

Main = { let id; new(id) (Main | (Stack<id,0>! free(id))) }?; 
Stack<id,s> = size(id,s)* { let val; push(id,val) Stack<id,s+1> pop(id,val) Stack<id,s> }?;
```