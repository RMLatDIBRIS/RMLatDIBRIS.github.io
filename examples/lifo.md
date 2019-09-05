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
matching `pop(val)` is expected; therefore `push` and `pop` events must be balanced. The prefix closure operator in `Main` allows the generated monitor to emit the **True** verdict even when the stack is not empty after the last event of the trace.

The `*` operator allows a more compact version for the definition of `Stack`; without the Kleene start we would need  the following version:
```js
Stack = { let val; push(val) Stack pop(val) Stack }?;
```
### Exercise
Show that the following variation of `stack1` is not correct.
```js
Main = Stack!;
Stack = { let val; (push(val) Stack pop(val))* };
```

## Single stack with push, pop and size
Let us now consider a more elaborated example where the specification has to verify also the `size` operation;
to manage this, we need to introduce the state variable `s` with a generic specification, to track the size of the stack.
If we follow the pattern of specification `stack1` we get a rather compact specification which is, however, not so readable.

```js
// stack2: single stack with push, pop and size

Main = Stack<0>!;
Stack<s> = size(s)* { let val; push(val)  Stack<s+1> pop(val) size(s)* }*;
```
The definition of `Main` is pretty clear: now `Stack` is generic and, therefore, has to be applied
to the value 0, since the specification assumes that the stack is initially empty.
The part that concerns us is the definition of `Stack<s>`;  perhaps, a more readable definition for `Stack<s>` is
```js
Stack<s> = size(s)* { let val; push(val) Stack<s+1> pop(val) Stack<s> }?;
```
This version makes more explicit how the state variable `s` changes in reaction to the events; this is made more evident in the
following version:
```js
Stack<s> = size(s) Stack<s> \/ { let val; push(val) Stack<s+1> pop(val) Stack<s> }?;
```

However, even this last version suffers from a problem: the pattern does not scale well
when new kinds of events have to be verified; let us consider, for instance, to extend
the specification to check also the `top` operation (see the exercise below).
In this case, a pattern based on a `divide et impera` approach helps to solve this problem:
with the use of the filter and intersection operators we can divide the verification problem
into simpler sub-problems.

```js
// stack2: single stack with push, pop and size
// divide et impera approach

Main = ((not_size>>Stack) /\ Size<0>)!;
Stack = { let val; push(val) Stack pop(val) }*;
Size<s> = (size(s) Size<s> \/ pop Size<s-1> \/ push Size<s+1>)?;
```
`Stack` now coincides with the definition given for `stack1` and concerns verification of the events matching
`push` or `pop` only; for this reason, the filter with event type `not_size` is needed in `Main`.

The new generic `Size` verifies events of type `size`, but also events of type `push` and `pop` must be involved,
because the size of the stack depends on `push` and `pop`; however, in this case tracking the pushed/popped values
is not needed.

The specification requires the definition of the following derived event types:

```js
push matches push(_); 
pop matches pop(_); 
not_size not matches size(_);
```
Despite its verbosity, this pattern favors extension of the specification (see the exercise below). 

## Multiple stacks with push, pop and size

```js
// stacks: multiple stacks with push, pop and size

Main = { let id; new(id) (Main | (Stack<id,0>! free(id))) }?; 
Stack<id,s> = size(id,s)* { let val; push(id,val) Stack<id,s+1> pop(id,val) Stack<id,s> }?;
```