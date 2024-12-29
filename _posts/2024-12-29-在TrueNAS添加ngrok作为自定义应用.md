---
layout: post
title: "在TrueNAS添加ngrok作为自定义应用"
---

介绍在TrueNAS SCALE 24.10.1上添加ngrok作为自定义应用的方法，从而轻松实现内网穿透。

### 预备知识

- TrueNAS SCALE的基本使用方法
- Docker的基本使用方法

### 环境

- VMware Workstation 17.6.2 build-24409262
- TrueNAS SCALE 24.10.1在虚拟机上

### 步骤

#### 进入自定义应用页面

点击**应用-探索应用程序-自定义应用程序**。

#### 填写配置

1. Application name

应用名称：`ngrok`

2. Image Configuration

Repository：`ngrok/ngrok`

标签：`latest`

Pull Policy：`Pull the image if it is not already present on the host.`

3. Container Configuration

主机名，留空或任意

Entrypoint，留空

命令，添加以下3条：
```
http
--url=nice-guy.ngrok.com（在ngrok上查询域名，网址为https://dashboard.ngrok.com/domains）
172.16.x.x:port（内网应用地址）
```

时区，按需填写，推荐填真实本地时间

Environment Variables，填写以下1条：
```
名称：NGROK_AUTHTOKEN
值：XXXXXXXXXXXXXXXXXX（在ngrok上查询token，网址为https://dashboard.ngrok.com/get-started/your-authtoken）
```

Restart Policy
```
Always - Restarts the container until its removal.
```

4. Security Context Configuration

全采用默认配置，不做更改。

5. Network Configuration

Host Network，开启

其他配置使用默认配置，不做更改。

6. Portal Configuration

全采用默认配置，不做更改。

7. Storage Configuration

全采用默认配置，不做更改。

8. Labels Configuration

全采用默认配置，不做更改。

9. Resources Configuration

全采用默认配置，不做更改。

### 参考文献

[1] 最好用的内网穿透工具合集 [Z/OL]. [https://github.com/SexyBeast233/SecBooks/blob/main/%E3%80%90%E5%86%85%E7%BD%91%E7%B3%BB%E5%88%97%E3%80%91intranet/%E6%9C%80%E5%A5%BD%E7%94%A8%E7%9A%84%E5%86%85%E7%BD%91%E7%A9%BF%E9%80%8F%E5%B7%A5%E5%85%B7%E5%90%88%E9%9B%86.md](https://github.com/SexyBeast233/SecBooks/blob/main/%E3%80%90%E5%86%85%E7%BD%91%E7%B3%BB%E5%88%97%E3%80%91intranet/%E6%9C%80%E5%A5%BD%E7%94%A8%E7%9A%84%E5%86%85%E7%BD%91%E7%A9%BF%E9%80%8F%E5%B7%A5%E5%85%B7%E5%90%88%E9%9B%86.md).

[2] Ngrok. Setup & Installation [Z/OL]. [https://dashboard.ngrok.com/get-started/setup/docker](https://dashboard.ngrok.com/get-started/setup/docker).
