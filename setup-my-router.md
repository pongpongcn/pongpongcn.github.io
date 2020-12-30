install dnsmasq-full
```bash
opkg update
opkg remove dnsmasq
opkg install dnsmasq-full
```

install dns-forwarder, shadowsocks  
Access [OpenWrt-dist](http://openwrt-dist.sourceforge.net/), and follow.  
Add sources to Custom feeds.
```bash
opkg update
opkg install ChinaDNS
opkg install luci-app-chinadns
opkg install dns-forwarder
opkg install luci-app-dns-forwarder
opkg install shadowsocks-libev
opkg install luci-app-shadowsocks
opkg install ip
opkg install iptables-mod-tproxy
opkg install simple-obfs
```

config dnsmasq-gfwlist
```bash
opkg update
opkg install coreutils-base64 curl ca-certificates ca-bundle
opkg install libustream-openssl
wget -O /usr/bin/gfwlist2dnsmasq.sh https://github.com/cokebar/gfwlist2dnsmasq/raw/master/gfwlist2dnsmasq.sh
chmod +x /usr/bin/gfwlist2dnsmasq.sh
mkdir /etc/dnsmasq.d
gfwlist2dnsmasq.sh -o /etc/dnsmasq.d/gfwlist.conf -s gfwlist
echo 'conf-dir=/etc/dnsmasq.d' >> /etc/dnsmasq.conf
/etc/init.d/dnsmasq restart
```

To speed up some websites, add custom entries to `/etc/dnsmasq.d/customlist.conf`.

Sample:
```
server=/adobe.com/127.0.0.1#5353
ipset=/adobe.com/gfwlist
server=/icloud.com/127.0.0.1#5353
ipset=/icloud.com/gfwlist
server=/speedtest.net/127.0.0.1#5353
ipset=/speedtest.net/gfwlist
```

Init gfwlist when start, add following lines to /etc/rc.local
```
ipset -L gfwlist >/dev/null 2>&1
if [ $? -ne 0 ]; then
    ipset create gfwlist hash:ip
fi

ipset add gfwlist 8.8.8.8
ipset add gfwlist 8.8.4.4
```

Config shadowsocks
* Zone WAN  
Bypassed IP List `ChinaDNS CHNRoute`
* Zone LAN  
Extra arguments `-m set --match-set gfwlist dst`

Config Scheduled Tasks
```
0 5 * * * /usr/bin/update_dnsmasq_gfwlist
0 6 * * * /usr/bin/update_chnroute
```

/usr/bin/update_dnsmasq_gfwlist
```shell
#!/bin/sh

TEMPFILE=$(mktemp -u)
DESTFILE=/etc/dnsmasq_gfwlist_ipset.conf

/usr/bin/gfwlist2dnsmasq.sh -s gfwlist -o $TEMPFILE
if [ $? -eq 0 ]
then
    cp $TEMPFILE $DESTFILE
    /etc/init.d/dnsmasq restart
fi

if [ -s $TEMPFILE ]
then
   rm -f $TEMPFILE
fi
```

/usr/bin/update_chnroute
```shell
#!/bin/sh

TEMPAPNICFILE=$(mktemp -u)
TEMPFILE=$(mktemp -u)
DESTFILE=/etc/chnroute.txt

wget 'http://ftp.apnic.net/apnic/stats/apnic/delegated-apnic-latest' -q -O $TEMPAPNICFILE
if [ $? -eq 0 ]
then
    awk -F\| '/CN\|ipv4/ { printf("%s/%d\n", $4, 32-log($5)/log(2)) }' $TEMPAPNICFILE > $TEMPFILE
    if [ $? -eq 0 ]
    then
        ipset flush chnroute
        for ip in $(cat $TEMPFILE)
        do
            ipset add chnroute $ip
        done
        cp $TEMPFILE $DESTFILE
    fi
fi

if [ -f $TEMPAPNICFILE ]
then
   rm -f $TEMPAPNICFILE
fi

if [ -f $TEMPFILE ]
then
   rm -f $TEMPFILE
fi
```

/etc/firewall.user
```shell
iptables -I PREROUTING -t nat -i eth0 -p udp -d 8.8.8.8 --dport 53 -j REDIRECT --to-ports 53
iptables -I PREROUTING -t nat -i eth0 -p udp -d 8.8.4.4 --dport 53 -j REDIRECT --to-ports 53
```
