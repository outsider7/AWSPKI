# 部署 Microsoft PKI

该文档基于以下文档进行汉化
https://aws-quickstart.github.io/quickstart-microsoft-pki/ 

## 概览

本指南提供有关在 AWS 云上部署 Microsoft PKI 快速入门参考架构的说明。

此快速入门适用于需要在其 Active Directory 环境中部署根和二级证书颁发机构 (CA) 的系统管理员和 IT 工程师。

## AWS上的Microsoft PKI

公钥基础设施 (PKI) 创建、管理、分发、存储和注销数字证书。 Windows 环境使用数字证书来保护多种类型的连接。 
通过 AWS 账户中 Windows 托管的 PKI，您可以维护自己的证书。 此功能可帮助您减少不安全、未签名的网络流量。 要在 Windows 上部署 PKI 环境，您需要在一台或多台 Windows 服务器上安装和配置证书颁发机构 (CA) 角色。

此 Quick Start 部署一层或两层 PKI 基础结构。 使用一层基础架构，Windows EC2 实例加入您的 Active Directory 域并安装 CA 角色，成为企业 CA。 使用两层基础架构，Windows EC2 实例加入您的 Active Directory 域，安装 CA 角色，并提升为域的根 CA； 然后，第二个 Windows EC2 实例加入域并成为一个 Suborodinate CA，之后 Root CA 被关闭。 两层 PKI 模型被认为比一层模型更安全； 由于根 CA 保持离线状态，它可以在从属 CA 受到威胁的情况下启动，然后可以生成一组新的密钥。 两层模型也更适合高可用性，因为可以将多个从属 CA 添加到环境中。

## AWS 费用

您负责 AWS 服务的费用以及运行此 Quick Start 时使用的任何第三方许可证的费用。 使用快速入门无需支付额外费用。

快速入门的 AWS CloudFormation 模板包括您可以自定义的配置参数。 某些设置（例如实例类型）会影响部署成本。 有关成本估算，请参阅您使用的每项 AWS 服务的定价页面。 价格随时可能更改。


## 软件许可证

此快速入门部署运行 Microsoft Windows Server 的 EC2 实例。 Windows Server 许可证由 AWS 提供。

## 架构

使用默认参数为新的虚拟私有云 (VPC) 部署此快速入门会在 AWS 云中构建以下 Microsoft PKI 环境。

