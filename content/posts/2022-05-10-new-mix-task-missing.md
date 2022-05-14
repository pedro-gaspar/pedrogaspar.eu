---
title: "Draft: Missing mix task"
date: 2022-05-10
description: "Why sometimes a new mix task doesn't show when running `mix help`"
image: images/posts/elixir-streams.png
images:
  - images/posts/elixir-streams.png
tags:
  - Elixir
  - Today I Learned
---

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
