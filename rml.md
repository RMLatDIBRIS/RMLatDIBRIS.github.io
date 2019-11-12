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
monitoring of [**FIFO** properties](examples/fifo):

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

## Definition of event types with numerical constraints
Sometimes pattern matching is not expressive enough to define event types; the specification of [priority queues](examples/priority), for instance,
requires an event type `deq` to verify that a dequeued integer verifies an inequality constraint.

```js
deq_geq(val) matches deq(val2) with val2 >= val;
```

Event type `deq_geq(val)` matches all events corresponding to dequeuing an integer greater than or equal to `val`.  
When defining an event type, **RML** allows the possibility of adding a set of inequality constraints over real numbers that have to be satisfied
in case one of the specified patterns matches the event; the possibility of satisfying numerical constraints is quite useful in domains
such as [IoT](examples/iot) or [robotic systems](examples/ros). 

```js
check_der(val1,time1,val2,time2) matches sensor(val2,time2)  // checks derivative anomaly of a sensor
			       with delta==(val2-val1)/(time2-time1) && abs(delta) <= 1; 
```

## Basic and derived operators
An **RML** specification denotes a set of event traces, obtained by combining simpler sets with the following basic binary operators (in
decreasing order of precedence):
- *concatenation* (juxtaposition) enforces sequentiality; 
- *intersection* (`/\`) requires simultaneous verification of multiple properties;  
- *union* (`\/`) expresses alternatives;
- *shuffle* (`|`)  allows interleaving of events in traces. 

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
Another useful derived operator is the constant `all` which denotes the universe of all traces, and is an abbreviation for
`any*`.

### More on intersection and shuffle
While concatenation and union are familiar operators used in many contexts, including regular expressions and context-free grammars,
shuffle and intersection are not so widespread, therefore some starting example is needed
to explain why they are so useful in **RML** and, more in general, in runtime verification and monitoring,
to favor conciseness and readability of specifications.

Let us consider the specification of the alternating bit protocol [DeniélouYoshida2012](https://link.springer.com/chapter/10.1007%2F978-3-642-28869-2_10) which has to verify the following
three properties, where event types `msg(1)`, `msg(2)`, `ack(1)`, and `ack(2)` are assumed to be mutually disjoint:
1. for any event `e_1` matching `msg(1)` there must be a subsequent event `e_2` matching `ack(1)` and no other
event matching `msg(1)` is allowed to occur between `e_1` and `e_2`;
2. for any event `e_1` matching `msg(2)` there must be a subsequent event `e_2` matching `ack(2)` and no other
event matching `msg(2)` is allowed to occur between `e_1` and `e_2`;
3. for any event `e_1` matching `msg(1)` there must be a subsequent event `e_2` matching `msg(2)` and no other
event matching `msg(1)` is allowed to occur between `e_1` and `e_2`.

Because of the independence of their event types, the first two properties can be conveniently combined with the shuffle operator: 

```js
Prop1_2 = (msg(1)ack(1))* | (msg(2)ack(2))*;
```

However, the same consideration does not apply to property 3; in this case, intersection is needed to allow a compositional
specification:

```js
no_msg not matches msg(_);
  
Prop1_2 = (msg(1)ack(1))* | (msg(2)ack(2))*;
Prop3 = (msg(1) no_msg* msg(2) no_msg*)*;
Main = Prop1_2/\Prop3;
```

Event type `no_msg` matches all events that do not match `msg(_)`, and is needed
because `Prop3` involves only events of type `msg(1)` or `msg(2)`, while
`Prop1_2` checks also events of type `ack(1)` or `ack(2)`; indeed,
the combination `Prop1_2/\(msg(1)msg(2))*` specifies only the empty trace.

The specification of the alternating bit protocol could be equivalently obtained in a non compositional way by defining a state machine
that is able to monitor the three properties above and that can
be directly translated into an **RML** specification which, however, would be much less readable.

Property 3 has been specified with a standard regular expression, but the obtained specification
is not so simple to parse and understand. Thanks again to the independence of the involved event types,
shuffle allows a more concise and readable specification:

```js
no_msg not matches msg(_);
  
Prop1_2 = (msg(1)ack(1))* | (msg(2)ack(2))*;
Prop3 = (msg(1) msg(2))* | no_msg*;
Main = Prop1_2/\Prop3;
```

This pattern obtained through the combination of the shuffle and
intersection operators occurs so often when composing specifications of different properties that it justifies the
introduction of the derived *filter* operator.

### Filter operators
The specification of property 3 of the [alternating bit protocol](#more-on-intersection-and-shuffle) can be
further simplified by using a filter operator:

```js
msg matches msg(_);

Prop1_2 = (msg(1)ack(1))* | (msg(2)ack(2))*;
Prop3 = msg >> (msg(1) msg(2))*;
Main = Prop1_2/\Prop3;
```

The specification `msg >> (msg(1) msg(2))*` denotes the set of all traces verifying
`(msg(1) msg(2))*` when only all events matching event type
`msg` are kept; event type `msg` matches all events matching `msg(_)`.

The `>>` operator can be generalized into a conditional filter which takes an
additional operand: `eventType >> Spec1 : Spec2` denotes the set of all traces verifying
`Spec1` when only all events matching `eventType` are kept, and
`Spec2` when only all events not matching `eventType` are kept.
The less general version `eventType >> Spec` presented above is equivalent to
`eventType >> Spec : all`.

Conditional filter can be derived from
the shuffle, intersection and star operators, with the assumption, as it is the case of **RML**, that event types
are closed w.r.t. negation.

### Derived operators and monitor verdicts 

Any time it receives a new event, the monitor generated from an **RML** specification emits a verdict in a 4-valued logic:
- **True**: the event trace monitored so far is correct, and any continuation will be correct as well;  
- **Maybe True**: the event trace monitored so far is correct, but some of its continuations are not correct; 
- **Maybe False**: the event trace monitored so far is not correct, but some of its continuations are correct;  
- **False**: the event trace monitored so far is not correct, and any continuation will be incorrect as well.

The first and last values are called *conclusive* since in those cases no other monitoring is required
to decide whether the **SUS** is correct or not; the others are *inconclusive* and
require the monitor to keep running.

The derived constant `all` is useful to allow monitors to emit **True** as verdict.

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

`Size<s>` is a generic specification declaring the single parameter `s` corresponding to the size of the queue; such parameters are called *state variables* because they define attributes of the state of the monitor generated from the specification. In `Main` the generic is applied to `0` because the queue is assumed to be initially empty;  the generic is defined recursively: either `size` is called and returns `s` (that is, the same value
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
