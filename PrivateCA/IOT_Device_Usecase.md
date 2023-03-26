## 物联网设备证书：

如果您正在构建客户端设备需要向中央控制服务器验证其身份的 IOT 应用程序，您可以使用 AWS IOT Core 或您的组织构建的您自己的控制服务器。 在研讨会的这一部分，我们将通过动手练习向您展示如何获取客户端设备证书并将它们部署到 IOT 设备模拟器，以及 AWS IOT Core 如何验证这些设备证书以建立成功的 TLS 连接。

### 0.创建基础模板

在您的 AWS 账户中运行 CF 模板堆栈 template-security-admin.yaml。

### 1. 一个名为 **IOTDevRole** 的 IAM 角色是 IOT 开发人员将担任的角色。

* 通过在您当前登录的 AWS 帐户中的 AWS 控制台上使用 switch 角色来承担名为 **IOTDevRole** 的角色

* 此角色拥有 IOT 开发人员构建 IOT 设备证书身份验证用例所需资源所需的权限。

* 如果您不熟悉角色切换，请根据需要遵循本教程：右键单击 [在控制台中承担角色](https://github.com/aws-samples/data-protection/blob/master/usecase-9/img/SwitchRole.pdf)

#### 2. 通过部署下面的 cloudformation 模板构建物联网用例所需的基础设施

请通过右键单击下载 CF 模板并将链接另存为文件名 *template-iot-dev.yaml* [IOT Developer Cloudformation Stack](https://raw.githubusercontent.com/aws-samples/data-protection/master/usecase-9/cf-templates/template-iot-dev.yaml) 通过右键单击并将 yaml 文件保存在笔记本电脑上。

在您的 AWS 账户中上传并启动 cloudformation 堆栈。 如果您对此不熟悉，请按照此处的说明进行操作，方法是右键单击并在新的浏览器选项卡中打开链接 [部署 IOT 开发人员 Cloudformation 堆栈说明](https://github.com/aws-samples/data-protection/blob/master/usecase-9/img/IoTStackSteps.pdf)
此 cloudformation 部署大约需要 2 分钟才能完成。

#### 3. 对于下一部分，您需要一个 Cloud9 IDE 环境设置来执行代码。 请按照以下说明进行操作：

* 在您的 AWS 控制台中导航到 Cloud9 服务
* 打开名为 **IOT 用例环境** 的 Cloud9 IDE 环境。 环境启动大约需要 30 秒。
* 在 Cloud9 IDE 环境中，您会在屏幕左侧的文件夹窗格中找到一个名为 **usecase-9** 的文件夹
* 删除那个文件夹

在您的 Cloud9 环境中打开 **usecase-9** 目录

* 在 IDE 中右键单击（在 MacOS 上：按住 control 单击）名为 **environment-setup.sh** 的文件，然后选择运行
* 这个脚本大约需要一分钟才能完成
* 该脚本为研讨会的这一部分安装了各种工具和包，请参阅脚本中的注释以获取更多信息
* 当您看到**环境设置完成**时，脚本已完成，可以在文件 `setup.log` 中找到其他日志信息

#### 4. 创建验证证书并由您之前创建的从属 CA 签名

验证证书是必需的，因为这是 AWS IOT 核心用来确定您有权访问从属 CA 以签署最终实体证书的机制。 在此过程中还需要一个注册码，AWS IOT Core 使用此代码来确定创建和上传验证证书操作的主体是否有权访问 AWS 控制台中的 IOT Core。

在 Cloud9 环境中：

打开一个 bash 终端。

使用以下命令将目录更改为 /home/ec2-user/environment ：

```
cd /home/ec2-user/environment/data-protection/usecase-9
```


使用以下命令生成 RSA 2048 密钥对。 将创建文件 verification_cert.key。

```
openssl genrsa -out verification_cert.key 2048
```

每个 AWS 账户都有一个唯一的 IOT 注册码。 作为验证过程的一部分，CSR（证书签名请求）的通用名称必须设置为注册码。

让我们从 IOT Core 获取注册码以放入 CSR 的通用名称中。 将注册码复制到剪贴板。

```
aws iot get-registration-code
```

我们现在将生成验证证书签名请求 (CSR)。 在提示符处填写值，并在提示输入“Common name”时输入上述命令返回的注册码（不带引号）。 对于其他参数，选择任何你喜欢的。 不要为挑战密码或可选的公司名称输入任何内容，只需按两次 enter 就可以了。

```
openssl req -new -key verification_cert.key -out verification_cert.csr
```

您应该会看到生成的文件 verification_cert.csr。

verification_cert.csr 现在将由您之前创建的从属 CA 签名，证书内容将放入 .pem 文件中

使用以下命令运行脚本“verification-cert.sh”：

```
bash verification-cert.sh
```

此脚本将颁发验证证书，并从 ACM PCA 获取从属 CA 证书并将其上传到名为“certificate-holder-<your-account-number>”的 S3 存储桶。 该存储桶将位于您当前运营所在的区域。

此时，AWS IOT 核心服务已确定您拥有使用您账户中的从属 CA 签署设备证书的必要权限。

####  5.从S3 bucket下载从属CA证书和验证证书到你的笔记本

按照此处的说明进行操作：

右键打开[下载验证证书和从属CA证书](https://github.com/aws-samples/data-protection/blob/master/usecase-9/img/DownloadCAVerificationCert.pdf)


#### 6.通过在AWS控制台上传从属CA证书和验证证书到IOT Core来注册CA

按照此处的说明进行操作：

右击打开[注册CA](https://github.com/aws-samples/data-protection/blob/master/usecase-9/img/UploadVeriCertSubordinateCAIOTCore.pdf)

现在 IOT 核心加载了从属 CA 证书。 这意味着任何由从属 CA 签署的设备证书都可以被 IOT 核心信任和验证。

#### 7.让我们在Cloud9环境中创建一个设备证书

在此步骤中，您将创建一个设备证书，然后将其与 IOT 事物相关联。 在 Cloud9 环境中，打开终端并执行以下命令为 IOT 设备创建 RSA 密钥对：

```
openssl genrsa -out device_cert.key 2048
```

让我们为 IOT 设备生成 CSR。 对于 CSR 参数，您可以将“iotdevice1”作为通用名称，但对于所有其他参数，您可以自行选择。 不要为挑战密码和可选的公司名称设置任何内容。 只需按回车键输入这些参数。 使用以下命令生成 CSR（证书签名请求）

```
openssl req -new -key device_cert.key -out device_cert.csr
```

运行脚本“device-cert.sh”，它会颁发设备证书，并由您之前创建的从属 CA 对其进行签名，并将其上传到名为“certificate-holder-<your-account-number>”的 S3 存储桶中。 您可以使用以下命令：

```
bash device-cert.sh
```

#### 8.从S3下载设备证书回Cloud9环境

转到 S3 控制台并按照以下说明操作：

右键单击并打开  [从 S3 下载设备证书](https://github.com/aws-samples/data-protection/blob/master/usecase-9/img/DownloadDeviceCert.pdf) 

#### 9. 创建提供发布和订阅 IOT 主题权限的 IOT Policy

附加到事物或设备的 IOT 策略为设备或事物提供 IOT 事物可以发布和订阅的主题的权限。 该策略还提供了为连接到 IOT 核心的事物添加名称的能力。

按照此处的说明进行操作：

右键单击并打开【创建 IOT 策略】(https://github.com/aws-samples/data-protection/blob/master/usecase-9/img/CreateIOTPolicy.pdf)

您现在已经创建了一个名为 `alexa_temperature_policy` 的策略。 此策略将允许 IOT 设备发布到名为“alexa/temperature”的主题。 例如，考虑一个 IOT 设备记录温度并发布到这个名为“alexa/temperature”的 IOT 主题。 您将在后续步骤中将该策略附加到 IOT 事物。

#### 10.注册设备证书并在IOT核心服务内创建一个东西

按照此处的说明进行操作：

右键单击并打开 [Create and register a IOT thing](https://github.com/aws-samples/data-protection/blob/master/usecase-9/img/RegisterAThing.pdf)

#### 11.配置MQTT（物联网消息传递标准）客户端

我们将使用 MQTT 客户端来模拟物联网设备。 您的 Cloud9 环境中已预安装 MQTT 客户端。 mqtt 客户端需要与 IOT 核心端点通信，并且 mqtt 客户端和 AWS IOT 核心端点之间相互 TLS 连接所需的各种证书和密钥已在 config.properties mqq 配置文件中配置。

在 Cloud9 环境中从 bash 终端打开 mqtt 配置文件：
```
c9 open /home/ec2-user/.mqtt-cli/config.properties
```

在我们的 Cloud9 环境中的 bash 终端中，在终端中使用以下 cli 命令获取 mqtt 端点 URL 或 mqtt 主机
```
aws iot describe-endpoint --endpoint-type iot:Data-ATS
```
输出将采用这种格式

{ "endpointAddress": "b12qfcnz2tnjiu-ats.iot.us-east-1.amazonaws.com" }


将参数 endpointAddress 的值复制到 config.properties 文件中的 mqtt.host 的值中。

还将其他属性更改为如下所示的值。 不要忘记保存文件。

```
mqtt.port=8883
mqtt.host= {Value of the endpointAddress produced by the above describe-endpoint CLI Command}
mqtt.version=3
client.id.prefix=mydevice
```

#### 12. Pub sub 练习，我们可以看到消息通过 HTTPS 连接在设备模拟器和 IOT 核心主题之间流动。

在浏览器选项卡中导航到 AWS IOT 核心服务，然后按照以下说明操作：

右键单击并打开[订阅 IOT 主题](https://github.com/aws-samples/data-protection/blob/master/usecase-9/img/SubscribeIOTTopic.pdf)

**从 MQTT 客户端发布到 IOT 核心：**

通过键入以下命令打开 mqtt shell：
```
mqtt shell
```
从 MQTT 客户端发布到 IOT Core

使用以下命令连接到 IOT 核心，然后使用 pub 命令发布 Hello 消息。 在每个命令后按回车键：
```
con -i mydevice

pub -t alexa/temperature -m Hello 

```
按照此处的说明查看 MQTT 客户端发布到 IOT 核心的消息：

右键打开[查看IOT设备消息发布](https://github.com/aws-samples/data-protection/blob/master/usecase-9/img/SeeIOTDevicePublishedMessage.pdf)

```
type **disconnect** and press enter to Disconnect from the IOT core connection

type exit and press enter to exit out of the MQTT shell
```
**从 IOT 核心发布到 MQTT 客户端：**

通过键入以下命令打开 mqtt shell：
```
mqtt shell
```
使用以下命令连接到 IOT 核心：
```
con -i mydevice

subscribe to the topic monthly_average_temperature by executing the following command :

sub -q 1 -t cloud/monthly_average_temperature -s -oc
```
从 IOT Core 控制台，发布到主题 cloud/monthly_average_temperature

您应该在 Cloud9 环境中的 mqtt 客户端上看到消息“Hello from the IOT console”