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

我们使用证书层次结构来限制潜在 CA 妥协的_影响半径_。 您可以在本视频中找到[有关 CA 层次结构的更多信息](https://www.youtube.com/watch?v=8FB12c1lDyo "https://www.youtube.com/watch?v=8FB12c1lDyo")。

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
    
9.  我们将再次需要安装 CA 证书。 这一次，我们将使用在上一节中创建的根 CA 证书来签署这个新的从属 CA，而不是自签名根 CA 证书。 这会将其添加到我们的 CA 层次结构和我们的_信任链_。
    
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

13.太棒了！ 我们创建了一个新的从属 CA 并使用我们的根 CA 对其进行了签名，将其添加到我们的_信任链_中。 您应该（至少）在 AWS Private CA 控制台中看到两个证书颁发机构，如下所示。

---

![从属CA创建完成](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/sub_complete.png)

---

###  3.2(b)  快速部署

### 自动创建私有 CA 层次结构

重要的

如果您跳过了 _Private CA Creation_ 部分，请仅完成此部分。 **不要同时完成这两项**。

-   通过右键单击此按钮 [PCA 层次结构快速部署 CloudFormation 堆栈](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/cloudformation_templates/pca_hierarchy.yaml) 下载并启动 CloudFormation 模板并将链接保存为 _pca-hierarchy.yaml_. 参见模版代码 [PCA CloudFormation 堆栈](https://github.com/outsider7/AWSPKI/blob/main/PrivateCA/code/pca_hierarchy.yaml) 
    
-   在您登录的 AWS 账户中上传并启动 CloudFormation 堆栈。 如果您对此不熟悉，请按照此处的说明进行操作：[部署 CloudFormation 堆栈说明](https://github.com/aws-samples/data-protection/blob/master/usecase-9/img/CAAdminSteps-1.pdf)


## 4. [optional]以PKI管理员身份管理私有CA

## 模块介绍：

在这一部分，我们将重点关注如何使用 AWS Private CA 在 AWS 上安全地管理公钥基础设施 (PKI)。 该模块有两个子部分，重点关注：_CA 监控_ 和 _证书模板。_

---

## 模块组件：

-   **监控私有 CA：** 在本节中，您将学习如何利用 SecurityHub 监控托管在 AWS 中的私有 CA
-   **使用证书模板：** 研讨会的这一部分将演示如何将证书模板用于最终实体证书，包括创建自定义模板

### 4.1 监控AWS私有CA

#### 安全监控：

![图片](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/architecture.png)

在本节中，我们将了解如何在构建证书管理基础结构时监控特权操作。 我们将研究两个场景。 _CA 证书的创建_ 和 _最终实体证书的大规模撤销_ 。

#### 场景 1：使用 Security Hub 进行监控

##### 承担名为**CaAdminRole** 的 IAM 角色

-   通过在您当前登录的 AWS 帐户中的 AWS 控制台上使用切换角色，代入名为**CaAdminRole** 的角色。
    
-   此角色使 CA 管理员能够审查 Security Hub 中的发现。
    

如何切换角色？

##### 场景一说明

-   创建 CA 证书是一项特权操作，只能由 CA Hierarchy Management 团队中的授权人员执行。 出于这个原因，我们希望在我们的层次结构中监控任何 CA 证书的创建。

1.  导航到 Security Hub 控制台并选择页面左侧的“调查结果”。

---

![安全中心调查结果](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/securityhubfindings.png)

---

2.  按标题过滤 IS 证书颁发机构创建

---

![安全中心过滤](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/securityhubfilter.png)

---

3.  通过选中复选框选择一项调查结果。 单击标题链接。 转到“证书颁发机构创建”下的右上角页面，然后选择查找 ID 链接。

---

![Security Hub 查找 ID](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/securityhubfindingid.png)

---

4.  向下滚动到资源 > 详细信息 > 其他。 请参阅资源 ID。 这是创建的证书颁发机构 ARN，它创建了此 Security Hub 结果。

---

![安全中心 json](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/securityhubfindingjson.png)

---

#### 场景 2：监控批量撤销

##### 承担名为**AppDevRole** 的 IAM 角色

-   通过在您当前登录的 AWS 帐户中的 AWS 控制台上使用切换角色来承担名为**AppDevRole** 的角色。
    
-   此角色使应用程序开发人员能够为其应用程序创建和撤销证书。
    

如何切换角色？

##### 场景2说明

-   此场景显示开发人员在短时间内吊销了许多终端实体证书。 如果发生此类特权操作，我们希望监控并通知安全团队以进行调查。

1.导航到Cloud9并打开IDE

---

![Cloud 9 控制台](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/cloud9open.png)

---

2.  打开 data-protection/usecase-9 下的**usecase-9 文件夹**。 右键单击 usecase-9 文件夹下的**environment-setup.sh**。 选择运行。

-   此脚本大约需要一分钟才能完成
-   确保您运行的是正确的**environment-setup.sh** 脚本。 它位于此处：`/home/ec2-user/environment/data-protection/usecase-9/environment-setup.sh`
-   该脚本会为研讨会的这一部分安装各种工具和包，请参阅脚本中的注释以获取更多信息
-   当您看到**环境设置完成**时，脚本已完成，可以在文件`setup.log` 中找到其他日志信息

---

![设置环境](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/cloud9environmentsetup.png) ![环境成功](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/cloud9environmentoutput.png)

---

3.  打开 data-protection/usecase-9/code 下的 **code** 文件夹。 双击 code 文件夹下的 **create-certs.py** 文件。 选择运行。 等待几秒钟，直到您看到打印了五个证书。 此脚本使用我们在第一个模块中创建的从属 CA 创建五个私有证书。 请随意阅读代码以更好地理解所使用的 API。

---

![创建证书](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/cloud9createcerts.png)

---

4.  导航回您的 AWS 控制台（在下一步的单独选项卡中保持 Cloud9 打开）。 转到证书管理器控制台。 您应该看到通过您刚刚作为 Application Developer 角色运行的脚本生成的所有证书。

---

![检查创建的证书](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/cloud9certs.png)

---

5.  导航回 Cloud9。 在**data-protection/usecase-9/code**下，双击**revoke-certs.py**打开。 然后运行脚本来撤销我们刚刚创建的所有证书。

---

![撤销证书](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/cloud9revoke.png)

---

6.  完成此[测验](https://amazonmr.au1.qualtrics.com/jfe/form/SV_3mHHKwvVlxQ0v1X "https://amazonmr.au1.qualtrics.com/jfe/form/SV_3mHHKwvVlxQ0v1X")

#### 监控证书吊销

-   我们将以 CA 管理员身份导航到 Security Hub，以监控证书的吊销。 请记住，我们现在正在从应用程序开发人员角色（和相关角色）切换到执行监控的 CA 管理员角色。

##### 承担名为**CaAdminRole** 的 IAM 角色

-   通过在您当前登录的 AWS 帐户中的 AWS 控制台上使用切换角色，代入名为**CaAdminRole** 的角色。
    
-   此角色使 CA 管理员能够审查 Security Hub 中的发现。
    

如何切换角色？

##### 吊销监控说明

1.  导航到 Security Hub 控制台并选择页面左侧的“调查结果”。

---

![安全中心调查结果](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/securityhubfindings.png)

---

2.  您应该在列表顶部看到五个 _Certificate Revocation_ 结果。 如果不这样做，您可以搜索 Title IS Certificate Revocation。

---

![安全中心调查结果](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/cloud9certrevocationsecurityhub.png)

---

您现在已经成功地完成了应用程序开发人员创建和撤销从属 CA 的私有证书的操作以及监视此类操作的 CA 管理员的操作。


## 4.2 证书模板

## 概述

这一部分，我们将了解证书扩展，这些扩展可以帮助您将证书用于应用程序，而不是识别 TLS 服务器端点的普遍情况。 这些包括：

-   代码签名
-   签署在线证书状态协议 (OCSP) 响应
-   ADCS证书

---

使一个证书对签署代码有用而另一个对终止 TLS 有用的证书是证书中的各种字段和扩展。 扩展字段，或简称为扩展，定义了证书的用途。 RFC 5280 中定义了一些常用和广泛支持的扩展，包括：

-   基本约束
-   关键用法
-   扩展密钥使用

---

AWS Private CA 在使用四种模板生成各种证书方面提供了完全的灵活性：

-   基本模板
-   CSRPassthrough模板
-   APIPassthrough模板
-   APICSRPassthrough模板

---

这一部分，您将了解如何使用可用的不同模板类型来颁发不同的证书类型。

### 承担名为**AppDevRole** 的 IAM 角色

-   通过在您当前登录的 AWS 帐户中的 AWS 控制台上使用切换角色来承担名为**AppDevRole** 的角色。
    
-   此角色使应用程序开发人员能够为其应用程序创建和撤销证书。
    

如何切换角色？ 如果您不熟悉角色切换，请根据需要遵循本教程：[在控制台中担任角色](https://github.com/aws-samples/data-protection/blob/master/usecase-9/img/SwitchRole.pdf)

### 练习 1 - 默认模板：

-   在本练习中，您将使用 AWS Private CA 提供的预建模板创建代码签名证书。

1.导航到Cloud9并打开IDE

---

![Cloud 9 控制台](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/cloud9open.png)

该部分代码请参见github repo：   [代码部分](https://github.com/aws-samples/data-protection/tree/master/usecase-9)

---

2.  打开 data-protection/usecase-9 下的**usecase-9 文件夹**。 右键单击 usecase-9 文件夹下的**environment-setup.sh**。 选择运行。

-   此脚本大约需要一分钟才能完成
-   确保您运行的是正确的**environment-setup.sh** 脚本。 它位于此处：`/home/ec2-user/environment/data-protection/usecase-9/environment-setup.sh`
-   该脚本会为研讨会的这一部分安装各种工具和包，请参阅脚本中的注释以获取更多信息
-   当您看到**环境设置完成**时，脚本已完成，可以在文件`setup.log` 中找到其他日志信息

---

![设置环境](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/cloud9environmentsetup.png) ![环境成功](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/cloud9environmentoutput.png)

---

3.  打开 **templates.py** data-protection/usecase-9/templates 目录。 运行脚本。 大约 2 分钟后，您应该会看到_成功创建代码签名证书 codesigning_cert.pem_。

---

![模板运行](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/certtemplaterun.png)

---

4.  打开新终端。

---

![模板运行](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/certtemplateopenterm.png)

---

5.  通过 `cd data-protection/usecase-9/templates` 将目录更改为模板文件夹。 然后运行以下命令创建代码签名证书，`openssl x509 -in codesigning_cert.pem -text -noout`。 您应该看到证书内容的输出。

---

![模板运行](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/certtemplatecodesigning.png)


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


### 更多案例

[IoT设备使用案例 ](https://github.com/outsider7/AWSPKI/blob/main/PrivateCA/IOT_Device_Usecase.md)

[HTTPS应用使用案例 ](https://github.com/outsider7/AWSPKI/blob/main/PrivateCA/HTTPS_Application_Usecase.md)