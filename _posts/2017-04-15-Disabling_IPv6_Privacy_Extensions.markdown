---
title: "Disabling IPv6 Privacy Extensions"
layout: post
date: 2017-04-15 00:56:23
comments: true
tags: [ipv6,slaac,linode,nmcli,"network manager",centos,rhel]
share: true
---

[RFC7217](https://tools.ietf.org/html/rfc7217) describes a privacy extension
that aims to improve upon [RFC4941](https://tools.ietf.org/html/rfc4941). In
contrast to RFC4941 which provides random and temporary addresses, RFC7217
provides a stable address that provides much of the same benefits as the
temporary addresses.

Known as 'stable-privacy' in Network Manager, RFC7217 is not desirable in some situations. For example, in a hosted environment where the IPv6 address is expected to be determinstic based on the MAC address, as is the case on Linode.

Linode uses SLAAC to provide IPv6 addresses to it's nodes. With SLAAC, Linode's routers advertise the prefix for the IPv6 network and the hosts, if configured properly, will use that information combined with the hardware address to generate an IPv6 address for use.

On Red Hat based systems like RHEL and CentOS, NetworkManager interferes with this process by defaulting to using  RFC7217 to configure stable addresses. In order to disable this behavior, we need to reconfigure NetworkManager to not use stable addresses.

To check and see if your interface is running in 'stable-privacy' mode, run:

    $ nmcli conn show "Wired interface 1" | grep ipv6.addr-gen-mode
    ipv6.addr-gen-mode:                     stable-privacy

Above, we are running in stable-privacy mode. We want to disable privacy extensins altogether and run in eui64 mode. Update our connection configuration by running:

    $ sudo nmcli conn edit "Wired interface 1" ipv6.addr-gen-mode eui64
    # reload our connection
    $ for action in down up; do sudo nmcli conn $action "Wired connection 1"; done

Our IPv6 address should now properly reflect the combination of your prefix and hardware address.
