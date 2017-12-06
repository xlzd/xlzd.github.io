---
title: 怎样在 Ubuntu 16.04 上安装 OpenVPN 服务
date: 2016-10-10 09:20:04
tags: 
  - Linux
  - GFW
---

![open_vpn_server_tw.jpg][1]

**参考自 [How To Set Up an OpenVPN Server on Ubuntu 16.04][2]**

## 为什么需要 OpenVPN

对我来讲，有两个原因：

 - 安全地在不安全的网络环境下上网：如需要在酒店、咖啡厅或者不可信任的 Wi-Fi 环境下上网时，我需要确保自己不会被监听。
 - 跨过防火墙，享受自由的网络环境。

那么，为什么我不用 ShadowSocks 呢？答案是，其实我也在用，不过它在 iOS 上的表现确实无法令人满意，另外，PC 端使用 Chrome 配合 ss 翻墙的时候需要很复杂的配置（或许我不会用吧），相比之下， OpenVPN 可以非常方便的全部搞定（除了安装比较复杂）。所以，这篇博客记录下 OpenVPN 服务的安装过程，以供参考。


文中使用的服务器是 Ubuntu 16.04，不过 Debian 系的操作系统应该是可以通用的。


<!--more-->


---

## 安装 OpenVPN，设置 CA 目录

首先，在服务器端安装 OpenVPN 服务。我们可以很方便地通过 `apt-get` 安装，另外我们也需要安装`easy-rsa`：
```
$ sudo apt-get update
$ sudo apt-get install openvpn easy-rsa
```

然后，复制 `easy-rsa` 模板到 `home` 目录：

```
$ make-cadir ~/openvpn-ca
$ cd ~/openvpn-ca
```
---

## 配置 CA 变量

进入 `openvpn-ca` 目录之后，用 vim (或者任意编辑器) 打开 `vars` 文件，到最后一部分，你将会看到如下内容：

```
. . .

export KEY_COUNTRY="US"
export KEY_PROVINCE="CA"
export KEY_CITY="SanFrancisco"
export KEY_ORG="Fort-Funston"
export KEY_EMAIL="me@myhost.mydomain"
export KEY_OU="MyOrganizationalUnit"

. . .
```

修改为任意你想要修改的值，只要不留空就好了：

```
. . .

export KEY_COUNTRY="CN"
export KEY_PROVINCE="BJ"
export KEY_CITY="Beijing"
export KEY_ORG="xlzd"
export KEY_EMAIL="what@the.fuck"
export KEY_OU="Community"

. . .
```

然后，还需要将 `KEY_NAME` 改为你喜欢的，这里简单起见，我们改成 `server`：

```
export KEY_NAME="server"
```

然后，保存并关闭文件。

---

## 构建 Certificate Authority

在刚才的目录中，执行 `source vars` ，然后，你将会看到如下输出：

```
NOTE: If you run ./clean-all, I will be doing a rm -rf on /home/xlzd/openvpn-ca/keys
```

然后执行：

```
$ ./clean-all
$ ./build-ca
```

这将会启动创建根证书颁发密钥、证书的过程。由于我们刚才修改了 `vars` 文件，所有值应该都会自动填充。所以，一路回车就好了：

```
Output
Generating a 2048 bit RSA private key
..........................................................................................+++
...............................+++
writing new private key to 'ca.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [CN]:
State or Province Name (full name) [BJ]:
Locality Name (eg, city) [Beijing]:
Organization Name (eg, company) [xlzd]:
Organizational Unit Name (eg, section) [Community]:
Common Name (eg, your name or your server's hostname) [the.fuck]:
Name [server]:
Email Address [waht@the.fuck]:
```

到此，我们就有了创建以下步骤需要的 CA 证书。

---

## 创建服务器端证书、密钥和加密文件

执行 `./build-key-server server` 命令，然后继续一路回车就好了。到最后，你需要输入两次 `y` 注册证书和提交：

```
. . .

Certificate is to be certified until May  1 17:51:16 2026 GMT (3650 days)
Sign the certificate? [y/n]:y


1 out of 1 certificate requests certified, commit? [y/n]y
Write out database with 1 new entries
Data Base Updated
```

然后还需要生成一些其他东西，在终端执行 `./build-dh`，这个操作大约会花费几分钟不等。然后，我们可以生成 HMAC 签名加强服务器的 TLS 完整性验证功能：

```
openvpn --genkey --secret keys/ta.key
```

---

## 生成客户端证书、密钥对

这一步之后可能会执行多次以生成不同的证书，这里我们以 `xclient` 作为第一组密钥对的名字：

```
cd ~/openvpn-ca
source vars
./build-key xclient
```

跟刚才一样，一路回车就好。

---

## 配置 OpenVPN 服务

首先，复制文件到 OpenVPN 的目录下：

```
$ cd ~/openvpn-ca/keys
$ sudo cp ca.crt ca.key server.crt server.key ta.key dh2048.pem /etc/openvpn
```

然后，复制并解压一个 OpenVPN 的配置示例：

```
$ gunzip -c /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz | sudo tee /etc/openvpn/server.conf
```

接着是调整配置，打开 `/etc/openvpn/server.conf` 文件，找到 `redirect-gateway` 的位置，去掉注释，修改为如下：

```
push "redirect-gateway def1 bypass-dhcp"
```

然后找到 `dhcp-option` 位置，修改为下面这样：

