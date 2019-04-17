---
stand_alone: true
ipr: trust200902
docname: draft-petrov-lpwan-ipv6-schc-over-lorawan-latest
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
- ins: O. Gimenez
  name: Olivier Gimenez
  org: Semtech
  street: 14 Chemin des Clos
  city: Meylan
  country: France
  email: ogimenez@semtech.com
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
- ins: M. Le Gourrierec
  name: Marc Le Gourrierec
  org: SagemCom
  street: 250 Route de l'Empereur
  city: 92500 Rueil Malmaison
  country: FRANCE
  email: marc.legourrierec@sagemcom.com
normative:
  RFC2119:
  RFC4944:
  RFC5795:
  RFC7136:
  RFC3385:
  RFC4291:
informative:
  I-D.ietf-lpwan-overview:
  I-D.ietf-lpwan-ipv6-static-context-hc:
  lora-alliance-spec:
    title: LoRaWAN Specification Version V1.0.3
    author:
      name: LoRa Alliance
    date: 07.2018
    target: https://lora-alliance.org/sites/default/files/2018-07/lorawan1.0.3.pdf

--- abstract

The Static Context Header Compression (SCHC) specification describes generic
header compression and fragmentation techniques for LPWAN (Low Power Wide Area
Networks) technologies. SCHC is a generic mechanism designed for great
flexibility, so that it can be adapted for any of the LPWAN technologies.

This document provides the adaptation of SCHC for use in LoRaWAN networks, and
provides elements such as efficient parameterization and modes of operation.
This is called a profile.

--- middle

# Introduction {#Introduction}

The Static Context Header Compression (SCHC) specification
{{I-D.ietf-lpwan-ipv6-static-context-hc}} describes
generic header compression and fragmentation techniques that can be used on all
LPWAN (Low Power Wide Area Networks) technologies defined in
{{I-D.ietf-lpwan-overview}}. Even though those technologies share a great
number of common features like star-oriented topologies, network architecture,
devices with mostly quite predictable communications, etc; they do have some
slight differences in respect of payload sizes, reactiveness, etc.

SCHC gives a generic framework that enables those devices to communicate with
other Internet networks. However, for efficient performance, some parameters
and modes of operation need to be set appropriately for each of the LPWAN
technologies.

This document describes the efficient parameters and modes of operation when
SCHC is used over LoRaWAN networks.

# Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be
interpreted as described in {{RFC2119}}.

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
and the SCHC C/D is an Application Server. It can be provided by the Network
server or any third party software. {{Fig-archi}} can be map in LoRaWAN
terminology to:

~~~~
     Dev                                                            App
+--------------+                                             +--------------+
|APP1 APP2 APP3|                                             |APP1 APP2 APP3|
|              |                                             |              |
|      UDP     |                                             |     UDP      |
|     IPv6     |                                             |    IPv6      |
|              |                                             |              |
|   SCHC C/D   |                                             |              |
|   (context)  |                                             |              |
+-------+------+                                             +-------+------+
         |   +-------+     +-------+     +-----------+              .
         +~~ |Gateway| === |Network| === |Application|... Internet ..
             +-------+|    |Server |     |   Server  |
                           +-------+     +-----------+
