\---

title: "OSPF Neighbor States"

vendor: "Cisco"

technology: "OSPF"

platform: "Cisco IOS and IOS XE"

software\_version: "Version-independent"

document\_type: "concept"

source\_type: "official\_documentation"

source\_title: ""

source\_url: ""

verified: false

last\_reviewed: ""

tags:

&#x20; - ospf

&#x20; - routing

&#x20; - neighbor

&#x20; - adjacency

&#x20; - neighbor-states

&#x20; - cisco-ios

&#x20; - cisco-ios-xe

\---



# OSPF Neighbor States



### Summary



OSPF routers use a defined state machine to discover neighbors, verify parameter compatibility, establish adjacencies, and synchronize their Link-State Databases (LSDBs).



For OSPFv2, a neighbor relationship can progress through these states:



* Down
* Attempt
* Init
* 2-Way
* ExStart
* Exchange
* Loading
* Full



The current neighbor state provides important information about where the adjacency formation process has reached and where a failure may be occurring.



### Neighbor and Adjacency

An OSPF neighbor is another OSPF router discovered through Hello packets on a common network segment.



An OSPF adjacency is a more advanced relationship in which eligible neighbors exchange and synchronize link-state information.



Therefore:



* Every adjacent router is an OSPF neighbor.
* Not every OSPF neighbor necessarily becomes fully adjacent.
* The expected final state depends on the OSPF network type and the roles of the routers on the segment.



### OSPF Neighbor State Sequence

A typical adjacency formation process follows this sequence:



Down → Init → 2-Way → ExStart → Exchange → Loading → Full



The 'Attempt' state is primarily associated with manually configured neighbors on Non-Broadcast Multi-Access (NBMA) networks.



#### Down State

The 'Down' state indicates that no valid Hello packet has been received from the neighbor within the Dead interval.



This state can indicate:



* The neighbor has not yet been discovered.
* The neighbor is unreachable.
* OSPF is not enabled on the remote interface.
* Hello packets are not being received.
* The Dead timer has expired.
* An underlying Layer 1, Layer 2, or Layer 3 problem exists.



The 'Down' state is the initial state of an OSPF neighbor relationship.



#### Attempt State

The 'Attempt' state is used primarily on NBMA networks when neighbors are configured manually.



In this state, the local router actively attempts to contact a manually configured neighbor by sending unicast Hello packets.



Example of manual neighbor configuration:



router ospf 10

&#x20;neighbor 10.0.12.2



This state is not normally seen on broadcast or point-to-point networks.



#### Init State

The 'Init' state indicates that the local router has received a Hello packet from the neighbor, but the local router’s Router ID is not listed in the received Hello packet. This means one-way communication has been detected.



Common causes include:



* Unidirectional connectivity.
* Access Control List (ACL) filtering.
* Multicast delivery problems.
* Layer 2 forwarding problems.
* The remote router not receiving local Hello packets.



A neighbor repeatedly remaining in the Init state should be



investigated as a possible one-way communication problem.



#### 2-Way State

The '2-Way' state indicates bidirectional communication. The local router has received a Hello packet containing its own Router ID in the neighbor list. This confirms that both routers are receiving each other’s Hello packets.



At this stage:



* Bidirectional neighbor communication is established.
* Designated Router (DR) and Backup Designated Router (BDR) election may occur on multiaccess networks.
* The routers determine whether a full adjacency should be formed.



On broadcast and NBMA networks, two DROTHER routers normally remain in the 2-Way state with each other. This is expected behavior and does not necessarily indicate a problem.



#### ExStart State

The ExStart state begins the database exchange process.



During this state, the routers:



* Establish a master and slave relationship.
* Select the initial Database Description (DBD) sequence number.
* Prepare to exchange summaries of their LSDBs.



The router with the higher OSPF Router ID normally becomes the master during the DBD exchange process.



A neighbor stuck in ExStart may indicate:



* Maximum Transmission Unit (MTU) mismatch.
* Duplicate Router ID.
* Problems with DBD packet exchange.
* Packet loss.
* Network instability.
* Platform or software interoperability issues.



#### Exchange State

In the 'Exchange' state, adjacent routers exchange Database Description packets.



DBD packets contain summaries of the LSAs present in each router’s LSDB. They do not normally carry the complete contents of every LSA.

Each router compares the received summaries with its local LSDB and identifies missing or outdated LSAs.



A neighbor stuck in the Exchange state may indicate:



* MTU mismatch.
* Packet loss.
* Duplicate Router ID.
* DBD sequence-number problems.
* Unstable connectivity.
* Oversized or malformed packets.



#### Loading State

In the 'Loading' state, routers request the complete contents of LSAs that are missing or outdated.



The following OSPF packet types are important during this stage:



* Link-State Request (LSR)
* Link-State Update (LSU)
* Link-State Acknowledgment (LSAck)



The router remains in the Loading state until the required link-state information has been received.



A neighbor stuck in Loading may indicate:



* Link-State Requests are not being answered.
* Link-State Updates are being dropped.
* Packet loss exists.
* The LSDB contains inconsistent or problematic information.
* The adjacency is repeatedly being reset.



#### Full State

The 'Full' state indicates that the routers have synchronized their LSDBs for the adjacency.



This is normally the expected state for:



* Point-to-point neighbors.
* Point-to-multipoint neighbors.
* DR-to-BDR relationships.
* DR-to-DROTHER relationships.
* BDR-to-DROTHER relationships.



