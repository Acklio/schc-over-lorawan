---
stand_alone: true
ipr: trust200902
docname: draft-ietf-lpwan-schc-over-lorawan-latest
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
- ins: O. Gimenez
  name: Olivier Gimenez
  org: Semtech
  street: 14 Chemin des Clos
  city: Meylan
  country: France
  email: ogimenez@semtech.com
  role: editor
- ins: I. Petrov
  name: Ivaylo Petrov
  org: Acklio
  street: 2bis rue de la Chataigneraie
  city: 35510 Cesson-Sevigne Cedex
  country: France
  email: ivaylo@ackl.io
  role: editor
- ins: J. Catalano
  name: Julien Catalano
  org: Kerlink
  street: 1 rue Jacqueline Auriol
  city: 35235 Thorigné-Fouillard
  country: France
  email: j.catalano@kerlink.fr
normative:
  RFC2119:
  RFC4944:
  RFC5795:
  RFC7136:
  RFC3385:
  RFC4291:
  RFC8376:
informative:
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
{{RFC8376}}. Even though those technologies share a great
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

  o  DevEUI: an IEEE EUI-64 identifier used to identify the end-device during the
     procedure while joining the network (Join Procedure)

  o  DevAddr: a 32-bit non-unique identifier assigned to a end-device statically or
     dynamically after a Join Procedure (depending on the activation mode)

  o  RCS: Reassembly Check Sequence. Used to verify the integrity of the
     fragmentation-reassembly process

  o  TBD: all significant LoRaWAN-related terms.

# Static Context Header Compression Overview

This section contains a short overview of Static Context Header Compression
(SCHC). For a detailed description, refer to the full specification
{{I-D.ietf-lpwan-ipv6-static-context-hc}}.

Static Context Header Compression (SCHC) avoids context synchronization, based
on the fact that the nature of data flows is highly predictable in LPWAN
networks, some static contexts may be stored on the Device (Dev). The contexts
MUST be stored in both ends, and it can either be learned by a provisioning
protocol or by out-of-band means or it can be pre-provisioned, etc. The way the
context is learned on both sides is out of the scope of this document.


~~~~
        Dev                                               App
+----------------+                                +----+ +----+ +----+
| App1 App2 App3 |                                |App1| |App2| |App3|
|                |                                |    | |    | |    |
|       UDP      |                                |UDP | |UDP | |UDP |
|      IPv6      |                                |IPv6| |IPv6| |IPv6|
|                |                                |    | |    | |    |
|SCHC C/D and F/R|                                |    | |    | |    |
+--------+-------+                                +----+ +----+ +----+
         |  +---+     +----+    +----+    +----+     .      .      .
         +~ |RGW| === |NGW | == |SCHC| == |SCHC|...... Internet ....
            +---+     +----+    |F/R |    |C/D |
                                +----+    +----+
