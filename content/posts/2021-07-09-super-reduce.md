---
title: Super reduce
date: 2021-07-09
description: "An important function in your tool belt that allows you to build pretty much anything. Let's implement some of `Enum` functions with reduce."
image: images/posts/super-reduce.png
images:
  - images/posts/super-reduce.png
tags:
  - Elixir
---

{{< alert "secondary" >}}
An important function in your tools belt that allows you to build pretty much anything. Let's implement some of `Enum` functions with reduce.
{{< /alert >}}

![](https://media.giphy.com/media/xT9DPErFASWkKCaAJW/giphy.gif)

**Enum** is a fantastic module. It offers you tools for almost any possible situation involving enumerating a collection.

But instead of just standing upon the shoulders of giants, it's good to know how things work under the hood.

Let's implement some of **Enum** ourselves using plain old Elixir without the help of any module.

## map

![](https://media.giphy.com/media/qVID3J8fLrlZK/giphy.gif)

Let's create a function that allows us to double each value of a list:

```elixir
defmodule HumbleEnum do
  def double([head|tail]), do: [head * 2 | double(tail)]
  def double([]), do: []
end

iex(1)> HumbleEnum.double([1,2,3])
[2, 4, 6]
```

Cool right.

Now let's do one to triple each value of a list:

```elixir
defmodule HumbleEnum do
  def triple([head|tail]), do: [head * 3 | triple(tail)]
  def triple([]), do: []
end

iex(2)> HumbleEnum.triple([1,2,3])
[3, 6, 9]
```

Best lines of code, ever ever seen. 😏

Now let's do a function to check if each value of a list is even:

```elixir
defmodule HumbleEnum do
  def even([head|tail]), do: [(rem(head, 2) == 0) | even(tail)]
  def even([]), do: []
end

iex(3)> HumbleEnum.even([1,2,3])
[3, 6, 9]
```

![](https://media.giphy.com/media/a5viI92PAF89q/giphy.gif)

Hmmm, similar, right?
We are enumerating all elements of a list and doing some calculations on them.
Let's write **map** with this in mind.

```elixir
defmodule HumbleEnum do
  def map([head|tail], fun), do: [fun.(head) | map(tail, fun)]
  def map([], _fun), do: []
end
```

Well, let's do the same that the previous specialized functions did, but using our humble **map**.

```elixir
list = [1, 2, 3]

HumbleEnum.map(list, fn n -> n * 2 end)
[2, 4, 6]

HumbleEnum.map(list, fn n -> n * 3 end)
[3, 6, 9]

HumbleEnum.map(list, fn n -> div(n, 2) == 0 end)
[true, false, false]
```

And it is working. 🥳

![](https://media.giphy.com/media/4UJyRK2TXNhgk/giphy.gif)

## count

Now let's count. This one is pretty simple. We just need to enumerate all elements of the list, summing 1 for each of the elements.

```elixir
defmodule HumbleEnum do
  def count([]), do: 0
  def count([_ | tail]), do: 1 + count(tail)
end

iex(1)> HumbleEnum.count([1, 2, 3])
3
```

## reverse

Let's switch orders just for fun. We could do it using `++`

```elixir
defmodule HumbleEnum do
  defp reverse([head | tail]), do: reverse(tail) ++ [head]
  defp reverse([]), do: []
end

iex(1)> HumbleEnum.reverse([1, 2, 3])
[3, 2, 1]
```

But that is a bit inefficient. We need to always transverse the first list to concat to the second list.

![](https://media.giphy.com/media/10DRaO76k9sgHC/giphy.gif)

Let's introduce an accumulator to help us out.

```elixir
defmodule HumbleEnum do
  def reverse(l), do: do_reverse(l, [])
  defp do_reverse([head | tail], reversed), do: do_reverse(tail, [head | reversed])
  defp do_reverse([], reversed), do: reversed
end

iex(1)> HumbleEnum.reverse([1, 2, 3])
[3, 2, 1]
```

Working like a charm 👌

## reduce

And now for the mighty powerful reduce.

```elixir
defmodule HumbleEnum do
  def reduce([head | tail], acc, f), do: reduce(tail, f.(head, acc), f)
  def reduce([], acc, _), do: acc
end
```

It receives the list, the initial state, and a function in the following format:

```elixir
fn element, acc -> ... do your magic end
```

Now let's see it in action to sum all the elements of a list, similar to what **Enum.sum** does:

```elixir
iex(1)> HumbleEnum.reduce([1, 2, 3], 0, fn n, acc -> n + acc end)
6
```

**reduce** is so powerful that we can build all functions of **Enum** using it.

## map implemented with _reduce_

```elixir
defmodule HumbleEnum do
  def reduce([head | tail], acc, f), do: reduce(tail, f.(head, acc), f)
  def reduce([], acc, _), do: acc

  def map(list, fun) do
    list |> reduce([], fn elem, mapped -> [fun.(elem) | mapped] end)
  end
end

iex(1)> HumbleEnum.map([1, 2, 3], & &1)
[3, 2, 1]
```

It almost works, but the order is not correct...

Let's fix it 🤔

We need to reverse the order of the resulting list, so we need **reverse**, but built with _reduce_.

![](https://media.giphy.com/media/11t0FcPMn6mjL2/giphy.gif)

## reverse implemented with _reduce_

```elixir
defmodule HumbleEnum do
  def reduce([head | tail], acc, f), do: reduce(tail, f.(head, acc), f)
  def reduce([], acc, _), do: acc

  def reverse(list) do
    list |> reduce([], fn elem, reversed -> [elem | reversed] end)
  end
end

iex(1)> HumbleEnum.reverse([1, 2, 3])
[3, 2, 1]
```

Now let's fix **map**.

## map implemented with _reduce_ v2

```elixir
defmodule HumbleEnum do
  def map(list, fun) do
    list
    |> reduce([], fn elem, mapped -> [fun.(elem) | mapped] end)
    |> reverse()
  end
end

iex(1)> HumbleEnum.map([1, 2, 3], & &1)
[1, 2, 3]
```

Pretty cool, right?

Sad that we have to go over the same list twice, first to enumerate with the function and then reverse.

We will come back to this later. 🤞

![](https://media.giphy.com/media/pPqxV1MVzfqes/giphy.gif)

## count implemented with _reduce_

```elixir
defmodule HumbleEnum do
  def count(list) do
    list |> reduce(0, fn _, total -> total + 1 end)
  end
end

iex(1)> HumbleEnum.count([1, 2, 3])
3
```

Cool, we have all the functions we built previously implemented with reduce.

## reduce can have an order

In Exercism, the mentor pointed out that reversing the result is fine, but **reduce** can have an order. 🙏

We are doing the left to right version:

```elixir
reduce([1,2,3], acc, f) = f(3, f(2, f(1, acc)))
```

But we could also do the right to left version:

```elixir
reduce_right([1,2,3], acc, f) = f(1, f(2, f(3, acc)))
```

![](https://media.giphy.com/media/z7eor89n5Qv60/giphy.gif)

Mindblown, right? Let's build a **reduce_right** and change our **map**.

## map _reduce_ version v3

```elixir
defmodule HumbleEnum do
  def reduce_right([head | tail], acc, f), do: f.(head, reduce_right(tail, acc, f))
  def reduce_right([], acc, _), do: acc

  def map(list, fun) do
    list |> reduce_right([], fn elem, mapped -> [fun.(elem) | mapped] end)
  end
end

iex(1)> HumbleEnum.map([1, 2, 3], & &1)
[1, 2, 3]
```

Elixir and functional programming rock, right?

![](https://media.giphy.com/media/wLDXxrBcH4FPO/giphy.gif)

**reduce** has superpowers, so better start using it and be a hero 💪
