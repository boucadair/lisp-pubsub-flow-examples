---
title: "LISP PubSub Flow Examples"
abbrev: "LISP PubSub Examples"
category: info

docname: draft-boucadair-lisp-pubsub-flow-examples-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: "Routing"
workgroup: "Locator/ID Separation Protocol"
keyword:
 - LISP

author:
 -
    fullname: Mohamed Boucadair
    organization: Orange
    city: Rennes
    code: 35000
    country: France
    email: mohamed.boucadair@orange.com


normative:

informative:

--- abstract

This document provides a set of flow examples to illustrate the use of LISP PubSub specification.

--- middle

# Introduction

This document provides a set of flow examples as a companion to the LISP PubSub specification {{!I-D.ietf-lisp-pubsub}}. The document is meant to illustrate and assess the behavior of LISP control nodes under specific conditions.

The examples use a simplified/simple setup for the sake of illustration.

# Terminology

This document uses the terms defined in {{!I-D.ietf-lisp-pubsub}}.

The following terms and notations are used in this document:

init_nonce:
: the nonce that is initially included in a Map-Request to create a subscription.

initial subscription request:
: the Map-Request that was used to create the initial subscription. This request has the nonce value set to init_nonce.

nonce++:
: incremented nonce by 1.

init_key_id:
: the key identifier that was used in the Map-Request with init_nonce.

trans_count:
: retransmission counter as per Section 5.7 of {{!RFC9301}}.

trans_timer:
: retransmission timer as per Section 5.7 of {{!RFC9301}}.

AT:
: Attacker


# Initial Successful Subscription {#sec-iss}

The following example assumes that a security association is in place between xTR/Map-Server (Section 7.1 of {{!I-D.ietf-lisp-pubsub}}) and that all subsequent integrity-protection checks were successfully passed.

~~~~ aasvg
                     +---+                          +----+
                     |xTR|                          | MS |
                     +-+-+                          +--+-+
                       |                               |
.--------------------. |                               |
| Generate a new key | | Map-Request(init_nonce,       | .--------------------.
| and an initial     | |            init_key_id,..)    | | Security/integrity |
| nonce. Store them  +-+==============================>+-+ protection check.  |
| locally for this   | |                               | | No State for this  |
| subscription       | |                               | | XTR-ID/EID is found|
'--------------------' |                               | | Create the sub and |
                       |                               | | store init_nonce,  |
.--------------------. | Map-Notify(init_nonce,...)    | | init_key_id, ...   |
| Security/integrity +-+<==============================+-+                    |
| protection check.  | |                               | '--------------------'
| Check that rcv     | |                               |
| nonce == init_nonce| | Map-Notify-Ack(init_nonce,...)| .--------------------.
| Confirm the sub and+-+==============================>+-+ Security/integrity |
| wait for notifs    | |                               | | protection checks. |
'--------------------' |                               | | This subscription  |
                       |                               | | is now ACKed       |
                       |                               | '--------------------'
