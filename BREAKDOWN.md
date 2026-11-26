****
**Physical Layout Setup**
- 5 Pcs per department (4x5)
	- Sales pcs 1-5 on Left switches ports fa/1-5
	- Accounting pcs 1-5 on Left switches ports fa/6-10
	- IT

- 2 2960 switch 1 for every 2 department 
	- 1 2960 switch connecting both switches 

- 1 PT8200 router connected to second layer switch on G0/0/0 port

- DNS server connected to Middle Switch port 0/24

- Webserver Connected to middle switch port 0/23

- 1 printer for each department (1X4)
	- sales - Left Switch Port 23
	- Accounting - Left Switch port 22
	- IT - Right Switch Port 23
	- Management - Right Switch port 22

- 2911 router connected to PT8200 and labeled as ISP router

- 1 PT-Server connected to ISP router to act as external web server

---
**Configuration Setup**

- Create VLAN for each department where switch is connected too.
	- VLAN 10 - Sales
	- VLAN 20 - Accounting
	- VLAN 30 - Management
	- VLAN 40 - IT
	- *Cisco Switch Command*
```
enable
configure terminal

vlan 10
 name SALES
exit
```

- Create all VLAN once again on Middle Switch and VLAN for DNS-Server end device. 
	- VLAN 50 - DNS-Server

- Assign all Lower layer switch ports connected to department devices to respective VLAN and allow access mode.
	- *Cisco Switch Command example* 
```
configure terminal

interface range fa0/1 - 5
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
exit
```

- Configure trunk mode on all switches where ports are used to connect to router and higher or lower layer switches.
	- *Cisco Switch Command example*
```
configure terminal

interface range fa0/22 
 switchport mode trunk
exit
```

- Configure router port where the layer 2 switch connects to prevent shutdown on start
	- *Cisco Switch Command example*
```
enable
configure terminal

interface g0/0
 no shutdown
exit
```

- Configure Router to create gateway for each VLAN created
	- `encapsulation dot1Q` tells router which VLAN tag belongs to sub interface
	- *Cisco Switch Command example*
```
interface g0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
exit
```

- Configure DHCP to reserve addresses 1-9 for static devices. (ignore VLAN for DNS server)
	- *Cisco Switch Command example*
```
ip dhcp excluded-address 192.168.10.1 192.168.10.9
ip dhcp excluded-address 192.168.20.1 192.168.20.9
ip dhcp excluded-address 192.168.30.1 192.168.30.9
ip dhcp excluded-address 192.168.40.1 192.168.40.9
```

- Create a DHCP pool for each department. (ignore VLAN for DNS server)
	- *Cisco Switch Command example*
```
ip dhcp pool VLAN10
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 192.168.50.10
exit
```

- Setup DNS server/VLAN on middle Switch/ Layer 2 switch with corresponding port
	- *Cisco Switch Command example*
```
configure terminal

interface fa0/24
 switchport mode access
 switchport access vlan 50
 spanning-tree portfast
exit
```

- enable a bpdugaurd on the layer two switch for the port where the DNS server is connected to prevent errors if another switch is added in the future
	- *Cisco Switch Command example*
```
interface fa0/10
 spanning-tree bpduguard enable
exit
```

- configure ISP router for upstream gateway for all pcs (typically ISP provider gives information)
	- *Cisco Switch Command example*
```
enable
configure terminal

interface g0/1
 ip address 198.51.100.2 255.255.255.252
 no shutdown
exit
```

- configure company router to send unknown destinations (e.g. Google.com) towards the ISP.
	- *Cisco Switch Command example*
```
enable
configure terminal

interface g0/1
 ip address 198.51.100.2 255.255.255.252
 no shutdown
exit

ip route 0.0.0.0 0.0.0.0 198.51.100.1
```

- mark VLAN as inside on router for NAT configuration 
	- *Cisco Switch Command example*
```
configure terminal

interface g0/0.10
 ip nat inside
exit
```

- mark port for ISP router as outside
	- *Cisco Switch Command example*
```
interface g0/1
 ip nat outside
exit
```

- Allow VLAN Network to use NAT
	- *Cisco Switch Command example*
```
access-list 1 permit 192.168.10.0 0.0.0.255
access-list 1 permit 192.168.20.0 0.0.0.255
access-list 1 permit 192.168.30.0 0.0.0.255
access-list 1 permit 192.168.40.0 0.0.0.255
access-list 1 permit 192.168.50.0 0.0.0.255
access-list 1 permit 192.168.60.0 0.0.0.255
```

