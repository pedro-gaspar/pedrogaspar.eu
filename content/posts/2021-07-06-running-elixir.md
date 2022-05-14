---
title: Running Elixir
date: 2021-07-06
description: "The different ways to execute elixir, from using elixir, or iex, running inside a mix project, using mix tasks and tests."
image: images/posts/elixir-tutorial-running-elixir.png
images:
  - images/posts/elixir-tutorial-running-elixir.png
tags:
  - Elixir
---

{{< alert "secondary" >}}
The different ways to execute elixir, from using elixir, or iex, running inside a mix project, using mix tasks and tests.
{{< /alert >}}

![](https://media.giphy.com/media/YM5eNbWmIYqll3Ar3m/giphy.gif)

## Running with elixir

You can run Elixir with ... erm ... the `elixir` command. Let's create a sample file:

**hey_elixir.exs**

```elixir
IO.puts("Hello there, Elixir 🙂")
```

The `.exs` identifies an _Elixir script file_, that doesn't require it to be compiled.

To run this file:

```sh
$ elixir hey_elixir.exs
```

As usual, add `--help` for more options when running it.

## Running with iex

You can also spin up an interactive console called **iex**. It's an elixir [REPL](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop):

```sh
$ iex
Erlang/OTP 24 [erts-12.0.2] [source] [64-bit] [smp:12:12] [ds:12:12:10] [async-threads:1] [jit]

Interactive Elixir (1.12.1) - press Ctrl+C to exit (type h() ENTER for help)
iex(1)> IO.puts("Hello there, Elixir 🙂")
Hello there, Elixir 🙂
```

You can also run a file:

```sh
iex(2)> c "hey_elixir.ex"
Hello there, Elixir 🙂
```

In this case, the file extension `.ex` means that this type of file is not a script, and it should be compiled. This is because scripts are compiled only when they run, while these files are for longer-term usage, like a library or an application, so they should be compiled first.
You can also edit the file and recompile it from inside the shell.

```sh
iex(3)> c "hey_elixir.ex"
Hello there, Elixir 👋
[]
```

In `iex` shell session, you can insert any Elixir expression.

To exit from `iex`, you need to press `Ctrl+c` twice.

## Defining and running modules in iex

Now let's create our first module. Modules allow you to group and "namespace" your functions.

**hello.ex**

```elixir
defmodule Hello do
  def world() do
    IO.puts("Hello world!")
  end
end
[Hello]
```

But this time, it doesn't run. It only returns the compiled module. Now you can run it with:

```sh
iex(2)> Hello.world()
Hello world!
:ok
```

Notice that after you type `Hello`, you can hit tab and have **autocomplete**.

`iex` is your friend, and in case you need help, well, just hit `h`.

One of the options you see there is `open`. So let's try to open our module.

```sh
iex(3)> open Hello
Invalid arguments for open helper: "/home/pedro-gaspar/tmp/hello.ex"
```

What is going on? Why isn't it working? Checking the documentation, it seems right. You need the EDITOR environment variable set to `code`, but I already have it.
After searching a bit for it, I found https://github.com/elixir-lang/elixir/blob/master/lib/iex/test/iex/helpers_test.exs#L269

So it doesn't work for in-memory modules, like the one we just did.

## Running with mix

![](https://media.giphy.com/media/hTI5Pl6SAA1XDbbcLe/giphy.gif)

Time to add mix into the ... erm ... mix (couldn't avoid it)

For persons who worked with Ruby on Rails, imagine a tool that mixes rails and rake. Mixing both, you get `mix` (today I'm on a roll).

`mix` is a tool that allows you to run commands, like generators, or tasks you or some module provides.

Let's create a new project, or as we call them in Elixirland, **application**, and for that, we use mix.

```sh
mix new hello
* creating README.md
* creating .formatter.exs
* creating .gitignore
* creating mix.exs
* creating lib
* creating lib/hello.ex
* creating test
* creating test/test_helper.exs
* creating test/hello_test.exs
```

We've just run a command in the format `mix [command] [options]`. In this case, the command is `new` and the options are the project's name.

You can then inspect the generated files:

```sh
$ cd hello
$ tree
.
├── lib
│   └── hello.ex
├── mix.exs
├── README.md
└── test
    ├── hello_test.exs
    └── test_helper.exs

2 directories, 5 files
```

If generated a couple of files. Let's look at some of them.

**mix.exs**

```elixir
defmodule Hello.MixProject do
  use Mix.Project

  def project do
    [
      app: :hello,                           # application name
      version: "0.1.0",                      # this
      elixir: "~> 1.12",                     # version of elixir
      start_permanent: Mix.env() == :prod,   # shutdown vm if application crashes
      deps: deps()                           # required dependencies
    ]
  end

  # Run "mix help compile.app" to learn about applications.
  def application do
    [
      extra_applications: [:logger]
    ]
  end

  # Run "mix help deps" to learn about dependencies.
  defp deps do
    [
      # {:dep_from_hexpm, "~> 0.3.0"},
      # {:dep_from_git, git: "https://github.com/elixir-lang/my_dep.git", tag: "0.1.0"}
    ]
  end
end
```

This is similar to the `Gemfile` for ruby or `package.json` for node.js. You define the application properties and dependencies in it.

**lib/hello.ex**

```elixir
defmodule Hello do
  @moduledoc """
  Documentation for `Hello`.
  """

  @doc """
  Hello world.

  ## Examples

      iex> Hello.hello()
      :world

  """
  def hello do
    :world
  end
end
```

This is a sample module for you to start adding some magic.

## Running with tests

The remaining files are basically for documentation and tests which, end up being documentation. You add tests to document your app expected behavior.

Speaking of tests, let's run them:

```sh
$ mix test
Compiling 1 file (.ex)
Generated hello app
..

Finished in 0.01 seconds (0.00s async, 0.01s sync)
1 doctest, 1 test, 0 failures

Randomized with seed 623986
```

All green :)

Now let's change our application to have a `world` method instead:

**lib/hello.ex**

```elixir
defmodule Hello do
  @moduledoc """
  Documentation for `Hello`.
  """

  @doc """
  Hello world.

  ## Examples

      iex> Hello.hello()
      :world

  """
  def world() do
    "Hello world!"
  end
end
```

So let's invoke this method:

```sh
$ iex
Erlang/OTP 24 [erts-12.0.2] [source] [64-bit] [smp:12:12] [ds:12:12:10] [async-threads:1] [jit]

Interactive Elixir (1.12.1) - press Ctrl+C to exit (type h() ENTER for help)
iex(1)> Hello.world()
** (UndefinedFunctionError) function Hello.world/0 is undefined (module Hello is not available)
    Hello.world()
```

Well, we thought it would work, right? Well, calling `iex` like this, we don't load the app. For that to work, we need to run the default mix task.

```sh
iex -S mix
Erlang/OTP 24 [erts-12.0.2] [source] [64-bit] [smp:12:12] [ds:12:12:10] [async-threads:1] [jit]

Compiling 1 file (.ex)
Generated hello app
Interactive Elixir (1.12.1) - press Ctrl+C to exit (type h() ENTER for help)
iex(1)> Hello.world()
"Hello world!"
```

Better right? It loaded our application, and now this virtual machine instance knows what the `Hello` module is.

We could also run this from the command line like this:

```sh
$ mix run -e "Hello.world()"
```

Well, nothing happens because we are just returning a value. Let's change our file to output something:

**lib/hello.ex**

```elixir
defmodule Hello do
  @moduledoc """
  Documentation for `Hello`.
  """

  @doc """
  Hello world.

  ## Examples

      iex> Hello.hello()
      :world

  """
  def world() do
    IO.puts("Greeting the world from elixir")
    "Hello world!"
  end
end
```

```sh
$ mix run -e "Hello.world()"
Greeting the world from elixir
```

Now let's try to rerun our tests:

```sh
$ mix test
Compiling 1 file (.ex)
warning: Hello.hello/0 is undefined or private
Found at 2 locations:
  (for doctest at) lib/hello.ex:11: HelloTest."doctest Hello.world/0 (1)"/1
  test/hello_test.exs:6: HelloTest."test greets the world"/1



  1) doctest Hello.world/0 (1) (HelloTest)
     test/hello_test.exs:3
     ** (UndefinedFunctionError) function Hello.hello/0 is undefined or private
     stacktrace:
       (hello 0.1.0) Hello.hello()
       (for doctest at) lib/hello.ex:11: (test)



  2) test greets the world (HelloTest)
     test/hello_test.exs:5
     ** (UndefinedFunctionError) function Hello.hello/0 is undefined or private
     code: assert Hello.hello() == :world
     stacktrace:
       (hello 0.1.0) Hello.hello()
       test/hello_test.exs:6: (test)



Finished in 0.02 seconds (0.00s async, 0.02s sync)
1 doctest, 1 test, 2 failures

Randomized with seed 42973
```

![](https://media.giphy.com/media/GDnomdqpSHlIs/giphy.gif)

Oops, we broke our app tests. Let's check the existing test:

**test/hello_test.exs**

```elixir
defmodule HelloTest do
  use ExUnit.Case
  doctest Hello

  test "greets the world" do
    assert Hello.hello() == :world
  end
end
```

Our test file needs to end with a `_test` suffix and have a `.exs` extension.
It's a module and the first line `use ExUnit.Case` must be including support for those `test`. It's not `def` like the last time we defined a function.

Ok, this is failing because it was testing the generated code. Let's fix it.

```elixir
defmodule HelloTest do
  use ExUnit.Case
  doctest Hello

  test "greets the world" do
    assert Hello.world() == "Hello world!"
  end
end
```

After re-running tests:

```sh
$ mix test
warning: Hello.hello/0 is undefined or private
  (for doctest at) lib/hello.ex:11: HelloTest."doctest Hello.world/0 (1)"/1

Greeting the world from elixir
.

  1) doctest Hello.world/0 (1) (HelloTest)
     test/hello_test.exs:3
     ** (UndefinedFunctionError) function Hello.hello/0 is undefined or private
     stacktrace:
       (hello 0.1.0) Hello.hello()
       (for doctest at) lib/hello.ex:11: (test)



Finished in 0.02 seconds (0.00s async, 0.02s sync)
1 doctest, 1 test, 1 failure

Randomized with seed 977408
```

Still, failing? But we fixed it. 🤔

But if we check the failing test is located in `lib/hello.ex` line 11 and not in the test file.

Well if we check the file at that location, it's a documentation block. It looks like documentation but, if you check the second line of our test file, you see `doctest Hello`. Oh, we are also running tests that exist in the docs. Let's fix them as well:

**lib/hello.ex**

```elixir
defmodule Hello do
  @moduledoc """
  Documentation for `Hello`.
  """

  @doc """
  Hello world.

  ## Examples

      iex> Hello.world()
      "Hello world!"

  """
  def world() do
    IO.puts("Greeting the world from elixir")
    "Hello world!"
  end
end
```

And if we re-run tests:

```sh
$ mix test
Compiling 1 file (.ex)
Greeting the world from elixir
.Greeting the world from elixir
.

Finished in 0.01 seconds (0.00s async, 0.01s sync)
1 doctest, 1 test, 0 failures

Randomized with seed 816150
```

Green again.

![](https://media.giphy.com/media/ugOaZ3Wi8lqZW/giphy.gif)

## Running with a mix task

Last but not least, let's add a new mix task that runs our function.

**lib/mix/tasks/hello.ex**

```elixir
defmodule Mix.Tasks.Hello do
  use Mix.Task

  def run(_) do
    Hello.world()
  end
end
```

Was trying over and over to run like this

```sh
$ mix run hello
```

So nothing like reading the docs:

```sh
$ mix help run
```

And run it:

```sh
$ mix hello
Compiling 1 file (.ex)
Greeting the world from elixir
```

![](https://media.giphy.com/media/3o7ZeEZUzRjyvWuuIg/giphy.gif)

And just like that, we know different ways to run Elixir code.
