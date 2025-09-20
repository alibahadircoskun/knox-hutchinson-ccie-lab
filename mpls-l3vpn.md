# MPLS L3VPN - BGP and VRF Configuration

## Overview
Implemented comprehensive MPLS L3VPN services using BGP VPNv4 address family with Route Reflector architecture. Configured customer segmentation through VRFs with EIGRP PE-CE routing protocols, ensuring proper route distribution and traffic isolation between Customer 1 and Customer 2 networks.

## Tasks

### Core MPLS Infrastructure
1. iBGP configuration with ASN 65000 across all MPLS routers
2. R1 configured as BGP Route Reflector with peer templates
3. VRF creation for customer segmentation with proper RT/RD values
4. VPNv4 route reflection from R1 to all PE routers

### Customer Segmentation and Routing
5. Customer 2 EIGRP relationships with MPLS Provider Edge routers
6. Customer 1 EIGRP relationships with MPLS Provider Edge routers
7. Traffic isolation between customers through VRF implementation
8. VPNv4 route advertisement from PE routers (R2-R5) to Route Reflector (R1)
9. Proper redistribution between EIGRP and BGP VPNv4

## Key Configuration Examples

### BGP Route Reflector with Templates (R1)

#### BGP Template Configuration
```cisco
! R1 - Route Reflector with Peer Templates (1.1.1.1)
router bgp 65000
 template peer-policy POLICY
  route-reflector-client
 exit-peer-policy
 
 template peer-session SESSION
  remote-as 65000
  update-source Loopback0
 exit-peer-session

 bgp router-id 1.1.1.1
 bgp log-neighbor-changes
 
 ! Apply templates to PE neighbors
 neighbor 2.2.2.2 inherit peer-session SESSION
 neighbor 3.3.3.3 inherit peer-session SESSION
 neighbor 4.4.4.4 inherit peer-session SESSION
 neighbor 5.5.5.5 inherit peer-session SESSION

 ! IPv4 Address Family
 address-family ipv4
  neighbor 2.2.2.2 activate
  neighbor 2.2.2.2 inherit peer-policy POLICY
  neighbor 3.3.3.3 activate
  neighbor 3.3.3.3 inherit peer-policy POLICY
  neighbor 4.4.4.4 activate
  neighbor 4.4.4.4 inherit peer-policy POLICY
  neighbor 5.5.5.5 activate
  neighbor 5.5.5.5 inherit peer-policy POLICY
 exit-address-family

 ! VPNv4 Address Family
 address-family vpnv4
  neighbor 2.2.2.2 activate
  neighbor 2.2.2.2 send-community both
  neighbor 2.2.2.2 route-reflector-client
  neighbor 3.3.3.3 activate
  neighbor 3.3.3.3 send-community both
  neighbor 3.3.3.3 route-reflector-client
  neighbor 4.4.4.4 activate
  neighbor 4.4.4.4 send-community both
  neighbor 4.4.4.4 route-reflector-client
  neighbor 5.5.5.5 activate
  neighbor 5.5.5.5 send-community both
  neighbor 5.5.5.5 route-reflector-client
 exit-address-family
```

### VRF Configuration with Route Targets

#### Customer VRF Setup (R2 Example)
```cisco
! Customer 1 VRF
vrf definition CUSTOMER1
 rd 1:1
 address-family ipv4
  route-target export 1:1
  route-target import 1:1
 exit-address-family

! Customer 2 VRF
vrf definition CUSTOMER2
 rd 2:2
 address-family ipv4
  route-target export 2:2
  route-target import 2:2
 exit-address-family
```

### PE Router Configuration (R2 Example)

#### EIGRP Named Mode VRF Configuration
```cisco
! EIGRP Named Mode for Customer VRFs
router eigrp CUSTOMER_EIGRP
 address-family ipv4 unicast vrf CUSTOMER1 autonomous-system 111
  topology base
   redistribute bgp 65000
  exit-af-topology
  network 192.169.28.0
 
 address-family ipv4 unicast vrf CUSTOMER2 autonomous-system 222
  topology base
   redistribute bgp 65000
  exit-af-topology
  network 192.169.29.0
```

#### BGP VPNv4 Configuration
```cisco
! R2 BGP Configuration with VRF Address Families
router bgp 65000
 neighbor 1.1.1.1 remote-as 65000
 neighbor 1.1.1.1 update-source loopback0
 
 address-family vpnv4
  neighbor 1.1.1.1 activate
  neighbor 1.1.1.1 send-community both
 exit-address-family
 
 ! Customer 1 VRF Address Family
 address-family ipv4 vrf CUSTOMER1
  redistribute eigrp 111
 exit-address-family
 
 ! Customer 2 VRF Address Family  
 address-family ipv4 vrf CUSTOMER2
  redistribute eigrp 222
 exit-address-family
```

### Customer-Facing EIGRP Configuration

