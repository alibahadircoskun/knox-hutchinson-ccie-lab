# Internet Network - BGP Configuration

## Overview
Configured complex BGP relationships across multiple autonomous systems with advanced features including confederations, route filtering, traffic engineering, and authentication.

## Requirements Completed

### Basic BGP Setup
- SP-100: ASN 100
- SP-200: ASN 200  
- SP-300: ASN 300
- ISP-400: ASN 400 (with confederation)

### BGP Confederation Configuration
- ISP-400-1 & ISP-400-2: Sub-AS 65005 with authentication "CISCO"
- ISP-400-3 & ISP-400-4: Sub-AS 65006 with authentication "CISCO1"

### Advanced Features Implemented
1. eBGP relationships between directly connected neighbors in different ASNs
2. ISP-300 neighbor verification (single-hop TTL security)
3. Loopback advertisements into BGP
4. Customer route advertisements with proper origination
5. ISP-400-1 loopback filtering (not advertised outside ASN 400)
6. ISP-300 selective advertisement (only loopback to ISP-400-4)
7. ISP-400-4 route filtering (ISP-300's loopback stays within local ASN)
8. ISP-100 default route propagation
9. Route summarization (ASN 200/300 summarizing ASN 400 loopbacks to ASN 100)
10. Traffic engineering (ASN 400 ingress via ISP-400-4)
11. Specific routing (ISP-400-2's loopback via ISP-300)
12. BGP soft reset procedures

## Configuration Examples

### BGP Confederation
```cisco
! ISP-400-1 (Sub-AS 65005)
router bgp 65005
 bgp confederation identifier 400
 bgp confederation peers 65006
 neighbor [peer-ip] remote-as 65006
 neighbor [peer-ip] password CISCO
 
! ISP-400-3 (Sub-AS 65006)  
router bgp 65006
 bgp confederation identifier 400
 bgp confederation peers 65005
 neighbor [peer-ip] remote-as 65005
 neighbor [peer-ip] password CISCO1
```

### TTL Security
```cisco
router bgp 300
 neighbor [peer-ip] ttl-security hops 1
```

### Route Summarization
```cisco
! ASN 200/300 summarizing ASN 400 routes
router bgp [200|300]
 aggregate-address [summary-network] [summary-mask] summary-only
```

### Route Filtering
```cisco
! Preventing ISP-400-1 loopback from being advertised externally
ip prefix-list FILTER_LOOPBACK deny [loopback-network]/32
ip prefix-list FILTER_LOOPBACK permit 0.0.0.0/0 le 32
router bgp 65005
 neighbor [external-peer] prefix-list FILTER_LOOPBACK out
```

## Challenges Encountered

### Confederation Complexity
- Establishing proper peer relationships between sub-ASNs
- Ensuring authentication worked correctly across confederation boundaries
- Understanding confederation identifier vs. sub-AS numbers

### Route Advertisement Control
- Implementing selective advertisement policies
- Balancing route filtering with connectivity requirements
- Traffic engineering with BGP attributes

## Verification Commands Used

```cisco
! BGP Status and Neighbors
show ip bgp summary
show ip bgp neighbors 172.30.103.100
show ip bgp confederation

! Route Information  
show ip bgp
show ip bgp 141.141.141.141
show ip route bgp

! Troubleshooting
debug ip bgp updates
clear ip bgp * soft out
clear ip bgp 172.30.103.100 soft out
ping 100.100.100.100 source loopback0
```

## Lessons Learned

1. **Confederation Benefits**: Solves iBGP full-mesh scaling issues while maintaining external AS appearance
2. **Route Filtering Precision**: Prefix-lists and route-maps provide granular control over advertisements
3. **Traffic Engineering**: Multiple techniques (AS-path prepending, local preference, MED) can influence path selection
4. **Authentication Importance**: BGP passwords prevent unauthorized peering relationships
5. **Soft Reset Value**: Allows policy changes without disrupting established sessions

---

*Configuration files and verification outputs stored in respective directories*
