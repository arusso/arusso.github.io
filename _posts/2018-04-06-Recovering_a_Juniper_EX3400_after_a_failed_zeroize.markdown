---
title: "Recovering a Juniper EX3400 after a failed zeroize"
layout: post
date: 2018-04-06 19:55:40
comments: true
tags: [ex2300, ex4400, JunOS, Juniper, 15.1X53-D55.5, 15.1X53-D56]
share: true
---

I recently zeroized[^1] a handful of EX3400s running 15.1X53-D55.5, and found
that they would no longer properly boot into the kernel once the zeroizing
process finished. Unable to get the USB recovery methods to work for me, I did
the following to recover the device.

First, boot into the JunOS volume manually

        loader> set currdev=disk0p2
        loader> include /boot/loader.rc
        loader> boot

Next, upgrade the firmware to 15.1X53-D56 or later. I found I couldn't proceed
without doing this first but you're mileage may vary. In my case I used our
ZTP provisioning tools to re-provision the device with a minimal configuration
using 15.1X53-D56. Use whatever works for you.

After you've upgraded, boot again into the JunOS volume and clean up the
`/var/tmp` (mounted on `/`) filesystem so you have as much space as possible.
You need around 700MB but I was able to clear over 800MB by cleaning up old
non-recovery snapshots as well as deleting the upgrade image out of `/var/tmp`.

Once you have enough space, you can run `request system recover oam-volume`.
This process can take about 10m, but should complete successfully. If you ran
into issues, see the end of the article for steps you need to take to try again.

After succesfull completion, reboot into the OAM and recover the JunOS volume
by running:

        request system reboot oam
        # after the host boots into the OAM volume
        request system recover junos-volume
        request system reboot junos

If everything ran successfully, you should be booted back into a working
switch.

**Recovering from a failed OAM recovery attempt**

If you ran out of space or some other error occurred during the recovery of the
oam-volume, you'll need to run `umount /dev/gpt/oam` to unmount the oam
filesystem. Otherwise you'll run into a bug in D55.5 and D56 versions that fail
to umount the correct filesystem and will continue to fail until you unmount or
reboot.

In one case the unmount process wasn't enough and I found I had to reboot and
start from the beginning. If it doesn't work for you, keep at it -- provided
there's no evidence of a hardware issue you should be able to recover all the
devices.

[^1]: https://www.juniper.net/documentation/en_US/junos/topics/reference/command-summary/request-system-zeroize.html
