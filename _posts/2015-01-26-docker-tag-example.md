---
title: Docker Tag Example
layout: post
excerpt: Example of how to use the tag functionality in docker
---

# Docker Tag Example

Docker tag documentation which you would find via running `docker tag --help`:

```
core@core-01 ~/share/containers/webapp-test $ docker tag --help

Usage: docker tag [OPTIONS] IMAGE[:TAG] [REGISTRYHOST/][USERNAME/]NAME[:TAG]

Tag an image into a repository

  -f, --force=false    Force
core@core-01 ~/share/containers/webapp-test $
```

If you're new to docker (as I am), this doesn't make a ton of sense. What's worse, the info for it on the docker website is a mirror of the above. 

### A Walkthrough

Let's say you have a super simple python based webapp. Just to show how simple, you have a directory that looks like: 

- app.py
- requirements.txt
- Dockerfile

Your app.py is the [flask example up on the flask website](http://flask.pocoo.org/docs/0.10/quickstart/), and the requirements.txt is a one liner with the text `Flask==0.10.1`.

The Dockerfile looks like so:

```
FROM ubuntu

# Install Python Setuptools
RUN apt-get update -y
RUN apt-get install -y python-setuptools

# Install pip
RUN easy_install pip

# Add and install Python modules
ADD requirements.txt /src/requirements.txt
RUN cd /src; pip install -r requirements.txt

# Bundle app source
ADD . /src

# Expose
EXPOSE  5000

# Run
CMD ["python", "/src/app.py"]
```

Great! So to kick off our docker experience we build the image. So in the directory with the Dockerfile and etc., we run:

```
docker build -t micahasmith/webapp-test .

```

Note that the reason why i began mine with "micahasmith" is because that's my username on [hub.docker.com](http://hub.docker.com), which is free to sign up for. 

To make sure our new image is up at hub.docker.com, let's push it:
```
docker push micahasmith/webapp-test
```

Now that we have a working image, we can tag its current state via:
```
docker tag micahasmith/webapp-test:latest micahasmith/webapp-test:v0.0.1
```

I guess you could describe the command as tagging from a tag to another tag-- the `:latest` suffix makes the "from" the last commit. So in the above we created a "v0.0.1" tag. Let's now push that tag up to the hub:
```
docker push micahasmith/webapp-test:v0.0.1
```

Done!

To see what images we have on our machine now, we can run `docker images` and see the result:

```
core@core-01 ~/share/containers/webapp-test $ docker images
REPOSITORY                TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
micahasmith/webapp-test   latest              935e7e110493        8 minutes ago       251.5 MB
micahasmith/webapp-test   v0.0.1              0b0ea5e4c69b        7 hours ago         251.5 MB
ubuntu                    14.04               b39b81afc8ca        10 days ago         192.7 MB
ubuntu                    14.04.1             b39b81afc8ca        10 days ago         192.7 MB
ubuntu                    latest              b39b81afc8ca        10 days ago         192.7 MB
ubuntu                    trusty              b39b81afc8ca        10 days ago         192.7 MB
```

Notice both versions of the image are there. Cool!


