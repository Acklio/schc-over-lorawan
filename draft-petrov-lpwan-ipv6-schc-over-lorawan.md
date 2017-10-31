---
stand_alone: true
ipr: trust200902
docname: draft-petrov-lpwan-ipv6-schc-over-lorawan-00
cat: info
pi:
  symrefs: 'yes'
  sortrefs: 'yes'
  strict: 'yes'
  compact: 'yes'
  toc: 'yes'
title: Static Context Header Compression (SCHC) over LoRaWAN
abbrev: SCHC-over-LORA
wg: lpwan Working Group
author:
- ins: I. Petrov
  name: Ivaylo Petrov
  org: Acklio
  street: 2bis rue de la Chataigneraie
  city: 35510 Cesson-Sevigne Cedex
  country: France
  email: ivaylo@ackl.io
- ins: A. Yegin
  name: Alper Yegin
  org: Actility
  street: .
  city: Paris, Paris
  country: France
  email: alper.yegin@actility.com
normative:
  RFC4944:
  RFC5795:
  RFC7136:
informative:
  I-D.ietf-lpwan-overview:
  I-D.ietf-lpwan-ipv6-static-context-hc:

--- abstract

The Static Context Header Compression (SCHC) specification describes generic
header compression and fragmentation techniques for LPWAN (Low Power Wide Area
Networks) technologies. SCHC is a generic mechanism designed for great
flexibility, so that it can be adapted for any of the LPWAN technologies.

This document provides the adaptation of SCHC for use in LoRaWAN networks, and
provides elements such as efficient parameterization and modes of operation.

--- middle

# Introduction {#Introduction}

The Static Context Header Compression (SCHC) specification
{{I-D.ietf-lpwan-ipv6-static-context-hc}} describes
generic header compression and fragmentation techniques that can be used on all
LPWAN (Low Power Wide Area Networks) technologies defined in
{{I-D.ietf-lpwan-overview}}. Even though those technologies share a great
number of common features like start-oriented topologies, network architecture,
devices with mostly quite predictable communications, etc; they do have some
slight differences in respect of payload sizes, reactiveness, etc.

SCHC gives a generic framework that enables those devices to communicate with
other Internet networks. However, for efficient performance, some parameters
and modes of operation need to be set appropriately for each of the LPWAN
technologies.

This document describes the efficient parameters and modes of operation when
SCHC is used over LoRaWAN networks.

# Terminology

This section defines the terminology and acronyms used in this document. For
all other definitions, please look up the SCHC specification
{{I-D.ietf-lpwan-ipv6-static-context-hc}}.

  o  DevEUI: an IEEE EUI-64 identifier used to identify the device during the
     procedure while joining the network (Join Procedure)

  o  DevAddr: a 32-bit non-unique identifier assigned to a device statically or
     dynamically after a Join Procedure (depending on the activation mode)

  o  TBD: all significant LoRaWAN-related terms.

# Static Context Header Compression Overview

This section contains a short overview of Static Context Header Compression
(SCHC). For a detailed description, refer to the full specification
{{I-D.ietf-lpwan-ipv6-static-context-hc}}.

Static Context Header Compression (SCHC) avoids context synchronization, which
is the most bandwidth-consuming operation in other header compression
mechanisms such as RoHC {{RFC5795}}. Based on the fact that the nature of data
flows is highly predictable in LPWAN networks, some static contexts may be
stored on the Device (Dev). The contexts must be stored in both ends, and it
can either be learned by a provisioning protocol or by out of band means or it
can be pre-provisioned, etc. The way the context is learned on both sides is
out of the scope of this document.


~~~~
     Dev                                                 App
+--------------+                                  +--------------+
|APP1 APP2 APP3|                                  |APP1 APP2 APP3|
|              |                                  |              |
|      UDP     |                                  |     UDP      |
|     IPv6     |                                  |    IPv6      |
|              |                                  |              |
|   SCHC C/D   |                                  |              |
|   (context)  |                                  |              |
+-------+------+                                  +-------+------+
         |   +--+     +----+     +---------+              .
         +~~ |RG| === |NGW | === |SCHC C/D |... Internet ..
             +--+     +----+     |(context)|
                                 +---------+
~~~~
{: #Fig-archi title='Architecture'}

{{Fig-archi}} represents the architecture for compression/decompression, it is
based on {{I-D.ietf-lpwan-overview}} terminology. The Device is sending
applications flows using IPv6 or IPv6/UDP protocols. These flows are compressed
by an Static Context Header Compression Compressor/Decompressor (SCHC C/D) to
reduce headers size. Resulting information is sent on a layer two (L2) frame to
a LPWAN Radio Network (RG) which forwards the frame to a Network Gateway (NGW).
The NGW sends the data to a SCHC C/D for decompression which shares the same
rules with the Dev. The SCHC C/D can be located on the Network Gateway (NGW) or
in another place as long as a tunnel is established between the NGW and the
SCHC C/D. The SCHC C/D in both sides must share the same set of Rules. After
decompression, the packet can be sent on the Internet to one or several LPWAN
Application Servers (App).

The SCHC C/D process is bidirectional, so the same principles can be applied in
the other direction.

In a LoRaWAN network, the RG is called a Gateway, the NGW is Network Server,
and the SCHC C/D can be embedded in different places, for example in the
Network Server and/or the Application Server.

Next steps for this section: detailed overview of the LoRaWAN architecture and
its mapping to the SCHC architecture.

# LoRaWAN Overview

## Device classes (A, B, C) and interactions
TBD

## Device addressing
TBD

## General Message Types
TBD

## LoRaWAN MAC Frames
TBD

# SCHC over LoRaWAN

## Rule ID management

Rule ID can be stored and transported in the FPort field of the LoRaWAN MAC
frame.

TBD

## IID computation
TBD

## Fragmentation {#Frag}
TBD

### Reliability options
TBD

### Supporting multiple window sizes
TBD

### Downlink fragment transmission
TBD

### SCHC behavior for devices in class A, B and C
TBD

# Security considerations
TBD

# Acknowledgements
TBD

--- back

# Examples

# Note
