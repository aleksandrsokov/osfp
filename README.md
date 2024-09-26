# osfp

Разворачиваем 3 ВМ для настройки асиметричного роутинга  

1. router1  
   10.0.10.1/30 - r1-r2  линк дорогой  
   10.0.12.1/30 - r1-r3    
   192.168.10.1/24 - net1  
   192.168.50.10/34 - for ssh  

2. router2  
   10.0.10.2/30 - r1-r2  
   10.0.11.2/30 - r2-r3  
   192.168.20.1/24 - net2  
   192.168.50.11/34 - for ssh  

3. router3  
   10.0.11.1/30 - r2-r3  
   10.0.12.2/30 - r1-r3  
   192.168.30.1/24 - net3  
   192.168.50.12/34 - for ssh  


Асимметричный роутинг включается sysctl net.ipv4.conf.all.rp_filter=0 

Симметричность роутинга добивается одинаковой стоимостью маршрута в файле rff.conf параметр ip ospf cost


   # тесты:  
vagrant@router1:~$ ip r  
default via 10.0.2.2 dev enp0s3 proto dhcp src 10.0.2.15 metric 100   
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 100   
10.0.2.2 dev enp0s3 proto dhcp scope link src 10.0.2.15 metric 100   
10.0.2.3 dev enp0s3 proto dhcp scope link src 10.0.2.15 metric 100   
10.0.10.0/30 dev enp0s8 proto kernel scope link src 10.0.10.1   
10.0.11.0/30 nhid 32 via 10.0.12.2 dev enp0s9 proto ospf metric 20   
10.0.12.0/30 dev enp0s9 proto kernel scope link src 10.0.12.1   
192.168.10.0/24 dev enp0s10 proto kernel scope link src 192.168.10.1   
192.168.20.0/24 nhid 32 via 10.0.12.2 dev enp0s9 proto ospf metric 20   
192.168.30.0/24 nhid 32 via 10.0.12.2 dev enp0s9 proto ospf metric 20   
192.168.50.0/24 dev enp0s16 proto kernel scope link src 192.168.50.10   
vagrant@router1:~$ ping 192.168.30.1  
PING 192.168.30.1 (192.168.30.1) 56(84) bytes of data.  
64 bytes from 192.168.30.1: icmp_seq=1 ttl=64 time=0.725 ms  
64 bytes from 192.168.30.1: icmp_seq=2 ttl=64 time=1.55 ms  
64 bytes from 192.168.30.1: icmp_seq=3 ttl=64 time=1.30 ms  
^C  
192.168.30.1 ping statistics   
3 packets transmitted, 3 received, 0% packet loss, time 2015ms  
rtt min/avg/max/mdev = 0.725/1.191/1.552/0.345 ms  
vagrant@router1:~$ traceroute 192.168.30.1  
traceroute to 192.168.30.1 (192.168.30.1), 30 hops max, 60 byte packets  
 1  192.168.30.1 (192.168.30.1)  0.638 ms  0.599 ms  0.570 ms  
vagrant@router1:~$ ifconfig enp0s9 down  
SIOCSIFFLAGS: Operation not permitted  
vagrant@router1:~$ sudo ifconfig enp0s9 down  
vagrant@router1:~$ traceroute 192.168.30.1  
traceroute to 192.168.30.1 (192.168.30.1), 30 hops max, 60 byte packets  
 1  10.0.10.2 (10.0.10.2)  0.669 ms  0.578 ms  0.529 ms  
 2  192.168.30.1 (192.168.30.1)  1.154 ms  0.995 ms  0.942 ms  
