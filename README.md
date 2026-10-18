# King Power Stadium VLAN Segmentation Lab

A multi-tier hierarchical network lab simulating the operational network architecture of King Power Stadium. This project demonstrates core enterprise networking principles by implementing VLAN segmentation, 802.1Q trunking, and Router-on-a-Stick routing to isolate public fan traffic from administrative operations.

---

## Network Architecture Overview

```text
 [ R1-KingPower-Edge ] (Router Gateway / Inter-VLAN Routing)
          | (802.1Q Trunk)
 [ DSW1-KingPower-Core ] (Distribution / Core Aggregation Switch)
          | (802.1Q Trunk)
 [ SW1-KingPower-Concourse ] (Access Switch)
          | (Access Port - VLAN 20)<img width="383" height="437" alt="Screenshot 2026-10-18 at 11 00 53 pm" src="https://github.com/user-attachments/assets/ba86ecc2-2268-4b34-9a42-b4ec47a34ca5" />

 [ PC1-Fan-Workstation ] (Public Fan Wi-Fi Terminal)
```

---

## Subnet & VLAN Allocation

| VLAN ID | Name | Subnet / Mask | Gateway IP | Description |
| :--- | :--- | :--- | :--- | :--- |
| **VLAN 10** | `Admin-Operations` | `192.168.10.0/24` | `192.168.10.1` | Secure backend management and ticketing systems |
| **VLAN 20** | `Fan-Wi-Fi` | `192.168.20.0/24` | `192.168.20.1` | Public concourse Wi-Fi and attendee terminals |

---

## Configuration Highlights

These excerpts highlight the key features implemented across the network topology.

### 1. Router Subinterfaces (R1-KingPower-Edge)
Configures virtual subinterfaces on a single physical port to provide inter-VLAN routing using Router-on-a-Stick.

```text
interface g0/0
 no shutdown

interface g0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0

interface g0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
```

### 2. Core Aggregation & Trunking (DSW1-KingPower-Core)
Establishes VLANs on the distribution switch and provisions 802.1Q trunk links upstream to the router and downstream to the access layer.

```text
vlan 10
 name Admin-Operations
vlan 20
 name Fan-Wi-Fi

interface g0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 no shutdown

interface g0/2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 no shutdown
```

### 3. Access Layer & Port Assignment (SW1-KingPower-Concourse)
Configures local VLAN mapping on the access switch and assigns end-user ports.

```text
vlan 10
 name Admin-Operations
vlan 20
 name Fan-Wi-Fi

interface f0/1
 switchport mode access
 switchport access vlan 20
 no shutdown

interface g0/1
 switchport mode trunk
 no shutdown
```

---

## Verification & Testing

To verify that the network topology and VLAN segmentation are operating correctly, execute the following test from your endpoint:

```text
ping 192.168.20.1
```
* **Verification Criteria:** Successful ICMP echo replies confirm that traffic traverses the access switch, crosses the 802.1Q trunk to the distribution switch, and successfully hits the router gateway subinterface.
