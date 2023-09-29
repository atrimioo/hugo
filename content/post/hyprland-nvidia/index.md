---
title: ArchLinux安装Hyprland+Nvidia
date: 2023-09-28
description: 在Intel+Nvidia的环境下让hyprland使用独显运行
image: hyprland.png
tags: 
    - Archlinux
    - Hyprland
    - Nvidia
    - Window Manager
categories:
    - Nvidia
    - Linux
---

## 安装驱动
### 使用开源的nvidia-open安装
1. `sudo pacman -S nvidia-open-dkms linux-headers nvidia-open`
2. 查看是否有显卡信息`nvidia-smi`
3. 加入内核模块
```
modprobe nvidia NVreg_OpenRmEnableUnsupportedGpus=1
modprobe nvidia_drm modeset=1
modprobe nvidia nvidia_drm.modeset=1 # 可以不加
modprobe nvidia NVreg_RegistryDwords="PowerMizerEnable=0x1; PerfLevelSrc=0x2222; PowerMizerLevel=0x3; PowerMizerDefault=0x3; PowerMizerDefaultAC=0x3"
```
4. 重新加载`mkinitcpio`检查模块是否存在
## 安装Seatd 和 Polkit
1. `sudo pacman -S polkit`
2. 添加用户组`sudo usermod -a -G seat username`
3. 设置Seatd开机启动`sudo systemctl enable seatd`
## 添加环境变量
在`nvim ~/.bash_profile`添加变量
```
export WLR_DRM_DEVICES=/dev/dri/card1:/dev/dri/card0  # 只启动nvidia输出
export LIBVA_DRIVER_NAME=nvidia
export XDG_SESSION_TYPE=wayland
#export GBM_BACKEND=nvidia-drm
export __GLX_VENDOR_LIBRARY_NAME=nvidia
export WLR_NO_HARDWARE_CURSORS=1
export MOZ_ENABLE_WAYLAND=1 # (REQUIRED: Firefox will break ot>
#export WLR_RENDERER=vulkan (WARNING: This breaks for me.)
exec Hyprland
```
## 安装Hyprland
```
sudo pacman -S hyprland-nvidia-git  wlroots-nvidia xdg-desktop-portal-hyprland-git xorg-xwayland 

```
## 配置显示器
`monitor=DP-1,2560x1440@144.001007,0x0,1`
## 部分软件依赖
```
sudo pacman -S qt5-wayland qt5ct libva
paru -S libva-nvidia-driver-git
```
## HyprLand完整配置
```   
      monitor=,preferred,auto,auto
	  monitor=DP-1,2560x1440@144.001007,0x0,1
	  monitor=HDMI-A-1,1920x1080@60,0x0,1
	  monitor=eDP-1,disable
	  
	  exec-once = swww query || swww init && swww img $HOME/image/bg-7.png
	  exec-once = waybar & mako & fcitx5 -d & clash-verge & nm-applet
	  exec-once = dbus-update-activation-environment --systemd WAYLAND_DISPLAY XDG_CURRENT_DESKTOP
	  exec-once = wl-paste --type text --watch cliphist store #Stores only text data
	  
	  env = XCURSOR_SIZE,24
	  
	  input {
	      kb_layout = us
	      kb_variant =
	      kb_model =
	      kb_options =
	      kb_rules =
	      follow_mouse = 1
	      touchpad {
	          natural_scroll = no
	      }
	      sensitivity = 0 # -1.0 - 1.0, 0 means no modification.
	  }
	  
	  general {
	      gaps_in = 5
	      gaps_out = 8
	      border_size = 0
	      col.active_border = rgb(5C5470) rgb(B9B4C7)
	      col.inactive_border = 0xff45475a
	      apply_sens_to_raw = false
	      col.group_border = 0xff89dceb
	      col.group_border_active = 0xfff9e2af    
	      layout = dwindle
	  }
	  
	  decoration {
	      rounding = 10
	      multisample_edges = true
	      drop_shadow = yes
	      shadow_range = 10
	      shadow_render_power = 3
	      col.shadow = rgba(1a1a1aee)
	      blur {
	          enabled = true
	          new_optimizations =true
	          size = 4
	          passes = 2
	          ignore_opacity = true
	      }
	  }
	  
	  misc {
	      disable_hyprland_logo = yes
	      #disable_splash_rendering = true
	      #no_direct_scanout = true
	      no_direct_scanout = true
	  }
	  
	  animations {
	      enabled=1
	      # bezier=overshot,0.05,0.9,0.1,1.1
	      bezier=overshot,0.13,0.99,0.29,1.1
	      animation=windows, 1, 4, overshot
	      animation=windowsOut,1, 7, overshot, popin 80%
	      animation=border, 1, 10, default
	      animation=fade, 1, 1, default
	      animation=workspaces,1,4,overshot,slidevert
	  }
	  
	  
	  
	  dwindle {
	      pseudotile = yes 
	      preserve_split = yes
	      no_gaps_when_only = 1 
	  }
	  
	  master {
	      new_is_master = true
	  }
	  
	  gestures {
	      workspace_swipe = off
	  }
	  
	  device:epic-mouse-v1 {
	      sensitivity = -0.5
	  }
	  
	  windowrule = float,^(pavucontrol)$
	  windowrule = float, title:^(btop)$
	  windowrule = maximize,title:^(nsxiv)$
	  
	  windowrule = float,^(thunar)$
	  
	  windowrulev2 = size 1200 800,class:^(thunar)$
	  windowrulev2 = center,class:^(thunar)$
	  windowrulev2 = animation popin 80%,class:^(thunar)$
	  windowrulev2 = opacity 0.95 0.95,class:^(thunar)$
	  
	  windowrulev2 = center,class:^(wofi)$
	  windowrulev2 = opacity 0.95 0.95,class:^(wofi)$
	  windowrulev2 = bordersize 2,class:^(wofi)$
	  
	  windowrule = float,title:^(fly_is_kitty)$
	  windowrule = move 30%,title:(fly_is_kitty)$
	  windowrule = size 1000 600,title:^(fly_is_kitty)$
	  windowrule = bordersize 2,title:^(fly_is_kitty)$
	  
	  #windowrule = tile,:^(kitty)$
	  windowrulev2 = opacity 0.8 0.8,class:^(kitty)$
	  
	  windowrule = animation slide,title:^(all_is_kitty)$
	  windowrule = float,title:^(all_is_kitty)$
	  
	  windowrule = float,title:^(lf)$
	  windowrule = size 1200 800,title:^(lf)$
	  windowrule = center,title:(lf)$
	  windowrule = bordersize 2,title:^(lf)$
	  
	  $mainMod = SUPER
	  bind = $mainMod, Q, exec, kitty  #open the terminal
	  bind = $mainMod, W, killactive, # close the active window
	  bind = $mainMod, M, exec, wlogout --protocol layer-shell # show the logout window
	  bind = $mainMod SHIFT, M, exit, # Exit Hyprland all together no (force quit Hyprland)
	  bind = $mainMod, E, exec, thunar # Show the graphical file browser
	  bind = $mainMod, V, togglefloating, # Allow a window to float
	  bind = $mainMod, SPACE, exec, wofi # Show the graphical app launcher
	  bind = $mainMod, P, pseudo, # dwindle
	  bind = $mainMod, J, togglesplit, # dwindle
	  bind = $mainMod, B, exec, killall waybar || waybar
	  bind = $mainMod, D, exec, kitty lf
	  bind = $mainMod, S, exec, fish -c 'hyprshot -m region'
	  bind = $mainMod, A, exec, kitty --title fly_is_kitty
	  bind = $mainMod, F, fullscreen
	  bind = $mainMod, L, exec, logseq
	  bind = $mainMod, X, exec, cliphist list | wofi --dmenu | cliphist decode | wl-copy
	  
	  # Move focus with mainMod + arrow keys
	  bind = $mainMod, left, movefocus, l
	  bind = $mainMod, right, movefocus, r
	  bind = $mainMod, up, movefocus, u
	  bind = $mainMod, down, movefocus, d
	  
	  # Switch workspaces with mainMod + [0-9]
	  bind = $mainMod, 1, workspace, 1
	  bind = $mainMod, 2, workspace, 2
	  bind = $mainMod, 3, workspace, 3
	  bind = $mainMod, 4, workspace, 4
	  bind = $mainMod, 5, workspace, 5
	  bind = $mainMod, 6, workspace, 6
	  bind = $mainMod, 7, workspace, 7
	  bind = $mainMod, 8, workspace, 8
	  bind = $mainMod, 9, workspace, 9
	  bind = $mainMod, 0, workspace, 10
	  
	  # Resize windows
	  bind = $mainMod SHIFT, left, resizeactive, -100 0
	  bind = $mainMod SHIFT, right, resizeactive, 100 0
	  bind = $mainMod SHIFT, up, resizeactive, 0 -100
	  bind = $mainMod SHIFT, down, resizeactive, 0 100
	  
	  # Move windows
	  bind = $mainMod ALT, left, movewindow, l
	  bind = $mainMod ALT, right, movewindow, r
	  bind = $mainMod ALT, up, movewindow, u
	  bind = $mainMod ALT, down, movewindow, d
	  
	  # Move active window to a workspace with mainMod + SHIFT + [0-9]
	  bind = $mainMod SHIFT, 1, movetoworkspace, 1
	  bind = $mainMod SHIFT, 2, movetoworkspace, 2
	  bind = $mainMod SHIFT, 3, movetoworkspace, 3
	  bind = $mainMod SHIFT, 4,movetoworkspace, 4
	  bind = $mainMod SHIFT, 5, movetoworkspace, 5
	  bind = $mainMod SHIFT, 6, movetoworkspace, 6
	  bind = $mainMod SHIFT, 7, movetoworkspace, 7
	  bind = $mainMod SHIFT, 8, movetoworkspace, 8
	  bind = $mainMod SHIFT, 9, movetoworkspace, 9
	  bind = $mainMod SHIFT, 0, movetoworkspace, 10
	  
	  # Scroll through existing workspaces with mainMod + scroll
	  bind = $mainMod, mouse_down, workspace, e+1
      bind = $mainMod, mouse_up, workspace, e-1
	  
	  # Move/resize windows with mainMod + LMB/RMB and dragging
	  bindm = $mainMod, mouse:272, movewindow
	  bindm = $mainMod, mouse:273, resizewindow
	```

