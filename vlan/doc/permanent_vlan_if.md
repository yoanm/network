# How to add a permanent vlan interface on linux

 * [8021q module configuration](#8021q-module)
 * [Vlan interface configuration](#vlan-interface)
   * [Vlan with DHCP ip assignation](#vlan-interface-dhcp)
   * [Vlan with static ip assignation](#vlan-interface-static)

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
