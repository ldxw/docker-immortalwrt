# ImmortalWrt (Multi-Arch Docker)

轻量级 OpenWrt 系统镜像，基于 ImmortalWrt 构建，支持以下平台：

- ✅ amd64
- ✅ arm64

## 🐳 使用示例

```bash
docker run --rm -it ghcr.io/ldxw/immortalwrt:latest sh
```

## 📦 镜像标签结构

- :arm64
- :amd64
- :latest
- :24.10.0-rc3

## 🔧 构建方式

通过 GitHub Actions 自动构建，详见 `.github/workflows/`。


---

## 🛠 使用 & 配置指南

### 1. 设置时间与依赖（可选）

```bash
sudo date -s "2025-01-14 16:01:00"
wget -qO pi.sh https://cafe.cpolar.top/wkdaily/zero3/raw/branch/main/zero3/pi.sh \
  && chmod +x pi.sh && ./pi.sh
```

### 2. 下载 ImmortalWrt rootfs 并解压

```bash
mkdir imm && cd imm
wget -O rootfs.tar.gz https://downloads.immortalwrt.org/releases/24.10.0-rc3/targets/armsr/armv8/immortalwrt-24.10.0-rc3-armsr-armv8-rootfs.tar.gz
gzip -d rootfs.tar.gz
```

### 3. 构建 Docker 镜像

```bash
cat <<EOF > Dockerfile
FROM scratch
ADD rootfs.tar /
EOF

docker build -t immortalwrt:latest .
```

### 4. 创建 macvlan 网络（让容器与主机共用网段）

```bash
docker network create -d macvlan \
  --subnet=192.168.66.0/24 \
  --gateway=192.168.66.1 \
  -o parent=eth0 \
  macnet

ip link add macvlan-shim link eth0 type macvlan mode bridge
ip addr add 192.168.66.2/24 dev macvlan-shim
ip link set macvlan-shim up
ip route add 192.168.66.0/24 dev macvlan-shim
```

### 5. 启动容器并进入

```bash
docker run --name immortalwrt -d --network macnet --privileged immortalwrt:latest /sbin/init
docker exec -it immortalwrt sh
```

### 6. 设置网络配置（容器内）

```bash
cat <<EOF > /etc/config/network
config interface 'lan'
  option device 'eth0'
  option proto 'static'
  option ipaddr '192.168.66.88'
  option netmask '255.255.255.0'
  option gateway '192.168.66.1'
  option dns '223.5.5.5 1.1.1.1'
EOF

/etc/init.d/network restart
```

### 7. 安装常用插件

```bash
opkg update
opkg install luci-i18n-ttyd-zh-cn \
             luci-i18n-filebrowser-go-zh-cn \
             luci-i18n-argon-config-zh-cn \
             openssh-sftp-server \
             luci-i18n-samba4-zh-cn
```

### 8. 安装 iStore 与网络助手（可选）

```bash
wget -qO imm.sh https://cafe.cpolar.top/wkdaily/zero3/raw/branch/main/zero3/imm.sh \
  && chmod +x imm.sh && ./imm.sh

is-opkg install luci-i18n-quickstart-zh-cn
```


---

## 📚 来源参考

本文运行命令及实践方法参考自：

🔗 https://wkdaily.cpolar.top/15
