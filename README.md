# cldemo-vxlan

This uses reference topology.

Server01 and Server02 are single attached to leaf01

Server03 and Server04 are single attached to leaf03

Spine01 is the Registration node.

Server01 and Server03 are connected to access ports in VLAN 13

Server02 and Server04 are connected to access ports in VLAN 24

VNI13 and VNI24 are configured (incorrectly) as trunks, carrying vlan 13 and vlan 24 between leaf01 and leaf03.

When any server sends a BUM packet, after the VxLAN decap on the remote leaf, the packet will be flooded back in the other VNI carrying the vlan. This causes a bridging loop.

In a tcpdump on the fabric port of leaf03 you can see the packets being sent in both VNIs

```
cumulus@leaf03:~$ sudo tcpdump -i swp51
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on swp51, link-type EN10MB (Ethernet), capture size 262144 bytes
22:55:39.002631 IP 10.1.1.1.41698 > 10.3.3.3.4789: VXLAN, flags [I] (0x08), vni 24
ARP, Request who-has 10.1.3.103 tell 10.1.3.101, length 46
22:55:39.002691 IP 10.3.3.3.41698 > 10.1.1.1.4789: VXLAN, flags [I] (0x08), vni 13
ARP, Request who-has 10.1.3.103 tell 10.1.3.101, length 46
22:55:39.002839 IP 10.1.1.1.51346 > 10.3.3.3.4789: VXLAN, flags [I] (0x08), vni 13
ARP, Reply 10.1.3.103 is-at 44:38:39:00:00:28 (oui Unknown), length 46
22:55:39.002856 IP 10.3.3.3.51346 > 10.1.1.1.4789: VXLAN, flags [I] (0x08), vni 24
ARP, Reply 10.1.3.103 is-at 44:38:39:00:00:28 (oui Unknown), length 46
22:55:39.003018 IP 10.1.1.1.51346 > 10.3.3.3.4789: VXLAN, flags [I] (0x08), vni 13
ARP, Reply 10.1.3.103 is-at 44:38:39:00:00:28 (oui Unknown), length 46
22:55:39.003030 IP 10.3.3.3.51346 > 10.1.1.1.4789: VXLAN, flags [I] (0x08), vni 24
ARP, Reply 10.1.3.103 is-at 44:38:39:00:00:28 (oui Unknown), length 46
22:55:39.003132 IP 10.3.3.3.51346 > 10.1.1.1.4789: VXLAN, flags [I] (0x08), vni 24
ARP, Reply 10.1.3.103 is-at 44:38:39:00:00:28 (oui Unknown), length 46
22:55:39.003188 IP 10.1.1.1.51346 > 10.3.3.3.4789: VXLAN, flags [I] (0x08), vni 13
ARP, Reply 10.1.3.103 is-at 44:38:39:00:00:28 (oui Unknown), length 46
22:55:39.003201 IP 10.3.3.3.51346 > 10.1.1.1.4789: VXLAN, flags [I] (0x08), vni 24
ARP, Reply 10.1.3.103 is-at 44:38:39:00:00:28 (oui Unknown), length 46
22:55:39.003371 IP 10.1.1.1.51346 > 10.3.3.3.4789: VXLAN, flags [I] (0x08), vni 13
ARP, Reply 10.1.3.103 is-at 44:38:39:00:00:28 (oui Unknown), length 46
```
