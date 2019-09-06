---
layout: default
title: Stacks
show_downloads: false
---
# Solution

Extend the 'divide et impera' version of [specification `stack2`](lifo#divide-et-impera-approach) to verify also the top operation with the corresponding event type:

* `top(val)`: value `val` has been computed as the top of the stack.

```js
// stack3: single stack with push, pop, size, and top with the 'divide et impera' approach

// event types needed for the 'divide et impera' approach 
push matches push(_); 
pop matches pop(_); 
not_size not matches size(_);
not_top not matches top(_);

Main = relevant >>((not_size>>Stack)/\(not_top>>Size<0>))!;
Stack = { let val; push(val) top(val)* Stack top(val)* pop(val) }*;
Size<s> = (size(s) Size<s> \/ pop Size<s-1> \/ push Size<s+1>)?;
```
