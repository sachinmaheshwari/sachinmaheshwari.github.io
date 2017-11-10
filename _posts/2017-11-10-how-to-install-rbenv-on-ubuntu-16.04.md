---
title: "How to install rbenv on ubuntu 16.04?"
layout: post
date: 2017-11-10
---



"rbenv" is an awesome utility that can be used to manage different versions of ruby on your machine at the same time. You can read more about it [here](https://github.com/rbenv/rbenv). 

### Steps to install rbenv

These are the steps that you need to perform to install rbenv on Ubuntu 16.04:

It updates the package lists for upgrades for packages that need upgrading, as well as new packages that have just come to the repositories.

```shell
sudo apt update 
```

It will install all the dependencies that are needed for rbenv and ruby build.

```shell
sudo apt -y install autoconf bison build-essential libssl-dev libyaml-dev \
     libreadline6-dev zlib1g-dev libncurses5-dev libffi-dev libgdbm3 libgdbm-dev
```

It will clone the latest version of rbenv in your home directory.

```shell
git clone https://github.com/rbenv/rbenv.git ~/.rbenv
```

It will add the bin folder in your path

```shell
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
```

It will initialize and load rbenv in your login session.

```shell
echo 'eval "$(rbenv init -)"' >> ~/.bashrc 
```

This will execute your bashrc so that PATH and rbenv init can happen in current shell.

```shell
source ~/.bashrc 
```

It will clone the latest version of ruby-build plugin in rbenv plugins folder. This is needed to compile ruby on your machine.

```shell
git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build 
```

You can run this command to see what versions of ruby are available to install using rbenv.

```shell
rbenv install -l
```

Now you can run this command to install any ruby version that you want 

```shell
rbenv install 2.4.0 
```

It will set the specified version as global version

```shell 
rbenv global 2.4.0
```

Now to make it easier to run all these in one single command you can copy and paste this on your terminal.

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





