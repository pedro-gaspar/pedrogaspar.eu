---
title: "Draft: Help inside IEx"
date: 2022-05-08
description: "Help inside IEx"
image: images/posts/elixir-streams.png
images:
  - images/posts/elixir-streams.png
tags:
  - Elixir
  - Today I Learned
---

Inside iex, we can also check the documentation for our module and function:

```sh
$ iex -S mix
Erlang/OTP 24 [erts-12.0.2] [source] [64-bit] [smp:12:12] [ds:12:12:10] [async-threads:1] [jit]

Compiling 1 file (.ex)
Interactive Elixir (1.12.1) - press Ctrl+C to exit (type h() ENTER for help)
iex(1)> h Hello

                      Hello

Documentation for Hello.

iex(2)> h Hello.world

                   def world()

Hello world.

## Examples

    iex> Hello.world()
    "Hello world!"
```
