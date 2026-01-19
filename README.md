Cisco Routing & Switching Practice Lab
Packet Tracer – Static & Dynamic Routing, VLAN, VTP, Inter-VLAN, OSPF

Ce dépôt contient une série de laboratoires Cisco Packet Tracer illustrant différents concepts de routage et commutation utilisés dans les réseaux d’entreprise.
Il s’agit d’exercices pratiques visant à renforcer la compréhension des protocoles, de la segmentation réseau et de la configuration des équipements Cisco.

🚀 Objectifs du projet
Mettre en pratique les concepts fondamentaux du routage statique et dynamique

Configurer des VLAN, du trunking, du VTP et du routage inter-VLAN

Déployer et tester une topologie OSPF multi‑accès

Résoudre des problèmes courants (DHCP, routage, segmentation)

Développer une approche structurée de la configuration Cisco

📁 Contenu du dépôt
Fichier	Description
staticRoute.pkt	Configuration complète du routage statique + résumé des routes
ospf multiaccess segment.pkt	Topologie OSPF sur segment multi‑accès
reseauJFtech.pkt	Réseau complet incluant DHCP, routage et corrections
trunk and access mode.pkt	Configuration des ports en mode trunk et access
vlan_trunking_protocol_interVlan.pkt	VTP + routage inter‑VLAN via switch L3
🛠️ Compétences techniques démontrées
Configuration Cisco IOS

Routage statique

Routage dynamique (OSPF)

VLAN & segmentation réseau

VTP (VLAN Trunking Protocol)

Trunking 802.1Q

Inter-VLAN routing

DHCP troubleshooting

Analyse et validation de topologies réseau

📚 Technologies utilisées
Cisco Packet Tracer

Cisco IOS CLI

Concepts CCNA (routage, switching, OSPF, VLAN)

📌 Comment utiliser les fichiers
Télécharger les fichiers .pkt

Ouvrir avec Cisco Packet Tracer

Explorer la topologie

Consulter les configurations via le CLI

Tester la connectivité (ping, traceroute, show commands)

*****fichier reseauJFtech*****
Network Project — Multi‑Router Hierarchical Architecture (Cisco Packet Tracer)
📌 1. Overview
This project simulates a multi‑router hierarchical enterprise network using Cisco Packet Tracer.
The design follows a 3‑layer architecture:

Core Layer: Routers R, R1, R2

Distribution Layer: Core switches (Core1, Core2)

Access Layer: Access switches with DHCP Snooping

Security Layer: ASA 5505 firewall (inside, outside, DMZ)

End Devices: PCs, servers, and test hosts

VLAN Subnetting
VLAN	Subnet
VLAN 10	10.10.10.0/24
VLAN 20	10.10.20.0/24
VLAN 30	10.10.30.0/27
VLAN 40	10.10.40.0/28
🚦 2. Router R — Central Services Router
Originally acted as the DHCP server for all VLANs.
Now serves as:

NAT boundary (private → public translation)

Routing gateway between internal networks and the upstream provider

DHCP relay using ip helper-address

Key Features
NAT pool: 203.0.113.3–203.0.113.6

ACL vlanNat permitting internal VLANs

PAT: ip nat inside source list vlanNat pool vlan overload

Dual default routes (ASA + provider)

DHCP relay to future external server: ip helper-address 172.16.0.1

🔁 3. Routers R1 & R2 — Inter‑VLAN Routing + HSRP
R1 and R2 provide:

Inter‑VLAN routing via subinterfaces

HSRP redundancy for gateway failover

Load distribution

DHCP relay to Router R

HSRP Roles
VLAN	Active Router
10	R1
20	R2
30	R1
40	R2
Each VLAN uses a virtual gateway ending in .1.

🔌 4. Switch Layer
Core Switches
Connect R1 and R2 to the Access Layer

Trunking and VLAN propagation

Access Switches
DHCP Snooping enabled

Uplinks marked as trusted

Prevents rogue DHCP servers

🔥 5. Firewall ASA 5505
The ASA provides segmentation and security enforcement.

Inside (VLAN1)
Subnet: 192.168.1.0/24

DHCPd tested but unreliable in Packet Tracer → static IPs used

Outside (VLAN2)
Subnet: 203.0.113.9/29

Connected to Router R (203.0.113.1)

Default route → Router R

NAT Configuration
Code
object network Inside-net
 subnet 192.168.1.0 255.255.255.0
 nat (inside,outside) dynamic interface
ACLs
Inside → Outside

Code
access-list in-out extended permit ip 192.168.1.0 255.255.255.0 any
access-group in-out in interface inside
Outside → DMZ (HTTP/HTTPS)

Code
access-list out-in extended permit tcp any host 203.0.113.11 eq 80
access-list out-in extended permit tcp any host 203.0.113.11 eq 443
access-group out-in in interface outside
Security Guarantees
Stateful inspection

Only DMZ web server exposed externally

All other unsolicited inbound traffic blocked

🔐 6. NAT and ACL Summary
Router R
NAT overload for VLANs

ACL vlanNat restricts translation to internal networks

ASA
Object NAT for inside users

Strict ACLs controlling inbound/outbound flows

📡 7. DHCP Evolution
Phase	Description
Phase 1	Router R as DHCP server
Phase 2	ASA DHCPd tested (failed in Packet Tracer)
Phase 3	Static IPs on ASA inside VLAN
Phase 4	Planned external DHCP server (172.16.0.1)
Router R already relays DHCP via ip helper-address.

🛰 8. Routing
EIGRP AS 1 deployed on R, R1, R2

Advertises:

VLAN networks

Transit links

Public subnets

Static route redistribution ensures ASA and external networks are reachable

⚠️ 9. Known Issues / Limitations
ASA DHCPd unreliable in Packet Tracer

Multiple default routes on Router R may cause instability

ASA 5505 licensing limits DMZ deployment (only 2 named interfaces)

🚀 10. Future Improvements
Deploy external DHCP server (172.16.0.1)

Enable full DMZ once Packet Tracer supports more ASA interfaces

Harden ACLs (least privilege)

Replace redundant default routes with proper failover


🧩 Améliorations futures
Ajout de schémas réseau (PNG)

Ajout de configurations CLI complètes (.txt)

Ajout d’un lab EIGRP

Ajout d’un lab NAT/PAT

Documentation détaillée de chaque topologie

👤 Auteur
Rhodian Japha Ndamen Fomen  
B.Sc. Informatique – UQO
Passionné par les réseaux, le support technique et le développement logiciel.
