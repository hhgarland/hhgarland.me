---
title: "CUBE Reconfiguration"
date: 2024-01-04T20:34:59-05:00
tags: ['cube','sip','ios']
description: "Simplifying a CUBE Configuration"
---

I was inspired to simplify my CUBE configuration after reading [this](https://learningnetwork.cisco.com/s/article/CUBE-Simplicity) article on the Cisco Learning website. The *voice service voip* section of the configuration was already in pretty good shape; however, the folliwing areas needed some work:

1. SIP URI definitions
2. E.164 pattern maps
3. Dial peer groups
4. Server groups
5. Translation profiles
6. Dial peers

### Defining the SIP URIs

For the inbound WAN and LAN dial peers, I wanted to match on the *incoming uri via* headers. Let's define them here for both the ITSP and CCM:
```
voice class uri ITSP sip 
 host ipv4:162.223.86.245 
 host ipv4:162.223.83.245 
! 
voice class uri CCM sip 
 host ipv4:10.113.20.20 
 host ipv4:10.113.20.30 
 host ipv4:192.168.70.40 
 host ipv4:192.168.70.50 
```

### E.164 Pattern Maps

The voice class e164-pattern-map 200 will match on any destination that includes one of the organization's DIDN. A single e164 entry of %.T would technically cover the incoming DIDNs; however, I wanted to list them in NPANXX…. to provide documentation of the actual DIDNs. 

The voice class e164-pattern-map 400 will match on any patterns dialed from a CCM endpoint. The voice translation rules and profile, *Normalize-To-ITSP*, will take care of the outbound emergency, local, long-distance, and international destination patterns. The route patterns themselves could also be referenced in the pattern map but using translation rules provides an easier way to manipulate the number(s) at the gateway level if needed. 

```
voice class e164-pattern-map 200 
 description ** DIDNs sent to CCM; CCM SIP trunk matching on last 4 ** 
  e164 333539.... 
  e164 333540.... 
  e164 333541.... 
  e164 333561.... 
  e164 333562.... 
 ! 
! 
voice class e164-pattern-map 400 
 description ** Route patterns to ITSP ** 
  e164 .T 
```
### Dial Peer groups

Dial peer groups provide myself a way to prevent looping calls back to ITSP provider. Dial peers are excluded as the dial-peer groups create a static binding to their respective destinations: 

```
voice class dpg 201 
 description ** Inbound WAN DPG to dial-peer 200 ** 
 dial-peer 200 
! 
voice class dpg 401 
 description ** Inbound LAN DPG to dial-peer 400 ** 
 dial-peer 400 
```
### Server Groups
```
voice class server-group 200 
 ipv4 10.113.20.20 preference 1
 ipv4 10.113.20.30 preference 2
 ipv4 192.168.70.40 preference 3
 ipv4 192.168.70.50 preference 4
 description ** CCM Targets ** 
! 
voice class server-group 400 
 ipv4 162.223.86.245 preference 1
 ipv4 162.223.83.245 preference 2
 description ** ITSP Targets ** 
```

We can point to multiple ITSP targets with server groups.

### Translation Rules and Profiles

The translation profile below is applied to dial peer voice 400. The called number is altered on the outgoing/destination dial peer. The destination must match how it was originally received on the incoming dial peer. 

Rules 1 and 2 are used for emergency calls. Rules 3 and 4 are used for local 7-digit and 10-digit calls. Rule 5 is used for long-distance 11-digit calls. Rule 6 is used for international calls. 

```
voice translation-rule 400 
 rule 1 /^911$/ /\0/ 
 rule 2 /^988$/ /\0/ 
 rule 3 /^[2-9]......$/ /1333\0/ 
 rule 4 /^333[2-9]......$/ /1\0/ 
 rule 5 /^1[2-9]..[2-9]......$/ /\0/ 
 rule 6 /^011.*/ /\0/ 
! 
voice translation-profile Normalize-To-ITSP 
 translate called 400
```
### Dial Peers

```
dial-peer voice 100 voip 
 description ** Inbound WAN dial-peer from ITSP to CUBE ** 
 session protocol sipv2 
 session transport udp 
 destination dpg 201 
 incoming uri via ITSP 
 voice-class codec 1   
 voice-class sip bind control source-interface GigabitEthernet0/0/1 
 voice-class sip bind media source-interface GigabitEthernet0/0/1 
 dtmf-relay cisco-rtp rtp-nte 
 no vad 
! 
dial-peer voice 200 voip 
 description ** Outbound LAN dial-peer from CUBE to CCM ** 
 session protocol sipv2 
 session transport udp 
 session server-group 200 
 destination e164-pattern-map 200 
 voice-class codec 1   
 voice-class sip bind control source-interface GigabitEthernet0/0/0 
 voice-class sip bind media source-interface GigabitEthernet0/0/0 
 dtmf-relay cisco-rtp rtp-nte 
 no vad 
! 
dial-peer voice 300 voip 
 description ** Inbound LAN dial-peer from CCM to CUBE ** 
 session protocol sipv2 
 session transport udp 
 destination dpg 401 
 incoming uri via CCM 
 voice-class codec 1   
 voice-class sip bind control source-interface GigabitEthernet0/0/0 
 voice-class sip bind media source-interface GigabitEthernet0/0/0 
 dtmf-relay cisco-rtp rtp-nte 
 no vad 
! 
dial-peer voice 400 voip 
 description ** Outbound WAN dial-peer from CUBE to ITSP ** 
 translation-profile outgoing Normalize-to-ITSP 
 session protocol sipv2 
 session transport udp 
 session server-group 400 
 destination e164-pattern-map 400 
 voice-class codec 1   
 voice-class sip bind control source-interface GigabitEthernet0/0/1 
 voice-class sip bind media source-interface GigabitEthernet0/0/1 
 dtmf-relay cisco-rtp rtp-nte 
 no vad
```
Of course this could be accomplished by using just two dial peers, but using four dial peers gives me a better understanding of the inbound/outbound call flow when debugging.

