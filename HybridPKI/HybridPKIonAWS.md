## 在AWS上部署混合云PKI


该文档由以下原始文档汉化而来：https://github.com/aws-samples/hybrid-pki-on-aws 

务必：

* 更改此 README 中的标题
* 在 GitHub 上编辑你的仓库描述

## 安全

有关更多信息，请参阅 [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications)。

## 许可证

该库根据 MIT-0 许可证获得许可。 请参阅LICENSE文件。
该解决方案使用可在本地或 Amazon VPC 中运行的离线根 CA，而二级 Windows CA 在 EC2 实例上运行并与 CloudHSM 集成以进行密钥管理和存储。 为了将 PKI 与外部访问隔离开来，CloudHSM 集群部署在受保护的子网中，EC2 实例部署在私有子网中，并且主机 VPC 具有与本地网络的站点到站点网络连接。 Amazon EC2 卷使用 AWS KMS 客户管理的密钥进行加密。 用户和设备通过网络负载均衡器（NLB）连接并注册到 PKI 接口。
该解决方案还包括一个从属 ACM 私有 CA，用于颁发证书，这些证书将安装在与 AWS Certificate Manager（ACM ）集成的 AWS 服务上。 例如，ELB、CloudFront 和 API 网关。 这样一来，用户看到的证书始终来自您组织的内部 PKI。

##  解决方案概览

如果您需要新的私有 PKI，或者想要从使用加密服务提供程序 (CSP) 的现有旧 PKI 升级到使用 Windows 下一代加密技术 (CNG) 的安全 PKI，则可以使用此混合 PKI。 混合 PKI 设计使您能够在组织的整个 IT 基础设施（从本地到多个 AWS 网络）中无缝管理加密密钥。

解决方案架构如上图-图3所示。该解决方案使用可在本地或Amazon VPC中运行的离线根CA，而从属Windows CA运行在EC2实例上并与CloudHSM集成以进行密钥管理 和存储。 为了将 PKI 与外部访问隔离开来，CloudHSM 集群部署在受保护的子网中，EC2 实例部署在私有子网中，并且主机 VPC 具有与本地网络的站点到站点网络连接。 Amazon EC2 卷使用 AWS KMS 客户管理的密钥进行加密。 用户和设备通过网络负载均衡器连接并注册到 PKI 接口。

该解决方案还包括一个从属 ACM 私有 CA，用于颁发证书，这些证书将安装在与 ACM 集成的 AWS 服务上。 例如，ELB、CloudFront 和 API 网关。 这样一来，用户看到的证书始终来自您组织的内部 PKI。
![[Pasted image 20230316234521.png]]


**在 AWS 中部署此混合内部 PKI 的先决条件**

部署和配置此解决方案需要使用 AWS 云、Windows Server 和 AD CS 的经验。用于部署云资源的 AWS 帐户。在 Windows 2016 或更新版本上运行的离线根 CA，用于签署 CloudHSM 和颁发 CA，包括私有 CA 和 Windows CA。 这是一篇 AWS 快速入门文章，用于在 VPC 中部署根 CA。 我们建议在其自己的 AWS 账户中安装 Windows 根 CA。一个至少有四个子网的 VPC。 两个或多个公共子网和两个或多个私有子网，跨两个或多个 AZ，具有安全防火墙规则，例如 HTTPS 通过负载均衡器与您的 PKI Web 服务器通信，以及 DNS、RDP 和其他端口在您的内部进行通信 组织网络。 您可以使用此 CloudFormation 示例 VPC 模板来帮助您开始进行 PKI VPC 配置。从您的 VPC 到本地网络和其他 VPC 的站点到站点 AWS Direct Connect 或 VPN 连接，以安全地管理多个网络。二级 CA 的 Windows 2016 EC2 实例。可以访问托管 PKI 服务器的 VPC 的 Active Directory 环境。 这是 Windows 企业 CA 实施所必需的。部署解决方案，以下 CloudFormation 代码和说明将帮助您部署和配置上述架构图中显示的所有 AWS 组件。 为实施该解决方案，您将通过 AWS 管理控制台部署一系列 CloudFormation 模板。
如果您不熟悉 CloudFormation，可以从 AWS CloudFormation 入门了解它。 此解决方案的模板可以使用 CloudFormation 控制台、AWS 服务目录或代码管道进行部署。

## 准备

**下载并查看模板包**

为了更轻松地部署此内部 PKI 解决方案的组件，您下载并部署了一个模板包。 该捆绑包包括一组 CloudFormation 模板和一个 PowerShell 脚本，用于完成 CloudHSM 和 Windows CA 服务器之间的集成。
此解决方案部署的资源会产生额外费用。 这些资源包括：CloudHSM、ACM PCA、ELB、EC2s、S3 和 KMS。该解决方案还部署了一些 AWS Identity and Access Management (IAM) 角色和策略。下载模板包，从 AWS GitHub 下载或克隆解决方案源代码存储库。查看每个模板中的说明以获取更多说明。
部署 CloudFormation 模板。现在您已经下载了模板，使用 CouldFormation 控制台部署它们。

