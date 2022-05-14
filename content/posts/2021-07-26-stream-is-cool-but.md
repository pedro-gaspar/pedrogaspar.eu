---
title: Stream is cool, but...
date: 2021-07-26
description: "Use it wisely. For some use cases, it is a lot worse."
image: images/posts/elixir-streams.png
images:
  - images/posts/elixir-streams.png
tags:
  - Elixir
---

{{< alert "secondary" >}}
{{< /alert >}}

![](https://media.giphy.com/media/xT5LMFVIKfCsH0rkoo/giphy.gif)

When you come to Elixir with big object-oriented baggage, you try to map some of your solutions to what you previously did. Let's try to build a list on even numbers, for instance using **javascript**:

```javascript
let even = [];
for (let i = 0; i < 2000; i += 2) {
  even.push(i);
}
```

Now let's try to do the same with Elixir. The first attempt, since I used `for` in javascript, let's use a list comprehension:

```elixir
for n <- 0..2000, rem(n, 2) == 0, do: n
```

Seems kind of dumb to have to do the filter.

Wait, we have that shinny [Stream](https://hexdocs.pm/elixir/1.12/Stream.html) that allows you to build infinite lists.

Stream is very cool. Due to their laziness, streams are helpful when working with large (or even infinite) lists.

```elixir
some_big_list |> Stream.map(...) |> Stream.map(...) |> Enum.sum()
```

For pipelines where you do lots of transformations it will for sure be helpful. Let's use it here.

```elixir
Stream.iterate(0, &(&1 + 2)) |> Stream.take_while(&(&1 <= 2000)) |> Enum.to_list()
```

![](https://media.giphy.com/media/NaboQwhxK3gMU/giphy.gif)

Easy peasy, lemmon squeezy. Now let's try it with a really big list:

> But at what cost?

```elixir
Stream.iterate(0, &(&1 + 2)) |> Stream.take_while(&(&1 <= 2000000)) |> Enum.to_list()
```

Weird, it seemed a bit slow. So let's make it even more significant.

![](https://media.giphy.com/media/vMbC8xqhIf9ny/giphy.gif)

```elixir
Stream.iterate(0, &(&1 + 2)) |> Stream.take_while(&(&1 <= 20000000)) |> Enum.to_list()
```

For our use case, it looks slow, really slow. How slow, you might ask. So instead of following gut feelings, let's benchmark it.

```sh
$ mix new big_list
```

Let's add [Benchee](https://github.com/bencheeorg/benchee) to `mix.exs`. And then build the even list in different ways to get a comparison of speed.

First let's use our comprehension:

```elixir
def using_comprehension(number), do: for(n <- 0..number, rem(n, 2) == 0, do: n)
```

Now let's try using `Enum`. Should be similar in speed 🤔:

```elixir
def using_enum(number), do: 0..number |> Enum.filter(&(rem(&1, 2) == 0))
```

Let's try to do it better by using a **range**:

```elixir
 def using_range(number), do: 0..number//2 |> Enum.to_list()
```

Next using our faithful **recursion**:

```elixir
  def using_recursion(number), do: using_recursion(number, 0)

  def using_recursion(number, n) when n <= number,
    do: [n | using_recursion(number, n + 2)]

  def using_recursion(number, n) when n > number, do: []
```

And check with **tail call optimization**:

```elixir
def using_tail_recursion(number), do: using_tail_recursion(number, 0, [])

def using_tail_recursion(number, n, even_numbers) when n <= number,
  do: using_tail_recursion(number, n + 2, [n | even_numbers])

def using_tail_recursion(number, n, even_numbers) when n > number,
  do: Enum.reverse(even_numbers)
```

And finally, with our **stream**:

```elixir
  def using_stream(number) do
    Stream.iterate(0, &(&1 + 2)) |> Stream.take_while(&(&1 <= number))
  end
```

So let's benchmark it:

![](https://media.giphy.com/media/p0FeUCcB2IrPq/giphy.gif)

```elixir
def benchmark(n \\ 200) do
  Benchee.run(%{
    "using_comprehension" => fn -> using_comprehension(n) end,
    "using_range" => fn -> using_range(n) end,
    "using_enum" => fn -> using_enum(n) end,
    "using_recursion" => fn -> using_recursion(n) end,
    "using_tail_recursion" => fn -> using_tail_recursion(n) end,
    "using_stream" => fn -> using_stream(n) |> Enum.to_list() end
  })
end
```

So the results are ...

![](https://media.giphy.com/media/yN42S62NnoLJGZxgQs/giphy-downsized-large.gif)

```elixir
iex(1)> BigList.benchmark(200)
...
Comparison:
using_recursion          2002.69 K
using_tail_recursion     1634.62 K - 1.23x slower +0.112 μs
using_range                713.44 K - 2.81x slower +0.90 μs
using_comprehension        277.10 K - 7.23x slower +3.11 μs
using_enum                 192.23 K - 10.42x slower +4.70 μs
using_stream              153.70 K - 13.03x slower +6.01 μs
```

![](https://media.giphy.com/media/xThta2S6BM1yIzVHqw/giphy.gif)

Our recursion function is by far the fastest, and followed by its tail optimization version 😏. The range version is pretty quick and a big surprise. The comprehension is faster than the enum version. Since they kind of do the same, I expected them to be similar, but it's quite a difference. And then down the bottom our slow stream 😿.

Let's try with a bigger list to see if it changes our benchmark.

```elixir
iex(1)> BigList.benchmark(2000)
...
Comparison:
using_recursion           248.05 K
using_tail_recursion      188.93 K - 1.31x slower +1.26 μs
using_range                 76.89 K - 3.23x slower +8.97 μs
using_comprehension         27.33 K - 9.08x slower +32.56 μs
using_enum                  20.69 K - 11.99x slower +44.30 μs
using_stream               15.20 K - 16.32x slower +61.77 μs
```

Nothing changed all that much. In comparison, the results are a lot worse compared to the recursion version. Now let's try with a huge list.

![](https://media.giphy.com/media/nM6H7dozprJa8/giphy.gif)

```elixir
iex(1)> BigList.benchmark(2000000)
...
Comparison:
using_tail_recursion         66.02
using_range                   43.65 - 1.51x slower +7.76 ms
using_recursion              36.93 - 1.79x slower +11.93 ms
using_comprehension           22.74 - 2.90x slower +28.82 ms
using_enum                    17.42 - 3.79x slower +42.27 ms
using_stream                  9.97 - 6.62x slower +85.11 ms
```

Tail recursion optimization starts to really optimize 😏. And all other versions are not as bad compared to the previous one.
Stream seems to be slow as hell compared to all other versions, regardless of the list size.

Now let's change the use case. We will sum all the even numbers that have the lucky number 7 on them.

![](https://media.giphy.com/media/Nx85vtTY70T3W/giphy.gif)

```elixir
  defp lucky_n7?(n), do: n |> Integer.digits() |> Enum.member?(7)
  def benchmark(n \\ 200) do
    Benchee.run(%{
      "using_range" => fn -> using_range(n) |> Enum.filter(&lucky_n7?/1) |> Enum.sum() end,
      "using_comprehension" => fn -> using_comprehension(n) |> Enum.filter(&lucky_n7?/1) |> Enum.sum() end,
      "using_enum" => fn -> using_enum(n) |> Enum.filter(&lucky_n7?/1) |> Enum.sum() end,
      "using_recursion" => fn -> using_recursion(n) |> Enum.filter(&lucky_n7?/1) |> Enum.sum() end,
      "using_tail_recursion" => fn -> using_tail_recursion(n) |> Enum.filter(&lucky_n7?/1) |> Enum.sum() end,
      "using_stream" => fn -> using_stream(n) |> Stream.filter(&lucky_n7?/1) |> Enum.sum() end
    })
  end
```

Let's start with a small list first.

```elixir
iex(1)> BigList.benchmark(200)
...
Comparison:
using_recursion           199.61 K
using_tail_recursion      195.24 K - 1.02x slower +0.112 μs
using_range                168.20 K - 1.19x slower +0.94 μs
using_comprehension        123.23 K - 1.62x slower +3.11 μs
using_stream              105.70 K - 1.89x slower +4.45 μs
using_enum                 103.47 K - 1.93x slower +4.66 μs
```

We end up with similar results as before. Increasing the list a bit:

```elixir
iex(1)> BigList.benchmark(2000)
...
Comparison:
using_recursion            16.03 K
using_tail_recursion       14.81 K - 1.08x slower +5.13 μs
using_range                 13.39 K - 1.20x slower +12.26 μs
using_comprehension         10.20 K - 1.57x slower +35.62 μs
using_enum                   8.88 K - 1.81x slower +50.26 μs
using_stream                8.37 K - 1.92x slower +57.13 μs
```

Even though the order is the same, there isn't such a massive difference between versions as it did in the benchmark for even numbers.
But now, let's go to the humongous list.

```elixir
iex(1)> BigList.benchmark(2000000)
...
Comparison:
using_stream                  6.49
using_tail_recursion          4.29 - 1.51x slower +78.98 ms
using_recursion               3.56 - 1.82x slower +126.46 ms
using_comprehension            3.13 - 2.07x slower +165.63 ms
using_enum                     3.04 - 2.13x slower +174.33 ms
using_range                    2.90 - 2.24x slower +191.28 ms
```

![](https://media.giphy.com/media/l4q7VhGsL6BnXJrc4/giphy.gif)

Stream starts to really shine. Not being eager and processing results, one at a time, makes a huge difference with large lists and more complicated pipelines.

When I started doing this little investigation, I wasn't expecting some findings. First, Enum with filter is a lot slower than a comprehension with a filter. I was expecting it to be very similar. Need to find out why.

Streams are better suited for larger pipelines and aren't the best tool for many use cases.
Like everything in engineering, it's a matter of choosing the most cost-effective, fastest and elegant solution.

And at the end... You need to choose wisely.

![](https://media.giphy.com/media/ZgYBhq1x7L1bW/giphy.gif)
