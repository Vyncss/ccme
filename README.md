# ccme
CCME Router Insatll and Configuration
| Machine | IP |
| - | - |
| Mint 18 br0 | 192.168.5.254 |
| Router 1 | 192.168.5.4 |
| Router 2 | 192.168.5.5 |
| Windows 1 | 192.168.5.10 |
| Windows 2 | 192.168.5.11 |

ISOs:

https://www.linuxmint.com/edition.php?id=248

https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2019

Create NAT Network in Virtual Box

Steps
1. Install mint 18
2. install VM guest additions
3. install updates and all programs needed for this
```shell
sudo apt update  
sudo apt install dynagen dynamips bridge-utils uml-utilities nano tftpd
```
4. Find out interface of network adapter
`ifconfig`
5. Then edit your interface file `sudo nano /etc/network/interfaces`
```shell
iface enp0s3 inet manual

iface br0 inet static
	bridge_ports enp0s3
	address 192.168.5.254
	netmask 255.255.255.0
	gateway 192.168.5.1
```
6. Then restart network services `sudo service networking restart`
7. Create router folder
8. Move c7200.bin file to folder
9. CD to folder
10. Create router.config with the following

```shell
autostart = False
[127.0.0.1:2000]  
workingdir = /home/mac/router
	udp = 10100  
	[[7200]]  
		image = c7200.bin 
		disk0 = 256  
		idlepc = 0x60be916c  
	[[ROUTER r1]]  
		model = 7200  
		console = 2521  
		aux = 2119  
		#wic0/0 = WIC-1T  
		#wic0/1 = WIC-1T  
		#wic0/2 = WIC-1T  
		f0/0 = nio_tap:tap1  
		x = 22.0  
		y = -351.0
	[[ROUTER r2]]  
		model = 7200  
		console = 2522  
		aux = 2119  
		#wic0/0 = WIC-1T  
		#wic0/1 = WIC-1T  
		#wic0/2 = WIC-1T  
		f0/0 = nio_tap:tap2
		x = 22.0  
		y = -351.0
```
10. Start R1 with dynagen
`dynamips -H 2000&`
`dynagen router.conf`
12. In the dynagen terminal type
`start r1`
`telnet r1`
12. Then you can telnet through a terminal
14. Once in router config interface f0/0 with IP address
```shell
en
conf t
hostname R1-Mac
int f0/0
ip add 192.168.5.4 255.255.255.0
no shut
```
16. Then use these commands to enable the interfaces
17. Finally add the fake router interfaces to the created br0
```shell
sudo brctl addif br0 tap1 tap2
sudo ifconfig tap1 up
sudo ifconfig tap2 up
```
17. Once you are ready to flash you first have to make a TFTP server on the Mint18
`sudo nano /etc/xinetd.d/tftp`
```shell
service tftp  
{  
	protocol =udp  
	socket_type =dgram  
	wait =yes  
	user =nobody  
	server =/usr/sbin/in.tftpd  
	server_args =/tftpboot  
	disable =no  
}
```
18. Then make a your TFTP directory and move your cmt.tar file to there
`sudo mkdir /tftpboot` 
`sudo mv cme.tar /tftpboot/`
`sudo chmod 777 -R /tftpboot`
19. Now restart xinetd: (The TFTP process)
	`sudo systemctl restart xinetd`
20. Once ready to flash go back to your router and then format disk0
`format disk0:` 
`archive tar /xtract tftp://(your mint vm's ip)/cme.tar disk0:`
21. Once finished restart and shutdown R1 using `shut r1`
22. Briefly start and then stop r2
23. Then CD to router folder and copy R1 disk0 to R2 disk0 using
`ls`
`cp c7200_r1_disk0 c7200_r2_disk0`
24. Finally back on R1 enter the following in global config mode
```shell
ip http server  
ip http path disk0:/gui  
telephony-service  
web admin system name admin password 123 
dn-webedit  
time-webedit  
max-ephones 5  
ip source-address 192.168.5.4 port 2000   
max-dn 25  
system message R1-Mac VOIP  
create cnf-files
```
25. Then on R2 set a static ip on the interface and then enter in the following commands as well
```shell
hostname R2-Mac
int f0/0
ip add 192.168.5.5 255.255.255.0
no shut
exit
ip http server  
ip http path disk0:/gui  
telephony-service  
web admin system name admin password 123 
dn-webedit  
time-webedit   
max-ephones 5  
ip source-address 192.168.5.5 port 2000 
max-dn 25  
system message R2-Mac VOIP  
create cnf-files
```
26. Then test if able to Ping br0 and other router after enabling the tap2 interface
27.  Now time to set up two windows server 2019 installs
28. Standard install besides the IP of the two machines being different
29. Install Virtual box Guest additions and Cisco IP communicator and then restart
30. Enter in IP of routers in TFTP server field
31. Once associated go to GUI of each router and register each phone
32. Then you are going to want to configure a DN for each phone on each router
33. Then on each router do these commands to associate the phone with the number
```shell
ephone 1  
button 1:1
```
33. Finally to enable calling between the two we need to enter these commands on both routers
R1
```shell
dial-peer voice 1 voip  
destination-pattern 79..  
session target ipv4:192.168.5.5
```
R2
```shell
dial-peer voice 2 voip  
destination-pattern 78..  
session target ipv4:192.168.5.4
```
34. If everything is entered in correctly you should be able to call the other phone
35. That concludes the tutorial
