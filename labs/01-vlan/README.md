# Configure Router-on-a-Stick Inter-VLAN Routing

![](scheme.png)

## VPC1 configuration
```
VPCS> ip 192.168.3.3/24 192.168.3.1
Checking for duplicate address...
PC1 : 192.168.3.3 255.255.255.0 gateway 192.168.3.1
VPCS> save vpc1
```

## VPC2 configuration
```
VPCS> ip 192.168.4.3/24 192.168.4.1
Checking for duplicate address...
PC1 : 192.168.4.3 255.255.255.0 gateway 192.168.4.1
VPCS> save vpc2
```

## R1 configuration
```
Router>enable
Router#conf t
Router#hostname r1
r1(config)#enable password class
r1(config)#end
r1#conf t
r1(config)#line vty 0
r1(config-line)#login
r1(config-line)#password cisco
r1(config-line)#end
r1#conf t
r1(config)#line console 0
r1(config-line)#logging synchronous
r1(config-line)#password cisco
r1(config-line)#motd-banner !Unathorized access is prohibited!
r1(config-line)#end
r1#conf t
r1(config)#service password-encryption
r1(config)#no ip domain lookup
r1(config)#clock timezone GMT +3
r1(config)#end
r1#clock set 8:00:00 Aug 27 2021
r1#copy running-config startup-config
r1#show clock

r1#conf t
r1(config)#interface e0/0
r1(config-if)#Description Trunk link to SW1
r1(config-if)#no ip address
r1(config-if)#no shut
r1(config-if)#interface e0/0.8
r1(config-subif)#encapsulation dot1q 8 native
r1(config-subif)#no ip address
r1(config-subif)#exit
r1(config)#interface e0/0.3
r1(config-subif)#Description Default Gateway for VLAN 3
r1(config-subif)#encapsulation dot1q 3
r1(config-subif)#ip add 192.168.3.1 255.255.255.0
r1(config-subif)#exit
r1(config)#interface e0/0.4
r1(config-subif)#Description Default Gateway for VLAN 4
r1(config-subif)#encapsulation dot1q 4
r1(config-subif)#ip add 192.168.4.1 255.255.255.0
r1(config-subif)#exit
r1(config)#interface e0/0.7
r1(config-subif)#encapsulation dot1q 7
r1(config-subif)#no ip address
r1(config-subif)#end
r1#copy running-config startup-config
```
```
r1#show ip interface brief
Interface                  IP-Address      OK? Method Status                Prot                  ocol
Ethernet0/0                unassigned      YES unset  up                    up                    
Ethernet0/0.3              192.168.3.1     YES manual up                    up                    
Ethernet0/0.4              192.168.4.1     YES manual up                    up                    
Ethernet0/0.7              unassigned      YES unset  up                    up                    
Ethernet0/0.8              unassigned      YES unset  up                    up                    
Ethernet0/1                unassigned      YES unset  administratively down down                  
Ethernet0/2                unassigned      YES unset  administratively down down                  
Ethernet0/3                unassigned      YES unset  administratively down down                  
Serial1/0                  unassigned      YES unset  administratively down down                  
Serial1/1                  unassigned      YES unset  administratively down down                  
Serial1/2                  unassigned      YES unset  administratively down down                  
Serial1/3                  unassigned      YES unset  administratively down down

```

