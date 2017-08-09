---
title: apt 使用
categories:
- linux
tags:
- apt
---


```bash
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install redis-server
sudo apt-get install -y python-software-properties
sudo add-apt-repository -y ppa:rwky/redis
sudo apt-get update
apt-get remove redis-server
apt-get autoremove
sudo apt-get install -y redis-server
```

```bash
export LANGUAGE=en_US.UTF_8
export LANG=en_US.UTF_8
export LC_ALL=en_US.UTF_8
locale-gen en_US.UTF-8
sudo dpkg-reconfigure locales
locale
```
