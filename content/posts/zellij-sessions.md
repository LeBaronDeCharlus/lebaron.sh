---
title: "Manage your Zellij sessions"
date: 2022-07-12
tags: ["shell"]
draft: false
---

![Zellij logo](/images/zellij.png)

Quick post about `Zellij`, a great tool I've been using for a few weeks as a `Tmux` replacement.

I kept missing a way to manage my `sessions` whenever I started a new `shell` or `terminal`, so I built a quick feature to handle it on startup.

![The session picker in action](/images/sessions.gif)

### Dependencies

You need [sk](https://github.com/lotabout/skim) binary installed and in your `$PATH` and of course [zellij](https://github.com/zellij-org/zellij/).

### Installation

Add this block at the end of your `$SHELLrc` file *(tested with BASH and ZSH)*:

```shell
ZJ_SESSIONS=$(zellij list-sessions)
NO_SESSIONS=$(echo "${ZJ_SESSIONS}" | wc -l)

if [ "{$ZELLIJ}" ] && [ -z "${ZELLIJ_SESSION_NAME}" ]; then
  echo -ne "Active Zellij sessions :\n"
  for i in $(echo "${ZJ_SESSIONS}"); do echo -ne "*${i}\n"; done
  echo -ne '\n'
  read REPLY\?"New zellij session [y/n] ? "
  if [ "${REPLY}" = "y" ]; then
    read SESS\?"Session name : "
    zellij --layout compact attach -c "${SESS}"
  else
    if [ "${NO_SESSIONS}" -ge 1 ]; then
      zellij --layout compact attach \
      "$(echo "${ZJ_SESSIONS}" | sk)"
    else
    fi
  fi
fi
```

### Contributing

Feel free to fork and contribute.

[Sources are here.](https://github.com/lebarondecharlus/zellij-sessions)
