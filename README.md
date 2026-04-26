# End to End Enterprise Network Flow Across Layered Architecture

## Overview
Designed and validated a structured enterprise style network integrating VLAN segmentation DHCP inter VLAN routing ACLs NAT static routing DNS and server access. Built the topology using a layered approach with access distribution edge and server segments to demonstrate how real networks move traffic through controlled paths.

## Objective
Implemented and analyzed full packet flow across multiple VLANs and routers to verify how segmented networks communicate internally and externally through routing policy enforcement address translation and DNS resolution.

## Network Setup
Devices Used
1 Cisco 2960 Switch
3 Cisco 1941 Routers
1 Server
9 End Devices PCs

Topology Design
- VLAN 10 HR uses subnet 192.168.10.0/24
- VLAN 20 IT uses subnet 192.168.20.0/24
- VLAN 30 Finance uses subnet 192.168.30.0/24
```
Router2 configured as the distribution router using Router on a Stick
Router4 configured as a transit router between distribution and edge
Router5 configured as the edge router connected toward the server network
Switch3 configured as the access layer switch for all VLAN endpoints
Server0 configured for DNS service using address 200.1.3.10
```
---

### Policy Design
- HR cannot reach Finance
- Finance cannot reach IT
- IT has full access

Logical Flow
PC to Switch3 to Router2 to Router4 to Router5 to Server0 and back

