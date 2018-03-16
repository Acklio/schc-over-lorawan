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
abbrev: SCHC-over-LoRaWAN
wg: lpwan Working Group
author:
- ins: N. Sornin
  name: Nicolas Sornin
  org: Semtech
  street: 14 Chemin des Clos
  city: Meylan
  country: France
  email: nsornin@semtech.com
  role: editor
- ins: M. Coracin
  name: Michael Coracin
  org: Semtech
  street: 14 Chemin des Clos
  city: Meylan
  country: France
  email: mcoracin@semtech.com
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
- ins: J. Catalano
  name: Julien Catalano
  org: Kerlink
  street: 1 rue Jacqueline Auriol
  city: 35235 Thorigné-Fouillard
  country: France
  email: j.catalano@kerlink.fr
- ins: V. Audebert
  name: Vincent AUDEBERT
  org: EDF R&D
  street: 7 bd Gaspard Monge
  city: 91120 PALAISEAU
  country: FRANCE
  email: vincent.audebert@edf.fr
normative:
  RFC4944:
  RFC5795:
  RFC7136:
informative:
  I-D.ietf-lpwan-overview:
  I-D.ietf-lpwan-ipv6-static-context-hc:
  lora-alliance-spec:
    title: LoRaWAN Specification Version V1.0.2
    author:
      name: LoRa Alliance
    date: 07.2016
    target: http://portal.lora-alliance.org/DesktopModules/Inventures_Document/FileDownload.aspx?ContentID=1398

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

#LoRaWAN Architecture

An overview of LoRaWAN {{lora-alliance-spec}} protocol and architecture is
described in {{I-D.ietf-lpwan-overview}}. Mapping between the LPWAN
architecture entities as described in {{I-D.ietf-lpwan-ipv6-static-context-hc}}
and the ones in {{lora-alliance-spec}} is as follows:

   o Devices (Dev) are the end-devices or hosts (e.g. sensors,
   actuators, etc.).  There can be a very high density of devices per
   radio gateway. This entity maps to the LoRaWAN End-device.

   o The Radio Gateway (RGW), which is the end point of the constrained
   link. This entity maps to the LoRaWAN Gateway.

   o The Network Gateway (NGW) is the interconnection node between the
   Radio Gateway and the Internet. This entity maps to the LoRaWAN Network
   Server.

   o LPWAN-AAA Server, which controls the user authentication and the
   applications. This entity maps to the LoRaWAN Join Server.

   o Application Server (App). The same terminology is used in LoRaWAN.


    ()   ()   ()       |                      +------+
     ()  () () ()     / \       +---------+   | Join |
    () () () () ()   /   \======|    ^    |===|Server|  +-----------+
     () ()  ()      |           | <--|--> |   +------+  |Application|
    () ()  ()  ()  / \==========|    v    |=============|  Server   |
     ()  ()  ()   /   \         +---------+             +-----------+
    End-Devices  Gateways     Network Server


                       Figure 1: LPWAN/LoRaWAN Architecture

   SCHC C/D (Compressor/Decompressor) and SCHC Fragmentation are performed on
   the LoRaWAN End-device and the Application Server. While the point-to-point
   link between the End-device and the Application Server constitutes single IP
   hop, the ultimate end-point of the IP communication may be an Internet node
   beyond the Application Server. In other words, the LoRaWAN Application
   Server acts as the first hop IP router for the End-device. Note that the
   Application Server and Network Server may be co-located, which effectively
   turns the Network/Application Server into the first hop IP router.


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
