# 在Ubuntu系统上，使用源码编译，更新内置OpenSSH。

```shell
apt install libssl-dev libpam-dev libsystemd-dev pkg-config
wget https://mirrors.aliyun.com/pub/OpenBSD/OpenSSH/portable/openssh-9.6p1.tar.gz
tar -xf openssh-9.6p1.tar.gz
cd openssh-9.6p1
# 应用补丁，Add systemd readiness notification support
# https://git.launchpad.net/ubuntu/+source/openssh/log/?h=applied/ubuntu/devel
autoreconf
./configure --prefix=/usr --sysconfdir=/etc/ssh --with-ssl-dir=/usr --with-pam --with-systemd
make
make install
systemctl restart sshd
```
