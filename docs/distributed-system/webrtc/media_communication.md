# Media communication

WebRTC uses RTP and RTCP for media communication.

## 1. RTP

The Real-time Transport Protocol(RTP) is a network protocol used to deliver streaming audio and video media over the internet.

## 2. RTCP

RTCP stands for Real-time Transport Control Protocol. RTCP works hand in hand with RTP. RTP does the delivery of the actual data, whereas RTCP is used to send control packets to participants in a call. The primary function is to provide feedback on the quality of service being provided by RTP. 

Every RTCP packet has the following structure:

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|V=2|P|    RC   |       PT      |             length            |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                            Payload                            |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

```

**Version (V)** 

`Version` is always `2`

**Padding (P)**

`Padding` is a bool that controls if the payload has padding.
The last byte of the payload contains a count of how many padding bytes was added

**Reception Report Count (RC)**

The number of reports in this packet. A single RTCP packet can contain multiple events.

**Packet Type (PT)**

Unique Identifier for what type of RTCP Packet this is.

- `192` Full INTRA-frame Request (FIR)
- `193` Negative ACKnowledgements (NACK)
- `200` Sender Report
- `201` Receiver Report
- `205` Generic RTP Feedback
- `206` Payload Specific Feedback

**Full INTRA-frame Request (FIR) and Picture Loss Indication (PLI)**

Both FIR and PLI messages serve a similar purpose. These messages request a full key frame from the sender

PLI is used when partial frames were given to the decoder, but it was unable to decode them. This could happen because you had lots of packet loss, or maybe the decoder crashed.

FIR shall not be used when packets or frames are lost. That is PLIs job. FIR requests a key frame for reasons other than packet loss - for example when a new member enters a video conference. They need a full key frame to start decoding video stream, the decoder will be discarding frames until key frame arrives.

**Negative Acknowledgment**

A NACK requests that a sender re-transmits a single RTP packet. This is usually caused by an RTP packet getting lost, but could also happen because it is late.

**Sender and Receiver Reports**

These reports are used to send statistics between agents. This communicates the amount of packets actually received and jitter.

The reports can be used for diagnostics and congestion control.
