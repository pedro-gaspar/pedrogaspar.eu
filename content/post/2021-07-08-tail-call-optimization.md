---
title: Should I always use tail recursion?
date: 2021-07-08
description: "The short answer is the dreaded ... it depends."
images:
- images/featured/tail-call-optimization.png
tags:
  - Elixir
---

While trying to implement my own version of `map` for an Exercism exercise, I've come across a doubt that should trouble the Elixir newbie.

![](https://media.giphy.com/media/YcFOfbeTcHtVS/giphy.gif)

Whenever we have recursive functions, if calling the function it's not the last operation being done, we may get into trouble by reaching the stack trace limit.


## Tail call recursion

A special form of recursion where the last operation of a function is a recursive call. The recursion may be optimized by executing the call in the current stack frame and returning its result rather than creating a new stack frame.

So if the last thing we call is the recursive function, that doesn't result as a new entry in the stack. Instead, the calling function will simply return the value it gets.

> So ... should I always use tail call recursion?

![](https://media.giphy.com/media/y65VoOlimZaus/giphy.gif)

I don't know; let's compare it.

First, let's create an app to test our assumptions.

```sh
$ mix new benchmark_recursion
```

## Mapping with Enum.map

```elixir
Enum.map([1,2,3], &(&1 * 2))
```

The swiss army knife of mapping listings.

![](https://media.giphy.com/media/xT5LMxAxpGSb5AZt8A/giphy.gif)

## Mapping without tail call recursion

**lib/benchmark_recursion.ex**

```elixir
defmodule BenchmarkRecursion do
  def map([head | tail], fun) do
    [fun.(head) | map(tail, fun)]
  end

  def map([], _fun), do: []
end
```

Pretty straight forward right?

![](https://media.giphy.com/media/3o7btNa0RUYa5E7iiQ/giphy.gif)

We have a recursive function to go over a list that invokes the function we pass as an argument, just like `Enum.map`.

Let's give it a try:

```sh
iex -S mix
iex(1)> list = Enum.to_list(1..100)
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20,
 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37,
 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, ...]
iex(2)> double_it = fn n -> n * 2 end
#Function<44.40011524/1 in :erl_eval.expr/5>
iex(3)> BenchmarkRecursion.map(list, double_it)
[2, 4, 6, 8, 10, 12, 14, 16, 18, 20, 22, 24, 26, 28, 30, 32, 34, 36,
 38, 40, 42, 44, 46, 48, 50, 52, 54, 56, 58, 60, 62, 64, 66, 68, 70,
 72, 74, 76, 78, 80, 82, 84, 86, 88, 90, 92, 94, 96, 98, 100, ...]
```

## Mapping with tail call recursion

Now let's do the same map function but with tail call optimization.

We need to write a function with an accumulator argument that stores the final mapped list for this to work.

```elixir
BenchmarkRecursion do
def map([], _fun), do: []

  def map_tail_call(list, fun), do: do_map_tail_call(list, fun, [])

  def do_map_tail_call([head | tail], fun, mapped) do
    do_map_tail_call(tail, fun, [fun.(head) | mapped])
  end

  def do_map_tail_call([], _fun, mapped), do: mapped
end
```

Let's try it:

```elixir
$ iex -S mix
iex(6)> BenchmarkRecursion.map_tail_call(list, double_it)
[200, 198, 196, 194, 192, 190, 188, 186, 184, 182, 180, 178, 176, 174,
 172, 170, 168, 166, 164, 162, 160, 158, 156, 154, 152, 150, 148, 146,
 144, 142, 140, 138, 136, 134, 132, 130, 128, 126, 124, 122, 120, 118,
 116, 114, 112, 110, 108, 106, 104, 102, ...]
```

![](https://media.giphy.com/media/lqNBMrcSQdXZ4KvQhZ/giphy.gif)

Hmmm, the list items are reversed, so we need to reverse the list in the end to keep the correct order.

```elixir
defmodule BenchmarkRecursion do
  def map([head | tail], fun) do
    [fun.(head) | map(tail, fun)]
  end

  def map([], _fun), do: []

  def map_tail_call(list, fun), do: do_map_tail_call(list, fun, [])

  def do_map_tail_call([head | tail], fun, mapped) do
    do_map_tail_call(tail, fun, [fun.(head) | mapped])
  end

  def do_map_tail_call([], _fun, mapped), do: reverse(mapped)

  def reverse(list), do: do_reverse(list, [])

  def do_reverse([head | tail], reversed) do
    do_reverse(tail, [head | reversed])
  end

  def do_reverse([], reversed), do: reversed
end
```

Let's give it another go:

```elixir
iex -S mix
Erlang/OTP 24 [erts-12.0.2] [source] [64-bit] [smp:24:24] [ds:24:24:10] [async-threads:1] [jit]

Compiling 1 file (.ex)
Interactive Elixir (1.12.1) - press Ctrl+C to exit (type h() ENTER for help)
iex(1)> list = Enum.to_list(1..100)
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20,
 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37,
 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, ...]
iex(2)> double_it = fn n -> n * 2 end
#Function<44.40011524/1 in :erl_eval.expr/5>
iex(3)> BenchmarkRecursion.map_tail_call(list, double_it)
[2, 4, 6, 8, 10, 12, 14, 16, 18, 20, 22, 24, 26, 28, 30, 32, 34, 36,
 38, 40, 42, 44, 46, 48, 50, 52, 54, 56, 58, 60, 62, 64, 66, 68, 70,
 72, 74, 76, 78, 80, 82, 84, 86, 88, 90, 92, 94, 96, 98, 100, ...]
```

It's the same as the version without tail call recursion.

Now let's try each with large lists:

```elixir
iex -S mix
Erlang/OTP 24 [erts-12.0.2] [source] [64-bit] [smp:24:24] [ds:24:24:10] [async-threads:1] [jit]

Interactive Elixir (1.12.1) - press Ctrl+C to exit (type h() ENTER for help)
iex(1)> biiiiiiiig_list = Enum.to_list(1..1_000_000)
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20,
 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37,
 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, ...]
iex(2)> double_it = fn n -> n * 2 end
#Function<44.40011524/1 in :erl_eval.expr/5>
iex(3)> BenchmarkRecursion.map(list, double_it)
** (CompileError) iex:3: undefined function list/0
    (stdlib 3.15.1) lists.erl:1358: :lists.mapfoldl/3
iex(3)> BenchmarkRecursion.map(bi list, double_it)
biiiiiiiig_list    binary_part/3      binding/0          binding/1
bit_size/1
iex(3)> BenchmarkRecursion.map(bi list, double_it)
biiiiiiiig_list    binary_part/3      binding/0          binding/1
bit_size/1
iex(3)> BenchmarkRecursion.map(biiiiiiiig_list, double_it)
[2, 4, 6, 8, 10, 12, 14, 16, 18, 20, 22, 24, 26, 28, 30, 32, 34, 36,
 38, 40, 42, 44, 46, 48, 50, 52, 54, 56, 58, 60, 62, 64, 66, 68, 70,
 72, 74, 76, 78, 80, 82, 84, 86, 88, 90, 92, 94, 96, 98, 100, ...]
iex(4)> BenchmarkRecursion.map_tail_call(biiiiiiiig_list, double_it)
[2, 4, 6, 8, 10, 12, 14, 16, 18, 20, 22, 24, 26, 28, 30, 32, 34, 36,
 38, 40, 42, 44, 46, 48, 50, 52, 54, 56, 58, 60, 62, 64, 66, 68, 70,
 72, 74, 76, 78, 80, 82, 84, 86, 88, 90, 92, 94, 96, 98, 100, ...]
```

Well, they almost feel to be taking the same time.

![](https://media.giphy.com/media/3jN3GziOKUEmI/giphy.gif)


Instead of gut feeling or going with the one that seems right after timing a couple of times, let's do it right.
We should put on our race judge hat and clock and let's make a race, or in fact, a lot of them.


## Benchmark FTW

Let's add a benchmark library:

Add the following to your `mix.exs` dependencies

`{:benchee, "~> 1.0", only: :dev}`

```elixir
defmodule BenchmarkRecursion.MixProject do
  use Mix.Project

  def project do
    [
      app: :benchmark_recursion,
      version: "0.1.0",
      elixir: "~> 1.12",
      start_permanent: Mix.env() == :prod,
      deps: deps()
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
      {:benchee, "~> 1.0", only: :dev}
    ]
  end
end
```

## Bencharmark small lists

So let's benchmark when using small lists.

```iex -S mix
list = Enum.to_list(1..100)
double_it = fn n -> n * 2 end

iex(4)> Benchee.run(
...(4)>   %{
...(4)>     "Enum.map" => fn -> Enum.map(list, double_it) end,
...(4)>     "map" => fn -> BenchmarkRecursion.map(list, double_it) end,
...(4)>     "map_tail_call" => fn -> BenchmarkRecursion.map_tail_call(list, double_it) end
...(4)>
...(4)>   }
...(4)> )
Operating System: Linux
CPU Information: AMD Ryzen 9 3900X 12-Core Processor
Number of Available Cores: 24
Available memory: 62.78 GB
Elixir 1.12.1
Erlang 24.0.2

Benchmark suite executing with the following configuration:
warmup: 2 s
time: 5 s
memory time: 0 ns
parallel: 1
inputs: none specified
Estimated total run time: 21 s

Benchmarking Enum.map...
Benchmarking map...
Benchmarking map_tail_call...

Name                    ips        average  deviation         median         99th %
map                 37.74 K       26.50 μs    ±16.38%       25.37 μs       41.90 μs
Enum.map            37.41 K       26.73 μs    ±17.86%       25.64 μs       42.18 μs
map_tail_call       35.76 K       27.96 μs    ±22.41%       26.75 μs       36.42 μs

Comparison:
map                 37.74 K
Enum.map            37.41 K - 1.01x slower +0.23 μs
map_tail_call       35.76 K - 1.06x slower +1.46 μs
%Benchee.Suite{
  configuration: %Benchee.Configuration{
    after_each: nil,
    after_scenario: nil,
    assigns: %{},
    before_each: nil,
    before_scenario: nil,
    formatters: [Benchee.Formatters.Console],
    inputs: nil,
    load: false,
    measure_function_call_overhead: true,
    memory_time: 0.0,
    parallel: 1,
    percentiles: '2c',
    pre_check: false,
    print: %{benchmarking: true, configuration: true, fast_warning: true},
    save: false,
    time: 5.0e9,
    title: nil,
    unit_scaling: :best,
    warmup: 2.0e9
  },
  scenarios: [
    %Benchee.Scenario{
      after_each: nil,
      after_scenario: nil,
      before_each: nil,
      before_scenario: nil,
      function: #Function<45.40011524/0 in :erl_eval.expr/5>,
      input: :__no_input,
      input_name: :__no_input,
      job_name: "map",
      memory_usage_data: %Benchee.CollectionData{
        samples: [],
        statistics: %Benchee.Statistics{
          absolute_difference: nil,
          average: nil,
          ips: nil,
          maximum: nil,
          median: nil,
          minimum: nil,
          mode: nil,
          percentiles: nil,
          relative_less: nil,
          relative_more: nil,
          sample_size: 0,
          std_dev: nil,
          std_dev_ips: nil,
          std_dev_ratio: nil
        }
      },
      name: "map",
      run_time_data: %Benchee.CollectionData{
        samples: [58041.0, 28480.0, 31241.0, 2.92e4, 27650.0, 29481.0, 39670.0,
         33780.0, 28721.0, 30370.0, 29670.0, 27591.0, 27880.0, 27840.0, 33651.0,
         31020.0, 29840.0, 27901.0, 28730.0, 27530.0, 27751.0, 27780.0, 28960.0,
         28001.0, 27350.0, 28090.0, 28681.0, 2.77e4, 2.81e4, 27770.0, 28701.0,
         27990.0, 27290.0, 28061.0, 28460.0, ...],
        statistics: %Benchee.Statistics{
          absolute_difference: nil,
          average: 26499.557949556358,
          ips: 37736.47854441819,
          maximum: 973691.0,
          median: 25370.0,
          minimum: 23650.0,
          mode: 25260.0,
          percentiles: %{50 => 25370.0, 99 => 41901.0},
          relative_less: nil,
          relative_more: nil,
          sample_size: 184721,
          std_dev: 4340.122892197869,
          std_dev_ips: 6180.5164717589205,
          std_dev_ratio: 0.16378095440156312
        }
      },
      tag: nil
    },
    %Benchee.Scenario{
      after_each: nil,
      after_scenario: nil,
      before_each: nil,
      before_scenario: nil,
      function: #Function<45.40011524/0 in :erl_eval.expr/5>,
      input: :__no_input,
      input_name: :__no_input,
      job_name: "Enum.map",
      memory_usage_data: %Benchee.CollectionData{
        samples: [],
        statistics: %Benchee.Statistics{
          absolute_difference: nil,
          average: nil,
          ips: nil,
          maximum: nil,
          median: nil,
          minimum: nil,
          mode: nil,
          percentiles: nil,
          relative_less: nil,
          relative_more: nil,
          sample_size: 0,
          std_dev: nil,
          std_dev_ips: nil,
          std_dev_ratio: nil
        }
      },
      name: "Enum.map",
      run_time_data: %Benchee.CollectionData{
        samples: [46661.0, 29090.0, 32020.0, 30051.0, 27940.0, 3.01e4, 40781.0,
         34170.0, 28510.0, 30571.0, 29750.0, 28090.0, 28221.0, 28020.0, 33690.0,
         34701.0, 30170.0, 28411.0, 29370.0, 28180.0, 28130.0, 27961.0, 2.88e4,
         28480.0, 28021.0, 28470.0, 28850.0, 27961.0, 28080.0, 2.82e4, 29131.0,
         28120.0, 27780.0, 28231.0, ...],
        statistics: %Benchee.Statistics{
          absolute_difference: 234.81912119593835,
          average: 26734.377070752296,
          ips: 37405.02340314527,
          maximum: 1076822.0,
          median: 25640.0,
          minimum: 23610.0,
          mode: 25520.0,
          percentiles: %{50 => 25640.0, 99 => 42180.0},
          relative_less: 0.99121658527616,
          relative_more: 1.0088612467288296,
          sample_size: 183146,
          std_dev: 4775.528289032688,
          std_dev_ips: 6681.612477482104,
          std_dev_ratio: 0.17862874741365
        }
      },
      tag: nil
    },
    %Benchee.Scenario{
      after_each: nil,
      after_scenario: nil,
      before_each: nil,
      before_scenario: nil,
      function: #Function<45.40011524/0 in :erl_eval.expr/5>,
      input: :__no_input,
      input_name: :__no_input,
      job_name: "map_tail_call",
      memory_usage_data: %Benchee.CollectionData{
        samples: [],
        statistics: %Benchee.Statistics{
          absolute_difference: nil,
          average: nil,
          ips: nil,
          maximum: nil,
          median: nil,
          minimum: nil,
          mode: nil,
          percentiles: nil,
          relative_less: nil,
          relative_more: nil,
          sample_size: 0,
          std_dev: nil,
          std_dev_ips: nil,
          std_dev_ratio: nil
        }
      },
      name: "map_tail_call",
      run_time_data: %Benchee.CollectionData{
        samples: [48401.0, 31590.0, 32261.0, 30040.0, 2.92e4, 31451.0, 2.82e4,
         30090.0, 28021.0, 30890.0, 34370.0, 32341.0, 3.28e4, 33020.0, 31901.0,
         32850.0, 37320.0, 30601.0, 32440.0, 32041.0, 32570.0, 32910.0, 32811.0,
         32220.0, 33040.0, 32701.0, 3.26e4, 37341.0, 32510.0, 29320.0, 28691.0,
         37550.0, 36940.0, ...],
        statistics: %Benchee.Statistics{
          absolute_difference: 1463.7712998073657,
          average: 27963.329249363724,
          ips: 35761.12096962682,
          maximum: 1745469.0,
          median: 26750.0,
          minimum: 24950.0,
          mode: 26240.0,
          percentiles: %{50 => 26750.0, 99 => 36420.609999999986},
          relative_less: 0.9476538974757209,
          relative_more: 1.0552375742491158,
          sample_size: 175238,
          std_dev: 6267.717277747323,
          std_dev_ips: 8015.518959640409,
          std_dev_ratio: 0.22414059577294212
        }
      },
      tag: nil
    }
  ],
  system: %{
    available_memory: "62.78 GB",
    cpu_speed: "AMD Ryzen 9 3900X 12-Core Processor",
    elixir: "1.12.1",
    erlang: "24.0.2",
    num_cores: 24,
    os: :Linux
  }
}
```

In the comparison section, we get the details we want:

```
Comparison:
map                 37.74 K
Enum.map            37.41 K - 1.01x slower +0.23 μs
map_tail_call       35.76 K - 1.06x slower +1.46 μs
```

Surprise, surprise. The non-optimized version is the fastest.

![](https://media.giphy.com/media/5VKbvrjxpVJCM/giphy.gif)

Well, it makes sense because it is the less complex of them all.
It only goes over the list once.

## Bencharmark bigger lists

```elixir
...
iex(1)> list = Enum.to_list(1..1_000)
...
iex(4)> Benchee.run(
...(4)>   %{
...(4)>     "Enum.map" => fn -> Enum.map(list, double_it) end,
...(4)>     "map" => fn -> BenchmarkRecursion.map(list, double_it) end,
...(4)>     "map_tail_call" => fn -> BenchmarkRecursion.map_tail_call(list, double_it) end
...(4)>
...(4)>   }
...(4)> )
....
Comparison:
map                  3.67 K
Enum.map             3.62 K - 1.01x slower +4.02 μs
map_tail_call        3.48 K - 1.05x slower +14.92 μs
```

Our modest `map` continues to be the fastest.

![](https://media.giphy.com/media/5VKbvrjxpVJCM/giphy.gif)

## Bencharmark huge lists

```elixir
...
iex(1)> list = Enum.to_list(1..1_000_000)
...
iex(4)> Benchee.run(
...(4)>   %{
...(4)>     "Enum.map" => fn -> Enum.map(list, double_it) end,
...(4)>     "map" => fn -> BenchmarkRecursion.map(list, double_it) end,
...(4)>     "map_tail_call" => fn -> BenchmarkRecursion.map_tail_call(list, double_it) end
...(4)>
...(4)>   }
...(4)> )
...
Comparison:
map_tail_call          1.96
Enum.map               1.73 - 1.13x slower +68.22 ms
map                    1.58 - 1.24x slower +122.09 ms
...
```

Only with huge lists we actually see advantages in using tail call optimization.

![](https://media.giphy.com/media/l3q2DgSFjbAyseViM/giphy.gif)


## So should we always use tail call optimization?

Well, it depends...

![](https://pbs.twimg.com/media/E5xjfo0XsAIENzD?format=jpg&name=900x900)

For instance, in a long-running function, a callback in a `GenServer` where we handle many calls or when we use huge lists, YES.

In the other cases, well, no.

If you check the implementations, the first version is a lot more readable than the second. So first, strive for readability and, only if required, reduce readability for the sake of speed.

But only if it's really needed.

This is actually one of the [miths of erlang](http://erlang.org/doc/efficiency_guide/myths.html).

At the end of the day, it's a decision that should be based on the usage of the system you are building.

For reference:
- [Iteration, Recursion, and Tail-call Optimization in Elixir](https://blog.appsignal.com/2019/03/19/elixir-alchemy-recursion.html)
- [The Seven Myths of Erlang Performance](http://erlang.org/doc/efficiency_guide/myths.html)
- [tail recursion](https://xlinux.nist.gov/dads/HTML/tailRecursion.html)
