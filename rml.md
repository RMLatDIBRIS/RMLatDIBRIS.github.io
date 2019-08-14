---
layout: default
title: An introductory guide to **RML**
show_downloads: false
---
# An introductory guide to **RML**

**RML** is a rewriting-based and system agnostic **Domain Specific Language** for **Runtime Verification**
which decouples monitoring from instrumentation by allowing users to write specifications and
to synthesize monitors from them, independently of the **System Under Scrutiny** (**SUS**) and its instrumentation. 

**RML** is more expressive than **Context-Free** grammars, for instance the following specification allows
monitoring of **FIFO** properties:

```js
// FIFO queues

enq(val) matches {event:'func_pre',name:'enqueue',args:[val]};
deq(val) matches {event:'func_post',name:'dequeue',res:val};
deq matches deq(_);

Main = {let val; enq(val) ((deq | Main) /\ (deq >> deq(val) all))}!;
```

## Events
**RML** is based on a general model where events are represented by object literals, with
the standard **JavaScript** syntax. Consequently, monitors generated from
**RML** specifications process events with the universal data-interchange format **JSON**.

Specific event models can be conceived, depending on the kinds of properties events can have.
For instance, the specification of **FIFO** queues above is based on a simple model consisting of the
following properties keys:
- `event`, specifying two possible kinds of events: 'func_pre' and 'func_post', corresponding,
to *function calls* and *returns from functions*, respectively;
-  `name`, specifying the name of the involved function;
-  `args`, specifying the array of the arguments passed to the function;
-  `res`, specifying the returned value, if `event` has value 'func_post'.

As an example, the following objects

```js
{event:'func_pre',name:'enqueue',args:[val]}

{event:'func_post',name:'dequeue',args:[],res:val}
```
specify the following events, respectively:
- function `enqueue` has been called with a single argument of value `val`;
- function `dequeue` has been called with no arguments, and returned value `val`.


## Event types
Event types define  sets of events and coincide with what is often referred as symbolic events in the **RV** jargon.
In **RML**  event types are terms built on top of names, associated with arities, and subterms representing data values of primitive, array, or object type;
the simplest way to define event types is by pattern matching, as in the following example:
```js
enq(val) matches {event:'func_pre',name:'enqueue',args:[val]};
deq(val) matches {event:'func_post',name:'dequeue',res:val};
```  

It is also possible to define derived event types, as below: 
```js
enq matches enq(_);
deq matches deq(_);
```

Four event types are defined above: `enq` and `deq`, each with both arity 0 and 1; event types can be overloaded: the same name can be
used for event types with different arities; we explicitly indicate the arity to distinguish overloaded event types as in `enq/0` and `enq/1`.
In this example the definitions for  `enq/0` and `deq/0` are derived from those of `enq/1` and `deq/1`, respectively.

The meaning of the definitions above is as follows:
- `enq(val)` matches all calls to function `enqueue` with single argument `val`;
- `enq` matches all events matching `enq(val)`, for any `val`;
- `deq(val)` matches all returns from function `dequeue` with result `val`
- `deq` matches all events matching `deq(val)`, for any `val`.

Event type definitions support also **negation**; furthermore, the **predefined event types** `none` and `any`
can be used to match no event and all events, respectively.

Event types favor modular and reusable specifications and enhance readability;
for instance, the specification of queues above can be easily adapted if one has to verify a library where the operations for enqueueing and dequeing
use different names, by only changing the definitions of `enq(val)` and `deq(val)`:

```js
enq(val) matches {event:'func_pre',name:'add',args:[val]};
deq(val) matches {event:'func_post',name:'remove',res:val};
```  

