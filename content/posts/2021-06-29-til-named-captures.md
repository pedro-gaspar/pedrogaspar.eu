---
title: Named captures in Elixir Regular Expressions
date: 2021-06-29
description: "It's a lot better to reference Elixir captures by name instead of index position."
image: images/posts/til-named-captures.png
images:
  - images/posts/til-named-captures.png
tags:
  - Elixir
  - Today I Learned
---

{{< alert "secondary" >}}
It's a lot better to reference Elixir captures by name instead of index position.
{{< /alert >}}

![](https://media.giphy.com/media/6uPOdgLIuhHYQ/giphy.gif)

When doing an exercise in Exercism, I learned something cool with regular expressions.

Sometimes you want to fetch parts of the string, so you put brackets in the regular expression.

```elixir
iex(1)> Regex.run(~r/a(b)c(d)/, "abcde")
["abcd", "b", "d"]
```

You get a list with all the matches. In this case, you also get a match for the whole matched regex.

That usually is not that useful, so you can skip it with `:all_but_first`.

```elixir
iex(2)> Regex.run(~r/a(b)c(d)/, "abcde", capture: :all_but_first)
["b", "d"]
```

Much better 😊.

Now you can pattern match:

```elixir
iex(3)> [first, second] = Regex.run(~r/a(b)c(d)/, "abcde", capture: :all_but_first)
["b", "d"]
iex(4)> first
"b"
iex(5)> second
"d"
```

But later on, you can perhaps do a slight change and include a new match:

```elixir
iex(6)> [first, second] = Regex.run(~r/(a)(b)c(d)/, "abcde", capture: :all_but_first)
** (MatchError) no match of right hand side value: ["a", "b", "d"]
```

The way to fix it is by adding a new variable, but:

```elixir
iex(7)> [first, second, third] = Regex.run(~r/(a)(b)c(d)/, "abcde", capture: :all_but_first)
["a", "b", "d"]
iex(8)> first
"a"
iex(9)> second
"b"
iex(10)> second
"d"
```

Well, the variables `first` and `second` are now capturing another thing.

To fix it you could do this:

```elixir
iex(11)> [third, first, second] = Regex.run(~r/(a)(b)c(d)/, "abcde", capture: :all_but_first)
["a", "b", "d"]
```

But everyone will look at you with crazy eyes 🙄🤬

> If you need it, give it a name. 😄

You can use something called **named captures**.

When you open a bracket for a match, add `?<name_of_the_match>` with the name of the match.

```elixir
iex(12)> Regex.named_captures(~r/a(?<first>b)c(?<second>d)/, "abcde")
%{"first" => "b", "second" => "d"}
```

Pretty neat, right?

![](https://media.giphy.com/media/yjN7s3fOXtdimCTPrk/giphy.gif)

For reference: [Named Captures with Elixir Regular Expressions](https://til.hashrocket.com/posts/d75339a700-named-captures-with-elixir-regular-expressions)
