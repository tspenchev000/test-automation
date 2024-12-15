## Routing information

### NB: RHEL 8/9 Network Manager is taking ownership of network configuration but terraform VMWare provider creates interface information in old format which brings some problems. 

VMWare Networks: Development-10.32.48.0_24 (internal),  Kubernetes-10.32.50.0_24   (external)
Because machine has 2 network interfaces, we need to configure packets to be returned to interface which they come from but NetworkManager is tooooooo smart and we are tired
 .... and as workaround configure external-ip as first network and as default-gw. In additin adding few static routes solved the big routing issue!!! 
One day when k8s has real external ips on the respective interface static route can be defined for entire segment 10.0.0.0/8 

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
```
#https://unix.stackexchange.com/questions/4420/reply-on-same-interface-as-incoming
echo 200 isp2 >> /etc/iproute2/rt_tables
ip rule add from <interface_IP> table isp2 prio 1
ip route add default via <gateway_IP> dev <interface> table isp2


#https://unix.stackexchange.com/questions/31096/sysctl-parameter-for-correct-arp-response
sysctl -w net.ipv4.conf.all.arp_filter=1

net.ipv4.conf.default.arp_filter=1
net.ipv4.conf.all.arp_filter=1


@reboot /sbin/ip rule add from 10.32.46.0/24 table isp2 prio 1; /sbin/ip route add default via 10.32.46.1 table isp2

mkdir -p /etc/iproute2
echo 200 isp2 >> /etc/iproute2/rt_tables
#ip rule add from 10.32.46.211 table isp2 prio 1
ip rule add from 10.32.46.0/24 table isp2 prio 1
ip route add default via 10.32.46.1 table isp2

[root@LD5TMLK8SC01 ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens192: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:50:56:ae:ad:a5 brd ff:ff:ff:ff:ff:ff
    altname enp11s0
    inet 10.32.48.11/24 brd 10.32.48.255 scope global noprefixroute ens192
       valid_lft forever preferred_lft forever
    inet6 fe80::250:56ff:feae:ada5/64 scope link
       valid_lft forever preferred_lft forever
3: ens224: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:50:56:ae:a9:d2 brd ff:ff:ff:ff:ff:ff
    altname enp19s0
    inet 10.32.46.211/24 brd 10.32.46.255 scope global noprefixroute ens224
       valid_lft forever preferred_lft forever
    inet6 fe80::250:56ff:feae:a9d2/64 scope link
       valid_lft forever preferred_lft forever


ip link property add dev `ip a | grep "2:" | awk '{split($0,a,":"); print a[2]}'` altname internal-nic
ip link property add dev `ip a | grep "3:" | awk '{split($0,a,":"); print a[2]}'` altname external-nic

lspci
lshw -c network -businfo   
```

```
https://blog.scottlowe.org/2013/05/29/a-quick-introduction-to-linux-policy-routing/
[root@ld5tmlk8sw02 multi-user.target.wants]# ip rule list
1:      from 10.32.50.1/24 lookup isp2
9:      from all fwmark 0x200/0xf00 lookup 2004
100:    from all lookup local
32766:  from all lookup main
32767:  from all lookup default
[root@ld5tmlk8sw02 multi-user.target.wants]#


nmcli set ipv4.routes 10.32.22.16/32 198.32.48.1
nmcli set ipv4.routes 10.32.21.48/32 198.32.48.1
nmcli set ipv4.routes 10.32.21.48/32 198.32.48.1

[root@LD5TMLK8SW04 ~]# ip route show table 200
default via 10.32.50.1 dev ens224
[root@LD5TMLK8SW04 ~]# ip route show table isp2
default via 10.32.50.1 dev ens224
[root@LD5TMLK8SW04 ~]#
```
