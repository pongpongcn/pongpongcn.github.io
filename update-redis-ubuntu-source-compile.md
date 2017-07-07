# 在Ubuntu系统上，使用源码编译，更新内置Redis。

```shell
apt install libssl-dev libsystemd-dev pkg-config
wget --content-disposition https://codeload.github.com/redis/redis/tar.gz/refs/tags/7.2.3
tar -xf redis-7.2.3.tar.gz
cd redis-7.2.3
make BUILD_TLS=yes USE_SYSTEMD=yes
make PREFIX=/usr install
```
