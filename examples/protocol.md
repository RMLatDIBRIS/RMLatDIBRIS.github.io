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

### Specification working with just two message types 

```js
// alt-bit: alternating bit protocols with two message types

msg matches msg(_);
ack matches ack(_);
type(ty) matches msg(ty)|ack(ty);

Main = (msg>>MM) /\ (ack>>AA) /\ (type(1)>>MA<1>) /\ (type(2)>>MA<2>);
MM = msg(1) msg(2) MM;
AA = (ack(1)\/ack(2)) AA;
MA<ty> = msg(ty) ack(ty) MA<ty>;
```
