---
title: Secret OTP Agent
date: 2021-07-15
description: "Let's learn OTP Agent by building our own version."
image: images/posts/secret-agent.png
images:
  - images/posts/secret-agent.png
tags:
  - Elixir
  - OTP
---

![](https://media.giphy.com/media/3oz8xILeZl6jt78kXm/giphy.gif)

Elixir has a secret agency called OTP, with plenty of operatives in its ranks. One of the foot soldiers is the [Agent](https://hexdocs.pm/elixir/1.12/Agent.html).

Whenever you need to share state among different processes or by the same process at other points in time, you can use Agent. It allows you to have a separate process that keeps your state and allows you to change it and retrieve it in a concurrently safe way.

Let's first start by testing how OTP Agent works:

```elixir
iex(1)> {:ok, pid} = Agent.start_link(fn -> [] end)
{:ok, #PID<0.112.0>}
iex(2)> Agent.get(pid, fn digits -> digits end)
[]
iex(3)> Agent.update(pid, fn digits -> digits ++ ["1"] end)
:ok
iex(4)> Agent.update(pid, fn digits -> digits ++ ["2"] end)
:ok
iex(5)> Agent.get(pid, fn digits -> digits end)
["1", "2"]
iex(6)> Agent.stop(pid)
:ok
```

In this case, we are using Agent to keep the combination of the digits to open a safe deposit box.

The Agent functions can be called anywhere in your safe deposit secret cracking program. You only need to know the pid of the Agent's process after starting it.

Anonymous functions are used to build, get and update state:

## create the state's initial version

```elixir
{:ok, pid} = Agent.start_link(fn -> [] end)
```

## Get the current version of state

```elixir
Agent.get(pid, fn digits -> digits end)
```

## Update the state

```elixir
Agent.update(pid, fn digits -> digits ++ ["4"] end)
```

![](https://media.giphy.com/media/1gRtl9mdLQvvvOfR9u/giphy.gif)

Let's build our own secret agency, OoohTP, and reverse engineer Agent to build our own **SecretAgent**. We will use the basic functions to handle processes in Elixir, namely **spawn_link**, **send**, and **receive**.

First, we need to create the SecretAgent

```elixir
mix new oooh_t_p
```

And create a file **secret_agent.ex** in **lib/oooh_tp/secret_agent.ex**.

```elixir
defmodule OoohTP.SecretAgent do
  def start_link(fun) do
    pid = spawn_link(fun)
    {:ok, pid}
  end
end
```

It will spawn a new process and run our anonymous state function. Let's test it.

```elixir
iex(1)> {:ok, pid} = OoohTP.SecretAgent.start_link(fn -> [] end)
{:ok, #PID<0.175.0>}
iex(2)> self()
#PID<0.173.0>
```

We have a new process id for our own SecretAgent.

But wait 😱, that process died just after it executed the anonymous function.

```elixir
iex(3)> Process.alive?(pid)
false
#PID<0.173.0>
```

So we need to prevent the process from dying. We can do it in two possible ways, recursion or waiting for a new message.
Let's start with the first one.

```elixir
defmodule OoohTP.SecretAgent do
  def start_link(fun) do
    pid = spawn_link(fn -> loop(fun.()) end)
    {:ok, pid}
  end

  defp loop(state) do
    loop(state)
  end
end
```

Now when we test it again:

```elixir
iex(1)> {:ok, pid} = OoohTP.SecretAgent.start_link(fn -> [] end)
{:ok, #PID<0.156.0>}
iex(2)> Process.alive?(pid)
true
```

It's alive and kicking. Notice that even though we have infinite recursion there, it doesn't blow up. That is because of tail call optimization. Elixir is smart enough to notice that the last call of the function is calling itself and doesn't add a new entry to the stack call.

![](https://media.giphy.com/media/NShlefwRfXmMg/giphy.gif)

Now we need to be able to get the value from the state of our Secret Agent.
Let's send a message to get the state from our server, and in the SecretAgent, let's receive that message.

```elixir
defmodule OoohTP.SecretAgent do
  def start_link(fun) do
    pid = spawn_link(fn -> loop(fun.()) end)
    {:ok, pid}
  end

  def get(pid, fun) do
    send(pid, {:get, fun})
  end

  defp loop(state) do
    receive do
      {:get, fun} ->
        IO.puts("Getting digits in #{inspect(self())}")
        fun.(state)
    end

    loop(state)
  end
end
```

Let's test it:

```elixir
iex(7)> {:ok, pid} = OoohTP.SecretAgent.start_link(fn -> [] end)
{:ok, #PID<0.183.0>}
iex(8)> OoohTP.SecretAgent.get(pid, fn state -> state end)
Getting digits in #PID<0.183.0>
{:get, #Function<44.40011524/1 in :erl_eval.expr/5>}
```

SecretAgent received our message because it printed "Getting digits..." but we didn't get the state. 🤔
Right, we need to send the state back to our calling function.

```elixir
defmodule OoohTP.SecretAgent do
  def start_link(fun) do
    pid = spawn_link(fn -> loop(fun.()) end)
    {:ok, pid}
  end

  def get(pid, fun) do
    send(pid, {:get, fun})
  end

  defp loop(state) do
    receive do
      {:get, fun} ->
        IO.puts("Getting digits in #{inspect(self())}")
        send(??to what pid??, {:reply, fun.(state)})
    end

    loop(state)
  end
end
```

But wait 😱, we need the pid of the process we want to send back the state. So left pass it as an argument.

```elixir
defmodule OoohTP.SecretAgent do
  def start_link(fun) do
    pid = spawn_link(fn -> loop(fun.()) end)
    {:ok, pid}
  end

  def get(pid, fun) do
    send(pid, {self(), {:get, fun}})

    receive do
      {:reply, state} -> state
    end
  end

  defp loop(state) do
    receive do
      {from, {:get, fun}} ->
        IO.puts("Getting digits in #{inspect(self())}")
        send(from, {:reply, fun.(state)})
    end

    loop(state)
  end
end
```

Let's give it another try:

```elixir
iex(1)> {:ok, pid} = OoohTP.SecretAgent.start_link(fn -> [] end)
{:ok, #PID<0.156.0>}
iex(2)> OoohTP.SecretAgent.get(pid, fn state -> state end)
Getting digits in #PID<0.156.0>
[]
```

![](https://media.giphy.com/media/5V5C67B3JWD04/giphy.gif)

Pretty cool, right? Now let's update our digits; otherwise, we can't open the safe deposit box. So it will be pretty similar to our get flow.

```elixir
defmodule OoohTP.SecretAgent do
  def start_link(fun) do
    pid = spawn_link(fn -> loop(fun.()) end)
    {:ok, pid}
  end

  def get(pid, fun) do
    send(pid, {self(), {:get, fun}})

    receive do
      {:reply, state} -> state
    end
  end

  def update(pid, fun) do
    send(pid, {self(), {:update, fun}})

    receive do
      {:reply, state} -> state
    end
  end

  defp loop(state) do
    receive do
      {from, {:get, fun}} ->
        IO.puts("Getting digits in #{inspect(self())}")
        send(from, {:reply, fun.(state)})

      {from, {:update, fun}} ->
        IO.puts("Updating digits in #{inspect(self())}")
        fun.(state)
        send(from, {:reply, :ok})
    end

    loop(state)
  end
end
```

Let's update some digits:

```elixir
iex(1)> {:ok, pid} = OoohTP.SecretAgent.start_link(fn -> [] end)
{:ok, #PID<0.156.0>}
iex(2)> OoohTP.SecretAgent.get(pid, fn state -> state end)
Getting digits in #PID<0.156.0>
[]
iex(3)> OoohTP.SecretAgent.update(pid, fn state -> state ++ ["1"] end)
Updating digits in #PID<0.156.0>
:ok
iex(4)> OoohTP.SecretAgent.get(pid, fn state -> state end)
Getting digits in #PID<0.156.0>
[]
```

Oh crap, doesn't seem to be working. 🤔
When we loop, we need to use the new state that resulted from executing the function; otherwise, it will continue to be the same. Let's fix it.

```elixir
defmodule OoohTP.SecretAgent do
  def start_link(fun) do
    pid = spawn_link(fn -> loop(fun.()) end)
    {:ok, pid}
  end

  def get(pid, fun) do
    send(pid, {self(), {:get, fun}})

    receive do
      {:reply, state} -> state
    end
  end

  def update(pid, fun) do
    send(pid, {self(), {:update, fun}})

    receive do
      {:reply, state} -> state
    end
  end

  defp loop(state) do
    receive do
      {from, {:get, fun}} ->
        IO.puts("Getting digits in #{inspect(self())}")
        send(from, {:reply, fun.(state)})
        loop(state)

      {from, {:update, fun}} ->
        IO.puts("Updating digits in #{inspect(self())}")
        new_state = fun.(state)
        send(from, {:reply, :ok})
        loop(new_state)
    end
  end
end
```

Another ride in the park:

```elixir
iex(1)> {:ok, pid} = OoohTP.SecretAgent.start_link(fn -> [] end)
{:ok, #PID<0.156.0>}
iex(2)> OoohTP.SecretAgent.get(pid, fn state -> state end)
Getting digits in #PID<0.156.0>
[]
iex(3)> OoohTP.SecretAgent.update(pid, fn state -> state ++ ["1"] end)
Updating digits in #PID<0.156.0>
:ok
iex(4)> OoohTP.SecretAgent.get(pid, fn state -> state end)
Getting digits in #PID<0.156.0>
["1"]
```

Excellent, our SecretAgent is almost similar to the OTP one.
Now let's add the secret command to allow stopping our fearless super agent.

```elixir
defmodule OoohTP.SecretAgent do
  def start_link(fun) do
    pid = spawn_link(fn -> loop(fun.()) end)
    {:ok, pid}
  end

  def get(pid, fun) do
    send(pid, {self(), {:get, fun}})

    receive do
      {:reply, state} -> state
    end
  end

  def update(pid, fun) do
    send(pid, {self(), {:update, fun}})

    receive do
      {:reply, state} -> state
    end
  end

  def stop(pid) do
    send(pid, {self(), {:stop, :normal}})

    receive do
      {:reply, state} -> state
    end
  end

  defp loop(state) do
    receive do
      {from, {:get, fun}} ->
        IO.puts("Getting digits in #{inspect(self())}")
        send(from, {:reply, fun.(state)})
        loop(state)

      {from, {:update, fun}} ->
        IO.puts("Updating digits in #{inspect(self())}")
        new_state = fun.(state)
        send(from, {:reply, :ok})
        loop(new_state)

      {from, {:stop, reason}} ->
        IO.puts("Stopping the secret agent #{inspect(self())}")
        send(from, {:reply, :ok})
        exit(reason)
    end
  end
end
```

![](https://media.giphy.com/media/ToMjGpKqqn4eZ37NIv6/giphy.gif)

And now, let's try to stop our secret agent.

```elixir
iex(2)> OoohTP.SecretAgent.get(pid, fn state -> state end)
Getting digits in #PID<0.156.0>
[]
iex(3)> OoohTP.SecretAgent.update(pid, fn state -> state ++ ["1"] end)
Updating digits in #PID<0.156.0>
:ok
iex(4)> OoohTP.SecretAgent.get(pid, fn state -> state end)
Getting digits in #PID<0.156.0>
["1"]
iex(5)> OoohTP.SecretAgent.stop(pid)
Stopping the secret agent #PID<0.156.0>
:ok
iex(6)> Process.alive?(pid)
false
```

Awesome, we have a fully functional secret agent. But let's just refactor it a bit because we see a lot of duplication in the client's functions to call our agent.

![](https://media.giphy.com/media/W3hLOKmGBwCsg/giphy.gif)

```elixir
defmodule OoohTP.SecretAgent do
  def start_link(fun) do
    pid = spawn_link(fn -> loop(fun.()) end)
    {:ok, pid}
  end

  def get(pid, fun), do: call(pid, {:get, fun})

  def update(pid, fun), do: call(pid, {:update, fun})

  def stop(pid), do: call(pid, {:stop, :normal})

  defp call(pid, message) do
    send(pid, {self(), message})

    receive do
      {:reply, state} -> state
    end
  end

  defp loop(state) do
    receive do
      {from, {:get, fun}} ->
        IO.puts("Getting digits in #{inspect(self())}")
        send(from, {:reply, fun.(state)})
        loop(state)

      {from, {:update, fun}} ->
        IO.puts("Updating digits in #{inspect(self())}")
        new_state = fun.(state)
        send(from, {:reply, :ok})
        loop(new_state)

      {from, {:stop, reason}} ->
        IO.puts("Stopping the secret agent #{inspect(self())}")
        send(from, {:reply, :ok})
        exit(reason)
    end
  end
end
```

And we now have a fully functional OoohTP SecretAgent working!

![](https://media.giphy.com/media/nu4DN4nKos3m0/giphy.gif)
