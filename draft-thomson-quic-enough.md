---
title: "Signaling That a QUIC Receiver Has Enough Stream Data"
abbrev: "QUIC Enough"
category: std

docname: draft-thomson-quic-enough-latest
submissiontype: IETF  # also: "independent", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: AREA
workgroup: WG Working Group
keyword:
 - next generation
 - unicorn
 - sparkling distributed ledger
venue:
  group: WG
  type: Working Group
  mail: WG@example.com
  arch: https://example.com/WG
  github: USER/REPO
  latest: https://example.com/LATEST

author:
 -
    fullname: Martin Thomson
    organization: Mozilla
    email: mt@lowentropy.net

normative:
  RESET-AT: I-D.seemann-quic-reliable-stream-reset
  QUIC: RFC9000

informative:


--- abstract

Sending on QUIC streams can only be aborted early by the sender with a RESET_STREAM frame.
This document describes how a receiver can indicate when the data they have received is enough,
allowing the sender to reliably deliver some data, but abort sending for anything more than the indicated amount.

--- middle

# Introduction

RFC 9000 {{QUIC}} describes how streams can be completed or reset.  A completed stream is delivered in its entirety, with the sender sending new STREAM frames until all data has been received and acknowledged by a receiver.  On the other hand, data on a reset stream might never be sent, or - if it is sent - it might not be retransmitted in the case of loss.

The RESET_AT frame {{RESET-AT}} provides a way for a sender to split a stream into two parts: an early part that is transmitted reliably and any remainder, which is effectively reset.  This combines aspects of both completed and reset streams, even though the entire content of a stream is not delivered.

This document describes the ENOUGH frame in {{enough}}, which provides an analogue of the STOP_SENDING QUIC frame ({{Section 19.11 of QUIC}}).   This allows a stream receiver to indicate that it has received enough data and that any data beyond the indicated offset is not needed.  A sender that receives this ENOUGH frame can cease sending and emit a RESET_AT frame at or beyond the indicated offset.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# QUIC ENOUGH Frame {#enough}

A QUIC ENOUGH frame (0xTBD) is shown in {{fig-enough}}.

~~~
ENOUGH Frame {
  Type (i) = 0xTBD,
  Stream ID (i),
  Application Protocol Error Code (i),
  Offset (i),
}
~~~
{: #fig-enough title="The QUIC ENOUGH Frame Format"}

Like STOP_SENDING, an ENOUGH frame can be sent for streams in the "Recv" or "Size Known" states; see {{Section 3.2 of QUIC}}. Receiving a STOP_SENDING frame for a locally initiated stream that has not yet been created MUST be treated as a connection error of type STREAM_STATE_ERROR; see {{Section 20.1 of QUIC}}. An endpoint that receives an ENOUGH  frame for a receive-only stream MUST terminate the connection with error STREAM_STATE_ERROR.

ENOUGH frames contain the following fields:

Stream ID:
:   A variable-length integer carrying the stream ID of the stream being ignored.

Application Protocol Error Code:
:   A variable-length integer containing the application-specified reason the sender is ignoring the stream; see {{Section 20.2 of QUIC}}.

Offset:
:   A variable-length offset into the stream.  Note that a value of 0 makes this frame equivalent to STOP_SENDING.  This value MAY be larger than the final size of the frame; see {{Section 4.5 of QUIC}}, in which case this frame has no effect.

ENOUGH frames have no direct effect on the stream state machine.

# Negotiation

Endpoints advertise their support of the extension described in this document by sending the enough (0xTBD) transport parameter ({{Section 7.4 of QUIC}}) with an empty value. An implementation that understands this transport parameter MUST treat the receipt of a non-empty value as a connection error of type TRANSPORT_PARAMETER_ERROR; see {{Section 20.1 of QUIC}}.

In order to allow reliable stream resets in 0-RTT packets, the client MUST remember the value of this transport parameter. If 0-RTT data is accepted by the server, the server MUST not disable this extension on the resumed connection.

This extension MUST NOT be advertised unless support for the RESET_AT frame is also advertised; see {{Section 3 of RESET-AT}}.  An endpoint MUST treat receipt of this transport parameter without the reliable_reset_stream transport parameter as a connection error of type TRANSPORT_PARAMETER_ERROR.

# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

Martin Duke asked if this was possible.  This is the result.
