---
title: 平铺窗口管理器 i3-wm
date: 2016-02-23 12:22:45
tags: Linux
---

> 公欲善其事，必先利其器。

[i3](http://i3wm.org/) 是一个 [X Window](http://www.x.org/) 下的一款 [平铺窗口管理器](https://en.wikipedia.org/wiki/Tiling_window_manager)。平铺是它的最大特点，它可以最大程度的利用屏幕上的每一个像素，让你可以眼观六路，耳听八方。你可以使用全键盘操作来轻松的控制你的窗口，极大的提高了工作的效率。当然，如果你的屏幕过小的话还是慎用。

{% asset_img screen-shot.png screen shot %}

官方的定位是高玩与开发者，但如果你想尝试一下平铺的魅力的话，也可以跟随本文体验一下（当然必须是 Linux 用户咯）。

# 安装
本文以 [Arch Linux](https://www.archlinux.org) 为例，其它发行版类似，可以参考官方的[安装文档](http://i3wm.org/downloads/)。

```bash
$ pacman -S i3-wm
```

安装结束后就可以使用 `i3` 命令来启动窗口管理器了。如果有桌面环境的话直接启动可能会报错，要先注销再在登陆管理器那里选择 i3 启动就可以了。如果没有登陆管理器，那么可以在 `.xinitrc` 中添加 `exec i3` 即可。

# 配置
初次使用，i3 会有一些提示，根据提示可以完成基本配置。i3 配置文件使用的是 Shell 脚本，所以也是很简单易懂。

i3 的配置文件是 `~/.i3/config`，如果没有此文件的话，我们可以从 `/etc/i3/config` 这 copy 一份。默认的配置里有许多的注释，我们可以自行调整。

这里我分享一份我自己使用的配置文件。

```bash
# 把修饰键设为 Win 键
set $mod Mod4

# 边框为 1px
new_window 1pixel

# 打开 Chromium 是自动放到 3 号工作区
assign [class="^chromium$"] 3: Chromium

# 打开 Atom 是自动放到 2 号工作区
assign [class="^Atom$"] 2: Atom

# 绑定快捷键 Win + p 打开 浏览器，注意这里要加上 --no-startup-id 以防止鼠标出现忙状态
bindsym $mod+p exec --no-startup-id chromium

# start a terminal
bindsym $mod+Return exec --no-startup-id i3-sensible-terminal

# kill focused window
bindsym $mod+Shift+q kill

# start dmenu (a program launcher)
bindsym $mod+d exec --no-startup-id dmenu_run

# change focus
bindsym $mod+Left focus left
bindsym $mod+Down focus down
bindsym $mod+Up focus up
bindsym $mod+Right focus right

# move focused window
bindsym $mod+Shift+Left move left
bindsym $mod+Shift+Down move down
bindsym $mod+Shift+Up move up
bindsym $mod+Shift+Right move right

# 横向分屏
bindsym $mod+h split h

# 纵向分屏
bindsym $mod+v split v

# 全屏切换
bindsym $mod+f fullscreen toggle

# focus the parent container
bindsym $mod+a focus parent

# 切换工作区
bindsym $mod+1 workspace 1: Terminal
bindsym $mod+2 workspace 2: Atom
bindsym $mod+3 workspace 3: Chromium
bindsym $mod+4 workspace 4: Other

# 把当前窗口移动到指定工作区
bindsym $mod+Shift+1 move container to workspace 1: Terminal
bindsym $mod+Shift+2 move container to workspace 2: Atom
bindsym $mod+Shift+3 move container to workspace 3: Chromium
bindsym $mod+Shift+4 move container to workspace 4: Other

# 重新载入配置文件  调试的时候比较好用
bindsym $mod+Shift+c reload

# 重启 i3
bindsym $mod+Shift+r restart

# exit i3 (logs you out of your X session)
bindsym $mod+Shift+e exec --no-startup-id "i3-nagbar -t warning -m 'You pressed the exit shortcut. Do you really want to exit i3? This will end your X session.' -b 'Yes, exit i3' 'i3-msg exit'"

# resize window (you can also use the mouse for that)
mode "resize" {
        bindsym Left resize shrink width 1 px or 1 ppt
        bindsym Down resize grow height 1 px or 1 ppt
        bindsym Up resize shrink height 1 px or 1 ppt
        bindsym Right resize grow width 1 px or 1 ppt

        # back to normal: Enter or Escape
        bindsym Return mode "default"
        bindsym Escape mode "default"
}

# 进入 resize 模式
bindsym $mod+r mode "resize"

# 把窗口放入便捷板
bindsym $mod+Shift+minus move scratchpad

# 打开便捷板
bindsym $mod+minus scratchpad show

# 切换平铺与浮动
bindsym $mod+t floating toggle

# 配置状态栏
bar {
  status_command ~/.i3/conky-i3bar
	position top
	font xft: Source Code Pro 8
	separator_symbol " | "
	colors {
		background #cccccc
		separator #ffff00
	}
}
```

这里只使用了平铺模式，而 i3 支持好几种窗口布局模式，具体可以参考官方的 [Guide](http://i3wm.org/docs/userguide.html)。这里的状态栏采用了 [conky](https://github.com/brndnmtthws/conky) 来输出系统信息。完整的配置文件（包括 conky 的配置）可以在[这里](https://github.com/fh250250/my-config/tree/master/i3)找到。

## 资源
- [dotshare](http://dotshare.it/) - 大量的配置文件，总有你的菜
- [i3 官方配置参考](http://i3wm.org/docs/userguide.html)
