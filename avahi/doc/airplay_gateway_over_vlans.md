# Airplay gateway over vlans

Document describe how to set up a MNDS gateway between two vlans. It use airplay service from an appleTv as example, but MDNS gateway works for any MDNS services.

**Disclaimer**

 * Unfortunately, i didn't found any easy solutions to filter MNDS services (e.g. filter only airplay or printer discovery).
 * This document is only about airplay discovery, for airplay exchanges see [this document](../../router/firewall/doc/airplay.md)

Sources : 
  * https://www.packetmischief.ca/2012/09/20/airplay-vlans-and-an-open-source-solution/
  * https://linux.die.net/man/5/avahi-daemon.conf
  
See also :
  * [How to add a permanent vlan interface on linux](../../vlan/doc/permanent_vlan_if.md)

Naming :
  * **"Avahi vlan interface"**, aka **AVAHI_VLAN_GATEWAY**
    
    The vlan interface that will be used by appleTv device to be registered on avahi

    * `$AVAHI_VID$` refers to the vlanId of **AVAHI_VLAN_GATEWAY**
    * `$AVAHI_VIF$` refers to the **AVAHI_VLAN_GATEWAY** interface itself
    * `$AVAHI_VIF_IP$` refers to the **AVAHI_VLAN_GATEWAY** interface ip
    
  * **"Airplay vlan interface"**, aka **AIRPLAY_VLAN_GATEWAY**
    
    The vlan interface that will be used by Airplay consumers devices to query or be notified by avahi

    * `$AIRPLAY_VID$` refers to the vlanId of **AIRPLAY_VLAN_GATEWAY**
    * `$AIRPLAY_VIF$` refers to the **AIRPLAY_VLAN_GATEWAY** interface itself
    * `$AIRPLAY_VIF_IP$` refers to the **AIRPLAY_VLAN_GATEWAY** interface ip
    
  * Airplay consumers devices
    
    Refers to any devices with airplay capability (usually apple devices like iphone/ipad/(i)mac/etc)
    
    * `$AIRPLAY_CONS_IP$` refers to IP of a specific airplay consumers device
    * `$AIRPLAY_CONS_IF$` refers to the interface of a specific airplay consumers device from router point of view (not server POV)

  * AppleTv device
    
    Refers to ... AppleTv device, quite logic :)
    
    * `$APPLETV_IP$` refers to IP of AppleTv device
    * `$APPLETV_IF$` refers to the interface of AppleTv device from router point of view (not server POV)

## Preconditions

 * An **AVAHI_VLAN_GATEWAY** must be configured on the server
 * An **AIRPLAY_VLAN_GATEWAY** must be configured on the server
 * If bridge are used in a the router :

   Vlan ids are not required to be the same between **AVAHI_VLAN_GATEWAY** and AppleTv nor **AIRPLAY_VLAN_GATEWAY** and Airplay consumers devices.
    
   But
    * **AVAHI_VLAN_GATEWAY** must be member of the same bridge than AppleTv device
    * **AIRPLAY_VLAN_GATEWAY** must be member of the same bridge than Airplay consumers devices
 * If no bridge are used
   * **AVAHI_VLAN_GATEWAY** must be in the same vlan than the appleTv device
   * **AIRPLAY_VLAN_GATEWAY** must be in the same vlan than Airplay consumers devices
  
## Install & configure avahi daemon

### Install
```bash
sudo apt-get install avahi-daemon
```

### Configure
Then edit `/etc/avahi/avahi-daemon.conf` and update following values:
```
[server]
allow-interfaces=$AVAHI_VIF$,$AIRPLAY_VIF$

[reflector]
enable-reflector=yes
```

#### Optional
Use only IpV4 (in case is configured to drop ipv6 traffic for instance)
```
[server]
use-ipv4=yes
use-ipv6=no
```

