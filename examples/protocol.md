---
layout: default
title: Verification of interaction protocols
show_downloads: false
---
# Verification of interaction protocols

## FIPA Request Protocol

Simple version, no uniqueness of message id is verified

```rml
send(sender,receiver,performative,content,msgId) matches {sender:{name:sender}, unicastReceiver:{name:receiver}, performative:performative, messageNumber:msgId,content:content};
send(sender,receiver,performative,msgId) matches {sender:{name:sender}, unicastReceiver:{name:receiver}, performative:performative, messageNumber:msgId};
request(sender,receiver,msgId) matches send(sender,receiver,'REQUEST',msgId);
agree(sender,receiver,msgId) matches send(sender,receiver,'AGREE',msgId);
refuse(sender,receiver,msgId) matches send(sender,receiver,'REFUSE',msgId);
failure(sender,receiver,msgId) matches send(sender,receiver,'FAILURE',msgId);
informDone(sender,receiver,msgId) matches send(sender,receiver,'INFORM',msgId);
informResult(sender,receiver,msgId) matches send(sender,receiver,'INFORM',_,msgId);

relevant matches send(_,_,_,_) | send(_,_,_,_,_);

Main = relevant >> Request;

Request = {let initiator, participant, msgId; request(initiator,participant,msgId) (ManageRequest<initiator,participant,msgId> | Request)}?;

ManageRequest<initiator,participant,msgId> = Agree<participant,initiator,msgId>\/refuse(participant,initiator,msgId);

Agree<participant,initiator,msgId> = agree(participant,initiator,msgId)?(failure(participant,initiator,msgId)\/informDone(participant,initiator,msgId)\/informResult(participant,initiator,result,msgId));
```

Specification requiring also the uniqueness of message id

```rml
send(sender,receiver,performative,content,msgId) matches {sender:{name:sender}, unicastReceiver:{name:receiver}, performative:performative, messageNumber:msgId,content:content};
send(sender,receiver,performative,msgId) matches {sender:{name:sender}, unicastReceiver:{name:receiver}, performative:performative, messageNumber:msgId};
request(sender,receiver,msgId) matches send(sender,receiver,'REQUEST',msgId);
agree(sender,receiver,msgId) matches send(sender,receiver,'AGREE',msgId);
refuse(sender,receiver,msgId) matches send(sender,receiver,'REFUSE',msgId);
failure(sender,receiver,msgId) matches send(sender,receiver,'FAILURE',msgId);
informDone(sender,receiver,msgId) matches send(sender,receiver,'INFORM',msgId);
informResult(sender,receiver,msgId) matches send(sender,receiver,'INFORM',_,msgId);

request(msgId) matches request(_,_,msgId);
notRequest not matches request(_,_,_);
closedRequest(msgId) matches refuse(_,_,msgId) | failure(_,_,msgId) | informDone(_,_,msgId) | informResult(_,_,msgId);
notRequestOrClosedRequest(msgId) not matches request(msgId) | closedRequest(msgId);

relevant matches send(_,_,_,_) | send(_,_,_,_,_);

Main = relevant >> (Request /\ UniqueId);

Request = {let initiator, participant, msgId; request(initiator,participant,msgId) (ManageRequest<initiator,participant,msgId> | Request)}?;

ManageRequest<initiator,participant,msgId> = Agree<participant,initiator,msgId>\/refuse(participant,initiator,msgId);

Agree<participant,initiator,msgId> = agree(participant,initiator,msgId)?(failure(participant,initiator,msgId)\/informDone(participant,initiator,msgId)\/informResult(participant,initiator,result,msgId));

UniqueId = notRequest* {let msgId; request(msgId) (notRequestOrClosedRequest(msgId)* closedRequest(msgId) all /\ UniqueId) }?;
```

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
