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

{{iss}} shows the example of a successful subscription. The example assumes that a security association is in place between the xTR ad the Map-Server (Section 7.1 of {{!I-D.ietf-lisp-pubsub}}) and that all integrity-protection checks are successfully passed.

~~~~ aasvg
                     +---+                    +----+
                     |xTR|                    | MS |
                     +-+-+                    +--+-+
                       |                         |
.--------------------. |                         |
| Generate a new key | | Map-Request(init_nonce, | .--------------------.
| and an initial     | |        init_key_id,..)  | | Security/integrity |
| nonce. Store them  +-+=========================+-+ protection check.  |
| locally for this   | |                         | | No State for this  |
| subscription       | |                         | | xTR-ID/EID is found|
'--------------------' |                         | | Create the sub and |
                       | Map-Notify(init_nonce,  | | store init_nonce,  |
.--------------------. |                   ,...) | | init_key_id, ...   |
| Security/integrity +-+<========================+-+                    |
| protection check.  | |                         | '--------------------'
| Check that rcv     | |Map-Notify-Ack(init_nonce|
| nonce == init_nonce| |                    ,...)| .--------------------.
| Confirm the sub and+-+========================>+-+ Security/integrity |
| wait for notifs    | |                         | | protection checks. |
'--------------------' |                         | | This subscription  |
                       |                         | | is now ACKed       |
                       |                         | '--------------------'
