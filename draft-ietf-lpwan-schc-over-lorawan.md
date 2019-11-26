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
  street: 1137A Avenue des Champs Blancs
  city: 35510 Cesson-Sevigne Cedex
  country: France
  email: ivaylo@ackl.io
  role: editor
normative:
  RFC2119:
  RFC8174:
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
  I-D.ietf-lpwan-ipv6-static-context-hc:
  lora-alliance-remote-multicast-set:
    title: LoRaWAN Remote Multicast Setup Specification Version 1.0.0
    author:
      name: LoRa Alliance
    date: 09.2018
    target: https://lora-alliance.org/sites/default/files/2018-09/remote_multicast_setup_v1.0.0.pdf

--- abstract

The Static Context Header Compression (SCHC) specification describes generic
header compression and fragmentation techniques for LPWAN (Low Power Wide Area
Networks) technologies. SCHC is a generic mechanism designed for great
flexibility so that it can be adapted for any of the LPWAN technologies.

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
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in BCP 14 {{RFC2119}} {{RFC8174}}
when, and only when, they appear in all capitals, as shown here.

This section defines the terminology and acronyms used in this document. For
all other definitions, please look up the SCHC specification
{{I-D.ietf-lpwan-ipv6-static-context-hc}}.

  o  DevEUI: an IEEE EUI-64 identifier used to identify the end-device during the
     procedure while joining the network (Join Procedure)

  o  DevAddr: a 32-bit non-unique identifier assigned to an end-device statically or
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
networks, some static contexts may be stored on the Device (Dev). The context
MUST be stored in both ends, and it can either be learned by a provisioning
protocol or by out-of-band means or it can be pre-provisioned, etc. The way the
context is learned on both sides is outside the scope of this document.


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
size and fragmented (SCHC F/R).  The resulting information is sent on a layer two
(L2) frame to an LPWAN Radio Gateway (RGW) which forwards the frame to a Network
Gateway (NGW). The NGW sends the data to a SCHC F/R for defragmentation, if
required, then C/D for decompression which shares the same rules with the
device. The SCHC F/R and C/D can be located on the Network Gateway (NGW) or in
another place as long as a tunnel is established between the NGW and the SCHC
F/R, then SCHC F/R and SCHC C/D. The SCHC C/D in both sides MUST share the same
set of rules. After decompression, the packet can be sent on the Internet to
one or several LPWAN Application Servers (App).

The SCHC F/R and SCHC C/D process is bidirectional, so the same principles can
be applied in the other direction.

In a LoRaWAN network, the RG is called a Gateway, the NGW is Network Server,
and the SCHC C/D is an Application Server. It can be provided by the Network
Server or any third party software. {{Fig-archi}} can be mapped in LoRaWAN
terminology to:

~~~~
       Dev                                                  App
+--------------+                                    +----+ +----+ +----+
|App1 App2 App3|                                    |App1| |App2| |App3|
|              |                                    |    | |    | |    |
|      UDP     |                                    |UDP | |UDP | |UDP |
|     IPv6     |                                    |IPv6| |IPv6| |IPv6|
|              |                                    |    | |    | |    |
|SCHC C/D & F/R|                                    |    | |    | |    |
+-------+------+                                    +----+ +----+ +----+
        |  +-------+     +-------+    +-----------+    .      .      .
        +~ |Gateway| === |Network| == |Application|..... Internet ....
           +-------+     |server |    |server     |
                         +-------+    | F/R - C/D |
                                      +-----------+
