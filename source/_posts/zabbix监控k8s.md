---
title: zabbix监控k8s
date: 2025-11-04 13:36:41
tags:
---
# zabbix监控k8s

## [参考资料](https://git.zabbix.com/projects/ZT/repos/kubernetes-helm/browse?at=refs%2Fheads%2Frelease%2F7.4)

## 现状我已经部署了zabbix-server，但是没有在k8s部署zabbix-agent

## zabbix-agent和zabbix-proxy的区别
- Zabbix 代理默认以 DaemonSet 的形式部署在所有集群节点上，以被动模式监控各个节点主机资源，使用“Linux by Zabbix agent”模板。
- Zabbix 代理以单副本的 Deployment 形式（默认）以活动模式安装在集群中。它从 Kubernetes API、kube-state-metrics 端点和 Zabbix 代理收集数据。所有收集的数据都会先在本地缓存，然后再传输到代理所属的外部 Zabbix 服务器 ，用于监控

## 安装步骤

- 添加仓库
``` bash
helm repo add zabbix-chart-7.0  https://cdn.zabbix.com/zabbix/integrations/kubernetes-helm/7.0
```

- 将图表 `zabbix-helm-chart` 的默认值导出到文件 `$HOME/zabbix_values`.yaml：
``` bash
helm show values zabbix-chart-7.0/zabbix-helm-chart > $HOME/zabbix_values.yaml
```

- 将 `env.ZBX_SERVER_HOST` 文件 `$HOME/zabbix_values.yaml` 中的环境变量更改为用于监控且可由 Zabbix 代理访问的 Zabbix 服务器的地址。例如我的zabbix-server的地址是192.168.1.100，那么我需要将 `env.ZBX_SERVER_HOST` 改为 `192.168.1.100`

- 列出集群的命名空间
``` bash
kubectl get namespaces
```

- 如果集群中不存在命名空间监控，请创建命名空间`monitoring`：
``` bash
kubectl create namespace monitoring
```

- 在 Kubernetes 集群中部署图表 `zabbix-helm-chart` 到命名空间 `monitoring`：
``` bash
helm install zabbix zabbix-chart-7.0/zabbix-helm-chart  --dependency-update -f $HOME/zabbix_values.yaml -n monitoring
```

- 获取服务帐户名称。如果使用其他版本名称。
``` bash
kubectl get serviceaccount -n monitoring
```

- 获取为服务帐户自动创建的令牌TOKEN：
``` bash
kubectl get secret -n monitoring zabbix-agent -o jsonpath='{.data.token}' | base64 -d
```

这样就安装了zabbix-agent和zabbix-proxy.

## zabbix-server配置

### 创建zabbix-proxy
- administration -> proxies -> create proxy ->proxy name: zabbix-proxy
- zabbix-proxy是默认值,无需修改.直接保存即可.等待一会,zabbix-proxy的状态变成online.

### 创建node监控

- 选择模板`Kubernetes nodes by HTTP`

- ![创建主机](https://github.com/ttkk1024/ttkk1024.github.io-src/blob/main/source/_posts/202511/image.png?raw=true)
- 在 宏 - 继承及主机 宏 中，修改两个宏：
``` yaml
{$KUBE.API.URL}
{$KUBE.API.TOKEN}
```

- ![修改参数](https://github.com/ttkk1024/ttkk1024.github.io-src/blob/main/source/_posts/202511/image-1.png?raw=true)