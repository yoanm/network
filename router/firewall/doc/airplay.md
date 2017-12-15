# Airplay firewall rules

Describe ip firewall rules required for airplay exchange between an AppleTv and Airplay consumers.
Commands are made for mikrotik RouterOs.

See also
  * [Airplay gateway over vlans](../../../avahi/doc/airplay_gateway_over_vlans.md)
  
## Ports
  
 * Airplay dynamic ports : UDP - 49152-65535
 * Airplay speaker : UDP - 6001, 6002
 * DAAP, Airplay and iTunes music sharing : TCP - 3689
 * Mirroring and Music : TCP - 5000
 * Picture and File : TCP - 7000
 * Video : TCP - 7001
 * Display Mirroring
   * TCP - 7100
   * UDP - 7010,7011
 * MDNS : UDP - 5353

## Address lists
Following firewall rules use address list as explained below. 
They can be replaced by specific IP/specific interface/interface list

 * `list-airplay-consumers` : Contains IPs of each airplay devices
 * `host-multimedia-appletv` :  Contains IP(s) of AppleTv device(s)

## Forward rules from AppleTv to Airplay consumers

### MDNS
Allow MDNS forwarding 
   ```
   add action=accept \
       chain=forward \
       comment="MDNS - Accept UDP from host-multimedia-appletv port 5353 to list-airplay-consumers port 5353" \
       protocol=udp \
       src-address-list=host-multimedia-appletv \
       src-port=5353 \
       dst-address-list=list-airplay-consumers \
       dst-port=5353 
   ```

### From dynamic ports

 * Allow UDP forwarding to dynamic ports
   ```
   add action=accept \
       chain=forward \
       comment="DYNAMIC PORTS - Accept UDP from host-multimedia-appletv port 49152-65535 to list-airplay-consumers port 49152-65535" \
       protocol=udp \
       src-address-list=host-multimedia-appletv \
       src-port=49152-65535 \
       dst-address-list=list-airplay-consumers \
       dst-port=49152-65535
   ```
 * Airplay speakers 
   
   *For use from Itunes.*

   Allow UDP forwarding from dynamic ports to port 6002
   ```
   add action=accept \
       chain=forward \
       comment="AIRPLAY SPEAKERS - Accept UDP from host-multimedia-appletv port 49152-65535 to list-airplay-consumers port 6002" \
       protocol=udp \
       src-address-list=host-multimedia-appletv \
       src-port=49152-65535 \
       dst-address-list=list-airplay-consumers \
       dst-port=6002
   ```

## Forward rules from Airplay consumer to AppleTv

### MDNS
Allow MDNS forwarding 
```
add action=accept \
    chain=forward \
    comment="MDNS - Accept UDP from list-airplay-consumers port 5353 to host-multimedia-appletv port 5353" \
    protocol=udp \
    src-address-list=list-airplay-consumers \
    src-port=5353 \
    dst-address-list=host-multimedia-appletv \
    dst-port=5353
```

### From dynamic ports

 * Allow forwarding to dynamic ports
   * TCP
   ```
   add action=accept \
       chain=forward \
       comment="DYNAMIC PORTS - Accept TCP from list-airplay-consumers port 49152-65535 to host-multimedia-appletv port 49152-65535" \
       protocol=tcp \
       src-address-list=list-airplay-consumers \
       src-port=49152-65535 \
       dst-address-list=host-multimedia-appletv \
       dst-port=49152-65535
   ```
   * UDP
   ```
   add action=accept \
       chain=forward \
       comment="DYNAMIC PORTS - Accept UDP from list-airplay-consumers port 49152-65535 to host-multimedia-appletv port 49152-65535" \
       protocol=udp \
       src-address-list=list-airplay-consumers \
       src-port=49152-65535 \
       dst-address-list=host-multimedia-appletv \
       dst-port=49152-65535
   ```
 * DAAP, Airplay and iTunes music sharing

   Allow TCP forwarding from dynamic ports to port 3689
   ```
   add action=accept \
       chain=forward \
       comment="DAAP, AIRPLAY and iTunes music sharing - Accept TCP from list-airplay-consumers port 49152-65535 to host-multimedia-appletv port 3689" \
       protocol=tcp \
       src-address-list=list-airplay-consumers \
       src-port=49152-65535 \
       dst-address-list=host-multimedia-appletv \
       dst-port=3689
   ```
 * Mirroring and Music

   Allow TCP forwarding to port 5000
   ```
   add action=accept \
       chain=forward \
       comment="MIRRORING AND MUSIC - Accept TCP from list-airplay-consumers port 49152-65535 to host-multimedia-appletv port 5000" \
       protocol=tcp \
       src-address-list=list-airplay-consumers \
       src-port=49152-65535 \
       dst-address-list=host-multimedia-appletv \
       dst-port=5000
   ```
 * Picture and File

   Allow TCP forwarding to port 7000
   ```
   add action=accept \
       chain=forward \
       comment="PICTURE AND FILE - Accept TCP from list-airplay-consumers port 49152-65535 to host-multimedia-appletv port 7000" \
       protocol=tcp \
       src-address-list=list-airplay-consumers \
       src-port=49152-65535 \
       dst-address-list=host-multimedia-appletv \
       dst-port=7000
   ```
 * Video

   Allow TCP forwarding to port 7001
   ```
   add action=accept \
       chain=forward \
       comment="VIDEO - Accept TCP from list-airplay-consumers port 49152-65535 to host-multimedia-appletv port 7001" \
       protocol=tcp \
       src-address-list=list-airplay-consumers \
       src-port=49152-65535 \
       dst-address-list=host-multimedia-appletv \
       dst-port=7001
   ```
 * Display Mirroring

   * Allow TCP forwarding to port 7100
   ```
   add action=accept \
       chain=forward-from-airplay-consumers \
       comment="DISPLAY MIRRORING -  Accept TCP from list-airplay-consumers port 49152-65535 to host-multimedia-appletv port 7100" \
       protocol=tcp \
       src-address-list=list-airplay-consumers \
       src-port=49152-65535 \
       dst-address-list=host-multimedia-appletv \
       dst-port=7100
   ```
   * Allow UDP forwarding to port 7010,7011
   ```
   add action=accept \
       chain=forward-from-airplay-consumers \
       comment="DISPLAY MIRRORING -  Accept UDP from list-airplay-consumers port 49152-65535 to host-multimedia-appletv port 7010-7011" \
       protocol=udp \
       src-address-list=list-airplay-consumers \
       src-port=49152-65535 \
       dst-address-list=host-multimedia-appletv \
       dst-port=7010-7011
   ```
### To dynamic ports

 * Airplay speakers

   *For use from Itunes.*

   Allow UDP forwarding from port 6001
   ```
   add action=accept \
       chain=forward \
       comment="AIRPLAY SPEAKERS - Accept UDP from list-airplay-consumers port 6001 to host-multimedia-appletv port 49152-65535"
       protocol=udp \
       src-address-list=list-airplay-consumers \
       src-port=6001 \
       dst-address-list=host-multimedia-appletv \
       dst-port=49152-65535 \
   ```
