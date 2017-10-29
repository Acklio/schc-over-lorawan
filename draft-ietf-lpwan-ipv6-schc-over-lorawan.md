---
stand_alone: true
ipr: trust200902
docname: draft-ietf-lpwan-ipv6-schc-over-lorawan
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
  I-D.toutain-lpwan-ipv6-static-context-hc:

--- abstract

The Static Context Header Compression (SCHC) specification describes generic
header compression and fragmentation techniques for LPWAN (Low Power Wide Area
Networks) technologies. SCHC is designed with great flexibility in mind, so
that it can be adapted for any of the LPWAN technologies.

This document describes the optimal parameters and modes of operation when SCHC
is used over LoRaWAN networks.

--- middle

# Introduction {#Introduction}

The Static Context Header Compression (SCHC) specification
{{I-D.toutain-lpwan-ipv6-static-context-hc}} describes
generic header compression and fragmentation techniques that can be used on all
LPWAN (Low Power Wide Area Networks) technologies defined in
{{I-D.ietf-lpwan-overview}}. Even though those technologies share a great
number of common features like start-oriented topologies, network architecture,
devices with mostly quite predictable communications, etc; they do have some
slight differences in respect of payload sizes, reactiveness, etc.

SCHC gives a generic framework that enables those devices to communicate with
other Internet networks. However, for optimal performance, some parameters and
modes of operation need to be set appropriately for each of the LPWAN
technologies.

This document describes the optimal parameters and modes of operation when SCHC
is used over LoRaWAN networks.

# Terminology

This section defines the terminology and acronyms used in this document.

  o  Context: A set of rules used to compress/decompress headers.

  o  Dev: Device. A Node connected to an LPWAN. A Dev may implement SCHC.

  o  DI: Direction Indicator is a differentiator for matching that allows to
     have different behavior based on the direction of the communication -
     uplink or downlink.

  o  Bi: Bidirectional, it can be used in both senses.

  o  App: LPWAN Application. An application sending/receiving IPv6 packets
     to/from the Device.

  o  APP-IID: Application Interface Identifier. Second part of the IPv6
     address to identify the application interface.

  o  Dev-IID: Device Interface Identifier. Second part of the IPv6 address to
     identify the device interface

  o  CDA: Compression/Decompression Action. An action that is performed for
     both functionalities to compress a header field or to recover its original
     value in the decompression phase.

  o  DTag: Datagram Tag is a fragmentation header field that is set to
      the same value for all fragments carrying the same IPv6 datagram.

  o  SCHC C/D: Static Context Header Compression Compressor/ Decompressor. A
     process in the network to achieve compression/decompressing headers. SCHC
     C/D uses SCHC rules to perform compression and decompression.

  o  Dw: Down Link direction for compression, from SCHC C/D to Dev

  o  FCN: Fragment Compressed Number is a fragmentation header field that
     carries an efficient representation of a larger-sized fragment number.

  o  FID: Field Identifier is an index to describe the header fields in the
     Rule.

  o  FL: Field Length is a value to identify if the field is fixed or variable
     length.

  o  FP: Field Position is a value that is used to identify each
     instance a field apears in the header.

  o  IID: Interface Identifier. See the IPv6 addressing architecture
     {{RFC7136}}.

  o  MIC: Message Integrity Check. A fragmentation header field
     computed over an IPv6 packet before fragmentation, used for error
     detection after IPv6 packet reassembly.

  o  MO: Matching Operator. An operator used to match a value contained in a
     header field with a value contained in a Rule.

  o  Rule: A set of header field values.

  o  Rule ID: An identifier for a rule, SCHC C/D, and Dev share the same Rule
     ID for a specific flow. A set of Rule IDs are used to support
     fragmentation functionality.

  o  TV: Target value. A value contained in the Rule that will be matched with
     the value of a header field.

  o  Up: Up Link direction for compression, from Dev to SCHC C/D.

  o  W: Window bit. A fragmentation header field used in Window mode (see
     section 9 of {{I-D.toutain-lpwan-ipv6-static-context-hc}}), which carries
     the same value for all fragments of a window.

# Static Context Header Compression

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

## SCHC Rules

TBD

## Rule ID

Rule ID size and placement discussion (8 bits in fport).
TBD

## IID computation

TBD

# Fragmentation {#Frag}

TBD

## Overview

TBD

## Reliability options

TBD

## Supporting multiple window sizes

TBD

## Aborting fragmented IPv6 datagram transmissions

TBD

## Downlink fragment transmission

TBD



# SCHC Compression for IPv6 and UDP headers

This section lists the different IPv6 and UDP header fields and how they can be
compressed.

## IPv6 version field

This field always holds the same value, therefore the TV is 6, the MO is
"equal" and the "CDA "not-sent"".

## IPv6 Traffic class field

