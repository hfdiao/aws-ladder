# Dashboard

Edit "Security Group", allow accessing UDP port 500, 4500 and 1701 from 0.0.0.0/0

# Server
## install required software
```
sudo apt-get install xl2tpd openswan ppp
```

## config ipsec

***be sure to contain at least one new line at the end of files: `/etc/ipsec.secrets` and `/etc/ipsec.conf`***

```
# file: /etc/ipsec.secrets

include /var/lib/openswan/ipsec.secrets.inc
$aws_private_ip %any: PSK "$psk_pasword" 



```
```
# file: /etc/ipsec.conf

config setup
    nat_traversal=yes
    protostack=netkey
    oe=off
    interfaces="%defaultroute"
    dumpdir=/var/run/pluto/
    virtual_private=%v4:10.0.0.0/8,%v4:192.168.0.0/16,%v4:172.16.0.0/12,%v4:25.0.0.0/8,%v4:100.64.0.0/10,%v6:fd00::/8,%v6:fe80::/10

conn L2TP-PSK-NAT
    rightsubnet=vhost:%priv
    also=L2TP-PSK-noNAT

conn L2TP-PSK-noNAT
    authby=secret
    pfs=no
    auto=add
    keyingtries=3
    rekey=no
    ikelifetime=8h
    keylife=1h
    type=transport
    left=$aws_private_ip
    leftprotoport=17/1701
    right=%any
    rightprotoport=17/%any
    dpddelay=40
    dpdtimeout=130
    dpdaction=clear










```

## config xl2tpd

```
# file: /etc/xl2tpd/xl2tpd.conf

[global]
ipsec saref = yes
[lns default]
ip range = 192.168.18.2-192.168.18.254
local ip = 192.168.18.1
;require chap = yes
refuse chap = yes
refuse pap = yes
require authentication = yes
ppp debug = no
pppoptfile = /etc/ppp/options.xl2tpd
length bit = yes





```

## config pppd
```
# file: /etc/ppp/options.xl2tpd

ipcp-accept-local
ipcp-accept-remote
require-mschap-v2
ms-dns 8.8.8.8
ms-dns 8.8.4.4
asyncmap 0
auth
crtscts
lock
hide-password
modem
debug
name l2tpd
proxyarp
lcp-echo-interval 30
lcp-echo-failure 4






```
```
# file: /etc/ppp/chap-secrets

# Secrets for authentication using CHAP
# client    server    secret    IP addresses
$client_username    l2tpd    $password       *



```

## config forwaring
```sh
# file: /etc/init.d/acrossgfw

# untested

iptables –table nat –append POSTROUTING –jump MASQUERADE
    
echo 1 > /proc/sys/net/ipv4/ip_forward
for each in /proc/sys/net/ipv4/conf/*
do
    echo 0 > $each/accept_redirects
    echo 0 > $each/send_redirects
done




```
```
update-rc.d acrossgfw defaults
```

## start services
```
sudo service acrossgfw start
sudo service ipsec restart
sudo service xl2tpd restart
```
