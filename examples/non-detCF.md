---
layout: default
title: Non-deterministic Context Free examples
show_downloads: false
---
## Language a^k b^h c^j with k=h or h=j and k,h,j>0

```js
a matches {event:'func_pre',name:'a'};
b matches {event:'func_pre',name:'b'};
c matches {event:'func_pre',name:'c'};

Main = a A<0>;
A<n> = a A<n+1> \/ b B<0,n>;
B<n,k> = b B<n+1,k-1> \/ c if(k==0) c* else C<n>;
C<n> = if(n>0) c C<n-1> else empty;
```
