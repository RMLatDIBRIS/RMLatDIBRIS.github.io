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

Main = {let val; enq(val) ((deq | Main) /\ (deq >> (deq(val) all)))}!;
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
- **Possibly True**: the event trace monitored so far is correct, but some of its continuations are not correct; 
- **Possibly False**: the event trace monitored so far is not correct, but some of its continuations are correct;  
- **False**: the event trace monitored so far is not correct, and any continuation will be incorrect as well.

The first and last values are called *conclusive* since in those cases no other monitoring is required
to decide whether the **SUS** is correct or not; the others are *inconclusive* and
require the monitor to keep running.

The derived constant `all`  corresponds to the specification `any*` denoting the set of all possible event traces; such a constant
is useful to allow monitors to emit **True** as verdict.

Let us consider, for instance, `deq >> (deq(val) all)` which is
part of the specification for **FIFO** properties as defined at the beginning of this page; according to the filter operator,
traces restricted to events matching type `deq` must verify `deq(val) all`, that is, they must start with an event matching
`deq(val)` and then can continue with any possible trace: after checking that the initial event matches `deq(val)`, the monitor can emit
the **True** verdict. Therefore, the specification `deq >> (deq(val) all)` defines the following constraint: the first dequeue operation, if any, must return
`val` as value.

Another useful operator for driving monitor verdicts is the **prefix closure operator** `!`; let us consider again the specification for
**FIFO** queues:
```js
// FIFO queues

enq(val) matches {event:'func_pre',name:'enqueue',args:[val]};
deq(val) matches {event:'func_post',name:'dequeue',res:val};
deq matches deq(_);

Main = {let val; enq(val) ((deq | Main) /\ (deq >> (deq(val) all)))};
```
Such a specification does not employ the constant `empty`, therefore it denotes a set where all traces are
necessarily infinite; this means that the only verdicts that can be emitted are **False** or **Possibly False**.
What can we do to allow the monitor to emit also **Possibly True** verdicts?
We need to specify that also all finite prefixes of the traces defined by the specification above are allowed;
the simplest way to do that is to employ the post-fix operator `!`:
```js
// FIFO queues

enq(val) matches {event:'func_pre',name:'enqueue',args:[val]};
deq(val) matches {event:'func_post',name:'dequeue',res:val};
deq matches deq(_);

Main = {let val; enq(val) ((deq | Main) /\ (deq >> (deq(val) all)))}!;
```
At this point, another question could arise: 'ok, now **Possibly True** verdicts can be emitted, but what about **True**?';
let us recall the meaning of the **True** verdict: the event trace monitored so far is correct, and any continuation will be correct as well.
Can this happen when monitoring a program which manipulates a queue? Not really! The best we can get is
**Possibly True**: the event trace monitored so far is correct, but some of its continuations are not correct; indeed, at each time
an event corresponding to dequeueing an incorrect value could occur.

## Parametric specifications

A specification is called **parametric** if it is able to express properties depending on the data values carried by the monitored events;
this is an important feature to ensure effectiveness of **RV**.

A very simple form of parametricty can be obtained in **RML** by allowing data value variables in specifications; let us consider again
the problem of verifying that files are properly opened and closed;
[property 1](#more-on-intersection-and-shuffle) defined above is oversimplified because it does not take into account an important detail:
an event matching `close` is correctly coupled with a previous event matching `open` only if the two events refer to the same file descriptor `fd`.
Typically, `fd` is returned by a call to function `open`, and it must be passed as an argument when calling `close`, but the value `fd` will
be known only at runtime.

A naive solution to the problem could be as follows:
```js
open(val) matches {event:'func_post',name:'open',res:val};
close(val) matches {event:'func_pre',name:'close',args:[val]};
oc not matches open(_) | close(_);

Main = oc >> (open(fd) close(fd))*;  // limited parametricity 
```
Variable `fd` is bound to the actual value when `open(fd)` is matched by an event for the first time; anyway, such a solution
works only partially: we do not have to know in advance the value of the file descriptor to write the specification,
but, anyway, we can check the correct use of `open` and `close` only for a unique file descriptor!

This is due to the fact that in the specification `fd` is used globally, whereas we would need to declare it as a local variable
to correctly delimit its scope:
```js
Main = oc >> {let fd; open(fd) close(fd)}*;  // true parametricity 
```
In **RML** the `let` keyword is used for declaring data value variables, as `fd` in the example, and the curly braces delimit their scope; now
it is possible to check the correct use of `open` and `close` for different file descriptors (although only one file at time can be opened; see [Examples](examples.md) for a complete specification).

As a final remark, the solution with limited parametricity provided above corresponds to the following specification, where the scope of `fd` goes beyond the Kleene operator `*`:
```js
Main = oc >> {let fd; (open(fd) close(fd))*}; 
```

## Generic specifications
Declaring data value variables in specifications allows us to make them parametric in the data values carried by the monitored events,
but still the **RML** features presented so far do not allow monitors to perform simple computations and to assign their values to variables:
initialization of `let` variables is fully driven by event matching.

Generic specifications allow users to declare parameters and to initialize them with values computed from expressions.
Let us consider for instance the problem of extending the specification of **FIFO** queues above to track call to the
`size` function returning the number of elements in the queue; this is not possible if we are not able to increment and decrement
integer values. 
```js
// FIFO queues with size
enq(val) matches {event:'func_pre',name:'enqueue',args:[val]};
deq(val) matches {event:'func_post',name:'dequeue',res:val};
size(val) matches {event:'func_post',name:'size',res:val};
enq matches enq (_);
deq matches deq (_);
enqDeq matches enq | deq ;


Queue = {let val; enq(val) ((deq | Queue) /\ (deq >> (deq(val) all)))}; // checks enqueue and dequeue
Size<s> = (size(s) Size<s>) \/ (enq Size<s+1>) \/ (deq Size<s-1>); // checks size

Main = ((enqDeq>>Queue) /\ Size<0>)!;
```
The main specification is the conjunction of two properties: `Queue` checking that `enqueue` and `dequeue` behave correctly, and
`Size<s>` checking calls to `size`. The filter operator with the event type `enqDeq` is needed, because the `Queue` specification
is restricted to event of types `enq` or `deq`; although the `Size<s>` specification checks only the `size` function,
calls to `enqueue` and `dequeue` have to be monitored as well to keep track of the correct size of the queue.

`Size<s>` is a generic specification declaring the single parameter `s` corresponding to the size of the queue; in `Main` the generic is applied with `0` because the queue is assumed to be initially empty;  the generic is defined recursively: either `size` is called and returns `s` (that is, the same value
held by the parameter `s`) and then the trace has to continue according to `Size<s>` (that is, the generic is applied with the same size),
or `enqueue` is called and then the trace has to continue according to `Size<s+1>` (that is, the generic is applied with the size increased by `1`),
or `dequeue` is called and then the trace has to continue according to `Size<s-1>` (that is, the generic is applied with the size decreased by `1`).

### Conditional expression
Let us consider the problem of checking that a trace contains exactly *n* events matching `eventType`, where *n* is a parameter; in this
case the conditional expression is required to correctly define the corresponding specification:
```js
Repeat<n> = if (n>0) eventType Repeat<n-1> else empty;
```
If `Repeat<n>` is applied to a non positive integer, then the only accepted trace is the empty one.
