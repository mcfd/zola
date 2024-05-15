+++
title =  "使用 maddy mail 搭建个人邮箱服务"

date = 2024-05-15T01:37:00Z



[taxonomies]

tags = ["maddy","email","自托管","个人邮箱"]
+++

在拥有一个域名，一台公网服务器的情况下使用 maddy 配置个人自托管邮箱服务。

<!--more-->

# 准备

**确保防火墙开放以下端口**

| 服务         | 端口  |
| ---------- | --- |
| SMTP 入站    | 25  |
| IMAP4      | 143 |
| IMAP4（SSL） | 993 |
| SMTP（SSL）  | 465 |
| SMTP（邮件提交） | 587 |

> 验证 25 端口是否开放的方法：
> 
>     1.将其他可访问服务切换到25端口尝试访问
> 
>     2.搭建完成后使用 `telnet ip port` 的方式尝试能否请求到 mail 服务



# 安装 maddy

##### 安装

这里使用 docker，参考[官网手册]([Docker - maddy](https://maddy.email/docker/))。

容器端口映射**不需要变更**，其中需要更改的变量是：

```bash
MADDY_HOSTNAME=mx.maddy.test
MADDY_DOMAIN=maddy.test

maddydata:/data
```

*maddy.test* 为你的~~一级~~域名 *xxx@xxx.com* 中的 *xxx.com*

*mx.maddy.test* 为你的邮箱服务域名 不一定是 *mx.* 也可以是 *mail.* 或者 *abab.mx.mail.* 这取决于一级域名 **mx** 记录的**指向**



**持久化** *maddydata:/data* 

位于 **:**  前面的是宿主机目录 后面的则是容器内目录

容器内目录不需要修改，需要将 *maddydata* 修改成和你用来存 *~~.avi~~* 一样安全的地方，例如 */opt/maddy*  下文需要在这个目录里放置域名证书



> 如果运行
> 
> ```bash
> docker volume create maddydata
> ```
> 
> 就不用更改**持久化**映射目录了 按照默认填写就行 一般目录为`/var/lib/docker/volumes/maddydata/_data/`



##### 配置域名证书

首先你需要申请一个证书 推荐使用 **acme.sh**

申请的对象为上文填写的 

- ~~mx.maddy.test~~

- ~~maddy.test~~



之后将得到的

- fullchain.pem

- privkey.pem

放入 */opt/maddy* 或 */var/lib/docker/volumes/maddydata/_data/*  （取决于上面你使用的方式）中的 *tls* 文件夹（如果没有就新建一个）



##### 重启

这下再重启容器就应该正常启动了



# 配置域名解析

| 类型           | 域名                    | 值                                                                     |
| ------------ | --------------------- | --------------------------------------------------------------------- |
| MX           | maddy.test            | *mx.maddy.test*（决定下面的）                                                |
| A（IPv6为AAAA） | mx.maddy.test         | 邮箱服务器IPv4地址（使用IPV6则替换即可）                                              |
| TXT          | mx.maddy.test         | v=spf1 mx ~all                                                        |
| TXT          | maddy.test            | v=spf1 mx ~all                                                        |
| TXT          | _dmarc.maddy.test     | v=DMARC1; p=quarantine; ruf=mailto:~~postmaster@example.com~~（修改成自己的） |
| TXT          | _mta-sts.maddy.test   | v=STSv1; id=1                                                         |
| TXT          | _smtp._tls.maddy.test | v=TLSRPTv1;rua=mailto:~~postmaster@example.com~~ （修改成自己的）             |

在上面的目录（tls的父目录）下，会有一个 `dkim_keys` 文件夹

里边会有两个文件

- ~~xxx.com~~_default.dns

- ~~xxx.com~~_default.key



需要将 `.dns` 结尾的文件里的内容添加为 `default._domainkey.xxx.com` 的 **txt** 域名解析

开头为：*v=DKIM1; k=~~xxxxxxxxxxxxxxxxxxxxx~~*



> 此外还需要做 *PTR* 反向域名解析，用于将 IP 解析到域名
> 
> 如果没有做，对于一些邮箱比如 **tuta** 会直接拒收 但 **gmail** **qq** 等邮箱一般只会扔入垃圾箱
> 
> **DMARC 策略** 和 **SPF 策略**  各有三种一般策略 **DMARC** 还有其他选项 详情可以自行[了解]([Cloudflare DMARC Management · Cloudflare DMARC Management docs](https://developers.cloudflare.com/dmarc-management/))



# 测试

运行

```bash
docker exec -it <maddy的容器名> maddy creds create <用户名>@maddy.test
```

之后你将输入密码（没有第二次输入验证）

接下来

```bash
docker exec -it <maddy的容器名> maddy imap-acct create <用户名 同上>@maddy.test
```

这样你就创建了一个新用户，快使用你的邮箱客户端登录上去看看吧！
