---
title: "'Failed type 6 encryption on password'"
date: 2023-06-17T06:44:59-05:00
tags: ['cube','sip','ios']
description: "Changing an encrypted SIP binding password"
---

I ran into an issue recently when attempting to change the password on an previously configured SIP binding (information has been changed for security reasons):

```
site-1-cube(config-sip-ua)#credentials username 3334445555 password 0 J2[%)_Ie realm nyc.provider.com
Failed type 6 encryption on password
```

I had never seen the error *Failed type 6 encryption on password* before and I struggled somewhat in understanding how to change this. I ran into [this](https://community.cisco.com/t5/networking-knowledge-base/configuring-type-6-passwords-in-ios-xe/ta-p/4438495) article and it was subsequently the same solution I was given by Cisco TAC:

```
configure terminal
  key config-key password-encrypt <MASTER PASSWORD>
  password encryption aes
end
```

After entering the commands above, I was able to change the SIP bindings:

```
site-1-cube(config)#no sip-ua
site-1-cube(config-sip-ua)#sip-ua
site-1-cube(config-sip-ua)#credentials username 3334445555 password 0 J2--[%)_Ie realm nyc.provider.com
site-1-cube(config-sip-ua)#authentication username 3334445555 password 0 J2i--[%)_Ie realm nyc.provider.com
```

Then I checked the registration status with:

```
site-1-cube#show sip-ua register status
```
