# 主机初始化

- 设置主机名

```
hostnamectl set-hostname k8s-m1
hostnamectl set-hostname k8s-w1
hostnamectl set-hostname k8s-w2
hostnamectl set-hostname k8s-w3
```
- 主机名解析

```
cat  >> /etc/hosts <<EOF
192.168.122.111 k8s-m1
192.168.122.211 k8s-w1
192.168.122.212 k8s-w2
192.168.122.213 k8s-w3
EOF
```
- 设置静态网络

```
cat /etc/netplan/00-installer-config.yaml
network:
  ethernets:
    eth0:
      dhcp4: no
      addresses:
      - 192.168.122.111/24
      gateway4: 192.168.122.1
      nameservers:
        addresses: [223.5.5.5, 114.114.114.114]
  version: 2

# 应用网络配置
netplan apply
```

- 初始化ssh配置

```
sed -ri 's@#PermitRootLogin prohibit-password@PermitRootLogin yes@g' /etc/ssh/sshd_config

systemctl restart sshd
```

- 关闭防火墙

```
# 放行端口
sudo ufw allow 3306/tcp

# 关闭防火墙
sudo ufw disable
```

- 设置时区

```
# 查看当前时区
date -R

# 修改为北京时区
sudo cp /usr/share/zoneinfo/Asia/Shanghai  /etc/localtime

# 或者使用交互方式配置为Beijing时区
sudo tzselect
```

- 配置软件源

修改`/etc/apt/sources.list`替换默认的字符串`archive.ubuntu.com/`为`mirrors.aliyun.com`，替换结果如下：

```
sudo sed -i 's@archive.ubuntu.com@mirrors.aliyun.com@g' /etc/apt/sources.list
sudo apt autoclean
sudo apt update

sudo apt install  -y vim net-tools tree wget  unzip htop ethtool bridge-utils tcpdump bash-completion ntpdate
```

- 时间同步

```
/usr/sbin/ntpdate ntp.aliyun.com &> /dev/null
crontab -l

*/3 * * * * /usr/sbin/ntpdate ntp.aliyun.com &> /dev/null
```
- 关闭selinux(略)
- 关闭swap

```
sed  -ri  's@/.*swap.*@# &@' /etc/fstab && swapoff -a
```

- 打开内核转发

```
cat > /etc/sysctl.d/k8s.conf <<EOF
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-arptables = 1
net.ipv4.tcp_tw_reuse = 0
net.core.somaxconn = 32768
net.netfilter.nf_conntrack_max=1000000
vm.swappiness = 0
vm.max_map_count=655360
fs.file-max=6553600
EOF

sysctl -p
```
