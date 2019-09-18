---
layout: default
title: Verification of interaction protocols
show_downloads: false
---
# Verification of interaction protocols


## Alternating bit protocol

The specifications are based on the following event types:
* `msg(ty)`: message of type `ty` has been received
* `ack(ty)`: acknowledge of type `ty` has been received

### Specification for two message types 

```js
// alt-bit: alternating bit protocol with two message types

msg matches msg(_);
ack matches ack(_);
type(ty) matches msg(ty)|ack(ty);

Main = (msg>>MM) /\ MA;
MM = msg(1)msg(2)MM;
MA = {let ty; msg(ty) ((type(ty) >> ack(ty) all) /\ (ack(ty) | MA))};
```

### Specification for arbitrary message types 

```js
// alt-bit-gen: alternating bit protocol with arbitrary message types

msg matches msg(_);
ack matches ack(_);
type(ty) matches msg(ty)|ack(ty);

Main = (msg>>msg(1) BS<1>) /\ MA;
BS<ty> = {let ty2; msg(ty2) if(ty2==ty+1) BS<ty2> else MM<2,ty>};
MM<ty,max> = msg(ty) if(ty>=max) MM<1,max> else MM<ty+1,max>; 
MA = {let ty; msg(ty) ((type(ty) >> ack(ty) all) /\ (ack(ty) | MA))};
```
