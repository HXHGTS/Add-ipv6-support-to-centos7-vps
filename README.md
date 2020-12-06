## 为未配置ipv6支持的CentOS7服务器添加ipv6地址

### 操作过程

1.打开HE隧道，[申请账号](https://tunnelbroker.net/)

2.创建Regular Tunnel，输入服务器公网ipv4地址

3.选择离服务器最近的网关服务器(香港服务器貌似不可用)，确定
```
North America
 Ashburn, VA, US 216.66.22.2
 Calgary, AB, CA 216.218.200.58
 Chicago, IL, US 184.105.253.14
 Dallas, TX, US 184.105.253.10
 Denver, CO, US 184.105.250.46
 Fremont, CA, US 72.52.104.74
 Fremont, CA, US 64.62.134.130
 Honolulu, HI, US 64.71.156.86
 Kansas City, MO, US 216.66.77.230
 Los Angeles, CA, US 66.220.18.42
 Miami, FL, US 209.51.161.58
 New York, NY, US 209.51.161.14
 Phoenix, AZ, US 66.220.7.82
 Seattle, WA, US 216.218.226.238
 Toronto, ON, CA 216.66.38.58
 Winnipeg, MB, CA 184.105.255.26
Europe
 Amsterdam, NL 216.66.84.46
 Berlin, DE 216.66.86.114
 Budapest, HU 216.66.87.14
 Frankfurt, DE 216.66.80.30
 Lisbon, PT 216.66.87.102
 London, UK 216.66.80.26
 London, UK 216.66.88.98
 Paris, FR 216.66.84.42
 Prague, CZ 216.66.86.122
 Stockholm, SE 216.66.80.90
 Warsaw, PL 216.66.80.162
 Zurich, CH 216.66.80.98
Asia
 Hong Kong, HK 216.218.221.6
 Singapore, SG 216.218.221.42
 Tokyo, JP 74.82.46.6
Africa
 Djibouti City, DJ 216.66.87.98
 Johannesburg, ZA 216.66.87.134
South America
 Bogota, CO 216.66.64.154
Oceania
 Sydney, NSW, AU 216.218.142.50
Middle East
 Dubai, AE 216.66.90.30
```
4.选择Example Configurations，OS选择linux-route2(配置如示例，注意如果有内网ip必须填内网ip)
```
modprobe ipv6
ip tunnel add he-ipv6 mode sit remote 网关服务器ipv4地址 local 本机内网ip(没有就填公网ip) ttl 255
ip link set he-ipv6 up
ip addr add 本机申请的ipv6地址/64 dev he-ipv6
ip route add ::/0 dev he-ipv6
ip -f inet6 addr
```
5.ssh连接服务器，配置ipv6支持(汉字注意替换成自己的内容)
```
echo 'IPV6ADDR=HE隧道IPV6网关' >> /etc/sysconfig/network-scripts/ifcfg-eth0
echo 'ONBOOT=yes' > /etc/sysconfig/network-scripts/ifcfg-sit1
echo 'DEVICE=sit1' >> /etc/sysconfig/network-scripts/ifcfg-sit1
echo 'BOOTPROTO=none' >> /etc/sysconfig/network-scripts/ifcfg-sit1
echo 'IPV6INIT=yes' >> /etc/sysconfig/network-scripts/ifcfg-sit1
echo 'IPV6TUNNELIPV4=隧道服务器的IPV4' >> /etc/sysconfig/network-scripts/ifcfg-sit1
echo 'IPV6TUNNELIPV4LOCAL=本机的IPV4地址' >> /etc/sysconfig/network-scripts/ifcfg-sit1
echo 'IPV6ADDR=本机IPV6地址' >> /etc/sysconfig/network-scripts/ifcfg-sit1
echo 'NETWORKING_IPV6=yes' >> /etc/sysconfig/network
echo 'IPV6_DEFAULTDEV="sit1"' >> /etc/sysconfig/network
```
6.重启网卡，设置定时任务与重启任务(bash执行)
```
systemctl restart network
crontab -e
*/10 * * * * ping ipv6.google.com -c3
crontab -l
```
