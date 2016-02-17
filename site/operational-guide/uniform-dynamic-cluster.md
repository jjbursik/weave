---
title: Uniform Dynamic Clusters
menu_order: 50
---

A uniform dynamic cluster has the following characteristics:

* Recovers automatically after reboots and partitions.
* All peers have identical configuration.
* Once an initial cluster has been bootstrapped, arbitrary numbers of
  new peers can be added in parallel without coordination.
* Initial cluster peers can make progress from the outset even if
  bootstrapping occurs under conditions of partition.

## Bootstrapping

    host1$ weave launch --name ::1 --ipam-seed ::1,::2,::3 host1 host2 host3
    host2$ weave launch --name ::2 --ipam-seed ::1,::2,::3 host1 host2 host3
    host3$ weave launch --name ::3 --ipam-seed ::1,::2,::3 host1 host2 host3

## Adding a Peer

On new peer:

    hostN$ weave launch --name ::N --ipam-seed ::1,::2,::3 host1 host3 host3 ... hostN

Note that unlike [Interactive](/site/operational-guide/interactive.md)
and [Uniform Fixed
Cluster](/site/operational-guide/uniform-fixed-cluster.md) deployments
there is no `weave prime` step; you can add as many new peers in
parallel as you like, even under conditions of partition, and they
will all (eventually) join safely. This is ideal for use in
conjunction with asynchronous provisioning systems such as puppet or
chef. 

On each existing peer:

    weave connect <new peer>

This step is not mandatory, but improves the robustness of network
reformation in the face of node failure as _nodes do not remember to
connect to discovered peers on restart_.

### Removing a Peer

On peer to be removed:

    weave reset

You may remove a seed peer, as long as there is at least one other
peer in the network (seed or non-seed) which can take ownership of its
range.

On each remaining peer:

    weave forget <removed peer>

This step is not mandatory, but it will eliminate log noise and
spurious network traffic by stopping reconnection attempts and
preventing further connection attempts after restart.