~~~~
{: #Fig-archi title='Architecture'}

{{Fig-archi}} represents the architecture for compression/decompression, it is
based on {{RFC8376}} terminology. The Device is sending applications flows
using IPv6 or IPv6/UDP protocols. These flow might be compressed by an Static
Context Header Compression Compressor/Decompressor (SCHC C/D) to reduce headers
size and fragmented (SCHC F/R).  Resulting information is sent on a layer two
(L2) frame to a LPWAN Radio Gateway (RGW) which forwards the frame to a Network
Gateway (NGW). The NGW sends the data to a SCHC F/R for defragmentation, if
required, then C/D for decompression which shares the same rules with the
device. The SCHC F/R and C/D can be located on the Network Gateway (NGW) or in
another place as long as a tunnel is established between the NGW and the SCHC
F/R, then SCHC F/R and SCHC C/D. The SCHC C/D in both sides MUST share the same
set of Rules. After decompression, the packet can be sent on the Internet to
one or several LPWAN Application Servers (App).

The SCHC F/R and SCHC C/D process is bidirectional, so the same principles can
be applied in the other direction.

In a LoRaWAN network, the RG is called a Gateway, the NGW is Network Server,
and the SCHC C/D is an Application Server. It can be provided by the Network
Server or any third party software. {{Fig-archi}} can be map in LoRaWAN
terminology to:

~~~~
        Dev                                                         App
+----------------+                                          +----+ +----+ +----+
| App1 App2 App3 |                                          |App1| |App2| |App3|
|                |                                          |    | |    | |    |
|       UDP      |                                          |UDP | |UDP | |UDP |
|      IPv6      |                                          |IPv6| |IPv6| |IPv6|
|                |                                          |    | |    | |    |
|SCHC C/D and F/R|                                          |    | |    | |    |
+--------+-------+                                          +----+ +----+ +----+
         |  +-------+     +-------+    +----------------+     .      .      .
         +~ |Gateway| === |Network| == |Application     |...... Internet ....
            +-------+     |server |    |server F/R - C/D|
                          +-------+    +----------------+
~~~~
{: #Fig-archi-lorawan title='SCHC Architecture mapped to LoRaWAN'}


#LoRaWAN Architecture

An overview of LoRaWAN {{lora-alliance-spec}} protocol and architecture is
described in {{RFC8376}}. Mapping between the LPWAN
architecture entities as described in {{I-D.ietf-lpwan-ipv6-static-context-hc}}
and the ones in {{lora-alliance-spec}} is as follows:

   o Devices (Dev) are the end-devices or hosts (e.g. sensors,
   actuators, etc.).  There can be a very high density of devices per
   radio gateway (LoRaWAN gateway). This entity maps to the LoRaWAN End-Device.

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
   are performed on the LoRaWAN End-Device and the Application Server (called
   SCHC gateway). While the point-to-point link between the End-Device and the
   Application Server constitutes single IP hop, the ultimate end-point of the
   IP communication may be an Internet node beyond the Application Server.
   In other words, the LoRaWAN Application Server (SCHC gateway) acts as the
   first hop IP router for the End-Device. The Application Server and Network
   Server may be co-located, which effectively turns the Network/Application
   Server into the first hop IP router.


## End-Device classes (A, B, C) and interactions

The LoRaWAN MAC layer supports 3 classes of end-devices named A, B and C.  All
end-devices implement the Class A, some end-devices MAY implement Class B or
class C. Class B and Class C are mutually exclusive.

* **Class A**: The Class A is the simplest class of end-devices. The end-device is
  allowed to transmit at any time, randomly selecting a communication channel.
  The network may reply with a downlink in one of the 2 receive windows
  immediately following the uplinks. Therefore, the network cannot initiate a
  downlink, it has to wait for the next uplink from the end-device to get a
  downlink opportunity. The Class A is the lowest power end-device class.
* **Class B**: Class B end-devices implement all the functionalities of Class A
  end-devices, but also schedule periodic listen windows. Therefore, as opposed the
  Class A end-devices, Class B end-devices can receive downlink that are initiated by the
  network and not following an uplink. There is a trade-off between the
  periodicity of those scheduled Class B listen windows and the power
  consumption of the end-device. The lower the downlink latency, the higher the
  power consumption.
* **Class C**: Class C end-devices implement all the functionalities of Class A
  end-devices, but keep their receiver open whenever they are not transmitting.
  Class C end-devices can receive downlinks at any time at the expense of a higher
  power consumption. Battery powered end-devices can only operate in Class C for a
  limited amount of time (for example for a firmware upgrade over-the-air).
  Most of the Class C end-devices are mains powered (for example Smart Plugs).

## End-Device addressing

LoRaWAN end-devices use a 32-bit network address (devAddr) to communicate with
the network over-the-air. However, that address might be reused several time
on the same network at the same time for different end-devices. End-devices using the
same devAddr are distinguish by the Network Server based on the cryptographic
signature appended to every single LoRaWAN MAC frame, as all end-devices use
different security keys.
To communicate with the SCHC gateway the Network Server MUST identify the
end-devices by a unique 64-bit device identifier called the devEUI. Unlike
devAddr, devEUI is guaranteed to be unique for every single end-device across
all networks.
The devEUI is assigned to the end-device during the manufacturing process by the
end-device's manufacturer. It is built like an Ethernet MAC address by
concatenating the manufacturer's IEEE OUI field with a vendor unique number.
e.g.: 24-bit OUI is concatenated with a 40-bit serial number.
The Network Server translates the devAddr into a devEUI in the uplink
direction and reciprocally on the downlink direction.

~~~~

+--------+         +----------+        +---------+            +----------+
| End-   | <=====> | Network  | <====> | SCHC    | <========> | Internet |
| Device | devAddr | Server   | devEUI | Gateway |  IPv6/UDP  |          |
+--------+         +----------+        +---------+            +----------+

~~~~
{: #Fig-LoRaWANaddresses title='LoRaWAN addresses'}

## General Message Types

* **Confirmed messages**:
  The sender asks the receiver to acknowledge the message.
* **Unconfirmed messages**:
  The sender does not ask the receiver to acknowledge the message.

As SCHC defines its own acknowledgment mechanisms, SCHC does not require to use
confirmed messages.

## LoRaWAN MAC Frames

* **JoinRequest**:
  This message is used by a end-device to join a network. It contains the end-device's
  unique identifier devEUI and a random nonce that will be used for session key
  derivation.
* **JoinAccept**:
  To on-board a end-device, the Network Server responds to the JoinRequest end-device's
  message with a JoinAccept message. That message is encrypted with the
  end-device's AppKey and contains (amongst other fields) the major network's
  settings and a network random nonce used to derive the session keys.
* **Data**

# SCHC-over-LoRaWAN

## LoRaWAN FPort

The LoRaWAN MAC layer features a frame port field in all frames. This field
(FPort) is 8-bit long and the values from 1 to 223 can be used. It allows
LoRaWAN networks and applications to identify data.

The FPort field is part of the SCHC Packet or the SCHC Fragment, as shown in
{{Fig-lorawan-schc-payload}}. The SCHC C/D and the SCHC F/R SHALL concatenate
the FPort field with the LoRaWAN payload to retrieve their payload as it is used
as a part of the ruleId field.

~~~~

| FPort | LoRaWAN payload  |
+ ------------------------ +
|       SCHC payload       |

~~~~
{: #Fig-lorawan-schc-payload title='SCHC payload in LoRaWAN'}

A fragmentation session with application payload transferred from device to
server, is called uplink fragmentation session. It uses a range of FPorts for
data uplink and its associated SCHC control downlinks, named FPortsUp in this
document. The other way, a fragmentation session with application payload
transferred from server to device, is called downlink fragmentation session. It
uses another range of FPorts for data downlink and its associated SCHC control
uplinks, named FPortsDown in this document.

FPorts can use arbitrary values inside the allowed FPort range and MUST be
shared by the end-device, the Network Server and SCHC gateway. The uplink and
downlink fragmentation FPorts MUST be different. Mechanism to share those FPorts
values is out-of-scope of this document. In order to improve interoperability,
it is RECOMMENDED to use:

* RuleID = b'TBC (8-bit) for uplink fragmentation, named FPortUp
* RuleID = b'TBC (8-bit) for downlink fragmentation, named FPortDown
* 8-bit RuleIDs for compression, in the range [1;223] except values previously
  defined RuleID fragmentation.

Application can have multiple fragmentation sessions between a device and one or
several SCHC gateways. A set of FPort values is REQUIRED for each gateway
instance the device is required to communicate with.

The only uplink messages using the FPortsDown port are the fragmentation SCHC
control messages of a downlink fragmentation session (ex ACKs). Similarly, the
only downlink messages using the FPortsUp ports are the fragmentation SCHC
control messages of an uplink fragmentation session.

## Rule ID management

RuleID is encoded in the LoRaWAN FPort for SCHC C/D. LoRaWAN supports up to 222
application FPorts in range [1;223]. SCHC-over-LoRaWAN defines FPort ranges
dedicated to SCHC F/R. Application MAY reserve some FPort values for other needs.
A RuleID SHOULD be reserved to tag packets for which SCHC compression was not
possible (no matching Rule was found), the RECOMMANDED value is b'TBC (8-bit).

RuleIDs FPortsUp and FPortsDown are reserved for fragmentation.

The remaining RuleIDs are available for compression. RuleIDs are shared between
uplink and downlink sessions.  A RuleID not FPortsUp nr FPortsDown means that
the fragmentation is not used, thus the packet SHOULD be send to C/D
layer.

## IID computation

As LoRaWAN network uses unique EUI-64 per end-device, the Interface IDentifier is
the LoRaWAN DevEUI.
It is compliant with [RFC4291] and IID starting with binary 000 must enforce
the 64-bit rule.

TODO: Derive IID from DevEUI with privacy constraints ? Ask working group ?

## Compression {#Comp}

SCHC C/D MUST concatenate FPort and LoRaWAN payload to retrieve the SCHC packet
as per [](#Fig-fragmentation-receiver-abort).

SCHC C/D RuleID size SHOULD be 8 bits to fit the LoRaWAN FPort field. RuleIDs
matching FPortsUp and FPortsDown are reserved for SCHC Fragmentation.


## Fragmentation {#Frag}

The L2 word size used by LoRaWAN is 1 byte (8 bits).
The SCHC fragmentation over LoRaWAN uses the ACK-on-Error for uplink
fragmentation and Ack-Always for downlink fragmentation. A LoRaWAN end-device
cannot support simultaneous interleaved fragmentation sessions in the same
direction (uplink or downlink). This means that only a single fragmented
IPv6 datagram may be transmitted and/or received by the end-device at a given
moment.

The fragmentation parameters are different for uplink and downlink
fragmentation sessions and are successively described in the next sections.

## DTag
A LoRaWAN device cannot interleave several fragmented SCHC datagrams. This one
bit field is used to distinguish two consecutive fragmentation sessions.

_Note_: While it is used to recover faster from transmission errors, it SHALL
NOT be considered as the only way to distinguish two fragmentation sessions.

### Uplink fragmentation: From device to SCHC gateway

In that case the device is the fragmentation transmitter, and the SCHC gateway
the fragmentation receiver.
A single fragmentation rule is defined.
SCHC F/R MUST concatenate FPort and LoRaWAN payload to retrieve the SCHC
fragment as per [](#Fig-fragmentation-receiver-abort).

* SCHC header is two bytes, including FPort byte.
* **RuleID**: size is 8 bits in SCHC header.
* **SCHC fragmentation reliability mode**: `ACK-on-Error`
* **DTag**: size is 1 bit.
* **FCN**: The FCN field is encoded on N = 7 bits, so WINDOW_SIZE = 127 tiles
  are allowed in a window (FCN=All-1 is reserved for SCHC).
* **Window index**: encoded on W = 2 bits. So 4 windows are available.
* **RCS calculation algorithm**: CRC32 using 0xEDB88320 (i.e. the reverse
  representation of the polynomial used e.g. in the Ethernet standard
  [RFC3385]) as suggested in {{I-D.ietf-lpwan-ipv6-static-context-hc}}.
* **MAX_ACK_REQUESTS**: 8
* **Tile**: size is 3 bytes (24 bits)
* **Retransmission and inactivity timers**:
  LoRaWAN end-devices do not implement a "retransmission timer". At the end of
  a window or a fragmentation session, corresponding ACK(s) is (are)
  transmitted by the network gateway (LoRaWAN application server) in the RX1 or
  RX2 receive slot of end-device.
  If this ACK is not received the end-device sends an all-0 (or an all-1)
  fragment with no payload to request an SCHC ACK retransmission. The
  periodicity between retransmission of the all-0/all-1 fragments is
  device/application specific and MAY be different for each device (not
  specified). The SCHC gateway implements an "inactivity timer". The default
  RECOMMENDED duration of this timer is 12 hours. This value is mainly driven
  by application requirements and MAY be changed by the application.
* **Last tile**: Last tile can be carried in the All-1 fragment.

With this set of parameters, the SCHC fragment header is 18 bits,
including FPort; payload overhead will be 10 bits as FPort is already a part of
LoRaWAN payload. MTU is: _4 windows * 127 tiles * 3 bytes per tile = 1524 bytes_

**Regular fragments**

~~~~

| FPort  |  LoRaWAN payload                  |
+ ------ + --------------------------------- +
| RuleID | DTag  | W      | FCN    | Payload |
+ ------ + ----- + ------ + ------ + ------- +
| 8 bits | 1 bit | 2 bits | 7 bits |         |

~~~~
{: #Fig-fragmentation-header-long-all0 title='All fragment except the last one. SCHC header size is 18 bits, including LoRaWAN FPort.'}


**Last fragment (All-1)**

~~~~

| FPort  | LoRaWAN payload                                          |
+ ------ + -------------------------------------------------------- +
| RuleID | DTag  | W      | FCN=All-1 | RCS     | Payload           |
+ ------ + ----- + ------ + --------- + ------- + ----------------- +
| 8 bits | 1 bit | 2 bits | 7 bits    | 32 bits | Last tile, if any |

~~~~
{: #Fig-fragmentation-header-all1 title='All-1 fragment detailed format for the last fragment.'}

**SCHC ACK**

~~~~

| FPort  | LoRaWAN payload                                   |
+ ------ + ------------------------------------------------- +
| RuleID | DTag  | W     | C     | Encoded bitmap (if C = 0) |
+ ------ + ----- + ----- + ----- + ------------------------- +
| 8 bits | 1 bit | 2 bit | 1 bit | 0 to 127 bits             |

~~~~
{: #Fig-fragmentation-header-long-schc-ack title='SCHC formats, failed RCS check.'}


**Receiver-Abort**

~~~~

| FPort  | LoRaWAN payload                                     |
+ ------ + --------------------------------------------------- +
| RuleID | DTag  | W = b'11 | C = 1 | b'1111 | 0xFF (all 1's)  |
+ ------ + ----- + -------- + ------+------- + ----------------+
| 8 bits | 1 bit | 2 bits   | 1 bit | 4 bits | 8 bits          |
                     next L2 Word boundary ->| <-- L2 Word --> |

~~~~
{: #Fig-fragmentation-receiver-abort title='Receiver-Abort format.'}


**SCHC acknowledge request**

~~~~

| FPort  | LoRaWAN payload                  |
+------- +--------------------------------- +
| RuleID | DTag  | W      | FCN = b'0000000 |
+ ------ + ----- + ------ + --------------- +
| 8 bits | 1 bit | 2 bits | 7 bits          |


~~~~
{: #Fig-fragmentation-schc-ack-req title='SCHC ACK REQ format.'}



### Downlink fragmentation: From SCHC gateway to device

In that case the device is the fragmentation receiver, and the SCHC gateway the
fragmentation transmitter. The following fields are common to all devices.
SCHC F/R MUST concatenate FPort and LoRaWAN payload to retrieve the SCHC
fragment as described in {{Fig-lorawan-schc-payload}}.

* **SCHC fragmentation reliability mode**: ACK-Always.
* **RuleID**: size is 8 bits in SCHC header.
* **Window index**: encoded on W=1 bit, as per {{I-D.ietf-lpwan-ipv6-static-context-hc}}.
* **DTag**: Not used, so its size is 0 bit.
* **FCN**: The FCN field is encoded on N=1 bits, so WINDOW_SIZE = 1 tile
  (FCN=All-1 is reserved for SCHC).
* **RCS calculation algorithm**: CRC32 using 0xEDB88320 (i.e. the reverse
  representation of the polynomial used e.g. in the Ethernet standard
  [RFC3385]), as per {{I-D.ietf-lpwan-ipv6-static-context-hc}}.
* **MAX_ACK_REQUESTS**: 8

As only 1 tile is used, its size can change for each downlink, and will be
maximum available MTU.

_Note_: The Fpending bit included in LoRaWAN protocol SHOULD NOT be used for
SCHC-over-LoRaWAN protocol. It might be set by the Network Server for other
purposes in but not SCHC needs.

**Regular fragments**

~~~~

| FPort  | LoRaWAN payload                     |
+ ------ + ----------------------------------- +
| RuleID | W     | FCN = b'0 | Payload         |
+ ------ + ----- + --------- + --------------- +
| 8 bits | 1 bit | 1 bit     | X bytes         |

~~~~
{: #Fig-fragmentation-downlink-header-all0 title='All fragments but the last one. Header size 1 byte (8 bits), including LoraWAN FPort.'}


**Last fragment (All-1)**

~~~~

| FPort  | LoRaWAN payload                                 |
+ ------ + ----------------------------------------------- +
| RuleID | W     | FCN = b'1 | RCS     | Payload           |
+ ------ + ----- + --------- + ------- + ----------------- +
| 8 bits | 1 bit | 1 bit     | 32 bits | Last tile, if any |

~~~~
{: #Fig-fragmentation-downlink-header-all1 title='All-1 SCHC ACK detailed format for the last fragment.'}

**SCHC acknowledge**

~~~~

| FPort  | LoRaWAN payload                    |
+ ------ + ---------------------------------- +
| RuleID | W     | C = b'1 | Padding b'000000 |
+ ------ + ----- + ------- + ---------------- +
| 8 bits | 1 bit | 1 bit   | 6 bits           |

~~~~
{: #Fig-fragmentation-downlink-header-schc-ack title='SCHC ACK format, RCS is correct.'}


**Receiver-Abort**

~~~~

| FPort  | LoRaWAN payload                                |
+ ------ + ---------------------------------------------- +
| RuleID | W = b'1 | C = b'1 | b'111111 | 0xFF (all 1's)  |
+ ------ + ------- + ------- + -------- + --------------- +
| 8 bits | 1 bit   | 1 bits  | 6 bits   | 8 bits          |
                next L2 Word boundary ->| <-- L2 Word --> |


~~~~
{: #Fig-fragmentation-downlink-header-abort title='Receiver-Abort packet (following an all-1 packet with incorrect RCS).'}


Class A and Class B or Class C end-devices do not manage retransmissions and
timers in the same way.

#### Class A end-devices

Class A end-devices can only receive in an RX slot following the transmission of an
uplink.  Therefore there cannot be a concept of "retransmission timer" for an
SCHC gateway. The SCHC gateway cannot initiate communication to a Class A
end-device.

The device replies with an ACK message to every single fragment received from
the SCHC gateway (because the window size is 1). Following the reception of a
FCN=0 fragment (fragment that is not the last fragment of the packet or
ACK-request, but the end of a window), the device MUST transmit the SCHC ACK
fragment until it receives the fragment of the next window. The device SHALL
transmit up to MAX_ACK_REQUESTS ACK messages before aborting. The device
should transmit those ACK as soon as possible while taking into consideration
potential local radio regulation on duty-cycle, to progress the fragmentation
session as quickly as possible. The ACK bitmap is 1 bit long and is always 1.

Following the reception of a FCN=All-1 fragment (the last fragment of a
datagram) and if the RCS is correct, the device SHALL transmit the ACK with
the "RCS is correct" indicator bit set (C=1). This message might be lost
therefore the SCHC gateway MAY request a retransmission of this ACK in the next
downlink. The device SHALL keep this ACK message in memory until it receives
a downlink, on SCHC FPortsDown from the SCHC gateway different from an
ACK-request: it indicates that the SCHC gateway has received the ACK message.

Following the reception of a FCN=All-1 fragment (the last fragment of a
datagram), if all fragments have been received and the RCS is not correct,
the device SHALL transmit a Receiver-Abort fragment. The device SHALL keep
this Abort message in memory until it receives a downlink, on SCHC FPortsDown,
from the SCHC gateway different from an ACK-request indicating that the SCHC
gateway has received the Abort message.  The fragmentation receiver (device)
does not implement retransmission timer and inactivity timer.

The fragmentation sender (the SCHC gateway) implements an inactivity timer with
default duration of 12 hours. Once a fragmentation session is started, if the
SCHC gateway has not received any ACK or Receiver-Abort message 12 hours after
the last message from the device was received, the SCHC gateway MAY flush the
fragmentation context.  For devices with very low transmission rates
(example 1 packet a day in normal operation) , that duration may be extended,
but this is application specific.


#### Class B or Class C end-devices

Class B and Class C end-devices can receive in scheduled RX slots or in RX
slots following the transmission of an uplink. The device replies with an ACK
message to every single fragment received from the SCHC gateway (because the
window size is 1). Following the reception of a FCN=0 fragment (fragment that
is not the last fragment of the packet or ACK-request), the device MUST always
transmit the corresponding SCHC ACK message even if that fragment has already
been received.
The ACK bitmap is 1 bit long and is always 1. If the SCHC gateway receives this
ACK, it proceeds to send the next window fragment. If the retransmission timer
elapses and the SCHC gateway has not received the ACK of the current window it
retransmits the last fragment. The SCHC gateway tries retransmitting up to
MAX_ACK_REQUESTS times before aborting.

Following the reception of a FCN=All-1 fragment (the last fragment of a
datagram) and if the RCS is correct, the device SHALL transmit the ACK with the
RCS is correct” indicator bit set. If the SCHC gateway receives this ACK, the
current fragmentation session has succeeded and its context can be cleared.

If the retransmission timer elapses and the SCHC gateway has not received the
SCHC ACK it retransmits the last fragment with the payload (not an ACK-request
without payload). The SCHC gateway tries retransmitting up to MAX_ACK_REQUESTS
times before aborting.

The device SHALL keep the SCHC ACK message in memory until it receives a
downlink from the SCHC gateway different from the last (FCN>0 and different
DTag) fragment indicating that the SCHC gateway has received the ACK message.

Following the reception of a FCN=All-1 fragment (the last fragment of a
datagram), if all fragments have been received and if the RCS is NOT correct,
the device SHALL transmit a Receiver-Abort fragment.  The retransmission
timer is used by the SCHC gateway (the sender), the optimal value is very much
application specific but here are some recommended default values.
For Class B end-devices, this timer trigger is a function of the periodicity of the
Class B ping slots. The RECOMMENDED value is equal to 3 times the Class B ping
slot periodicity. For Class C end-devices which are nearly constantly receiving,
the RECOMMENDED value is 30 seconds. This means that the end-device shall try to
transmit the ACK within 30 seconds  of the reception of each fragment.  The
inactivity timer is implemented by the end-device to flush the context in-case it
receives nothing from the SCHC gateway over an extended period of time. The
RECOMMENDED value is 12 hours for both Class B and Class C end-devices.

# Security considerations

This document is only providing parameters that are expected to be better
suited for LoRaWAN networks for {{I-D.ietf-lpwan-ipv6-static-context-hc}}. As
such, this parameters does not contribute to any new security issues in
addition of those identified in {{I-D.ietf-lpwan-ipv6-static-context-hc}}.

# Acknowledgements
{:numbered="false"}

Thanks to all those listed in the Contributors section for the excellent text,
insightful discussions, reviews and suggestions.

# Contributors
{:numbered="false"}

Contributors ordered by family name.

- ins: V. Audebert
  name: Vincent AUDEBERT
  org: EDF R&D
  street: 7 bd Gaspard Monge
  city: 91120 PALAISEAU
  country: FRANCE
  email: vincent.audebert@edf.fr
- ins: M. Coracin
  name: Michael Coracin
  org: Semtech
  street: 14 Chemin des Clos
  city: Meylan
  country: France
  email: mcoracin@semtech.com
- ins: M. Le Gourrierec
  name: Marc Le Gourrierec
  org: SagemCom
  street: 250 Route de l'Empereur
  city: 92500 Rueil Malmaison
  country: FRANCE
  email: marc.legourrierec@sagemcom.com
- ins: N. Sornin
  name: Nicolas Sornin
  org: Semtech
  street: 14 Chemin des Clos
  city: Meylan
  country: France
  email: nsornin@semtech.com
- ins: A. Yegin
  name: Alper Yegin
  org: Actility
  street: .
  city: Paris, Paris
  country: France
  email: alper.yegin@actility.com

--- back

# Examples

## Uplink - Compression example - No fragmentation
{{Fig-example-uplink-no-fragmentation}} is representing an applicative payload
going through SCHC, no fragmentation required

~~~~

An applicative payload of 78 bytes is passed to SCHC compression layer
using rule 1, allowing to compress it to 40 bytes and 5 bits: 21 bits
residue + 38 bytes payload.


| RuleID | Compression residue |  Payload  | Padding=b'00000 |
+ ------ + ------------------- + --------- + --------------- +
|   1    |       21 bits       |  38 bytes |     5 bits      |


The current LoRaWAN MTU is 51 bytes, although 2 bytes FOpts are used by
LoRaWAN protocol: 49 bytes are available for SCHC payload; no need for
fragmentation. The payload will be transmitted through FPort = 1


| LoRaWAN Header | FPort  | LoRaWAN payload                                   |
+ -------------- + ------ + ------------------------------------------------- +
|                | RuleID | Compression residue |  Payload  | Padding=b'00000 |
+ -------------- + ------ + ------------------- + --------- + --------------- +
|       XXXX     |   1    |       21 bits       |  38 bytes |     5 bits      |

~~~~
{: #Fig-example-uplink-no-fragmentation title='Uplink example: compression without fragmentation'}

## Uplink - Compression and fragmentation example

{{Fig-example-uplink-fragmentation-long}} is representing  an applicative payload
going through SCHC, with fragmentation.

~~~~

An applicative payload of 478 bytes is passed to SCHC compression layer
using rule 1, allowing to compress it to 442 bytes and 5 bits: 21 bits
residue + 440 bytes payload.


| RuleID | Compression residue |  Payload  |
+ ------ + ------------------- + --------- +
|   1    |       21 bits       | 440 bytes |


The current LoRaWAN MTU is 11 bytes, although 2 bytes FOpts are used by
LoRaWAN protocol: 9 bytes are available for SCHC payload + FPort field.
SCHC header is 18 bits (including FPort) so 2 tiles are send in first
fragment.

| LoRaWAN Header | FOpts   | FPortUp | LoRaWAN payload                            |
+ -------------- + ------- + ------- + ------------------------------------------ +
|                |         | RuleID  | DTag  |   W   |  FCN   | 2 tiles | Padding |
+ -------------- + ------- + ------- + ----- + ----- + ------ + ------- + ------- +
|       XXXX     | 2 bytes | 1 byte  |   0   | b'00  |  126   | 6 bytes | 6 bits  |

Content of the two tiles is:
| RuleID | Compression residue |  Payload           |
+ ------ + ------------------- + ------------------ +
|   1    |       21 bits       |  2 bytes + 3 bits  |


Next transmission MTU is 242 bytes, no FOpts. 80 tiles are transmitted:

| LoRaWAN Header | FPortUp | LoRaWAN payload                              |
+ -------------- + --------+ -------------------------------------------- +
|                | RuleID  | DTag  |   W   |  FCN   | 80 tiles  | Padding |
+ -------------- + ------- + ----- + ----- + ------ + --------- + ------- +
|       XXXX     | 1 byte  |   0   | 0   0 |  124   | 240 bytes | 6 bits  |


Next transmission MTU is 242 bytes, no FOpts. All 66 remaining tiles are
transmitted, last tile is only 2 bytes. Padding is added for the remaining 3 bits.

| LoRaWAN Header | FPortUp | LoRaWAN payload                                      |
+ -------------- + ------- + ---------------------------------------------------- +
|                | RuleID  | DTag  |   W   |  FCN   |  RCS  | 66 tiles  | Padding |
+ -------------- + ------- + ----- + ----- + ------ + ----- + --------- + ------- +
|       XXXX     | 1 byte  |   0   | 0   0 |  127   | CRC32 | 198 bytes | 6 bits  |


All packets have been received by the SCHC gateway, computed RCS is
correct so the following ACK is send to the device:

| LoRaWAN Header | FPortUp | LoRaWAN payload             |
+ -------------- + ------- + --------------------------- +
|                | RuleID  | DTag  |   W   | C | Padding |
+ -------------- + ------- + ----- + ----- + - + ------- +
|       XXXX     | 1 byte  |   0   | 0   0 | 1 | 4 bits  |

~~~~
{: #Fig-example-uplink-fragmentation-long title='Uplink example: compression and fragmentation'}

## Downlink


~~~~

An applicative payload of 43 bytes is passed to SCHC compression layer using
rule 1, allowing to compress it to 30 bytes and 5 bits: 21 bits residue + 28 bytes
payload.


| RuleID | Compression residue |  Payload  |
+ ------ + ------------------- + --------- +
|   1    |       21 bits       |  28 bytes |


The current LoRaWAN MTU is 11 bytes, although 2 bytes FOpts are used by
LoRaWAN protocol: 9 bytes are available for SCHC payload + FPort field => it
has to be fragmented.


| LoRaWAN Header | FOpts   | FPortDown | LoRaWAN payload                     |
+ -------------- + ------- + ---   --- + ----------------------------------- +
|                |         | RuleID    |   W    |  FCN   | 1 tile  | Padding |
+ -------------- + ------- + --------- + ------ + ------ + ------- + ------- +
|       XXXX     | 2 bytes | 1 byte    |   0    |   0    | 8 bytes | 6 bits  |

Content of the tile is:
| RuleID | Compression residue |        Payload     |
+ ------ + ------------------- + ------------------ +
|   1    |       21 bits       |  4 bytes + 3 bits  |


The receiver answers with an SCHC ACK

| FPortDown | LoRaWAN payload           |
+ --------- + ------------------------- +
| RuleID    | W = 0 | C = b'1 | Padding |
+ --------- + ----- + ------- + ------- +
| 1 byte    | 1 bit | 1 bit   | 6 bits  |


The second downlink is send, no FOpts:


| LoRaWAN Header | FPortDown | LoRaWAN payload                             |
+ -------------- + --------- + ------------------------------------------- +
|                | RuleID    |   W    |  FCN   | 1 tile          | Padding |
+ -------------- + --------- + ------ + ------ + --------------- + ------- +
|       XXXX     | 1 byte    |   1    |   0    | 10 bytes        | 6 bits  |


The receiver answers with an SCHC ACK

| FPortDown | LoRaWAN payload           |
+ --------- + ------------------------- +
| RuleID    | W = 1 | C = b'1 | Padding |
+ --------- + ----- + ------- + ------- +
| 1 byte    | 1 bit | 1 bit   | 6 bits  |


The third downlink is send, no FOpts:


| LoRaWAN Header | FPortDown                | LoRaWAN payload           |
+ -------------- + ------------------------ + ------------------------- +
|                | RuleID |   W    |  FCN   | 1 tile          | Padding |
+ -------------- + ------ + ------ + ------ + --------------- + ------- +
|       XXXX     | 001100 |   0    |   0    | 10 bytes        | 6 bits  |


The receiver answers with an SCHC ACK


| FPortDown | LoRaWAN payload           |
+ --------- + ------------------------- +
| RuleID    | W = 0 | C = b'1 | Padding |
+ --------- + ----- + ------- + ------- +
| 1 byte    | 1 bit | 1 bit   | 6 bits  |


The last downlink is send, no FOpts:


| LoRaWAN Header | FPortDown | LoRaWAN payload                              |
+ -------------- + --------- + -------------------------------------------- +
|                | RuleID    |   W    |  FCN   |  1 tile          | Padding |
+ -------------- + --------- + ------ + ------ + ---------------- + ------- +
|       XXXX     | 1 byte    |   1    |   1    | 2 bytes + 5 bits | 1 bit   |


The receiver answers with an SCHC ACK

| FPortDown | LoRaWAN payload |
+ --------- + --------------- +
| RuleID    | W = 1 |  C = 1  |
+ --------- + ----- + ------- +
| 1 byte    | 1 bit | 1 bit   |

~~~~
{: #Fig-example-downlink-fragmentation title='Downlink example: compression and fragmentation'}

# Note
