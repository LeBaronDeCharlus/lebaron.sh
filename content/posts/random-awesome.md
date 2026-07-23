---
title: "Random wallpaper with AwesomeWM !"
date: 2020-01-10
tags: ["linux", "shell"]
draft: false
---

I wanted to shake up my routine a bit while working on my laptop, so I figured automatically changing my wallpaper at random would be a good way to break the monotony.

I use awesomeWM (version 4):

```
[f00b@void ~]$ awesome --version
awesome v4.3 (Too long)
 • Compiled against Lua 5.3.5 (running with Lua 5.3)
 • D-Bus support: ✔
 • execinfo support: ✘
 • xcb-randr version: 1.6
 • LGI version: 0.9.2
```

So I needed three things:

- A folder full of images
- A little script that will choose one at random
- A call to this script from awesome init

For the images, I use [the excellent repo](https://github.com/LukeSmithxyz/wallpapers) by **Luke Smith**.

The script itself isn't too complicated:

```bash
#!/bin/bash
# author : lebarondecharlus
# descr : Make your wallpaper change on each start !
#
# I'm using Luke Smith wallpaper git repo for all images
# link : https://github.com/LukeSmithxyz/wallpapers

# current awesome theme
THEME="powerarrow-dark"
# Awesome conf path
AWPATH="$HOME/.config/awesome/themes/$THEME"
# image should have absolute path to image folder
IMAGE=$(find $HOME/Pictures/wallpapers/ -type f -name "*.png" -o -name "*.jpeg" -o -name "*.jpg"| shuf -n 1 | sed 's/\ /\\ /g')

cp -f $IMAGE $AWPATH/wall.png

# don't forget to add those lines at the end of your rc.lua (replace with your correct path and script name)
#
# -- Startup programs
# awful.util.spawn_with_shell("~/bin/wallpaper.sh")
```

As noted in the comment, add these two lines (or just the last one, if you skip the comment) to call the script from awesomeWM's init.

Put this at the end of `~/.config/awesome/rc.lua`.

```lua
-- Startup programs
awful.util.spawn_with_shell("~/bin/wallpaper.sh")
```

Now, every time you restart awesomeWM, you'll get a new (hopefully pretty) wallpaper.
