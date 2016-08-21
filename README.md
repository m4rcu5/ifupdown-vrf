# ifupdown-vrf

Some simple scripts to transparently set up VRF's on
Debian-like systems.  Copy the directories to `/etc/network`.

To configure an interface enslaved to a VRF use the following stanza:
```
auto veth0
iface veth0 inet static
    address 10.0.0.1/24
    vrf-id  10
```
By running `ifup veth0` it will then create your device, transparently set up the VRF device and enslave your device to it.

Resulting in:
```
3: veth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master vrf-10 state UNKNOWN group default qlen 1000
    link/ether 72:ed:36:22:30:6d brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.1/24 brd 10.0.0.255 scope global veth0
       valid_lft forever preferred_lft forever
4: vrf-10: <NOARP,MASTER,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN group default qlen 1000
    link/ether e2:c7:9d:73:d3:32 brd ff:ff:ff:ff:ff:ff

```

Upon running `ifdown veth0` it will destroy your device and if the VRF is no longer in use, also destroy the VRF.

### Thoughts

To use VRF's in linux, you will need a recent kernel and even more recent iproute2 package installed (>= iproute2-4.6).

You can also add a device route to the VRF device in order to access the routing table in the VRF. Example:
`ip route add 10.0.0.0/24 dev vrf-10`

The VRF routing table is calculated as (VRF_ID + 100) and can be accessed by the iproute command:
```
~ # ip route show table 110
broadcast 10.0.0.0 dev veth0  proto kernel  scope link  src 10.0.0.1
10.0.0.0/24 dev veth0  proto kernel  scope link  src 10.0.0.1 
local 10.0.0.1 dev veth0  proto kernel  scope host  src 10.0.0.1 
broadcast 10.0.0.255 dev veth0  proto kernel  scope link  src 10.0.0.1
```
