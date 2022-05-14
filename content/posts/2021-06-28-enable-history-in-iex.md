---
title: Enable history in iex
date: 2021-06-28
description: "How to enable shell history in IEx REPL sessions and increase history size"
image: images/posts/enable-history-in-iex.png
images:
  - images/posts/enable-history-in-iex.png
tags:
  - IEx
  - Elixir
---

{{< alert "secondary" >}}
How to enable shell history in IEx REPL sessions and increase history size
{{< /alert >}}

After spending some time in `iex`, you want to run a past command, especially from previous sessions.
Being used to `irb` or `ipython`, you hit the up arrow.

But, and nothing happens...

![not working](https://media.giphy.com/media/oziNormWuA6JrnbzY8/giphy.gif)

Well, that is not enabled by default.

For that to work, add the following to your `.zshrc` or `.bashrc`

```sh
export ERL_AFLAGS="-kernel shell_history enabled"
```

And since the default history size is `512KB`, let's increase it.

```sh
export ERL_AFLAGS="-kernel shell_history enabled -kernel shell_history_file_bytes 1024000"
```
