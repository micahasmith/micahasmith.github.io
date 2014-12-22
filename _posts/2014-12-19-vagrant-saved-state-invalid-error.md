---
title: Fixing Vagrant "The Saved State File Is Invalid" Error
layout: post
excerpt: Fixing Vagrant "The Saved State File Is Invalid" error
---

# Vagrant's "The Saved State File Is Invalid"

The error i got was:

```
==> core-01: Setting the name of the VM: coreos-vagrant_core-01_1419018174821_58431
There was an error while executing `VBoxManage`, a CLI used by Vagrant
for controlling VirtualBox. The command and stderr is shown below.

Command: ["showvminfo", "a366e78b-cf42-41a5-8186-b710acdbb42e", "--machinereadable"]

Stderr: VBoxManage.exe: error: The saved state file 'C:\Users\micah\VirtualBox VMs\vagrant_1380538074\Snapshots\2013-11-24T23-17-05-863735600Z.sav' is invalid (VERR_FILE_NOT_FOUND). Delete the saved state and try again
VBoxManage.exe: error: Details: code VBOX_E_FILE_ERROR (0x80bb0004), component Console, interface IConsole, callee IUnknown
```

The fix was to go into virtualbox and manually delete the images, and reprovision/`vagrant up` a new setup. 