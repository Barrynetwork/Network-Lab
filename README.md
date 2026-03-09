# Network-Lab
This lab simulates a secure connection between two sites (Toronto and Montreal) using an IPsec VPN, allowing communication between Data and Voice VLANs. It includes the configuration of Inter-VLAN routing, DHCP, and Cisco CME to enable IP phone communication and ensure secure network connectivity between the two locations.

<img width="1535" height="668" alt="image" src="https://github.com/user-attachments/assets/972269a4-79c1-41d1-bbb5-9c405483d5bf" />

1. ** Router Toronto**
```powershell
ip dhcp excluded-address 10.10.20.1 10.10.20.10
ip dhcp excluded-address 10.10.30.1 10.10.30.100

ip dhcp pool DATA
network 10.10.20.0 255.255.255.0
default-router 10.10.20.1


ip dhcp pool VOICE
network 10.10.30.0 255.255.255.0
default-router 10.10.30.1
option 150 ip 10.10.30.1
```

## IP Config
```powershell
int fa0/0
ip add 172.16.10.2 255.255.255.252
no shut

int fa0/1.20
encapsulation dot1q 20
ip add 10.10.20.1 255.255.255.0

int fa0/1.30
encapsulation dot1q 30
ip add 10.10.30.1 255.255.255.0

int fa0/1
no shut
```

## ROutage
```powershell
ip route 0.0.0.0 0.0.0.0 172.16.10.1
```

### ACL
```powershell
access-list 110 permit ip 10.10.20.0 0.0.0.255 10.10.120.0 0.0.0.255
access-list 110 permit ip 10.10.20.0 0.0.0.255 10.10.130.0 0.0.0.255
access-list 110 permit ip 10.10.30.0 0.0.0.255 10.10.120.0 0.0.0.255
access-list 110 permit ip 10.10.30.0 0.0.0.255 10.10.130.0 0.0.0.255
```
### IPSEC
```powershell
crypto isakmp policy 10
encryption aes 256
authentication pre-share
group 5
exit
crypto isakmp key vpnboreal address 172.16.20.2


crypto ipsec transform-set VPNIPSEC esp-aes esp-sha-hmac
crypto map VPN-MAP 10 ipsec-isakmp
description VPN connection to TOR
set peer 172.16.20.2
set transform-set VPNIPSEC
match address 110
exit

int fa0/0
crypto map VPN-MAP
```

### TELEPHONY SERVICE
```powershell
telephony-service
max-ephones 10
max-dn 10
ip source-address 10.10.30.1 port 2000
auto assign 1 to 5
exit

ephone-dn 1
number 10001

ephone-dn 2
number 10002

ephone-dn 3
number 10003

ephone-dn 4
number 10004

ephone-dn 5
number 10005
```

### DIAL PEER 
```powershell
dial-peer voice 1 voip
destination-pattern 20...
session target ipv4:10.10.130.1
```


