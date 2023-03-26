## 5 [optional]在EKS上传输中的数据保护


### 模块介绍：

这一部分，我们将重点关注容器工作负载，特别是向 EKS 上的 Kubernetes 部署颁发私有证书的能力。 许多客户正在使用容器来更好地划分他们的计算工作负载，并希望利用 AWS Certificate Manager Private CA 来安全地管理他们的证书颁发机构和分发到计算节点的终端实体证书。

---

使用 AWS Private CA 可以通过三种方式保护 EKS 工作负载的传输数据：

- **在Ingress控制器处终止 TLS**
- **在各个 pod 终止 TLS**
- **pod 之间的相互 TLS**

---

该模块将专注于在入口控制器和单个 pod 级别终止 TLS，以促进端到端加密。 pod 之间的相互 TLS 是一个更复杂的主题，将在其自己的模块中介绍。 请注意，您也可以在负载均衡器上终止 TLS，但这通常将_公共证书_与 ACM 一起使用，因此不在本模块的范围内。

---


### 模块组件：


- **Kubernetes** 是一个开源系统，用于自动部署、扩展和管理容器化应用程序。
- **Amazon EKS** 是一种托管服务，您可以使用它在 AWS 上运行 Kubernetes，而无需安装、操作和维护您自己的 Kubernetes 控制平面或节点。
- **cert-manager** 是 Kubernetes 的附加组件，用于提供 TLS 证书管理。 cert-manager 请求证书，将它们分发到 Kubernetes 容器，并自动更新证书。 cert-manager 确保证书有效和最新，并尝试在到期前的适当时间更新证书。
- **AWS Private CA** 支持创建私有 CA 层次结构，包括根和从属 CA，而无需运营本地 CA 的投资和维护成本。 借助 AWS Private CA，您可以颁发证书以验证内部用户、计算机、应用程序、服务、服务器和其他设备，以及签署计算机代码。 私有 CA 的私钥存储在经过 FIPS 140-2 认证的 AWS 托管硬件安全模块 (HSM) 中，与 Kubernetes 中的默认 CA 相比，提供更好的安全配置文件。 私有证书有助于识别和保护私有网络上连接的资源（例如服务器、移动和物联网设备以及应用程序）之间的通信。
- **AWS Private CA Issuer 插件** Kubernetes 容器和应用程序使用数字证书通过 TLS 提供安全身份验证和加密。 使用此插件，cert-manager 从私有 CA 请求 TLS 证书。 该集成支持一系列配置中的 TLS 证书自动化，包括在入口处、在 pod 上以及 pod 之间的相互 TLS。 您可以将 AWS Private CA Issuer 插件与 Amazon Elastic Kubernetes Service、AWS 上的自我管理 Kubernetes 和本地 Kubernetes 结合使用。
- **AWS 负载均衡器**控制器管理 Kubernetes 集群的 AWS 弹性负载均衡器。 控制器提供以下资源。
     - 创建 Kubernetes Ingress 时的**AWS 应用程序负载均衡器** (ALB)。
     - 当您创建 LoadBalancer 类型的 Kubernetes 服务时，**AWS 网络负载均衡器** (NLB)。




### 5.1 TLS 终止选项

### 我应该在哪里终止 TLS？
终止 TLS 连接的方式和位置取决于您的用例、安全策略以及是否需要遵守法规要求。 本节讨论四种经常用于终止 TLS 的不同用例。

**在负载均衡器：** 在负载均衡器级别终止 TLS 的最常见用例是使用公共信任的证书。 这个用例易于部署，证书绑定到负载均衡器本身。 例如，您可以使用 AWS 颁发公共证书并将其与 AWS NLB 绑定。 您可以从如何使用 ACM 终止 Amazon EKS 工作负载上的 HTTPS 流量？了解更多信息 本模块将不涉及案例，因为它使用 ACM 的公共证书，而不是 AWS Private CA 的私有证书

**在入口处：** 如果对端到端加密没有严格要求，可以将此处理卸载到入口控制器或NLB。 这有助于您优化工作负载的性能并使其更易于配置和管理。 我们将在本模块的下一节中检查此用例

**在 pod 上**： 在 Kubernetes 中，pod 是最小的可部署计算单元，它封装了一个或多个应用程序。 从客户端一直到 Kubernetes pod 的流量的端到端加密提供了一种安全的通信模型，其中 TLS 在 Kubernetes 集群内的 pod 处终止。 这对于满足某些安全要求可能很有用。 我们稍后将在本模块中重点介绍此用例

**Pod 之间的相互 TLS：** 此用例侧重于 Kubernetes 集群内数据流动的传输加密。 您可以将 AWS Private CA Issuer 插件与 cert-manager 结合使用，以使用 AWS Private CA 颁发证书以保护 pod 之间的通信。 您可以在How to use AWS Private CA for enabling mTLS in AWS App Mesh

