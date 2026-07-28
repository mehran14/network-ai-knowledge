\---

title: "OSPF Hello and Dead Timers"
vendor: "Cisco"
technology: "OSPF"
platform: "Cisco IOS and IOS XE"
software\_version: "Version-independent"
document\_type: "concept"
source\_type: "official\_documentation"
source\_title: "IP Routing: OSPF Configuration Guide, Cisco IOS XE 17.x"
source\_url: "https://www.cisco.com/c/en/us/td/docs/routers/ios/config/17-x/ip-routing/b-ip-routing/m\_iro-cfg-0.html"
verified: true
last\_reviewed: "2026-07-28"
tags:

* ospf
* routing
* hello
* dead-interval
* neighbor
* adjacency
* cisco-ios
* cisco-ios-xe

\---

# OSPF Hello and Dead Timers

### Summary

OSPF routers use Hello packets to discover neighbors, verify bidirectional communication, maintain neighbor relationships, and support Designated Router (DR) and Backup Designated Router (BDR) election on multiaccess networks.

The Hello interval controls how frequently an OSPF interface sends Hello packets. The Dead interval defines how long a router waits without receiving a valid Hello from a neighbor before declaring that neighbor down.

For an OSPF neighbor relationship to form and remain stable, the Hello and Dead intervals must match between routers connected to the same network segment.

### Why Hello and Dead Timers Matter

Hello and Dead timers directly affect neighbor discovery, failure detection, and adjacency stability.

Shorter timers can detect failures more quickly but increase OSPF control-plane activity. Longer timers reduce Hello packet frequency but delay detection of a failed neighbor. Timer changes must therefore be planned and applied consistently to every OSPF router on the shared link.

A timer mismatch causes received Hello packets to be rejected. The routers may have IP connectivity and compatible OSPF area settings but still fail to become neighbors.

### Hello Interval

The Hello interval is the time, in seconds, between Hello packets sent from an OSPF-enabled interface.

Hello packets perform several functions:

* Discover OSPF neighbors.
* Maintain existing neighbor relationships.
* Confirm bidirectional communication through the neighbor list.
* Advertise interface parameters required for neighbor compatibility.
* Participate in DR and BDR election on broadcast and NBMA networks.

On Cisco IOS and IOS XE, the interface-level command used to change the Hello interval is:

```text
ip ospf hello-interval seconds
```

### Dead Interval

The Dead interval is the maximum time a router waits to receive a valid Hello packet from a neighbor. If the timer expires, OSPF declares the neighbor down and removes the associated adjacency.

Each valid Hello received from the neighbor resets the inactivity timer. The `Dead Time` field in `show ip ospf neighbor` displays the remaining time before the neighbor is declared down.

On Cisco IOS and IOS XE, the interface-level command used to change the Dead interval is:

```text
ip ospf dead-interval seconds
```

The default Dead interval is commonly four times the default Hello interval. When timers are manually configured, do not assume that changing one timer automatically changes the other; verify both values explicitly.

### Default Timer Values

Default OSPF timer values depend on the OSPF network type associated with the interface.

|OSPF Network Type|Hello Interval|Dead Interval|
|-|-:|-:|
|Broadcast|10 seconds|40 seconds|
|Point-to-point|10 seconds|40 seconds|
|Non-Broadcast Multi-Access (NBMA)|30 seconds|120 seconds|

Ethernet interfaces normally use the broadcast OSPF network type. Serial links using HDLC or PPP normally operate as point-to-point networks. Legacy technologies such as Frame Relay can operate as NBMA networks.

The effective values must always be verified on the actual interface because the network type or timers may have been manually changed.

### How Timers Affect Neighbor Formation

OSPF Hello packets carry both the HelloInterval and RouterDeadInterval values. When a router receives a Hello packet, it compares these values with the values configured on the receiving interface.

For OSPFv2, RFC 2328 specifies that the received HelloInterval and RouterDeadInterval must match the interface values. If either value differs, processing of the Hello packet stops.

A simplified neighbor-discovery sequence is:

1. An OSPF interface sends Hello packets at its configured Hello interval.
2. A neighboring router receives a Hello packet and validates required parameters.
3. If the parameters are compatible, the receiving router records the sender as a neighbor.
4. Bidirectional communication is confirmed when each router sees its own Router ID in the neighbor field of the other router's Hello packet.
5. Each accepted Hello resets the neighbor inactivity timer.
6. If no valid Hello arrives before the Dead interval expires, the neighbor transitions to the Down state.

In addition to timers, OSPFv2 Hello processing checks parameters such as the network mask and area-related option compatibility. Matching timers alone does not guarantee adjacency formation.

### Timer Mismatch and Common Problems

