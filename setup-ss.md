Server(libev)
--------------------

### [shadowsocks-libev](https://github.com/shadowsocks/shadowsocks-libev)

Installation

Debian 9 Stretch
```bash
apt-get install --no-install-recommends gettext build-essential autoconf libtool libpcre3-dev asciidoc xmlto libev-dev libc-ares-dev automake libmbedtls-dev libsodium-dev
git clone https://github.com/shadowsocks/shadowsocks-libev.git
cd shadowsocks-libev
git submodule update --init --recursive
./autogen.sh
./configure
make
make install
cd ..
```

Configuration
```json
{
    "server": "127.0.0.1",
    "server_port": 8388,
    "password": "barfoo!",
    "timeout": 60,
    "method": "chacha20-ietf-poly1305"
}
```

### [simple-obfs](https://github.com/shadowsocks/simple-obfs)

Installation

Debian 9 Stretch
```bash
apt-get install --no-install-recommends build-essential autoconf libtool libssl-dev libpcre3-dev libc-ares-dev libev-dev asciidoc xmlto automake
git clone https://github.com/shadowsocks/simple-obfs.git
cd simple-obfs
git submodule update --init --recursive
./autogen.sh
./configure
make
make install
cd ..
```

Configuration
```json
{
    "server": "127.0.0.1",
    "server_port": 80,
    "password": "barfoo!",
    "timeout": 60,
    "method": "chacha20-ietf-poly1305",
    "plugin": "obfs-server",
    "plugin_opts": "obfs=http;failover=example.com:80"
}
```
```json
{
    "server": "127.0.0.1",
    "server_port": 443,
    "password": "barfoo!",
    "timeout": 60,
    "method": "chacha20-ietf-poly1305",
    "plugin": "obfs-server",
    "plugin_opts": "obfs=tls;failover=example.com:443"
}
```

### [kcptun](https://github.com/shadowsocks/kcptun)

Installation
```bash
cp server_linux_amd64 /usr/local/bin/kcptun-server
cp client_linux_amd64 /usr/local/bin/kcptun-client
```

Configuration
```json
{
    "server": "127.0.0.1",
    "server_port": 29900,
    "password": "barfoo!",
    "timeout": 60,
    "method": "chacha20-ietf-poly1305",
    "plugin": "kcptun-server",
    "plugin_opts": "key=barfoo!;crypt=aes-192"
}
```


### [mbed TLS](https://tls.mbed.org/)

Installation
```bash
apt-get install --no-install-recommends build-essential
curl -O https://tls.mbed.org/download/mbedtls-2.6.0-gpl.tgz
tar -xzvf mbedtls-2.6.0-gpl.tgz
cd mbedtls-2.6.0
make
make install
cd ..
```

Tips
```
"server":["[::]", "0.0.0.0"]
https://wiki.archlinux.org/index.php/Haveged_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)
https://www.digitalocean.com/community/tutorials/how-to-setup-additional-entropy-for-cloud-servers-using-haveged
https://github.com/xtaci/kcptun/issues/137
```

