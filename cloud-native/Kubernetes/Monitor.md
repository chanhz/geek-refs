# Prometheus、Metrics Server 与 Kubernetes 监控体系
-- 极客时间 第 48 讲
## Prometheus
- CNCF 的第二把交椅，全面接管 Kubernetes 项目的整套监控体系。
- 源于 Google Borg 的 BorgMon，几乎与 Borg 同时诞生的内部监控系统。


组件：
- 存储： TSDB 时序数据库 - Influx DB 和 OpenTSDB
- Metric 监控数据获取-Retrieval： pull from Job exporters / Kubernetes discover target / Pushgateway
- HTTP Server push alerts to Alertmanager
- Promethues web UI and Grafana get data using Prometheus alerting、

- **Pushgateway**，可以允许被监控对象以 Push 的方式向 Prometheus 推送 Metrics 数据；
- **Alertmanager**，则可以根据 Metrics 信息灵活地设置报警；
- 最受用户欢迎的功能，是通过 **Grafana** 对外暴露出的、**可以灵活配置的监控数据可视化界面**。

## 什么是 metric 、 Metric 有哪些分类

Metric 即监控指标数据

Prometheus 项目工作的核心，是使用 Pull （抓取）的方式去搜集被监控对象的 Metrics 数据（监控指标数据），然后，再把这些数据保存在一个 TSDB （时间序列数据库，比如 OpenTSDB、InfluxDB 等）当中，以便后续可以按照时间进行检索。

按 Metrics 数据的来源，可分为：
- 宿主机的监控数据：需要借助 Prometheus 维护的 Node Exporters 工具。一般来说，该工具以 DaemonSet 的方式运行在宿主机上。这些Metrics 包括节点的负载（Load）、CPU 、内存、磁盘以及网络。[更多指标](https://github.com/prometheus/node_exporter#enabled-by-default)
- 第二种 Metrics，是来自于 Kubernetes 的 API Server、kubelet 等组件的 metrics API. 除了 CPU 和内存，还包括各个 Controller 的工作队列（Work Queue）的长度、请求的 QPS(Queries_per_second)和延迟数据等等。这些信息，是检查 Kubernetes 本身工作情况的主要依据。

 


metric 收集的两种设计模式：
- push
- pull