The Full state does not guarantee that every desired route is installed in the routing table. Route installation can still be affected by:



* Route preference.
* Administrative distance.
* Filtering.
* Summarization.
* Area design.
* Route type.
* Reachability to the next hop.
* Routing Information Base (RIB) selection.



### Expected States on Broadcast Networks

On an OSPF broadcast network, routers elect a DR and BDR.



The expected relationships are:





| Local Role | Remote Role | Expected State |

|---|---|---|

| DR | BDR | Full |

| DR | DROTHER | Full |

| BDR | DROTHER | Full |

| DROTHER | DROTHER | 2-Way |



A '2-Way' state between two DROTHER routers is normal because they do not establish a full adjacency directly with each other.



### OSPF Packet Types and Neighbor Formation



OSPF uses five main packet types:



| Type | Packet Name | Primary Function |

|---|---|---|

| 1 | Hello | Neighbor discovery and maintenance |

| 2 | Database Description | LSDB summary exchange |

| 3 | Link-State Request | Request specific LSAs |

| 4 | Link-State Update | Send one or more LSAs |

| 5| Link-State Acknowledgment | Acknowledge received LSAs |



The packet exchange can be summarized as:



Hello packets:

Down → Init → 2-Way



Database Description packets:

ExStart → Exchange



Link-State Request, Update, and Acknowledgment packets:

Loading → Full



### Parameters That Affect Neighbor Formation

OSPF neighbors must agree on several important parameters before an adjacency can form successfully.



Commonly relevant parameters include:



* Area ID.
* Hello interval.
* Dead interval.
* Authentication settings.
* Authentication credentials.
* Stub area flags and area-type options.
* Network connectivity.
* IP subnet compatibility where required by the network type.
* OSPF network type.
* Unique OSPF Router IDs.



Some mismatches prevent neighbor discovery, while others allow the routers to reach an intermediate state but prevent the adjacency from reaching Full.



### Basic Verification

Use the following command to display OSPFv2 neighbors:



show ip ospf neighbor



Example structure:



Neighbor ID     Pri   State           Dead Time   Address         Interface

2.2.2.2           1   FULL/DR         00:00:33    10.0.12.2   GigabitEthernet0/0



Important fields include:



| Field | Meaning |

|---|---|

| Neighbor ID | OSPF Router ID of the neighbor |

| Pri | OSPF interface priority of the neighbor |

| State | Current neighbor state and multiaccess role |

| Dead Time | Remaining time before the neighbor is declared down |

| Address | Source IP address used by the neighbor |

| Interface | Local interface on which the neighbor was discovered |



For detailed information about a specific neighbor, use:



show ip ospf neighbor detail



Useful interface verification commands include:



show ip ospf interface brief

show ip ospf interface

show ip ospf interface GigabitEthernet0/0



### Interpreting Common State Problems



| Observed State | General Interpretation |

|---|---|

| Down | No valid Hello packets are being received |

| Attempt | The router is trying to contact a manually configured NBMA neighbor |

| Init | Hello received, but bidirectional communication is not confirmed |

| 2-Way | Bidirectional communication exists; may be normal between DROTHER routers |

| ExStart | Master/slave negotiation or initial DBD exchange is not completing |

| Exchange | DBD exchange is not completing |

| Loading | Requested link-state information is not being fully received |

| Full | LSDB synchronization for the adjacency is complete |



The neighbor state identifies the phase of the failure, but it does not by itself prove the exact root cause. Additional interface, packet, timer, authentication, and LSDB checks may be required.



### Common Misconceptions



"Every OSPF Neighbor Must Reach Full", Incorrect.



On broadcast and NBMA networks, DROTHER routers normally remain in the 2-Way state with other DROTHER routers.



"A Full Adjacency Guarantees Route Installation", Incorrect.



A Full adjacency indicates LSDB synchronization between adjacent routers. Routing table installation is a separate process and depends on route calculation and RIB selection.



"ExStart Problems Are Always Caused by MTU Mismatch", Incorrect.



An MTU mismatch is a common cause, but duplicate Router IDs, packet loss, DBD negotiation problems, and software issues can also prevent the adjacency from progressing.



"The OSPF Process ID Must Match", Incorrect.



The Cisco OSPF process ID is locally significant and does not need to match between neighboring routers.



### Related Commands



show ip ospf

show ip ospf neighbor

show ip ospf neighbor detail

show ip ospf interface

show ip ospf interface brief

show ip ospf database

show ip protocols



Debug commands should be used carefully, particularly on production devices:



debug ip ospf adj

debug ip ospf hello

debug ip ospf packet



### Operational Warning



OSPF debug commands can generate significant output and may affect device performance.



Before enabling debugging on a production device:



* Assess the potential operational impact.
* Restrict debugging when the platform supports filtering.
* Send logs to an appropriate buffer or monitoring destination.
* Disable debugging immediately after collecting the required data.



To disable all active debugging:



undebug all



### Related Topics

* OSPF overview
* OSPF Hello and Dead timers
* OSPF network types
* OSPF DR and BDR election
* OSPF Database Description packets
* OSPF MTU mismatch
* OSPF adjacency troubleshooting
* OSPF Link-State Database
* OSPF packet types



### References

1. Cisco OSPF Configuration Guide
2. Cisco OSPF Command Reference
3. RFC 2328 — OSPF Version 2



