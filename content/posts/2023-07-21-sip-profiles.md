---
title: "Using SIP Profiles on Cisco VoIP Gateways"
date: 2023-07-21T18:15:59-05:00
tags: ['cube','sip','ios','freepbx']
description: "Manipulating SIP headers on Cisco VoIP Gateways"
---

### The problem

I stumbled upon an issue with outbound calls failing that were being made from a Sangoma analog gateway with FreePBX being used as a SIP proxy between the endpoint and the PSTN gateway.

The call flow looks like this:

```
1. Sangoma Analog Gateway <-----> 2. FreePBX <-----> 3. Cisco Gateway <-----> 4. PSTN
```

### Issue with the SIP INVITE

Upon further inspection I found that the initial INVITE was being rejected by the carrier:

```
INVITE sip:13334445567@10.128.20.3 SIP/2.0
Via: SIP/2.0/UDP 10.112.20.12:5060;branch=z9hG4bK28cf9e37
Max-Forwards: 70
From: "vega" <sip:Unknown@10.112.20.12>;tag=as1f8a8d71
To: <sip:13334445567@10.128.20.3>
Contact: <sip:Unknown@10.112.20.12:5060>
Call-ID: 76ef67175a6c86290ec0eac90f561392@10.112.20.12:5060
CSeq: 102 INVITE
User-Agent: FPBX-14.0.17(13.32.0)
Date: Fri, 21 Jul 2023 18:14:17 GMT
Allow: INVITE, ACK, CANCEL, OPTIONS, BYE, REFER, SUBSCRIBE, NOTIFY, INFO, PUBLISH, MESSAGE
Supported: replaces, timer
Content-Type: application/sdp
Content-Length: 250
v=0
o=root 216717988 216717988 IN IP4 10.112.20.12
s=Asterisk PBX 13.32.0
c=IN IP4 10.112.20.12
t=0 0
m=audio 10604 RTP/AVP 0 101
a=rtpmap:0 PCMU/8000
a=rtpmap:101 telephone-event/8000
a=fmtp:101 0-16
a=ptime:20
a=maxptime:150
a=sendrecv
```

In the INVITE above, the From and Contact fields with "Unknown" listed were causing the issue:

```
From: "vega" <sip:Unknown@10.112.20.12>;tag=as1f8a8d71
...
Contact: <sip:Unknown@10.112.20.12:5060>
```

### Creating and applying a SIP profile

Since I don't know much about the Sangoma analog gateway and how to set the CLID for an endpoint, I knew that the quickest method to correct this would be to apply a SIP profile to the outbound dial-peer that would the change the From and Contact fields:

```
Router(config)# voice class sip-profiles 20
Router(config-class)# request INVITE sip-header From modify "(<.*:)(Unknown)" "\13334447009"
Router(config-class)# request INVITE sip-header Contact modify "(<.*:)(Unknown)" "\13334447009"
```

Apply the sip profile to the appropriate dial-peer:

```
Router(config)# dial-peer voice 100 voip
Router(config-dial-peer)# voice-class sip profile 20 
```

Here's the subsequent INVITE from the egress interface towards the carrier:

```
INVITE sip:13334445567@10.38.65.4:5060 SIP/2.0
Via: SIP/2.0/UDP 10.57.5.2:5060;branch=z9hG4bK7088D53
From: "vega" <sip:3334447009@10.38.65.4>;tag=5EC61220-118C
To: <sip:13334445567@10.38.65.4>
Date: Fri, 21 Jul 2023 20:01:18 GMT
Call-ID: 30C464A0-273811EE-947BE063-44373704@10.57.5.2
Supported: 100rel,timer,resource-priority,replaces,sdp-anat
Min-SE:  1800
Cisco-Guid: 0817977151-0657986030-2490753123-1144469252
User-Agent: Cisco-SIPGateway/IOS-12.x
Allow: INVITE, OPTIONS, BYE, CANCEL, ACK, PRACK, UPDATE, REFER, SUBSCRIBE, NOTIFY, INFO, REGISTER
CSeq: 101 INVITE
Timestamp: 1689969678
Contact: <sip:3334447009@10.57.5.2:5060>
Expires: 180
Allow-Events: telephone-event
Max-Forwards: 69
Content-Type: application/sdp
Content-Disposition: session;handling=required
Content-Length: 311
v=0
o=CiscoSystemsSIP-GW-UserAgent 3350 2815 IN IP4 10.57.5.2
s=SIP Call
c=IN IP4 10.57.5.2
t=0 0
m=audio 19402 RTP/AVP 0 100 101 19
c=IN IP4 10.57.5.2
a=rtpmap:0 PCMU/8000
a=rtpmap:100 X-NSE/8000
a=fmtp:100 192-194
a=rtpmap:101 telephone-event/8000
a=fmtp:101 0-16
a=rtpmap:19 CN/8000
a=ptime:20
```

As you can see above, the From and Contact headers have been modified to show a legitimate number.

### Update

Rebuilding the configuration between the Sangoma gateway and FreePBX seemed to do the trick with regards to setting the 
caller ID. I installed the *Vega Gateway Module* on the FreePBX server and setup each analog line as its own 
SIP user/extension.
