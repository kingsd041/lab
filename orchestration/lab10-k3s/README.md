<div align=center><img src="https://rancher.com/assets/img/brand-guidelines/project-logos/k3s/horizontal/logo-horizontal-k3s.svg" width="500px"></div>

---

# K3s 介绍

K3s 是一个经过 CNCF 认证的轻量级的 Kubernetes 发行版，可用于生产、易于安装、内存减半，2020 年捐献给 CNCF。

**非常适合：**

- Edge
- IoT
- CI
- Development
- ARM
- Embedding k8s

**特点：**

- CNCF 认证的 Kubernetes 发行版
- 不到 100MB 二进制包，500MB 内存消耗
- 单一进程包含 Kubernetes master, Kubelet 和 containerd
- 支持 SQLite/Mysql/PostgreSQL 和 etcd
- 同时为 x86_64, Arm64, 和 Armv7 平台发布

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h5hmpty0wbj21bm0u0wh3.jpg)

## 为什么叫 K3s?

我们希望安装的 Kubernetes 在内存占用方面只是一半的大小。Kubernetes 是一个 10 个字母的单词，简写为 K8s。所以，有 Kubernetes 一半大的东西就是一个 5 个字母的单词，简写为 K3s。K3s 没有全称，也没有官方的发音。

## 为什么比 Kubernetes 节省资源？

- 通过在单个进程内运行许多组件来减少内存占用。这消除了每个组件会重复的大量开销。
- 通过删除第三方存储驱动程序和云提供商，二进制文件变得更小。

## 发布周期

K3s 与上游 Kubernetes 版本保持同步。目标是在几天内与上游版本和次要版本在同一天发布补丁版本。

**v1.20.4+k3s1**: `v1.20.4` 为 K8s 版本，`k3s1` 为补丁版本，如果我们在 `v1.20.4+k3s1` 中发现了一个严重程度较高的错误并需要立即发布修复，我们将发布 `v1.20.4+k3s2`。

## 架构

