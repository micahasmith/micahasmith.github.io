---
title: CoreOS Cloud Config/Securing etcd
layout: post
excerpt: In this post we jump into just what the cloud config file does, specifically in terms of etcd.
---

# CoreOS Cloud Config/Securing etcd

In our previous intro to CoreOS, we spun up a three node cluster using vagrant. One of the parts of spinning that up was declaring a cloud-config file we had named 'user-data'. In this tutorial we're going to dig into what that cloud-config represents.

A cloud-config is a way of declaritively specifying both your OS and your docker/systems that will run in it. Let's dive right in to the one we did in the last blog:

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

Step by step details of this:

### coreos

Here we're specifying OS config. Note that there are [possible other siblings](https://coreos.com/docs/cluster-management/setup/cloudinit-cloud-config/#file-format) to this `coreos` section. We'll use some of them later. 

### coreos -> etcd

In the `etcd` section, we're first configuring our etcd discovery url (something we also discussed in the last post), and then we're using the templating features of cloud-config to substitute in IP addresses. Specifically, the two settings are:

- `addr: $public_ipv4:4001`-- the address for client communication, meaning this is the [RESTful API](https://github.com/coreos/etcd/blob/master/Documentation/api.md) endpoint your clients use to get values in and out of `etcd`
- `peer-addr: $public_ipv4:7001`-- the address for peer communication, meaning this is the address+port used for `etcd` to `etcd` communication between `etcd` peers in a cluster 

These values get transformed into environmental variables on our machine. [Other possible `etcd` config options are laid out here](https://github.com/coreos/etcd/blob/master/Documentation/configuration.md).

#### Securing etcd?

Is our `etcd` in the cloud-config above secure? Not so much! Let's fix that:

** Step 1 ** (Windows Only)

Note that I assume you Windows users are using git bash...

Let's create a vagrant box just for us to use [etcd-ca](https://github.com/coreos/etcd-ca) in. In a new random dir, let's run `vagrant init hashicorp/precise32 && vagrant up && vagrant ssh`. 

Once you're SSH-ed in, run `sudo apt-get update && sudo apt-get install -y git golang` to update the box and install git and golang. 

** Step 2 **

This step assumes you have both git and golang v1.3+ setup on your box.

To download etcd-ca, run `git clone https://github.com/coreos/etcd-ca.git`.

Let's also `cd` in; run `cd etcd-ca`. 

** Step 3 **

Let's setup the etcd-ca tool. Run `. build`.  

If you get errors like the following:

```
src/github.com/coreos/etcd-ca/pkix/csr.go:89: undefined: x509.CertificateRequest
src/github.com/coreos/etcd-ca/pkix/csr.go:125: undefined: x509.CertificateRequest
src/github.com/coreos/etcd-ca/pkix/csr.go:143: undefined: x509.CertificateRequest
src/github.com/coreos/etcd-ca/pkix/key.go:31: undefined: crypto.PublicKey
src/github.com/coreos/etcd-ca/pkix/key.go:36: undefined: crypto.PublicKey
src/github.com/coreos/etcd-ca/pkix/key.go:132: undefined: crypto.PublicKey
```

That means you don't have golang 1.3, and [you need to build it from source](http://golang.org/doc/install/source)! Make sure you reset your `GOROOT` and `PATH` vars after you do. To be honest, if you need to upgrade to golang 1.3 and you're doing this from within a vagrant box, blow away the box (`vagrant destroy`) and start again (`vagrant up`).

If you didn't get those errors, YAY.

** Step 4 **

Let's run the etcd-ca tool. Since we built the tool in the last step, let's do `cd bin` in order to get into the build output's directory. 

To get the cert files we need out of etcd-ca, we do:

- Run `./etcd-ca init`. It will ask for a password. When this is done, you will have a certificate authority
- Run `/.etcd-ca new-cert mycert`. It will ask for a password, and when this is done you will have a new host identity. 
- Run `/.etcd-ca sign mycert` to sign the cert
- Run `/.etcd-ca chain mycert > mycert.pem` to export the cert chain (thats the file that starts with `----BEGIN CERTIFICATE-----`)
- Run `./etcd-ca export --passphrase="" --insecure | tar xvf -` with your passphrase to export the ca.crt and ca.key.insecure files. 
- Run `./etcd-ca export --passphrase="" --insecure mycert | tar xvf -` to export the host's private and public keys (mycert.key.insecure and mycert.crt)

** Step 5 **

Copy the five files created from the last step into the directory where you're working on your coreos cluster. If you were running etcd-ca from a vagrant machine, you can exit the machine and `vagrant suspend` it. 

### Configuring etcd Security

Now that we have our certificates, we can [configure our etcd to be secure](https://github.com/coreos/etcd-ca/blob/master/Documentation/work_with_etcd.md) in our cloud-config. **Note that some of these settings also go in the fleet section as well**. To do so, we need to set the following settings:

- `peer-ca-file` - to ca.crt
- `peer-cert-file` - to mycert.crt
- `peer-key-file` - to mycert.key.insecure
- `cert-file` - to mycert.crt
- 'ca-file' - to ca.crt
- `key-file` - to mycert.key.insecure

Let's add some configuration values in there:

{% highlight yaml %}
#cloud-config

coreos:
  etcd:
    # generate a new token for each unique cluster from https://discovery.etcd.io/new
    # WARNING: replace each time you 'vagrant destroy'
    discovery: https://discovery.etcd.io/7005dd479484de7f10a44c2377976bcd
    addr: $public_ipv4:4001
    peer-addr: $public_ipv4:7001

    # these files don't exist just yet, but don't worry bout that
    ca-file: /etc/etcd/mycert.pem
    key-file: /etc/etcd/mycert.key.insecure
    cert-file: /etc/etcd/mycert.crt
    peer-key-file: /etc/etcd/mycert.key.insecure
    peer-cert-file: /etc/etcd/mycert.crt
    peer-ca-file: /etc/etcd/ca.crt
  fleet:
    public-ip: $public_ipv4
    #
    # you have to put the settings in here as well!
    #
    etcd-ca-file: /etc/etcd/mycert.pem
    etcd-keyfile: /etc/etcd/mycert.key.insecure
    etcd-certfile: /etc/etcd/mycert.crt

{% endhighlight %}

But what about the fact that those files don't exist in our file system? How do we pass them in?

#### Passing Files Into CoreOS

The CoreOS documentation outlines a means to [pass files in via our very own cloud-config](https://coreos.com/docs/cluster-management/setup/cloudinit-cloud-config/#write_files)!

For this part I am only going to include an example of one part of passing in files (since its straightforward).

{% highlight yaml %}
#cloud-config

coreos:
  etcd:
    # generate a new token for each unique cluster from https://discovery.etcd.io/new
    # WARNING: replace each time you 'vagrant destroy'
    discovery: https://discovery.etcd.io/7005dd479484de7f10a44c2377976bcd
    addr: $public_ipv4:4001
    peer-addr: $public_ipv4:7001

    # these files don't exist just yet, but don't worry bout that
    ca-file: /etc/etcd/mycert.pem
    key-file: /etc/etcd/mycert.key.insecure
    cert-file: /etc/etcd/mycert.crt
    peer-key-file: /etc/etcd/mycert.key.insecure
    peer-cert-file: /etc/etcd/mycert.crt
    peer-ca-file: /etc/etcd/ca.crt
  fleet:
    public-ip: $public_ipv4
    etcd-ca-file: /etc/etcd/mycert.pem
    etcd-keyfile: /etc/etcd/mycert.key.insecure
    etcd-certfile: /etc/etcd/mycert.crt
write_files:
  - path: /etc/etcd/ca.crt
    permissions: 0644
    owner: root
    # put the content for the cert in here
    # note that the cert will obvs be much longer than what i have here
    content: |
      -----BEGIN CERTIFICATE-----
      MIIFNDCCAx6gAwIBAgIBAfsQWjHa9OEBQu9MHPex8o7zK+GJoejOVJKbGEdT
      -----END CERTIFICATE-----
  # also add in the other cert files after here

{% endhighlight %}

Ok, after you have all the files in your `write_files` section let's run `vagrant reload --provision` to test it out.

So, after we ssh in to one of our coreos instances we run `curl --key /etc/etcd/mycert.key.insecure --cert /etc/etcd/mycert.crt --cacert /etc/etcd/mycert-chain.pem -L https://127.0.0.1:4001/v2/keys/foo -XPUT -d value=bar -v` and the output should have something like `{"action":"set","node":{"key":"/foo","value":"bar","modifiedIndex":3,"createdIndex":3}` in it... success!

Just to be extra-secure, lets see what it does without the certs:

`curl: (60) SSL certificate problem: unable to get local issuer certificate`

Denied!

### That's It

Perhaps next we'll get into real deployments...