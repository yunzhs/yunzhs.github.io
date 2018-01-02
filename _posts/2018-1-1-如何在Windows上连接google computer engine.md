---
layout:     post
title:      在Windows上连接google computer engine
subtitle:   
date:       2018-1-1
author:     yunzhs
header-img: img/Dyanna the Luna.jpg
catalog: true
tags:
    - linux
    - 服务器部署
typora-root-url: ..
typora-copy-images-to: ..\img\posts
---

# 用putty使用SSH密钥连接google computer engine

1. 下载 `puttygen.exe`](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html)。

2. 运行 PuTTYgen。对于本示例而言，只需运行您已下载的 `puttygen.exe` 文件即可。系统会打开一个窗口，您可以在其中配置自己的密钥生成设置。

   ![Snipaste_2018-01-01_01-30-57](/img/posts/Snipaste_2018-01-01_01-30-57.png)

3. 点击 **Generate** 按钮以生成新的密钥对。在大多数情况下，使用默认参数就可以，但您必须生成至少有 2048 位的密钥。您生成完密钥对后，该工具将显示您的公钥值。

   ![Snipaste_2018-01-01_01-36-42](/img/posts/Snipaste_2018-01-01_01-36-42.png)

4. 在 **Key comment** 部分中，将现有文本替换为实例中您将要为其应用密钥的用户的名称。

5. （可选）输入一个 **Key passphrase** 来保护您的密钥。

6. 点击 **Save private key** 将私钥保存到一个文件中。在本示例中，将私钥保存为 `my-ssh-key.ppk`。

7. 点击 **Save public key** 将您的公钥写入文件以备稍后使用。在本示例中，将公钥保存为 `my-ssh-key`。现在，让 PuTTYgen 窗口保持打开状态。

8. 转到您项目的元数据页面的 **SSH 密钥**部分。

   ![Snipaste_2018-01-01_01-24-47](/img/posts/Snipaste_2018-01-01_01-24-47.png)



## 在 Windows 工作站上使用 PuTTY 进行连接

在 Windows 工作站上，您可以使用 [PuTTY](https://en.wikipedia.org/wiki/PuTTY) 工具连接到您的实例。要使用 PuTTY 连接到您的实例，请执行以下操作：

1. 如果您还没有将任何公钥应用于您的 Cloud Platform 控制台项目，请[生成新的密钥对并将其应用于该项目](https://cloud.google.com/compute/docs/instances/connecting-to-instance#generatesshkeypair)。

2. [下载 `putty.exe`](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html)。

3. 运行 PuTTY 工具。对于本示例而言，只需运行您已下载的 `putty.exe` 文件即可。系统会打开一个窗口，您可以在其中配置您的连接设置。

4. 在 [Google Cloud Platform 控制台](https://console.cloud.google.com/)中，找到您要连接到的实例的外部 IP 地址。转到您的实例列表。

   [转到“实例”页面](https://console.cloud.google.com/compute/instances)

   ​

5. 在 PuTTY 工具中，在****主机名字段中指定您的用户名和您要连接到的实例的外部 IP 地址。在下面的示例中，用户名为 `jane_doe`，外部 IP 地址为 `203.0.113.2`。

   ![将主机名字段设置为 jane_doe@203.0.113.2](https://cloud.google.com/compute/images/connecting/putty_set_hostname.png)

   按以下格式输入您的用户名和外部 IP 地址：

   ​

   ```
    [USERNAME]@[EXTERNAL_IP_ADDRESS]
   ```

   其中：

   - `[USERNAME]` 是连接到实例的用户的名称。在创建 SSH 密钥对时，就指定了该 SSH 密钥对的用户名。如果实例具有该用户的有效 SSH 公钥，并且您拥有与之匹配的 SSH 私钥，则您可以以该用户的身份连接到此实例。
   - `[EXTERNAL_IP_ADDRESS]` 是您要连接到的实例的外部 IP 地址。

6. 在 PuTTY 窗口的左侧，导航到 **Connection** > **SSH** > **Auth**。

7. 将 **Private key file for authentication** 字段设置为您的私钥文件的路径。

   ![在私钥文件字段中设置 my-ssh-key.ppk 文件的路径。](https://cloud.google.com/compute/images/connecting/putty_set_private_key.png)

8. 点击 **Open** 以打开一个将要连接到您的实例的终端。

如果连接成功，您就可以使用终端在您的实例上运行命令。完成操作后，可使用 `exit` 命令断开与实例的连接。

最后记得在打开前将你的配置保存一下.

![1514780289807](C:\Users\hasee\AppData\Local\Temp\1514780289807.png)



## 将Putty生成的PrivateKey转换为SecureCRT所需的PublicKey

步骤：

1. 打开Putty Key Generator，点击"Load"按钮，然后选择之前生成的私钥；
2. Load成功后，选择菜单中的"Conversions”->"Export OpenSSH key"；
3. 然后会弹出保存文件对话框，选择一个你需要的名字，比如"openssh-key"（ 注意：这一步保存的文件名不能有任何后缀，按照原文作者所述，如果用了比如openssh-key.pub的公钥文件，则SecureCRT会在同样目录下寻找名为"openssh-key"的私钥。）；
4. 此时SecureCRT使用上述不带后缀的openssh-key文件就可以成功登录；
5. 根据原文作者所述，还需要再次保存为名为"openssh-key.pub"的文件（即多了个pub的后缀），此时既可以使用"openssh-key.pub"在SecureCRT中进行登录；（本人第4步即可成功使用，但保险起见还是把原作者的第5步给出来，以供所需人士参考）；