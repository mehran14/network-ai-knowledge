\---

title: "OSPF Overview"

vendor: "Cisco"

technology: "OSPF"

platform: "Cisco IOS and IOS XE"

software\_version: "Version-independent"

document\_type: "concept"

source\_type: "official\_documentation"

source\_title: "OSPF Configuration Guide — Cisco IOS XE"

source\_url: "https://www.cisco.com/c/en/us/td/docs/switches/lan/c9000/lyr3-fwd/ospf/ospf-configuration-guide/ospf.html"

verified: true

last\_reviewed: "2026-07-28"

tags:

&#x20; - ospf

&#x20; - routing

&#x20; - igp

&#x20; - link-state

&#x20; - cisco-ios

&#x20; - cisco-ios-xe

\---



# OSPF Overview



### Summary



Open Shortest Path First (OSPF) is a link-state Interior Gateway Protocol (IGP) used to exchange routing information within an Autonomous System (AS).



OSPF routers build a link-state database, calculate the shortest paths with the Shortest Path First (SPF) algorithm, and install the selected routes in the routing table.



### Key Characteristics



* OSPF is a link-state routing protocol.
* It uses the SPF algorithm to calculate the best paths.
* Its administrative distance is 110 on Cisco IOS and IOS XE.
* Its IP protocol number is 89.
* OSPF supports hierarchical network design through areas.
* OSPF supports classless routing and Variable Length Subnet Masks (VLSM).
* OSPF sends triggered updates when topology changes occur.
* OSPF maintains neighbor relationships with adjacent routers.



### OSPF Versions



| Version | Address Family | Description |

|---|---|---|

| OSPFv2 | IPv4 | Used to exchange IPv4 routing information |

| OSPFv3 | IPv6 and supported address families | Originally designed for IPv6 and extended through address-family support |



OSPFv2 and OSPFv3 have similar link-state principles, but they differ in packet handling, addressing, authentication mechanisms, and configuration behavior.



### Core OSPF Components



#### Neighbor Table



The neighbor table records directly connected OSPF routers that have been discovered through OSPF Hello packets.



Useful command:



show ip ospf neighbor



#### Link-State Database

The Link-State Database (LSDB) contains topology information learned through Link-State Advertisements (LSAs).

Routers in the same OSPF area are expected to maintain synchronized link-state information after adjacency formation and database exchange complete successfully.



Useful command:



show ip ospf database



#### Routing Table

OSPF runs the SPF algorithm against the LSDB and submits its selected routes to the Routing Information Base (RIB).



Useful command:



show ip route ospf



#### OSPF Router ID

The OSPF router ID is a 32-bit identifier written in IPv4 dotted-decimal format, for example:



1.1.1.1



The router ID identifies an OSPF router within the OSPF routing domain.



It does not need to be a reachable IPv4 address, but it must be unique.



On Cisco IOS and IOS XE, the router ID can be configured explicitly:



router ospf 10

&#x20;router-id 1.1.1.1



Explicit router ID configuration is preferred because it provides



predictable behavior.



#### OSPF Areas

OSPF uses areas to create a hierarchical routing design.



The backbone area is:



Area 0



Other OSPF areas should normally connect to Area 0 directly or through



an appropriate virtual-link design when required.



Area design can:



* Reduce the size of the LSDB.
* Reduce the scope of SPF calculations.
* Limit the propagation of some topology changes.
* Support route summarization at area boundaries.



### OSPF Router Roles

Depending on interface and area placement, an OSPF router can have one or more roles.





| Role | Description |

|---|---|

| Internal Router | All OSPF interfaces belong to the same area |

| Backbone Router | At least one OSPF interface belongs to Area 0 |

| Area Border Router (ABR) | Connects Area 0 to one or more non-backbone areas |

| Autonomous System Boundary Router (ASBR) | Redistributes routes from another routing source into OSPF |

| Designated Router (DR) | Represents a multiaccess network and reduces adjacency requirements |

| Backup Designated Router (BDR) | Provides backup functionality for the Designated Router |



### OSPF Metric

OSPF uses cost as its routing metric.



A lower total path cost is preferred over a higher total path cost.



The cost of an interface is commonly derived from:



Reference bandwidth / Interface bandwidth



Because modern interfaces may have much higher speeds than the default reference bandwidth, the reference bandwidth should be reviewed and configured consistently across the OSPF domain.



###### Example:



router ospf 10

&#x20;auto-cost reference-bandwidth 100000



The value is expressed in Mbps on Cisco IOS and IOS XE.



### Basic OSPF Operation

A simplified OSPF process is:



1. OSPF-enabled routers send Hello packets.
2. Compatible routers become neighbors.
3. Eligible neighbors form adjacencies.
4. Adjacent routers exchange link-state information.
5. Routers synchronize their LSDBs.
6. Each router runs the SPF algorithm.
7. Selected OSPF routes are submitted to the routing table.
8. Topology changes trigger new link-state information and SPF processing.



### Basic Configuration Example



router ospf 10

&#x20;router-id 1.1.1.1

&#x20;network 10.0.12.0 0.0.0.255 area 0



In this example:

* 10 is the locally significant OSPF process ID.
* 1.1.1.1 is the OSPF router ID.
* The network statement enables OSPF on matching interfaces.
* Matching interfaces are assigned to Area 0.



### Basic Verification



show ip ospf

show ip ospf interface brief

show ip ospf neighbor

show ip ospf database

show ip route ospf



These commands help verify:



* OSPF process operation
* Router ID
* Enabled interfaces
* Area assignment
* Neighbor relationships
* LSDB contents
* Installed OSPF routes



### Common Misconceptions



"The OSPF Process ID Must Match Between Routers" Incorrect.



The OSPF process ID is locally significant on Cisco IOS and IOS XE. Neighboring routers can use different process IDs.



"All OSPF Neighbors Must Reach the FULL State" Incorrect.



On broadcast and non-broadcast multiaccess networks, DROTHER routers normally reach the FULL state with the DR and BDR but may remain in the 2-WAY state with other DROTHER routers.



"A Network Statement Advertises a Network Directly" Not exactly.



On Cisco IOS and IOS XE, the OSPF network statement matches local interface addresses. OSPF is then enabled on matching interfaces, and those interfaces are assigned to the specified area.



### Related Topics

* OSPF neighbor states
* OSPF Hello and Dead timers
* OSPF network types
* OSPF Designated Router and Backup Designated Router
* OSPF Link-State Advertisements
* OSPF area types
* OSPF route types
* OSPF authentication
* OSPF configuration
* OSPF troubleshooting



### References

1\. \[Cisco — OSPF Configuration Guide, Cisco IOS XE](https://www.cisco.com/c/en/us/td/docs/switches/lan/c9000/lyr3-fwd/ospf/ospf-configuration-guide/ospf.html)

2\. \[IETF RFC 2328 — OSPF Version 2](https://www.rfc-editor.org/rfc/rfc2328)

3\. \[IETF RFC 5340 — OSPF for IPv6](https://www.rfc-editor.org/info/rfc5340/)

