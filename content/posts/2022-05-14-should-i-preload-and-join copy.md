---
title: "Draft: Should I preload and join?"
date: 2022-05-14
description: "Break in iex with unmatched quotes or parens without exiting"
image: images/posts/elixir-streams.png
images:
  - images/posts/elixir-streams.png
tags:
  - Elixir
  - Today I Learned
---

{{< alert "secondary" >}}
Break in iex with unmatched quotes or parens without exiting
{{< /alert >}}

![](https://media.giphy.com/media/Rk8CZk8M7UHzG/giphy.gif)

Sometimes, when you forget to close a quote, double quote, or brackets, it will continue to create new lines when you press enter.

You can exit by pressing `Ctrl+c` twice, but you will leave the interactive shell.

If you want to stop the statement, you are inserting just add `#iex:break`. It will error out, but you will still be in the shell.

```sh
iex(1)> IO.puts("dssdsda
...(1)> asdasdsda
...(1)> sadsadsda
...(1)> #iex:break
** (TokenMissingError) iex:1: incomplete expression
```
