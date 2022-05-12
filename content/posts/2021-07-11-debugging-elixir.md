---
title: Debugging Elixir
date: 2021-07-11
description: "Solving your own code commited crimes"
image: images/posts/debugging-elixir.png
images:
  - images/posts/debugging-elixir.png
tags:
  - Elixir
  - Debugging
  - VSCode
---

> Debugging is being the detective in crime movies where you are also the murderer.

![](https://media.giphy.com/media/3o6ZtksqDdL9K3u4rC/giphy.gif)

Detectives have tools to help them. They can be old-fashioned Sherlock Holmes and need a magnifying glass or go full CSI and have a computer that cross-checks fingerprints slowly showing one at a time on the tv screen. 🤦

You will indeed commit crimes 💩🐛 during development, so what tools does Elixir have to get your back?

## Old school _IO.puts_

![](https://media.giphy.com/media/3oEjHD20zWeDvGWoF2/giphy.gif)

There are many better tools for that, but the good old print will save your day many times.

```elixir
iex(3)> username = "º-º"
"º-º"
iex(4)> IO.puts("------>" <> username)
------>º-º
:ok
```

The problem with it is that we have to print a string or string-compatible format.

## Old school with steroids _IO.inspect_

![](https://media.giphy.com/media/W4NvANl5jdrji/giphy.gif)

Doing **IO.puts** is not ideal, especially if you want to print info about some complex data structure like a map or a tuple.

**IO.inspect** got your back. It will do the proper conversion to a string and display helpful information.

```elixir
iex(1)> conn = %{status: 504}
%{status: 504}
iex(2)> IO.inspect(conn)
%{status: 504}
%{status: 504}
```

But how to debug what happens during a pipeline?

```elixir
["some", "weird", "debugging"]
|> does_some_stuff()
|> does_more_stuff()
|> does_even_more_stuff()
```

Immutability is incredible; it allows you to know that an error is happening in one of the above functions. But which one?

You could do the following:

```elixir
stuff =
  ["some", "weird", "debugging"]
  |> does_some_stuff()

IO.inspect(stuff)

stuff
|> does_more_stuff()
|> does_even_more_stuff()
```

But `IO.inspect` is a perfect match for this because it returns it's argument unchanged so you can just do:

```elixir
["some", "weird", "debugging"]
|> does_some_stuff()
|> IO.inspect()
|> does_more_stuff()
|> does_even_more_stuff()
```

Awesome right? 😎

You can even add labels to help you during this kind of debugging:

```elixir
["some", "weird", "debugging"]
|> does_some_stuff()
|> IO.inspect(label: "suspect num 1")
|> does_more_stuff()
|> IO.inspect(label: "suspect num 2")
|> does_even_more_stuff()
```

## IEX.pry magnifying glass

![](https://media.giphy.com/media/l2JdZX7Fk0hqQGwzm/giphy.gif)

Sometimes, instead of printing every variable, you want to check some variables or see the results of applying a function given a specific state.

**IEX.pry** will help you with that. It will allow you to stop the application in a specific line in your code.

Consider the following example. There's a crime being committed, but we don't know where. 🙈🙉🙊

```elixir
defmodule Crime do
  def double_or_nothing(number) do
    number / 0 + 10
  end
end
```

To use **IEX.pry** you just need to add the following to the place you want to debug:

```elixir
require IEX; IEX.pry
```

So in this case is just doing the following:

```elixir
defmodule Crime do
  def double_or_nothing(number) do
    require IEx; IEx.pry
    number / 0 + 10
  end
end
```

Then you can start a mix session:

```elixir
$ iex -S mix

iex(1)> Crime.double_or_nothing(10)
Break reached: Crime.double_or_nothing/1 (iex:2)
pry(1)> i number
Term
  10
Data type
  Integer
Reference modules
  Integer
Implemented protocols
  IEx.Info, Inspect, List.Chars, String.Chars
pry(2)> number
10
pry(3)> number + 10
20
pry(4)> h # if you need help
...
pry(5)> respawn()
```

You finish by calling **respawn** to continue execution in a new shell session.

![](https://media.giphy.com/media/l2JedQnrjOz9X69Ms/giphy.gif)

This approach is great but has some cons 😐:

- You have to change the code you want to debug
- You can only inspect places where you have a pry. You can't step or add breakpoints like other debuggers you might have used.

If you want to have multiple debug steps, add `require IEx; IEx.pry` for each, and continue to the next breakpoint.

## IEx _break!_ it out

![](https://media.giphy.com/media/l2JdT10OBwXcl8g1O/giphy.gif)

Instead of needing to hardcode the **IEX.pry** breakpoints, you can just set a breakpoint before executing the code.

Using the same example:

```elixir
defmodule Crime do
  def double_or_nothing(number) do
    number / 0 + 10
  end
end
```

You can then:

```elixir
$ iex -S mix

iex(1)> break! Crime.double_or_nothing/1
1
iex(2)> Crime.double_or_nothing(20)
Break reached: Crime.double_or_nothing/1 (lib/crime.ex:2)

    1: defmodule Crime do
    2:   def double_or_nothing(number) do
    3:     number / 0 + 10
    4:   end
pry(1)> i number
Term
  20
Data type
  Integer
Reference modules
  Integer
Implemented protocols
  IEx.Info, Inspect, List.Chars, String.Chars
pry(2)> whereami
Location: lib/crime.ex:2

    1: defmodule Crime do
    2:   def double_or_nothing(number) do
    3:     number / 0 + 10
    4:   end

    (crime 0.1.0) Crime.double_or_nothing/1
pry(3)> respawn()
```

We didn't need to change any line of our code to set the breakpoint, which is a lot better. 👌 You can also specify in break the number of times it will break.

## Testing with _IEx.pry_

![](https://media.giphy.com/media/l0G18t2rt94wUI6yc/giphy.gif)

If you want to debug a test if you add **`require IEx; IEx.pry`** to it. When you run the tests with **`mix test`**, you will get the following error:

```elixir
Cannot pry #PID<0.123.0> at Crime.CrimeTest ...
Is an IEx shell running?
```

This is because you are not inside an interactive shell session. Simple enough, let's run tests in an interactive shell. For that run:

```elixir
iex -S mix test
```

## Avoiding timeouts ⏰

![](https://media.giphy.com/media/3o6MbjelZFV8tCVi5a/giphy.gif)

To avoid timeouts, run tests with the trace option:

```elixir
$ iex -S mix test --trace
```

## VSCode and ElixirLS

![](https://media.giphy.com/media/3o7TKw6X7zBf3IbiPm/giphy.gif)

Left the best for last. If you are a Visual Studio Code user, you are in a good place for debugging Elixir.

The first step is to install the **ElixirLS** extension.

![](/images/elixirls.png)

Then you need to add a configuration file to set up debug. Finally, click on _"Run and Debug"_.

![](/images/vscode-debug1.png)

After that, click on _"create a launch.json file"_. This will create a default configuration for Elixir.

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "mix_task",
      "name": "mix (Default task)",
      "request": "launch",
      "projectDir": "${workspaceRoot}"
    },
    {
      "type": "mix_task",
      "name": "mix test",
      "request": "launch",
      "task": "test",
      "taskArgs": ["--trace"],
      "startApps": true,
      "projectDir": "${workspaceRoot}",
      "requireFiles": ["test/**/test_helper.exs", "test/**/*_test.exs"]
    }
  ]
}
```

I like to add a _debug mix task_ to run some code I want to debug. For that, add the following to the _lauch.json_ file.

```json
    {
      "type": "mix_task",
      "name": "debug",
      "request": "launch",
      "startApps": true,
      "projectDir": "${workspaceRoot}",
      "task": "debug",
    },
```

And inside your project, add the task file:

**lib/mix/tasks.debug**

```elixir
defmodule Mix.Tasks.Debug do
  use Mix.Task

  @impl Mix.Task
  def run(_args) do
    IO.puts("debugging...")

    # Code to test
    Crime.double_or_nothing(10)
  end
end
```

With this configuration in place, you are ready, set, go for debugging.
You can either debug when running a test or execute the code you want to debug in the _debug mix task_.

Add a breakpoint(s) to the code you want to check.

![](/images/vscode-debug2.png)

And then click to start the debug session.

![](/images/vscode-debug3.png)

It will stop in the breakpoint to inspect values, add watches and step into and over the following lines.

![](/images/vscode-debug4.png)

With these tools, you are ready to solve any crime you commit in your code.

![](https://media.giphy.com/media/26FLa6peMp3ZNzKnu/giphy.gif)

**References**

- [Debugging](https://elixir-lang.org/getting-started/debugging.html)
- [Debugging techniques in Elixir](http://blog.plataformatec.com.br/2016/04/debugging-techniques-in-elixir-lang/)
- [How to Use IEx.pry in Elixir Tests](https://adamdelong.com/iex-pry-test/)
