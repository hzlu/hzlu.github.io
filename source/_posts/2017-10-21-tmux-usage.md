---
title: 使用Tmux
tags:
  - tmux
categories:
  - 命令行工具
date: 2017-10-21 15:06:59
---



## 基本概念

- 终端复用
- session 用来区分不同的工作环境
- window 一个session下能创建多个window
- pane 一个window下可以创建多个视图，切分窗口

底部状态栏上有tmux的session window 及 pane 信息，右侧有主机、时间日期信息，状态栏可定制。

## 基本命令

- 命令前缀 `Ctrl b` 用 `C - b` 表示

### pane 命令

- `C - b %` 创建垂直分割的pane
- `C - b "` 创建水平分割的pane
- `C - b o` 切换到下一个pane
- `C - b 方向箭头` 切换pane
- `C - b z` 最大化当前pane
- `C - b x` 关闭pane

### window 命令

- `C - b c` 新建window
- `C - b p` 切换到前一个window
- `C - b n` 切换到下一个window
- `C - b <数字>` 选择window 状态栏显示的window序号
- `tmux rename-window -t <name> <new-name>` 重命名window
- `C - b &` 关闭window

### session 命令

- `C - b d` detach session
- `C - b s` session 切换
- `tmux attach -t <session-name>` attach session
- `tmux rename-session -t <name> <new-name>` 重命名session
- `tmux new-session -s <session-name>` 直接创建命名session
- `tmux new-session -s <session-name> -d` 创建detach状态的命名session
- `tmux new-session -s <session-name> -n <window-name>` 创建命名session及命名window

### 其他命令

- `C - b [` 进入复制模式 空格开始选择 回车把选择内容复制到剪切板
- `C - b ]` 粘贴
- `C - b ?` 查看帮助

