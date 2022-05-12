---
title: Installing Elixir in your local machine
date: 2021-07-04
description: "ASDF FTW"
images:
- images/featured/elixir-tutorial-install-elixir.png
tags:
  - Elixir
---

The first thing you need to start hacking with Elixir is ... well ... Elixir 😏.

If you want to install it on your local machine, there are plenty of options. You can see them in the Elixir [docs](https://elixir-lang.org/install.html).

I prefer to use a version manager, similar to _rvm_ or _nvm_.

Because of Elixir, I came across **[asdf](https://asdf-vm.com/#/)**, and it is awesome 🤯.

It's the last version manager you will ever need.

That's because you use the same version manager for different languages. You usually will also need node.js installed anyways. Or even a Postgres database.

You can set up everything in one place. And you can even have different versions installed and have a `.tool_versions` file specifying which versions are used when you are inside a directory.

To install it, just follow the [instructions](https://asdf-vm.com/#/core-manage-asdf). In my case, I'll be installing in a Ubuntu machine, but the steps are pretty similar for Mac, except installing the required dependencies.

## Install git

```sh
$ sudo apt install curl git
```

## Clone the asdf repo

```sh
$ git clone https://github.com/asdf-vm/asdf.git ~/.asdf
$ cd ~/.asdf
$ git checkout "$(git describe --abbrev=0 --tags)"
```

## Add to your shell

In my case to `.zshrc`, append the following line:

```sh
. $HOME/.asdf/asdf.sh
```

## Enable completions

In `.zshrc` include the _asdf_ plugin to enable completions:

```sh
plugins=(
  asdf
  git
)
```

## Reload the shell

```sh
$ source ~/.zshrc
```

## Install Erlang

You have a list of [available plugins](https://asdf-vm.com/#/plugins-all), with instructions on installing required dependencies for a specific language.

For Elixir you will need to install Erlang first. First you install the required system dependencies for Erlang to work:

```sh
$ sudo apt-get -y install build-essential autoconf m4 libncurses5-dev libwxgtk3.0-gtk3-dev libgl1-mesa-dev libglu1-mesa-dev libpng-dev libssh-dev unixodbc-dev xsltproc fop libxml2-utils libncurses-dev openjdk-11-jdk
```

Then you install the plugin:

```sh
$ asdf plugin add erlang
```

Now you can query for all available versions you can install:

```sh
$ asdf list-all erlang
```

Pick the latest or a previous one:

```sh
$ asdf install erlang 24.0.2
```

Then you can set it as the global version used in your system:

```sh
$ asdf global erlang 24.0.2
```

Or inside a directory, mark a specific version inside it:

```sh
$ asdf local erlang 24.0.2
```

## Install Elixir

The steps are pretty similar to erlang, as with any other language.

You don't need to install any additional system dependency, so you first add the plugin.

```sh
$ asdf plugin add elixir
```

Now you can query for all available versions:

```sh
$ asdf list-all elixir
```

Pick the latest:

```sh
$ asdf install elixir 1.12.1
```

Then you can set it as the global version used in your system:

```sh
$ asdf global elixir 1.12.1
```

## Test it

```sh
$ elixir -v
Erlang/OTP 24 [erts-12.0.2] [source] [64-bit] [smp:12:12] [ds:12:12:10] [async-threads:1] [jit]

Elixir 1.12.1 (compiled with Erlang/OTP 24)
```

## Have fun

![](https://media.giphy.com/media/43wsmkuWDuNQX78KwS/giphy.gif)