## Basic and derived operators
An **RML** specification denotes a set of event traces, obtained by combining simpler sets with the following basic binary operators (in
decreasing order of precedence):
- concatenation (juxtaposition), to force sequentiality; 
- intersection (`/\`), to ensure that event traces satisfy simultaneously different properties;  
- union (`\/`), to express alternatives;
- shuffle (`|`), to allow interleaving of events in traces. 

Specifications can be (mutually) recursive, and the keyword `empty` is the basic constant operator
denoting the set containing just the empty trace.

Thanks to recursion and the basic operators above, several derived operators can be defined.

Standard postfix operators `?`, `+` and `*` are borrowed from regular expressions: 
for any expression `exp`, `(exp)?` is equivalent to `empty \/ (exp)`,
while `(exp)*` and `(exp)+` correspond to the following specifications `Star` and `Plus`, respectively:
```js
Star = empty \/ (exp) Star  // exp*
Plus = (exp) Star  // exp+
```
### More on intersection and shuffle
Before introducing other more advanced derived operators, let us focus on the two basic intersection and
shuffle operators; indeed, while we are all familiar with concatenation and union, which are both used in
regular expressions and context-free grammars, these latter operators deserve more attention to
understand why they are useful in **RV**.

For intersection, let us consider the following two simple properties on event traces:
1. for any event `e_1` matching `open` there must be a subsequent event `e_2` matching `close` and no other
event matching `open` is allowed to occur between `e_1` and `e_2`;
      2. for any event `e_1` matching `alloc` there must be a subsequent event `e_2` matching `dealloc` and no other
event matching `alloc` is allowed to occur between `e_1` and `e_2`.

If `noc` (shorthand for 'not open and close') and `nad` (shorthand for 'not alloc and dealloc') are event types defining all events not matching `open` and `close`, and `alloc` and `dealloc`,
respectively, then the two properties above can be specified as follows:

```js
(noc* (open noc* close)?)*    // property 1

(nad* (alloc nad* dealloc)?)* // property 2
```
Now, what can we do if we need to specify traces that have to verify both properties 1 and 2? The intersection operator
comes to our aid, by allowing us to combine the two properties above in a compositional way to get the desired
specification:
```js
(noc* (open noc* close)?)*  /\  (nad* (alloc nad* dealloc)?)* // both property 1 and 2
```

Property 1 and 2 above have been specified with standard regular expressions; however, it takes some time to parse and
interpret them correctly. Shuffle allows us to specify such properties in a more direct way:

```js
(open close)* | noc*	// property 1

(alloc dealloc)* | nad* // property 2
```
Shuffle can be used to express concisely that any event trace in `(open close)*` can be interleaved with any trace in `noc*` and
any event trace in `(alloc dealloc)*` can be interleaved with any trace in `nad*`. Again, if we need both properties to be verified
by all traces, then we can combine the specifications above with the intersection operator:

```js
((open close)* | noc*) /\ ((alloc dealloc)* | nad*) // property 1 and 2
```

### Filter operator
The derived filter operator `>>` is useful for restricting verification to events that match a certain type; for instance, the previous specification
can be written in a even more readable way:

```js
(oc >> (open close)*) /\ (ad >> (alloc dealloc)*) // property 1 and 2
```

Event types `oc`and `ad` are the complement of `noc` and `nad`, respectively:
an event matches `oc` only when it matches either `open` or `close`,
an event matches `ad` only when it matches either `alloc` or `dealloc`. The specification
reads as follows: events matching `oc` have to satisfy `(open close)*`, and events matching
`ad` have to satisfy `(alloc dealloc)*`.

### Derived operators and monitor verdicts 

Any time it receives a new event, the monitor generated from an **RML** specification emits a verdict in a 4-valued logic:
- **True**: the event trace monitored so far is correct, and any continuation will be correct as well;  
- **Possibly true**: the event trace monitored so far is correct, but some of its continuations are not correct; 
- **Possibly false**: the event trace monitored so far is not correct, but some of its continuations are correct;  
- **False**: the event trace monitored so far is not correct, and any continuation will be incorrect as well.

The first and last values are called *conclusive* since in those cases no other monitoring is required
to decide whether the **SUS** is correct or not; the others are *inconclusive* and
require the monitor to keep running.

