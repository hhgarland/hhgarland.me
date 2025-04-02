+++
title = 'CAR Scheduler High CPU Utilization'
date = 2025-04-01T20:26:12-05:00
draft = false
+++

## Issue

High CPU utilization showing on Call Manager Publisher node.

## Troubleshooting

### CUCM CLI

Issue the command `show process load` on the publisher node:

```
admin:show process load
top - 10:30:14 up 179 days, 12:13,  1 user,  load average: 6.83, 6.28, 5.77
Tasks: 264 total,   1 running, 262 sleeping,   0 stopped,   1 zombie
Cpu(s): 25.4%us,  5.4%sy,  0.0%ni, 68.9%id,  0.1%wa,  0.1%hi,  0.2%si,  0.0%st
Mem:   5994000k total,  5841020k used,   152980k free,   266876k buffers
Swap:  2064284k total,  1330460k used,   733824k free,   803676k cached
  PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND
27044 ccmservi  20   0  908m 530m 7020 S 173.6  9.1  11387:53 carschlr
 2142 administ  30  10 1345m  72m  15m S  3.3  1.2   0:05.62 java
 6657 ccmservi  20   0  391m  41m  22m S  3.3  0.7   6054:03 RisDC
  843 root      20   0  4060  556  504 S  1.6  0.0  67:46.09 rngd
 7141 ccmbase   20   0  530m 196m  29m S  1.6  3.4   4861:31 ccm
10208 servmgr   20   0 99.2m 7964 2468 S  1.6  0.1 197:58.06 servM
10867 ccmservi  20   0  797m  88m 6152 S  1.6  1.5 238:16.42 BPS
    1 root      20   0 19592 1572 1076 S  0.0  0.0  27:29.75 init
    2 root      20   0     0    0    0 S  0.0  0.0   0:00.00 kthreadd
    3 root      RT   0     0    0    0 S  0.0  0.0  10:50.02 migration/0
```

The process above, *carschlr*, with PID 27044 shows a high CPU percentage. The command `show process using-most cpu` also confirmed the PID above was causing an issue.

## Correction

1. Log into *Cisco Unified Serviceability* page.
2. Navigate to *Tools > Control Center - Network Services*
3. Select the Call Manager Publisher node IP.
4. Scroll down to *CDR Services* and restart the service *Cisco CAR Scheduler.*