~~~~
{: #Fig-archi-lorawan title='SCHC Architecture mapped to LoRaWAN'}


#LoRaWAN Architecture

An overview of LoRaWAN {{lora-alliance-spec}} protocol and architecture is
described in {{RFC8376}}. The mapping between the LPWAN
architecture entities as described in {{I-D.ietf-lpwan-ipv6-static-context-hc}}
and the ones in {{lora-alliance-spec}} is as follows:

   o Devices (Dev) are the end-devices or hosts (e.g. sensors,
   actuators, etc.).  There can be a very high density of devices per
   radio gateway (LoRaWAN gateway). This entity maps to the LoRaWAN End-Device.

   o The Radio Gateway (RGW), which is the endpoint of the constrained
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
end-devices implement the Class A, some end-devices may implement Class B or
Class C. Class B and Class C are mutually exclusive.

* Class A: The Class A is the simplest class of end-devices. The end-device is
  allowed to transmit at any time, randomly selecting a communication channel.
  The network may reply with a downlink in one of the 2 receive windows
  immediately following the uplinks. Therefore, the network cannot initiate a
  downlink, it has to wait for the next uplink from the end-device to get a
  downlink opportunity. The Class A is the lowest power end-device class.
* Class B: Class B end-devices implement all the functionalities of Class A
  end-devices, but also schedule periodic listen windows. Therefore, opposed to the
  Class A end-devices, Class B end-devices can receive downlinks that are initiated by the
  network and not following an uplink. There is a trade-off between the
  periodicity of those scheduled Class B listen windows and the power
  consumption of the end-device. The lower the downlink latency, the higher the
  power consumption.
* Class C: Class C end-devices implement all the functionalities of Class A
  end-devices, but keep their receiver open whenever they are not transmitting.
  Class C end-devices can receive downlinks at any time at the expense of a higher
  power consumption. Battery-powered end-devices can only operate in Class C for a
  limited amount of time (for example for a firmware upgrade over-the-air).
  Most of the Class C end-devices are grid powered (for example Smart Plugs).

## End-Device addressing

LoRaWAN end-devices use a 32-bit network address (devAddr) to communicate with
the network over-the-air. However, that address might be reused several times
on the same network at the same time for different end-devices. End-devices using the
same devAddr are distinguished by the Network Server based on the cryptographic
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

* Confirmed messages:
  The sender asks the receiver to acknowledge the message.
* Unconfirmed messages:
  The sender does not ask the receiver to acknowledge the message.

As SCHC defines its own acknowledgment mechanisms, SCHC does not require to use
confirmed messages.

## LoRaWAN MAC Frames

* JoinRequest:
  This message is used by an end-device to join a network. It contains the end-device's
  unique identifier devEUI and a random nonce that will be used for session key
  derivation.
* JoinAccept:
  To on-board an end-device, the Network Server responds to the JoinRequest end-device's
  message with a JoinAccept message. That message is encrypted with the
  end-device's AppKey and contains (amongst other fields) the major network's
  settings and a network random nonce used to derive the session keys.
* Data

## Unicast and multicast technology

LoRaWAN technology supports unicast downlinks, but also multicast: a packet
send over LoRaWAN radio link can be received by several devices.  It is
useful to address many end-devices with same content, either a large binary
file (firmware upgrade), or same command (e.g: lighting control).
As IPv6 is also a multicast technology this feature MAY be used to address a
group of devices.

_Note 1_: IPv6 multicast addresses must be defined as per [RFC4291].  LoRaWAN
multicast group definition in a network server and the relation between those
groups and IPv6 groupID are out of scope of this document.

_Note 2_: LoRa Alliance defined {{lora-alliance-remote-multicast-set}} as
RECOMMENDED way to setup multicast groups on devices and create a synchronized
reception window.

# SCHC-over-LoRaWAN

## LoRaWAN FPort {#lorawan-schc-payload}

The LoRaWAN MAC layer features a frame port field in all frames. This field
(FPort) is 8 bits long and the values from 1 to 223 can be used. It allows
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
server, is called uplink fragmentation session. It uses an FPort for data uplink
and its associated SCHC control downlinks, named FPortUp in this document. The
other way, a fragmentation session with application payload transferred from
server to device, is called downlink fragmentation session. It uses another
FPort for data downlink and its associated SCHC control uplinks, named FPortDown
in this document.

FPorts can use arbitrary values inside the allowed FPort range and MUST be
shared by the end-device, the Network Server and SCHC gateway prior to the
communication. The uplink and downlink fragmentation FPorts MUST be different.

## Rule ID management  {#rule-id-management}

RuleID MUST be 8 bits, encoded in the LoRaWAN FPort as described in
{{lorawan-schc-payload}}. LoRaWAN supports up to 223 application FPorts in
the range \[1;223\] as defined in section 4.3.2 of {{lora-alliance-spec}}, it implies
that RuleID MSB SHOULD be inside this range. An application MAY reserve some
FPort values for other needs as long as they don't conflict with FPorts used
for SCHC C/D and SCHC F/R.

In order to improve interoperability RECOMMENDED fragmentation RuleID values are:

* RuleID = 20 (8-bit) for uplink fragmentation, named FPortUp
* RuleID = 21 (8-bit) for downlink fragmentation, named FPortDown
* RuleID = 22 (8-bit) for which SCHC compression was not possible (no matching
rule was found)

The remaining RuleIDs are available for compression. RuleIDs are shared between
uplink and downlink sessions.  A RuleID different from FPortUp or FPortDown
means that the fragmentation is not used, thus the packet SHOULD be sent to C/D
layer.

The only uplink messages using the FPortDown port are the fragmentation SCHC
control messages of a downlink fragmentation session (ex ACKs). Similarly, the
only downlink messages using the FPortUp port are the fragmentation SCHC
control messages of an uplink fragmentation session.

An application can have multiple fragmentation sessions between a device and one
or several SCHC gateways.  A set of FPort values is REQUIRED for each SCHC gateway
instance the device is required to communicate with.

The mechanism for sharing those RuleID values is outside the scope of this document.

## IID computation

As LoRaWAN network uses unique EUI-64 per end-device, the Interface IDentifier is
the LoRaWAN DevEUI.
It is compliant with [RFC4291] and IID starting with binary 000 must enforce
the 64-bit rule.

TODO: Derive IID from DevEUI with privacy constraints ? Ask working group ?

## Padding

All padding bits MUST be 0.

## Compression {#Comp}

SCHC C/D MUST concatenate FPort and LoRaWAN payload to retrieve the SCHC packet
as per {{lorawan-schc-payload}}.

RuleIDs matching FPortUp and FPortDown are reserved for SCHC Fragmentation.

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

### DTag

A LoRaWAN device cannot interleave several fragmented SCHC datagrams on the same
FPort.  This field is not used and its size is 0.

Note: The device can still have several parallel fragmentation sessions with one
or more SCHC gateway(s) thanks to distinct sets of FPorts, cf {{rule-id-management}}

### Uplink fragmentation: From device to SCHC gateway

In that case the device is the fragmentation transmitter, and the SCHC gateway
the fragmentation receiver. A single fragmentation rule is defined.
SCHC F/R MUST concatenate FPort and LoRaWAN payload to retrieve the SCHC
fragment as per {{lorawan-schc-payload}}.

* SCHC header size is two bytes (the FPort byte + 1 additional byte).
* RuleID: 8 bits stored in LoRaWAN FPort.
* SCHC fragmentation reliability mode: `ACK-on-Error`
* DTag: Size is 0 bit, not used
* FCN: The FCN field is encoded on N = 6 bits, so WINDOW_SIZE = 63 tiles
  are allowed in a window
* Window index: encoded on W = 2 bits. So 4 windows are available.
* RCS calculation algorithm: CRC32 using 0xEDB88320 (i.e. the reverse
  representation of the polynomial used e.g. in the Ethernet standard
  [RFC3385]) as suggested in {{I-D.ietf-lpwan-ipv6-static-context-hc}}.
* MAX_ACK_REQUESTS: 8
* Tile: size is 10 bytes
* Retransmission and inactivity timers:
  LoRaWAN end-devices do not implement a "retransmission timer". At the end of
  a window or a fragmentation session, corresponding ACK(s) is (are)
  transmitted by the network gateway (LoRaWAN application server) in the RX1 or
  RX2 receive slot of end-device.
  If this ACK is not received by the end-device at the end of its RX windows,
  it sends an all-0 (or an all-1) fragment with no payload to request an SCHC
  ACK retransmission. The periodicity between retransmission of the all-0/all-1
  fragments is device/application specific and MAY be different for each device
  (not specified). The SCHC gateway implements an "inactivity timer".
  The default RECOMMENDED duration of this timer is 12 hours. This value is
  mainly driven by application requirements and MAY be changed by the
  application.
* Last tile: The last tile can be carried in the All-1 fragment.

With this set of parameters, the SCHC fragment header is 16 bits,
including FPort; payload overhead will be 8 bits as FPort is already a part of
LoRaWAN payload. MTU is: _4 windows * 63 tiles * 10 bytes per tile = 2520 bytes_

#### Regular fragments

~~~~

| FPort  |  LoRaWAN payload          |
+ ------ + ------------------------- +
| RuleID |   W    | FCN    | Payload |
+ ------ + ------ + ------ + ------- +
| 8 bits | 2 bits | 6 bits |         |

~~~~
{: #Fig-fragmentation-header-long-all0 title='All fragments except the last one. SCHC header size is 16 bits, including LoRaWAN FPort.'}


#### Last fragment (All-1)

~~~~

| FPort  | LoRaWAN payload                                  |
+ ------ + ------------------------------------------------ +
| RuleID |   W    | FCN=All-1 |  RCS    | Payload           |
+ ------ + ------ + --------- + ------- + ----------------- +
| 8 bits | 2 bits | 6 bits    | 32 bits | Last tile, if any |

~~~~
{: #Fig-fragmentation-header-all1 title='All-1 fragment detailed format for the last fragment.'}

#### SCHC ACK

~~~~

| FPort  | LoRaWAN payload                           |
+ ------ + ----------------------------------------- +
| RuleID |   W   | C     | Encoded bitmap (if C = 0) |
+ ------ + ----- + ----- + ------------------------- +
| 8 bits | 2 bit | 1 bit | 0 to 127 bits             |

~~~~
{: #Fig-fragmentation-header-long-schc-ack title='SCHC formats, failed RCS check.'}


#### Receiver-Abort

~~~~

| FPort  | LoRaWAN payload                              |
+ ------ + -------------------------------------------- +
| RuleID | W = b'11 | C = 1 | b'11111 | 0xFF (all 1's)  |
+ ------ + -------- + ------+-------- + ----------------+
| 8 bits |  2 bits  | 1 bit | 5 bits  | 8 bits          |
              next L2 Word boundary ->| <-- L2 Word --> |

~~~~
{: #Fig-fragmentation-receiver-abort title='Receiver-Abort format.'}


#### SCHC acknowledge request

~~~~

| FPort  | LoRaWAN payload          |
+------- +------------------------- +
| RuleID | W      | FCN = b'000000  |
+ ------ + ------ + --------------- +
| 8 bits | 2 bits | 6 bits          |


~~~~
{: #Fig-fragmentation-schc-ack-req title='SCHC ACK REQ format.'}



### Downlink fragmentation: From SCHC gateway to a device

In that case the device is the fragmentation receiver, and the SCHC gateway the
fragmentation transmitter. The following fields are common to all devices.
SCHC F/R MUST concatenate FPort and LoRaWAN payload to retrieve the SCHC
fragment as described in {{lorawan-schc-payload}}.

* SCHC fragmentation reliability mode:
  * Unicast downlinks: ACK-Always.
  * Multicast downlinks: No-ACK, reliability has to be ensured by the upper
    layer. This feature is OPTIONAL and may not be implemented by SCHC gateway.
* RuleID: 8 bits stored in LoRaWAN FPort.
* Window index (unicast only): encoded on W=1 bit, as per {{I-D.ietf-lpwan-ipv6-static-context-hc}}.
* DTag: Size is 0 bit, not used
* FCN: The FCN field is encoded on N=1 bit, so WINDOW_SIZE = 1 tile
  (FCN=All-1 is reserved for SCHC).
* RCS calculation algorithm: CRC32 using 0xEDB88320 (i.e. the reverse
  representation of the polynomial used e.g. in the Ethernet standard
  [RFC3385]), as per {{I-D.ietf-lpwan-ipv6-static-context-hc}}.
* MAX_ACK_REQUESTS: 8

As only 1 tile is used, its size can change for each downlink, and will be
maximum available MTU.

_Note_: The Fpending bit included in LoRaWAN protocol SHOULD NOT be used for
SCHC-over-LoRaWAN protocol. It might be set by the Network Server for other
purposes but not SCHC needs.

#### Regular fragments

~~~~

| FPort  | LoRaWAN payload                     |
+ ------ + ----------------------------------- +
| RuleID | W     | FCN = b'0 | Payload         |
+ ------ + ----- + --------- + --------------- +
| 8 bits | 1 bit | 1 bit     | X bytes         |

~~~~
{: #Fig-fragmentation-downlink-header-all0 title='All fragments but the last one. Header size 10 bits, including LoraWAN FPort.'}


#### Last fragment (All-1)

~~~~

| FPort  | LoRaWAN payload                                 |
+ ------ + ----------------------------------------------- +
| RuleID | W     | FCN = b'1 | RCS     | Payload           |
+ ------ + ----- + --------- + ------- + ----------------- +
| 8 bits | 1 bit | 1 bit     | 32 bits | Last tile, if any |

~~~~
{: #Fig-fragmentation-downlink-header-all1 title='All-1 SCHC ACK detailed format for the last fragment.'}

#### SCHC acknowledge

~~~~

| FPort  | LoRaWAN payload                    |
+ ------ + ---------------------------------- +
| RuleID | W     | C = b'1 | Padding b'000000 |
+ ------ + ----- + ------- + ---------------- +
| 8 bits | 1 bit | 1 bit   | 6 bits           |

~~~~
{: #Fig-fragmentation-downlink-header-schc-ack title='SCHC ACK format, RCS is correct.'}


#### Receiver-Abort

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

Following the reception of an FCN=All-1 fragment (the last fragment of a
datagram) and if the RCS is correct, the device SHALL transmit the ACK with
the "RCS is correct" indicator bit set (C=1). This message might be lost
therefore the SCHC gateway MAY request a retransmission of this ACK in the next
downlink. The device SHALL keep this ACK message in memory until it receives
a downlink, on SCHC FPortDown from the SCHC gateway different from an
ACK-request: it indicates that the SCHC gateway has received the ACK message.

Following the reception of a FCN=All-1 fragment (the last fragment of a
datagram), if all fragments have been received and the RCS is not correct,
the device SHALL transmit a Receiver-Abort fragment. The device SHALL keep
this Abort message in memory until it receives a downlink, on SCHC FPortDown,
from the SCHC gateway different from an ACK-request indicating that the SCHC
gateway has received the Abort message.  The fragmentation receiver (device)
does not implement retransmission timer and inactivity timer.

The fragmentation sender (the SCHC gateway) implements an inactivity timer with
a default duration of 12 hours. Once a fragmentation session is started, if the
SCHC gateway has not received any ACK or Receiver-Abort message 12 hours after
the last message from the device was received, the SCHC gateway MAY flush the
fragmentation context.  For devices with very low transmission rates
(example 1 packet a day in normal operation) , that duration may be extended,
but this is application specific.


#### Class B or Class C end-devices

Class B and Class C end-devices can receive in scheduled RX slots or in RX
slots following the transmission of an uplink. The device replies with an ACK
message to every single fragment received from the SCHC gateway (because the
window size is 1). Following the reception of an FCN=0 fragment (fragment that
is not the last fragment of the packet or ACK-request), the device MUST always
transmit the corresponding SCHC ACK message even if that fragment has already
been received.
The ACK bitmap is 1 bit long and is always 1. If the SCHC gateway receives this
ACK, it proceeds to send the next window fragment. If the retransmission timer
elapses and the SCHC gateway has not received the ACK of the current window it
retransmits the last fragment. The SCHC gateway tries retransmitting up to
MAX_ACK_REQUESTS times before aborting.

Following the reception of an FCN=All-1 fragment (the last fragment of a
datagram) and if the RCS is correct, the device SHALL transmit the ACK with the
"RCS is correct" indicator bit set. If the SCHC gateway receives this ACK, the
current fragmentation session has succeeded and its context can be cleared.

If the retransmission timer elapses and the SCHC gateway has not received the
SCHC ACK it retransmits the last fragment with the payload (not an ACK-request
without payload). The SCHC gateway tries retransmitting up to MAX_ACK_REQUESTS
times before aborting.

Following the reception of an FCN=All-1 fragment (the last fragment of a
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
such, this document does not contribute to any new security issues in
addition to those identified in {{I-D.ietf-lpwan-ipv6-static-context-hc}}.

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
- ins: J. Catalano
  name: Julien Catalano
  org: Kerlink
  street: 1 rue Jacqueline Auriol
  city: 35235 Thorigné-Fouillard
  country: France
  email: j.catalano@kerlink.fr
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
using rule 1, allowing to compress it to 40 bytes and 5 bits: 1 byte
ruleID, 21 bits residue + 37 bytes payload.


| RuleID | Compression residue |  Payload  | Padding=b'000 |
+ ------ + ------------------- + --------- + ------------- +
|   1    |       21 bits       |  38 bytes |    3 bits     |


The current LoRaWAN MTU is 51 bytes, although 2 bytes FOpts are used by
LoRaWAN protocol: 49 bytes are available for SCHC payload; no need for
fragmentation. The payload will be transmitted through FPort = 1


| LoRaWAN Header            | LoRaWAN payload (40 bytes)              |
+ ------------------------- + --------------------------------------- +
|      |  FOpts  | RuleID=1 | Compression | Payload   | Padding=b'000 |
|      |         |          | residue     |           |               |
+ ---- + ------- + -------- + ----------- + --------- + ------------- +
| XXXX | 2 bytes | 1 byte   | 21 bits     |  37 bytes |    3 bits     |

~~~~
{: #Fig-example-uplink-no-fragmentation title='Uplink example: compression without fragmentation'}

## Uplink - Compression and fragmentation example

{{Fig-example-uplink-fragmentation-long}} is representing  an applicative payload
going through SCHC, with fragmentation.

~~~~

An applicative payload of 478 bytes is passed to SCHC compression layer
using rule 1, allowing to compress it to 282 bytes and 5 bits: 1 byte
ruleID, 21 bits residue + 279 bytes payload.


| RuleID | Compression residue |  Payload  |
+ ------ + ------------------- + --------- +
|   1    |       21 bits       | 279 bytes |


The current LoRaWAN MTU is 11 bytes, 0 bytes FOpts are used by LoRaWAN
protocol: 11 bytes are available for SCHC payload + 1 byte FPort field.
SCHC header is 2 bytes (including FPort) so 1 tile is sent in first
fragment.

| LoRaWAN Header             | LoRaWAN payload (11 bytes) |
+ -------------------------- + -------------------------- +
|                | RuleID=20 |   W   |  FCN   |  1 tile   |
+ -------------- + --------- + ----- + ------ + --------- +
|       XXXX     | 1 byte    | 0   0 |   62   | 10 bytes  |


Content of the tile is:
| RuleID | Compression residue |  Payload          |
+ ------ + ------------------- + ----------------- +
|   1    |       21 bits       |  6 byte + 3 bits  |


Next transmission MTU is 11 bytes, although 2 bytes FOpts are used by
LoRaWAN protocol: 9 bytes are available for SCHC payload + 1 byte FPort
field, a tile does not fit inside so LoRaWAN stack will send only FOpts.

Next transmission MTU is 242 bytes, 4 bytes FOpts. 23 tiles are transmitted:

| LoRaWAN Header                        | LoRaWAN payload (231 bytes) |
+ --------------------------------------+ --------------------------- +
|                |  FOpts  | RuleID=20  |   W   |  FCN  |  23 tiles   |
+ -------------- + ------- + ---------- + ----- + ----- + ----------- +
|       XXXX     | 4 bytes |  1 byte    | 0   0 |   61  | 230 bytes   |


Next transmission MTU is 242 bytes, no FOpts. All 5 remaining tiles are
transmitted, the last tile is only 2 bytes + 5 bits. Padding is added for
the remaining 3 bits.

| LoRaWAN Header    | LoRaWAN payload (44 bytes)                      |
+ ---- + -----------+ ----------------------------------------------- +
|      | RuleID=20  |   W   |  FCN  |      5 tiles      | Padding=b'000 |
+ ---- + ---------- + ----- + ----- + ----------------- + ------------- +
| XXXX | 1 byte     | 0   0 |   38  | 42 bytes + 5 bits |    3 bits     |


All packets have been received by the SCHC gateway, computed RCS is
correct so the following ACK is sent to the device:

| LoRaWAN Header             | LoRaWAN payload     |
+ -------------- + --------- + ------------------- +
|                | RuleID=20 |   W   | C | Padding |
+ -------------- + --------- + ----- + - + ------- +
|       XXXX     | 1 byte    | 0   0 | 1 | 5 bits  |

~~~~
{: #Fig-example-uplink-fragmentation-long title='Uplink example: compression and fragmentation'}

## Downlink


~~~~

An applicative payload of 443 bytes is passed to SCHC compression layer
using rule 1, allowing to compress it to 130 bytes and 5 bits: 1 byte
ruleId, 21 bits residue + 127 bytes payload.


| RuleID | Compression residue |  Payload  |
+ ------ + ------------------- + --------- +
|   1    |       21 bits       | 127 bytes |


The current LoRaWAN MTU is 51 bytes, no FOpts are used by LoRaWAN
protocol: 48 bytes are available for SCHC payload + FPort field => it
has to be fragmented.


| LoRaWAN Header    | LoRaWAN payload (51 bytes)                    |
+ ---- + ---------- + --------------------------------------------- +
|      | RuleID=21  |  W  | FCN |     1 tile     | Padding=b'000000 |
+ ---- + ---------- + --- + --- + -------------- + ---------------- +
| XXXX | 1 byte     |  0  |  0  |    50 bytes    |      6 bits      |

Content of the tile is:
| RuleID | Compression residue |        Payload     |
+ ------ + ------------------- + ------------------ +
|   1    |       21 bits       | 46 bytes + 3 bits  |


The receiver answers with an SCHC ACK

| FPortDown | LoRaWAN payload                    |
+ --------- + ---------------------------------- +
| RuleID    | W = 0 | C = b'1 | Padding=b'000000 |
+ --------- + ----- + ------- + ---------------- +
| 1 byte    | 1 bit | 1 bit   |     6 bits       |


The second downlink is sent, two FOpts:


| LoRaWAN Header              | LoRaWAN payload (49 bytes)            |
+ --------------------------- + ------------------ + ---------------- +
|      |  FOpts  | RuleID=21  | W | FCN | 1 tile   | Padding=b'000000 |
+ ---- + ------- + ---------- + - + --- + -------- + ---------------- +
| XXXX | 2 bytes | 1 byte     | 1 |  0  | 48 bytes |      6 bits      |


The receiver answers with an SCHC ACK

| FPortDown | LoRaWAN payload                    |
+ --------- + ---------------------------------- +
| RuleID    | W = 1 | C = b'1 | Padding=b'000000 |
+ --------- + ----- + ------- + ---------------- +
| 1 byte    | 1 bit | 1 bit   |     6 bits       |


The last downlink is sent, no FOpts:


| LoRaWAN Header    | LoRaWAN payload (33 bytes)                      |
+ ---- + ---------- + ----------------------------------------------- +
|      | RuleID=21  |  W  | FCN |    1 tile             | Padding=b'0 |
+ ---- + ---------- + --- + --- + --------------------- + ----------- +
| XXXX | 1 byte     |  0  |  1  |   32 bytes + 5 bits   | 1 bit       |


The receiver answers with an SCHC ACK


| FPortDown | LoRaWAN payload                    |
+ --------- + ---------------------------------- +
| RuleID    | W = 0 | C = b'1 | Padding=b'000000 |
+ --------- + ----- + ------- + ---------------- +
| 1 byte    | 1 bit | 1 bit   |      6 bits      |

~~~~
{: #Fig-example-downlink-fragmentation title='Downlink example: compression and fragmentation'}

# Note
