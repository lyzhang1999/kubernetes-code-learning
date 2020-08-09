# 声明式 API
我们知道，命令式操作即执行一条命令，例如：
```
docker service create --name nginx --replicas 2 nginx
```
而声明式操作则基于声明的操作“对象”，通过对这些“对象”不断调协使得对象的状态接近声明的状态。

常见的声明式可以是 `YAML` 格式，或者是 `JSON` 格式来描述声明式对象的状态。

在 `Kubernetes` 中，`YAML` 声明文件是 `Kubernetes` 实现声明式 `API` 核心。

在 `Kubernetes` 中，执行 `kubectl apply -f` 命令就是一次典型的声明式 API 交互。
另外，如果对已经应用的 `YAML` 文件进行修改，并进行重新 `kubectl apply -f` 操作，则是相当于对一个原有的对象进行 `PATCH` 操作。

# Kubernetes 声明式 API
`Kubernetes` 中，所有对象如图：
![Kubernetes API](apis.png)

例如，对于一个 `CronJob` 对象，yaml 文件可能是这样的：
```
apiVersion: batch/v2alpha1
kind: CronJob...
```
在这个 YAML 文件中，“CronJob” 就是这个 API 对象的资源类型（Resource），“batch” 就是它的组（Group），v2alpha1 就是它的版本（Version）。

当提交一个 `CronJob` 对象时，大致的流程图是：

![Kubernetes API](apis-apply.png)

1. 当创建 CronJob POST 请求时，client 将 YAML 信息提交到 APIServer，而 APIServer 会做一些前置性的动作，例如 ratelimit、鉴权。
2. 进入 MUX 和 Routes 流程，通过 url 绑定 Handler，业务逻辑由 Handler 执行。
3. 执行 convert 动作，将 yaml 转化为对象，并创建 CronJob 对象。
4. Admission() 和 Validation() 操作。
5. 最后，会把 api 对象转化进行序列化操作，并调用 etcd api 进行存储。

# 自定义 crd 对象
从 Kubernetes 1.7 之后开始，引入了 CRD 的机制，此机制相当于可插拔的用户定义的 API 列表。

例如我们声明一个 Network 对象：
```
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: networks.samplecrd.k8s.io
spec:
  group: samplecrd.k8s.io
  version: v1
  names:
    kind: Network
    plural: networks
  scope: Namespaced
```
在这个 CRD 中，指定了 `group` 为 `samplecrd.k8s.io`，`version: v1` 这样的 API 信息，也指定了这个 CR 的资源类型叫作 Network，复数（plural）是 networks。然后，我还声明了它的 scope 是 Namespaced，即：我们定义的这个 Network 是一个属于 Namespace 的对象，类似于 Pod。

现在，就可以创建这个具体的对象了：
```
apiVersion: samplecrd.k8s.io/v1
kind: Network
metadata:
  name: example-network
spec:
  cidr: "192.168.0.0/16"
  gateway: "192.168.0.1"
```

# 