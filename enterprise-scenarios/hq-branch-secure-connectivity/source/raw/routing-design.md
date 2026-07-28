# Routing Design



### 1\. Routing Architecture



The enterprise network uses a hierarchical routing design:



* OSPF is used as the internal routing protocol.
* BGP is used for external connectivity and inter-domain routing.
* GRE over IPsec is used for secure inter-site connectivity.
* Loopback interfaces are used as stable router identifiers.
* Point-to-point links are used between routed infrastructure devices.



### 2\. OSPF Design



#### HQ OSPF Domain



OSPF is configured between:



* HQ-EDGE1
* HQ-EDGE2
* HQ-CORE1
* HQ-CORE2



All internal HQ links and enterprise loopbacks are advertised through OSPF.



#### Branch OSPF Domain



OSPF is configured between:



* BR-EDGE1
* BR-CORE1
* BR-CORE2



All internal Branch links and loopbacks are advertised through OSPF.



#### OSPF Security



OSPF authentication is enabled on routed interfaces to prevent unauthorized adjacency formation and route injection.



#### OSPF Verification



show ip ospf neighbor

show ip ospf interface brief

show ip route ospf

show ip protocols



### 3\. BGP Design



BGP is used for:



* Enterprise-to-ISP connectivity
* ISP internal route exchange
* Internet route propagation
* Inter-site route exchange over the GRE tunnel



**The enterprise AS numbers are:**



* HQ: AS 65001
* Branch: AS 65002



**The ISP AS numbers are:**



* ISP 1: AS 100
* ISP 2: AS 200



### 4\. Inter-Site Routing



HQ and Branch exchange routes through the GRE over IPsec tunnel.



**Expected behavior:**



* HQ internal prefixes are reachable from Branch.
* Branch internal prefixes are reachable from HQ.
* Inter-site traffic uses the tunnel.
* Inter-site traffic is excluded from NAT.
* BGP neighbor authentication is enabled across the tunnel.



### 5\. Default Route Strategy



Internet-facing edge routers receive or advertise a default route according to the lab design.



The enterprise routers use BGP-learned or statically configured default routing toward the ISP.



**Validation commands:**





* show ip route 0.0.0.0
* show ip bgp 0.0.0.0
* traceroute 8.8.8.8



### 6\. Routing Verification



* show ip route
* show ip route connected
* show ip route ospf
* show ip route bgp
* show ip cef
* ping <destination> source Loopback0
* traceroute <destination>























