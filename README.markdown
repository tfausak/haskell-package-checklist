# Haskell package checklist

:warning:
This was originally a blog post.
It is very opinionated.
It is not kept up to date.
You may be interested in [this Reddit discussion](https://www.reddit.com/r/haskell/comments/5iok3l/haskell_package_checklist/).
Or [this one](https://www.reddit.com/r/haskell/comments/9dsi2y/an_opinionated_haskell_package_checklist/).

This checklist covers everything you need to know about developing a Haskell package.
The [Cabal user guide](https://cabal.readthedocs.io/en/latest/developing-packages.html#developing-packages) provides good low-level information,
and @sebastiaanvisser's post "[Towards a better Haskell package](http://fvisser.nl/post/2013/may/28/towards-a-better-haskell-package.html)" gives some nice high-level guidance.
This checklist covers both of those and fills in some of the gaps left by them.
It was adapted from @tfausak's post "[Haskell package checklist](http://taylor.fausak.me/2016/12/05/haskell-package-checklist/)".

If you're looking for a starting point that ticks most of these boxes, consider the [Haskeleton](http://taylor.fausak.me/2014/03/04/haskeleton-a-haskell-project-skeleton/) Stack template.
It will give you a good base to start from.
If you're looking for an actual package that follows the guidelines, check out [Rattletrap](http://taylor.fausak.me/2016/11/15/parse-and-generate-rocket-league-replays-with-haskell/), a Rocket League replay parser and generator.
It can show you exactly how some of these things are implemented.

- [Use Git for source control](#use-git-for-source-control)
- [Host on GitHub](#host-on-github)
- [Build with Stack](#build-with-stack)
- [Define with hpack](#define-with-hpack)
- [Name with kebab case](#name-with-kebab-case)
- [Use Semantic Versioning](#use-semantic-versioning)
- [Choose a license](#choose-a-license)
- [Write a README](#write-a-readme)
- [Keep a change log](#keep-a-change-log)
- [Write a synopsis](#write-a-synopsis)
- [Avoid heavy dependencies](#avoid-heavy-dependencies)
- [Include extra source files](#include-extra-source-files)
- [Fix package warnings](#fix-package-warnings)
- [Separate source files from metadata](#separate-source-files-from-metadata)
- [Match package and module names](#match-package-and-module-names)
- [Require one import](#require-one-import)
- [Expose implementation details](#expose-implementation-details)
- [Fix all GHC warnings](#fix-all-ghc-warnings)
- [Follow HLint suggestions](#follow-hlint-suggestions)
- [Format code with Brittany](#format-code-with-brittany)
- [Write documentation with examples](#write-documentation-with-examples)
- [Test with Hspec](#test-with-hspec)
- [Run tests on Travis CI](#run-tests-on-travis-ci)
- [Keep executables small](#keep-executables-small)
- [Benchmark with Criterion](#benchmark-with-criterion)
- [Automate releases](#automate-releases)

## Use Git for source control

If it's not in source control, it doesn't exist.
[Git](https://git-scm.com) is the most popular choice, but its interface can be difficult to understand.
Consider using a GUI like GitHub Desktop.

## Host on GitHub

[GitHub](https://github.com) allows other developers to easily contribute to your package.
Compared to other hosts, you are more likely to receive contributions.
Also GitHub integrates nicely with many other services.

## Build with Stack

[Stack](https://docs.haskellstack.org/en/stable/README/) painlessly manages Haskell dependencies.
It installs GHC for you and ensures you get a build plan that actually works.
If your package builds today, Stack ensures it will continue to build tomorrow.

## Define with hpack

The default Cabal package file format is custom, tedious, and verbose.
[hpack](https://github.com/sol/hpack) uses YAML and avoids unnecessary boilerplate.
Stack integrates hpack to automatically convert your `package.yaml` into a `*.cabal` file when necessary.

## Name with kebab case

Keep everything lowercase to avoid confusion.
You don't want people trying to install `http` when they really meant `HTTP`.
Use hyphens to separate words, but keep it short and memorable.
Nobody wants to type out `hypertext-transfer-protocol`.

## Use Semantic Versioning

Unfortunately Hackage recommends the [Package Versioning Policy](https://pvp.haskell.org).
The PVP adds ambiguity by using two major version numbers.
That encourages packages to stay on major version 0, which doesn't inspire confidence.
Many other languages use [Semantic Versioning](https://semver.org).
SemVer matches how developers generally think about version numbers.

## Choose a license

Nobody can use a package without a license.
The most popular license for Haskell is [BSD 3-Clause](https://choosealicense.com/licenses/bsd-3-clause-clear/), followed by MIT.
Whichever license you choose, include the license file in your package.

## Write a README

Most people will familiarize themselves with your package by reading your README.
It should describe the problem that your package solves.
Be sure to include at least one concrete example in it.
GitHub will automatically format your `README.markdown`.

## Keep a change log

Most packages will use Git tags to mark releases, but reading diffs is not an acceptable way for users to discover changes.
Put a human-readable summary of changes in the GitHub releases.
Then link to that from your `CHANGELOG.markdown`.

## Write a synopsis

The `synopsis` shows up when searching and viewing your package.
Keep it short, imperative, and descriptive.
Also write a `description`, which can be pretty much the same as the `synopsis`.
For example:

``` yaml
name: aeson
synopsis: Encode and decode JSON.
description: Aeson encodes and decodes JSON.
```

## Avoid heavy dependencies

Only add a dependency if your package needs it to function.
Don't include stuff that's just nice to have.
For example, you should probably avoid `lens` even though it makes code easier to write.
In addition to avoiding heavy dependencies, you should avoid having too many dependencies.
Think about how long it would take to install your package starting from scratch.

## Include extra source files

If a file is necessary for your package to build, it belongs in `extra-source-files`.
This includes files that tests and benchmarks need.
You should also include package metadata like your README and change log.
Note that you don't need to include your license file if your package's `license-file` is set.

## Fix package warnings

Stack prints these out when you run `stack sdist`.
You should fix all of them, even though some aren't that useful.
For instance, Stack warns you if you don't set a category even though the categories on Hackage aren't that useful.

## Separate source files from metadata

In other words, separate your package metadata from actual source files.
This makes it easy to write scripts that work on every Haskell file, like formatting or counting lines of code.
The exact names aren't important, but you should end up with a structure like this:

- `library/YourPackage.hs`
- `executables/Main.hs`
- `tests/Main.hs`

## Match package and module names

If your package is named `tasty-burrito`, you should have a top-level module called `TastyBurrito`.
Avoid the unnecessary module hierarchy like `Data.ByteString` or `Text.ParserCombinators.Parsec`.
While the package name uses `kebab-case`, module names should use `CamelCase`.

## Require one import

Users should be able to get started with nothing more than `import YourPackage`.
If necessary, re-export stuff from other packages.
Design for qualified imports; don't be afraid to take common names like `singleton`.

## Expose implementation details

People will want to use your package in ways you didn't think of.
Make everything public, but not necessarily part of your published API.
Use `Internal` module names like `Data.Text.Internal` to signal that things aren't published.

## Fix all GHC warnings

GHC finds all kinds of problems with `-Wall`.
Few of them are false positives and they generally help you write better code.
But if you don't like one, disable it with `-Wno-warn-whatever`.
Be sure to use `stack build --pedantic` when developing your package.
It will force you to fix warnings.

## Follow HLint suggestions

[HLint](https://github.com/ndmitchell/hlint) is a great tool for improving code quality.
However some suggestions aren't worth following.
For example, the re-export shortcut suggestion breaks the Haddock documentation.
Use your own judgment when deciding which suggestions to follow.

## Format code with Brittany

[Brittany](https://github.com/lspitzner/brittany) is a robust and customizable Haskell source code formatter.
Using it frees you from ever thinking about formatting again.
If you don't like how something looks, fix it in Brittany and everyone's formatting will improve.
Using Brittany also avoids pointless arguments about style in pull requests.

## Write documentation with examples

Types are not a substitute for documentation.
Neither are laws.
Usually functions are added to solve a specific problem.
Show that problem in the documentation as an example.

## Test with Hspec

[Hspec](https://hspec.github.io) is a library for writing human-readable tests.
It is inspired by Ruby's RSpec.
It integrates with QuickCheck and SmallCheck for writing property-based tests.

## Run tests on Travis CI

[Travis CI](https://travis-ci.org) is free for open source projects and integrates with GitHub.
Every time you push a commit to GitHub, Travis CI will run your test suite.
This makes it easy to keep your package buildable.
By default Travis CI runs on Linux, but you can also run on macOS.
If you want to run your test suite on Windows, consider using [AppVeyor](https://www.appveyor.com).

## Keep executables small

If your package provides an executable, define it in your library and re-export it.
This allows other packages to use your executable's behavior from Haskell.
Your executable should look like this:

``` haskell
module Main (module YourPackage) where
import YourPackage (main)
```

## Benchmark with Criterion

If your package needs to be fast, [Criterion](http://www.serpentine.com/criterion/) is the best tool for measuring it.
On the other hand if your package doesn't need to be fast, there's no sense in maintaining benchmarks for it.

## Automate releases

Don't manually create distribution tarballs and upload them to Hackage.
Instead, get Travis CI to do it.
Travis CI sets the `TRAVIS_TAG` environment variable.
If that's set, you can run `stack upload .` to upload your package.
Travis CI will need your Hackage credentials, so be sure not to leak those into the build log.
