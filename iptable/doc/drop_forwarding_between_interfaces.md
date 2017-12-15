# Prevent traffic forwarding from one interface to anothers

See also : 
 * [Prevent traffic forwarding from one interface to anothers](../../iptable/doc/drop_forwarding_between_interfaces.md)
 * [Force reply on same interface than query](../../routing/doc/force_reply_on_same_interface.md)

## Strategy
Drop all forwarding traffic by default and then allow only ones you want

## Drop all forwarding by default
```bash
iptables -P FORWARD DROP
```

## Allow forwarding for specific ip on a specific interface

**:warning: Except if system is used as router/gateway/..., you should not need to allow forwarding !**

Allow only `192.168.0.X` ip to be forwarded on `eth1` :
```bash
$ iptables -A FORWARD -i eth0 -o eth1 -s 192.168.0.0/24 -j ACCEPT
```