If the DiffServ field identified by the rest of the rule do not vary and is
known by both sides, the TV should contain this well-known value, the MO should
be "equal" and the CDA must be "not-sent.

If the DiffServ field identified by the rest of the rule varies over time or is
not known by both sides, then there are two possibilities depending on the
variability of the value, the first one is to do not compressed the field and
sends the original value, or the second where the values can be computed by
sending only the LSB bits:

* TV is not set to any value, MO is set to "ignore" and CDA is set to
  "value-sent"

* TV contains a stable value, MO is MSB(X) and CDA is set to LSB

## Flow label field

If the Flow Label field identified by the rest of the rule does not vary and is
known by both sides, the TV should contain this well-known value, the MO should
be "equal" and the CDA should be "not-sent".

If the Flow Label field identified by the rest of the rule varies during time
or is not known by both sides, there are two possibilities depending on the
variability of the value, the first one is without compression and then the
value is sent and the second where only part of the value is sent and the
decompressor needs to compute the original value:

* TV is not set, MO is set to "ignore" and CDA is set to "value-sent"

* TV contains a stable value, MO is MSB(X) and CDA is set to LSB

## Payload Length field

If the LPWAN technology does not add padding, this field can be elided for the
transmission on the LPWAN network. The SCHC C/D recomputes the original payload
length value. The TV is not set, the MO is set to "ignore" and the CDA is
"compute-IPv6-length".

If the payload length needs to be sent and does not need to be coded in 16
bits, the TV can be set to 0x0000, the MO set to "MSB (16-s)" and the CDA to
"LSB". The 's' parameter depends on the expected maximum packet length.

On other cases, the payload length field must be sent and the CDA is replaced
by "value-sent".

## Next Header field

If the Next Header field identified by the rest of the rule does not vary and
is known by both sides, the TV should contain this Next Header value, the MO
should be "equal" and the CDA should be "not-sent".

If the Next header field identified by the rest of the rule varies during time
or is not known by both sides, then TV is not set, MO is set to "ignore" and
CDA is set to "value-sent". A matching-list may also be used.

## Hop Limit field

The End System is generally a device and does not forward packets, therefore
the Hop Limit value is constant. So the TV is set with a default value, the MO
is set to "equal" and the CDA is set to "not-sent".

Otherwise the value is sent on the LPWAN: TV is not set, MO is set to ignore
and CDA is set to "value-sent".

Note that the field behavior differs in upstream and downstream. In upstream,
since there is no IP forwarding between the Dev and the SCHC C/D, the value is
relatively constant. On the other hand, the downstream value depends of
Internet routing and may change more frequently. One solution could be to use
the Direction Indicator (DI) to distinguish both directions to elide the field
in the upstream direction and send the value in the downstream direction.

## IPv6 addresses fields

As in 6LoWPAN {{RFC4944}}, IPv6 addresses are split into two 64-bit long
fields; one for the prefix and one for the Interface Identifier (IID). These
fields should be compressed. To allow a single rule, these values are
identified by their role (DEV or APP) and not by their position in the frame
(source or destination). The SCHC C/D must be aware of the traffic direction
(upstream, downstream) to select the appropriate field.

### IPv6 source and destination prefixes

Both ends must be synchronized with the appropriate prefixes. For a specific
flow, the source and destination prefix can be unique and stored in the
context. It can be either a link-local prefix or a global prefix. In that case,
the TV for the source and destination prefixes contains the values, the MO is
set to "equal" and the CDA is set to "not-sent".

In case the rule allows several prefixes, mapping-list must be used. The
different prefixes are listed in the TV associated with a short ID. The MO is
set to "match-mapping" and the CDA is set to "mapping-sent".

Otherwise the TV contains the prefix, the MO is set to "equal" and the CDA is
set to value-sent.

### IPv6 source and destination IID

If the DEV or APP IID are based on an LPWAN address, then the IID can be
reconstructed with information coming from the LPWAN header. In that case, the
TV is not set, the MO is set to "ignore" and the CDA is set to "DEViid" or
"APPiid". Note that the LPWAN technology is generally carrying a single device
identifier corresponding to the DEV. The SCHC C/D may also not be aware of
these values.

If the DEV address has a static value that is not derived from an IEEE EUI-64,
then TV contains the actual Dev address value, the MO operator is set to
"equal" and the CDA is set to "not-sent".

If several IIDs are possible, then the TV contains the list of possible IIDs,
the MO is set to "match-mapping" and the CDA is set to "mapping-sent".

Otherwise the value variation of the IID may be reduced to few bytes. In that
case, the TV is set to the stable part of the IID, the MO is set to MSB and the
CDA is set to LSB.

Finally, the IID can be sent on the LPWAN. In that case, the TV is not set, the
MO is set to "ignore" and the CDA is set to "value-sent".

## IPv6 extensions

