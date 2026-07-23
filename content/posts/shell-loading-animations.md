---
title: "Shell loading animations"
date: 2022-09-22
tags: ["shell"]
draft: false
---

## Why

I've recently been writing some shell scripts to automate and distribute actions that used to be done manually (I'll write a post about that later). For now, I'd like to share how I built a 100% shell-based loader library to use in my scripts.

I looked around for an existing library and found a few interesting GitHub projects, but none of them felt ready to use as an actual library. Most were also limited to old, basic ASCII templates, and I wanted to try building something closer to modern loaders and spinners in shell.

So I ended up creating [`shloader`](https://github.com/kaderovski/shloader). (Sorry, naming things has never been my strong suit.)

## Features

shloader comes with a few nice features:

- emoji support
- loader support
- dynamic message on load step
- message on step ending
- multiple loading templates
- light and easy to use on existing scripts

## Templating

First, we need templates, one for each loader that will be displayed. I decided to use shell arrays so we can iterate over them later in custom functions.

Here's what I ended up with:

```
# EMOJIS
emoji_hour=( 0.08 '🕛' '🕐' '🕑' '🕒' '🕓' '🕔' '🕕' '🕖' '🕗' '🕘' '🕙' '🕚')
emoji_face=( 0.08 '😐' '😀' '😍' '🙄' '😒' '😨' '😡')
emoji_earth=( 0.1 🌍 🌎 🌏 )
emoji_moon=( 0.08 🌑 🌒 🌓 🌔 🌕 🌖 🌗 🌘 )
emoji_orange_pulse=( 0.1 🔸 🔶 🟠 🟠 🔶 )
emoji_blue_pulse=( 0.1 🔹 🔷 🔵 🔵 🔷 )
emoji_blink=( 0.06 😐 😐 😐 😐 😐 😐 😐 😐 😐 😑 )
emoji_camera=( 0.05 📷 📷 📷 📷 📷 📷 📷 📷 📷 📷 📷 📷 📷 📷 📷 📷 📷 📷 📷 📷 📸 📷 📸 )
emoji_sick=( 0.2 🤢 🤢 🤮 )
emoji_monkey=( 0.2 🙉 🙈 🙊 🙈 )
emoji_bomb=( 0.2 '💣   ' ' 💣  ' '  💣 ' '   💣' '   💣' '   💣' '   💣' '   💣' '   💥' '    ' '    ' )
# ASCII
ball=( 0.2 '(●)' '(⚬)')
arrow=( 0.06 '↑' '↗' '→' '↘' '↓' '↙' '←' '↖')
cym=( 0.1 '⊏' '⊓' '⊐' '⊔')
x_plus=( 0.08 '×' '+')
line=( 0.08 '☰' '☱' '☳' '☷' '☶' '☴')
ball_wave=( 0.1 '𓃉𓃉𓃉' '𓃉𓃉∘' '𓃉∘°' '∘°∘' '°∘𓃉' '∘𓃉𓃉')
old=( 0.07 '—' "\\" '|' '/' )
dots=( 0.04 '⣾' '⣽' '⣻' '⢿' '⡿' '⣟' '⣯' '⣷' )
dots2=( 0.04 '⠋' '⠙' '⠹' '⠸' '⠼' '⠴' '⠦' '⠧' '⠇' '⠏' )
dots3=( 0.04 '⠋' '⠙' '⠚' '⠞' '⠖' '⠦' '⠴' '⠲' '⠳' '⠓' )
dots4=( 0.04 '⠄' '⠆' '⠇' '⠋' '⠙' '⠸' '⠰' '⠠' '⠰' '⠸' '⠙' '⠋' '⠇' '⠆' )
dots5=( 0.04 '⠋' '⠙' '⠚' '⠒' '⠂' '⠂' '⠒' '⠲' '⠴' '⠦' '⠖' '⠒' '⠐' '⠐' '⠒' '⠓' '⠋' )
dots6=( 0.04 '⠁' '⠉' '⠙' '⠚' '⠒' '⠂' '⠂' '⠒' '⠲' '⠴' '⠤' '⠄' '⠄' '⠤' '⠴' '⠲' '⠒' '⠂' '⠂' '⠒' '⠚' '⠙' '⠉' '⠁' )
dots7=( 0.04 '⠈' '⠉' '⠋' '⠓' '⠒' '⠐' '⠐' '⠒' '⠖' '⠦' '⠤' '⠠' '⠠' '⠤' '⠦' '⠖' '⠒' '⠐' '⠐' '⠒' '⠓' '⠋' '⠉' '⠈' )
dots8=( 0.04 '⠁' '⠁' '⠉' '⠙' '⠚' '⠒' '⠂' '⠂' '⠒' '⠲' '⠴' '⠤' '⠄' '⠄' '⠤' '⠠' '⠠' '⠤' '⠦' '⠖' '⠒' '⠐' '⠐' '⠒' '⠓' '⠋' '⠉' '⠈' '⠈' )
dots9=( 0.04  '⢹' '⢺' '⢼' '⣸' '⣇' '⡧' '⡗' '⡏' )
dots10=( 0.04  '⢄' '⢂' '⢁' '⡁' '⡈' '⡐' '⡠' )
dots11=( 0.04 '⠁' '⠂' '⠄' '⡀' '⢀' '⠠' '⠐' '⠈' )
```

Each array is split into two parts: the first element is a `time` interval in seconds, and the rest are the loader's frame transitions.

## Usage

I wanted the library to behave like a regular command, with arguments passed through options.

First, let's write the default `usage` function to display to the user.

```
usage() {
  cat <<EOF
Available options:
-h, --help            <OPTIONAL>    Print this help and exit
-l, --loader          <OPTIONAL>    Chose loader to display
-m, --message         <OPTIONAL>    Text to display while loading
-e, --ending          <OPTIONAL>    Text to display when finishing
EOF
  exit 0
}
```

Here's how I defined the argument options:

### Display help

```
# e.g
shloader -h
```

Parameter : `-h --help`  
Type : Optional  
Description : Show help usage

### Choose loader

```
# e.g
shloader -l arrow
```

Parameter : `-l --loader`  
Type : Optional  
Description : Choose which loader to display

### Display info on loading

```
# e.g
shloader -m "my loading message"
```

Parameter : `-m --message`  
Type : Optional  
Description : Show a text message while displaying loader

### Display info on ending

```
# e.g
shloader -e "\u2728 all done"
```

Parameter : `-e --end`  
Type : Optional  
Description : Show an end text message when loader ends

## Parsing Arguments

To handle user input, we need to parse the options passed in, so the library can run in a modular way.

```
shloader() {
  loader=''
  message=''
  ending=''
  while :; do
    case "${1-}" in
    -h | --help) usage;;
    -l | --loader)
      loader="${2-}"
      shift
      ;;
    -m | --message)
      message="${2-}"
      shift
      ;;
    -e | --ending)
      ending="${2-}"
      shift
      ;;
    -?*) die "Unknown option: $1" ;;
    *) break ;;
    esac
    shift
  done
[…]
}
```

You'll notice I placed this directly inside the main function, so it's the first thing that runs.

If the user doesn't specify a loader, we fall back to a default, let's say the `dots` one.

```
# shloader func
[…]
  if [[ -z "${loader}" ]] ; then
    loader=dots[@] 
  else
    loader=$loader[@]
  fi
[…]
```

One last thing: I use a small custom `die` function to exit cleanly if an option is unknown.

```
die() {
  local code=${2-1}
  exit "$code"
}
```

## Shell Configurations

Let's set up some quick configuration at the top of the script, right after the shebang.

```
#!/bin/bash
# https://github.com/Kaderovski/shloader
# me@kaderovski.com
set -Eeuo pipefail
trap end_shloader SIGINT SIGTERM ERR EXIT RETURN
tput civis
```

First, `set -Eeuo pipefail` makes sure execution stops if any command in a pipeline fails.

Then, `trap end_shloader SIGINT SIGTERM ERR EXIT RETURN` lets us call a custom function to clean up on exit, whether that's a normal stop, an interrupt, or an error.

Finally, `tput civis` hides the cursor.

## Trap Error and Exit

The trap calls a custom function, but how does that actually work?

```
end_shloader() {
  kill "${shloader_pid}" &>/dev/null
  tput cnorm
  if [[ "${ending}" ]]; then
    printf "\r${ending}"; echo
  fi
}
```

Here I make sure to kill the loader's `PID` first, then restore the cursor with `tput cnorm`.

## Display Loader

Now we can interact with shell output to actually generate the loading animation.

Let's create a new function, `play_shloader()`:

```
play_shloader() {
  while true ; do
    for frame in "${loader[@]}" ; do
      printf "\r%s" "${frame} ${message}"
      sleep "${speed}"
    done
  done
}
```

Here we assume `loader` is an array we can display, with each frame shown for the duration set in the `speed` variable (more on that below).

## Call Loader

Now what? We have all the building blocks for great loaders, let's put them to work together.

```
# shloader function
[…]
  loader=( ${!loader} )
  speed="${loader[0]}"
  unset "loader[0]"
  tput civis
  play_shloader &
  shloader_pid="${!}"
[…]
```

What's happening here?

We first read the `loader` array's content and save it into the `loader` variable. Since we know the first element is always the time duration, we save it separately in the `speed` variable.

Now that the array is split, we can remove the time duration since it's already stored in `speed`. We then hide the cursor, call `play_shloader`, and save its `PID`.

You can find the [Full Library Code](https://github.com/Kaderovski/shloader/blob/main/lib/shloader.sh) here.

## Script Library Integration

Nothing complicated here.

```
source ./lib/shloader.sh
shloader -l emoji_hour -m "Testing" -e "✨ All good !" 
  sleep 2   # remove it in your code
  # … your logic goes here
end_shloader
```

With a bit more detail:

```
#!/bin/bash
# if you want to try just add this block code in your code 
source ./lib/shloader.sh
# you can chose (see more in lib/loader.sh):
# ball, arrow, cym, x_plus, line, ball_wave, npm and old.
# you can specify message to display during loading
# and message to display after your code finished
# eg with npm style
# notice end message -e use unicode emoji to display
# this is for better terminal support
# \u2728 == ✨ but you can use emoji if your settings support it !
shloader -l emoji_face -m "Testing" -e "\u2728 All good !" 
  sleep 2   # remove it in your code
  # … your logic goes here
  # if you want to hide some output from loader
  # don't forget to redirect your STD*
  #
  # eg :
  # STDOUT
  # my_cmd 1> /dev/null
  # STDERR
  # my_cmd 2> /dev/null
  # BOTH
  # my_cmd &> /dev/null
# stop loader
end_shloader
```

## Conclusion

Like most useless things, it may just end up indispensable.

