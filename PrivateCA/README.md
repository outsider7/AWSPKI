# 介绍

本文档将介绍如何使用 AWS 私有证书颁发机构 (AWS Private CA) 安全地设置完整的 CA 层次结构，并为各种用例创建证书。 这些用例包括终止 TLS、代码签名、文档签名、IoT 设备身份验证和电子邮件真实性验证的内部应用程序。 我们将涵盖 CA 管理员、应用程序开发人员和安全管理员等工作职能，并向您展示这些角色如何遵循最小权限原则并执行与证书管理相关的各种功能。 此外，您还将学习如何使用 AWS Security Hub 监控您的 PKI 基础设施。

完整英文版文档链接：https://github.com/aws-samples/data-protection  
完整英文版workshop链接：https://catalog.workshops.aws/certificatemanager


## 1. 使用个人账号

-   登录您想要的 AWS 账户
    
-   确认您位于所需区域。
    
    -   请使用 AWS Cloud9、Amazon Elastic Kubernetes Service 和 AWS Certificate Manager (ACM) 服务 [可用](https://aws.amazon.com/about-aws/global-infrastructure/regional-product) 的 AWS 区域 -服务
-   通过右键单击此链接下载 CloudFormation 模板： [Security Admin CloudFormation Stack](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/cloudformation_templates/security_admin.yaml) 并将链接另存为 _security-admin.yaml_
    
-   在您登录的 AWS 账户中上传并启动 CloudFormation 堆栈。 如果您对此不熟悉，请遵循[此处的说明](https://catalog.workshops.aws/certificatemanager/en-US/gettingstarted/self-paced/security_admin_cf_instructions)
    
    -   **注意：** CloudFormation 堆栈可能需要大约 20 分钟来部署环境

为避免您的帐户出现任何权限问题，请确保您具有管理员访问权限。 此外，需要为 CRL S3 存储桶禁用 S3 块公共访问，以便 TLS 客户端可以访问这些存储桶。

## 2.CloudFormation步骤

步骤 1:

![CF Instructions](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/cf_instructions/securityadmin_cf_instructions.png)

步骤 2:

![CF Instructions](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/cf_instructions/search_step.png)

步骤 3:

![CF Instructions](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/cf_instructions/upload_step.png)

步骤 4:

![CF Instructions](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/cf_instructions/namestack_step.png)

步骤 5:

![CF Instructions](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/cf_instructions/scroll_step.png)

步骤 6:

![CF Instructions](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/cf_instructions/createstack_step.png)


## 3. CA层次结构设计

### 3.1 CA 层次结构设置

### 承担名为**CaAdminRole** 的 IAM 角色

-   通过在您当前登录的 AWS 帐户中的 AWS 控制台上使用切换角色来承担名为**CaAdminRole** 的角色
    
-   此角色具有证书颁发机构管理员进行 CA 管理所需的权限。 作为 CA 管理员，您将负责创建根和从属证书颁发机构
    
-   如果您不熟悉角色切换，请根据需要遵循本教程：[在控制台中担任角色](https://github.com/aws-samples/data-protection/blob/master/usecase-9/img/SwitchRole.pdf "https://github.com/aws-samples/data-protection/blob/master/usecase-9/img/SwitchRole.pdf")
    


### 3.2 创建私有 CA 层次结构

-   您可以手动或通过 CloudFormation 模板创建私有根证书和从属证书颁发机构。
-   请选择手动 _私有 CA 创建_ 或 _快速部署_ 选项，但 _不要两者都做_。

如果您以前从未使用过 AWS Private CA，我们强烈建议您使用手动说明。

### 3.2(a) 手动创建私有 CA 层次结构

我们将从创建一个新的 CA 层次结构开始：一个充当信任根的根 CA，以及一个可用于在整个研讨会的其余部分颁发最终实体证书的从属 CA。 您可以找到[在此处创建私有 CA 的视频说明](https://www.youtube.com/watch?v=pKymN_ICpv8 "https://www.youtube.com/watch?v=pKymN_ICpv8")。

### 创建根 CA

1.  在 AWS 控制台导航到 AWS Certificate Manager 服务
    
2.  展开控制台左侧的边栏，然后单击**AWS Private CA**
    
3.  选择**创建私有 CA**
    

---

![创建私有 CA](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/acm_welcome.png)

---

4.  我们将从创建一个根 CA 开始，所以在 CA 类型选项下选择 **Root**

---

![根 CA 类型](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/ca_type.png)

---

5.  对于**主题专有名称选项**，输入您选择的值。 请注意，**您必须在此页面上至少输入一个姓名**，但所有字段都是可选的，_并非所有字段都需要填写。_
    -   我们建议至少输入一个通用名称 (CN)，以便轻松地将 CA 识别为根或从属。

---

![根 CA 参数](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/subject_distinguished_names.png)

---

6.  接下来，向下滚动以配置**密钥算法选项**。 这部分允许我们为我们的 CA 选择一个密钥算法。 在本次研讨会中，我们将使用默认值：_RSA 2048_。
    -   大多数客户将使用 RSA 2048，但有些客户可能会选择使用更长的密钥长度（例如 RSA 4096）以提高安全性
    -   某些设备的处理能力可能有限，需要使用椭圆曲线加密 (ECC) 证书：ECC 需要比 RSA 更少的计算能力来进行加密操作。 ECDSA 通常用于物联网 (IoT) 设备。

---

![根 CA 密钥算法](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/key_algo_options.png)

---

7.  现在我们需要为我们的新根 CA 配置吊销设置。 选中_激活 CRL 分发_复选框，并取消选中_创建 S3 存储桶_选项。 单击_浏览 S3_ 按钮并从下拉菜单中选择以以下内容开头的 S3 存储桶：_acm-private-ca-crl_
    -   此存储桶是在您的帐户中预先创建的，作为研讨会设置的一部分

---

![根 CA 撤销](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/cert_revo_options.png)

---

8.  添加标签以标识 CA _（可选）_
    
9.  在**CA 权限选项**下，我们可以授权 ACM 自动续订此 CA 颁发的证书。 由于我们不会直接从根 CA 颁发证书，我们可以取消选中此框
    

---

![根 CA ACM 权限](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/ca_permissions_options.png)

---

10.  在**定价**部分，勾选方框表示您同意。 现在点击 **创建 CA**
    
11.  最后一步，我们需要在新创建的根 CA 中安装 CA 证书。 在您新创建的 CA 上，点击 _Actions_ 下拉菜单并选择 _Install CA Certificate_。
    

---

![动作](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/new_ca_actions.png)

---

12.  这里我们可以设置CA证书的有效期，以及签名算法。 保留默认设置，然后单击**确认并安装**。

---

![根 CA 创建成功](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/install_root_ca_cert.png)

---

12.  在幕后，AWS Private CA 使用我们在前面步骤中输入的信息创建证书签名请求 (CSR)。

13.我们已经创建了一个处于活动状态并准备颁发证书的根 CA。 但是，AWS 强烈建议不要直接使用根 CA 颁发证书。 为了遵循最佳实践，我们现在将创建一个从属 CA，它将由我们刚刚创建的根 CA 签名。

![根 CA 证书审查和创建](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/root_complete.png)

### 创建从属CA

我们已经创建了我们的根 CA，但现在我们需要创建一个可用于颁发证书的从属 CA（有时称为颁发 CA）。

由于根 CA 是我们信任链的顶端，我们只想用它来签署从属 CA，以表明该 CA 受到我们组织的信任。 如果从属 CA 遭到破坏，则它颁发的任何终端实体证书也将受到破坏。 但是，如果我们的根 CA 的私钥以某种方式泄露，这将意味着我们整个组织的所有证书现在都处于危险之中。

我们使用证书层次结构来限制潜在CA 折中的 _影响半径_ 。 您可以在本视频中找到[有关 CA 层次结构的更多信息](https://www.youtube.com/watch?v=8FB12c1lDyo "https://www.youtube.com/watch?v=8FB12c1lDyo")。

---

按照以下步骤创建一个新的从属 CA，我们将使用我们在上一节中创建的根 CA 对其进行签名：

---

1.  在私有 CA 控制台中，单击页面右上角附近的**创建 CA** 按钮

---

![开始创建子 CA](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/sub_ca_begin.png)

---

2.  这一次，选择**下属**并继续配置**主题专有名称选项**

---

![父 CA 类型](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/subordinate_ca_type.png)

---

3.  输入您想要的**主题专有名称选项**。 请注意，**您必须在此页面上至少输入一个姓名**，但所有字段都是可选的，_并非所有字段都需要填写。_
    -   我们建议至少输入一个通用名称 (CN)，以便轻松地将 CA 识别为根或从属。

---

![从属 CA 类型](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/subordinate_ca_parameters.png)

---

4.在**密钥算法选项**下使用默认值 RSA 2048

5.接下来我们需要配置我们的撤销方法，遵循与根CA创建部分相同的过程

---

![从属 CA 吊销](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/subordinate_revocation_options.png)

---

6.  添加标签以标识 CA _（可选）_

虽然标签不是强制性的，但它们可用于跟踪组织各部分之间的成本，或者它们可用于通过 IAM 策略促进基于属性的访问控制 (ABAC)。

7.  与根 CA 创建部分不同，我们将为从属 CA 保留 _授权 ACM 访问以续订此帐户请求的证书_。 通过将证书续订过程卸载到 ACM 与执行手动续订相比，此功能可以大大减少运营开销

---

![从属 CA ACM 权限](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/subord_perms_options.png)

---

8.  勾选**定价**部分下的框以表示我们同意。 现在点击 **创建 CA**
    
9.  我们将再次需要安装 CA 证书。 这一次，我们将使用在上一节中创建的根 CA 证书来签署这个新的从属 CA，而不是自签名根 CA 证书。 这会将其添加到我们的 CA 层次结构和我们的 _信任链_ 。
    
10.  在接下来的页面上，选择 _Actions_ 下拉菜单并 _install CA Certificate_
    

---

![从属行动](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/subordinate_actions_install.png)

---

11.  在此页面上，我们需要选择 _AWS Private CA_ 以便我们可以选择之前创建的根 CA

-   如果您想使用本地托管的根或中间 CA 证书签署这个新的从属 CA，您可以选择 _外部私有 CA_

---

![从属证书父 CA 类型](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/install_subordinate_cert.png)

---

11.  现在我们必须从下拉菜单中选择我们的根 CA 作为 _父私有 CA_。 将有效性、算法和路径长度选项保留为默认设置。
    -   从下拉列表中选择一个 CA 后，将填充通用名称：确保它与您在上一节中创建的根 CA 的通用名称相匹配
    -   **有效期** 指定 CA 证书到期前的时间长度
    -   **路径长度**表示可以由该 CA 签名的证书颁发机构分支/层级的数量。 例如，如果我们要创建一个具有根、中间和颁发 CA 的 3 层 CA 层次结构，我们将使此路径长度为 1：这将允许此从属 CA 充当中间 CA 并签署作为一个“层”的 CA " 在 CA 层次结构中低于它。 您可以在此处找到有关 [CA 层次结构和路径长度限制](https://docs.aws.amazon.com/acm-pca/latest/userguide/ca-hierarchy.html "https://docs.aws.amazon.com/acm-pca/latest/userguide/ca-hierarchy.html") 的更多信息。

---

![从属证书父 CA 选项](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/parent_ca_details.png)

## ![从属证书有效性](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/subordinate_ca_validity.png)

路径长度为 1 不**不**意味着只能使用一个 CA 证书签署一个 CA。 您可以使用路径长度为 1 的 CA 证书签署 5 个或更多的颁发 CA，但您不能**不**签署另一个低于颁发 CA“层级”的 CA。 您可以使用路径长度限制来防止其他证书颁发机构签署新的 CA。

12.最后，我们将检查我们的选择并点击**确认并安装**

---

13.太棒了！ 我们创建了一个新的从属 CA 并使用我们的根 CA 对其进行了签名，将其添加到我们的 _信任链_ 中。 您应该（至少）在 AWS Private CA 控制台中看到两个证书颁发机构，如下所示。

---

![从属CA创建完成](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/sub_complete.png)

---

###  3.2(b)  快速部署

### 自动创建私有 CA 层次结构

重点

如果您跳过了 _Private CA Creation_ 部分，请仅完成此部分。 **不要同时完成这两项**。

-   通过右键单击此按钮 [PCA 层次结构快速部署 CloudFormation 堆栈](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/cloudformation_templates/pca_hierarchy.yaml) 下载并启动 CloudFormation 模板并将链接保存为 _pca-hierarchy.yaml_. 参见模版代码 [PCA CloudFormation 堆栈](https://github.com/outsider7/AWSPKI/blob/main/PrivateCA/code/pca_hierarchy.yaml) 
    
-   在您登录的 AWS 账户中上传并启动 CloudFormation 堆栈。 如果您对此不熟悉，请按照此处的说明进行操作：[部署 CloudFormation 堆栈说明](https://github.com/aws-samples/data-protection/blob/master/usecase-9/img/CAAdminSteps-1.pdf)





### 更多案例
[以PKI管理员身份管理私有CA ](https://github.com/outsider7/AWSPKI/blob/main/PrivateCA/PrivateCA/PKI_Managemennt.md)

[在EKS上传输中的数据保护 ](https://github.com/outsider7/AWSPKI/blob/main/PrivateCA/PrivateCA/PKI_Protection.md)


[IoT设备使用案例 ](https://github.com/outsider7/AWSPKI/blob/main/PrivateCA/IOT_Device_Usecase.md)

[HTTPS应用使用案例 ](https://github.com/outsider7/AWSPKI/blob/main/PrivateCA/HTTPS_Application_Usecase.md)