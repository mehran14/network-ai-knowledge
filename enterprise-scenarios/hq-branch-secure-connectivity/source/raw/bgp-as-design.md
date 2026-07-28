# BGP and Autonomous System Design



### 1\. Autonomous Systems



| AS | Organization | Routers |

|---:|---|---|

| 100 | ISP 1 | ISP-R1, ISP-R5 |

| 200 | ISP 2 | ISP-R2, ISP-R3, ISP-R4 |

| 65001 | HQ Enterprise | HQ-EDGE1, HQ-EDGE2 |

| 65002 | Branch Enterprise | BR-EDGE1 |



### 2\. ISP1 – AS 100



ISP 1 consists of:



* ISP-R1
* ISP-R5



The routers inside AS 100 use iBGP for internal BGP route exchange.



External eBGP sessions are established with neighboring autonomous systems.





### 3\. ISP2 – AS 200



ISP 2 consists of:



* ISP-R2
* ISP-R3
* ISP-R4



ISP-R4 serves as an internal core/transit router and also operate as a Route Reflector, base on the final BGP implementation.



The use of a Route Reflector reduces the requirement for a full-mesh iBGP topology.







### 4\. Enterprise BGP



#### HQ



HQ uses AS 65001.



An iBGP session is established between:

HQ-EDGE1 ↔ HQ-EDGE2



HQ edge routers establish eBGP sessions toward the ISP infrastructure.



#### Branch



Branch uses AS 65002.



**BR-EDGE1 establishes:**

* eBGP toward ISP-R3
* eBGP toward HQ-EDGE1 over the GRE tunnel



### 5\. BGP Security



BGP neighbors are protected using TCP MD5 authentication.



**Additional controls include:**

* Explicit neighbor activation
* Prefix filtering
* Maximum-prefix limits where applicable
* Proper next-hop handling
* Controlled route advertisement
* Local preference and MED policies where required



### 6\. BGP Verification



show ip bgp summary

show ip bgp neighbors

show ip bgp

show ip route bgp

show ip bgp <prefix>



Expected BGP neighbor state:

Established





### 7\. BGP Troubleshooting Checklist



**If a BGP session is not Established, verify:**



1. IP reachability between neighbors.
2. Correct remote-AS configuration.
3. Correct update-source configuration.
4. Correct source interface or tunnel address.
5. TCP/179 reachability.
6. BGP authentication keys.
7. eBGP multihop configuration, if required.
8. Route availability to loopback-based neighbors.
9. ACLs or control-plane policies.
10. Duplicate or incorrect router IDs.



**Useful commands:**



ping <neighbor-ip> source <source-interface>

telnet <neighbor-ip> 179

show tcp brief

show ip bgp neighbors

show access-lists

