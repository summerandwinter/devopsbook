# 安装

Prometheus 可以通过多种方式部署：通过预编译二进制包，源码，Docker，配置管理系统（[Ansible](https://github.com/cloudalchemy/ansible-prometheus)，[Chef](https://github.com/elijah/chef-prometheus)，[Puppet](https://forge.puppet.com/puppet/prometheus)，[SaltStack](https://github.com/bechtoldt/saltstack-prometheus-formula)），安装包的方式安装。这里我们介绍通过安装包的方式安装，已应对需要内网安装的场景，其他方式的安装可以参考[官方文档](https://prometheus.io/docs/prometheus/latest/installation/)。

> 这里我们已 2.10.0 版本为例，服务器系统为 CentOS 7.5

首先需要在[官方](https://github.com/prometheus/prometheus/releases)下载最新安装包，官方针对不同的操作系统、 CPU 体系结构发布了不同版本的安装包，如  prometheus-2.10.0.linux-amd64.tar.gz 中 linux 代表操作系统， amd64 代表 CPU（中央处理器）的指令集。

Linux 系统通过下面的命令可以查看 CPU 硬件架构

```bash
arch
```

这里需要注意的是，在我们的测试机器上 `arch`  命令得到的结果是 x86\_64，而在下载页面中我们找不到 x86\_64 相关的包。经过[调研](https://desert3.iteye.com/blog/1666404)我们发现，x86\_64 是 64 位微处理器架构及其相应指令集的一种，也是 Intel x86架构的延伸产品。x86-64 1999由AMD设计，AMD 首次公开 64 位集以扩充给 IA-32，称为 x86-64（后来改名为 AMD64）。其后也为 Intel 所采用，现时英特尔称之为 Intel64 . 由于 AMD64 和Intel64 基本上一致，很多软硬件产品都使用一种不倾向任何一方的词汇来表明它们对两种架构的同时兼容。出于这个目的，AMD 对这种 CPU 架构的原始称呼——“x86-64”被不时地使用，还有变体 “x86\_64”。

综上 x86\_64 对应的是 AMD64，所以这里我们选择 linux-amd64。

## 上传

在 opt 目录下新建一个 devops 目录

```bash
mkdir /opt/devops
```

上传 prometheus-2.10.0.linux-amd64.tar.gz 包到 /opt/devops 目录，可以在该目录下执行 `rz`  命令上传文件。然后把压缩包解压并重命名，统一安装目录方便统一管理。

```bash
tar -zxvf prometheus-2.10.0.linux-amd64.tar.gz
mv prometheus-2.10.0.linux-amd64 prometheus
```

## 启动

在 /opt/devops/promethues 目录下执行下面的命令

```bash
./prometheus
```

Prometheus 会默认加载当前路径下的 prometheus.yaml 文件，打开 [localhost:9090](http://localhost:9090/) 如果能看到下面首页代表启动成功。

![Prometheus &#x9996;&#x9875;](../../.gitbook/assets/prometheus-ui-graph.png)

## 创建 systemd 服务

在生产环境中我们经常会遇到机器重启的情况，每次重启都需要重启服务，为了避免这种情况我们需要把 Prometheus 服务注册到系统守护进程中。

创建 prometheus.service 文件

```bash
vim /etc/systemd/system/prometheus.service
```

复制下面的内容到 prometheus.service

{% code-tabs %}
{% code-tabs-item title="prometheus.service" %}
```text
[Unit]
Description=prometheus
After=network.target

[Service]
Type=simple
User=root 
ExecStart=/opt/devops/prometheus/prometheus \
    --config.file=/opt/devops/prometheus/prometheus.yml
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
{% endcode-tabs-item %}
{% endcode-tabs %}

启动并注册 Prometheus 服务

```bash
systemctl daemon-reload # 刷新配置
systemctl start prometheus # 启动服务
systemctl status prometheus # 查看服务启动是否成功
systemctl enable prometheus # 注册系统服务
```

完成这些操作后我们就可以通过 `systemctl`  命令来启动 Prometheus 服务了，下面是一些常用的命令

```bash
systemctl start prometheus # 启动
systemctl stop prometheus # 关闭
systemctl restart prometheus # 重启
```