![Architecture1](https://aws-quickstart.github.io/quickstart-microsoft-pki/images/image1.png)

图 1. AWS 上 Microsoft PKI 的快速入门架构

如 [图 1](https://aws-quickstart.github.io/quickstart-microsoft-pki/#architecture1), 快速入门创建了以下内容:

-   跨越两个可用区的架构
    
-   根据 AWS 最佳实践配置了公有子网和私有子网的 VPC，为您在 AWS 上提供您自己的虚拟网络
    
-   在公共子网中:
    
    -   托管网络地址转换 (NAT) 网关，允许对私有子网中的资源进行出站互联网访问
        
    -   Auto Scaling 组中的远程桌面网关（RD 网关）实例，以允许入站远程桌面协议 (RDP) 访问公共和私有子网中的 EC2 实例
        
    
-   在私有子网中:
    
    -   在可用区 1 中，运行 Windows 的 EC2 实例充当离线根 CA
        
    -   在可用区 2 中，运行 Windows 的 EC2 实例充当二级CA
        
    
-   AWS Directory Service，有助于部署 Active Directory 证书服务 (AD CS) 环境
    
-   用于存储凭证的 AWS Secrets Manager
    
-   AWS Systems Manager（以前称为 Amazon Simple Systems Manager，或 SSM），用于自动化 CA 部署过程并存储生成的证书。
    
-   AWS Identity and Access Management (IAM) 使 EC2 实例和 Systems Manager 自动化文档能够执行它们的任务。
    

* 将快速入门部署到现有 VPC 的模板会跳过标有星号的组件，并提示您进行现有 VPC 配置

## The AD CS 部署流程

本快速入门部署了一个 AD CS 环境，其中包括一个离线根 CA 和一个在线二级 CA。 此快速入门创建的 AWS Systems Manager 自动化文档可自动执行部署中的步骤。 该过程完成后，根 CA 已生成域根证书并关闭离线，二级 CA 可用于为域签署证书请求。

## 部署计划

### 专业知识

此部署需要对 AWS 服务有一定的了解。 如果您是 AWS 的新手，请参阅 [Getting Started Resource Center](https://aws.amazon.com/getting-started/) 和 [AWS Training and Certification](https://aws.amazon.com/training/)。这些站点提供了学习如何在 AWS 云上设计、部署和操作您的基础设施和应用程序的材料。

本快速入门假定您熟悉 Microsoft CA 角色。

### AWS 账号

如果您还没有 AWS 账户，请按照屏幕上的说明在 [https://aws.amazon.com](https://aws.amazon.com/) 创建一个。 注册过程的一部分涉及接听电话和使用电话键盘输入 PIN。

您的 AWS 账户会自动注册所有 AWS 服务。 您只需为您使用的服务付费。

### 技术要求

在启动快速入门之前，请查看以下信息并确保您的帐户配置正确。 否则，部署可能会失败。

#### 资源配额

如有必要，请为以下资源申请[服务配额增加](https://console.aws.amazon.com/servicequotas/home?region=us-east-2#!/)。 您可能会请求增加配额以避免超过跨多个部署共享的任何资源的默认限制。 [服务配额控制台](https://console.aws.amazon.com/servicequotas/home?region=us-east-2#!/) 显示您对某些服务某些方面的使用情况和配额。 有关更多信息，请参阅[什么是服务配额？](https://docs.aws.amazon.com/servicequotas/latest/userguide/intro.html) 和[AWS 服务配额](https://docs.aws.amazon.com/general/latest/gr/aws_service_limits.html)


| 资源           | 部署使用资源数量 |
| -------------- | ---------------- |
| VPC            | 1                |
| Elastic IP地址 | 1                |
|安全组|           1       |
|          IAM角色      |         1         |
|       EC2实例         |         3         |
|      Secrets Manager          |        2          |
|     AWS托管Directories           |     1             |



#### 支持区域

-   us-east-1 (N. Virginia)
    
-   us-east-2 (Ohio)
    
-   us-west-1 (N. California)
    
-   us-west-2 (Oregon)
    
-   ca-central-1 (Canada Central)
    
-   eu-central-1 (Frankfurt)
    
-   eu-west-1 (Ireland)
    
-   eu-west-2 (London)
    
-   eu-west-3 (Paris)
    
-   ap-southeast-1 (Singapore)
    
-   ap-southeast-2 (Sydney)
    
-   ap-south-1 (Mumbai)
    
-   ap-northeast-1 (Tokyo)
    
-   ap-northeast-2 (Seoul)
    
-   sa-east-1 (South America)
    
-   eu-north-1 (Stockholm)
    
-   ap-east-1 (Hong Kong)
    
-   me-south-1 (Bahrain)
    
-   af-south-1 (Cape Town)
    
-   eu-south-1 (Milan)
    

某些区域可在选择加入的基础上使用。 有关更多信息，请参阅[管理 AWS 区域](https://docs.aws.amazon.com/general/latest/gr/rande-manage.html)。

#### EC2 密钥对

确保您计划所在区域的 AWS 账户中至少存在一个 [Amazon EC2 密钥对](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html) 部署快速入门。 记下密钥对名称。 您在部署期间需要它。 要创建密钥对，请参阅 [Amazon EC2 密钥对和 Linux 实例](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)。

出于测试或概念验证目的，我们建议创建一个新的密钥对，而不是使用已被生产实例使用的密钥对。

#### IAM 权限

在启动快速入门之前，您必须使用模板部署的资源的 IAM 权限登录 AWS 管理控制台。 IAM 中的_AdministratorAccess_ 托管策略提供了足够的权限，但您的组织可能会选择使用具有更多限制的自定义策略。 有关更多信息，请参阅[工作职能的 AWS 托管策略](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_job-functions.html)。

#### 部署选项

此快速入门提供了两个部署选项：

- **将 Microsoft PKI 部署到新的 VPC**。 此选项构建一个新的 AWS 环境，包括 VPC、子网、NAT 网关、安全组、远程桌面网关和其他基础设施组件。 然后它将 Microsoft PKI 部署到这个新的 VPC 中。
    
- **将 Microsoft PKI 部署到现有 VPC 中**。 此选项在您现有的 AWS 基础设施中配置 Microsoft PKI。
    

快速入门为这些选项提供了单独的模板。 它还允许您配置无类域间路由 (CIDR) 块、实例类型和 Microsoft PKI 设置，如本指南后面所述。

## 部署步骤

### 登录您的 AWS 账户

1. 使用具有必要权限的 IAM 用户角色在 [https://aws.amazon.com](https://aws.amazon.com/) 登录您的 AWS 账户。 有关详细信息，请参阅本指南前面的[规划部署](https://aws-quickstart.github.io/quickstart-microsoft-pki/#_planning_the_deployment)。
    
2. 确保您的 AWS 账户配置正确，如[技术要求](https://aws-quickstart.github.io/quickstart-microsoft-pki/#_technical_requirements) 部分所述。

### 启动快速启动

如果您要将 Microsoft PKI 部署到现有 VPC 中，请确保您的 VPC 在工作负载实例的不同可用区中有两个私有子网，并且子网不共享。 此快速入门不支持[共享子网](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-sharing.html)。 这些子网的路由表中需要 [NAT 网关](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html)，以允许实例下载包和软件而不暴露 他们上网。

您负责在运行此快速入门参考部署时使用的 AWS 服务的费用。 使用此 Quick Start 无需支付额外费用。 有关完整详细信息，请参阅此 Quick Start 使用的每项 AWS 服务的定价页面。 价格随时可能更改。

每次部署大约需要 30 分钟才能完成。

1. 登录您的 AWS 账户，然后选择以下选项之一启动 AWS CloudFormation 模板。 如需有关选择选项的帮助，请参阅本指南前面的[部署选项](https://aws-quickstart.github.io/quickstart-microsoft-pki/#_deployment_options)。
    
| [Deploy Microsoft PKI into a new VPC on AWS](https://fwd.aws/V4P9Q)       | [View template](https://fwd.aws/aa5WK) | 登陆远程桌面需要 RD Host>0 ,RDP与SubCA一个AZ，保证RDP可以公网访问   |
| ------------------------------------------------------------------------- | -------------------------------------- | --- |
| [Deploy Microsoft PKI into an existing VPC on AWS](https://fwd.aws/vgDqv) | [View template](https://fwd.aws/Axdrp) |     |

2. 勾选导航栏右上角显示的AWS Region，必要时进行更改。 这是将构建 Microsoft PKI 网络基础结构的地方。 该模板默认在 us-east-1 区域启动。
    

3. 在**创建堆栈**页面上，保留模板 URL 的默认设置，然后选择**下一步**。
    
4. 在**指定堆栈详细信息**页面上，根据需要更改堆栈名称。 查看模板的参数。 为需要输入的参数提供值。 对于所有其他参数，请查看默认设置并根据需要自定义它们。 有关每个参数的详细信息，请参阅本指南的[参数参考](https://aws-quickstart.github.io/quickstart-microsoft-pki/#_parameter_reference)部分。 当您完成查看和自定义参数后，选择 **下一步**。
    
5. 在**配置堆栈选项**页面上，您可以[指定标签](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-resource-tags.html)(关键 -值对）用于堆栈中的资源并[设置高级选项](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-add-tags.html)。 完成后，选择**下一步**。
    
6. 在**审核**页面上，审核并确认模板设置。 在**功能**下，选中两个复选框以确认模板创建 IAM 资源并且可能需要自动扩展宏的能力。
    
7. 选择**创建堆栈**以部署堆栈。
    
8. 监控栈的状态。 当状态为**CREATE_COMPLETE** 时，Microsoft PKI 部署就绪。
    
9. 使用堆栈的 **Outputs** 选项卡中显示的值，如 [图 2](https://aws-quickstart.github.io/quickstart-microsoft-pki/#cfn_outputs) 所示，以查看 创建的资源。
    

[![cfn_outputs](https://aws-quickstart.github.io/quickstart-microsoft-pki/images/cfn_outputs.png)](https://aws-quickstart.github.io/quickstart-microsoft-pki/images/cfn_outputs.png)

图 2. 成功部署后的 Microsoft PKI 输出

## 部署后步骤

### 运行 Windows 更新

为了确保部署的服务器的操作系统和安装的应用程序具有最新的 Microsoft 更新，请在每台服务器上运行 Windows Update。

1. 创建从远程桌面网关服务器到每个已部署服务器的 RDP 会话。
    
2. 打开**设置**应用程序。
    
3. 打开**更新和安全**。
    
4. 点击**检查更新**。
    
5. 如有必要，安装任何更新并重新启动。
    

### 测试部署

1. 通过RDP连接到RD Gateway，然后通过RDP连接到二级CA的IP地址，输入自定义的域账号名和密码。
    
2. 导航至 http://<subordinate CA FQDN>/certsrv
注意：在该QuickStart文档中，并不会配置Web Enrollment，如果需要网页版签发证书，请在Server Manager的“manage”处安装Certification Enrollment Web Service和其他相关的设置。在IIS的Server Certificates处您也可以进行签发证书、导出证书等操作。另外您也可以利用mmc的Add Snap-ins定制化证书管理界面。
    
3. 颁发测试证书

## 在 AWS 上使用 Microsoft PKI 的最佳实践

此快速入门将相关文件夹和文件（与证书服务相关的文件夹和文件）迁移到 Amazon Elastic Block Store (Amazon EBS) 卷 [D:\]。 备份 Microsoft PKI 部署，包括私钥和当前证书数据库。

## 安全

本快速入门部署根 CA 并生成根证书后，根 CA 将被关闭。 根 CA 旨在保持离线状态，直到需要更新域根证书。

域管理员凭证存储在 AWS Secrets Manager 中并由从属 CA 使用。 从属 CA 使用这些凭据加入域、安装证书服务并添加所需的 DNS 记录。

## 其他有用信息

-   [Configure the Server Certificate Template](https://docs.microsoft.com/en-us/windows-server/networking/core-network-guide/cncg/server-certs/configure-the-server-certificate-template)
    
-   [Configure certificate auto-enrollment](https://docs.microsoft.com/en-us/windows-server/networking/core-network-guide/cncg/server-certs/configure-server-certificate-autoenrollment)
    

## FAQ

**问。**我在启动快速入门时遇到了**CREATE_FAILED**错误。

**A.** 如果 AWS CloudFormation 无法创建堆栈，请重新启动模板并将**失败时回滚**设置为**否**。 （此设置位于 AWS CloudFormation 控制台的**高级**下，**配置堆栈选项**页面。）使用此设置，堆栈的状态将保留并且实例保持运行状态，因此您可以解决问题。 （对于 Windows，请查看 `%ProgramFiles%\Amazon\EC2ConfigService` 和 `C:\cfn\log` 中的日志文件。）

当您将**失败时回滚**设置为**禁用**时，您将继续为此堆栈产生 AWS 费用。 完成故障排除后删除堆栈。

有关更多信息，请参阅 [AWS CloudFormation 故障排除](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/troubleshooting.html)。

**问。** 我在部署 AWS CloudFormation 模板时遇到了大小限制错误。

**A.** 从本指南中的链接或从另一个 S3 存储桶启动快速入门模板。 如果您从计算机上的本地副本或 S3 存储桶以外的位置部署模板，您可能会遇到模板大小限制。 有关更多信息，请参阅 [AWS CloudFormation 配额](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cloudformation-limits.html)。

##  故障排除

此部署中使用的 Systems Manager 自动化文档将其 PowerShell 脚本的输出存储在 CloudWatch Logs 中。

另请参阅[系统管理器运行命令故障排除](https://docs.aws.amazon.com/systems-manager/latest/userguide/troubleshooting-remote-commands.html)。

## 向我们发送反馈

要发布反馈、提交功能想法或报告错误，请使用此快速入门的 [GitHub 存储库](https://github.com/aws-quickstart/quickstart-microsoft-pki) 的**问题**部分。 要提交代码，请参阅[快速入门贡献者指南](https://aws-quickstart.github.io/)。

## 快速入门参考部署

请参阅 [AWS 快速入门主页](https://aws.amazon.com/quickstart/)。

## GitHub 仓库

访问我们的 [GitHub 存储库](https://github.com/aws-quickstart/quickstart-microsoft-pki) 下载此快速入门的模板和脚本，发表您的评论，并与他人分享您的自定义设置。

---

## 注意

本文档仅供参考。 它代表截至本文档发布之日 AWS 当前的产品供应和实践，如有更改，恕不另行通知。 客户有责任自行独立评估本文档中的信息以及对 AWS 产品或服务的任何使用，其中每一项产品或服务均“按原样”提供，不提供任何明示或暗示的保证。 本文档不构成 AWS、其附属公司、供应商或许可方的任何保证、陈述、合同承诺、条件或保证。 AWS 对其客户的责任和义务由 AWS 协议控制，本文档不属于也不修改 AWS 与其客户之间的任何协议。

本文中包含的软件根据 Apache 许可证 2.0 版（“许可证”）获得许可。 除非遵守许可证，否则您不得使用此文件。 许可副本位于 [http://aws.amazon.com/apache2.0/](http://aws.amazon.com/apache2.0/) 或随附的“许可”文件中。 此代码按“原样”发布，不附带任何明示或暗示的保证或条件。 有关特定语言的管理权限和限制，请参阅许可证。


#### 参考参数
**表1. 网络配置**
| **Parameter label (name)**                         | **Default value**    | **Description**                                      |
| -------------------------------------------------- | -------------------- | ---------------------------------------------------- |
| VPC CIDR                                           | 10.0.0.0/16          | CIDR Block for the VPC                               |
| VPC ID (VPCID)                                     | **_Requires input_** | ID of the VPC (e.g., vpc-0343606e)                   |
| CA(s) Subnet ID (CaServerSubnet)                   | **_Requires input_** | ID of the CA(s) subnet (e.g., subnet-a0246dcd)       |
| Domain Members Security Group ID (DomainMembersSG) | **_Requires input_** | Security Group ID for Domain Members Security Group. |

**表2. Amazon EC2 配置**

| **Parameter label (name)**                                                          | **Default value**                                                     | **Description**                                                                                    |
| ----------------------------------------------------------------------------------- | --------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| Offline Root CA Instance Type (Only Used For Two Tier PKI) (OrCaServerInstanceType) | t3.medium                                                             | Amazon EC2 instance type for the Offline Root CA instance (Only Used For Two Tier PKI)             |
| Offline Root CA Data Drive Size (Only Used For Two Tier PKI) (OrCaDataDriveSizeGiB) | 2                                                                     | Size of the Data Drive in GiB for the Offline Root CA (Only Used For Two Tier PKI)                 |
| Enterprise Root or Subordinate CA Instance Type (EntCaServerInstanceType)           | t3.medium                                                             | Amazon EC2 instance type for the Enterprise Root or Subordinate CA instance                        |
| Enterprise Root or Subordinate CA Data Drive Size (EntCaDataDriveSizeGiB)           | 2                                                                     | Size of the Data Drive in GiB for the Enterprise Root or Subordinate CA                            |
| Key Pair Name (KeyPairName)                                                         | **_Requires input_**                                                  | Name of the key pair that you will use to securely connect to your EC2 instance after it launches. |
| SSM Parameter Value for latest AMI ID (AMI)                                         | /aws/service/ami-windows-latest/Windows_Server-2019-English-Full-Base | CA(s) SSM Parameter Value to grab the latest AMI ID                                                |

Table 3. Microsoft Active Directory Domain Services Configuration
| **Parameter label (name)**                                         | **Default value**    | **Description**                                                                                                                                                             |
| ------------------------------------------------------------------ | -------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Active Directory Domain Services Type (DirectoryType)              | **_Requires input_** | Type of Active Directory the CA(s) will be integrated with, AWS Managed Microsoft AD or Self Managed AD                                                                     |
| Domain FQDN DNS Name (DomainDNSName)                               | **_Requires input_** | Fully qualified domain name (FQDN) of the forest root domain e.g. example.com                                                                                               |
| Domain NetBIOS Name (DomainNetBIOSName)                            | **_Requires input_** | NetBIOS name of the domain (up to 15 characters) for users of earlier versions of Windows e.g. EXAMPLE                                                                      |
| IP used for DNS (Must be accessible) (DomainController1IP)         | **_Requires input_** | IP of DNS server that can resolve domain (Must be accessible)                                                                                                               |
| IP used for DNS (Must be accessible) (DomainController2IP)         | **_Requires input_** | IP of DNS server that can resolve domain (Must be accessible)                                                                                                               |
| Secret ARN Containing CA Install Credentials (AdministratorSecret) | **_Requires input_** | ARN for the CA Install Credentials Secret used to to install the CA (Must be a member of Enterpise Admins or AWS Delegated Enterprise Certificate Authority Administrators) |

**表4. Microsoft Active Directory Certificate Services 配置**

| **Parameter label (name)**                | **Default value** | **Description**                                |
| ----------------------------- | ----------------- | ---------------------------------------------- |
| CA Deployment Type (CADeploymentType)                                                                       | Two-Tier          | Deploy Two Tier (Offline Root with Subordinate Enterprise CA) or One Tier (Enterprise Root CA) PKI Infrastructure                                                      |
| Offline Root CA NetBIOS Name (Only Used For Two Tier PKI) (OrCaServerNetBIOSName)                           | ORCA1             | NetBIOS name of the Offline Root CA server (Only Used For Two Tier PKI) (up to 15 characters)                                                                          |
| Enterprise Root or Subordinate CA NetBIOS Name (EntCaServerNetBIOSName)                                     | ENTCA1            | NetBIOS name of the Enterprise Root or Subordinate CA server (up to 15 characters)                                                                                     |
| CA Key Length (CaKeyLength)                                                                                 | 2048              | CA(s) Cryptographic Provider Key Length                                                                                                                                |
| CA Hash Algorithm (CaHashAlgorithm)                                                                         | SHA256            | CA(s) Hash Algorithm for Signing Certificates                                                                                                                          |
| Offline Root CA Certificate Validity Period in Years (Only Used For Two Tier PKI) (OrCaValidityPeriodUnits) | 10                | Validity Period in Years (Only Used For Two Tier PKI)                                                                                                                  |
| Enterprise Root or Subordinate CA Certificate Validity Period in Years (CaValidityPeriodUnits)              | 5                 | Validity Period in Years                                                                                                                                               |
| Use S3 for CA CRL Location (UseS3ForCRL)                                                                    | No                | Store CA CRL(s) in an S3 bucket                                                                                                                                        |
| CA CRL S3 Bucket Name (S3CRLBucketName)                                                                     | examplebucket     | S3 bucket name for CA CRL(s) storage. Bucket name can include numbers, lowercase letters, uppercase letters, and hyphens (-). It cannot start or end with a hyphen (-) |


**表5. AWS Quick Start 配置**
| **Parameter label (name)**                      | **Default value**         | **Description**                                                                                                                                                                          |
| ----------------------------------------------- | ------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Quick Start S3 bucket name (QSS3BucketName)     | aws-quickstart            | S3 bucket name for the Quick Start assets. Quick Start bucket name can include numbers, lowercase letters, uppercase letters, and hyphens (-). It cannot start or end with a hyphen (-). |
| Quick Start S3 bucket Region (QSS3BucketRegion) | us-east-1                 | The AWS Region where the Quick Start S3 bucket (QSS3BucketName) is hosted. When using your own bucket, you must specify this value                                                       |
| Quick Start S3 key prefix (QSS3KeyPrefix)       | quickstart-microsoft-pki/ | S3 key prefix for the Quick Start assets. Quick Start key prefix can include numbers, lowercase letters, uppercase letters, hyphens (-), and forward slash (/)                           |







