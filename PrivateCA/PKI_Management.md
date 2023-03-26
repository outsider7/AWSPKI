## 4. 以PKI管理员身份管理私有CA

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