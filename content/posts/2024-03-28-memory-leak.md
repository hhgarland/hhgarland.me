---
title: "Memory Leak in Directory Sync Code"
date: 2024-03-28T05:37:05-05:00
tags: ['cucm']
---
When making a configuration change to a SIP trunk in CUCM, I was met with the following error: *Update failed. Memory allocation failed during query processing.*  

Troubleshooting steps:

1. From the CUCM publisher node issue the `show status` command to check the memory totals.
2. Also check the service list using `utils service list` to verify if the *A Cisco DB* service is started.
3. Restart the *A Cisco DB* service with `utils service restart A Cisco DB`.
4. After the service has restarted, try saving the configuration again for the SIP trunk.