~~~~
{: #iss title="An Example of Successful Initial Subscription" artwork-align="center"}

# Successful Notification

{{sn}} illustrates the example of a successful delivery of notification updates that match an existing subscription state. This example assumes that a security association is in place between the xTR and the Map-Server (Section 7.1 of {{!I-D.ietf-lisp-pubsub}}) and that all subsequent integrity-protection checks are successfully passed.

~~~~ aasvg
                     +---+                     +----+
                     |xTR|                     | MS |
                     +-+-+                     +--+-+
                       |                          |
.--------------------. |                          | .--------------------.
| Security/integrity | | Map-Notify(nonce++, ...) | | Update is triggered|
| protection check.  +-+<=========================+-+ Increment the nonce|
| Check that rcv     | |                          | | Set trans_count and|
| nonce >= local     | |                          | | trans_timer        |
| nonce + 1          | |                          | '--------------------'
|                    | |                          |
| Confirms the notif | |                          | .--------------------.
| and update the     | |Map-Notify-Ack(nonce++,..)| | Security/integrity |
| entry              +-+=========================>+-+ protection checks. |
|                    | |                          | | This notification  |
'--------------------' |                          | | is now ACKed       |
                       |                          | '--------------------'
~~~~
{: #sn title="An Example of Successful Notification" artwork-align="center"}

# Successful Notification with Retransmission

Unlike the example depicted in {{sn}}, {{sretrans}} illustrates the behavior that is experienced  when a subset of Map-Notify messages are lost during their transfer. This example assumes that at least one of these Map-Notify messages is received by the target xTR.

~~~~ aasvg
                     +---+                   +----+
                     |xTR|                   | MS |
                     +-+-+                   +--+-+
                       |                        |
                       |                        | .--------------------.
                       | Map-Notify(nonce, ...) | | Update is triggered|
                       |     <==================+-+ Increment the nonce|
                       |                        | | Set trans_count and|
                       |                        | | trans_timer        |
                       |                        | '--------------------'
                       |                        |
                       |                        | .--------------------.
                       | Map-Notify(nonce, ...) | | Increment          |
                       |     <==================+-+ trans_count and    |
                       |                        | | reset trans_timer  |
                       |                        | '--------------------'
                       |                        |
.--------------------. |                        | .--------------------.
| Security/integrity | |Map-Notify(nonce, ...)  | | Increment          |
| protection check.  +-+<=======================+-+ trans_count and    |
| Check that rcv     | |                        | | reset trans_timer  |
| nonce >= local     | |                        | '--------------------'
| nonce + 1          | |                        |
|                    | |                        | .--------------------.
| Confirms the notif | |Map-Notify-Ack(nonce,...) | Security/integrity |
| and update the     +-+=======================>+-+ protection checks. |
| entry              | |                        | | This notification  |
'--------------------' |                        | | is now ACKed       |
                       |                        | '--------------------'
~~~~
{: #sretrans title="An Example of Successful Notification with Retransmission" artwork-align="center"}

# Failed Notification with Retransmission

{{fretrans}} assumes that, due to network conditions, all Map-Notifies are lost.

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

Note that no specific action is currently specified in {{!I-D.ietf-lisp-pubsub}} when such a failure occurs. That is, the entry is kept active and future updates will trigger new Map-Notify cycles. Also, the current specification does not recommend a behavior (e.g., regular refreshes) so that an xTR avoids maintaining stale mappings. Such details are implementation specific (see, for example, {{sec-sub-update}}). In order to accommodate Map-Notify messages lost, the nonce checks on the xTR should not be on the exact match vs "nonce + 1"; messages with "received nonce >= local nonce + 1" should be accepted.

# Successful Subscription Update {#sec-sub-update}

{{ssu}} illustrates the example of uccessful update of an existing subscription. The triggers for such a refresh are implementation specific.

~~~~ aasvg
                     +---+                     +----+
                     |xTR|                     | MS |
                     +-+-+                     +--+-+
                       |                          |
.--------------------. |                          | .--------------------.
| Increment the last | | Map-Request(nonce, ...)  | | Security/integrity |
| seen nonce         +-+=========================>+-+ protection check.  |
'--------------------' |                          | | Found an entry for |
                       |                          | | this xTR-ID        |
.--------------------. | Map-Notify(nonce,...)    | | Check that rcv     |
| Security/integrity +-+<=========================+-+ nonce >= local     |
| protection check.  | |                          | | nonce + 1          |
| Check that rcv     | |                          | '--------------------'
| nonce == snd nonce | |                          |
| Confirm the sub and| | Map-Notify-Ack(nonce,...) .--------------------.
| wait for notifs    +-+=========================>+-+ Security/integrity |
'--------------------' |                          | | protection check.  |
                       |                          | | This subscription  |
                       |                          | | update is ACKed    |
                       |                          | '--------------------'
~~~~
{: #ssu title="An Example of Successful Subscription Update" artwork-align="center"}


# Failed Subscription with Lost Map-Notify-Ack

This example is similar to {{sec-iss}}, except that the Map-Notify-Ack is not delivered to the Map-Server. The Map-Server retransmits the Map-Notify 3 times and then removes the subscription. A Map-Notify to explicitly indicate the reason for such a removal is also generated by the Map-Server. If the xTR receives this Map-Notify, the xTR may decide to send the Map-Request to reinstall back the removed state. The procedure to reinstall the state is similar to {{iss}}.

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
                       |Map-Notify(nonce, AFI, ACT,...)| | Remove the subscri-|
                       |     <=========================+-+ ption              |
                       |                               | '--------------------'
                      ...                              |
~~~~
{: #fiss title="An Example of Failed Initial Subscription" artwork-align="center"}

# Stale Subscriptions

For various reasons, an xTR may lose its subscriptions (or at least the nonce of a subscription). Note that losing the nonce is not compliant with the following from the PubSub specification:

   > The xTR MUST keep track of the last nonce seen in a Map-Notify received as a publication from the Map-Server for the EID-Record.

If the same key is used, the Map-Request is likely to be rejected by the Map-Server and, thus, stale subscriptions will be maintained by the Map-Server. The request is silently discarded by the Map-Server. This behavior is similar to this behavior in {{!RFC9301}}:

> If a Map-Register is received with a nonce value that is not greater than the saved nonce, it MUST drop the Map-Register message and SHOULD log the fact that a replay attack could have occurred.

~~~~ aasvg
                     +---+                          +----+
                     |xTR|                          | MS |
                     +-+-+                          +--+-+
                       |                               |
                       | Map-Request(nonce,            | .--------------------.
                       |            init_key_id,..)    | | Security/integrity |
                       +==============================>+-+ protection check.  |
                       |                               | | A state for        |
                       |                               | | xTR-ID/EID is found|
                       |                               | | but the nonce check|
                       |                               | | fails: rcv nonce < |
                       |                               | | local nonce + 1.   |
                       |                               | | Discard the packet |
                       |                               | '--------------------'
~~~~
{: #stale title="An Example of Stale Subscriptions" artwork-align="center"}

If the Map-Server stores all the key-ids that were used by an xTR for its subscriptions, the Map-Server may accept overriding an existing state without enforcing the nonce check but if and only if a new key is used (see {{stale-new-key}}) and that the new security association succeeds.

~~~~ aasvg
                     +---+                     +----+
                     |xTR|                     | MS |
                     +-+-+                     +--+-+
                       |                          |
                       | Map-Request(nonce,       | .--------------------.
                       |         new key_id, ...) | | Security/integrity |
                       +=========================>+-+ protection check.  |
                       |                          | | A state for        |
.--------------------. | Map-Notify (nonce, ...)  | | xTR-ID/EID is found|
| Security/integrity +-+<=========================+-+ but the new auth   |
| protection check.  | |                          | | key is used, the   |
| Check that rcv     | |                          | | state is updated   |
| nonce == snd nonce | |                          | '--------------------'
| Confirm the sub and| | Map-Notify-Ack(nonce,...) .--------------------.
| wait for notifs    +-+=========================>+-+ Security/integrity |
'--------------------' |                          | | protection check.  |
                       |                          | | This subscription  |
                       |                          | | update is ACKed    |
                       |                          | '--------------------'
~~~~
{: #stale-new-key title="An Example of Stale Subscriptions Avoidance with New KEys" artwork-align="center"}

However, the approach in {{stale-new-key}} may have scalability issues as the Map-Server must store all the key identifiers that were ever used. Otherwise, an attacker can replay a message for which the key-id is not stored anymore by the Map-Server. This issue is not encountered if LISP-SEC messages are timestamped.

> Note that currently none of LISP specifications use timestamps.

# xTR-triggered Subscription Withdrawal

{{xmd}} illustrates the observed exchange to successfully delete a subscription.

~~~~ aasvg
                     +---+                          +----+
                     |xTR|                          | MS |
                     +-+-+                          +--+-+
                       |                               |
.--------------------. |                               | .--------------------.
| Increment the last | | Map-Request(nonce, AFI=0...)  | | Security/integrity |
| seen nonce         +-+==============================>+-+ protection check.  |
'--------------------' |                               | | Found an entry for |
                       |                               | | this xTR-ID        |
.--------------------. | Map-Notify(nonce,...)         | | Check that rcv     |
| Security/integrity +-+<==============================+-+ nonce >= local     |
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
| nonce >= local     | |                               | | trans_timer        |
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

# Bootstrapping an xTR

When first bootrsapped, an xTR may delete any (stale) state that might be associated with its provisioned xTR-ID and security association. To that aim, the xTR sends a Map-Request that has only one ITR-RLOC with AFI = 0.

A Map-Notify will be sent back by the Map-Server even if no subscription is found as illustrated in {{boot}}.

~~~~ aasvg
                     +---+                          +----+
                     |xTR|                          | MS |
                     +-+-+                          +--+-+
                       |                               |
.--------------------. |                               | .--------------------.
| Generate a random  | | Map-Request(nonce, AFI=0...)  | | Security/integrity |
| nonce and new key  +-+==============================>+-+ protection check.  |
'--------------------' |                               | | No entry is found  |
                       |                               | | for this xTR-ID    |
.--------------------. | Map-Notify(nonce, ...)        | |                    |
| Security/integrity +-+<==============================+-+                    |
| protection check.  | |                               | |                    |
| Check that rcv     | |                               | '--------------------'
| nonce == snd nonce | |                               |
| Send Map-Notfiy-ACK| | Map-Notify-Ack(nonce,...)     |
|                    +-+==============================>+
'--------------------' |                               |
                       |                               |
~~~~
{: #boot title="An Example of Clearing State when Bootstrapping" artwork-align="center"}


## Replay Attacks

### Replayed Subscription (Update)

{{riss}} shows the example of a replayed subscription request. The request will be silently dropped the Map-Server because of nonce check failure. This example assumes that a state is maintained by the Map-Server for this xTR.

~~~~ aasvg
                     +---+                          +----+
                     | AT|                          | MS |
                     +-+-+                          +--+-+
                       |                               |
                       | Map-Request(init_nonce,       | .--------------------.
                       |            init_key_id,..)    | | Security/integrity |
                       +==============================>+-+ protection check.  |
                       |                               | | A state is for     |
  +---+                                                | | xTR-ID/EID is found|
  |xTR|                                                | | but the nonce check|
  +-+-+                                                | | fails: rcv nonce < |
    |                                                  | | local nonce + 1.   |
    |                                                  | | Discard the packet |
    |                                                  | '--------------------'
    |                                                  |
~~~~
{: #riss title="An Example of Handling of Replayed Initial Subscription" artwork-align="center"}

Note that legitimate Map-Requests issued from the authentic xTR may be blocked as a side effect of enforcing a rate-lmit of the replayed messages. An example is shown in {{riss-rate}}.

~~~~ aasvg
                  +---+                      +----+
                  | AT|                      | MS |
                  +-+-+                      +--+-+
                    |                           |
                    | Map-Request(init_nonce,   | .--------------------.
                    |          init_key_id,..)  | | Security/integrity |
                    +==========================>+-+ protection check.  |
                    |            ...            | | A state is found   |
                    +==========================>+-+ xTR-ID/EID is found|
                    |                           | | but the nonce check|
                    |                           | | fails: rcv nonce < |
                    |                           | | local nonce + 1    |
                    |                           | '--------------------'
                    |                           |
                    |    (more requests)        | .--------------------.
                    |                           +-+ Rate-limit xTR-ID  |
                    |                           | | requests is reached|
  +----+                                        | '--------------------'
  |xTR |                                        |
  +-+--+         Map-Request(...)               | .--------------------.
    |==========================================>+-+ Discard            |
                                                | '--------------------'
~~~~
{: #riss-rate title="An Example of Handling of Replayed Initial Subscription" artwork-align="center"}

If replayed attacks are not counted as part of the rate-limit policy, legitimate Map-Requests will be procecced as illustrate in {{riss-rate2}}.

~~~~ aasvg
                  +---+                      +----+
                  | AT|                      | MS |
                  +-+-+                      +--+-+
                    |                           |
                    | Map-Request(init_nonce,   | .--------------------.
                    |          init_key_id,..)  | | Security/integrity |
                    +==========================>+-+ protection check.  |
                    |            ...            | | A state is found   |
                    +==========================>+-+ xTR-ID/EID is found|
                    |                           | | but the nonce check|
                    |                           | | fails: rcv nonce < |
                    |                           | | local nonce + 1    |
                    |                           | '--------------------'
  +----+                                        |
  |xTR |                                        |
  +-+--+         Map-Request(...)               | .--------------------.
    |==========================================>+-+ Process            |
                                                | '--------------------'
~~~~
{: #riss-rate2 title="An Example of Handling of Replayed Initial Subscription" artwork-align="center"}

Suppose now that the xTR deletes it subscription. An attacker may replay valida Map-Request messages that were used for subscription or updates. These messages can't be detected by the Map-Server as being replay messages. The attacker may vary the source IP address of the Map-Request to trigger as many Map-Notifies sent to other xTRs.These Map-Notify messages will be ignored by the xTR as they don't have any matching state.

~~~~ aasvg
+---+                                              +----+
|xTR|                                              | MS |
+---+                                              +--+-+
  |    Map-Request(nonce, AFI=0...)                   |
  +==================================================>+
  |       Map-Notify(nonce, AFI=0...)                 |
  |<==================================================+
  |    Map-Notify-Ack                                 |
  +==================================================>+
  |                 +---+                             |
                    | AT|                             |
                    +-+-+                             |
                      | Map-Request(nonce,            | .--------------------.
                      |              key_id, ..)      | | Security/integrity |
                      +==============================>+-+ protection check.  |
                      |                               | | No state is found  |
  +---+                                               | | for xTR-ID/EID.    |
  |xTR|                                               | | Add a subscription |
  +-+-+        Map-Notify(nonce, ...)                 | | entry for this xTR |
    |<================================================+-+                    |
    |                 ...                             | |                    |
    |                                                 | '--------------------'
    |                                                 |
~~~~
{: #replay-no-state title="An Example of Handling of Replayed Map-Requests when no State" artwork-align="center"}

   Note that if LISP-SEC messages are timestamped, the replayed packets
   would be detected and, thus, be silently ignored by the Map-Server.
   Such invalid messages won't then interfere with legitimate Map-
   Requests if the Map-Server has sufficient resources to process the
   timestamp of all received requests.  An example of processing
   timestamped Map-Requests (rate-limit not reached) is depicted in
   {{replay-no-state-ts}}.

~~~~ aasvg
                  +---+                      +----+
                  | AT|                      | MS |
                  +-+-+                      +--+-+
                    |                           |
                    | Map-Request(init_nonce,   | .--------------------.
                    |          init_key_id,..)  | | Security/integrity |
                    +==========================>+-+ protection check.  |
                    |                           | | The message is     |
                                                | | discarded because  |
  +---+                                         | | timestamp checks   |
  |xTR|                                         | | fail               |
  +-+-+                                         | '--------------------'
    |                                           |
    |       Map-Request(...)                    | .--------------------.
    |==========================================>+-+  Processed         |
                                                | '--------------------'
~~~~ aasvg
{: #replay-no-state-ts title="An Example of Handling of Replayed Subscription with Timestamp" artwork-align="center"}

### Replayed Withdrawal

{{rew}} depicts the example of the exchange that occurs when an attacker sends a replayed withdrawal request. The request will be silently discared by the Map-Server if state is already present.

~~~~ aasvg
                     +---+                          +----+
                     | AT|                          | MS |
                     +-+-+                          +--+-+
                       |                               |
                       |                               | .--------------------.
                       | Map-Request(nonce, AFI=0,...) | | Security/integrity |
                       +==============================>+-+ protection check.  |
                       |                               | | A state is found   |
  +---+                                                | | xTR-ID/EID is found|
  |xTR|                                                | | but the nonce check|
  +-+-+                                                | | fails: rcv nonce < |
    |                                                  | | local nonce + 1    |
    |                                                  | | Discard the packet |
    |                                                  | '--------------------'
    |                                                  |
~~~~
{: #rew title="An Example of Handling of Replayed Removal of a Subscription" artwork-align="center"}

### Replayed Notification Updates

{{rmsw}} illustrates the observed exchange when a replayed notification update is sent by a misbehaving node (AT) to an xTR. This example assumes that the replayed message is a replay of Map-Server triggered withdrawal and that a state matching this notification is maintained by the xTR.

~~~~ aasvg
                     +---+                          +----+
                     |xTR|                          | AT |
                     +-+-+                          +--+-+
                       |                               |
.--------------------. |                               |
| Security/integrity | | Map-Notify(nonce, TTL=0, ...) |
| protection check.  +-+<==============================+
| Check that rcv     | |                               |
| nonce >= local     | |                               |
| nonce + 1          | |                               |
|                    | |                               |
| Discard the message| |                               |
| because the nonce  | |                               |
| checks fails       | |                               |
'--------------------' |                               |
                       |                               |
~~~~
{: #rmsw title="An Example of Replayed Notification of Subscription Withdrawal" artwork-align="center"}

Note that if no state is maintained by the xTR, the Map-Notify will be silently discarded.

# Explicit Subscriptions

TBC.

# Security Considerations

This document does not introduce any security considerations beyond those already discussed in {{!I-D.ietf-lisp-pubsub}}.

# IANA Considerations {#IANA}

This document does not make any request to IANA.

--- back

# Acknowledgments
{:numbered="false"}

Thanks to TBC.
