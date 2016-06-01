---
title: Uniform Dynamic Clusters
menu_order: 50
---

A uniform dynamic cluster has the following characteristics:

* Recovers automatically after reboots and partitions.
* All peers have identical configuration.
* Initial cluster peers can make progress from the outset even if
  bootstrapping occurs under conditions of partition.
* Once an initial cluster has been bootstrapped, arbitrary numbers of
  new peers can be added in parallel without coordination. This makes
  it ideally suited for use with convergent provisioning tools that
  operate across multiple hosts in an asynchronous fashion.

## Bootstrapping

On each initial peer, at boot, via
[systemd](/site/installing-weave/systemd.md):

    host1$ weave launch --no-restart --name ::1 --ipam-seed ::1,::2,::3 $PEERS
    host2$ weave launch --no-restart --name ::2 --ipam-seed ::1,::2,::3 $PEERS
    host3$ weave launch --no-restart --name ::3 --ipam-seed ::1,::2,::3 $PEERS

Where `$PEERS` is obtained from `/etc/sysconfig/weave` as described in
the linked systemd documentation, and includes the complete set of
initial peers.

## Adding a Peer

On each new peer, at boot, via
[systemd](/site/installing-weave/systemd.md):

    hostN$ weave launch --no-restart --name ::N --ipam-seed ::1,::2,::3 $PEERS

* `--no-restart` disables the Docker restart policy as this will be
  handled by systemd.
* `--name` specifies a unique name for this new peer.
* `--ipam-seed` specifies the names of only those peers that were
  involved in the initial cluster bootstrap - even if they have been
  subsequently removed from the cluster. You can view this as a kind
  of 'cluster identity' - peers may only interoperate in the same
  cluster if they share the same seed.
* `$PEERS` is obtained from `/etc/sysconfig/weave` as described in the
  linked systemd documentation. For convenience, this may contain the
  address of the peer which is being launched, so that you don't have
  to compute separate lists of 'other' peers tailored to each peer -
  just supply the same complete list of peer addresses to every peer.

Note that unlike [Interactive](/site/operational-guide/interactive.md)
and [Uniform Fixed
Cluster](/site/operational-guide/uniform-fixed-cluster.md) deployments
there is no `weave prime` step; you can add as many new peers in
parallel as you like, even under conditions of partition, and they
will all (eventually) join safely. This is ideal for use in
conjunction with asynchronous provisioning systems such as puppet or
chef. 

For maximum robustness, you should then distribute an updated
`/etc/sysconfig/weave` file including the new peer to all existing
peers.

### Removing a Peer

On peer to be removed:

    weave reset

You may remove a seed peer, as long as there is at least one other
peer in the network (seed or non-seed) which can take ownership of its
range.

Then distribute an updated `/etc/sysconfig/weave` to the remaining
peers, omitting the removed peer from `$PEERS`.

On each remaining peer:

    weave forget <removed peer>

This step is not mandatory, but it will eliminate log noise and
spurious network traffic by stopping reconnection attempts.
