---
layout: post
title: Getting started with vagrant and ansible
---

[vagrant](http://vagrantup.com) is a tool to create and configure reproducible
development environments. Wat?!

Using virtual machines as your development environment has got multiple
benefits: When you're a developer it's easy to provide every team member with
the same environment. When your a designer you don't have the hassle of setting
up the workspace and can focus on the design instead. When you're done and want
to deploy your software you can use the same process you used to setup your
virtual machines to provision your production system.

### Sounds dope. Where can I get some? ###
[Download](http://www.vagrantup.com/downloads.html) the vagrant version for
your system and install it. On Ubuntu that's 

    sudo dpkg -i vagrant_1.5.4_x86_64.deb

Eventually you have to install the virtualbox packages, too. Enter

    sudo apt-get install virtualbox

That's it. Now you're ready to setup your first virtual machine. Keep calm, I
will guide you:

    vagrant init hashicorp/precise32 
    vagrant up
    vagrant ssh

Easy, heh? We just downloaded an Ubuntu precise image, created a so called
[Vagrantfile](http://docs.vagrantup.com/v2/vagrantfile/index.html), booted the
machine and connected to it. Halt the machine by disconnecting from the ssh
session and entering `vagrant halt`.