## SW1
### SW1 VLAN config
```
Switch>enable
Switch#conf t
Switch#hostname sw1
sw1(config)#enable password class
sw1(config)#end
sw1#conf t
sw1(config)#line vty 0
sw1(config-line)#login
sw1(config-line)#password cisco
sw1(config-line)#end
sw1#conf t
sw1(config)#line console 0
sw1(config-line)#logging synchronous
sw1(config-line)#password cisco
sw1(config-line)#motd-banner !Unathorized access is prohibited!
sw1(config-line#end
sw1#conf t
sw1(config)#service password-encryption
sw1(config)#no ip domain lookup
sw1(config)#clock timezone GMT +3
sw1(config)#end
sw1#clock set 8:00:00 Aug 27 2021
sw1#copy running-config startup-config
sw1#show clock
sw1#copy running-config startup-config

sw1(config)#ip default-gateway 192.168.3.1
sw1(config)#interface vlan 3
sw1(config-if)#ip address 192.168.3.11 255.255.255.0
sw1(config-if)#no shut
sw1(config-if)#end
sw1#conf t
sw1(config)#interface e0/2
sw1(config-if)#Switchport mode access
sw1(config-if)#Switchport access vlan 3
sw1(config-if)#no shut
sw1(config-if)#end
sw1(config)#interface e0/3
sw1(config-if)#Switchport mode access
sw1(config-if)#Switchport access vlan 7
sw1(config-if)#shutdown
sw1(config-if)#end
sw1#copy running-config startup-config
```
### SW1 trunk to SW2
```
sw1#conf t
sw1(config)#interface e0/1
sw1(config-if)#Switchport trunk encapsulation dot1q
sw1(config-if)#Switchport mode trunk
sw1(config-if)#Switchport trunk native vlan 8
sw1(config-if)#Switchport trunk allowed vlan 3,4,8
sw1(config-if)#no shut
sw1(config-if)#end
```
### SW1 trunk to R1
```
sw1#conf t
sw1(config)#interface e0/0
sw1(config-if)#Switchport trunk encapsulation dot1q
sw1(config-if)#Switchport mode trunk
sw1(config-if)#Switchport trunk native vlan 8
sw1(config-if)#Switchport trunk allowed vlan 3,4,8
sw1(config-if)#no shut
sw1(config-if)#end
```
```
sw1#show interfaces trunk

Port        Mode             Encapsulation  Status        Native vlan
Et0/0       on               802.1q         trunking      8
Et0/1       on               802.1q         trunking      8

Port        Vlans allowed on trunk
Et0/0       3-4,8
Et0/1       3-4,8

Port        Vlans allowed and active in management domain
Et0/0       3
Et0/1       3

Port        Vlans in spanning tree forwarding state and not pruned
Et0/0       3
Et0/1       3
```

```
sw1#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active
3    VLAN0003                         active    Et0/2
7    VLAN0007                         active    Et0/3
1002 fddi-default                     act/unsup
1003 token-ring-default               act/unsup
1004 fddinet-default                  act/unsup
1005 trnet-default                    act/unsup
```

```
sw1#show interface e0/1 switchport
Name: Et0/1
Switchport: Enabled
Administrative Mode: trunk
Operational Mode: trunk
Administrative Trunking Encapsulation: dot1q
Operational Trunking Encapsulation: dot1q
Negotiation of Trunking: On
Access Mode VLAN: 1 (default)
Trunking Native Mode VLAN: 8 (Inactive)
Administrative Native VLAN tagging: enabled
Voice VLAN: none
Administrative private-vlan host-association: none
Administrative private-vlan mapping: none
Administrative private-vlan trunk native VLAN: none
Administrative private-vlan trunk Native VLAN tagging: enabled
Administrative private-vlan trunk encapsulation: dot1q
Administrative private-vlan trunk normal VLANs: none
Administrative private-vlan trunk associations: none
Administrative private-vlan trunk mappings: none
Operational private-vlan: none
Trunking VLANs Enabled: 3,4,8
Pruning VLANs Enabled: 2-1001
Capture Mode Disabled
Capture VLANs Allowed: ALL

Appliance trust: none
```
```
sw1#show interface e0/0 switchport
Name: Et0/0
Switchport: Enabled
Administrative Mode: trunk
Operational Mode: trunk
Administrative Trunking Encapsulation: dot1q
Operational Trunking Encapsulation: dot1q
Negotiation of Trunking: On
Access Mode VLAN: 1 (default)
Trunking Native Mode VLAN: 8 (Inactive)
Administrative Native VLAN tagging: enabled
Voice VLAN: none
Administrative private-vlan host-association: none
Administrative private-vlan mapping: none
Administrative private-vlan trunk native VLAN: none
Administrative private-vlan trunk Native VLAN tagging: enabled
Administrative private-vlan trunk encapsulation: dot1q
Administrative private-vlan trunk normal VLANs: none
Administrative private-vlan trunk associations: none
Administrative private-vlan trunk mappings: none
Operational private-vlan: none
Trunking VLANs Enabled: 3,4,8
Pruning VLANs Enabled: 2-1001
Capture Mode Disabled
Capture VLANs Allowed: ALL

Appliance trust: none

```
## SW2
### SW2 VLAN config
```
Switch>enable
Switch#conf t
Switch#hostname sw2
sw2(config)#enable password class
sw2(config)#end
sw2#conf t
sw2(config)#line vty 0
sw2(config-line)#login
sw2(config-line)#password cisco
sw2(config-line)#end
sw2#conf t
sw2(config)#line console 0
sw2(config-line)#logging synchronous
sw2(config-line)#password cisco
sw2(config-line)#motd-banner !Unathorized access is prohibited!
sw2(config-line#end
sw2#conf t
sw2(config)#service password-encryption
sw2(config)#no ip domain lookup
sw2(config)#clock timezone GMT +3
sw2(config)#end
sw2#clock set 8:00:00 Aug 27 2021
sw2#copy running-config startup-config
sw2#show clock
sw2#copy running-config startup-config

sw2(config)#ip default-gateway 192.168.3.1
sw2(config)#interface vlan 3
sw2(config-if)#ip address 192.168.3.12 255.255.255.0
sw2(config-if)#no shut
sw2(config-if)#end
sw2#conf t
sw2(config)#interface e0/2
sw2(config-if)#Switchport mode access
sw2(config-if)#Switchport access vlan 4
sw2(config-if)#no shut
sw2(config-if)#end
sw2(config)#interface e0/0
sw2(config-if)#Switchport mode access
sw2(config-if)#Switchport access vlan 7
sw2(config-if)#shutdown
sw2(config-if)#end
sw2(config)#interface e0/3
sw2(config-if)#Switchport mode access
sw2(config-if)#Switchport access vlan 7
sw2(config-if)#shutdown
sw2(config-if)#end
sw2#copy running-config startup-config
```
```
sw2#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active
4    VLAN0004                         active    Et0/2
7    VLAN0007                         active    Et0/0, Et0/3
1002 fddi-default                     act/unsup
1003 token-ring-default               act/unsup
1004 fddinet-default                  act/unsup
1005 trnet-default                    act/unsup

```

