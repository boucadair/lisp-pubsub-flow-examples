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

# Terminology

This document uses the terms defined in {{!I-D.ietf-lisp-pubsub}}.

The following terms and notations are used in this document:

initial_nonce:
: the nonce that is initially included in a Map-Request to create a subscription.

initial subscription request:
: the Map-Request that was used to create the initial subscription. This request has the nonce value set to initial_nonce.

nonce++:
: incremented nonce by 1.

initial_key_id:
: the key identifier that was used in the Map-Request with initial_nonce.


# Initial Successful Subscription {#sec-iss}

The following example assumes that a security association is in place between xTR/Map-Server (Section 7.1 of {{!I-D.ietf-lisp-pubsub}}) and that integrity-protection checks were successfully passed.

~~~~ aasvg
+---+                                            +----+
|xTR|                                            | MS |
+-+-+                                            +--+-+
  | Map-Request(initial_nonce, initial_key_id,..)   |
  |================================================>|
  |                                                 |
  |                                                 |
  |                                                 |
~~~~
{: #iss title="xxxx" artwork-align="center"}


# Successful Notification

XXXXX

# Successful Notification

XXXXX


# Successful Subscription Update

XXXXX

# Failed Subscription Update

XXXXX

# xTR-triggered Subscription Withdrawal


# 'Map-Server'-triggered Subscription Withdrawal



# Security Considerations

This document does not introduce any security considerations beyond those already discussed in {{!I-D.ietf-lisp-pubsub}}.

# IANA Considerations {#IANA}

This document does not make any request to IANA.

--- back

# Acknowledgments
{:numbered="false"}

Thanks to TBC.
