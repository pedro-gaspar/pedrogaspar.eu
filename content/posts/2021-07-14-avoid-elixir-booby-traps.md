---
title: Avoid Elixir booby traps
date: 2021-07-14
description: "Coming from an object-oriented background, you will fall into some Elixir booby traps."
image: images/posts/traps.png
images:
  - images/posts/traps.png
tags:
  - elixir
---

{{< alert "secondary" >}}
{{< /alert >}}

Coming from an object-oriented background, you will fall into some Elixir booby traps.

I'll highlight some common ones where, I spent a few minutes looking at the screen, looking like this:

![](https://media.giphy.com/media/toB3AnUDkqE3GENKx0/giphy.gif)

It is pretty helpful to get familiar with error messages because you will see them a lot, especially during the initial days.

## Single quotes aren't strings

In many languages you are used to use both double and single quotes. They both represent a string.
So you will probably do something like:

```elixir
iex(1)> "123" <> '456'
** (ArgumentError) expected binary argument in <> operator but got: '456'
    (elixir 1.12.1) lib/kernel.ex:1893: Kernel.wrap_concatenation/3
    (elixir 1.12.1) lib/kernel.ex:1884: Kernel.extract_concatenations/2
    (elixir 1.12.1) lib/kernel.ex:1880: Kernel.extract_concatenations/2
    (elixir 1.12.1) expanding macro: Kernel.<>/2
    iex:11: (file)

iex(2)> '123' <> '456'
** (ArgumentError) expected binary argument in <> operator but got: '123'
    (elixir 1.12.1) lib/kernel.ex:1893: Kernel.wrap_concatenation/3
    (elixir 1.12.1) lib/kernel.ex:1880: Kernel.extract_concatenations/2
    (elixir 1.12.1) expanding macro: Kernel.<>/2
    iex:12: (file)

iex(3)> String.reverse('123')
** (FunctionClauseError) no function clause matching in String.reverse/1

    The following arguments were given to String.reverse/1:

        # 1
        '123'

    Attempted function clauses (showing 1 out of 1):

        def reverse(string) when is_binary(string)

    (elixir 1.12.1) lib/string.ex:1585: String.reverse/1
```

![](https://media.giphy.com/media/wIxBzHWegpOUM/giphy.gif)

When you have single quotes in Elixir you are using a **char list**. The name implies what it is, a list of chars.

```elixir
iex(1)> '123'
'123'
iex(2)> [?1, ?2, ?3]
'123'
iex(3)> ?1
49
iex(4)> is_list('123')
true
```

It's a list of integers, more concretely Unicode codepoints.

```elixir
iex(1)> ?1
49 # codepoint for char 1
```

A string on the other and is a **binary**. We are not used to this term related to string, coming from other languages. Since Elixir uses the Erlang VM, you don't have strings as you are used. Only binary representations of strings.

```elixir
iex(1)> "123"
"123"
iex(2)> is_binary("123")
```

The best thing to do is always use double quotes and only resort to single quotes if you really need them.

# Lists with small integers

```elixir
iex(39)> [10]
'\n'
iex(40)> [112]
'p'
iex(41)> [10, 112]
'\np'
```

![](https://media.giphy.com/media/2XskdWuNUyqElkKe4bm/giphy.gif)

In **mix**, Elixir tries to infer the type. And if it quacks like a char list, it will show a char list. If the integer is bigger than the possible codepoints, you won't see it as a char list because there is no associated code points.

```elixir
iex(42)> [100000]
[100000]
```

## Changing variables inside if statements

You are used to doing something like this.

```elixir
x = 1
y = 1

if (x == 1) do
  x = 2
  y = 3
else
  x = 4
  y = 5
end
```

You expect the obvious right?

```elixir
iex(1)> x
1
iex(2)> y
1
```

![](https://media.giphy.com/media/pPhyAv5t9V8djyRFJH/giphy.gif)

If you declare or change any variable inside an **if**, **unless**, **cond**, **case**, and similar constructs, the variable, and any change you make, will only be visible inside it.

If you want to change some value inside the **if** statement, you have to return it.

```elixir
x = 1
y = 1

{x, y} = if (x == 1) do
  x = 2
  y = 3
  {x, y}
else
  x = 4
  y = 5
  {x, y}
end

iex(1)> x
2
iex(2)> y
3
```

This is a pretty good language decision. It enables you to refactor your code with more confidence.

## Variables rebind when pattern matched

Variables are kind of left-handed. They are very _"dynamic"_, and change value a lot on the left side of the **=** and pretty _"static"_ on the right side.

```elixir
iex(1)> x = 5
1
iex(2)> y = 1
1
iex(3)> {w, y} = {3, x}
{3, 5}
iex(4)> w
3
iex(5)> y
5
```

![](https://media.giphy.com/media/CiYImHHBivpAs/giphy.gif)

You could think that the **y** would keep its value, but we are on the left side, so it tries to pattern match if possible.

And it's not only the left side of **=**. It's on any place where there pattern matching happens.

```elixir
y = "1"
x = "2"

case y do
  x -> "wtf"
  _ -> "can't touch me."
end
```

Exactly the same behavior as when you are on the left side of the **`=`**

```elixir
iex(25)> x = "1"
"1"
iex(26)> y = "2"
"2"
iex(27)> case y do
...(27)>   x -> "wtf"
...(27)>   _ -> "can't touch me"
...(27)> end
warning: variable "x" is unused (there is a variable with the same name in the context, use the pin operator (^) to match on it or prefix this variable with underscore if it is not meant to be used)
  iex:28

"wtf"
```

![](https://media.giphy.com/media/uN5iwZB2v2dH2/giphy.gif)

Check the error message. Elixir kindly says:

> Are you sure you didn't 💩?

If you don't want the variable to change, use the pin operator **^x**

```elixir
case y do
  ^x -> "wtf"
  _ -> "now it works!"
end
```

is similar to

```elixir
case y do
  "1" -> "wtf"
  _ -> "now it works"
end
```

## Maps aren't objects

If you use dicts in python you will probably do something like this:

```elixir
iex(1)> x = %{"show_me" => 987}
%{"show_me" => 987}
iex(2)> x.get("show_me")
** (ArgumentError) you attempted to apply a function on %{"show_me" => 987}. Modules (the first argument of apply) must always be an atom
    :erlang.apply(%{"show_me" => 987}, :get, ["show_me"])
```

![](https://media.giphy.com/media/h4Z6RfuQycdiM/giphy.gif)

There are no objects in Elixir. You don't have properties. It may seem like properties when you use atoms as keys. But they aren't properties; they are just keys of maps.

```elixir
iex(1)> x = %{show_me: 987}
%{show_me: 987}
iex(2)> x.show_me
987
```

You only do this for keys that already exist in the map, otherwise:

```elixir
iex(1)> x.something
** (KeyError) key :something not found in: %{show_me: 987}
```

You need to use a function to get the value of a map, or a list:

```elixir
iex(1)> x = %{"show_me" =>  987}
%{"show_me" => 987}
iex(2)> Map.get(x, "show_me")
987

iex(3)> y = [1, 2, 3]
[1, 2, 3]
iex(4)> Enum.at(y, 2)
3
```

## _for_ is not the for you know

Let's do a for loop 3 times and change a map's value.

```elixir
map = %{x: 0}
for _ <- 1..3 do
  map = %{map | x: map.x + 1}
end
```

Simple enough, let's try it.

```elixir
iex(3)> map = %{x: 0}
%{x: 0}
iex(4)> for i <- 1..3 do
...(4)>   map = %{map | x: map.x + 1}
...(4)> end
warning: variable "i" is unused (if the variable is not meant to be used, prefix it with an underscore)
  iex:4

warning: variable "map" is unused (there is a variable with the same name in the context, use the pin operator (^) to match on it or prefix this variable with underscore if it is not meant to be used)
  iex:5

[%{x: 1}, %{x: 1}, %{x: 1}]
iex(5)> map = %{x: 0}
%{x: 0}
iex(6)> for _ <- 1..3 do
...(6)>   map = %{map | x: map.x + 1}
...(6)> end
warning: variable "map" is unused (there is a variable with the same name in the context, use the pin operator (^) to match on it or prefix this variable with underscore if it is not meant to be used)
  iex:7

[%{x: 1}, %{x: 1}, %{x: 1}]
```

![](https://media.giphy.com/media/NITFX5emjpMQ0/giphy.gif)

You expect the map to have changed, right? Think again.

It's lexical scoped inside the **do**, so its value doesn't change.

Whenever you want loops to store state, you will need to resource to **Enum.reduce** or **recursion**.

**Using Enum.reduce**

```elixir
iex(9)> map = %{x: 0}
%{x: 0}
iex(10)> map = 1..3 |> Enum.reduce(map, fn _, acc -> %{acc | x: acc.x + 1} end)
%{x: 3}
```

**Using recursion**

```elixir
defmodule Works do
  def with_recursion() do
    with_recursion(%{x: 0}, 3)
  end

  def with_recursion(map, 0), do: map
  def with_recursion(map, counter) do
    with_recursion(%{map | x: map.x + 1}, counter - 1)
  end
end

iex(13)> Works.with_recursion()
%{x: 3}
```

## Updating maps

Now let's try to change a map value.

```elixir
iex(14)> looks_like_a_python_dict = %{"x": "1"}
warning: found quoted keyword "x" but the quotes are not required. Note that keywords are always atoms, even when quoted. Similar to atoms, keywords made exclusively of ASCII letters, numbers, and underscores do not require quotes
  iex:14

%{x: "1"}
iex(15)> looks_like_a_python_dict["x"] = 1
** (CompileError) iex:15: cannot invoke remote function Access.get/2 inside a match

iex(15)> looks_like_a_python_dict["y"] = 1
** (CompileError) iex:15: cannot invoke remote function Access.get/2 inside a match
```

![](https://media.giphy.com/media/t6WvtUluR8V2NSxLlk/giphy.gif)

This is not an array and not and object.

But there's the right way to do this in Elixir:

```elixir
iex(2)> looks_like_a_python_dict = %{x: "1"}
%{x: "1"}
iex(3)> looks_like_a_python_dict = %{looks_like_a_python_dict | x: 1}
%{x: 1}
iex(4)> looks_like_a_python_dict = Map.put(looks_like_a_python_dict, "y", 1)
%{:x => 1, "y" => 1}
```

## strings are not arrays of chars

Whenever you want a specific char of a string you are used to the index syntax:

```elixir
iex(2)> x_marks_the_spot = ".X......."
".X......."
iex(3)> x_marks_the_spot[1]
** (FunctionClauseError) no function clause matching in Access.get/3

    The following arguments were given to Access.get/3:

        # 1
        ".X......."

        # 2
        1

        # 3
        nil

    Attempted function clauses (showing 5 out of 5):

        def get(%module{} = container, key, default)
        def get(map, key, default) when is_map(map)
        def get(list, key, default) when is_list(list) and is_atom(key)
        def get(list, key, _default) when is_list(list)
        def get(nil, _key, default)

    (elixir 1.12.1) lib/access.ex:283: Access.get/3
```

![](https://media.giphy.com/media/5t9wJjyHAOxvnxcPNk/giphy.gif)

Nope, this doesn't work either.

You have to use the [String](https://hexdocs.pm/elixir/1.12/String.html#at/2) module to help you out.

```elixir
iex(2)> x_marks_the_spot = ".X......."
".X......."
iex(3)> x_marks_the_spot |> String.at(1)
"X"
```

## lists are not arrays

When you ear lists, coming from other languages you will think of arrays. So you will probably try something like:

```elixir
iex(5)> game_scores = [500, 333, 456, 665, 943]
[500, 333, 456, 665, 943]
iex(6)> game_scores[1]
** (ArgumentError) the Access calls for keywords expect the key to be an atom, got: 1
    (elixir 1.12.1) lib/access.ex:310: Access.get/3
```

![](https://media.giphy.com/media/IzvWnlXpUcNXdo4xAq/giphy.gif)

Forget about it. Say it out loud:

> Lists aren't arrays. They are LINKED LISTS.

Once again, you need to ask [Enum](https://hexdocs.pm/elixir/1.12/Enum.html#at/3) for help.

```elixir
iex(1)> game_scores = [500, 333, 456, 665, 943]
[500, 333, 456, 665, 943]
iex(2)> game_scores |> Enum.at(1)
333
```

Leaving your object-oriented comfort zone can be challenging. You will have a lot of WTF moments, but slowly things start clicking and making sense. Don't give up and keep running, or else the object-oriented boulder will catch you again 😱

![](https://media.giphy.com/media/UVqDh08BFyGfrx9tRU/giphy.gif)