### SW2 trunk to SW1
```
sw2#conf t
sw2(config)#interface e0/1
sw2(config-if)#Switchport trunk encapsulation dot1q
sw2(config-if)#Switchport mode trunk
sw2(config-if)#Switchport trunk native vlan 8
sw2(config-if)#Switchport trunk allowed vlan 3,4,8
sw2(config-if)#no shut
sw2(config-if)#end
```
```
sw2#show interface trunk

Port        Mode             Encapsulation  Status        Native vlan
Et0/1       on               802.1q         trunking      8

Port        Vlans allowed on trunk
Et0/1       3-4,8

Port        Vlans allowed and active in management domain
Et0/1       4

Port        Vlans in spanning tree forwarding state and not pruned
Et0/1       4

```

```
sw2#show interface e0/1 switchport
Name: Et0/1
Switchport: Enabled
Administrative Mode: trunk
Operational Mode: trunk
Administrative Trunking Encapsulation: dot1q
Operational Trunking Encapsulation: dot1q
Negotiation of Trunking: On
Access Mode VLAN: 1 (default)
Trunking Native Mode VLAN: 8 (Inactive)
Administrative Native VLAN tagging: enabled
Voice VLAN: none
Administrative private-vlan host-association: none
Administrative private-vlan mapping: none
Administrative private-vlan trunk native VLAN: none
Administrative private-vlan trunk Native VLAN tagging: enabled
Administrative private-vlan trunk encapsulation: dot1q
Administrative private-vlan trunk normal VLANs: none
Administrative private-vlan trunk associations: none
Administrative private-vlan trunk mappings: none
Operational private-vlan: none
Trunking VLANs Enabled: 3,4,8
Pruning VLANs Enabled: 2-1001
Capture Mode Disabled
Capture VLANs Allowed: ALL

Appliance trust: none

```

## Ping test from PC1 to R1 gateway
```
VPCS> ping 192.168.3.1

84 bytes from 192.168.3.1 icmp_seq=1 ttl=255 time=1.881 ms
84 bytes from 192.168.3.1 icmp_seq=2 ttl=255 time=2.373 ms
```

## Ping test from PC2 to R1 gateway
```
VPCS> ping 192.168.4.1

84 bytes from 192.168.4.1 icmp_seq=1 ttl=255 time=1.564 ms
84 bytes from 192.168.4.1 icmp_seq=2 ttl=255 time=1.317 ms

```
## Ping test from PC1 to PC2
```
VPCS> ping 192.168.4.3

84 bytes from 192.168.4.3 icmp_seq=1 ttl=255 time=1.951 ms
84 bytes from 192.168.4.3 icmp_seq=2 ttl=255 time=2.593 ms
```
