---
title: 在本地登录远程服务器
date: 2019.06.15 12:00:00
categories:
  - Operation
tags: 
  - Linux
---

> 在 Mac 上使用终端，在 WIndow 上建议使用 `Git Bash` 。<br/>
> 假设服务器IP是 `120.130.140.150`, 使用 `root` 用户登录。

## 一、IP登录
输入 `ssh 用户名@你的IP地址` 后回车，再输入密码，如下。
```
$ ssh root@120.130.140.150
```

::: warning
这里可能回遇到 `ssh: connect to host 120.79.21.151 port 22: Connection refused`，通常引发该问题的原因是以下前两个，当然也可能是你不太可能遇到的第三个原因:
1. 22端口没开启外网访问
2. 默认 `ssh` 端口不是 `22` , 假设是 `10089` 。解决方案就是[别名登录](#2)
3. 之前使用过[SSH登录](#3)，已经在 `~/.ssh/known_hosts` 中添加了[一条记录](#recode)，打开文件删除即可
:::

<span id="2"></span>

## 二、别名登录

1. 在 `~/.ssh/config` 中加入以下配置，当然你可能没有 `config` 文件，甚至没有 `.ssh` 目录，那么请先行创建它。
```
Host aliyun                 # 必选 | aliyun 是自定义的标识，看心情定义就好了
HostName 120.130.140.150    # 必选 | 你的IP地址
Port 10089                  # 非必选 | 默认是ssh端口是22的可以不写，若是其他端口一定要写
User root                   # 必选 | 登录服务器的用户名
IdentitiesOnly yes          # 听我的写上
```

2. 输入 `ssh aliyun` 后回车，再输入密码，如下。
```
$ ssh aliyun
```

::: tip 拓展：Linux 命令
1. 进入目标目录: `cd ~`
2. 创建目录: `mkdir .ssh`
3. 创建文件: `touch config` 或者 `vim config`（不了解`vim`就直接用`touch`或`手动创建`）
:::

<span id="3"></span>

## 三、SSH登录

### 3.1 创建 SSH Keys

#### 1. 关于 SSH
使用 `SSH` 协议，可以连接和验证远程服务器和服务，也可以在每次访问时无需提供用户名或密码即可连接到 `GitHub`。

#### 2. 检查是否已经存在 SSH Keys
```
$ ls -al ~/.ssh
```
列出 `~/.ssh` 里所有的文件，如果存在下列文件中的一个就是说明已经存在，可以直接跳到 `3.2`，如果没有则进行下一步。
```
# id_dsa.pub
# id_ecdsa.pub
# id_ed25519.pub
# id_rsa.pub
```

#### 3. 生成 `SSH Keys`
最简单的方法是，输入`ssh-keygen -t rsa`命令后直接按两次回车，就会在`~/.ssh`目录下生成`id_rsa` 和 `id_rsa.pub`。
```
$ ssh-keygen -t rsa
```

::: tip 拓展：非对称密钥
1. 私钥: `id_rsa`
2. 公钥: `id_rsa.pub`
:::

### 3.2 将公钥添加到服务器
```
$ cat ~/.ssh/id_rsa.pub | ssh root@120.130.140.150 "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >>  ~/.ssh/authorized_keys"
```
其中 `ssh root@120.130.140.150` 就是`IP登录`中的命令, 可以使用[别名登录](#2)中的命令代替，随后输入密码。完成这一步之后，以后再登录该服务器只需要用[别名登录](#2)且不需要密码了，即只需要输入`ssh aliyun`就可以直接登录了。

<span id="recode"></span>

这个操作会在 `~/.ssh/known_hosts` 中添加一条类似如下的记录。
```
[120.130.140.150]:10089 ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEA0AYjUzafLeFPV86ol9oOg5p54A+mRZCZyxFtlXwLST7cLNb/NqtaAIEXAosnODU942fcIWdC8Oi66XZVEecMyoGh4n26UPnZfHprd4LdJrASsjCupiBd4akWN8XBwUWrwx+mKWjev3cH0QtUM85c6ZiP5+2od/qKLM+DamokImgotY0llAwrQnMjMagKTiWPE6ctontpfw2SPpTC5rOMNozd5sMwFZuxschk7hwxvinZp54tMFB6Ctxp49/thgbVGOYLQtXqZfXxDUIg3N6RM9eV1QltXNG5uSmXw1q5WO7g9NEMpstutNooOujQr0gF1dsDKTwKN+7lPdgsoqTu2w==
```

::: tip
如果有GitHub账户，建议直接根据 [Connecting to GitHub with SSH](https://help.github.com/en/articles/connecting-to-github-with-ssh) 生成 `SSH Keys` 。
:::

## 参考文献
1. [How To Set Up SSH Keys](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2)
2. [Connecting to GitHub with SSH](https://help.github.com/en/articles/connecting-to-github-with-ssh)