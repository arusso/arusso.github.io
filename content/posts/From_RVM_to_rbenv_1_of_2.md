---
title: "From RVM to rbenv (1 of 2)"
layout: post
date: 2015-05-27 21:25:16
comments: true
tags: [ruby,rvm,rbenv,puppet]
share: true
summary: >
  Migrating from RVM to rbenv (ruby management). There is no part 2.
aliases:
  - /From_RVM_to_rbenv_1_of_2
---

* [Introduction](#introduction)
* [Installation](#installation)
* [Moar Rubies!](#moar-rubies)
* [Activating a Ruby](#activating-a-ruby)

## Introduction

Over the last day or so I've been slowly moving my ruby projects over to using rbenv instead of RVM. There's nothing inherently wrong with RVM, but I do lots of interesting things with my shell that, when combined with my tmux setup, seems to always be giving me flak.

So at the recommendation of a friend, I sat down with `rbenv` for a couple of hours, and these are my notes from that experience.

## Installation

Typically, I would not make a change so drastically, but I found the process of converting to be fairly painless and simple enough for me to go back to if need be without much fanfare. The process for me was to ...

  1. Remove any source references to RVM scripts in my .bashrc and .bash\_profile files
  2. Remove any path modifications that include the RVM bin dir
  3. Use homebrew to install rbenv (alternatively, you could clone the repository[^1] into ~/.rbenv 
  4. Add the following into my .bashrc

{{<codeWide language="shell" line-numbers="false">}}
if which rbenv > /dev/null; then eval "$(rbenv init -)"; fi
{{</codeWide>}}

## Moar Rubies!

If all went well above, we should have a working rbenv installation. Now let's take a look at the rubies currently available --

{{<codeWide language="shell" line-numbers="false">}}
$ rbenv versions
* system (set by /Users/arusso/.rbenv/version)
{{</codeWide>}}

I only have a single ruby version install initially, but with the help of the ruby-build[^2] plugin (available via hombere), I get access to the `rbenv install` command where I can install new versions of ruby.

In this case, my system ruby is version 2.0.0-p481 (`ruby -v`). This is too new for all my work, since I do a good deal with Puppet on RHEL6 which ships with 1.8.7-p374. Let's start by installing that version --

{{<codeWide language="shell" line-numbers="false">}}
$ rbenv install 1.8.7-p374
Downloading ruby-1.8.7-p374.tar.gz...
-> http://dqw8nmjcqpjn7.cloudfront.net/876eeeaaeeab10cbf4767833547d66d86d6717ef48fd3d89e27db8926a65276c
Installing ruby-1.8.7-p374...
Installed ruby-1.8.7-p374 to /Users/arusso/.rbenv/versions/1.8.7-p374

Downloading rubygems-1.6.2.tgz...
-> http://dqw8nmjcqpjn7.cloudfront.net/cb5261818b931b5ea2cb54bc1d583c47823543fcf9682f0d6298849091c1cea7
Installing rubygems-1.6.2...
Installed rubygems-1.6.2 to /Users/arusso/.rbenv/versions/1.8.7-p374
{{</codeWide>}}

Now looking at the versions available to us, we see --

{{<codeWide language="shell" line-numbers="false">}}
$ rbenv versions
* system (set by /Users/arusso/.rbenv/version)
  1.8.7-p374
{{</codeWide>}}

## Activating a Ruby

I typically activate rubies in two ways. First and foremost, when I'm switching between rubies for testing I used to use `rvm use $version` to get the ruby I want. With rbenv, this becomes `rbenv shell $version`.

The second way I choose rubies is by setting my ruby version in the _.ruby-version_ file in my project directory. Fortunately, this does not really change and I can mostly leave it alone[^3].

For more information on how rbenv chooses a ruby version, see the project's README section[^4] on the subject

## Next Time

The next post will dive into the differences in gemset management between RVM and rbenv, as well as some useful plugins that make rbenv a better tool all around.

[^1]: https://github.com/sstephenson/rbenv
[^2]: https://github.com/sstephenson/ruby-build
[^3]: https://github.com/sstephenson/rbenv#choosing-the-ruby-version
[^4]: rvm conviently allows you to select which gemset you want to use within the _.ruby-version_ file. rbenv on the other hand does not even support gemsets without the help of the rbenv-gemset[^5] plugin. With it, you need only move the gemset information into the _.ruby-gemset_ file. Part 2 will go into more detail about gemsets.
[^5]: https://github.com/jf/rbenv-gemset
