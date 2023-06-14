# Network Simulation 

- STEP1:

KONYA Router'ında kullanılmak üzere 172.16.42.0/24 IP adres aralığı verilmiştir.

Bu routera bağlı SW2 ağı için 100 kullanıcı, SW3 ağı için de 60 kullanıcı olacaktır.

İlk IP’ler Gateway olacak şekilde ağ bölümlemesi,

PC’lere 2. IP adreslerini verme

172.16.42.0/24 for KONYA:

    100 user for SW2- needs 128 IP- 172.16.42.0/25
    60 user for SW3- needs 64 IP- 172.16.42.128/26
    ranges:
    172.16.42.0 - 172.16.42.127 (mask 255.255.255.128)
    172.16.42.128 - 172.16.42.191 (mask 255.255.255.192)



Command Screen:

	enable
	configure terminal
	hostname KONYA
	interface FastEthernet 0/0
	ip address 172.16.42.1 255.255.255.128
	no shutdown
	exit
	interface FastEthernet 0/1
	ip address 172.16.42.129 255.255.255.192
	no shutdown
	exit
	
PC IP Addresses:

	PC name	IP Address		        mask			gateway
	PC4		172.16.42.2		255.255.255.128		172.16.42.1
	PC5		172.16.42.130		255.255.255.192		172.16.42.129
	
- STEP2:

ANKARA-KONYA IZMIR routerları EIGRP 100 routing protocolü ile haberleşecek.

in KONYA Router	:

	enable
	configure terminal
	router eigrp 100
	network 10.0.0.0 0.255.255.255
	network 172.16.42.0 0.0.0.255
	no auto summary
	
in IZMIR Router	:

	enable
	configure terminal
	hostname IZMIR
	router eigrp 100
	network 10.0.0.0 0.255.255.255
	network 172.16.35.0 0.0.0.255
	no auto summary
	
in ANKARA Router	:

	enable
	configure terminal
	hostname ANKARA
	router eigrp 100
	network 10.0.0.0 0.255.255.255
	no auto summary
	
- STEP3:

IZMIRe bağlı switche ağ dışından da erişilebilecek şekilde telnet ile erişim

in SW1:

	enable
	configure terminal
	hostname SWITCH1
	line vty 0 4
	password switch1access
	login
	exit
	enable secret switch1access
	interface vlan1
	ip address 172.16.35.3 255.255.255.0
	no shutdown
	ip default-gateway 172.16.35.1
	exit
	wr
	
- STEP4/5/6/7:

VAN Routera bağlı switchlere yeni bir switch (SW4) eklendi.

Bu yeni  switchin hem SW6’ya hem de SW5’e bağlanması.

in SW6:

	enable
	configure terminal
	vlan 10
	name KULLANICI
	exit
	vlan 20
	name SUNUCU
	exit
	vlan 30
	name MISAFIR
	exit
	
in SW4:

	enable
	configure terminal
	interface range fa0/1-10
	switchport access vlan10
	exit
	interface range fa0/11-20
	switchport access vlan30
	
in SW5:

	enable
	configure terminal
	interface range fa0/1-20
	switchport access vlan20
	
in SW4:

	PC1 ip address: 192.168.65.5
	mask: 255.255.255.0
	gateway: 192.168.65.1
	
	MISAFIR PC ip address: 192.168.200.5
	mask: 255.255.255.0
	gateway: 192.168.200.1
	
in VAN Router:

	enable
	configure terminal
	hostname VAN
	interface fa0/0.10
	encapsulation dot1q 10
	ip address 192.168.65.1 255.255.255.0
	exit
	interface fa0/0.20
	encapsulation dot1q 20
	ip address 192.168.165.1 255.255.255.0
	exit
	interface fa0/0.30
	encapsulation dot1q 30
	ip address 192.168.200.1 255.255.255.0
	exit
	
	ip dhcp pool vlan10
	network 192.168.65.0 255.255.255.0
	default-router 192.168.65.1
	ip dhcp pool vlan20
	network 192.168.165.0 255.255.255.0
	default-router 192.168.165.1
	ip dhcp pool vlan30
	network 192.168.200.0 255.255.255.0
	default-router 192.168.200.1
	
using per vlan stp, we can arrange different root bridges for different vlan's.

also using rapid-pvst, we can make faster port connection.

	SW4(config)#spanning tree vlan 10 priority 0
		    spanning-tree mode rapid-pvst
	SW5(config)#spanning tree vlan 20 priority 0
		    spanning-tree mode rapid-pvst
	SW6(config)#spanning tree vlan 30 priority 0
		    spanning-tree mode rapid-pvst

After that, we need to arrange ports in SW6:

	interface fa0/22
	switchport access vlan 10
	switchport access vlan 20
	switchport access vlan 30
	switchport mode access
fa0/24 port must be trunk, this is our only connection to the router.





- STEP8:

For fast connection between router and end-users, we can use portfast. 

in SW4:

	interface range fa0/1-20
	spanning-tree portfast
It gives us a warning message about loop threat for each port, we need to use it carefully.

For port security:

	switchport port-security maximum 1
	switchport port-security violation protect.
	


