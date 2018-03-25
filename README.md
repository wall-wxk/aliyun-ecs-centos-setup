# 阿里云 ECS Centos 7 配置指南

配置过好多台 CentOS 机器了，为了避免重复输入命令和查阅 StackOverflow、DigitOcean、Nodejs 等文档，在这里记录下一次比较完整的配置以供以后参考。对你有用那就更好了，当然也欢迎贡献。

## 查看机器信息

```bash
# 发行版
cat /etc/redhat-release # 除 Centos 外标准的是 cat /etc/issue
# 机器配置
cat /proc/cpuinfo
```

## SSH 信任关系

在做一切事情前建议先配置信任关系，让你的电脑可以免密码登录 ECS：

```bash
ssh-copy-id root@xxx.xxx.xxx.xxx # 你的 ECS IP
```

建议你的 `~/.ssh/config` 中包含这样一段配置：

```
Host ecs1
    Hostname xxx.xxx.xxx.xxx
    User root
```

参考：http://harttle.land/2016/09/14/ssh-auto-login.html

## 配置 yum 源

既然有包管理，速度一定要到 10M/s。阿里云官方提供了源，把默认源替换掉：

```bash
cd  /etc/yum.repos.d/
wget http://mirrors.aliyun.com/repo/Centos-7.repo
mv CentOS-Base.repo CentOS-Base.repo.bak
mv Centos-7.repo CentOS-Base.repo
```

然后更新一下：

```bash
yum clean all
yum makecache
yum update
```

## 用户与权限

添加和修改用户的命令总是记不住，这里写出来供参考：

```bash
# 添加一个名叫 harttle 的用户，放到 users 组
useradd harttle -G users
# 把 harttle 添加到 wheel 组
usermod harttle -g wheel
# 删除用户
userdel harttle
# 设置密码
passwd harttle
# 切换到用户
su harttle
```

> `wheel` 组默认开启了 sudo 权限，如果没有你可以去 `/etc/sudoers` 中把包含 `%wheel  ALL=(ALL)       ALL` 的一行取消注释。

## 访问 Git 服务

为了能够访问 Git 服务，需要安装 Git 以及配置公钥。首先安装 Git：

```bash
yum install git
```

生成公钥并且把 ECS 的公钥配置到你的 Git 服务上：

```bash
ssh-keygen
cat ~/.ssh/id_rsa.pub 
```

## 安装 Nodejs

```bashrc
# 先获取 epel
yum install epel-release
# 再安装 Node.js
yum install nodejs
# 配置到淘宝源
npm config set registry http://registry.npm.taobao.org
```

参考：https://www.digitalocean.com/community/tutorials/how-to-install-node-js-on-a-centos-7-server

## 安装 MongoDB

注意这里安装的是 `mongodb-org` 而不是源里面的 `mongodb`。先添加源，打开 `/etc/yum.repos.d/mongodb-org.repo` 并添加以下内容：

```
[mongodb-org-3.4]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.4/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-3.4.asc
```

使用 `yum repolist` 可以查看是否成功，然后就可以安装 MongoDB 了：

```bash
yum install mongodb-org
```

如果希望开机自己启动可以用 systemd 来管理：

```bash
# 立即启动
sudo systemctl start mongod
# 设置开机启动
sudo systemctl enable mongod
```

## 防火墙开启端口

CentOS 中引入的 `firewall` 会默认阻止端口，如果发生此情况需要开启你要的端口，并重新加载 `firewall`。

```bash
firewall-cmd --zone=public --add-port=8080/tcp --permanent
firewall-cmd --reload
```

见： https://stackoverflow.com/questions/24729024/open-firewall-port-on-centos-7

> 如果还不行可能是需要去管理网站设置防火墙，参考： https://help.aliyun.com/knowledge_detail/31710.html
