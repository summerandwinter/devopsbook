# 主机监控： Node Exporter

Node Exporter 主要用来收集 \*NIX 系统的硬件和OS度量信息，基于 Golang 编写。有些度量信息可能只有某一种系统支持，下面的表格列出来各个度量信息，以及支持的系统

## 度量信息

### 默认开启的

| Name | Description | OS |
| :--- | :--- | :--- |
| arp | Exposes ARP statistics from `/proc/net/arp`. | Linux |
| bcache | Exposes bcache statistics from `/sys/fs/bcache/`. | Linux |
| bonding | Exposes the number of configured and active slaves of Linux bonding interfaces. | Linux |
| boottime | Exposes system boot time derived from the `kern.boottime` sysctl. | Darwin, Dragonfly, FreeBSD, NetBSD, OpenBSD, Solaris |
| conntrack | Shows conntrack statistics \(does nothing if no `/proc/sys/net/netfilter/` present\). | Linux |
| cpu | Exposes CPU statistics | Darwin, Dragonfly, FreeBSD, Linux, Solaris |
| cpufreq | Exposes CPU frequency statistics | Linux, Solaris |
| diskstats | Exposes disk I/O statistics. | Darwin, Linux, OpenBSD |
| edac | Exposes error detection and correction statistics. | Linux |
| entropy | Exposes available entropy. | Linux |
| exec | Exposes execution statistics. | Dragonfly, FreeBSD |
| filefd | Exposes file descriptor statistics from `/proc/sys/fs/file-nr`. | Linux |
| filesystem | Exposes filesystem statistics, such as disk space used. | Darwin, Dragonfly, FreeBSD, Linux, OpenBSD |
| hwmon | Expose hardware monitoring and sensor data from `/sys/class/hwmon/`. | Linux |
| infiniband | Exposes network statistics specific to InfiniBand and Intel OmniPath configurations. | Linux |
| ipvs | Exposes IPVS status from `/proc/net/ip_vs` and stats from `/proc/net/ip_vs_stats`. | Linux |
| loadavg | Exposes load average. | Darwin, Dragonfly, FreeBSD, Linux, NetBSD, OpenBSD, Solaris |
| mdadm | Exposes statistics about devices in `/proc/mdstat` \(does nothing if no `/proc/mdstat` present\). | Linux |
| meminfo | Exposes memory statistics. | Darwin, Dragonfly, FreeBSD, Linux, OpenBSD |
| netclass | Exposes network interface info from `/sys/class/net/` | Linux |
| netdev | Exposes network interface statistics such as bytes transferred. | Darwin, Dragonfly, FreeBSD, Linux, OpenBSD |
| netstat | Exposes network statistics from `/proc/net/netstat`. This is the same information as `netstat -s`. | Linux |
| nfs | Exposes NFS client statistics from `/proc/net/rpc/nfs`. This is the same information as `nfsstat -c`. | Linux |
| nfsd | Exposes NFS kernel server statistics from `/proc/net/rpc/nfsd`. This is the same information as `nfsstat -s`. | Linux |
| pressure | Exposes pressure stall statistics from `/proc/pressure/`. | Linux \(kernel 4.20+ and/or [CONFIG\_PSI](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/Documentation/accounting/psi.txt)\) |
| sockstat | Exposes various statistics from `/proc/net/sockstat`. | Linux |
| stat | Exposes various statistics from `/proc/stat`. This includes boot time, forks and interrupts. | Linux |
| textfile | Exposes statistics read from local disk. The `--collector.textfile.directory` flag must be set. | _any_ |
| time | Exposes the current system time. | _any_ |
| timex | Exposes selected adjtimex\(2\) system call stats. | Linux |
| uname | Exposes system information as provided by the uname system call. | FreeBSD, Linux |
| vmstat | Exposes statistics from `/proc/vmstat`. | Linux |
| xfs | Exposes XFS runtime statistics. | Linux \(kernel 4.4+\) |
| zfs | Exposes [ZFS](http://open-zfs.org/) performance statistics. | [Linux](http://zfsonlinux.org/), Solaris |

### 默认关闭的

| Name | Description | OS |
| :--- | :--- | :--- |
| buddyinfo | Exposes statistics of memory fragments as reported by /proc/buddyinfo. | Linux |
| devstat | Exposes device statistics | Dragonfly, FreeBSD |
| drbd | Exposes Distributed Replicated Block Device statistics \(to version 8.4\) | Linux |
| interrupts | Exposes detailed interrupts statistics. | Linux, OpenBSD |
| ksmd | Exposes kernel and system statistics from `/sys/kernel/mm/ksm`. | Linux |
| logind | Exposes session counts from [logind](http://www.freedesktop.org/wiki/Software/systemd/logind/). | Linux |
| meminfo\_numa | Exposes memory statistics from `/proc/meminfo_numa`. | Linux |
| mountstats | Exposes filesystem statistics from `/proc/self/mountstats`. Exposes detailed NFS client statistics. | Linux |
| ntp | Exposes local NTP daemon health to check [time](https://github.com/prometheus/node_exporter/blob/master/docs/TIME.md) | _any_ |
| processes | Exposes aggregate process statistics from `/proc`. | Linux |
| qdisc | Exposes [queuing discipline](https://en.wikipedia.org/wiki/Network_scheduler#Linux_kernel) statistics | Linux |
| runit | Exposes service status from [runit](http://smarden.org/runit/). | _any_ |
| supervisord | Exposes service status from [supervisord](http://supervisord.org/). | _any_ |
| systemd | Exposes service and system status from [systemd](http://www.freedesktop.org/wiki/Software/systemd/). | Linux |
| tcpstat | Exposes TCP connection status information from `/proc/net/tcp` and `/proc/net/tcp6`. \(Warning: the current version has potential performance issues in high load situations.\) | Linux |
| wifi | Exposes WiFi device and station statistics. | Linux |
| perf | Exposes perf based metrics \(Warning: Metrics are dependent on kernel configuration and settings\). | Linux |

## 安装和启动

### 二进制安装

首先需要到[下载页面](https://github.com/prometheus/node_exporter/releases)下载最新的二进制安装包，这里我们以 0.18.1 版本为例。官方针对不同的操作系统和CPU架构发布了不同的安装包，关于安装包的选择可以参考[这一篇](../../../visual-diagram/grafana/installation.md)，这里我们选择 linux-amd64。

#### 上传

在 opt 目录下新建一个 devops 目录

```bash
mkdir /opt/devops
```

上传  node\_exporter-0.18.1.linux-amd64.tar.gz 包到 /opt/devops 目录，可以在该目录下执行 `rz`  命令上传文件。然后把压缩包解压并重命名，统一安装目录方便统一管理。

```bash
tar -zxvf node_exporter-0.18.1.linux-amd64.tar.gz
mv node_exporter-0.18.1.linux-amd64.tar.gz node_exporter
```

#### 启动

在 /opt/devops/node\_exporter 目录下执行下面的命令

```bash
./node_exporter 
```

如果看到类似输出，表示启动成功。

```text
INFO[0000] Starting node_exporter (version=0.14.0, branch=master, revision=840ba5dcc71a084a3bc63cb6063003c1f94435a6)  source="node_exporter.go:140"
INFO[0000] Build context (go=go1.7.5, user=root@bb6d0678e7f3, date=20170321-12:13:32)  source="node_exporter.go:141"
INFO[0000] No directory specified, see --collector.textfile.directory  source="textfile.go:57"
INFO[0000] Enabled collectors:                           source="node_exporter.go:160"
.....
INFO[0000] Listening on :9100                            source="node_exporter.go:186
```

#### 创建 systemd 服务

创建 node\_exporter.service 文件

```bash
vim /etc/systemd/system/node_exporter.service
```

复制下面的内容到 node\_exporter.service

```text
[Unit]
Description=node_exporter
After=network.target

[Service]
Type=simple
User=root 
ExecStart=/opt/devops/node_exporter/node_exporter
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

启动并注册 node\_exporter 服务

```bash
systemctl daemon-reload # 刷新配置
systemctl start node_exporter # 启动服务
systemctl status node_exporter # 查看服务启动是否成功
systemctl enable node_exporter # 注册系统服务
```

完成这些操作后我们就可以通过 `systemctl`  命令来启动 node\_exporter 服务了，下面是一些常用的命令

```bash
systemctl start node_exporter # 启动
systemctl stop node_exporter # 关闭
systemctl restart node_exporter # 重启
```

#### 验证

```bash
curl 127.0.0.1:9100
curl 127.0.0.1:9100/metrics
```

## 数据存储

我们可以利用 Prometheus 的 static\_configs 来拉取 node\_exporter 的数据。

打开 prometheus.yml 文件, 在 scrape\_configs 中添加如下配置：

```text
- job_name: "node"
    static_configs:
      - targets: ["127.0.0.1:9100"]
```

重启加载配置，然后到 Prometheus Console 查询，你会看到 node\_exporter 的数据。