## 步骤

**1. 将此模板部署到现有 VPC 中以创建受保护的子网以部署 CloudHSM 集群**

导航到 CloudFormation 控制台。选择适当的 AWS 区域，然后选择创建堆栈。选择上传模板文件。
选择 01_PKI_Automated-VPC_Modifications.yaml 作为 CloudFormation 堆栈文件，然后选择下一步。在指定堆栈详细信息页面上，输入堆栈名称和参数。 一些参数有一个下拉列表，您可以使用它来选择现有值。选择 Next、Next 和 Create Stack。

**2.部署 PKI CDP S3 存储桶模板**

此模板根据您的输入为 CRL 和 AIA 分发点创建一个 S3 存储桶，其中初始存储桶策略允许从 PKI VPC 以及本地网络中的 PKI 用户和设备进行访问。 要授予对其他 AWS 账户、VPC 和本地网络的访问权限，请参阅模板中的说明。导航到 CloudFormation 控制台。选择上传模板文件。选择 02_PKI_Automated-Central-PKI_CDP-S3bucket.yaml 作为 CloudFormation 堆栈文件，然后选择下一步。在指定堆栈详细信息页面上，输入堆栈名称和参数。选择下一步，下一步然后创建堆栈。

**3.部署 ACM Private CA 从属模板**

此步骤预配由现有 Windows 根 CA 签名的 ACM 私有 CA。 使用 CloudFormation 预置您的私有 CA 可以使用 Windows 根 CA 对 CA 进行签名。
导航到 CloudFormation 控制台。选择上传模板文件。选择 03_PKI_Automated-ACMPrivateCA-Provisioning.yaml 作为 CloudFormation 堆栈文件，然后选择下一步。在指定堆栈详细信息页面上，输入堆栈名称和参数。 一些参数有一个下拉列表，您可以使用它来选择现有值。选择 Next、Next 和 Create Stack。部署网络负载均衡器模板在此步骤中，您供应网络负载均衡器。

**5. 导航到 CloudFormation 控制台**

选择上传模板文件。选择 05_PKI_Automated-LoadBalancer-Provisioning.yaml 作为 CloudFormation 堆栈文件，然后选择下一步。
在指定堆栈详细信息页面上，输入堆栈名称和参数。 一些参数是自动填写的，或者有一个下拉列表，您可以使用它来选择现有值。选择 Next、Next 和 Create Stack。

**6. 部署 HTTPS 侦听器配置模板**

以下步骤使用负载均衡器的初始配置创建 HTTPS 侦听器。
导航到 CloudFormation 控制台：选择上传模板文件。选择 06_PKI_Automated-HTTPS-Listener.yaml 作为 CloudFormation 堆栈文件，然后选择下一步。在指定堆栈详细信息页面上，输入堆栈名称和参数。 一些参数是自动填写的，或者有一个下拉列表，您可以使用它来选择现有值。选择 Next、Next 和 Create Stack。

**4. 部署 AWS KMS CMK 模板**

在此步骤中，您将创建一个 AWS KMS CMK 来加密 EC2 EBS 卷和其他资源。 此解决方案中的 EC2 实例需要这样做。打开 CloudFormation 控制台。选择上传模板文件。选择 04_PKI_Automated-KMS_CMK-Creation.yaml 作为 CloudFormation 堆栈文件，然后选择下一步。在指定堆栈详细信息页面上，输入堆栈名称和参数。选择 Next、Next 和 Create Stack。

**7. 部署 Windows EC2 实例供应模板**

此模板在现有 VPC 中提供一个专门构建的 Windows EC2 实例。 它将为 Windows CA 提供一个 EC2 实例，使用 KMS 来加密 EBS 卷，一个 IAM 实例配置文件，并自动在您的实例上安装 SSM 代理。
它还具有可选功能和灵活性。 例如，模板可以自动创建新的目标组，或将实例添加到现有的目标组。 它还可以配置侦听器规则、创建 Route 53 记录并自动加入 Active Directory 域。
注意：需要 AWS KMS CMK 和 IAM 角色来配置 EC2，而目标组、侦听器规则和域加入功能是可选的。
导航到 CloudFormation 控制台。选择上传模板文件。选择 07_PKI_Automated-EC2-Servers-Provisioning.yaml 作为 CloudFormation 堆栈文件，然后选择下一步。在指定堆栈详细信息页面上，输入堆栈名称和参数。 一些参数是自动填写的，或者有一个下拉列表，您可以使用它来选择现有值。
注意：如果您没有将 EC2 实例加入 Active Directory 域，则不需要参数列表末尾的可选属性部分。
选择 Next、Next 和 Create Stack。

**将 CloudHSM 集群集成到 Windows Server AD CS**

打开脚本 09_PKI_AWS_CloudHSM-Windows_CA-Integration-Playbook.txt 并按照说明完成 CloudHSM 与 Windows 服务器的集成。


### 参考文档
1. https://aws.amazon.com/cn/blogs/security/how-to-implement-a-hybrid-pki-solution-on-aws/