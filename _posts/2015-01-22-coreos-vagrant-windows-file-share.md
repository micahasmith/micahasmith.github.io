---
title: CoreOS Windows Vagrant File Sharing
layout: post
excerpt: How to get file sharing working between CoreOS and Vagrant on Windows via rsync. 
---

# CoreOS Windows Vagrant File Sharing

As soon as one turns on the [file sharing settings detailed on the CoreOS Vagrant into](https://coreos.com/docs/running-coreos/platforms/vagrant/#shared-folder-setup), you invariably end up with the following error:

```
No synced folder implementation is available for your synced folders!
Please consult the documentation to learn why this may be the case.
You may force a synced folder implementation by specifying a "type:"
option for the synced folders. Available synced folder implementations
are listed below.

docker, nfs, rsync, smb, virtualbox
```

How do we fix this?

### The rsync Fix

Steps to set up rsync:

1. [Download cygwin](https://cygwin.com/install.html)
2. Be sure to install `rsync` on the cygwin packages selection screen (its under Net)
3. After install, add the `c:/cygwin64/bin` dir to your PATH

Great! Now that we have rsync installed, let's modify our Vagrantfile to use it-- we're going to go from:

{% highlight ruby %}
config.vm.synced_folder ".", "/home/core/share", id: "core", :nfs => true,  :mount_options   => ['nolock,vers=3,udp']
{% endhighlight %}

to:

{% highlight ruby %}
config.vm.synced_folder ".", "/home/core/share", id: "core", type: "rsync", rsync__auto: "true"
{% endhighlight %}

Great! So we `vagrant destroy && vagrant up` to totally delete and reprovision/restart our VMs and we get...

```
rsync: change_dir "/c/Users/micah/coreos-vagrant" failed: No such file or directory (2)
```

RRRRRRRGHGHGHGH!!!

Fortunately [the fix for this is pretty simple](https://github.com/mitchellh/vagrant/issues/3230#issuecomment-62588180), near the top of your Vagrantfile, add the line:

{% highlight ruby %}
ENV["VAGRANT_DETECTED_OS"] = ENV["VAGRANT_DETECTED_OS"].to_s + " cygwin"
{% endhighlight %}

Done! Note that I had to restart my PC for it to *entirely* work for me. 