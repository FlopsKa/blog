---
layout: post
title: Speedup Vagrant Execution With a Ramdisk
---

<h3>{{ page.title }}</h3>

Currently I am using [vagrant](http://www.vagrantup.com/) in conjunction with
[ansible](http://www.vagrantup.com/) to provide all team members of our
university project with a consistent development and testing environment.
Before I push my changes I want to make sure that every ansible playbook I
wrote is working and idempotent. To achieve this goal I have to test the
scripts a lot and in doing so I spin up a dozen of new machines.

To be able to run the necessary software the machines have to download a lot of
packages. Some are only available as source and have to be compiled. Because
the bottleneck when running the machines often lies within the HDD I thought it
would be nice to speed it up a bit by putting the virtual machines on a
ramdisk.

#### Setting up the ramdisk ####

In ubuntu 13.10 'saucy' it is rather simple to set up a ramdisk:

    sudo mkdir /media/ramdisk
    sudo mount -t ramfs ramfs /media/ramdisk
		sudo chown -R user:group /media/ramdisk

__Note:__ Please don't store any important information on the ramdisk as the
data is lost when powering off the computer.

To automatically mount the partition when booting add this to your
`/etc/fstab`:

    ramfs		/media/ramdisk		ramfs		defaults		0		0

It is not possible to limit the size of the ramdisk. In extreme cases the host
system may run out of memory. 

#### Moving vagrant and virtualbox into the RAM ####

Vagrant usually stores it's files in `~/.vagrant.d/`. To use another path set
the VAGRANT_HOME environment variable.

    cp -r ~/.vagrant.d /media/ramdisk/vagrant_home
    export VAGRANT_HOME=/media/ramdisk/vagrant_home

Now you have to change the location of your virtual machines from within
virtualbox. Open the client by entering `virtualbox` into the terminal, go to
'File -> Preferences...' and enter your new ramdisk location as 'Default
Machine Folder'.

#### Performance improvements ####

_All tests were run with a new Vagrantfile and a 'raring64' box._

Startup time without ramdisk:

    time vagrant up
    ...
    vagrant up  2,63s user 1,99s system 12% cpu 35,825 total

Startup time with ramdisk:

    time vagrant up
    ...
    vagrant up  2,55s user 2,07s system 17% cpu 26,853 total

As you can see starting the image from a ramdisk is faster than starting from a
regular HDD. This is the case because vagrant copies the base box (in this case
'raring64') before starting it. However, there are no performance gains while
running the vm (tested with `hdparm -Tt /dev/sda`). This is  the case because
virtualbox is already caching the hard drive access into the ram.
