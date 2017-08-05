---
title: Vagrant Cheat Sheet
type: notes
category: devtools
tags: [Vagrant]
order: 3
---

作用 | 命令
---  | ---
添加box | `vagrant box add base <url或者本地文件名>`
显示已添加box列表 | `vagrant box list`
初始化 | `vagrant init <name>`
启动虚拟机 | `vagrant up`
虚拟机状态 | `vagrant status`
连接到虚拟机 | `vagrant ssh`
输出用于ssh连接的信息 | `vagrant ssh-config`
修改配置后加载配置 | `vagrant reload`
挂起 | `vagrant suspend`
恢复前面被挂起的状态 | `vagrant resume`
关机 | `vagrant halt`
打包环境 | `vagrant package`
删除box | `vagrant box remove <box-name>`
停止运行虚拟机并销毁所有资源 | `vagrant destroy`

