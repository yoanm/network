# Force reply on same interface than query


Sources : 
 * https://unix.stackexchange.com/questions/4420/reply-on-same-interface-as-incoming
 * https://superuser.com/questions/325128/how-can-i-ensure-outbound-traffic-uses-the-same-interface-as-that-of-inbound-tra
 
See also : 
 * [Prevent traffic forwarding from one interface to anothers](../../iptable/doc/drop_forwarding_between_interfaces.md)
 
## Stategy

Create a routing table for each interface


## Create a routing table 
Routing table configuration file : `/etc/iproute2/rt_tables`

For instance to add a routing table named `vlan200` with the identifier `10`, add the following at end of configuration file :
```
10 vlan200
```
**Each routing table must have a uniq identifier**

Repeat this step for each interfaces you want to isolate

### Bind to a routing table

First , add the gateway for the routing table
```bash
$ ip route add default via <gateway ip> dev <interface> table <routing table name>
```

Then bind with one (or a mixed) of following solutions :

#### From an specific ip network :
```bash
$ ip route add from <ip network for the interface> table <routing table name>
```

#### From a specific interface
```bash
$ ip route add dev <interface> table <routing table name>
```

#### From a specific ip
```bash
$ ip route add src <ip> table <routing table name>
```
