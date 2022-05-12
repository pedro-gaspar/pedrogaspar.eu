---
title: Quacks like a list, but it's not a list
date: 2021-07-07
description: "This will happen someday"
image: images/posts/til-improper-lists.png
images:
  - images/posts/til-improper-lists.png
tags:
  - Elixir
  - Today I Learned
---

![](https://media.giphy.com/media/12rqBPpSz9jIA0/giphy.gif)

Elixir lists allow you to group lists of elements. Even though we see them in `iex` in this way:

```elixir
list = [1, 2]
```

It almost looks like an array in other languages, but it is not. It's a **linked list**.

Each list item only contains its value and a pointer to the next element in the list. And the last element is an empty list.

So the previous list, in fact under the hoods, is built like this:

```elixir
list = [1 | [2 | []]]
```

It makes sense to make it look like an array, or else this is a bit unreadable.

One thing that can happen is that we can mess up and don't construct the list correctly.

```elixir
iex(1)> list = [1 | 2]
[1 | 2]
```

In this case, the last element is not an empty list, so this ends up being an **improper list**.

> It looks like a duck...

![](https://media.giphy.com/media/H3MXq3XT4z2ec/giphy.gif)

```elixir
iex(2)> is_list(list)
true
```

> It quacks like a duck...

![](https://media.giphy.com/media/eRYiZlDanh2ta/giphy.gif)

```elixir
iex(3)> [head|tail] = list
[1 | 2]
iex(4)> head
1
iex(5)> tail
2
```

> But it's not a duck.

![](https://media.giphy.com/media/LPHXLKEOZw6T6/giphy.gif)

```elixir
iex()> length(list)
** (ArgumentError) errors were found at the given arguments:
  * 1st argument: not a list

    :erlang.length([1 | 2])
```

One day this will happen for sure.
