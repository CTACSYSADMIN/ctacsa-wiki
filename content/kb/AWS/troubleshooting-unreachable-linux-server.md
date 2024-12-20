---
title: "troubleshooting unreachable linux server"
date: 2022-02-05T10:31:29-05:00
---
```
Knowledge Base Article - CTAC (Communications Training Analysis Corporation)  
=================================================


Subject: Troubleshooting a server that won't start/you can't ssh into/is unreachable

Revision History:
    2017-07-13 12:53:00 - Version 1 created by Bao Nguyen
    

Details: (include steps to reproduce, symptoms, corrective actions, contact information, links and a any other helpful information)


Note:
- this applies to Linux. Not sure the steps for a Windows server.

Basically you're just detaching the volume from an existing instance and attaching it to another one. But here are the details (https://aws.amazon.com/premiumsupport/knowledge-center/ec2-unresponsive-troubleshoot/. There is one other link out there somewhere that pretty much describes the same steps, but I can't find it. Feel free to add it here if you come across that link.

troubleshooting an ebs volume:
- stop the instance
- make a note of the availability zone for the instance.
- make a note of the mountpoint for the instance
- create a temporary instance (t2.micro will do) in the availability zone of the above instance
- make a note of the volume to detach and detach it from the broken server
eg vol-8cf03a53
- Attach the detached volume to the forensics server
- ssh into the forensics server, mkdir a directory somewhere (ex: mkdir /forensicsdir), and mount the volume. You will then be able to navigate throughout the filesystem as a mount (ex: vi /forensicsdir/etc/fstab)
- make corrections
- shut down the forensics server
- detach the volume from the forensics server
- attach the volume to the original server 
- turn on the original server
- ssh into the original server


=================================================
This document is proprietary and confidential. No part of this document may be 
disclosed in any manner to a third party without the prior written consent of 
CTAC.```
