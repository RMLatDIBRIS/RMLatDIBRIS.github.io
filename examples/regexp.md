---
layout: default
title: Regular expressions
show_downloads: false
---
# Multi-line C comments

## Built from a regular expression

```js
// built from regexp /\*/*(([^*/][^*]*)?\*+)*/

star matches {event:'func_pre',name:'star'};
slash matches {event:'func_pre',name:'slash'};
other matches {event:'func_pre',name:'other'};

not_star_slash matches other;
not_star matches slash | other;

Main=slash star slash* ((not_star_slash not_star*)?star+)* slash;
```

## Built from a DFA

```js
// built from  a DFA 

star matches {event:'func_pre',name:'star'};
slash matches {event:'func_pre',name:'slash'};
other matches {event:'func_pre',name:'other'};

not_star_slash matches other;
not_star matches slash | other;

Main=slash star Inner;
Inner=not_star Inner \/ star MayStop;
MayStop=slash \/ star MayStop \/ not_star_slash Inner;
```

## ReDoS

```js
a matches {event:'func_pre',name:'a'};
b matches {event:'func_pre',name:'b'};

Main = (a+)+;
```