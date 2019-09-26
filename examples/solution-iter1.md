---
layout: default
title: Object-oriented interfaces
show_downloads: false
---
Modify [`iterator`](ooi#iterators) above to define a weaker vesion which does not require the iterator to be fully consumed.

# Solution

```js 
Main = (hasNext(true) next)* hasNext(false)?;
```