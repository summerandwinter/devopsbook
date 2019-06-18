# 配置

截止到 2019年6月18日，Grafana 已经发布了 [v6.2.3](https://github.com/grafana/grafana/releases/tag/v6.2.3) 版本，而官方的教程示例还停留在 5.x 版本，6.x 版本与 5.x 版本界面上有了很大的改版，下面我们基于 v6.2.1 版本讲解一些 Grafana 中常用的配置，没一种类型的面板都会用一个具体的场景演示各个面板的用法。

这里我们会用到 Prometheus 做为数据源，用 Prometheus 官方提供的 node\_exporter 作为 metrics 采集源， Prometheus 和 node\_exporter 的安装和配置可以参考前文。

## 初始化密码

![](../../../../.gitbook/assets/grafana-login.png)

Grafana 的默认登陆账号密码都是 admin，第一次登陆时会跳转到密码重置页面，我们可以重置默认密码（推荐），当然也可以跳过。

## 添加数据源

![](../../../../.gitbook/assets/grafana-home.png)

登陆成功后在首页点击 Add data source 开始添加数据源

![](../../../../.gitbook/assets/grafana-datasource.png)

这里选择 Prometheus

![](../../../../.gitbook/assets/grafana-add-datasoure-prometheus.png)

**URL**： 填写 promethues 服务的完整地址，我们这里填 http://localhost:9090

**Auth**：出于安全考虑我们可能要对数据源进行权限验证，这里支持 Basic Auth 和 TLS，我们这里只做演示不做勾选。

**Scrape interval**： 抓取数据源的时间间隔，默认 15 秒，这里我们也使用默认值

## 使用模板

Grafana 社区提供了很多用户制作的第三方[模板](https://grafana.com/dashboards)，下面介绍怎么使用这些模板。

![](../../../../.gitbook/assets/grafana-dashboards.png)

首先在模板库中选好我们需要的模板，拷贝模板ID

![](../../../../.gitbook/assets/grafana-manage.png)

点击 import 

![](../../../../.gitbook/assets/grafana-import.png)

粘贴刚刚拷贝的模板ID点击 load

![](../../../../.gitbook/assets/grafana-import-done.png)

选择好前面配置的数据源，到此模板导入完成。