#### Interface VRF Assignment
```cisco
! Customer 1 interface
interface [customer1-interface]
 description To Cust1-SiteX
 vrf forwarding CUSTOMER1
 ip address 192.169.28.2 255.255.255.0

! Customer 2 interface (R2 example)
interface Ethernet0/2
 description To Cust2-Site2
 vrf forwarding CUSTOMER2
 ip address 192.169.29.2 255.255.255.0
 duplex auto
```

#### EIGRP Named Mode VRF Implementation
```cisco
! R2 - EIGRP Named Mode for Both Customers
router eigrp CUSTOMER_EIGRP
 address-family ipv4 unicast vrf CUSTOMER1 autonomous-system 111
  topology base
   redistribute bgp 65000
  exit-af-topology
  network 192.169.28.0 0.0.0.255
  
 address-family ipv4 unicast vrf CUSTOMER2 autonomous-system 222
  topology base
   redistribute bgp 65000
  exit-af-topology  
  network 192.169.29.0 0.0.0.255
```

#### Customer Equipment EIGRP Configuration Examples

##### Customer 1 Equipment (R6 - ASN 111)
```cisco
! R6 - Customer 1 Router with OSPF Integration
router eigrp CCIE
 address-family ipv4 unicast autonomous-system 111
  topology base
   redistribute ospf 1 metric 1000000 10 255 1 1500
  exit-af-topology
  network 192.169.46.0

! OSPF integration within Customer 1 network
router ospf 1
 auto-cost reference-bandwidth 1000000
 redistribute eigrp 111 metric 10 metric-type 1 subnets
 passive-interface default
 no passive-interface Ethernet0/2
 network 6.6.6.6 0.0.0.0 area 2
 network 10.0.11.0 0.0.0.255 area 1
 network 192.0.0.0 0.255.255.255 area 1
```

##### Customer 2 Equipment (R7 - Multiple ASNs)
```cisco
! R7 - Customer 2 Router with Advanced EIGRP Features
! EIGRP ASN 50 for internal customer routing
router eigrp CCIE
 address-family ipv4 unicast autonomous-system 50
  topology base
   variance 2
   traffic-share min across-interfaces
   redistribute eigrp 222
   offset-list R38LOOP in 65536010 Ethernet0/3 
  exit-af-topology
  network 7.7.7.7 0.0.0.0
  network 10.0.21.0 0.0.0.255
  network 10.0.57.0 0.0.0.255
  network 10.0.99.0 0.0.0.255

! EIGRP ASN 222 for PE-CE connection
router eigrp CCIE2
 address-family ipv4 unicast autonomous-system 222
  topology base
   redistribute eigrp 50
   redistribute bgp 65000
  exit-af-topology
  network 192.169.47.0
```

## Design Principles

### Route Reflector Architecture
Using R1 as a Route Reflector eliminates the need for full-mesh iBGP between all PE routers. This design:
- **Scales efficiently** as new PE routers only need to peer with R1
- **Centralizes route distribution** for easier management and troubleshooting
- **Reduces configuration complexity** through BGP peer templates
- **Maintains redundancy** through multiple route reflectors in production

### BGP Template Strategy
Peer templates provide configuration consistency and operational efficiency:
- **Session templates** handle connection parameters (timers, source interface)
- **Policy templates** manage routing behaviors (route reflection, communities)
- **Inheritance model** allows for easy template modifications affecting all peers
- **Scalability benefits** when managing large numbers of PE routers

### VRF Segmentation Model
Virtual Routing and Forwarding (VRF) provides customer traffic isolation:
- **Route Distinguisher (RD)** creates unique route namespaces
- **Route Target (RT)** controls route import/export between VRFs
- **Address family separation** maintains routing table independence

### Route Redistribution Strategy
Bidirectional redistribution between EIGRP and BGP enables:
- **Customer route learning** from CE routers via EIGRP
- **VPN route distribution** through BGP VPNv4 address family
- **Metric preservation** maintaining original path costs

## Verification Commands

```cisco
! BGP VPNv4 Verification
show bgp vpnv4 unicast all summary
show bgp vpnv4 unicast vrf CUSTOMER1
show bgp vpnv4 unicast vrf CUSTOMER2
show bgp vpnv4 unicast neighbors

! VRF Verification
show ip vrf
show ip vrf interfaces
show ip route vrf CUSTOMER1
show ip route vrf CUSTOMER2

! EIGRP VRF Verification
show eigrp address-family ipv4 vrf CUSTOMER1 neighbors
show eigrp address-family ipv4 vrf CUSTOMER2 neighbors
show eigrp address-family ipv4 vrf CUSTOMER1 topology

! Route Reflector Verification
show ip bgp summary
show ip bgp neighbors [peer] advertised-routes
show ip bgp neighbors [peer] routes
show bgp vpnv4 unicast all neighbors [peer] routes

! Extended Community Verification
show bgp vpnv4 unicast all
show bgp vpnv4 unicast [network] detail
debug bgp vpnv4 unicast updates
```
