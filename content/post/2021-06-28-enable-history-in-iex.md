---
title: Enable history in iex
date: 2021-06-28
description: "It\'s pretty handy to get shell history in iex."
images:
- images/featured/enable-history-in-iex.png
tags:
  - IEx
  - Elixir
---

After spending some time in `iex`, you want to run a past command, especially from previous sessions.
Being used to `irb` or `ipython`, you are used to hitting the up arrow.
But, and nothing happens...

![](https://media.giphy.com/media/oziNormWuA6JrnbzY8/giphy.gif)

Well, that is not enabled by default.
For that to work, add the following to your `.zshrc` or `.bashrc`

```sh
export ERL_AFLAGS="-kernel shell_history enabled"
```

And since the default history size is `512KB`, let's increase it.

```sh
export ERL_AFLAGS="-kernel shell_history enabled -kernel shell_history_file_bytes 1024000"
```
