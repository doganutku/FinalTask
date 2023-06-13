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

![sw1](https://github.com/doganutku/FinalTask/assets/93640554/a7ecdddf-f8ab-403c-8acc-40a742636cdc)

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

	PC name	IP Address		mask				gateway
	PC4		172.16.42.2		255.255.255.128		172.16.42.1
	PC5		172.16.42.130	255.255.255.192		172.16.42.129
	
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
	
- STEP4:

VAN Routera bağlı switchlere yeni bir switch (SW4) eklendi.

Bu yeni  switchin hem SW6’ya hem de SW5’e bağlanması.
