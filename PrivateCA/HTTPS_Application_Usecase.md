# ACM 私有证书颁发机构（ACM 私有 CA）

## 内部 HTTPS 应用程序用例

#### 7. 一个名为 **AppDevRole** 的 IAM 角色是应用程序开发人员将担任的角色。

* 通过在您当前登录的 AWS 账户中的 AWS 控制台上使用 switch role 来承担名为 **AppDevRole** 的角色

* 此角色具有应用程序开发人员构建 Web 应用程序所需的权限，该应用程序由应用程序负载均衡器前面和负载均衡器后面是一个 lambda 来源
提供网站的 HTML 代码。 应用程序开发人员还将有权根据他们选择的证书颁发机构颁发证书。

* 如果您不熟悉角色切换，请根据需要遵循本教程：[在控制台中担任角色](https://github.com/aws-samples/data-protection/blob/master/usecase-9/img/SwitchRole .pdf)

#### 8.通过部署下面的cloudformation模板构建应用程序基础设施

请通过右键单击下载 CF 模板并将链接另存为文件名 *template-appdev.yaml* [AppDev Cloudformation Stack](https://raw.githubusercontent.com/aws-samples/data-protection/master/usecase-9/cf-templates/template-app-dev.yaml) 通过右键单击并将 yaml 文件保存在笔记本电脑上。


#### 9. 下一步是颁发私有证书以放置在应用程序负载均衡器上。

* 在新的浏览器选项卡中打开此链接以获取步骤：[颁发私有证书](https://github.com/aws-samples/data-protection/blob/master/usecase-9/img/IssuePrivateCertificate.pdf)
* [测验](https://bit.ly/2KXE06k)

#### 10. 将 HTTPS 侦听器和私有证书附加到 ALB。

* 在新的浏览器选项卡中打开此链接以获取步骤：[附加 HTTPS 侦听器](https://github.com/aws-samples/data-protection/blob/master/usecase-9/img/ApplyCertToLoadBalancer.pdf)


#### 11. 使用您正在使用的浏览器验证 ALB 的身份。 请在新的浏览器选项卡中打开链接

对于 Firefox：[在 Firefox 浏览器上验证证书身份](https://github.com/aws-samples/data-protection/blob/master/usecase-9/img/ValidateALBIdentityFirefox.pdf)

对于 Google Chrome：[在 Chrome 浏览器上验证证书身份](https://github.com/aws-samples/data-protection/blob/master/usecase-9/img/ValidateALBIdentityChrome.pdf)

对于 Microsoft Edge：[在 Microsoft Edge 浏览器上验证证书身份](https://github.com/aws-samples/data-protection/blob/master/usecase-9/img/ValidateALBIdentityEdge.pdf)

对于 Windows 上的 Google Chrome：[在 Windows 上验证证书身份](https://github.com/aws-samples/data-protection/blob/master/usecase-9/img/WindowsALBCert.pdf)