No extension rules are currently defined. They can be based on the MOs and
CDAs described above.

## UDP source and destination port

To allow a single rule, the UDP port values are identified by their role (DEV
or APP) and not by their position in the frame (source or destination). The
SCHC C/D must be aware of the traffic direction (upstream, downstream) to
select the appropriate field. The following rules apply for DEV and APP port
numbers.

If both ends know the port number, it can be elided. The TV contains the port
number, the MO is set to "equal" and the CDA is set to "not-sent".

If the port variation is on few bits, the TV contains the stable part of the
port number, the MO is set to "MSB" and the CDA is set to "LSB".

If some well-known values are used,  the TV can contain the list of this
values, the MO is set to "match-mapping" and the CDA is set to "mapping-sent".

Otherwise the port numbers are sent on the LPWAN. The TV is not set, the MO is
set to "ignore" and the CDA is set to "value-sent".

## UDP length field

If the LPWAN technology does not introduce padding, the UDP length can be
computed from the received data. In that case the TV is not set, the MO is set
to "ignore" and the CDA is set to "compute-UDP-length".

If the payload is small, the TV can be set to 0x0000, the MO set to "MSB" and
the CDA to "LSB".

On other cases, the length must be sent and the CDA is replaced by
"value-sent".

## UDP Checksum field

IPv6 mandates a checksum in the protocol above IP. Nevertheless, if a more
efficient mechanism such as L2 CRC or MIC is carried by or over the L2 (such as
in the LPWAN fragmentation process (see section {{Frag}})), the UDP checksum
transmission can be avoided. In that case, the TV is not set, the MO is set to
"ignore" and the CDA is set to "compute-UDP-checksum".

In other cases the checksum must be explicitly sent. The TV is not set, the MO
is set to "ignore" and the CDF is set to "value-sent".

# Security considerations

## Security considerations for header compression
A malicious header compression could cause the reconstruction of a
wrong packet that does not match with the original one, such corruption
may be detected with end-to-end authentication and integrity mechanisms.
Denial of Service may be produced but its arise other security problems
that may be solved with or without header compression.

## Security considerations for fragmentation
This subsection describes potential attacks to LPWAN fragmentation
and suggests possible countermeasures.

A node can perform a buffer reservation attack by sending a first fragment to a
target. Then, the receiver will reserve buffer space for the IPv6 packet. Other
incoming fragmented packets will be dropped while the reassembly buffer is
occupied during the reassembly timeout. Once that timeout expires, the attacker
can repeat the same procedure, and iterate, thus creating a denial of service
attack. The (low) cost to mount this attack is linear with the number of
buffers at the target node. However, the cost for an attacker can be increased
if individual fragments of multiple packets can be stored in the reassembly
buffer. To further increase the attack cost, the reassembly buffer can be split
into fragment-sized buffer slots. Once a packet is complete, it is processed
normally. If buffer overload occurs, a receiver can discard packets based on
the sender behavior, which may help identify which fragments have been sent by
an attacker.

In another type of attack, the malicious node is required to have overhearing
capabilities. If an attacker can overhear a fragment, it can send a spoofed
duplicate (e.g. with random payload) to the destination. If the LPWAN
technology does not support suitable protection (e.g. source authentication and
frame counters to prevent replay attacks), a receiver cannot distinguish
legitimate from spoofed fragments. Therefore, the original IPv6 packet will be
considered corrupt and will be dropped. To protect resource-constrained nodes
from this attack, it has been proposed to establish a binding among the
fragments to be transmitted by a node, by applying content-chaining to the
different fragments, based on cryptographic hash functionality. The aim of this
technique is to allow a receiver to identify illegitimate fragments.

Further attacks may involve sending overlapped fragments (i.e. comprising some
overlapping parts of the original IPv6 datagram). Implementers should make sure
that correct operation is not affected by such event.

In Window mode – ACK on error, a malicious node may force a fragment sender to
resend a fragment a number of times, with the aim to increase consumption of
the fragment sender’s resources. To this end, the malicious node may repeatedly
send a fake ACK to the fragment sender, with a bitmap that reports that one or
more fragments have been lost. In order to mitigate this possible attack,
MAX_FRAG_RETRIES may be set to a safe value which allows to limit the maximum
damage of the attack to an acceptable extent. However, note that a high setting
for MAX_FRAG_RETRIES benefits fragment delivery reliability, therefore the
trade-off needs to be carefully considered.

# Acknowledgements

Thanks to Dominique Barthel, Carsten Bormann, Philippe Clavier, Arunprabhu
Kandasamy, Antony Markovski, Alexander Pelov, Pascal Thubert, Juan Carlos
Zuniga and Diego Dujovne for useful design consideration and comments.

--- back

# SCHC Compression Examples {#compressIPv6}

# Fragmentation Examples

# Note
