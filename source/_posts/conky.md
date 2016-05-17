---
title: 究极桌面监控应用 conky
date: 2016-02-24 09:18:23
tags:
  - Linux
---

[conky] 是一个 X Window 下的监控应用。他能监控系统的一系列状态信息，比如：uname, uptime, CPU 使用量, MEM 使用量, 网络状况等。还可以使用 [lua] 来扩展，配个 [Cairo] 可以绘制出任何你想要的图型。

废话不多说，先来张图直观的体验一下。

{% asset_img my-conky.png 我的 conky 配置 %}

以上是我的 [conky] 配置，有没有点科幻电影的感觉。

## 安装
以 ubuntu 为例：

```bash
$ sudo apt-get install conky-all
```

安装 conky-all 包会把 [lua] 扩展也安装上，这正是我们需要的，因为我们要用 [lua] 来会圆环。

## 配置
[conky] 的配置比较繁杂，但是常用的就那么几个。

```bash
alignment tr    # 放置在右上角 top-right
background yes    # 作为背景嵌入桌面
border_inner_margin 0    #
border_outer_margin 0    # 不使用边框
border_width 0           #
cpu_avg_samples 2    # cpu 采样率，一般为 2
default_bar_size 100 8
default_color cccccc
diskio_avg_samples 2
double_buffer yes    # 开启双缓冲，防止闪屏
draw_borders no
draw_graph_borders no
draw_outline no
draw_shades no
format_human_readable yes    # 使用 MB，GB 等单位
gap_x 50
gap_y 50
if_up_strictness address    # 网卡探测标准为 是否有地址
minimum_size 500 500
maximum_width 500
net_avg_samples 2    # 网卡采样率，一般为 2
no_buffers yes
override_utf8_locale yes
own_window yes
own_window_transparent yes
own_window_type override
short_units yes
update_interval 1    # 每秒刷新一次
use_xft yes    # 使用 xfont
xftfont Ubuntu Mono:size=10

# 加载 lua 脚本
lua_load ~/.conky/lines.lua ~/.conky/rings.lua ~/.conky/main.lua
# 绘制图形
lua_draw_hook_pre draw


# 以下为输出到桌面的内容
TEXT
#### 磁盘
${font Ubuntu Mono:size=20}DISK${goto 200}${fs_used_perc /}%$font
${voffset 10}\
Used${goto 200}${fs_used /}
Total${goto 200}${fs_size /}
IO${goto 200}${diskio /dev/sda}/s
${diskiograph /dev/sda 30,200}
${top_io name 1}${goto 120}${top_io pid 1}${goto 180}${top_io io_perc 1}%
${top_io name 2}${goto 120}${top_io pid 2}${goto 180}${top_io io_perc 2}%
${top_io name 3}${goto 120}${top_io pid 3}${goto 180}${top_io io_perc 3}%
${top_io name 4}${goto 120}${top_io pid 4}${goto 180}${top_io io_perc 4}%
${top_io name 5}${goto 120}${top_io pid 5}${goto 180}${top_io io_perc 5}%
#### 内存和 CPU
${voffset 20}\
${font Ubuntu Mono:size=20}RAM${goto 200}${memperc}%\
${goto 260}CPU${goto 450}${cpu}%$font
${voffset 10}\
Used${goto 200}${mem}\
${goto 260}Frequency${alignr}${freq_g}GHz
Total${goto 200}${memmax}\
${goto 260}Load${alignr}${loadavg}
Cache${goto 200}${cached}\
${goto 260}Temperature${alignr}${acpitemp}°C
Swap${goto 200}${swapperc}%\
${goto 260}Uptime${alignr}${uptime}

${top_mem name 1}${goto 120}${top_mem pid 1}${goto 180}${top_mem mem_res 1}\
${goto 260}${top name 1}${alignr}${top pid 1}${top cpu 1}%
${top_mem name 2}${goto 120}${top_mem pid 2}${goto 180}${top_mem mem_res 2}\
${goto 260}${top name 2}${alignr}${top pid 2}${top cpu 2}%
${top_mem name 3}${goto 120}${top_mem pid 3}${goto 180}${top_mem mem_res 3}\
${goto 260}${top name 3}${alignr}${top pid 3}${top cpu 3}%
${top_mem name 4}${goto 120}${top_mem pid 4}${goto 180}${top_mem mem_res 4}\
${goto 260}${top name 4}${alignr}${top pid 4}${top cpu 4}%
${top_mem name 5}${goto 120}${top_mem pid 5}${goto 180}${top_mem mem_res 5}\
${goto 260}${top name 5}${alignr}${top pid 5}${top cpu 5}%


# 网卡信息
${if_up wlan0}\
${font Ubuntu Mono:size=20}Wireless$font

${addr wlan0}${alignc}${wireless_essid wlan0}${alignr}${wireless_link_qual_perc wlan0}%
${upspeedgraph wlan0 30,350}${font Ubuntu Mono:size=20}${alignr}${upspeed wlan0}/s$font
${downspeedgraph wlan0 30,350}${font Ubuntu Mono:size=20}${alignr}${downspeed wlan0}/s$font
${endif}
${if_up eth0}\
${font Ubuntu Mono:size=20}Ethernet$font ${addr eth0}

${upspeedgraph eth0 30,350}${font Ubuntu Mono:size=20}${alignr}${upspeed eth0}/s$font
${downspeedgraph eth0 30,350}${font Ubuntu Mono:size=20}${alignr}${downspeed eth0}/s$font
${endif}
```

