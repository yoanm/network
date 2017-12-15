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
See 

To block all forwarding between interfaces (that should be the case if server is not use as router) : 
```bash
# Block all forwarding traffic by default
iptables -P FORWARD DROP
```
