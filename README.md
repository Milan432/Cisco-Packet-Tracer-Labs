Coffee Shop Network Lab (Cisco Packet Tracer)

A small business network design built in Cisco Packet Tracer as part of my CompTIA Network+ studies. The goal was to design a realistic, segmented network for a coffee shop — separating management, point-of-sale, and guest wifi traffic.

Topology Overview

The network is split into four VLANs:

VLAN	Purpose	Subnet
10	Office (manager's PC, printer)	192.168.10.0/24
20	POS	192.168.20.0/24
30	Guest Wifi	192.168.30.0/24
99	Network Management (switches, APs)	192.168.99.0/24


Design Decisions

Why segment POS traffic? Real-world coffee shops (and most small businesses handling card payments) need to keep POS systems isolated from public wifi — this is a PCI DSS requirement, not just a nice-to-have. In this lab, the POS VLAN is kept completely separate from the guest network.

Guest network restrictions Guest wifi (VLAN 30) can reach the internet but is explicitly blocked from accessing the Office network (VLAN 10), POS network (VLAN 20), and the Network Management VLAN (VLAN 99), using an extended named ACL applied on the multilayer switch.

DHCP per VLAN Each VLAN has its own DHCP scope, with the first 20 addresses in each subnet excluded from dynamic assignment — reserved for static IPs (gateways, switches, APs, and any other infrastructure that needs a fixed address).

Key Configuration
Extended ACL — Guest Network Isolation
ip access-list extended GUEST_RESTRICTION
 10 permit udp any eq bootpc any eq bootps
 20 deny ip 192.168.30.0 0.0.0.255 192.168.10.0 0.0.0.255
 30 deny ip 192.168.30.0 0.0.0.255 192.168.20.0 0.0.0.255
 50 deny ip 192.168.30.0 0.0.0.255 192.168.99.0 0.0.0.255
 60 permit ip 192.168.30.0 0.0.0.255 any

Line 10 ensures DHCP traffic is always allowed through before the deny statements take effect. Lines 20 and 30 block guest access to internal networks, while line 60 still permits general internet access.

DHCP Exclusions
ip dhcp excluded-address 192.168.10.1 192.168.10.20
ip dhcp excluded-address 192.168.20.1 192.168.20.20
ip dhcp excluded-address 192.168.30.1 192.168.30.20

(VLAN 99 devices — switches and APs — are assigned static IPs rather than pulling from DHCP, since network infrastructure typically shouldn't rely on a dynamic address.)

Switch Port Security (Access Ports)
interface fa0/1
 spanning-tree portfast
 spanning-tree bpduguard enable

Applied on access ports connecting to end devices to speed up port activation while protecting against accidental loops from unauthorized switches.
