---
title: Copying Files Into boot2docker
layout: post
excerpt: How to copy a file into boot2docker. Useful for getting your Dockerfiles into your boot2docker VM.
---

# Copying Files Into boot2docker

Getting files into boot2docker isn't the easiest thing. You would think a tech that is based on Dockerfiles (let alone the other config files devs make) would make getting files in and out simple-- not the case.

[This is the most common method of copying files into boot2docker](http://stackoverflow.com/questions/24196956/how-to-deploy-dockerfile-and-application-files-to-boot2docker). I couldn't get it to work. Maybe because i'm on windows.

### Bash To The Rescue

After you see it, you wonder how you didn't think of it.

So we all know we do the following to get into a boot2docker VM:

{% highlight bash %}
$ boot2docker ssh
{% endhighlight %}

To copy a file in?

{% highlight bash %}
#! /bin/bash

# get the contents of our file
dockerfile=`cat Dockerfile`;

# copy it into the vm
boot2docker ssh "echo $dockerfile > Dockerfile"
{% endhighlight %}

And its waiting in your VM for you. Obviously you could mix this with an `ls` and a while loop to copy in many files.