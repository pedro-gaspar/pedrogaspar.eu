---
title: Elixir, the documentary
date: 2021-07-01
description: "Learn the origins of the Elixir programming language..."
image: images/posts/elixir-the-documentary.png
images:
  - images/posts/elixir-the-documentary.png
tags:
  - Elixir
---

{{< youtube lxYFOM3UJzo >}}

Watched the documentary and stayed even more pumped up about the language.🤗

In this short video, you get to know the origins of the Elixir programming language from the voice of its creator, the one and only José Valim.
He explains how it handles concurrency and the speed with which it had grown since its creation back in 2011.

He noticed that Erlang was the perfect fit for concurrency issues and the challenges we face in modern web apps. But even though the Erlang virtual machine is a fantastic piece of technology, it was missing some stuff that we tried to fix by developing the Elixir programming language on top of it.

Testimonial from _Qixxit_:

> Of course, we had some hiccups with bugs in the code. Still, we never had outages in terms of the infrastructure, which is an excellent indication that we are using the right technology because I've never experienced this before.

It's just amazing that he developed and built a language like Elixir with a great ecosystem and a warm and welcoming community. Most other recent languages like Go or Rust, to achieve the same, had a big corporation like Google or Mozilla to support its growth.

José Valim:

> One of the big things of Elixir that we get from building on top of the Erlang VM is that it allows us to write distributed software. Software that runs in more than one machine.

On one machine (_bob_):

```sh
> iex --name bob --cookie "shared-secret"
```

And on another machine (_alice_):

```sh
> iex --name alice --cookie "shared-secret"
```

If we have a module in the _bob_ machine.

```elixir
defmodule Hello do
  def world do
    IO.puts("Hello world")
  end
end
```

Then open a shell and run this amazing code.

```sh
➜  ~ iex --name bob --cookie "shared-secret"
Erlang/OTP 24 [erts-12.0.2] [source] [64-bit] [smp:12:12] [ds:12:12:10] [async-threads:1] [jit]

Interactive Elixir (1.12.1) - press Ctrl+C to exit (type h() ENTER for help)
iex(bob@bob.localdomain)1> defmodule Hello do
...(bob@bob.localdomain)1>   def world do
...(bob@bob.localdomain)1>     IO.puts("Hello world")
...(bob@bob.localdomain)1>   end
...(bob@bob.localdomain)1> end
{:module, Hello,
 <<70, 79, 82, 49, 0, 0, 5, 4, 66, 69, 65, 77, 65, 116, 85, 56, 0, 0, 0, 152, 0,
   0, 0, 16, 12, 69, 108, 105, 120, 105, 114, 46, 72, 101, 108, 108, 111, 8, 95,
   95, 105, 110, 102, 111, 95, 95, 10, ...>>, {:world, 0}}
iex(bob@bob.localdomain)2> Hello.world()
Hello world
:ok
```

It of course displays "Hello word". Going back to _alice_ machine:

```sh
➜  ~ iex --name alice --cookie "shared-secret"
Erlang/OTP 24 [erts-12.0.2] [source] [64-bit] [smp:24:24] [ds:24:24:10] [async-threads:1] [jit]

Interactive Elixir (1.12.1) - press Ctrl+C to exit (type h() ENTER for help)
iex(alice@alice.localdomain)1> Hello.world()
** (UndefinedFunctionError) function Hello.world/0 is undefined (module Hello is not available)
    Hello.world()
```

It doesn't work because it was defined on another computer (_bob_).
However, if we spawn a process running in _bob_:

```elixir
iex(alice@spiderman.localdomain)1> Node.spawn(:"bob@bob.localdomain", fn -> Hello.world() end)
Hello world
#PID<11982.130.0>
```

It works 🤯 !!!

Invoking from _alice_ computer executes the program in _bob_ node.

Had to check the same example with two home computers. 👌

With the Phoenix Web Framework, it has a Presence feature, which has a pub-sub mechanism in which you can send and receive messages from anybody connected from any machine. And you can know who is connected in the whole cluster (who is joining, who is leaving) without needing databases or third-party dependencies.

Development was always open from the get-go. And the interest started to grow and grow.

Justin Schneck, co-author of the Nerves Project:

> It has capabilities of IoT connectivity on the scale of millions of devices.

Chris McCord, the creator of the Phoenix Framework

> Phoenix is a web framework for the Elixir programming language, and it really is like a batteries-included web framework for the platform. It allowed an experiment to connect 2 million users to a single server. It enables a lot of innovation. 🔋🔋🔋

José Valim

> If I try to centralize and do everything on my own, I won't be able to do it. But if everybody can contribute a small part to this and with everybody together, we can do that and bring the community forward. Then we have a chance of actually making a lasting impact.

![clap](https://media.giphy.com/media/a0Lgc1JvbfS4o/giphy.gif)
