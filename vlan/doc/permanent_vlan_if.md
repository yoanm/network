# How to add a permanent vlan interface on linux

 * [8021q module configuration](#8021q-module)
 * [Vlan interface configuration](#vlan-interface)
   * [Vlan with DHCP ip assignation](#vlan-interface-dhcp)
   * [Vlan with static ip assignation](#vlan-interface-static)
   
   
Sources :
 * https://wiki.ubuntu.com/vlan

See also : 
 * [Prevent traffic forwarding from one interface to anothers](../../iptable/doc/drop_forwarding_between_interfaces.md)
 * [Force reply on same interface than query](../../routing/doc/force_reply_on_same_interface.md)
 * [Incoming traffic from an interface wrongly outgoing to an another interface](../../routing/doc/wrong_traffic_forwarding_between_interfaces.md)
 
<a name="8021q-module"></a>
## 8021q module configuration
**8021q module is required**

### Check if already configured
Execute the following : 
```bash
$ grep 8021q /etc/modules
```
In case module is already configured, output will be : 
```bash
$ grep 8021q /etc/modules
8021q
$
```

### Configure module to be loaded at boot time

_This step must be skipped if module is already configured !_

```bash
echo "8021q" >> /etc/modules
```

<a name="vlan-interface"></a>
## Vlan interface configuration
### Naming
 - Vlan identifier : `$VID$`
 - Physical interface through which vlan network will transit : `$MIF$`
 - Vlan interface : `$VIF$` (usually `$VIF$ = "vlan$VID$"`)
 
### Conventions
* Configuration file will take place under `/etc/network/interfaces.d/` directory.
* Configuration file will be named with vlan interface vlan (`$VIF$`)

### Precondition
* `/etc/network/interfaces` contains instruction to load the configuration folder (e.g. `source-directory /etc/network/interfaces.d`)

<a name="vlan-interface-dhcp"></a>
### Vlan with DHCP ip assignation
```
#/etc/network/interfaces.d/$VIF$
auto $VIF$
iface $VIF$ inet dhcp
    vlan-raw-device $MIF$
```

<a name="vlan-interface-static"></a>
### Vlan with static ip assignation
```
#/etc/network/interfaces.d/$VIF$
auto $VIF$
iface $VIF$ inet static
    address 192.168.0.20   #<-- to update according to the you configuration
    netmask 255.255.255.0  #<-- to update according to the you configuration
    gateway 192.168.0.254  #<-- to update according to the you configuration
    vlan-raw-device $MIF$
```

## interface up & down

### Wake up 
```bash
$ ifup $VIF$
```
Will produce the following output (example taken from an interface configured with DHCP)
```
Set name-type for VLAN subsystem. Should be visible in /proc/net/vlan/config
Added VLAN with VID == $VID$ to IF -:$MIF$:-
Internet Systems Consortium DHCP Client 4.3.5
Copyright 2004-2016 Internet Systems Consortium.
All rights reserved.
For info, please visit https://www.isc.org/software/dhcp/

Listening on LPF/$VIF$/AA:BB:CC:DD:EE:FF
Sending on   LPF/$VIF$/AA:BB:CC:DD:EE:FF
Sending on   Socket/fallback
DHCPDISCOVER on $VIF$ to 255.255.255.255 port 67 interval 4
DHCPREQUEST of 192.168.$VID$.20 on $VIF$ to 255.255.255.255 port 67
DHCPOFFER of 192.168.$VID$.20 from 192.168.$VID$.254
DHCPACK of 192.168.$VID$.20 from 192.168.$VID$.254
bound to 192.168.$VID$.20 -- renewal in 271 seconds.
```

### Shutdown
```bash
$ ifdown $VIF$
```

## Routing
*Add the required `$VIF$` routing table (see [there](../../routing/doc/force_reply_on_same_interface.md#create-a-routing-table))*

In order to correctly route vlan traffic, create or update following scripts (and be sure they have execution flag) : 

```
#!/bin/sh
#/etc/network/if-up.d/vlan

case "$IFACE" in
  $VIF$)
    ip route add default via 192.168.$VID$.254 dev $VIF$ table $VIF$
    ip rule add from 192.168.$VID$.0/24 table $VIF$
  ;;
# Ignore Others
  *)
    exit 0
  ;;
esac

exit 0
```

```
#!/bin/sh
/etc/network/if-down.d/vlan 

case "$IFACE" in
  vlan690)
    ip route delete default via 192.168.$VID$.254 dev $VIF$ table $VIF$
    ip rule delete from 192.168.$VID$.0/24 table $VIF$
  ;;
# Ignore Others
  *)
    exit 0
  ;;
esac

exit 0
```

Think also about firewall in order to block traffic between servers interfaces.

See [Prevent traffic forwarding from one interface to anothers](../../iptable/doc/drop_forwarding_between_interfaces.md)

To block all forwarding between interfaces (that should be the case if server is not use as router) : 
```bash
# Block all forwarding traffic by default
iptables -P FORWARD DROP
```

## Issues

### Packing incoming from master interface wrongly outgoing to a vlan interface

Example : 

 * Default network :
   * network IP: `192.168.0.0/24`
   * gateway: `192.168.0.254`
 * Vlan60 network :
   * network IP: `192.168.60.0/24`
   * gateway: `192.168.60.254`
 * Server
   * eth0 interface is on default network (IP: `192.168.0.10`)
     * :warning: By default, your system will add a routing rule saying that all packet from `192.168.0.0/24` will go through **eth0 interface**
   * vlan60 interface is on vlan60 network (IP: `192.168.60.20`)
     * :warning: By default, your system will add a routing rule saying that all packet from `192.168.60.0/24` will go through **vlan60 interface**
 * Client PC is on vlan60 network (IP: `192.168.60.30`)

Let's say 
  * vlan60 interface on **Server** is only made to be used by avahi (see [there](../../avahi/doc/airplay_gateway_over_vlans.md)), and so not for ssh for instance.
  * but you also want to allow ssh from **Server** for device that are in **Vlan60** network. 
    
    Furthermore, by security (and because of first point), you want to manage those accesses from your router and so you allow ssh to **Server** only with it's default network IP (`192.168.0.10`)

In case you try a ssh connection from **Client PC** to **Server** with IP `192.168.0.10`, packets will transit through your router to eventually goes to **Server** and then going through **eth0 interface** (as `192.168.0.10` is the IP of that interface).

Unfortunately, due to default routing rule, and as **Client PC** IP is included in `192.168.60.0/24` network, outgoing packets will go through **vlan60 interface**. And then :boom:.

To prevent that and be sure a request incoming from **eth0 interface** will outgoing to **eth0 interface**, you must :
 * Add a routing table (see [there](../../routing/doc/force_reply_on_same_interface.md#create-a-routing-table)) named `lan` (for instance).
 * Add a default route to the same gateway than **eth0 interface** to `lan` routing table
 * Add a rule saying that all packet to **default network** IPs must go to `lan` routing table
 * Add a rule saying that all packet from **default network** IPs must go to `lan` routing table.
   
 Last rule is the most important one for our case, it will force any packet comming from **default network** to outgoing through **eth0** (due to the default route added to `lan` routing table)
 
For current example, it will looks like the following : 
```bash
# Add lan routing table (if not already made). 200 is a random number, choose it based on what it already exist
$ echo "200 lan" >> /etc/iproute2/rt_tables

# Add default route for the new routing table
$ ip route add default via 192.168.0.254 dev eth0 table lan

# Add rule for incoming packets
$ ip rule add to 192.168.0.0/24 table lan

# Add rule for outgoing packets.
$ ip rule add from 192.168.0.0/24 table lan
```

Unluckily, this does not solve the issue when you are on the server (e.g. want ping **Client PC** IP from **Server**).

As **Client PC** IP is included in `192.168.60.0/24` network, outgoing packets will go through **vlan60 interface**.

Rules regarding `lan` table will not be applied as packets don't come from `192.168.0.0/24`, they come from **Server** itsef.

A solution could be to add a rule saying that all packet must go through **eth0 interface**, but I cannot do else other interfaces will not be able to send traffic anymore, they will only be able to receive traffic (and so, avahi will be broken for instance).
