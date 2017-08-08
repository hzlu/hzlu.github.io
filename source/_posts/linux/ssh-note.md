---
title: ssh 命令
category: linux
tags: [ssh]
---

## `ssh-kengen`

> `-b`：指定密钥长度；
> `-e`：读取openssh的私钥或者公钥文件；
> `-C`：添加注释；
> `-f`：指定用来保存密钥的文件名；
> `-i`：读取未加密的ssh-v2兼容的私钥/公钥文件，然后在标准输出设备上显示openssh兼容的私钥/公钥；
> `-l`：显示公钥文件的指纹数据；
> `-N`：提供一个新密语；
> `-P`：提供（旧）密语；
> `-q`：静默模式；
> `-t`：指定要创建的密钥类型。

```bash
ssh-kengen -t ras -C hzlu2010@gmail.com
```

## `ssh-add`

把ssh密钥添加到`ssh-agent`的高速缓存中

> `-D`：删除ssh-agent中的所有密钥
> `-d`：从ssh-agent中的删除密钥
> `-L`：显示ssh-agent中的公钥
> `-l`：显示ssh-agent中的密钥
> `-X`：对ssh-agent进行解锁
> `-x`：对ssh-agent进行加锁
> `-t life`：对加载的密钥设置超时时间，超时ssh-agent将自动卸载密钥

```bash
ssh-add ~/.ssh/id_rsa # 添加密钥到ssh-agent高速缓存中
ssh-add -d ~/.ssh/id_xxx.pub # 删除密钥
ssh-add -l # 查看ssh-agent中的密钥
```

## `ssh-copy-id`

把本机公钥复制到远程主机的`authorized_keys`文件上，并适当设置`.ssh`（0700）目录及`.ssh/authorized_keys`（0600）文件的权限

```bash
ssh-copy-id user@server
ssh-copy-id -i ~/.ssh/id_rsa.pub user@server
# -i 指定公钥文件
```


