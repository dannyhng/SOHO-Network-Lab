# Small Office / Home Office (SOHO) Network Lab

This project is a fully configured small office network built in Cisco Packet Tracer. It demonstrates a foundational understanding of core networking principles in routing, switching, and security. The goal was to create a segmented, secure, and scalable network for a small business with multiple departments.

## Network Topology

*A visual representation of the SOHO network, including VLANs, IP subnets, and device connections.*

```
![Network Diagram](https://raw.githubusercontent.com/dannyhng/SOHO-Network-Lab/main/network-diagram.png)

```

## Key Features & Technologies Implemented

This lab demonstrates proficiency in the following areas:

### Switching
* **VLANs:** Segmented the network into three distinct broadcast domains: `CORP` (VLAN 10), `GUEST` (VLAN 20), and `SERVERS` (VLAN 30) for improved security and traffic management.
* **802.1Q Trunking:** Configured a trunk link between the switch and the router to carry traffic for all VLANs.
* **Port Assignment:** Statically assigned switch ports to their respective VLANs.

### Routing
* **Inter-VLAN Routing:** Implemented a "Router-on-a-Stick" configuration to enable communication between the CORP and SERVERS VLANs while keeping them logically separate.
* **Static Routing:** Configured a static default route on the Office Router to provide a path to the internet.

### Network Services
* **DHCP Server:** Configured the Office Router to act as a DHCP server, dynamically assigning IP addresses to hosts in the CORP and GUEST VLANs.
* **NAT/PAT:** Implemented Port Address Translation (PAT) on the edge router, allowing multiple internal devices with private IPs to access the internet using a single public IP.

### Security
* **Access Control Lists (ACLs):** (Future Addition) Standard and extended ACLs will be configured to filter traffic between VLANs, preventing the GUEST network from accessing internal resources.
* **Secure Management:** (Future Addition) SSH will be configured for secure remote management of network devices, and Telnet will be disabled.

## IP Addressing Plan

| VLAN ID & Name        | Network Address  | Subnet Mask     | Gateway Address  |
| --------------------- | ---------------- | --------------- | ---------------- |
| VLAN 10 (CORP)        | `192.168.10.0`   | `255.255.255.0` | `192.168.10.1`   |
| VLAN 20 (GUEST)       | `192.168.20.0`   | `255.255.255.0` | `192.168.20.1`   |
| VLAN 30 (SERVERS)     | `192.168.30.0`   | `255.255.255.0` | `192.168.30.1`   |
| WAN Link (Office-ISP) | `10.0.0.0`       | `255.255.255.252` | `10.0.0.1`       |

## Device Configurations

The full `show running-config` for each device is included below. **Click on a device to expand its configuration.**

<details>
  <summary> R0 - Office Router</summary>
  
  ```cisco
  !
hostname R0
!
!=================================================
! DHCP Configuration
!=================================================
ip dhcp excluded-address 192.168.10.1
ip dhcp excluded-address 192.168.20.1
ip dhcp excluded-address 192.168.30.1
!
ip dhcp pool CORP_VLAN
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8
!
ip dhcp pool GUEST_VLAN
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
 dns-server 8.8.8.8
!
!=================================================
! Interface Configuration
!=================================================
interface GigabitEthernet0/0
 description Trunk Link to SW1
 no ip address
 ip nat inside
!
interface GigabitEthernet0/0.10
 description Gateway for CORP VLAN
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
!
interface GigabitEthernet0/0.20
 description Gateway for GUEST VLAN
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
!
interface GigabitEthernet0/0.30
 description Gateway for SERVERS VLAN
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
!
interface GigabitEthernet0/1
 description WAN Link to ISP
 ip address 10.0.0.1 255.255.255.252
 ip nat outside
!
!=================================================
! NAT and Routing Configuration
!=================================================
ip nat inside source list 1 interface GigabitEthernet0/1 overload
ip route 0.0.0.0 0.0.0.0 10.0.0.2
!
access-list 1 permit 192.168.10.0 0.0.0.255
access-list 1 permit 192.168.20.0 0.0.0.255
access-list 1 permit 192.168.30.0 0.0.0.255
!
line con 0
 password cisco
 login
line vty 0 4
 password cisco
 login
!
end
  ...
  ```
</details>

<details>
  <summary> SW1 - Main Switch</summary>
  
  ```cisco
  !
hostname SW1
!
!=================================================
! VLAN Database
!=================================================
vlan 10
 name CORP
!
vlan 20
 name GUEST
!
vlan 30
 name SERVERS
!
!=================================================
! Interface Configuration
!=================================================
!
! --- Access Ports for CORP VLAN ---
interface range FastEthernet0/1 - 10
 switchport mode access
 switchport access vlan 10
!
! --- Access Ports for GUEST VLAN ---
interface range FastEthernet0/11 - 20
 switchport mode access
 switchport access vlan 20
!
! --- Access Port for SERVER VLAN ---
interface FastEthernet0/24
 switchport mode access
 switchport access vlan 30
!
! --- Trunk Port to Office Router ---
interface GigabitEthernet0/1
 switchport mode trunk
!
line con 0
 password cisco
 login
line vty 0 15
 password cisco
 login
!
end
  ...
  ```
</details>

<details>
  <summary> R1 - ISP Router</summary>

  ```cisco
  !
hostname R1
!
!=================================================
! Interface Configuration
!=================================================
!
! --- Link to Office Router ---
interface GigabitEthernet0/1
 description WAN Link from Customer R0
 ip address 10.0.0.2 255.255.255.252
 no shutdown
!
line con 0
 password cisco
 login
line vty 0 4
 password cisco
 login
!
end
  ...
  ```
</details>

## How to Use

1.  Download and install [Cisco Packet Tracer](https://www.netacad.com/courses/packet-tracer).
2.  Download the `SOHO-Network.pkt` file from this repository.
3.  Open the file in Packet Tracer to explore the topology, run simulations, and inspect device configurations.
