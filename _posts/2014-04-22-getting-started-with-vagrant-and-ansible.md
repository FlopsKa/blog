---
layout: post
title: Getting started with vagrant and ansible
---

<h3>{{ page.title }}</h3>

[vagrant](http://vagrantup.com) is a tool to create and configure reproducible
development environments. As a developer you don't have to care about your
dependencies and how you can keep the environment the team runs your code in
consistent. Team member designing the frontend don't have to care about the
setup of your product and can focus on the design instead.


#### Why is this cool? ####

Imagine you are deploying a webapp on a LAMPP stack. Every team member needs:

- a flavour of linux
- apache
- php5
- mysql

And probably some other project dependent dependencies, too. Imagine entering
`vagrant up` and the installation of the dependencies and you app is taken care
of. Imagine the same system working on your production server / servers -
without changing anything.

This is exactly what vagrant and [ansible](http://ansibleworks.com) can do for
you.


