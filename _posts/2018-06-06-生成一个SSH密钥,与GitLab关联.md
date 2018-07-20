---
layout:     post
title:      生成SSH密钥,与GitLab关联
date:       2018-06-06
author:     yunzhs
header-img: img/post-bg-rwd.jpg
catalog: true
tags:
    - 随笔
typora-root-url: ..
typora-copy-images-to: ..\img\posts
---

## GitLab和SSH密钥

Git是一个分布式版本控制系统，这意味着你可以在本地工作，但你也可以共享或“推”你的变化到其他服务器。在将更改推送到GitLab服务器之前，您需要一个用于共享信息的安全通信通道。

SSH协议提供了这种安全性，并允许您在每次不提供用户名和密码的情况下向GitLab远程服务器进行身份验证。

## 查找现有的SSH密钥对

在生成新的SSH密钥对之前，通过打开shell或Windows命令提示符并运行以下命令，检查系统是否已在默认位置有一个：

Windows命令提示符：

```
type %userprofile%\.ssh\id_rsa.pub
```

在Windows / GNU / Linux / macOS / PowerShell上的Git Bash：

```
cat ~/.ssh/id_rsa.pub
```

如果您看到一个以ssh-rsa您为开头的字符串已经有一个SSH密钥对，并且您可以跳过下一节的生成部分并跳到复制到剪贴板步骤。如果您没有看到该字符串，或者想要使用自定义名称生成SSH密钥对，请继续下一步。

请注意，公共SSH密钥也可以如下命名：

`id_dsa.pub`
`id_ecdsa.pub`
`id_ed25519.pub`

## 生成一个新的SSH密钥对

要生成新的SSH密钥对，请使用以下命令：

在Windows / GNU / Linux / macOS上的Git Bash：

```
ssh-keygen -t rsa -C "your.email@example.com" -b 4096
```

windows：

或者在Windows上，您可以下载
 [PuttyGen](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html) 并按照[此文档](https://the.earth.li/~sgtatham/putty/0.67/htmldoc/Chapter8.html#pubkey-puttygen)生成SSH密钥对。


接下来，系统会提示您输入文件路径以保存您的SSH密钥对。

如果您还没有SSH密钥对，请按Enter键以使用建议的路径。使用建议的路径通常会允许您的SSH客户端自动使用SSH密钥对，而无需其他配置。

如果你已经有了一个SSH密钥对建议的文件路径，则需要输入一个新的文件路径，并声明此SSH密钥对将在您使用什么主机.ssh/config文件


输入文件路径后，系统将提示您输入密码以保护您的SSH密钥对。对SSH密钥对使用密码是最佳做法，但不是必需的，您可以通过按Enter键跳过创建密码。

注意：注意： 
 如果您想更改SSH密钥对的密码，可以使用
  `ssh-keygen -p <keyname>。`


下一步是复制公钥SSH密钥，因为之后我们会需要它。

要将公共SSH密钥复制到剪贴板，请使用以下相应的代码：

MacOS的：

`pbcopy < ~/.ssh/id_rsa.pub`
GNU / Linux（需要xclip软件包）：

`xclip -sel clip < ~/.ssh/id_rsa.pub`
Windows命令行：

`type %userprofile%\.ssh\id_rsa.pub | clip`
Windows / Windows PowerShell上的Git Bash：

`cat ~/.ssh/id_rsa.pub | clip`


最后一步是将您的公共SSH密钥添加到GitLab。

导航到“配置文件设置”中的“SSH密钥”标签。将你的钥匙粘贴在'钥匙'部分，并给它一个相关的'标题'。使用可识别的标题，如“工作笔记本电脑 -  Windows 7”或“家用MacBook Pro 15”。

如果您手动复制您的公用SSH密钥，请确保您复制了整个密钥，ssh-rsa并以您的电子邮件结尾。

或者，您可以运行ssh -T git@example.com
（替换example.com您的GitLab域）并验证您收到Welcome to GitLab消息来测试您的设置。![Snipaste_2018-06-05_13-24-09](/img/posts/Snipaste_2018-06-05_13-24-09.png)

## 使用非默认的SSH密钥对路径

如果您为GitLab SSH密钥对使用了非默认文件路径，则必须配置SSH客户端以查找您的GitLab私有SSH密钥以连接到GitLab服务器（可能gitlab.com）。

对于您当前的终端会话，您可以使用以下命令（替换other_id_rsa为您的专用SSH密钥）执行此操作：

在Windows / GNU / Linux / macOS上的Git Bash：

```
eval $(ssh-agent -s)

ssh-add ~/.ssh/other_id_rsa

```

要保留这些设置，您需要将它们保存到配置文件中。对于OpenSSH客户端，这是在~/.ssh/config某些操作系统的文件中配置的。以下是使用自己的SSH密钥的两个示例主机配置：

```
#GitLab.com server

Host gitlab.com

RSAAuthentication yes

IdentityFile ~/.ssh/config/private-key-filename-01

#Private GitLab server

Host gitlab.company.com

RSAAuthentication yes

IdentityFile ~/.ssh/config/private-key-filename

```

由于各种SSH客户端及其大量的配置选项，对这些主题的进一步解释超出了本文的范围。

公共SSH密钥必须是唯一的，因为它们将绑定到您的帐户。您的SSH密钥是您通过SSH推送代码时唯一的标识符。这就是为什么它需要唯一映射到单个用户。