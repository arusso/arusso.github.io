---
title: "Podman CNI Networking on CentOS 7"
slug: Podman_CNI_Networking_on_CentOS_7
layout: post
date: 2019-12-01 00:32:12
comments: true
tags: [podman, cni, networking]
share: true
summary: >
  CNI failed to setup DNAT rules for our Podman containers, leaving them unreachable.
aliases:
  - /Podman_CNI_Networking_on_CentOS_7
  - /Troubleshooting_Podman_CNI_Networking_on_CentOS_7
---
Since moving exclusively to [podman](https://podman.io) a ways back, the only
major issue I've run into is that the CNI networking will break periodically
with version updates.

Normally when a port-binding exists, you'll see the dnat rules on the
`CNI-HOSTPORT-DNAT` chain of the `nat` table in `iptables`. Similar rules should
show up in `nftables`. For instance --


{{<codeWide language="shell" line-numbers="false">}}
# iptables -nvL CNI-HOSTPORT-DNAT -t nat
Chain CNI-HOSTPORT-DNAT (2 references)
 pkts bytes target     prot opt in     out     source               destination
    4   240 CNI-DN-550b4bf3691bef6919331  tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            /* dnat name: "podman" id: "aea317babc7f3ec7607b242227b39c90a43b50a7dd0f0b8fbccc82620370831d" */ multiport dports 8081

# iptables -nvL CNI-DN-550b4bf3691bef6919331 -t nat
Chain CNI-DN-550b4bf3691bef6919331 (1 references)
 pkts bytes target     prot opt in     out     source               destination
    0     0 CNI-HOSTPORT-SETMARK  tcp  --  *      *       10.88.0.66           0.0.0.0/0            tcp dpt:8081
    4   240 CNI-HOSTPORT-SETMARK  tcp  --  *      *       127.0.0.1            0.0.0.0/0            tcp dpt:8081
    4   240 DNAT       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:8081 to:10.88.0.66:80
{{</codeWide>}}

As we can see, traffic bound for the port `0.0.0.0:8081` on the host will be
sent to `10.88.0.66:80` which is the containers address on the cni-bridge.

Periodically, this setup will stop functioning altogether and the `nat` table
chains and rules will not be created. The easiest way I've found to track this
issue down is to run podman with logging turned way up and look for warnings
and errors.

In my case, the following warning stood out to me when I ran into issues last:

{{<codeWide language="shell" line-numbers="false">}}
WARN[0000] Error loading CNI config file /etc/cni/net.d/87-podman-bridge.conf: error parsing configuration: missing 'type'
{{</codeWide>}}

This file was taken verbatim from the libpod repo, and it turns out that the
instructions were actually incorrect -- the file is not a standard
configuration file but is actually considered to be a [config
list](https://github.com/containernetworking/cni/blob/master/SPEC.md#network-configuration-lists)
file, and thus should be named accordingly.

TL;DR:

The solution was dead simple -- rename `/etc/cni/net.d/87-podman-bridge.conf`
to `/etc/cni/net.d/87-podman-bridge.conflist` and relaunch my pods.
