# LEDE Script
## Cron
```
0 5 * * * /usr/bin/dnsmasq-gfwlist.update.sh
0 6 * * * /usr/bin/ss-bypassed-ip-list.update.sh
```
## dnsmasq-gfwlist.update.sh
```
TEMPFILE=/tmp/gfwlist.conf.tmp
/usr/bin/gfwlist2dnsmasq.sh -d 127.0.0.1 -p 5300 -o $TEMPFILE
if [ $? -eq 0 ]; then
    sed -i 's/127.0.0.1#5300/2001:4860:4860::8888/' $TEMPFILE
    if [ $? -eq 0 ]; then
        cp $TEMPFILE /etc/dnsmasq.d/gfwlist.conf
        /etc/init.d/dnsmasq restart
    fi
fi
rm $TEMPFILE
```

## ss-bypassed-ip-list.update.sh
```
TEMPFILE=/tmp/chinadns_chnroute.tmp
CHNROUTE_TEMPFILE=/etc/chinadns_chnroute.txt.tmp
wget 'http://ftp.apnic.net/apnic/stats/apnic/delegated-apnic-latest' -q -O $TEMPFILE
if [ $? -eq 0 ]; then
    cat $TEMPFILE|awk -F\| '/CN\|ipv4/ { printf("%s/%d\n", $4, 32-log($5)/log(2)) }' > $CHNROUTE_TEMPFILE
    if [ -s $CHNROUTE_TEMPFILE ]
    then
        cp $CHNROUTE_TEMPFILE /etc/chinadns_chnroute.txt
        /etc/init.d/shadowsocks restart
        rm $CHNROUTE_TEMPFILE
    fi
fi
rm $TEMPFILE
```

## DHCP and DNS
* /etc/dnsmasq.conf
```
conf-dir=/etc/dnsmasq.d
```

* Resolve file
```
/tmp/resolv.conf.auto
```
# OpenVPN
* /etc/config/openvpn
```
config openvpn 'my_server'
	option dev 'tun'
	option comp_lzo 'yes'
	option mssfix '1420'
	option keepalive '10 60'
	option verb '3'
	option server '10.0.100.0 255.255.255.0'
	option ca '/etc/openvpn/ca.crt'
	option cert '/etc/openvpn/my-server.crt'
	option key '/etc/openvpn/my-server.key'
	option dh '/etc/openvpn/dh2048.pem'
	option topology 'subnet'
	list push 'redirect-gateway def1'
	list push 'route 192.168.0.0 255.255.255.0'
	list push 'dhcp-option DOMAIN vpn.lx70.darkroc.com'
	list push 'dhcp-option DNS 10.0.100.1'
	option enabled '1'
```
