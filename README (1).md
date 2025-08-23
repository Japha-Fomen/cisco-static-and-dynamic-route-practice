# Network Project Documentation

## 1. Overview
This project simulates a multi-router hierarchical network using Cisco Packet Tracer.  
The topology includes:  
- **Core Routers** (R, R1, R2)  
- **Core Switches** (Core1, Core2)  
- **Access Switches** with DHCP snooping enabled  
- **Firewall ASA 5505** (inside, outside, DMZ zones)  
- **End devices** (PCs, servers)  

- **Subinterfaces** created for VLANs:
  - VLAN 10 → 10.10.10.0/24  
  - VLAN 20 → 10.10.20.0/24  
  - VLAN 30 → 10.10.30.0/27  
  - VLAN 40 → 10.10.40.0/28  

The design follows a **3-layer hierarchy**: Core, Distribution, Access.

---

## 2. Router R (Central Services Router)
Initially, **Router R** was configured as the **DHCP server** for all VLANs.  
Later, this function was offloaded, and Router R now:  
- Acts as **NAT boundary** (translating inside private IPs to public).  
- Routes between internal subnets and the outside provider.  
- Forwards DHCP requests using `ip helper-address`.

**Key Configuration**:
- `ip nat pool vlan` defined for 203.0.113.3–203.0.113.6.  
- `ip access-list standard vlanNat` permits VLAN subnets (10.10.10.0/24, 10.10.20.0/24, 10.10.30.0/27, 10.10.40.0/28).  
- `ip nat inside source list vlanNat pool vlan overload` for PAT.  
- Routes:  
  - Default routes towards both the ASA (203.0.113.9) and outside provider (203.0.113.2).  
- `ip helper-address 172.16.0.1` configured for forwarding DHCP requests (planned external DHCP server).  

---

## 3. Routers R1 and R2
R1 and R2 provide **Inter-VLAN Routing** and **HSRP redundancy**:  
- Subinterfaces configured on FastEthernet0/0 for VLANs 10, 20, 30, and 40.  
- Each VLAN has:  
  - **HSRP Virtual IP** ending with `.1`.  
  - R1 as **active** for VLAN 10 & 30.  
  - R2 as **active** for VLAN 20 & 40.  
- Both forward DHCP requests to Router R / external DHCP using `ip helper-address`.

---

## 4. Switch Layer
- Core switches (Core1, Core2) connect R1 and R2 to the Access Layer.  
- **Access switches**:  
  - Configured with **DHCP snooping**.  
  - Uplinks to distribution routers are marked as **trusted**.  
  - This prevents rogue DHCP servers at the edge.

---

## 5. Firewall ASA 5505
The ASA enforces network security and segmentation:  
- **Inside (VLAN1)**: 192.168.1.0/24  
  - Initially configured with `dhcpd` to provide DHCP leases.  
  - However, end devices were not receiving addresses due to ASA limitations in Packet Tracer.  
  - As a workaround, devices were set with **static IPs** in the subnet.  
- **Outside (VLAN2)**: 203.0.113.9/29  
  - Connected to Router R’s Serial2/0 (203.0.113.1).  
  - Default route points to Router R.  
- **NAT**: Inside addresses translated dynamically to the outside interface.  
- **ACLs**:  
  - `in-out` applied **inside**:  
    - Permits inside → outside traffic.  
    - Permits communication with specific external hosts.  
  - `out-in` applied **outside**:  
    - Permits selected inbound traffic (e.g. web server in DMZ).  

NAT

Configured for inside users to reach outside:

object network Inside-net
 subnet 192.168.1.0 255.255.255.0
 nat (inside,outside) dynamic interface

ACLs

Inside → Outside traffic (allow internal users access to outside):

access-list in-out extended permit ip 192.168.1.0 255.255.255.0 any
access-group in-out in interface inside


Outside → DMZ server access (HTTP/HTTPS):

access-list out-in extended permit tcp any host 203.0.113.11 eq 80
access-list out-in extended permit tcp any host 203.0.113.11 eq 443
access-group out-in in interface outside


This ensures:

Internal users can initiate sessions to the outside (stateful inspection).

External users can only reach the web server in DMZ on ports 80/443.

All other unsolicited traffic from outside is blocked.

5. 🔹 Security Features

DHCP Snooping enabled on access switches, with trusted uplinks.

ACLs carefully bound to the correct interfaces:

access-group in-out in interface inside

access-group out-in in interface outside

##### end
---

## 6. NAT and ACLs
- **On Router R**:  
  - NAT overload via pool `vlan`.  
  - ACL `vlanNat` ensures only internal VLANs are translated.  
- **On ASA**:  
  - Object NAT (`nat (inside,outside) dynamic interface`).  
  - ACLs explicitly control allowed inbound/outbound flows.  

---

## 7. DHCP Evolution
- **Phase 1**: Router R was DHCP server for all VLANs.  
- **Phase 2**: DHCP tested directly on ASA (using `dhcpd` inside). Failed in Packet Tracer.  
- **Phase 3 (Current)**: End devices on ASA’s inside VLAN1 configured with static IPs.  
- **Planned Phase 4**: Introduce **external DHCP server** (reachable at 172.16.0.1). Router R relays DHCP requests using `ip helper-address`.

---

## 8. Routing
- **EIGRP (AS 1)** configured on Router R, R1, R2.  
- Networks advertised: internal VLANs, transit networks, and public subnets.  
- Redistribution of static routes allows ASA and external networks to be reachable.

---

## 9. Known Issues / Limitations
- **ASA DHCPd**: In Packet Tracer ASA 5505, DHCP service is unreliable. Static IPs were used instead.  
- **Multiple Default Routes on Router R**: Redundant static defaults towards ASA and upstream provider may cause instability; should choose one primary.  
- **Firewall Licensing Limits**: ASA only supports 2 active named interfaces without special license, limiting DMZ deployment.  

---

## 10. Future Work
- Deploy **dedicated external DHCP server** at `172.16.0.1`.  
- Configure **ASA DMZ** once Packet Tracer allows more than 2 named VLANs.  
- Fine-tune ACLs to follow the principle of least privilege.  
- Replace redundant default routes with proper failover design.  
