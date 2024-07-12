# Kubernetes 实验
----

本实验使用四台 Ubuntu 20.04.6 LTS 组成，机器配置如下:

| 主机名 | IP地址 | 配置 |
| --- | --- | --- |
| k8s-m1 | 192.168.122.111 | 2c8g/125G |
| k8s-w1 | 192.168.122.211 | 4c8g/125G |
| k8s-w2 | 192.168.122.212 | 4c8g/125G |
| k8s-w3 | 192.168.122.213 | 4c8g/125G |

软件使用版本如下：
- k8s: v1.23.4
- docker: 23.0.3

- [系统初始化](docs/01_initsys-ubuntu.md)
- [安装容器运行时](docs/02_install-docker.md)
- [安装Kubernetes集群](docs/03_install-kubernetes.md)

