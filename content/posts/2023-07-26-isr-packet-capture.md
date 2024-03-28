---
title: "Packet Capture on Cisco ISR"
date: 2023-07-26T09:02:59-05:00
tags: ['cube','isr','ios']
---
1. Determine the interface where you want to capture traffic:
```
show ip interface brief
```

2. Stage the capture filters:
```
monitor capture CAP1 interface gig0/0/0 both
monitor capture CAP1 match ipv4 any any
```

3. Start the capture:
```
monitor capture CAP1 start
```

4. Reproduce the issue.
5. Stop the capture:
```
monitor capture CAP1 stop
```

6. Export the capture to a TFTP server:
```
monitor capture CAP1 export tftp://<tftp IP>/CAP1.pcap
```

7. Disable the monitor capture:
```
no monitor capture CAP1
```

8. Verify the capture:
```
router1#sh monitor capture CAP1 buffer brief
 ----------------------------------------------------------------------------
 #   size   timestamp     source             destination      dscp    protocol
 ----------------------------------------------------------------------------
   0  118    0.000000   10.113.20.7      ->  10.114.1.10      0  BE   UDP
   1  543    0.261980   10.113.20.7      ->  162.223.86.235   26 AF31 UDP
   2  520    0.310989   162.223.86.235   ->  10.113.20.7      24 CS3  UDP
   3  792    6.020003   162.223.86.235   ->  10.113.20.7      24 CS3  UDP
   4 1128    6.022994   10.113.20.7      ->  162.223.86.235   26 AF31 UDP
   5  118   10.131997   10.148.202.2     ->  10.113.20.7      0  BE   TCP
   6  118   10.133996   10.113.20.7      ->  10.148.202.2     48 CS6  TCP
   7   54   10.182989   10.148.202.2     ->  10.113.20.7      0  BE   TCP
   8  118   10.211995   10.148.202.2     ->  10.113.20.7      0  BE   TCP
   9  118   10.214985   10.113.20.7      ->  10.148.202.2     48 CS6  TCP
  10   54   10.261980   10.148.202.2     ->  10.113.20.7      0  BE   TCP
  11  118   10.342029   10.148.202.2     ->  10.113.20.7      0  BE   TCP
  <cont'd...>
```