![](https://camo.githubusercontent.com/fb753e178fce8d3b13a7d8c66a39ad7e37f693a80ae589fd877a732098179c8b/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303865476d5a456c7931676f3079773531766c676a33316976307530337a372e6a7067)

## 参考文档

- 中文文档：https://docs.rancher.cn/k3s
- 英文文档：https://rancher.com/docs/k3s/latest/en/
- Github 地址：https://github.com/k3s-io/k3s

# K3s 安装

## 安装要求

### OS

K3s 有望在大多数现代 Linux 系统上运行:

- SLES: 15 SP3, 15 SP2, 15 SP1
- SLE Micro: 5.1
- OpenSUSE Leap: 15.3
- Ubuntu: 18.04, 20.04
- CentOS: 7.8, 7.9
- Oracle Linux: 8.3、7.9
- RHEL: 7.8, 7.9, 8.2, 8.3, 8.4, 8.5
- Rocky Linux: 8.4

### CPU 和内存

- CPU： 最低 1
- 内存： 最低 512MB（建议至少为 1GB）

### 磁盘

K3s 的性能取决于数据库的性能。为了确保最佳速度，我们建议尽可能使用 SSD。在使用 SD 卡或 eMMC 的 ARM 设备上，磁盘性能会有所不同。

## 单节点安装

![](https://docs.rancher.cn/assets/images/k3s-architecture-single-server-42bb3c4899985b4f6d8fd0e2130e3c0e.png)

#### 启动 K3s Server

```
curl -sfL https://get.k3s.io | sh -
```

国内推荐使用：

```
curl -sfL https://rancher-mirror.oss-cn-beijing.aliyuncs.com/k3s/k3s-install.sh | \
    INSTALL_K3S_MIRROR=cn sh - \
    --system-default-registry "registry.cn-hangzhou.aliyuncs.com"
```

#### 添加 K3s Agent 节点

```
curl -sfL https://get.k3s.io | \
    K3S_URL=https://myserver:6443 \
    K3S_TOKEN=mynodetoken \
    sh -
```

国内推荐使用：

```
curl -sfL https://rancher-mirror.oss-cn-beijing.aliyuncs.com/k3s/k3s-install.sh | INSTALL_K3S_MIRROR=cn \
    K3S_URL=https://myserver:6443 \
    K3S_TOKEN=mynodetoken sh - \
    --system-default-registry "registry.cn-hangzhou.aliyuncs.com"
```

## 高可用安装

### 使用外部数据库实现高可用安装

![](https://docs.rancher.cn/assets/images/k3s-architecture-ha-server-46bf4c38e210246bda5920127bbecd53.png)

K3s 支持以下外部数据存储选项：

- PostgreSQL (经过认证的版本：10.7 和 11.5)
- MySQL (经过认证的版本：5.7)
- MariaDB (经过认证的版本：10.3.20)
- etcd (经过认证的版本：3.3.15)

单节点 k3s server 集群可以满足各种用例，但是对于需要 Kubernetes control-plane 稳定运行的重要环境，您可以在 HA 配置中运行 K3s。一个 K3s HA 集群由以下几个部分组成：

- 两个或多个 server 节点，将为 Kubernetes API 提供服务并运行其他 control-plane 服务。
- 零个或多个 agent 节点，用于运行您的应用和服务。
- 外部数据存储 (与单个 k3s server 设置中使用的嵌入式 SQLite 数据存储相反)
- 固定的注册地址，位于 server 节点的前面，以允许 agent 节点向集群注册

添加第一个 server 节点：

```
curl -sfL https://rancher-mirror.oss-cn-beijing.aliyuncs.com/k3s/k3s-install.sh | INSTALL_K3S_MIRROR=cn \
    sh -s - \
    server \
    --token=SECRET \
    --datastore-endpoint="mysql://username:password@tcp(hostname:3306)/database-name" \
    --system-default-registry "registry.cn-hangzhou.aliyuncs.com"
```

加入其他的 server 节点:

```
# 检索 token
cat /var/lib/rancher/k3s/server/token

# 然后可以使用 token添加其他 server 节点:
curl -sfL https://rancher-mirror.oss-cn-beijing.aliyuncs.com/k3s/k3s-install.sh | INSTALL_K3S_MIRROR=cn \
    sh -s - \
    server \
    --token=SECRET \
    --datastore-endpoint="mysql://username:password@tcp(hostname:3306)/database-name" \
    --system-default-registry "registry.cn-hangzhou.aliyuncs.com"
```

更多集群数据存储选项，请参考 [K3s 文档](https://docs.rancher.cn/docs/k3s/installation/datastore/_index/)

### 嵌入式 DB 的高可用

要在这种模式下运行 K3s，你必须有奇数的 server 节点。我们建议从三个节点开始。

要开始运行，首先启动一个 server 节点，使用 cluster-init 标志来启用集群，并使用一个标记作为共享的密钥来加入其他服务器到集群中。

```
curl -sfL https://rancher-mirror.oss-cn-beijing.aliyuncs.com/k3s/k3s-install.sh | INSTALL_K3S_MIRROR=cn \
    K3S_TOKEN=SECRET sh -s -
    server \
    --cluster-init
```

启动第一台 server 后，使用共享密钥将第二台和第三台 server 加入集群。

```
curl -sfL https://rancher-mirror.oss-cn-beijing.aliyuncs.com/k3s/k3s-install.sh | INSTALL_K3S_MIRROR=cn \
    K3S_TOKEN=SECRET sh -s - server \
    --server https://<ip or hostname of server1>:6443
```

查询 ETCD 集群状态：

```
ETCDCTL_ENDPOINTS='https://172.31.12.136:2379,https://172.31.4.43:2379,https://172.31.4.190:2379' \
ETCDCTL_CACERT='/var/lib/rancher/k3s/server/tls/etcd/server-ca.crt' \
ETCDCTL_CERT='/var/lib/rancher/k3s/server/tls/etcd/server-client.crt' \
ETCDCTL_KEY='/var/lib/rancher/k3s/server/tls/etcd/server-client.key' \
ETCDCTL_API=3 etcdctl endpoint status --write-out=table
```

> etcd 证书默认目录：/var/lib/rancher/k3s/server/tls/etcd etcd 数据默认目录：/var/lib/rancher/k3s/server/db/etcd

# 访问集群

存储在/etc/rancher/k3s/k3s.yaml 的 kubeconfig 文件用于对 Kubernetes 集群的访问。如果你已经安装了上游的 Kubernetes 命令行工具，如 kubectl 或 helm，你需要用正确的 kubeconfig 路径配置它们。这可以通过导出 KUBECONFIG 环境变量或调用--kubeconfig 命令行标志来完成。详情请参考下面的例子。

```
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
kubectl get pods --all-namespaces
helm ls --all-namespaces
```

# 网络选项

默认情况下，K3s 将以 flannel 作为 CNI 运行，使用 VXLAN 作为默认后端。CNI 和默认后端都可以通过参数修改。

## Flannel 选项

Flannel 的默认后端是 VXLAN。要启用加密，请使用下面的 IPSec（Internet Protocol Security）或 WireGuard 选项。

| CLI Flag 和 Value             | 描述                                                                    |
| :---------------------------- | :---------------------------------------------------------------------- |
| `--flannel-backend=vxlan`     | (默认) 使用 VXLAN 后端。                                                |
| `--flannel-backend=ipsec`     | 使用 IPSEC 后端，对网络流量进行加密。                                   |
| `--flannel-backend=host-gw`   | 使用 host-gw 后端。                                                     |
| `--flannel-backend=wireguard` | 使用 WireGuard 后端，对网络流量进行加密。可能需要额外的内核模块和配置。 |

## 自定义 CNI

使用 `--flannel-backend=none` 运行 K3s，然后在安装你选择的 CNI。

#### Calico

参考：https://docs.projectcalico.org/getting-started/kubernetes/k3s/quickstart

#### Cilium

参考：https://docs.cilium.io/en/v1.9/gettingstarted/k3s/

# 仪表盘

## 仪表盘

- Kubernetes Dashboard
- kube-explorer
- Rancher UI

## 1. Kubernetes Dashboard

参考 [K3s 官网](http://docs.rancher.cn/docs/k3s/installation/kube-dashboard/_index/)

## 2. Rancher UI

可以将 K3s 导入到 Rancher UI 中去管理，参考 [Rancher 官网](http://docs.rancher.cn/docs/rancher2/cluster-provisioning/imported-clusters/_index/#%E5%AF%BC%E5%85%A5-k3s-%E9%9B%86%E7%BE%A4)

导入 K3s 集群时，Rancher 会将其识别为 K3s，除了其他导入的集群支持的功能之外，Rancher UI 还提供以下功能：

- 能够升级 K3s 版本。
- 能够配置在升级集群时，同时可以升级的最大节点数。
- 在主机详情页，能够查看（不能编辑）启动 K3s 集群时每个节点的 K3s 配置参数和环境变量。

## 3. kube-explorer

项目地址：https://github.com/cnrancher/kube-explorer

kube-explorer 是 Kubernetes 的便携式资源管理器，没有任何依赖。 并提供了一个几乎完全无状态的 Kubernetes 资源管理器。

#### 使用

从[发布页面](https://github.com/cnrancher/kube-explorer/releases)下载二进制文件

运行：

```
./kube-explorer --kubeconfig=xxxx --http-listen-port=9898 --https-listen-port=0
```

然后，打开浏览器访问 http://x.x.x.x:9898

# K3s 升级

## 使用安装脚本升级 K3s

要从旧版本升级 K3s，你可以使用相同的标志重新运行安装脚本，例如：

```
curl -sfL https://get.k3s.io | sh -

# 国内环境可使用：
curl -sfL https://rancher-mirror.oss-cn-beijing.aliyuncs.com/k3s/k3s-install.sh | INSTALL_K3S_MIRROR=cn sh -
```

如果你想升级到一个特定 channel 的较新版本（如最新），你可以指定 channel：

```
curl -sfL https://get.k3s.io | INSTALL_K3S_CHANNEL=latest sh -
```

如果你想升级到特定的版本，你可以运行以下命令:

```
curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION=vX.Y.Z-rc1 sh -
```

## 使用二进制文件手动升级 K3s

1. 从[发布](https://github.com/rancher/k3s/releases)下载所需版本的 K3s 二进制文件
2. 将下载的二进制文件复制到 `/usr/local/bin/k3s`（或您所需的位置）
3. 重启 k3s

## 自动升级

使用 Rancher 的 system-upgrad-controller 来管理 K3s 集群升级。

#### 安装 system-upgrade-controller

```
kubectl apply -f https://github.com/rancher/system-upgrade-controller/releases/latest/download/system-upgrade-controller.yaml
```

#### 配置计划

建议您最少创建两个计划：升级 server（master）节点的计划和升级 agent（worker）节点的计划。根据需要，您可以创建其他计划来控制跨节点的滚动升级。以下两个示例计划将把您的集群升级到 K3s v1.17.4+k3s1。创建计划后，控制器将接收这些计划并开始升级您的集群。

```
# Server plan
apiVersion: upgrade.cattle.io/v1
kind: Plan
metadata:
  name: server-plan
  namespace: system-upgrade
spec:
  concurrency: 1
  cordon: true
  nodeSelector:
    matchExpressions:
    - key: node-role.kubernetes.io/master
      operator: In
      values:
      - "true"
  serviceAccountName: system-upgrade
  upgrade:
    image: rancher/k3s-upgrade
  version: v1.17.4+k3s1
---
# Agent plan
apiVersion: upgrade.cattle.io/v1
kind: Plan
metadata:
  name: agent-plan
  namespace: system-upgrade
spec:
  concurrency: 1
  cordon: true
  nodeSelector:
    matchExpressions:
    - key: node-role.kubernetes.io/master
      operator: DoesNotExist
  prepare:
    args:
    - prepare
    - server-plan
    image: rancher/k3s-upgrade
  serviceAccountName: system-upgrade
  upgrade:
    image: rancher/k3s-upgrade
  version: v1.17.4+k3s1
```

# 备份和恢复

K3s 的备份和恢复方式取决于使用的数据存储类型：

- 使用嵌入式 SQLite 数据存储进行备份和恢复
- 使用外部数据存储进行备份和恢复
- 使用嵌入式 etcd 数据存储进行备份和恢复

## 使用嵌入式 SQLite 数据存储进行备份和恢复

#### 方式 1：备份/恢复数据目录

- 备份

```
# cp -rf /var/lib/rancher/k3s/server/db /opt/db
```

- 恢复

```
# systemctl stop k3s
# rm -rf /var/lib/rancher/k3s/server/db
# cp -rf /opt/db /var/lib/rancher/k3s/server/db
# systemctl start k3s
```

#### 方式 2：通过 SQLite cli

- 备份

```
# sqlite3 /var/lib/rancher/k3s/server/db/state.db
SQLite version 3.22.0 2018-01-22 18:45:57
Enter ".help" for usage hints.
sqlite> .backup "/opt/kine.db"
sqlite> .exit
```

- 恢复

```
# systemctl stop k3s

# sqlite3 /var/lib/rancher/k3s/server/db/state.db
SQLite version 3.22.0 2018-01-22 18:45:57
Enter ".help" for usage hints.
sqlite> .restore '/opt/kine.db'
sqlite> .exit

# systemctl start k3s
```

## 使用外部数据存储进行备份和恢复

当使用外部数据存储时，备份和恢复操作是在 K3s 之外处理的。数据库管理员需要对外部数据库进行备份，或者从快照或转储中进行恢复。我们建议将数据库配置为执行定期快照。

- 备份

```
# mysqldump -uroot -p --all-databases --master-data > k3s-dbdump.db
```

- 恢复

停止 K3s 服务

```
# systemctl stop k3s
```

恢复 mysql 数据

```
mysql -uroot -p  < k3s-dbdump.db
```

启动 K3s 服务

```
# systemctl start k3s
```

## 使用嵌入式 etcd 数据存储进行备份和恢复

- 创建快照

K3s 默认启用快照。快照目录默认为 `/var/lib/rancher/k3s/server/db/snapshots`。要配置快照间隔或保留的快照数量，请参考：

| 参数                            | 描述                                                                                                                           |
| :------------------------------ | :----------------------------------------------------------------------------------------------------------------------------- |
| `--etcd-disable-snapshots`      | 禁用自动 etcd 快照                                                                                                             |
| `--etcd-snapshot-schedule-cron` | 以 Cron 表达式的形式配置触发定时快照的时间点，例如：每 5 小时触发一次`* */5 * * *`，默认值为每 12 小时触发一次：`0 */12 * * *` |
| `--etcd-snapshot-retention`     | 保留的快照数量，默认值为 5。                                                                                                   |
| `--etcd-snapshot-dir`           | 保存数据库快照的目录路径。(默认位置：`${data-dir}/db/snapshots`)                                                               |
| `--cluster-reset`               | 忘记所有的对等体，成为新集群的唯一成员，也可以通过环境变量`[$K3S_CLUSTER_RESET]`进行设置。                                     |
| `--cluster-reset-restore-path`  | 要恢复的快照文件的路径                                                                                                         |

- 从快照恢复集群

当 K3s 从备份中恢复时，旧的数据目录将被移动到`/var/lib/rancher/k3s/server/db/etcd-old/`。然后 K3s 会尝试通过创建一个新的数据目录来恢复快照，然后从一个带有一个 etcd 成员的新 K3s 集群启动 etcd。

要从备份中恢复集群，运行 K3s 时，请使用`--cluster-reset`选项运行 K3s，同时给出`--cluster-reset-restore-path`，如下：

```shell
./k3s server \
  --cluster-reset \
  --cluster-reset-restore-path=<PATH-TO-SNAPSHOT>
```

**结果:** 日志中出现一条信息，**Etcd 正在运行，现在需要在没有 `--cluster-reset` 标志的情况下重新启动。备份和删除每个对等 etcd 服务器上的 ${datadir}/server/db 并重新加入节点**

### S3 兼容 API 支持

K3s 支持向具有 S3 兼容 API 的系统写入 etcd 快照和从系统中恢复 etcd 快照。S3 支持按需和计划快照。

下面的参数已经被添加到 `server` 子命令中。这些标志也存在于 `etcd-snapshot` 子命令中，但是 `--etcd-s3` 部分被删除以避免冗余。

```
k3s etcd-snapshot \
  --s3 \
  # --s3-endpoint minio.kingsd.top:9000 \
  --s3-bucket=<S3-BUCKET-NAME> \
  --s3-access-key=<S3-ACCESS-KEY> \
  --s3-secret-key=<S3-SECRET-KEY>
```

要从 S3 中执行按需的 etcd 快照还原，首先确保 K3s 没有运行。然后运行以下命令：

```
k3s server \
  --cluster-init \
  --cluster-reset \
  --etcd-s3 \
  # --etcd-s3-endpoint minio.kingsd.top:9000 \
  --cluster-reset-restore-path=<SNAPSHOT-NAME> \
  --etcd-s3-bucket=<S3-BUCKET-NAME> \
  --etcd-s3-access-key=<S3-ACCESS-KEY> \
  --etcd-s3-secret-key=<S3-SECRET-KEY>
```

# Helm

## 自动部署 Helm charts

在`/var/lib/rancher/k3s/server/manifests`中找到的任何 Kubernetes 清单将以类似`kubectl apply`的方式自动部署到 K3s。以这种方式部署的 manifests 是作为 AddOn 自定义资源来管理的，可以通过运行`kubectl get addon -A`来查看。你会发现打包组件的 AddOns，如 CoreDNS、Local-Storage、Traefik 等。AddOns 是由部署控制器自动创建的，并根据它们在 manifests 目录下的文件名命名。

也可以将 Helm Chart 作为 AddOns 部署。K3s 包括一个[Helm Controller](https://github.com/rancher/helm-controller/)，它使用 HelmChart Custom Resource Definition(CRD)管理 Helm Chart。

## 使用 Helm CRD

[HelmChart CRD](https://github.com/rancher/helm-controller#helm-controller)捕获了大多数你通常会传递给`helm`命令行工具的选项。下面是一个例子，说明如何从默认的 Chart 资源库中部署 Grafana，覆盖一些默认的 Chart 值。请注意，HelmChart 资源本身在 `kube-system` 命名空间，但 Chart 资源将被部署到 `monitoring` 命名空间。

```yaml
apiVersion: helm.cattle.io/v1
kind: HelmChart
metadata:
  name: grafana
  namespace: kube-system
spec:
  chart: stable/grafana
  targetNamespace: monitoring
  set:
    adminPassword: "NotVerySafePassword"
  valuesContent: |-
    image:
      tag: master
    env:
      GF_EXPLORE_ENABLED: true
    adminUser: admin
    sidecar:
      datasources:
        enabled: true
```

### HelmChart 字段定义

| 字段                 | 默认值  | 描述                                                                  | Helm Argument / Flag Equivalent |
| :------------------- | :------ | :-------------------------------------------------------------------- | :------------------------------ |
| name                 | N/A     | Helm Chart 名称                                                       | NAME                            |
| spec.chart           | N/A     | 仓库中的 Helm Chart 名称，或完整的 HTTPS URL（.tgz）。                | CHART                           |
| spec.targetNamespace | default | Helm Chart 目标命名空间                                               | `--namespace`                   |
| spec.version         | N/A     | Helm Chart 版本(从版本库安装时使用的版本号)                           | `--version`                     |
| spec.repo            | N/A     | Helm Chart 版本库 URL 地址                                            | `--repo`                        |
| spec.helmVersion     | v3      | Helm 的版本号，可选值为 `v2` 和`v3`，默认值为 `v3`                    | N/A                             |
| spec.set             | N/A     | 覆盖简单的默认 Chart 值。这些值优先于通过 valuesContent 设置的选项。  | `--set` / `--set-string`        |
| spec.jobImage        |         | 指定安装 helm chart 时要使用的镜像。如：rancher/klipper-helm:v0.3.0。 |                                 |
| spec.valuesContent   | N/A     | 通过 YAML 文件内容覆盖复杂的默认 Chart 值。                           | `--values`                      |
| spec.chartContent    | N/A     | Base64 编码的 Chart 存档.tgz - 覆盖 spec.chart。                      | CHART                           |

放在`/var/lib/rancher/k3s/server/static/`中的内容可以通过 Kubernetes APIServer 从集群内匿名访问。这个 URL 可以使用`spec.chart`字段中的特殊变量`%{KUBERNETES_API}%`进行模板化。例如，打包后的 Traefik 组件从`https://%{KUBERNETES_API}%/static/charts/traefik-1.81.0.tgz`加载其 Chart。

## 使用 HelmChartConfig 自定义打包的组件

为了允许覆盖作为 HelmCharts（如 Traefik 或其他通过 helm crd 部署的应用）部署的打包组件的值，从 v1.19.0+k3s1 开始的 K3s 版本支持通过 HelmChartConfig CRD 部署。HelmChartConfig 资源必须与其对应的 HelmChart 的名称和命名空间相匹配，并支持提供额外的 "valuesContent"，它作为一个额外的值文件传递给`helm`命令。

> **注意：** HelmChart 的`spec.set`值覆盖了 HelmChart 和 HelmChartConfig 的`spec.valuesContent`设置。

```yaml
apiVersion: helm.cattle.io/v1
kind: HelmChartConfig
metadata:
  name: grafana
  namespace: kube-system
spec:
  valuesContent: |-
    service:
      type: NodePort
```

如果要自定义打包后的 Traefik 入口配置，你可以创建一个名为`/var/lib/rancher/k3s/server/manifests/traefik-config.yaml`的文件，并将其填充为以下内容。

# Service Load Balancer

K3s 提供了一个名为[Klipper Load Balancer](https://github.com/rancher/klipper-lb)的负载均衡器，它可以使用可用的主机端口。 允许创建 LoadBalancer 类型的 Service，但不包括 LB 的实现。某些 LB 服务需要云提供商，例如 Amazon EC2 或 Microsoft Azure。相比之下，K3s service LB 使得可以在没有云提供商的情况下使用 LB 服务。

## Service LB 如何工作

K3s 创建了一个控制器，该控制器为 service load balancer 创建了一个 Pod，这个 Pod 是[Service](https://kubernetes.io/docs/concepts/services-networking/service/)类型的 Kubernetes 对象。

对于每个 service load balancer，都会创建一个[DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)。 DaemonSet 在每个节点上创建一个前缀为`svc`的 Pod。

Service LB 控制器会监听其他 Kubernetes Services。当它找到一个 Service 后，它会在所有节点上使用 DaemonSet 为该服务创建一个代理 Pod。这个 Pod 成为其他 Service 的代理，例如，来自节点上 8000 端口的请求可以被路由到端口 8888 上的工作负载。

如果创建多个 Services，则为每个 Service 创建一个单独的 DaemonSet。

只要使用不同的端口，就可以在同一节点上运行多个 Services。

如果您尝试创建一个在 80 端口上监听的 Service LB，Service LB 将尝试在集群中找到 80 端口的空闲主机。如果该端口没有可用的主机，LB 将保持 Pending 状态。

## 用法

在 K3s 中创建一个[LoadBalancer 类型的 Service](https://kubernetes.io/docs/concepts/services-networking/service/#loadbalancer)。

### 从节点中排除 Service LB

要排除节点使用 Service LB，请将以下标签添加到不应排除的节点上：

```
svccontroller.k3s.cattle.io/enablelb
```

如果使用标签，则 service load balancer 仅在标记的节点上运行。

## 禁用 Service LB

要禁用嵌入式 LB，请使用`--disable servicelb`选项运行 k3s server。

如果您希望运行其他 LB，例如 MetalLB，这是必需的。

# 使用 docker 作为容器运行时

```
curl -sfL https://rancher-mirror.oss-cn-beijing.aliyuncs.com/k3s/k3s-install.sh | INSTALL_K3S_MIRROR=cn \
    INSTALL_K3S_VERSION=v1.23.9+k3s1 sh -s - \
    --docker
```

# 周边项目

- [Rancher Desktop](https://github.com/rancher-sandbox/rancher-desktop)：Rancher Desktop 是一款在桌面上提供容器和 Kubernetes 管理的应用。它可以在 Windows、macOS 和 Linux 上运行
- [K3d](https://github.com/k3d-io/k3d)：K3d 创建容器化的 k3s 集群。这意味着，你可以使用 docker 在单台机器上启动多节点 k3s 集群。
- [K3sup](https://github.com/alexellis/k3sup)：K3sup 是一个轻量级应用程序，可以在任何本地或远程主机上安装和使用 k3s。你只需要 ssh 访问权限和 k3sup 二进制文件即可立即获得 kubectl 访问权限。
- [AutoK3s](https://github.com/cnrancher/autok3s)：AutoK3s 是用于简化 K3s 集群管理的轻量级工具，你可以使用 AutoK3s 在任何地方运行 K3s 服务。支持：阿里云、腾讯云、AWS、Google、K3d、Harvester、Native

# 关注 K3s

![](https://camo.githubusercontent.com/2b75358b397e2e6b0517fe4389b28af657ccfde24f9a7ac51571f516bab3b50a/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303865476d5a456c7931676f30327275346334796a333072363065347769352e6a7067)
