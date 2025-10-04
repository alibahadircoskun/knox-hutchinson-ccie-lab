# Multicast - PIM Sparse Mode and BSR Configuration

## Overview
Implemented comprehensive multicast routing across Customer 2 Sites 1, 2, and the backbone network using PIM Sparse Mode with Bootstrap Router (BSR) for Rendezvous Point (RP) discovery. Configured advanced features including IGMP tracking, group-specific RP assignments, and bidirectional PIM for optimized multicast distribution.

## Tasks

### Basic Multicast Infrastructure
1. Multicast routing enabled on all Customer 2 devices (Sites 1, 2, and backbone)
2. PIM Sparse Mode configured on all active interfaces including loopbacks
3. R37 configured as Rendezvous Point using BSR (Loopback0)
4. SW21 configured for IGMP join tracking

### Multicast Group Testing and Verification
5. Stream receiver configured to join multicast group 224.40.40.1
6. Join request verification on R7 and SW21
7. Multicast traffic generation from R38's loopback to 224.40.40.1
8. RP scope limitation to specific group 224.40.40.1

### Advanced Multicast Features
9. New loopback interface created on R37 for secondary RP
10. Bidirectional PIM BSR configuration for group 224.40.40.2
11. IGMP join from R38's loopback to group 224.40.40.2
12. Multicast traffic generation from R61's loopback to 224.40.40.2

## Configuration Examples

### Global Multicast Routing

#### Enable Multicast Routing
```cisco
! All routers in Customer 2 Sites
ip multicast-routing
```

### PIM Sparse Mode Configuration

#### Interface-Level PIM Configuration
```cisco
! R37 - Rendezvous Point Configuration
interface loopback0
 ip pim sparse-mode

interface e0/2
 ip pim sparse-mode

! All other routers - Standard PIM Configuration
interface e0/0
 ip pim sparse-mode

interface loopback0
 ip pim sparse-mode
```

### Bootstrap Router (BSR) and RP Configuration

#### Initial BSR/RP Setup (R37)
```cisco
! R37 - BSR and RP Configuration
ip pim bsr-candidate loopback0 0
ip pim rp-candidate loopback0 group-list MULTICAST_GROUPS

! Access list for group 224.40.40.1
ip access-list standard MULTICAST_GROUPS
 permit 224.40.40.1
```

#### Group-Specific RP Configuration
```cisco
! R37 - Limit RP to specific group
ip pim rp-candidate loopback0 group-list SPECIFIC_GROUP

ip access-list standard SPECIFIC_GROUP
 permit 224.40.40.1 0.0.0.0
```

### Bidirectional PIM Configuration

#### Secondary Loopback and Bidir RP
```cisco
! R37 - Create new loopback for bidirectional RP
interface loopback1
 ip address 137.137.137.137 255.255.255.255
 ip pim sparse-mode

! Configure bidirectional BSR RP for group 224.40.40.2
ip pim bsr-candidate loopback1 0
ip pim rp-candidate loopback1 group-list BIDIR_GROUP bidir

ip access-list standard BIDIR_GROUP
 permit 224.40.40.2 0.0.0.0
```

### IGMP Configuration

#### IGMP Join Configuration
```cisco
! R61 - Join multicast group 224.40.40.1
interface loopback0
 ip igmp join-group 224.40.40.1

! R38 - Join multicast group 224.40.40.2
interface loopback0
 ip igmp join-group 224.40.40.2
```

### Multicast Traffic Generation

#### Ping to Multicast Groups
```cisco
! R38 - Send multicast traffic to 224.40.40.1
ping 224.40.40.1 source loopback0 repeat 100

! R61 - Send multicast traffic to 224.40.40.2
ping 224.40.40.2 source loopback0 repeat 100
```

## Design Principles

### PIM Sparse Mode Architecture
Sparse Mode assumes receivers are sparsely distributed and uses explicit join messages to build distribution trees. This approach:
- **Reduces bandwidth waste** by not flooding multicast traffic to all interfaces
- **Scales efficiently** for large networks with few receivers per group
- **Requires RP** for initial source discovery and shared tree construction
- **Supports SPT switchover** from shared tree to source-specific tree for optimized paths

### Bootstrap Router (BSR) Mechanism
BSR provides dynamic RP discovery and advertisement across the PIM domain:
- **Eliminates static RP configuration** on all routers in the domain
- **Provides RP redundancy** through candidate RP priority mechanisms
- **Automatically distributes RP information** via BSR messages
- **Simplifies management** in large multicast deployments

### Rendezvous Point (RP) Role
The RP serves as the meeting point for sources and receivers:
- **Shared tree root** where initial multicast traffic flows
- **Source registration point** where first-hop routers register new sources
- **Join aggregation point** where receiver join messages converge
- **SPT trigger point** where receivers can switch to source-specific trees

### Group-Specific RP Assignment
Limiting RP responsibility to specific groups provides:
- **Load distribution** across multiple RPs for different group ranges
- **Administrative boundaries** for different multicast applications
- **Failure isolation** where RP issues affect only specific groups
- **Policy enforcement** through controlled group-to-RP mappings

### Bidirectional PIM Benefits
Bidir-PIM optimizes many-to-many multicast applications:
- **No source registration** required at the RP
- **Symmetrical routing** with bidirectional shared trees
- **Reduced state** as no source-specific state is maintained
- **Efficient for many sources** in applications like financial data feeds

## Multicast Distribution Trees

### Shared Tree (*,G)
Initial distribution tree rooted at the RP:
- Used when receivers first join a group
- All traffic flows through the RP
- Less optimal path but simpler state

### Source-Specific Tree (S,G)
Optimized tree from source to receivers:
- Created after SPT threshold is reached
- Shortest path from source to receiver
- More optimal but requires more state

### Bidirectional Shared Tree (*,G bidir)
Shared tree used for both upstream and downstream:
- No (S,G) state creation
- Traffic flows both directions on same tree
- Optimal for many-to-many scenarios

## Verification Commands Used

```cisco
! PIM Verification
show ip pim interface
show ip pim neighbor
show ip pim rp mapping
show ip pim bsr-router

! Multicast Routing Table
show ip mroute
show ip mroute 224.40.40.1
show ip mroute 224.40.40.2
show ip mroute count

! IGMP Verification
show ip igmp groups
show ip igmp interface
show ip igmp membership

! BSR and RP Verification
show ip pim rp
show ip pim bsr-router

! Multicast Traffic Verification
show ip mroute active
show ip pim traffic
```