Topology
![Network Topology](https://github.com/Pelumi-Johnson/End-to-End-Enterprise-Network-Flow-Across-Layered-Architecture/blob/main/Screenshot%202026-04-25%20200400.png)

## Configuration

### VLAN Configuration
```
enable
configure terminal

vlan 10
name HR

vlan 20
name IT

vlan 30
name Finance

interface fastEthernet 0/1
switchport mode access
switchport access vlan 10

interface fastEthernet 0/2
switchport mode access
switchport access vlan 10

interface fastEthernet 0/3
switchport mode access
switchport access vlan 10

interface fastEthernet 0/4
switchport mode access
switchport access vlan 20

interface fastEthernet 0/5
switchport mode access
switchport access vlan 20

interface fastEthernet 0/6
switchport mode access
switchport access vlan 20

interface fastEthernet 0/7
switchport mode access
switchport access vlan 30

interface fastEthernet 0/8
switchport mode access
switchport access vlan 30

interface fastEthernet 0/9
switchport mode access
switchport access vlan 30

interface fastEthernet 0/24
switchport mode trunk
exit
```
VLAN Config
![VLAN Config](https://github.com/Pelumi-Johnson/End-to-End-Enterprise-Network-Flow-Across-Layered-Architecture/blob/main/Screenshot%202026-04-25%20200426.png)

---

### Router2 Inter VLAN Routing Configuration/DHCP Configuration
```
interface gigabitEthernet 0/0
no shutdown

interface gigabitEthernet 0/0.10
encapsulation dot1Q 10
ip address 192.168.10.1 255.255.255.0
ip nat inside

interface gigabitEthernet 0/0.20
encapsulation dot1Q 20
ip address 192.168.20.1 255.255.255.0
ip nat inside

interface gigabitEthernet 0/0.30
encapsulation dot1Q 30
ip address 192.168.30.1 255.255.255.0
ip nat inside

interface gigabitEthernet 0/1
ip address 200.1.1.1 255.255.255.0
ip nat outside
no shutdown
```
---

```
ip dhcp pool HR
network 192.168.10.0 255.255.255.0
default-router 192.168.10.1
dns-server 200.1.3.10

ip dhcp pool IT
network 192.168.20.0 255.255.255.0
default-router 192.168.20.1
dns-server 200.1.3.10

ip dhcp pool Finance
network 192.168.30.0 255.255.255.0
default-router 192.168.30.1
dns-server 200.1.3.10
```
Router2 Subinterfaces
![Router2 Subinterfaces](https://github.com/Pelumi-Johnson/End-to-End-Enterprise-Network-Flow-Across-Layered-Architecture/blob/main/Screenshot%202026-04-25%20201247.png)

---

### Router2 ACL Configuration
```
access-list 100 deny ip 192.168.10.0 0.0.0.255 192.168.30.0 0.0.0.255
access-list 100 permit ip any any

access-list 110 deny ip 192.168.30.0 0.0.0.255 192.168.20.0 0.0.0.255
access-list 110 permit ip any any

interface gigabitEthernet 0/0.10
ip access-group 100 in

interface gigabitEthernet 0/0.30
ip access-group 110 in
```
ACL Config
![ACL Config](https://github.com/Pelumi-Johnson/End-to-End-Enterprise-Network-Flow-Across-Layered-Architecture/blob/main/Screenshot%202026-04-25%20202708.png)

---

### Router2 NAT Configuration
```
access-list 1 permit 192.168.10.0 0.0.0.255
access-list 1 permit 192.168.20.0 0.0.0.255
access-list 1 permit 192.168.30.0 0.0.0.255

ip nat inside source list 1 interface gigabitEthernet 0/1 overload
```
NAT Config
![NAT Config](https://github.com/Pelumi-Johnson/End-to-End-Enterprise-Network-Flow-Across-Layered-Architecture/blob/main/Screenshot%202026-04-25%20205054.png)

---

### Router2 Static Route Configuration
```
ip route 0.0.0.0 0.0.0.0 200.1.1.2
```
Router2 Static Route
![Router2 Static Route](https://github.com/Pelumi-Johnson/End-to-End-Enterprise-Network-Flow-Across-Layered-Architecture/blob/main/Screenshot%202026-04-25%20202708.png)

---

### Router2 Interface and Route Verification
```
show ip interface brief
show ip route
```
Router2 Interface Brief
![Router2 Interface Brief](https://github.com/Pelumi-Johnson/End-to-End-Enterprise-Network-Flow-Across-Layered-Architecture/blob/main/Screenshot%202026-04-25%20203059.png)

Router2 Route Table
![Router2 Route Table](https://github.com/Pelumi-Johnson/End-to-End-Enterprise-Network-Flow-Across-Layered-Architecture/blob/main/Screenshot%202026-04-25%20203228.png)

---

### Router4 Transit Configuration
```
interface gigabitEthernet 0/0
ip address 200.1.1.2 255.255.255.0
no shutdown

interface gigabitEthernet 0/1
ip address 200.1.2.1 255.255.255.0
no shutdown

ip route 200.1.3.0 255.255.255.0 200.1.2.2
```
Router4 Config
![Router4 Config](https://github.com/Pelumi-Johnson/End-to-End-Enterprise-Network-Flow-Across-Layered-Architecture/blob/main/Screenshot%202026-04-25%20214003.png)

---

### Router5 Edge Configuration
```
interface gigabitEthernet 0/0
ip address 200.1.2.2 255.255.255.0
no shutdown

interface gigabitEthernet 0/1
ip address 200.1.3.1 255.255.255.0
no shutdown

ip route 200.1.1.0 255.255.255.0 200.1.2.1
```
Router5 Config
![Router5 Config](https://github.com/Pelumi-Johnson/End-to-End-Enterprise-Network-Flow-Across-Layered-Architecture/blob/main/Screenshot%202026-04-25%20214102.png)

---

### Server Configuration
```
Server0
IP Address 200.1.3.10
Subnet Mask 255.255.255.0
Default Gateway 200.1.3.1

DNS Record
pelumi.com mapped to 200.1.3.10
```
Server IP Config
![Server IP Config](https://github.com/Pelumi-Johnson/End-to-End-Enterprise-Network-Flow-Across-Layered-Architecture/blob/main/Screenshot%202026-04-25%20214205.png)

DNS Config
![DNS Config](https://github.com/Pelumi-Johnson/End-to-End-Enterprise-Network-Flow-Across-Layered-Architecture/blob/main/Screenshot%202026-04-25%20214242.png)

---

## End to End Flow Validation

### Step 1 DHCP Assignment
Validated that client devices in HR IT and Finance received correct IP addresses default gateways and DNS server information from Router2 DHCP pools

Expected Results
- HR clients receive 192.168.10.0/24 addressing
- IT clients receive 192.168.20.0/24 addressing
- Finance clients receive 192.168.30.0/24 addressing

HR DHCP Validation
![HR DHCP Validation](https://github.com/Pelumi-Johnson/End-to-End-Enterprise-Network-Flow-Across-Layered-Architecture/blob/main/Screenshot%202026-04-25%20201637.png)

IT DHCP Validation
![IT DHCP Validation](https://github.com/Pelumi-Johnson/End-to-End-Enterprise-Network-Flow-Across-Layered-Architecture/blob/main/Screenshot%202026-04-25%20201651.png)

Finance DHCP Validation
![Finance DHCP Validation](https://github.com/Pelumi-Johnson/End-to-End-Enterprise-Network-Flow-Across-Layered-Architecture/blob/main/Screenshot%202026-04-25%20201704.png)


---

### Step 2 Baseline Local Communication Testing
Tested communication before validating policy enforcement
```
HR to Finance
ping 192.168.30.3

Finance to IT
ping 192.168.20.2
```
Result
Initial routing confirmed traffic could move between VLANs through Router2 before ACL restrictions were enforced

HR to Finance Ping
![HR to Finance Ping](https://github.com/Pelumi-Johnson/End-to-End-Enterprise-Network-Flow-Across-Layered-Architecture/blob/main/Screenshot%202026-04-25%20201820.png)

Finance to IT Ping
![Finance to IT Ping](https://github.com/Pelumi-Johnson/End-to-End-Enterprise-Network-Flow-Across-Layered-Architecture/blob/main/Screenshot%202026-04-25%20202805.png)

---

### Step 3 ACL Policy Validation

Validated restricted communication based on departmental policy
```
HR to Finance
ping 192.168.30.3
Result destination unreachable confirming HR cannot reach Finance

Finance to IT
ping 192.168.20.2
Result destination unreachable confirming Finance cannot reach IT

IT to HR and Finance
ping 192.168.10.2
ping 192.168.30.3
Result successful confirming IT has full access. It could reach both vlan but vlan 30 is restricted from responding back so it timed out
```
HR Blocked from Finance
![HR Blocked from Finance](https://github.com/Pelumi-Johnson/End-to-End-Enterprise-Network-Flow-Across-Layered-Architecture/blob/main/Screenshot%202026-04-25%20202737.png)

Finance Blocked from IT
![Finance Blocked from IT](https://github.com/Pelumi-Johnson/End-to-End-Enterprise-Network-Flow-Across-Layered-Architecture/blob/main/Screenshot%202026-04-25%20202805.png)

IT Full Access
![IT Full Access](https://github.com/Pelumi-Johnson/End-to-End-Enterprise-Network-Flow-Across-Layered-Architecture/blob/main/Screenshot%202026-04-25%20225336.png)

---

### Step 4 Router Interface Verification
Verified Router2 subinterfaces and external interface status
```
show ip interface brief
```
Result
VLAN subinterfaces and external interface were active and assigned to the correct IP networks

Screenshot Placeholder Interface Verification
![Interface Verification](./screenshots/interface-verification.png)

---

### Step 5 Static Routing Validation
Verified routed paths across Router2 Router4 and Router5
```
show ip route
```
Result
Static routes provided forward and return paths between internal translated traffic and the server network

Screenshot Placeholder Static Route Validation
![Static Route Validation](./screenshots/static-route-validation.png)

---

### Step 6 NAT Translation Validation
Validated NAT overload for internal VLAN traffic exiting toward the external network
```
show ip nat translations
```
Result
Internal private addresses were translated through Router2 outside interface for external communication

Screenshot Placeholder NAT Translation
![NAT Translation](./screenshots/nat-translation.png)

---

### Step 7 DNS Resolution Validation
Tested domain based communication to server
```
ping pelumi.com
```
Result
Domain resolved to 200.1.3.10 and traffic reached the server through the routed path

Screenshot Placeholder DNS Resolution
![DNS Resolution](./screenshots/dns-resolution.png)

---

## End to End Traffic Flow
```
Client receives IP configuration from DHCP
Client sends traffic to VLAN default gateway on Router2
Router2 evaluates ACL policy based on source and destination
Allowed traffic is routed toward Router4
Router4 forwards traffic toward Router5 using static routing
Router5 forwards traffic to the server network
DNS resolves pelumi.com to 200.1.3.10
Server response returns through Router5 and Router4
Router2 translates return traffic and delivers it back to the correct VLAN host
```
Screenshot Placeholder End to End Flow
![End to End Flow](./screenshots/end-to-end-flow.png)

---

## Key Concepts Applied
- Layered network design using access distribution edge and server segments
- VLAN segmentation for HR IT and Finance departments
- Router on a Stick for inter VLAN routing
- DHCP service delivery across multiple VLANs
- Extended ACL policy enforcement between departments
- NAT overload for private to external communication
- Static routing for multi router path control
- DNS resolution for domain based access
- End to end traffic flow validation across multiple services

## Outcome
Validated a complete enterprise style network flow across VLANs routing ACLs NAT static routing DHCP DNS and server access. Demonstrated that traffic behavior matched the intended design where HR was blocked from Finance Finance was blocked from IT and IT maintained full access. Confirmed that internal clients received addressing dynamically resolved DNS records and reached external services through a controlled routed path.
