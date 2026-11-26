A solid all-around project because it covers switching, routing, DHCP, wireless, and basic security.

**Scenario:** A small company has Sales, Accounting, Management, and IT departments.

**Build:**
- 1 router
- 1 ISP router
- 2–3 switches
- 15–25 PCs
- 4 Laptops
- 2 server
- 1 wireless access point
- Printers for different departments

**Network design:**
- VLAN 10 — Sales
- VLAN 20 — Accounting
- VLAN 30 — Management
- VLAN 40 — IT
- VLAN 50 — DNS server
- VLAN 60 — WIFI
- Separate IP subnet for each VLAN
- Router-on-a-stick for inter-VLAN routing
- ISP router 

**Configure:**
- [x] VLANs and trunk ports
- [x] DHCP for each department
- [x] DNS Server 
- [x] ISP-Router
- [x] web server
- [x] Inter-VLAN routing
- [x] Wi-Fi
- [x] ACL restricting WIFI access to DNS server
- [x] SSH access for network router

**Testing:**
- [x] Ping PCs within the same VLAN
- [x] Ping between VLANs
- [x] Test DHCP
- [x] Open the internal website
- [x] Confirm unauthorized departments cannot access Accounting resources