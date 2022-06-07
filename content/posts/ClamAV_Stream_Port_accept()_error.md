---
title: "ClamAV Stream Port accept() error"
date: 2012-01-17 16:08:00-07:00
summary: Identifying and fixing SELinux misconfiguration when using ClamAV Stream Ports
tags: [clamav,selinux]
comments: true
aliases:
  - /ClamAV_Stream_Port_accept()_error
---
I was working on a server where the customer was using ClamAV and Stream Ports
to check for viruses.  They had a problem where the server would not accept
their connection.  The logfiles for the error showed up like:


{{<codeWide language="plaintext">}}
Sun Jan  1 08:00:03 2012 -> ERROR: ScanStream 1088: accept() failed.
{{</codeWide>}}

At first I thought it was a firewall rule, but after looking things over it
looked ok.  In my googling, I noticed a lot of people had problems with SELinux
and since this system did in fact run SELinux, I started looking at those
logfiles.  I found the following errors (formatted for readability):

{{<codeWide language="plaintext" line-numbers="false">}}
type=AVC msg=audit(1326835719.949:35310855): avc:  denied  { name_bind } for
    pid=4901 comm="clamd" src=1505 scontext=user_u:system_r:clamd_t:s0
    tcontext=system_u:object_r:port_t:s0 tclass=tcp_socket
type=SYSCALL msg=audit(1326835719.949:35310855): arch=c000003e syscall=49
    success=no exit=-13 a0=c a1=415d0e50 a2=10 a3=41a705af1fe3fb79 items=0
    ppid=1 pid=4901 auid=4294967295 uid=3218 gid=3218 euid=3218 suid=3218
    fsuid=3218 egid=3218 sgid=3218 fsgid=3218 tty=(none) ses=4294967295
    comm="clamd" exe="/usr/sbin/clamd" subj=user_u:system_r:clamd_t:s0
    key=(null)
{{</codeWide>}}

This was one of my first rodeo's with SELinux, so further research on the
`name_bind` permission was necessary. I found that this occurs when an
application tries to open a port they aren't allowed to. I checked the SELinux
configuration to see what ports it would allow ClamAV to open:

{{<codeWide language="bash" line-numbers="false">}}
$ sudo semanage port -l | grep clamd
clamd_port_t tcp 3310
{{</codeWide>}}

Bingo! The conf file for my client was defaulting to the streamports being open
from 1024-2048, so I added that exception:

{{<codeWide language="bash" line-numbers="false">}}
$ sudo semanage port -a -t clamd_port_t -p tcp 1024-2048
$ sudo semanage port -l | grep 3310
clamd_port_t tcp 1024-2048, 3310
{{</codeWide>}}

Once done, I tested...

{{<codeWide language="bash" line-numbers="false">}}
$ telnet localhost 3310 Trying 127.0.0.1...
Connected to localhost.localdomain (127.0.0.1).
Escape character is '^]'.
STREAM
PORT 1246
^]
{{</codeWide>}}

And confirmed it worked!
