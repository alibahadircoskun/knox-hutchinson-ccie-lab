# DMVPN - Dynamic Multipoint VPN Migration

## Overview
Migrated Customer 2 from MPLS L3VPN to DMVPN solution, implementing dual-hub Phase 3 DMVPN architecture with IKEv2 encryption. Configured Site 3 and Site 4 as hub routers with Site 1 and Site 2 as spoke routers, enabling dynamic spoke-to-spoke tunnel establishment over the Internet. Integrated EIGRP Named Mode for dynamic routing across the DMVPN overlay network.

## Tasks

### MPLS L3VPN Decommissioning
1. Removed all MPLS L3VPN configurations from Customer 2 edge devices

### DMVPN Infrastructure
4. Dual-hub DMVPN topology with Site 3 and Site 4 as hubs
5. Site 1 and Site 2 configured as DMVPN spokes
6. DMVPN Phase 3 implementation for optimal spoke-to-spoke communication
7. Tunnel IP addressing from 192.168.43.0/24 subnet

### Security and Routing
8. NHRP authentication configured with password "DATAKNOX"
9. IKEv2 IPsec encryption securing all DMVPN tunnels
10. EIGRP ASN 50 deployed across DMVPN network
11. Dynamic route exchange between all sites via DMVPN tunnels
12. Inter-site WAN link reachability verification

## Key Configuration Examples

### DMVPN Hub Configuration

#### Hub Router - Crypto Configuration (R11 Example)
```cisco
! IKEv2 Keyring Configuration
crypto ikev2 keyring IKE2KEY
 peer ANY
  address 0.0.0.0 0.0.0.0
  pre-shared-key DATAKNOX

! IKEv2 Profile Configuration
crypto ikev2 profile IKE2PROF
 match identity remote address 0.0.0.0
 authentication remote pre-share
 authentication local pre-share
 keyring local IKE2KEY

! IPsec Transform Set
crypto ipsec transform-set TRANSFORM ah-sha256-hmac esp-aes 192
 mode transport

! IPsec Profile
crypto ipsec profile PROFILE
 set transform-set TRANSFORM
 set ikev2-profile IKE2PROF
```

#### Hub Tunnel Interface Configuration

##### Hub Site 3 Configuration (R13)
```cisco
! Tunnel interface configuration - Hub Site 3 (R13)
interface Tunnel0
 bandwidth 1000000
 ip address 192.168.43.13 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp authentication DATAKNOX
 ip nhrp map multicast dynamic
 ip nhrp network-id 1
 ip nhrp redirect
 ip tcp adjust-mss 1360
 delay 2
 tunnel source Ethernet0/1
 tunnel mode gre multipoint
 tunnel protection ipsec profile PROFILE
```

##### Hub Site 4 Configuration (R11)
```cisco
! Tunnel interface configuration - Hub Site 4 (R11)
interface Tunnel0
 bandwidth 1000000
 ip address 192.168.43.11 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp authentication DATAKNOX
 ip nhrp map multicast dynamic
 ip nhrp network-id 1
 ip nhrp redirect
 ip tcp adjust-mss 1360
 delay 2
 tunnel source Ethernet0/1
 tunnel mode gre multipoint
 tunnel protection ipsec profile PROFILE
```

### DMVPN Spoke Configuration

#### Spoke Router - Crypto Configuration
```cisco
! Same crypto configuration as hubs
crypto ikev2 keyring IKE2KEY
 peer ANY
  address 0.0.0.0 0.0.0.0
  pre-shared-key DATAKNOX

crypto ikev2 profile IKE2PROF
 match identity remote address 0.0.0.0
 authentication remote pre-share
 authentication local pre-share
 keyring local IKE2KEY

crypto ipsec transform-set TRANSFORM ah-sha256-hmac esp-aes 192
 mode transport

crypto ipsec profile PROFILE
 set transform-set TRANSFORM
 set ikev2-profile IKE2PROF
```

#### Spoke Tunnel Interface - Site 1 Configuration (R9 Example)
```cisco
! Tunnel interface configuration - Spoke Site 1 (R9)
interface Tunnel1
 bandwidth 1000000
 ip address 192.168.43.9 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp authentication DATAKNOX
 ip nhrp network-id 1
 ip nhrp nhs 192.168.43.13 nbma 192.168.13.13 multicast priority 1 cluster 1
 ip nhrp nhs 192.168.43.11 nbma 192.168.11.11 multicast priority 2 cluster 1
 ip nhrp shortcut
 ip tcp adjust-mss 1360
 delay 2
 tunnel source Ethernet0/1
 tunnel mode gre multipoint
 tunnel protection ipsec profile PROFILE
```