~~~~
{: #Fig-archi-lorawan title='Architecture'}


#LoRaWAN Architecture

An overview of LoRaWAN {{lora-alliance-spec}} protocol and architecture is
described in {{I-D.ietf-lpwan-overview}}. Mapping between the LPWAN
architecture entities as described in {{I-D.ietf-lpwan-ipv6-static-context-hc}}
and the ones in {{lora-alliance-spec}} is as follows:

   o Devices (Dev) are the end-devices or hosts (e.g. sensors,
   actuators, etc.).  There can be a very high density of devices per
   radio gateway (LoRaWAN gateway). This entity maps to the LoRaWAN End-device.

   o The Radio Gateway (RGW), which is the end point of the constrained
   link. This entity maps to the LoRaWAN Gateway.

   o The Network Gateway (NGW) is the interconnection node between the
   Radio Gateway and the Internet. This entity maps to the LoRaWAN Network
   Server.

   o LPWAN-AAA Server, which controls the user authentication and the
   applications. This entity maps to the LoRaWAN Join Server.

   o Application Server (App). The same terminology is used in LoRaWAN.
   In that case, the application server will be the SCHC gateway, doing
   C/D and F/R.

~~~~

    ()   ()   ()       |                      +------+
     ()  () () ()     / \       +---------+   | Join |
    () () () () ()   /   \======|    ^    |===|Server|  +-----------+
     () ()  ()      |           | <--|--> |   +------+  |Application|
    () ()  ()  ()  / \==========|    v    |=============|  Server   |
     ()  ()  ()   /   \         +---------+             +-----------+
    End-Devices  Gateways     Network Server

~~~~
{: #Fig-LPWANarchi title='LPWAN Architecture'}

   SCHC C/D (Compressor/Decompressor) and SCHC F/R (Fragmentation/Reassembly)
   are performed on the LoRaWAN End-device and the Application Server (called
   SCHC gateway). While the point-to-point link between the End-device and the
   Application Server constitutes single IP hop, the ultimate end-point of the
   IP communication may be an Internet node beyond the Application Server.
   In other words, the LoRaWAN Application Server (SCHC gateway) acts as the
   first hop IP router for the End-device. The Application Server and Network
   Server may be co-located, which effectively turns the Network/Application
   Server into the first hop IP router.


## Device classes (A, B, C) and interactions

The LoRaWAN MAC layer supports 3 classes of devices named A, B and C.
All devices implement the classA, some devices implement classA+B or class
A+C. ClassB and classC are mutually exclusive.

* **ClassA**: The classA is the simplest class of devices. The device is
  allowed to transmit at any time, randomly selecting a communication channel.
  The network may reply with a downlink in one of the 2 receive windows
  immediately following the uplinks. Therefore, the network cannot initiate a
  downlink, it has to wait for the next uplink from the device to get a
  downlink opportunity. The classA is the lowest power device class.
* **ClassB**: classB devices implement all the functionalities of classA
  devices, but also schedule periodic listen windows. Therefore, as opposed the
  classA devices, classB devices can receive downlink that are initiated by the
  network and not following an uplink. There is a trade-off between the
  periodicity of those scheduled classB listen windows and the power
  consumption of the device. The lower the downlink latency, the higher the
  power consumption.
* **ClassC**: classC devices implement all the functionalities of classA
  devices, but keep their receiver open whenever they are not transmitting.
  ClassC devices can receive downlinks at any time at the expense of a higher
  power consumption. Battery powered devices can only operate in classC for a
  limited amount of time (for example for a firmware upgrade over the air).
  Most of the classC devices are main powered (for example Smart Plugs).

## Device addressing

LoRaWAN devices use a 32 bits network address (devAddr) to communicate with
the network over the air. However, that address might be reused several time
on the same network at the same time for different devices. Devices using the
same devAddr are distinguish by the network server based on the cryptographic
signature appended to every single LoRaWAN MAC frame, as all devices use
different security keys.  
To communicate with the SCHC gateway the network server MUST identify the
devices by a unique 64bits device ID called the devEUI. Unlike devAddr,
devEUI is guaranteed to be unique for every single device across all
networks.  
The devEUI is assigned to the device during the manufacturing process by the
device's manufacturer. It is built like an Ethernet MAC address by
concatenating the manufacturer's IEEE OUI field with a vendor unique number.
ex: 24bits OUI is concatenated with a 40 bits serial number.
The network server translates the devAddr into a devEUI in the uplink
direction and reciprocally on the downlink direction.

~~~~

+--------+         +---------------+        +----------+             +------------+
| device | <=====> | Network Server| <====> | GTW SCHC | <=========> |  Internet  |
+--------+ devAddr +---------------+ devEUI +----------+   IPv6/udp  +------------+

~~~~
{: #Fig-LoRaWANaddresses title='LoRaWAN addresses'}

## General Message Types

* **Confirmed messages**:
  The sender asks the receiver to acknowledge the message.
* **Unconfirmed messages**:
  The sender does not ask the receiver to acknowledge the message.

## LoRaWAN MAC Frames

* **JoinRequest**:
  This message is used by a device to join a network. It contains the device’s
  unique identifier devEUI and a random nonce that will be used for session key
  derivation.
* **JoinAccept**:
  To on-board a device, the network server responds to the JoinRequest device’s
  message with a JoinAccept message. That message is encrypted with the
  device’s AppKey and contains (amongst other fields) the major network’s
  settings and a network random nonce used to derive the session keys.
* **Data**

# SCHC over LoRaWAN

## LoRaWAN FPort

The LoRaWAN MAC layers features a port field in all frames. This port field
(FPort) is 8 bit long and the values from 1 to 223 can be used. It allows
LoRaWAN network and application to identify data.

A fragmentation session with application payload transfered from device
to server, is called uplink fragmentation session. It uses FPortUpShort or
FPortUpLong for data uplink and SCHC control downlinks.  
The other way, a fragmentation session with application payload transfered
from server to device, is called downlink fragmentation session. It uses
FPortDown for uplink and downlinks.

FPorts can use arbitrary values inside the allowed FPort range and must be
shared by the end-device, the network server and SCHC gateway. The uplink and
downlink SCHC ports must be different.  
In order to improve interoperability, it is recommended to use:

* FPortUpShort = 20
* FPortUpLong = 21
* FPportDown = 22

Those are recommended values and are application defined. Also application can
have multiple fragmentation session between a device and one or several SCHC
gateways. A set of three FPort values is required for each gateway instance the
end-device is required to communicate with.

The only uplink messages using the FPortDown port are the fragmentation SCHC
control messages of a downlink fragmentation session (ex ACKs). Similarly, the
only downlink messages using the FPortUpShort or FPortUpShort ports are the
fragmentation SCHC control messages of an uplink fragmentation session.

## Rule ID management

SCHC over LoRAWAN SHOULD support encoding RuleID on 6 bits for uplink
(64 possible rules) and 5 bits for downlinks (32 possible rules).  

The RuleID 0 is reserved for fragmentation in both directions.  
The uplink ruleID 63 is used to tag packets for which SCHC compression was not
possible (no matching Rule was found).  
The downlink ruleID 31 is used to tag packets for which SCHC compression was
not possible (no matching Rule was found).  

The remaining RuleIDs are available for header compression. RuleIDs are
independent. The same RuleID may have different meanings on the uplink and
downlink paths, at the exception of rules 0. A RuleId different from 0 means
that the fragmentation is not used, thus the packet should be send to C/D
layer.  

## IID computation

As LoRaWAN network uses unique EUI-64 per device, the Interface IDentifier is
the LoRaWAN DevEUI.  
It is compliant with [RFC4291] and IID starting with binary 000 must enforce
the 64-bits rule.

## Fragmentation {#Frag}

The L2 word size used by LoRaWAN is 1 byte (8 bits).  
The SCHC fragmentation over LoRaWAN uses the ACK-on-Error for uplink
fragmentation and Ack-Always for downlink fragmentation. A LoRaWAN device
cannot support simultaneous interleaved fragmentation sessions in the same
direction (uplink or downlink). This means that only a single fragmented
IPV6 datagram may be transmitted and/or received by the device at a given
moment.  

The fragmentation parameters are different for uplink and downlink
fragmentation sessions and are successively described in the next sections.

### Uplink fragmentation: From device to SCHC gateway

In that case the device is the fragmentation transmitter, and the SCHC gateway
the fragmentation receiver.  
Two fragmentation rules are defined regarding the FPort:

* **FPortUpShort**: SCHC header is only one byte. Used when compression is
  required and payload size is less than 381 bytes.
  bytes
* **FPortUpLong**: SCHC header is two bytes. Used for all other cases: no
  fragmentation required or payload size is between 382 and 1524 byte.

Both rules have share common parameters:

* **SCHC fragmentation reliability mode**: `ACK-on-Error`
* **DTag**: size is 1 bit. This field is used to clearly separate two consecutive
  fragmentation sessions. A LoRaWAN device cannot interleave several fragmented
  SCHC datagrams.
* **FCN**: The FCN field is encoded on N = 7 bits, so WINDOW_SIZE = 127 tiles are
  allowed in a window (FCN=All-1 is reserved for SCHC).
* **MIC calculation algorithm**: CRC32 using 0xEDB88320 (i.e. the reverse
  representation of the polynomial used e.g. in the Ethernet standard
  [RFC3385]) as suggested in {{I-D.ietf-lpwan-ipv6-static-context-hc}}.
* **MAX_ACK_REQUESTS**: 8
* **Tile**: size is 3 bytes (24 bits)
* **Retransmission and inactivity timers**:
  LoRaWAN devices do not implement a "retransmission timer". At the end of a
  window the ACK corresponding to this window is transmitted by the network
  gateway (LoRaWAN application server) in the RX1 or RX2 receive slot of
  device if tiles are missing.  
  If this ACK is not received the device sends an all-0 (or an all-1) fragment
  with no payload to request an ACK retransmission. The periodicity between
  retransmission of the all-0/all-1 fragments is device/application specific
  and may be different for each device (not specified). The SCHC gateway
  implements an "inactivity timer". The default recommended duration of this
  timer is 12h. This value is mainly driven by application requirements and may
  be changed by the application.

The following fields are differents:

* RuleId size
* Window index size W


#### FPortUpShort - 1 byte header
In that case RuleId size is 0, the rule is the FPort=FPortUpShort and only
fragmented payload can be transported.  
In order to minimise ACK, windows size is set to 0.

* **RuleId**: size is 0 bit in SCHC header, not used.
* **Window index**: encoded on W = 0 bit, not used

With this set of parameters, the SCHC fragment header overhead is 1 byte
(8 bits).  
MTU is: _127 tiles * 3 bytes per tile = 381 bytes_

**Regular fragments**

~~~~

| DTag  | FCN    | Payload |
+ ----- | ------ + ------- |
| 1 bit | 7 bits |         |

~~~~
{: #Fig-fragmentation-header-short-all0 title='All fragment except the last one. Header size is 8 bits (1 byte).'}


**All-1 ACK**

~~~~

| DTag  | C     | Encoded bitmap (if C = 0) | Padding (0s) |
+ ----- + ----- + ------------------------- + ------------ +
| 1 bit | 1 bit | 0 to 127 bits             | 6 to 0 bits  |

~~~~
{: #Fig-fragmentation-header-short-all1-ack title='ACK format for All-1 windows, failed mic check.'}

#### FPortUpLong - 2 bytes header

* **RuleId**: size is 6 bits (64 possible rules, 62 available for user)
* **Window index**: encoded on W = 2 bits. So 4 windows are available.

With this set of parameters, the SCHC fragment header overhead is 2 bytes
(16 bits).  
MTU is: _4 windows * 127 tiles * 3 bytes per tile = 1524 bytes_

_Note_: Even if it is less efficient, this rule can also be used for fragemented
payload size less than 382 bytes.

**Regular fragments**

~~~~

| RuleID | DTag  | W      | FCN    | Payload |
+ ------ + ----- + ------ | ------ + ------- +
| 6 bits | 1 bit | 2 bits | 7 bits |         |

~~~~
{: #Fig-fragmentation-header-long-all0 title='All fragment except the last one. Header size is 16 bits (2 bytes).'}


**Last fragment (All-1)**

~~~~

| RuleID | DTag  | W      | FCN=All-1 | MIC     |
+ ------ + ----- + ------ | --------- + ------- +
| 6 bits | 1 bit | 2 bits | 7 bits    | 32 bits |

~~~~
{: #Fig-fragmentation-header-all1 title='All-1 fragment detailed format for the last fragment.'}


**Windows ACK (All-0 ACK)**

~~~~

| RuleID | DTag  | W      | Encoded bitmap | Padding (0s) |
+ ------ + ----- + ------ | -------------- + ------------ +
| 6 bits | 1 bit | 2 bits | 0 to 127 bits   | 6 to 0 bits |

~~~~
{: #Fig-fragmentation-header-all0-ack title='ACK format for All-0 windows.'}


**All-1 ACK**

~~~~

| RuleID | DTag  | W     | C     | Encoded bitmap (if C = 0) | Padding (0s) |
+ ------ + ----- + ----- + ----- + ------------------------- + ------------ +
| 6 bits | 1 bit | 2 bit | 1 bit | 0 to 127 bits             | 6 to 0 bits  |

~~~~
{: #Fig-fragmentation-header-long-all1-ack title='ACK format for All-1 windows, failed mic check.'}


**Receiver-Abort**

~~~~

| RuleID | DTag  | W = b’11 | C = 1 | b’111111 | 0xFF (all 1's) |
+ ------ + ----- + -------- + ------+--------- + ---------------|
| 6 bits | 1 bit | 2 bits   | 1 bit | 6 bits   | 8 bits         |

~~~~
{: #Fig-fragmentation-receiver-abort title='Receiver-Abort format.'}


**SCHC acknowledge request**

~~~~

| RuleID | DTag  | W      | FCN = b’0000000 |
+ ------ + ----- + ------ + --------------- +
| 6 bits | 1 bit | 2 bits | 7 bits          |


~~~~
{: #Fig-fragmentation-schc-ack-req title='SCHC ACK REQ format.'}



### Downlink fragmentation: From SCHC gateway to device

In that case the device is the fragmentation receiver, and the SCHC gateway the
fragmentation transmitter. The following fields are common to all devices.

* **SCHC fragmentation reliability mode**: ACK-Always.
* **RuleId**: size is 5 bits (32 possible rules, 31 for user).
* **Window index**: encoded on W=1 bit, as per {{I-D.ietf-lpwan-ipv6-static-context-hc}}.
* **DTag**: size is 1 bit. This field is used to clearly separate two consecutive
  fragmentation sessions. A LoRaWAN device cannot interleave several fragmented
  SCHC datagrams.
* **FCN**: The FCN field is encoded on N=1 bits, so WINDOW_SIZE = 1 tile
  (FCN=All-1 is reserved for SCHC).
* **MIC calculation algorithm**: CRC32 using 0xEDB88320 (i.e. the reverse
  representation of the polynomial used e.g. in the Ethernet standard
  [RFC3385]), as per {{I-D.ietf-lpwan-ipv6-static-context-hc}}.
* **MAX_ACK_REQUESTS**: 8

As only 1 tile is used, its size can change for each downlink, and will be
maximum available MTU minus header (1 byte)

_Note_: The Fpending bit included in LoRaWAN protocol SHOULD not be used for
SCHC over LoRaWAN protocol. It might be set by the network server for other
purposes in but not SCHC needs.

**Regular fragments**

~~~~

| RuleID | DTag  | W     | FCN = b'0 | Payload |
+ ------ + ----- + ----- | --------- + ------- +
| 5 bits | 1 bit | 1 bit | 1 bits    | X bytes |

~~~~
{: #Fig-fragmentation-downlink-header-all0 title='All fragments but the last one. Header size 1 byte (8 bits).'}


**Last fragment (All-1)**

~~~~

| RuleID | DTag  | W     | FCN = b'1 | MIC     | Payload | Padding (0s) |
+ ------ + ----- + ----- | --------- + ------- + ------- + ------------ +
| 5 bits | 1 bit | 1 bit | 1 bits    | 32 bits | X bytes | 0 to 7 bits  |

~~~~
{: #Fig-fragmentation-downlink-header-all1 title='All-1 SCHC ACK detailed format for the last fragment.'}

**All-0 acknowledge**

~~~~

| RuleID | DTag  | W     | Encoded bitmap |
+ ------ + ----- + ----- | -------------- +
| 5 bits | 1 bit | 1 bit | 1 bit          |

~~~~
{: #Fig-fragmentation-header-downlink-all0-ack title='ACK format for All-0 windows.'}


**All-1 acknowledge**

~~~~

| RuleID | DTag  | W     | C = b’1 |
+ ------ + ----- + ----- + ------- +
| 5 bits | 1 bit | 1 bit | 1 bit   |

~~~~
{: #Fig-fragmentation-downlink-header-all1-ack title='ACK format for All-1 windows, MIC is correct.'}


**Receiver-Abort**

~~~~

| RuleID | DTag  | W     | C = b'0 | b’11111111 |
+ ------ + ----- + ----- + ------- + ---------- +
| 5 bits | 1 bit | 1 bit | 1 bits  | 8 bits     |

~~~~
{: #Fig-fragmentation-downlink-header-abort title='Receiver-Abort packet (following an all-1 packet with incorrect MIC).'}


Class A and classB&C devices do not manage retransmissions and timers in the
same way.

#### ClassA devices

Class A devices can only receive in an RX slot following the transmission of an
uplink.  Therefore there cannot be a concept of "retransmission timer" for an
SCHC gateway. The SCHC gateway cannot initiate communication to a classA
device.

The device replies with an ACK fragment to every single fragment received from
the SCHC gateway (because the window size is 1). Following the reception of a
FCN=0 fragment (fragment that is not the last fragment of the packet or
ACK-request, but the end of a window), the device MUST transmit the SCHC ACK
fragment until it receives the fragment of the next window. The device shall
transmit up to MAX_ACK_REQUESTS ACK fragments before aborting. The device
should transmit those ACK as soon as possible while taking into consideration
potential local radio regulation on duty-cycle, to progress the fragmentation
session as quickly as possible. The ACK bitmap is 1 bit long and is always 1.

Following the reception of a FCN=All-1 fragment (the last fragment of a
datagram) and if the MIC is correct, the device shall transmit the ACK with
the "MIC is correct" indicator bit set (C=1). This message might be lost
therefore the SCHC gateway may request a retransmission of this ACK in the next
downlink. The device SHALL keep this ACK message in memory until it receives
a downlink, on SCHC FPortDown from the SCHC gateway different from an
ACK-request: it indicates that the SCHC gateway has received the ACK message.

Following the reception of a FCN=All-1 fragment (the last fragment of a
datagram), if all fragments have been received and the MIC is NOT correct,
the device shall transmit a Receiver-Abort fragment. The device SHALL keep
this Abort message in memory until it receives a downlink, on SCHC FPortDown,
from the SCHC gateway different from an ACK-request indicating that the SCHC
gateway has received the Abort message.  The fragmentation receiver (device)
does not implement retransmission timer and inactivity timer.

The fragmentation sender (the SCHC gateway) implements an inactivity timer with
default duration of 12 hours. Once a fragmentation session is started, if the
SCHC gateway has not received any ACK or Receiver-Abort message 12 hours after
the last message from the device was received, the SCHC gateway may flush the
fragmentation context.  For devices with very low transmission rates
(example 1 packet a day in normal operation) , that duration may be extended,
but this is application specific.


#### Class B or C devices

Class B&C devices can receive in scheduled RX slots or in RX slots following
the transmission of an uplink. The device replies with an ACK fragment to
every single fragment received from the SCHC gateway (because the window size
is 1). Following the reception of a FCN=0 fragment (fragment that is not the
last fragment of the packet or ACK-request), the device MUST always transmit
the corresponding SCHC ACK fragment even if that fragment has already been
received.  
The ACK bitmap is 1 bit long and is always 1. If the SCHC gateway receives this
ACK, it proceeds to send the next window fragment If the retransmission timer
elapses and the SCHC gateway has not received the ACK of the current window it
retransmits the last fragment. The SCHC gateway tries retransmitting up to
MAX_ACK_REQUESTS times before aborting.

Following the reception of a FCN=All-1 fragment (the last fragment of a
datagram) and if the MIC is correct, the device shall transmit the ACK with the
“MIC is correct” indicator bit set. If the SCHC gateway receives this ACK, the
current fragmentation session has succeeded and its context can be cleared.

If the retransmission timer elapses and the SCHC gateway has not received the
All-1 ACK it retransmits the last fragment with the payload (not an ACK-request
without payload). The SCHC gateway tries retransmitting up to MAX_ACK_REQUESTS
times before aborting.

The device SHALL keep the All-1 ACK message in memory until it receives a
downlink from the SCHC gateway different from the last (FCN>0 and different
DTag) fragment indicatingthat the SCHC gateway has received the ACK message.

Following the reception of a FCN=All-1 fragment (the last fragment of a
datagram), if all fragments have been received and if the MIC is NOT correct,
the device shall transmit a Receiver-Abort fragment.  The retransmission
timer is used by the SCHC gateway (the sender), the optimal value is very much
application specific but here are some recommended default values.  
For classB devices, this timer trigger is a function of the periodicity of the
classB ping slots. The recommended value is equal to 3 times the classB ping
slot periodicity. For classC devices which are nearly constantly receiving,
the recommended value is 30 seconds. This means that the device shall try to
transmit the ACK within 30 seconds  of the reception of each fragment.  The
inactivity timer is implemented by the device to flush the context in-case it
receives nothing from the SCHC gateway over an extended period of time. The
recommended value is 12 hours for both classB&C devices.

# Security considerations

This document is only providing parameters that are expected to be better
suited for LoRaWAN networks for {{I-D.ietf-lpwan-ipv6-static-context-hc}}. As
such, this parameters does not contribute to any new security issues in
addition of those identified in {{I-D.ietf-lpwan-ipv6-static-context-hc}}.

# Acknowledgementss
TBD

--- back

# Examples

# Note