Finally, restart avahi daemon : 
```bash
$ sudo systemctl restart avahi-daemon
```
In avahi logs, you should see something similar to : 
```
XXXXXXX XXX avahi-daemon[123456]: avahi-daemon 0.6.32 starting up.
XXXXXXX XXX avahi-daemon[123456]: Successfully called chroot().
XXXXXXX XXX avahi-daemon[123456]: Successfully dropped remaining capabilities.
XXXXXXX XXX avahi-daemon[123456]: No service file found in /etc/avahi/services.
XXXXXXX XXX avahi-daemon[123456]: Joining mDNS multicast group on interface $AVAHI_VIF$.IPv4 with address $AVAHI_VIF_IP$.
XXXXXXX XXX avahi-daemon[123456]: New relevant interface $AVAHI_VIF$.IPv4 for mDNS.
XXXXXXX XXX avahi-daemon[123456]: Joining mDNS multicast group on interface $AIRPLAY_VIF$.IPv4 with address $AIRPLAY_VIF_IP$.
XXXXXXX XXX avahi-daemon[123456]: New relevant interface $AIRPLAY_VIF$.IPv4 for mDNS.
XXXXXXX XXX avahi-daemon[123456]: Network interface enumeration completed.
XXXXXXX XXX avahi-daemon[123456]: Registering new address record for AAAA::BBBB:CCCC:DDDD:EEEE on $AVAHI_VIF$.*.
XXXXXXX XXX avahi-daemon[123456]: Registering new address record for $AVAHI_VIF_IP$ on $AVAHI_VIF$.IPv4.
XXXXXXX XXX avahi-daemon[123456]: Registering new address record for $AIRPLAY_VIF_IP$ on $AIRPLAY_VIF$.IPv4.
XXXXXXX XXX avahi-daemon[123456]: Server startup complete. Host name is XXX.local. Local service cookie is 677078695.
```

### Logging
By default avahi log are on syslog log file.
```bash
$ tail -f /var/log/syslog | grep avahi
```


## Monitoring / Debugging
Install `avahi-discover` packet :
```bash
$ sudo apt-get install avahi-discover
```

And then to follow discovering
```bash
$ avahi-browse -a -k
```

 * If you want to just list current entries, add `-c`
 * If you want to resolve entries, add `-r`
 
 
## Firewall rules
In case bridges are used, following traffic must be authorized in bridge filters :
 * ARP to FF:FF:FF:FF:FF:FF mac address
   * from `$AVAHI_VIF$` to `$APPLETV_IF$` *(Avahi gateway => AppleTv)*
   * from  `$APPLETV_IF$` to `$AVAHI_VIF$` *(AppleTv => Avahi gateway)*
   * from `$AIRPLAY_VIF$` to each `$AIRPLAY_CONS_IF$` *(Airplay gateway => Airplay consumers)*
   * from  each `$AIRPLAY_CONS_IF$` to `$AIRPLAY_VIF$` *(Airplay consumers => Airplay gateway)*

Following traffic must be authorized in ip firewall
 * MDNS (UDP) traffic  from `*:5353` to `224.0.0.251:5353`
   * from `$AVAHI_VIF$` to `$APPLETV_IF$` *(Avahi gateway => AppleTv)*
   * from `$APPLETV_IF$` to `$AVAHI_VIF$` *(AppleTv => Avahi gateway)*
   * from `$AIRPLAY_VIF$` to each `$AIRPLAY_CONS_IF$` *(Airplay gateway => Airplay consumers)*
   * from each `$AIRPLAY_CONS_IF$` to `$AIRPLAY_VIF$` *(Airplay consumers => Airplay gateway)*
 * IGMP traffic to `224.0.0.22`
   * from `$AVAHI_VIF$` to `$APPLETV_IF$` *(Avahi gateway => AppleTv)*
   * from `$APPLETV_IF$` to `$AVAHI_VIF$` *(AppleTv => Avahi gateway)*
   * from `$AIRPLAY_VIF$` to each `$AIRPLAY_CONS_IF$` *(Airplay gateway => Airplay consumers)*
   * from each `$AIRPLAY_CONS_IF$` to `$AIRPLAY_VIF$` *(Airplay consumers => Airplay gateway)*
 
 ## Known issues
 
  * When appleTv is restarted, a counter is bumping just after the name
