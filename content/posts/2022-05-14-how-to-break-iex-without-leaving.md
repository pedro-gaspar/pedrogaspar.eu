---
title: "How to safely break in iex"
date: 2022-05-14
description: "Break from current statement in iex, with unmatched quotes or parentheses, without exiting shell"
image: images/posts/iex_break.png
images:
  - images/posts/iex_break.png
tags:
  - Elixir
  - Today I Learned
---

{{< alert "secondary" >}}
Break from current statement in iex, with unmatched quotes or parentheses, without exiting shell
{{< /alert >}}

![](https://media.giphy.com/media/Rk8CZk8M7UHzG/giphy.gif)

Sometimes, when you forget to close a quote, double quote, or brackets, it will continue to create new lines when you press enter.

You can exit by pressing `Ctrl+c` twice, but you will leave the interactive shell.

If you want to stop the statement, you are inserting, just add `#iex:break`. It will error out, but you will still be in the shell.

```sh
iex(1)> IO.puts("dssdsda
...(1)> asdasdsda
...(1)> sadsadsda
...(1)> #iex:break
** (TokenMissingError) iex:1: incomplete expression
```