以上是 [conky] 的配置，但是要绘制图形还需要使用 [lua] 来做。

```lua
require('cairo')

function rgb_to_r_g_b(colour,alpha)
    return ((colour / 0x10000) % 0x100) / 255., ((colour / 0x100) % 0x100) / 255., (colour % 0x100) / 255., alpha
end

---------------------------------> draw lines
function draw_line(cr, line)

  local color, alpha, width, path = line['color'], line['alpha'], line['width'], line['path']

  cairo_set_source_rgba(cr,rgb_to_r_g_b(color, alpha))
  cairo_set_line_width(cr, width)

  for i in pairs(path) do
    if i == 1 then
      cairo_move_to(cr, path[i][1], path[i][2])
    else
      cairo_line_to(cr, path[i][1], path[i][2])
    end
  end

  cairo_stroke(cr)
end

---------------------------------> draw rings
function draw_ring(cr,t,pt)
    local w,h=conky_window.width,conky_window.height

    local xc,yc,ring_r,ring_w,sa,ea=pt['x'],pt['y'],pt['radius'],pt['thickness'],pt['start_angle'],pt['end_angle']
    local bgc, bga, fgc, fga=pt['bg_colour'], pt['bg_alpha'], pt['fg_colour'], pt['fg_alpha']

    local angle_0=sa*(2*math.pi/360)-math.pi/2
    local angle_f=ea*(2*math.pi/360)-math.pi/2
    local t_arc=t*(angle_f-angle_0)

-- Draw background ring

    cairo_arc(cr,xc,yc,ring_r,angle_0,angle_f)
    cairo_set_source_rgba(cr,rgb_to_r_g_b(bgc,bga))
    cairo_set_line_width(cr,ring_w)
    cairo_stroke(cr)

-- Draw indicator ring

    cairo_arc(cr,xc,yc,ring_r,angle_0,angle_0+t_arc)
    cairo_set_source_rgba(cr,rgb_to_r_g_b(fgc,fga))
    cairo_stroke(cr)
end

function conky_draw()
    local function setup_rings(cr,pt)
    local str=''
    local value=0

    str=string.format('${%s %s}',pt['name'],pt['arg'])
    str=conky_parse(str)

    value=tonumber(str)
    if not value then
        value=0
    end
    pct=value/pt['max']

    draw_ring(cr,pct,pt)
end


-- Check that Conky has been running for at least 5s

if conky_window==nil then return end
    local cs=cairo_xlib_surface_create(conky_window.display,conky_window.drawable,conky_window.visual, conky_window.width,conky_window.height)

    local cr=cairo_create(cs)

    local updates=conky_parse('${updates}')
    update_num=tonumber(updates)

    if update_num>5 then
        for i in pairs(rings_table) do
            setup_rings(cr,rings_table[i])
        end

        for i in pairs(lines_table) do
            draw_line(cr, lines_table[i])
        end
    end
end
```

画圆环的代码是修改了 [Minimal-Conky-Rings](https://github.com/James5979/Minimal-Conky-Rings)。

详细的代码可以在[这里](https://github.com/fh250250/my-config/tree/master/conky)找到。

## 资源
[Harmattan](https://github.com/zagortenay333/Harmattan) 是一套 [conky] 的主题。

最后来看一看大神们的 [conky] 是什么样的吧，来感受以下被支配的恐惧吧！

{% asset_img 大神.jpg 大神的桌面 %}





[conky]: https://github.com/brndnmtthws/conky
[lua]: http://www.lua.org/
[Cairo]: http://www.cairographics.org/