```
push "dhcp-option DNS 208.67.222.222"
push "dhcp-option DNS 208.67.220.220"
```

再找到 `tls-auth` 位置，去掉注释，并在下面新增一行：

```
tls-auth ta.key 0 # This file is secret
key-direction 0
```

最后，去掉 `user` 和 `grup` 行前的注释：

```
user nobody
group nogroup
```

---

## 调整服务器网络配置

### 允许 IP 转发

编辑 `/etc/sysctl.conf` 文件，去掉 `net.ipv4.ip_forward` 设置前的注释：

```
net.ipv4.ip_forward=1
```
输入 `sudo sysctl -p` 以读取文件并对当前会话生效。

### 调整 UFW 规则

编辑 `/etc/ufw/before.rules` 文件，在文件顶部，新增如下 11-18 行的内容：

```
01 #
02 # rules.before
03 #
04 # Rules that should be run before the ufw command line added rules. Custom
05 # rules should be added to one of these chains:
06 #   ufw-before-input
07 #   ufw-before-output
08 #   ufw-before-forward
09 #
10 
11 # START OPENVPN RULES
12 # NAT table rules
13 *nat
14 :POSTROUTING ACCEPT [0:0] 
15 # Allow traffic from OpenVPN client to eth0
16 -A POSTROUTING -s 10.8.0.0/8 -o eth0 -j MASQUERADE
17 COMMIT
18 # END OPENVPN RULES
19
20 # Don't delete these required lines, otherwise there will be errors
*filter
. . .

```

其中，第 16 行还需要做一点调整。在终端执行 `ip route | grep default` 命令，你会看到类似如下的输出：

```
default via 100.110.78.1 dev ens3
```

`dev` 之后的内容便是我们需要的，如我执行后输出如上，则我需要的是 `ens3`，每个人的结果可能不同，用它替换掉刚才文件第 16 行的 `eth0`，然后保存文件，退出。

接着需要修改 `/etc/default/ufw` 文件，找到 `DEFAULT_FORWARD_POLICY` 设置，修改为：
```
DEFAULT_FORWARD_POLICY="ACCEPT"
```

### 打开 OpenVPN 端口并使变化生效

执行下面的命令：

```
$ sudo ufw allow 1194/udp

$ sudo ufw disable
$ sudo ufw enable
```

---

## 启动 OpenVPN

执行：

```
$ sudo systemctl start openvpn@server
```

然后设置自启动：
```
$ sudo systemctl enable openvpn@server
```

---

## 创建客户端配置

执行：
```
$ mkdir -p ~/client-configs/files

$ chmod 700 ~/client-configs/files

$ cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf ~/client-configs/base.conf

```

然后打开 `~/client-configs/base.conf` 文件，修改 `remote server_IP_address 1194` 一行为你的服务器公网 IP，然后去掉 `user` 和 `group` 前的注释：

```
# Downgrade privileges after initialization (non-Windows only)
user nobody
group nogroup
```

找到 `ca`/`cert`/`key`，注释掉：

```
# SSL/TLS parms.
# See the server config file for more
# description.  It's best to use
# a separate .crt/.key file pair
# for each client.  A single ca
# file can be used for all clients.
#ca ca.crt
#cert client.crt
#key client.key
```

最后在文件末新增一行：

```
key-direction 1
```
保存，退出文件。

### 创建配置生成脚本

新建 `~/client-configs/make_config.sh` 文件，复制如下内容：

```
#!/bin/bash

# First argument: Client identifier

KEY_DIR=~/openvpn-ca/keys
OUTPUT_DIR=~/client-configs/files
BASE_CONFIG=~/client-configs/base.conf

cat ${BASE_CONFIG} \
    <(echo -e '<ca>') \
    ${KEY_DIR}/ca.crt \
    <(echo -e '</ca>\n<cert>') \
    ${KEY_DIR}/${1}.crt \
    <(echo -e '</cert>\n<key>') \
    ${KEY_DIR}/${1}.key \
    <(echo -e '</key>\n<tls-auth>') \
    ${KEY_DIR}/ta.key \
    <(echo -e '</tls-auth>') \
    > ${OUTPUT_DIR}/${1}.ovpn
```

保存并赋予执行权限：

```
$ chmod 700 ~/client-configs/make_config.sh
```

---

## 生成客户端配置

执行：

```
$ cd ~/client-configs
$ ./make_config.sh xclient
```

然后，在 `~/client-configs/files` 目录下，便有了 `xclient.ovpn` 文件。将文件下载到本地即可使用了。

---

## 客户端下载

Windows： [OpenVPN][3]
OS X： [Tunnelblick][4]
iOS： [OpenVPN Connect][5]
Android: [OpenVPN Connect][6]

下载安装客户端之后，导入刚才的配置文件，就可以愉快又安全地体验没有墙的互联网啦~~~


  [1]: http://7xkpi6.com1.z0.glb.clouddn.com/blog/2016/06/27/329318860.jpg
  [2]: https://www.digitalocean.com/community/tutorials/how-to-set-up-an-openvpn-server-on-ubuntu-16-04
  [3]: https://openvpn.net/index.php/open-source/downloads.html
  [4]: https://tunnelblick.net/downloads.html
  [5]: https://itunes.apple.com/us/app/id590379981
  [6]: https://play.google.com/store/apps/details?id=net.openvpn.openvpn