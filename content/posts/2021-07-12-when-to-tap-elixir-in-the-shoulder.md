---
title: When to tap Elixir in the shoulder
date: 2021-07-12
description: "Whenever you can, there is no try."
image: images/posts/baby-yoda.png

images:
  - images/posts/baby-yoda.png
tags:
  - Elixir
---

![](https://media.giphy.com/media/f9jfbOcTBLBAFXIABe/giphy.gif)

Big features like LiveView or LiveBook are amazing and prove that the ecosystem is growing a lot, and that's great. But the small little improvements sometimes are the best proof that there is love to the language, even in the smallest details.

While doing some Exercism exercises, sometimes I had to stop doing a pipeline to get some value or change the order of some tuple. In the end, it made me feel sad because the flow of the pipeline had to be interrupted, just because...

Let's see an example, we want the result of a function to be a sum of filtered version of persons that like starwars:

```elixir
defmodule Starwars do
  def fans(persons) do
    fans_count =
      persons
      |> Enum.reject(&(&1.favourite_movie == "Star Trek"))
      |> Enum.filter(& &1.likes_yoda)
      |> Enum.count()

    {:ok, fans_count, Enum.count(persons)}
  end
end

persons = [
  %{name: "Pedro", favourite_movie: "The Empire Strikes Back", likes_yoda: true},
  %{name: "Regina", favourite_movie: "The Phantom Menace", likes_yoda: false},
  %{name: "Filipe", favourite_movie: "Return of the Jedi", likes_yoda: true},
  %{name: "Maria", favourite_movie: "Star Trek", likes_yoda: false}
]

iex(1)> Starwars.fans(persons)
{:ok, 2, 4}
```

We had to add a variable `fans_count` just to return the tuple with the agreed format. Seems like it could be better. It doesn't seem very idiomatic.

And we are also wondering if the fans count should be higher. Maybe it's a bug in a filter, so we do a quick `IO.inspect` between filters.

```elixir
defmodule Starwars do
  def fans(persons) do
    fans_count =
      persons
      |> Enum.reject(&(&1.favourite_movie == "Star Trek"))
      |> IO.inspect()
      |> Enum.filter(& &1.likes_yoda)
      |> Enum.count()

    {:ok, fans_count, Enum.count(persons)}
  end
end

iex(2)> Starwars.fans(persons)
[
  %{favourite_movie: "The Empire Strikes Back", likes_yoda: true, name: "Pedro"},
  %{favourite_movie: "The Phantom Menace", likes_yoda: false, name: "Regina"},
  %{favourite_movie: "Return of the Jedi", likes_yoda: true, name: "Filipe"}
]
{:ok, 2, 4}
```

Meeh, I just wanted to check the `likes_yoda value, but I'm getting everything...

![](https://media.giphy.com/media/WkOAurEV1T42tCq5VF/giphy.gif)

## The old way(s)

Let's try to fix these two issues the old way. Well, it was the way I knew. 😅

```elixir
defmodule Starwars do
  def fans(persons) do
      persons
      |> Enum.reject(&(&1.favourite_movie == "Star Trek"))
      |> IO.inspect()
      |> Enum.filter(& &1.likes_yoda)
      |> Enum.count()
      |> case do
        result -> {:ok, fans_count, Enum.count(persons)}
      end
  end
end
```

**`case`** can help in this case (pun not intended), and we don't need extra variables. The work is all done in a pipeline flow of data transformations.

Beautiful? 🤔

![](https://media.giphy.com/media/j6sijUUfTW2XL4KUNu/giphy.gif)

Not that much, right...

If we needed to have multiple evaluations of the result, that would be a good fit, something like:

```elixir
      |> Enum.count()
      |> case do
        0 -> {:ko, "No fans, no party"}
        result -> {:ok, result, Enum.count(persons)}
      end
```

Let's try something else:

```elixir
defmodule Starwars do
  def fans(persons) do
      persons
      |> Enum.reject(&(&1.favourite_movie == "Star Trek"))
      |> IO.inspect()
      |> Enum.filter(& &1.likes_yoda)
      |> Enum.count()
      |> (fn result -> {:ok, result, Enum.count(persons)} end).()
  end
end
```

This looks a bit better; you are doing an anonymous function to do the transformation, that sounds ok.

But this kind of reminded me of `(function($) { })(jQuery);`. The weird links I have in my head. 😜

But well, it was the best way, the old way.

Now for improving the debugging.

```elixir
defmodule Starwars do
  def fans(persons) do
      persons
      |> Enum.reject(&(&1.favourite_movie == "Star Trek"))
      |> Enum.map(fn p ->
        IO.puts(p.likes_yoda)
        p
      end)
      |> Enum.filter(& &1.likes_yoda)
      |> Enum.count()
      |> (fn result -> {:ok, result, Enum.count(persons)} end).()
  end
end

iex(3)> Starwars.fans(persons)
true
false
true
{:ok, 2, 4}
```

That prints only the part of data that I wanted to check, the persons that like Yoda. But after doing the quick print, I also have to return the person. Otherwise, the map would change the persons to check.

![](https://media.giphy.com/media/3jVlAzkbvVRfRPsThL/giphy.gif)

## The new way

Elixir grows and improves to better fit common use cases. For example, in version 1.12, we got some goodies called **`tap`** and **`then`**.

Let's see how they can make our life easier in these cases:

```elixir
defmodule Starwars do
  def fans(persons) do
      persons
      |> Enum.reject(&(&1.favourite_movie == "Star Trek"))
      |> Enum.map(fn p -> tap(p, & IO.puts(&1.likes_yoda)) end)
      |> Enum.filter(& &1.likes_yoda)
      |> Enum.count()
      |> then(& {:ok, &1, Enum.count(persons)})
  end
end
```

![](https://media.giphy.com/media/Ld77zD3fF3Run8olIt/giphy.gif)

Pretty neat, right? ♥

**`tap`** is useful for side effects during the pipeline where you want to do your side effect but return the exact same value if received as an argument:

```elixir
tap(20, fn _argument -> 0 end) == 20
```

and

**`then`** lets you run a function on the argument. And with Elixir, you can pattern match to that is a great new tool.

```elixir
almost_yoda = ["There is no try.", "Do or do not"]
tap(almost_yoda, fn [a, b] -> [b, a] end)
```

> God is in the details

And we have the Elixir version of that.

> José Valim is in the details 😄

![](https://media.giphy.com/media/KFhwTLFngMTd3GDbd3/giphy.gif)