~~~~
{: #iss title="An Example of Successful Initial Subscription" artwork-align="center"}

# Successful Notification

The following example assumes that a security association is in place between xTR/Map-Server (Section 7.1 of {{!I-D.ietf-lisp-pubsub}}) and that all subsequent integrity-protection checks were successfully passed.

~~~~ aasvg
                     +---+                          +----+
                     |xTR|                          | MS |
                     +-+-+                          +--+-+
                       |                               |
.--------------------. |                               | .--------------------.
| Security/integrity | | Map-Notify(nonce++, ...)      | | Update is triggered|
| protection check.  +-+<==============================+-+ Increment the nonce|
| Check that rcv     | |                               | | Set trans_count and|
| nonce == local     | |                               | | trans_timer        |
| nonce + 1          | |                               | '--------------------'
|                    | |                               |
| Confirms the notif | |                               | .--------------------.
| and update the     | | Map-Notify-Ack(nonce++,...)   | | Security/integrity |
| entry              +-+==============================>+-+ protection checks. |
|                    | |                               | | This notification  |
'--------------------' |                               | | is now ACKed       |
                       |                               | '--------------------'
~~~~
{: #sn title="An Example of Successful Notification" artwork-align="center"}

# Successful Notification with Retransmission

The following example assumes that a security association is in place between xTR/Map-Server (Section 7.1 of {{!I-D.ietf-lisp-pubsub}}) and that all subsequent integrity-protection checks were successfully passed. Due to network conditions, some Map-Notifies are lost.

~~~~ aasvg
                     +---+                          +----+
                     |xTR|                          | MS |
                     +-+-+                          +--+-+
                       |                               |
                       |                               | .--------------------.
                       |        Map-Notify(nonce, ...) | | Update is triggered|
                       |            <==================+-+ Increment the nonce|
                       |                               | | Set trans_count and|
                       |                               | | trans_timer        |
                       |                               | '--------------------'
                       |                               |
                       |                               | .--------------------.
                       |        Map-Notify(nonce, ...) | | Increment          |
                       |            <==================+-+ trans_count and    |
                       |                               | | reset trans_timer  |
                       |                               | '--------------------'
                       |                               |
.--------------------. |                               | .--------------------.
| Security/integrity | |   Map-Notify(nonce, ...)      | | Increment          |
| protection check.  +-+<==============================+-+ trans_count and    |
| Check that rcv     | |                               | | reset trans_timer  |
| nonce == local     | |                               | '--------------------'
| nonce + 1          | |                               |
|                    | |                               | .--------------------.
| Confirms the notif | | Map-Notify-Ack(nonce,...)     | | Security/integrity |
| and update the     +-+==============================>+-+ protection checks. |
| entry              | |                               | | This notification  |
'--------------------' |                               | | is now ACKed       |
                       |                               | '--------------------'
~~~~
{: #sretrans title="An Example of Successful Notification with Retransmission" artwork-align="center"}

# Failed Notification with Retransmission

The following example assumes that a security association is in place between xTR/Map-Server (Section 7.1 of {{!I-D.ietf-lisp-pubsub}}) and that all subsequent integrity-protection checks were successfully passed. Due to network conditions, All Map-Notifies are lost.

~~~~ aasvg
                     +---+                          +----+
                     |xTR|                          | MS |
                     +-+-+                          +--+-+
                       |                               |
                       |                               | .--------------------.
                       |        Map-Notify(nonce, ...) | | Update is triggered|
                       |            <==================+-+ Increment the nonce|
                       |                               | | Set trans_count and|
                       |                               | | trans_timer        |
                       |                               | '--------------------'
                       |                               |
                       |                               | .--------------------.
                       |        Map-Notify(nonce, ...) | | Increment          |
                       |            <==================+-+ trans_count and    |
                       |                               | | reset trans_timer  |
                       |                               | '--------------------'
                       |                               |
                       |                               | .--------------------.
                       |        Map-Notify(nonce, ...) | | Increment          |
                       |            <==================+-+ trans_count and    |
                       |                               | | reset trans_timer  |
                       |                               | '--------------------'
~~~~
{: #fretrans title="An Example of Failed Notification Delivery" artwork-align="center"}

Note that no specific action is currently specified in {{!I-D.ietf-lisp-pubsub}} when such a failure occurs. That is, the entry is kept active and future updates will trigger new Map-Notify cycles.


# Successful Subscription Update

The following example illustrates the case of a successful update of a subscription.

~~~~ aasvg
                     +---+                          +----+
                     |xTR|                          | MS |
                     +-+-+                          +--+-+
                       |                               |
.--------------------. |                               | .--------------------.
| Increment the last | | Map-Request(nonce, ...)       | | Security/integrity |
| seen nonce         +-+==============================>+-+ protection check.  |
'--------------------' |                               | | Found an entry for |
                       |                               | | this xTR-ID        |
.--------------------. | Map-Notify(nonce,...)         | | Check that rcv     |
| Security/integrity +-+<==============================+-+ nonce == local     |
| protection check.  | |                               | | nonce + 1          |
| Check that rcv     | |                               | '--------------------'
| nonce == snd nonce | |                               |
| Confirm the sub and| | Map-Notify-Ack(nonce,...)     | .--------------------.
| wait for notifs    +-+==============================>+-+ Security/integrity |
'--------------------' |                               | | protection check.  |
                       |                               | | This subscription  |
                       |                               | | updated is ACKed   |
                       |                               | '--------------------'
~~~~
{: #ssu title="An Example of Successful Subscription Update" artwork-align="center"}


# Failed Subscription with Lost Map-Notify-Ack

This example is similar to {{sec-iss}}, except the Map-Notify-Ack is not delivered to the Map-Server. The Map-Server retransmits the Map-Notify 3 times, and then removes the subscription. A Map-Notify to explicitly indicate the reason for such a removal is also generated by the Map-Server. If the xTR received this Map-Notify, the xTR has to send the Map-Request to reinstall back the removed state.

~~~~ aasvg
                     +---+                          +----+
                     |xTR|                          | MS |
                     +-+-+                          +--+-+
                       |                               |
.--------------------. |                               |
| Generate a new key | | Map-Request(init_nonce,       | .--------------------.
| and an initial     | |            init_key_id,..)    | | Security/integrity |
| nonce. Store them  +-+==============================>+-+ protection check.  |
| locally for this   | |                               | | No State for this  |
| subscription       | |                               | | xTR-ID/EID is found|
'--------------------' |                               | | Create the sub and |
                       |                               | | store init_nonce,  |
.--------------------. | Map-Notify(init_nonce,...)    | | init_key_id, ...   |
| Security/integrity +-+<==============================+-+ Set trans_count and|
| protection check.  | |                               | | trans_timer        |
| Check that rcv     | |                               | '--------------------'
| nonce == init_nonce| | Map-Notify-Ack(init_nonce,...)|
| Confirm the sub and+-+===========>                   |
| wait for notifs    | |                               |
'--------------------' |                               |
                       |                               | .--------------------.
                       |        Map-Notify(nonce, ...) | | Increment          |
                       |            <==================+-+ trans_count and    |
                       |                               | | reset trans_timer  |
                       |                               | '--------------------'
                       |                               |
                       |                               | .--------------------.
                       |        Map-Notify(nonce, ...) | | Increment          |
                       |            <==================+-+ trans_count and    |
                       |                               | | reset trans_timer  |
                       |                               | '--------------------'
                       |                               |
                       |                               | .--------------------.
                       |        Map-Notify(nonce, ...) | | Remove the subscri-|
                       |     <=========================+-+ ption              |
                       |                               | '--------------------'
                      ...                              |

~~~~
{: #fiss title="An Example of Failed Initial Subscription" artwork-align="center"}

# Bootstrapping an xTR

When first bootrsapped, an xTR may delete any (stale) state that might be associated with its provisioned xTR-ID and security association. To that aim, the xTR sends a Map-Request that has only one ITR-RLOC with AFI = 0.

# Stale Subscriptions

For various reasons, an xTR may lose its subscription (or at leastt the nonce of a subscription). Under such conditions, the xTR must use a new authentication key and a new nonce for the Map-Request.

If the same key is used, the Map-Request is likely to be rejected by the Map-Server and, thus, stale subscriptions will be maintained by the Map-Server.

~~~~ aasvg
                     +---+                          +----+
                     |xTR|                          | MS |
                     +-+-+                          +--+-+
                       |                               |
                       | Map-Request(nonce,            | .--------------------.
                       |            init_key_id,..)    | | Security/integrity |
                       +==============================>+-+ protection check.  |
                       |                               | | A state for        |
                       | Map-Reply(ACT=5)              | | xTR-ID/EID is found|
                       |<==============================+-+ but the nonce check|
                       |                               | | fails              |
                       |                               | '--------------------'
~~~~
{: #stale title="An Example of Stale Subscription" artwork-align="center"}

# xTR-triggered Subscription Withdrawal

{{xmd}} illustrates the observed exchange to delete a subscription.

~~~~ aasvg
                     +---+                          +----+
                     |xTR|                          | MS |
                     +-+-+                          +--+-+
                       |                               |
.--------------------. |                               | .--------------------.
| Increment the last | | Map-Request(nonce, AF=0...)   | | Security/integrity |
| seen nonce         +-+==============================>+-+ protection check.  |
'--------------------' |                               | | Found an entry for |
                       |                               | | this xTR-ID        |
.--------------------. | Map-Notify(nonce,...)         | | Check that rcv     |
| Security/integrity +-+<==============================+-+ nonce == local     |
| protection check.  | |                               | | nonce + 1          |
| Check that rcv     | |                               | '--------------------'
| nonce == snd nonce | |                               |
| Send Map-Notfiy-ACK| | Map-Notify-Ack(nonce,...)     | .--------------------.
|                    +-+==============================>+-+ Security/integrity |
'--------------------' |                               | | protection check.  |
                       |                               | | This withdrawal is |
                       |                               | | confirmed          |
                       |                               | '--------------------'
~~~~
{: #xmd title="An Example of Successful Subscription Withdrawal" artwork-align="center"}

# 'Map-Server'-triggered Subscription Withdrawal

{{msw}} illustrates the observed exchange to notify the withdrawal of a subscription at the initiative of the Map-Server.

~~~~ aasvg
                     +---+                          +----+
                     |xTR|                          | MS |
                     +-+-+                          +--+-+
                       |                               |
.--------------------. |                               | .--------------------.
| Security/integrity | | Map-Notify(nonce, TTL=0, ...) | | Update is triggered|
| protection check.  +-+<==============================+-+ Increment the nonce|
| Check that rcv     | |                               | | Set trans_count and|
| nonce == local     | |                               | | trans_timer        |
| nonce + 1          | |                               | '--------------------'
|                    | |                               |
| Confirms the notif | |                               | .--------------------.
| and remove the     | | Map-Notify-Ack(nonce, ...)    | | Security/integrity |
| entry              +-+==============================>+-+ protection checks. |
|                    | |                               | | This notification  |
'--------------------' |                               | | is now ACKed       |
                       |                               | '--------------------'
~~~~
{: #msw title="An Example of Successful Notification of Subscription withdrawal" artwork-align="center"}

## Replay Attacks

### Replayed Subscription (Update)

{{riss}} shows the example of a replayed subscription request. This example assumes that the attacker does not change the source IP address that was initially captured in the packet to be replayed. The received Map-Reply is silently ignored by the xTR.

~~~~ aasvg
                     +---+                          +----+
                     | AT|                          | MS |
                     +-+-+                          +--+-+
                       |                               |
                       | Map-Request(init_nonce,       | .--------------------.
                       |            init_key_id,..)    | | Security/integrity |
                       +==============================>+-+ protection check.  |
                       |                               | | A state is found   |
  +---+                                                | | xTR-ID/EID is found|
  |xTR|                                                | | but the nonce check|
  +-+-+                Map-Reply(init_nonce,...)       | | fails              |
    |<=================================================+-+                    |
    |                                                  | '--------------------'
    |                                                  |
~~~~
{: #riss title="An Example of Handling of Replayed Initial Subscription" artwork-align="center"}

The attacker may vary the source IP address of the Map-Request to trigger as many Map-Replies sent to other xTRs (see {{riss-m}}.

~~~~ aasvg
                     +---+                          +----+
                     | AT|                          | MS |
                     +-+-+                          +--+-+
                       |                               |
                       | Map-Request(init_nonce,       | .--------------------.
                       |            init_key_id,..)    | | Security/integrity |
                       +==============================>+-+ protection check.  |
                       |              ...              | | A state is found   |
  +-----+              +==============================>+-+ xTR-ID/EID is found|
  |xTR1 |                                              | | but the nonce check|
  +-+--+                Map-Reply(init_nonce,...)      | | fails              |
    |<=================================================+-+                    |
                                                       | '--------------------'
                 +-----+                               |
                 | xTRn|                               |
                 +--+--+    Map-Reply(init_nonce,...)  |
                    |<=================================+
                    |                                  |
               ...
~~~~
{: #riss-m title="An Example of Handling of Replayed Initial Subscription" artwork-align="center"}

Note that if LISP messages supported timestamping, the replayed packet would be detected and thus be silently ignored by the Map-Server.

~~~~ aasvg
                     +---+                          +----+
                     | AT|                          | MS |
                     +-+-+                          +--+-+
                       |                               |
                       | Map-Request(init_nonce,       | .--------------------.
                       |            init_key_id,..)    | | Security/integrity |
                       +==============================>+-+ protection check.  |
                       |                               | | A state is found   |
                                                       | | xTR-ID/EID is found|
  +---+                                                | | but the nonce check|
  |xTR|                                                | | fails              |
  +-+-+                                                | | init_key_id, ...   |
    |                                                  | |                    |
    |                                                  | '--------------------'
~~~~
{: #riss-ts title="An Example of Handling of Replayed Initial Subscription with Timestamp" artwork-align="center"}

### Replayed Withdrawal



### Replayed Notification Updates

{{rmsw}} illustrates the observed exchange when a replayed notification update is sent by a misbehaving node (AT) to an xTR.

~~~~ aasvg
                     +---+                          +----+
                     |xTR|                          | AT |
                     +-+-+                          +--+-+
                       |                               |
.--------------------. |                               |
| Security/integrity | | Map-Notify(nonce, TTL=0, ...) |
| protection check.  +-+<==============================+
| Check that rcv     | |                               |
| nonce == local     | |                               |
| nonce + 1          | |                               |
|                    | |                               |
| Discard the message| |                               |
| because the nonce  | |                               |
| cheks failed       | |                               |
'--------------------' |                               |
                       |                               |
~~~~
{: #rmsw title="An Example of Replayed Notification of Subscription Withdrawal" artwork-align="center"}


# Security Considerations

This document does not introduce any security considerations beyond those already discussed in {{!I-D.ietf-lisp-pubsub}}.

# IANA Considerations {#IANA}

This document does not make any request to IANA.

--- back

# Acknowledgments
{:numbered="false"}

Thanks to TBC.
