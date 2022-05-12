---
title: Running Elixir
date: 2021-07-06
description: "Different ways of running Elixir"
images:
- images/featured/elixir-tutorial-running-elixir.png
tags:
  - Elixir
---

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
In this case, the file extension `.ex` means that this type of file is not a script, and it should be compiled. This is because scripts are compiled only they run, while these files are for longer-term usage, like a library or an application, so they should be compiled first.
You can also edit the file and recompile it from inside the shell.

```sh
iex(3)> c "hey_elixir.ex"
Hello there, Elixir 👋
[]
```

In `iex` shell session, you can insert any Elixir expression.

To exit from `iex`, you need to press `Ctrl+c` twice.

## iex tip

![](https://media.giphy.com/media/Rk8CZk8M7UHzG/giphy.gif)

Sometimes, when you forget to close a quote, double quote, or brackets, it will continue to create new lines when you press enter.
You can exit by pressing `Ctrl+c` twice, but you will leave the interactive shell. If you want to stop the statement, you are inserting
add `#iex:break` it will error out, but you will still be in the shell.

```sh
iex(1)> IO.puts("dssdsda
...(1)> asdasdsda
...(1)> sadsadsda
...(1)> #iex:break
** (TokenMissingError) iex:1: incomplete expression
```

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

Notice that after you type `Hello`,  you can hit tab and have **autocomplete**.

`iex` is your friend, and in case you need help, well, just hit `h`.

One of the options you see there is `open`. So let's try to open our module.

```sh
iex(3)> open Hello
Invalid arguments for open helper: "/home/pedro-gaspar/tmp/hello.ex"
```

What is going on? Why isn't it working? Checking the documentation, it seems right. You need the EDITOR env var set to code, but I already have it.
After searching a bit for it, I found https://github.com/elixir-lang/elixir/blob/master/lib/iex/test/iex/helpers_test.exs#L269

So it doesn't work for in-memory modules, like the one we just did.

## Running with mix

![](https://media.giphy.com/media/hTI5Pl6SAA1XDbbcLe/giphy.gif)

Time to add mix into the ... erm ... mix (couldn't avoid it)

For persons who worked with Ruby on Rails, imagine a tool that mixes rails and rake. Mixing both, you get mix (today I'm on a roll).

Mix is a tool that allows you to run commands, like generators, or tasks you or some module provides.

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

We've just run a command `mix command options`. In this case, the command is `new` and the options are the project's name.

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

This similar to the `Gemfile` for ruby or `package.json` for node.js. You define the application properties and dependencies in it.

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

The remaining files are basically for documentation and tests which, end up being documentation. You add tests
to list and document your app expected behavior.

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

Better right? It loaded our application, and now this vm instance knows what the `Hello` module is.

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

Our test file needs to end with a `__test` suffix and has `.exs` extension.
It's a module and the first line `use ExUnit.Case` must be including support for those `test`. It's not `def` like the last time we defined a function.

Ok, this is failing because it is testing the generated code. Let's fix it.

```elixir
defmodule HelloTest do
  use ExUnit.Case
  doctest Hello

  test "greets the world" do
    assert Hello.world() == "Hello world!"
  end
end
```

After rerunning tests:

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

Still failing? But we fixed it. But if we check the failing test is located in lib/hello.ex line 11 and not in the test file.
Well if we check the file at that location, it's documentation. It looks like documentation but, if you check the second line of our test file, you see `doctest Hello`. Oh, we are also running tests that exist in the docs. Let's fix them as well:

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

And if we rerun tests:

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

It wasn't listed when I was writing `mix help`, but that is because it needs to be compiled for it to appear.

```sh
mix help # no showing
mix compile
mix help
mix                   # Runs the default task (current: "mix run")
mix app.config        # Configures all registered apps
mix app.start         # Starts all registered apps
mix app.tree          # Prints the application tree
mix archive           # Lists installed archives
mix archive.build     # Archives this project into a .ez file
mix archive.install   # Installs an archive locally
mix archive.uninstall # Uninstalls archives
mix clean             # Deletes generated application files
mix cmd               # Executes the given command
mix compile           # Compiles source files
mix deps              # Lists dependencies and their status
mix deps.clean        # Deletes the given dependencies' files
mix deps.compile      # Compiles dependencies
mix deps.get          # Gets all out of date dependencies
mix deps.tree         # Prints the dependency tree
mix deps.unlock       # Unlocks the given dependencies
mix deps.update       # Updates the given dependencies
mix do                # Executes the tasks separated by comma
mix escript           # Lists installed escripts
mix escript.build     # Builds an escript for the project
mix escript.install   # Installs an escript locally
mix escript.uninstall # Uninstalls escripts
mix format            # Formats the given files/patterns
mix hello             # Simply calls the Hello.world/0 function.
mix help              # Prints help information for tasks
mix hex               # Prints Hex help information
mix hex.audit         # Shows retired Hex deps for the current project
mix hex.build         # Builds a new package version locally
mix hex.config        # Reads, updates or deletes local Hex config
mix hex.docs          # Fetches or opens documentation of a package
mix hex.info          # Prints Hex information
mix hex.organization  # Manages Hex.pm organizations
mix hex.outdated      # Shows outdated Hex deps for the current project
mix hex.owner         # Manages Hex package ownership
mix hex.package       # Fetches or diffs packages
mix hex.publish       # Publishes a new package version
mix hex.registry      # Manages local Hex registries
mix hex.repo          # Manages Hex repositories
mix hex.retire        # Retires a package version
mix hex.search        # Searches for package names
mix hex.sponsor       # Show Hex packages accepting sponsorships
mix hex.user          # Manages your Hex user account
mix loadconfig        # Loads and persists the given configuration
mix local             # Lists local tasks
mix local.hex         # Installs Hex locally
mix local.phx         # Updates the Phoenix project generator locally
mix local.public_keys # Manages public keys
mix local.rebar       # Installs Rebar locally
mix new               # Creates a new Elixir project
mix phx.new           # Creates a new Phoenix v1.5.9 application
mix phx.new.ecto      # Creates a new Ecto project within an umbrella project
mix phx.new.web       # Creates a new Phoenix web project within an umbrella project
mix profile.cprof     # Profiles the given file or expression with cprof
mix profile.eprof     # Profiles the given file or expression with eprof
mix profile.fprof     # Profiles the given file or expression with fprof
mix release           # Assembles a self-contained release
mix release.init      # Generates sample files for releases
mix run               # Starts and runs the current application
mix test              # Runs a project's tests
mix test.coverage     # Build report from exported test coverage
mix xref              # Prints cross reference information
iex -S mix            # Starts IEx and runs the default task
```

There it is 🥳

![](https://media.giphy.com/media/3o7ZeEZUzRjyvWuuIg/giphy.gif)

And just like that, we know different ways to run Elixir code.
