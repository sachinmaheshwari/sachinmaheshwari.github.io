---
title: "How to install rbenv on ubuntu 16.04?"
layout: post
date: "2017-11-10"
---



"rbenv" is an awesome utility that can be used to manage different versions of Ruby on your machine at the same time. You can read more about it [here](https://github.com/rbenv/rbenv).

### Steps to install rbenv

These are the steps that you need to perform to install rbenv on Ubuntu 16.04:

Update the package lists for upgrades for packages that need upgrading, as well as new packages that have just come to the repositories.

```shell
sudo apt update
```

Install all the dependencies that are needed for rbenv and ruby-build.

```shell
sudo apt -y install autoconf bison build-essential libssl-dev libyaml-dev \
     libreadline6-dev zlib1g-dev libncurses5-dev libffi-dev libgdbm3 libgdbm-dev
```

Clone the latest version of rbenv in your home directory.

```shell
git clone https://github.com/rbenv/rbenv.git ~/.rbenv
```

Add the bin folder to your PATH.

```shell
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
```

Initialize and load rbenv in your login session.

```shell
echo 'eval "$(rbenv init -)"' >> ~/.bashrc
```

Execute your .bashrc so that PATH and rbenv init can happen in the current shell.

```shell
source ~/.bashrc
```

Clone the latest version of the ruby-build plugin in the rbenv plugins folder. This is needed to compile Ruby on your machine.

```shell
git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build
```

Run this command to see what versions of Ruby are available to install using rbenv.

```shell
rbenv install -l
```

Run this command to install any Ruby version you want.

```shell
rbenv install 2.4.0
```

Set the specified version as the global version.

```shell
rbenv global 2.4.0
```

To run all of these with a single command, copy and paste this in your terminal.

```shell
sudo apt update && \
sudo apt -y install autoconf bison build-essential libssl-dev libyaml-dev libreadline6-dev zlib1g-dev libncurses5-dev libffi-dev libgdbm3 libgdbm-dev && \
git clone https://github.com/rbenv/rbenv.git ~/.rbenv && \
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc && \
echo 'eval "$(rbenv init -)"' >> ~/.bashrc && \
source ~/.bashrc && \
git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build && \
rbenv install 2.4.0 && \
rbenv global 2.4.0
```