#### Spoke Tunnel Interface - Site 2 Configuration (R7 Example)
```cisco
! Tunnel interface configuration - Spoke Site 2 (R7)
interface Tunnel2
 bandwidth 1000000
 ip address 192.168.43.7 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp authentication DATAKNOX
 ip nhrp network-id 1
 ip nhrp nhs 192.168.43.13 nbma 192.168.13.13 multicast priority 1 cluster 1
 ip nhrp nhs 192.168.43.11 nbma 192.168.11.11 multicast priority 2 cluster 1
 ip nhrp shortcut
 ip tcp adjust-mss 1360
 delay 2
 tunnel source Ethernet0/1
 tunnel mode gre multipoint
 tunnel protection ipsec profile PROFILE
```

## Design Principles

### DMVPN Phase 3 Architecture
Phase 3 DMVPN enables optimal spoke-to-spoke communication:
- **Direct spoke-to-spoke tunnels** without hub involvement for data traffic
- **NHRP redirect** messages from hubs trigger spoke shortcut tunnel creation
- **Scalability** through dynamic tunnel establishment only when needed
- **Bandwidth optimization** by eliminating hub as transit point for spoke traffic

### Dual-Hub Design Benefits
Implementing two hub routers provides:
- **Redundancy** for hub failure scenarios
- **Load distribution** across multiple hub routers
- **Geographic diversity** with hubs in different locations
- **No single point of failure** in the DMVPN network

### IKEv2 Security Implementation
IKEv2 offers improvements over IKEv1:
- **Built-in NAT traversal** without additional configuration
- **Improved reliability** through dead peer detection
- **DoS protection** with cookie-based anti-clogging
- **Simplified configuration** with fewer security associations
- **Mobility support** for roaming clients

### NHRP Authentication Strategy
NHRP authentication provides:
- **Spoke verification** before NHS registration
- **Rogue device prevention** through shared secret
- **Network integrity** by controlling DMVPN membership
- **Simple management** with single password across deployment

### EIGRP over DMVPN Optimization
EIGRP configuration requires specific tuning for DMVPN:
- **Split-horizon disabled** on hub tunnel interfaces to advertise spoke routes
- **Next-hop-self disabled** to preserve original next-hop for Phase 3
- **Named mode** provides better scalability and features
- **Dynamic neighbor discovery** via multicast over NHRP mappings

### Migration from MPLS L3VPN
Moving from MPLS to DMVPN provides:
- **Cost reduction** by leveraging Internet connectivity
- **Simplified management** without service provider dependencies
- **Direct spoke-to-spoke** communication without MPLS backbone
- **Flexibility** to add sites without provider involvement

## DMVPN Phase 3 Mechanics

### Hub Operations
1. **Spoke registration** via NHRP registration requests
2. **Route advertisement** with original next-hop information
3. **NHRP redirect** when detecting suboptimal spoke-to-hub-to-spoke traffic
4. **NHS functionality** maintaining NHRP database of spoke mappings

### Spoke Operations
1. **NHS registration** with both hub routers
2. **Route learning** from hubs via EIGRP
3. **NHRP shortcut** processing redirect messages from hubs
4. **Dynamic tunnel** creation directly to other spokes as needed

### Spoke-to-Spoke Communication Flow
1. **Initial traffic** flows spoke → hub → spoke
2. **Hub sends redirect** to source spoke with destination spoke NBMA address
3. **Source spoke** performs NHRP resolution for destination
4. **Direct tunnel** established between spokes
5. **Subsequent traffic** flows directly spoke-to-spoke


## Verification Commands Used

```cisco
! DMVPN Status Verification
show dmvpn
show ip nhrp
show ip nhrp nhs
show ip nhrp shortcut

! Tunnel Interface Verification
show interface tunnel0
show ip interface brief | include Tunnel
show ip route | include Tunnel

! NHRP Verification
show ip nhrp brief
show ip nhrp multicast
show ip nhrp traffic
debug nhrp

! EIGRP over DMVPN Verification
show ip eigrp neighbors
show ip eigrp topology
show ip route eigrp
show ip eigrp interfaces detail Tunnel0

! Connectivity Testing
ping [remote-site-tunnel-ip] source tunnel0
traceroute [remote-site-lan] source [local-lan]
```