- enable NAT overload
	- *Cisco Switch Command example*
```
ip nat inside source list 1 interface g0/1 overload
```

- Because Packet Tracer simulated networks and can not access real world network servers. we have to create out own using a server connected to the ISP.
- Configure ISP to connect to external server
	- *Cisco Switch command example*
```
enable
configure terminal

interface g0/1
 ip address 203.0.113.1 255.255.255.0
 no shutdown
exit
end
```

- Configure server IP address
	- server -> desktop -> IP configuration
		- IP Address `203.0.113.10` 
		- Subnet Mask `255.255.255.0` 
		- Default Gateway `203.0.113.1`
	- Service -> HTTP
		- Ensure HTTP is ON
	- select index.html
		- edit and add following for test purposes
```
<html>
<head>
    <title>Test Website</title>
</head>
<body>
    <h1>Welcome to the Internet!</h1>
    <p>External Web Server is working.</p>
</body>
</html>
```

- Configure DNS services to On and add web server to record
	- Services -> DNS (DNS ON)
	- add record web server
		- Name `internet.test`
		- Type `A Record`
		- Address `203.0.113.10`

- configure the each switch that a AC-PT is connected to create a new vlan
```
enable
configure terminal

vlan 60
 name WIFI
exit

interface fa0/21
 description ACCESS-POINT-1
 switchport mode access
 switchport access vlan 60
 spanning-tree portfast
exit
```

- reconfigure switch to allow VLAN 60
```
enable
configure terminal
interface fa0/24
 switchport trunk allowed vlan 10,20,60
exit
```

- reconfigure middle switch to allow VLAN 60
```
enable
configure terminal

vlan 60
 name WIFI
exit
```

- configure each AC-PT 
	- Access Point-PT -> Config -> Port 1
		- SSID `Company-Wifi`
		- Authentication `WPA2-PSK`
		- Encryption `AES`
		- PSK Pass Phrase `WifiTest12`
		- Channel (2.4Hz) AC-PT1 - `1` AC-PT2 - `6`

- Configure WIFI into the DHCP pool
	- *router Command example*
```
ip dhcp excluded-address 192.168.60.1 192.168.60.19

ip dhcp pool WIFI
 network 192.168.60.0 255.255.255.0
 default-router 192.168.60.1
 dns-server 192.168.50.10
exit
```

- ensure VLAN 60 has a gateway
	- *router Command Example*
```
interface g0/0.60
 encapsulation dot1Q 60
 ip address 192.168.60.1 255.255.255.0
exit

interface g0/0
 no shutdown
exit
end
```

- Connect Laptops to WIFI
	- using Desktop -> pc wireless connect using information
	- or use config -> wireless setting 
		- SSID `Company-Wifi`
		- Authentication `WPA2-PSK`
		- Encryption `AES`
		- PSK Pass Phrase `WifiTest12`

- Restrict router access to only IT
	- *Cisco Command example*
```
enable
configure terminal

ip access-list standard ROUTER_ADMIN
 permit 192.168.40.0 0.0.0.255
exit

line vty 0 4
 access-class ROUTER_ADMIN in
 transport input ssh
 login local
exit
```

- Enable SSH to router with encryption
	- *Cisco Router Command example*
```
configure terminal
hostname COMPANY-RTR
ip domain-name company.test
username admin privilege 15 secret RouterPassword

crypto key generate rsa
2048
ip ssh version 2

line vty 0 4
 login local
 transport input ssh
 access-class ROUTER_ADMIN in
exit

end
copy running-config startup-config
```

- Restrict WIFI VLAN 60 access to the DNS server by configuring router ACL
	- *Cisco Router Command example*
```
enable
configure terminal

ip access-list extended WIFI_SECURE
 permit udp any host 192.168.50.10 eq 53
 permit tcp any host 192.168.50.10 eq 53

 deny ip any 192.168.10.0 0.0.0.255
 deny ip any 192.168.20.0 0.0.0.255
 deny ip any 192.168.30.0 0.0.0.255
 deny ip any 192.168.40.0 0.0.0.255
 deny ip any 192.168.50.0 0.0.0.255

 permit ip any any
exit

configure terminal

interface g0/0.60
 ip access-group WIFI_SECURE in
exit

end
```