SSH
```
.ssh/authorized_keys 
ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAu1rXjLN4OD4OgKezbepLKB9CL6q9HEmwMeSg04nz+DbZDDObpVYz2C39j1HssH/aop2yyRx3QfDRYmzF34TTHeyPBL4d/aGlwecOS5HMsMAU5Gc85fYw3E0QzEvppSoNtedM/tlXBikq+A+UD8qIGtOAEAvdUVBAiT/pck2SleKF7XBLkHdJSyifhezSq9nIIhJavPFbanFd/Jsbvz9VwypMzFVHHVOO4/d+4K4+pCCSO26FqqKKd9I2xU2oeW0CsYsdmNEjjlz2YfTtr8gXq/DmBQs0PF+/YieMzd008ILTFqbDGcB0Lld/Q5JO6xj6PseuUELkZ+jyQRYNbNBGtQ== Pocketpc@XPS13
ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEA3jZpFkCNe7Zm9on8GPh7U4vdwZ6+BAPRl6RbkIvMAReVcN2KMepwVE0BiXJMB+M1a9z/Qrpwi5vYeA1gOfoY/ZvQiz8qcZTF/mk2fwxNTAhUcJRD/1MTrDI0UJMuk9NcYlbqLnqFC8/VGUfGsABycvdBpXLM/5OgSZuuELuAm3SlYRzBuVmz4JzBhokZL7126prWNlMr7MQZ6db5rBusNQ6CgSpZ8vYVBp2bav4b3u45EOBYNSMtEUvBjBC/fkxW/TTYExQmrfcFlk6LdmFyyYgCu1Wlifv1Kj1QrhfNZnNJyu8va6b0GXxdjpieI3nAHK3JAXDIaPq/1PeVKmDsZQ== Pocketpc@LX70PC
ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAv0P5EyPdhw8dkf3RimOLd3MLiCY8/DfNn/EKEmcuZuZG9LJS6ssiU45NufA+/NyORFn9ftNQY1ZqbI06PqOfwLfMuyaWDleISwNLExwHcmvTd4q0oc9g1vVF6ef2fJEDLqjFrobDu8slhmcSTL1CZDrT4Wq7R7p/2OBxxSn+DyBx+N/by7UJgRW4H0Tb8IB4CXODWiYitbemX5GLWKETLVzJ5BHGfn8orexWpvuq++Hke874ris5T6ECCc81qfw64lFKbeBFtZjYgzoXWWFvHcBaZLrj0tWa0alovYg9rXERD0KiyxEdRWErxb1MNqtkXz5qOnGeBz6Ut82qs46/FQ== Pocketpc@iMac
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQCvkvgy6t9APcwHDDk/E1fqBI3z+O9B0aAunadr2gmLB/FQUOMfoE5cL594Km44tl4TrLEv+QkqOARq/bv+IvfuFA3pyP5LZDiuOYrPITsFdHtfk3yavEKGbJmN2bDhSmIojP/zm0ORtFfoD2ZmcWH81w7O8ku/lg36JweJrC/B3GGBYsMlQvo3ai+Bh3faaBqmIsxHNSrzb/wcZ7SMQsrkEhbZidTFwWlqVAm1MWWvK0lVKUnsoCHCBTR7qngtFYGkKFehIiBWQhRStrxRktVp/GYuTKtP0iZXdmc12cbzQwItWzF5PXfCpm9LGIjxz/BPIFiht+F/kmbtUCzY4uYFUcyZikOkzLTLgUnFdEFE3/h799wr7Cvl4iieqFghW7KF4YiMOsnd35JINc3v6m/d2QCIkS34yGe/7dsBNT88VWXNsDGVsUp07z6sRGPBcNel6ENGeAKnhHyFhenw/C/jT4Dlk24ocpCvdSeWHGNtIff8BPWXAqvOIrAFj/9FL9BMSv1WISF5h+G3qSoRmZHPpnUF5rrPOf5kiasay/TPP0MFWTXaFXCXVHW0lsJY939fGZYHeNrE/hHgU6UBNpBeEU5HS43wWcf73stcX1RVTnpbW6oDh//R7nituVzkDxK9rDLGDKhziBflY4iyqE4Ybx9/PY64vSqtZkZPvDjnHQ== Pocketpc@iPhone6
```

/etc/systemd/system/shadowsocks-libev-{port}.service
```
[Unit]
Description=Shadowsocks-libev {port} Server Service

[Service]
User=nobody
ExecStart=/usr/bin/ss-server -c /etc/shadowsocks-libev/config-{port}.json -u
Restart=always
LimitNOFILE=51200

[Install]
WantedBy=multi-user.target
```

client
```
config servers
option fast_open '0'
option server '67.216.204.120'
option server_port '443'
option timeout '60'
option password 's6yI@!cMhR6c0vCq6T*8CvS048N*cx%tqnyZv^Fw4x6%0CjfrIKMiF5uX'
option encrypt_method 'chacha20-ietf-poly1305'
option plugin 'obfs-local'
option plugin_opts 'obfs=tls;obfs-host=www.amazon.com'
option alias 'USA bandwagon 443'

config servers
option alias 'USA bandwagon 80'
option fast_open '0'
option server '67.216.204.120'
option server_port '80'
option timeout '60'
option password 'Wu$qXm$CvgMAIp8#7JjCN4F4gfhr6d2Vx5*cvB872UnE233G#L64hNw%0IrUY'
option encrypt_method 'chacha20-ietf-poly1305'
option plugin 'obfs-local'
option plugin_opts 'obfs=http;obfs-host=www.amazon.com'
```



$ wget https://github.com/jedisct1/libsodium/releases/download/1.0.12/libsodium-1.0.12.tar.gz
$ tar xf libsodium-1.0.12.tar.gz && cd libsodium-1.0.12
$ ./configure && make -j2 && make install
$ ldconfig
Shadowsocks-libev

$ git clone https://github.com/shadowsocks/shadowsocks-libev.git
$ cd shadowsocks-libev
$ git submodule update --init
$ ./autogen.sh && ./configure && make
$ make install
Systemd

编辑/etc/systemd/system/shadowsocks-libev.service

[Unit]
Description=Shadowsocks-Libev Custom Server Service
Documentation=man:ss-server(1)
After=network.target

[Service]
Type=simple
CapabilityBoundingSet=CAP_NET_BIND_SERVICE
ExecStart=/usr/local/bin/ss-server -c /etc/shadowsocks.json

[Install]
WantedBy=multi-user.target
编辑/etc/shadowsocks.json

{
    "server":"0.0.0.0",
    "server_port":8388,
    "local_port":1080,
    "password":"barfoo!",
    "timeout":60,
    "method":"chacha20-ietf-poly1305"
}
启动服务

$ systemctl enable shadowsocks-libev.service
$ systemctl start shadowsocks-libev.service