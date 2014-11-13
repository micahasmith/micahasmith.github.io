---
title: Intro to CoreOS
layout: post
excerpt: Getting started with CoreOS
---

# Intro to CoreOS

[CoreOS](https://coreos.com/) is a linux distro that is super-stripped down and optimized for running and managing Docker VMs. In this post we'll investigate getting a local CoreOS cluster set up, while along the way discussing what the various interesting parts of CoreOS are. 

So we've already established that CoreOS manages and runs Docker VMs. Our goal in setting up a CoreOS cluster is an architecture that supports horizontal scalability and system failover with no user interaction. So in the end we'd end up with a set of actual CoreOS instances running and managing Docker VMs so as to ensure uptime and provide room for future horizontal growth. CoreOS does all of this really beautifully, as we'll see.

First let's discuss some terms related to CoreOS:

- **CoreOS** is the actual base system OS that will run our Docker VMs
- **Docker** is a container/VM solution, where our applications actually reside
- **etcd** is a distributed configuration system that is key/value based-- think of it as "a mixture of redis and mq for application settings" (well i guess redis kinda is an mq)
- **fleet** is an application that runs on the CoreOS systems and orchestrates how and where our Docker VMs operate

So we're going to end up with a system that is these pieces composed together. 

### The Vagrant Setup

This'll be the second time i've setup CoreOS via vagrant; to be honest i think the CoreOS setup instructions on their getting started page kinda suck. 

** Step 1 **

`git clone` the CoreOS vagrant provisioning setup and cd into the resultant directory.

{% highlight bash %}
git clone https://github.com/coreos/coreos-vagrant.git
cd coreos-vagrant
{% endhighlight %}

** Step 2 **

Get an `etcd` token url. An `etcd` token/url is a unique id that is used to sync multiple `etcd` systems together. Remember how `etcd` is a redis/mq system for application configuration values? Well since its distributed, it needs this id/url to enable coordinating nodes in its cluster. So if you ping [https://discovery.etcd.io/new](https://discovery.etcd.io/new) you'll receive a response like `https://discovery.etcd.io/7005dd479484de7f10a44c2377976bcd`.

** Step 3 **

Take our token from step 2 and plug it into the `user-data.sample` file. Remember to uncomment the line that its on.

Our `user-data.sample` file should now look something like this:

{% highlight yaml %}
#cloud-config

coreos:
  etcd:
    # generate a new token for each unique cluster from https://discovery.etcd.io/new
    # WARNING: replace each time you 'vagrant destroy'
    discovery: https://discovery.etcd.io/7005dd479484de7f10a44c2377976bcd
    addr: $public_ipv4:4001
    peer-addr: $public_ipv4:7001
  fleet:
    public-ip: $public_ipv4
  units:
    - name: etcd.service
      command: start
    - name: fleet.service
      command: start
    - name: docker-tcp.socket
      command: start
      enable: true
      content: |
        [Unit]
        Description=Docker Socket for the API

        [Socket]
        ListenStream=2375
        Service=docker.service
        BindIPv6Only=both

        [Install]
        WantedBy=sockets.target
{% endhighlight %}

** Step 4 **

Rename `user-data.sample` to `user-data`.

** Step 5 **

Let's ensure we're setting up a 3 node cluster that's running the latest stable version of CoreOS by editing the `config.rb.sample` file to have the following values:

{% highlight yaml %}
# Size of the CoreOS cluster created by Vagrant
$num_instances=3

# Official CoreOS channel from which updates should be downloaded
$update_channel='stable'
{% endhighlight %}

In my file i had to change the values and uncomment them. 

** Step 6 **

Rename `config.rb.sample` to `config.rb`.

** Step 7 **

Run `vagrant up` to boot up our CoreOS cluster!

![CoreOS Vagrant Up](/resources/img/coreos-docker-vagrant-01.gif)

** Step 8 **

SSH into the first VM in our cluster by running `vagrant ssh core-01 -- -A`. You could substitute the `01` for `02` or `03` to get into one of the other VMs. 

### Now What

You know have a cluster of 3 CoreOS VMs ready to get up and running. Next we'll start diving into details of etcd, fleet, and/or security. 