A Hello or Dead interval mismatch commonly prevents neighbors from progressing beyond the early stages of neighbor formation.

Typical symptoms include:

* No neighbor entry appears in `show ip ospf neighbor`.
* A neighbor relationship repeatedly forms and drops.
* Syslog or debug output reports mismatched Hello or Dead parameters.
* One router declares the neighbor down after its inactivity timer expires.
* Basic IP reachability works, but OSPF adjacency does not form.

Common causes include:

* A timer was changed on only one side of a link.
* The two interfaces use different OSPF network types.
* A configuration template was applied inconsistently.
* A migrated or replaced router retained different interface defaults.
* Fast timers were introduced without changing every router on a shared segment.

Because network type determines default timers, an apparent timer mismatch can be a symptom of a network-type mismatch. Verify both settings.

### Configuration Example

The following example changes the Hello interval to 5 seconds and the Dead interval to 20 seconds:

```text
interface GigabitEthernet0/0
 ip ospf hello-interval 5
 ip ospf dead-interval 20
```

Equivalent timer values must be configured on the neighboring OSPF interface:

```text
interface GigabitEthernet0/0
 ip ospf hello-interval 5
 ip ospf dead-interval 20
```

On a broadcast segment containing more than two OSPF routers, the compatible settings must be applied to all participating OSPF interfaces on that segment.

### Basic Verification

Use the following commands to verify OSPF timers and neighbor status:

```text
show ip ospf interface GigabitEthernet0/0
show ip ospf neighbor
show ip ospf neighbor detail
show running-config interface GigabitEthernet0/0
```

`show ip ospf interface` displays the configured Hello and Dead intervals together with the interface network type. A typical relevant line is:

```text
Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
```

`show ip ospf neighbor` displays the current neighbor state and remaining Dead Time:

```text
Neighbor ID     Pri   State           Dead Time   Address       Interface
2.2.2.2           1   FULL/DR         00:00:33    10.0.12.2    GigabitEthernet0/0
```

|Field|Meaning|
|-|-|
|Hello|Interval between locally transmitted Hello packets|
|Dead|Maximum time without a valid Hello before the neighbor is declared down|
|Dead Time|Remaining inactivity time for a specific neighbor|
|Network Type|OSPF interface type that influences default timers and adjacency behavior|

Compare the output from both sides of the link. The Hello and Dead intervals must match; the OSPF network type should also be intentionally selected and operationally compatible.

### Interpreting Common Timer-Related Problems

If no neighbor is present, verify the interface-level timers on both routers first:

```text
show ip ospf interface interface-id
```

If a neighbor repeatedly transitions between Up and Down states, check for intermittent packet loss, interface errors, congestion, CPU pressure, or control-plane filtering in addition to timer consistency.

Debug commands can provide direct evidence of rejected Hello packets, but they should be used carefully on production systems:

```text
debug ip ospf hello
debug ip ospf adj
```

Disable debugging after collecting the required information:

```text
undebug all
```

A stable `FULL` state and a repeatedly resetting `Dead Time` indicate that valid Hello packets are being received. However, `FULL` confirms LSDB synchronization with that neighbor; it does not by itself confirm that every expected OSPF route is installed in the routing table.

### Common Misconceptions

* **The Dead Time shown in neighbor output is a fixed configured value.** It is a countdown that resets whenever a valid Hello is received.
* **Changing the Hello interval always changes the Dead interval automatically.** Treat the two interface settings separately and verify both after configuration.
* **A Dead interval must always equal four times the Hello interval.** The common defaults use a 4:1 ratio, but neighbor formation fundamentally requires matching values between peers.
* **Matching timers guarantee a Full adjacency.** Authentication, area, subnet, network type, MTU, and other OSPF parameters can still prevent neighbor formation or LSDB synchronization.
* **The lowest possible timers are always best.** Aggressive timers increase sensitivity to transient loss and control-plane delays and should be introduced only after considering platform capacity and link quality.
* **Only two routers need matching timers on a broadcast network.** Every OSPF router participating on the shared segment must use compatible Hello parameters.

### References

1. Cisco, "IP Routing: OSPF Configuration Guide, Cisco IOS XE 17.x."  
https://www.cisco.com/c/en/us/td/docs/routers/ios/config/17-x/ip-routing/b-ip-routing/m\_iro-cfg-0.html
2. Cisco, "IP Routing: OSPF Configuration Guide, Cisco IOS XE 16.x."  
https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/iproute\_ospf/configuration/xe-16/iro-xe-16-book/iro-cfg.html
3. J. Moy, "OSPF Version 2," RFC 2328, Sections 7.1, 10.5, and A.3.2, Internet Engineering Task Force, April 1998.  
https://www.rfc-editor.org/rfc/rfc2328.html

