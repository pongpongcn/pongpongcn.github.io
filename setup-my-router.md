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
opkg install dns-forwarder
opkg install luci-app-dns-forwarder
opkg install shadowsocks-libev
opkg install luci-app-shadowsocks
opkg install ip
opkg install iptables-mod-tproxy
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
Zone LAN Extra arguments`-m set --match-set gfwlist dst`