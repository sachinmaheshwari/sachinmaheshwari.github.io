---
layout: post
title:  "Ruby on Rails and Vagrant"
date:   "2014-09-27"
categories: ruby vagrant
---

Recently I got introduced to a great tool called Vagrant. I like this tool because it is helping my life as a developer. My new project is using Ruby on Rails. I liked the simplicity of Ruby but all these gems that it installs in some internal directory got me thinking that it should be done on a separate virtual machine so that I can reclaim the space whenever I need it. Vagrant helped me in achieving that.

So here it goes:
You need to install 2 softwares first:

1. [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
2. [Vagrant](http://downloads.vagrantup.com/)

After that you need to create a folder where you want to work on Ruby on Rails projects.
In our example it will be

`mkdir /Users/sachin/Development/RubyTraining`

Next step is to create a Vagrantfile. Here are the contents of my Vagrantfile

```ruby
# -- mode: ruby --
# vi: set ft=ruby :
# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "precise64"
  config.vm.provision :shell, :path => "bootstrap.sh"  
  config.vm.box_url = "http://files.vagrantup.com/precise64.box"
  config.vm.network :private_network, ip: "192.168.33.11"
end
```



As you see I'm using bootstrap.sh to provision my virtual machine. Here are the contents of my bootstrap.sh

```shell
#!/bin/bash
sudo apt-get update
sudo apt-get -y  install curl
\curl -sSL https://get.rvm.io | bash -s stable --ruby
source /usr/local/rvm/scripts/rvm
gem install rails -V
sudo apt-get -y install nodejs
```

After this you only need to run one command and then sit back and enjoy watching your machine get ready for Ruby on Rails projects

`vagrant up`

Now you can ssh on that machine and run your Rails applications from your virtual machine.
