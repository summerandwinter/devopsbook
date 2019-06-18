# 安装

## 上传

首先从[官网](https://grafana.com/grafana/download?platform=linux)下载最近的安装包，这里我们以  grafana-6.2.1-1.x86\_64.rpm  版本为例，上传下载的 rpm 包到 /opt/devops/ 目录。

执行下面的命令安装

```bash
yum localinstall grafana-6.2.1-1.x86_64.rpm
```

## 启动服务

```bash
systemctl daemon-reload
systemctl start grafana-server
systemctl status grafana-server
```

## 开机启动

```bash
systemctl enable grafana-server.service
```

## 环境配置

Grafana 后台启动的时候会用到 /etc/sysconfig/grafana-server 文件里面配置的环境变量。我们可以覆盖文件里面的默认配置实现自己自定义的环境变量

### 日志

Grafana 默认的日志文件放在 /var/log/grafana 目录，我们可以设置 LOG\_DIR 的值自定义日志文件目录。

```text
LOG_DIR=/var/log/grafana
```

### 数据库

默认使用 sqllite3 数据库，数据库文件放在 /var/lib/grafana/grafana.db 目录，我们也可以使用 MySQL 或者 Postgres 作为 Grafana 的数据库。

> 需要注意的是，如果我们使用的是默认的 sqllite3 数据库，升级之前要记得做好备份。

### 配置

在 /etc/grafana/grafana.ini 目录中我们可以对 Grafana 做更详细的配置，这里我们不再做详细说明，具体配置可以参考[官方文档](https://grafana.com/docs/installation/configuration/)。

