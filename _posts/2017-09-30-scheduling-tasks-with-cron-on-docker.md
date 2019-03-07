---
layout: post
title: "Scheduling tasks with cron on Docker"
excerpt: "When you need to setup a cron job, you can do it using Docker. In this post I show how to do that with an Alpine Linux image"
tags: [docker, alpine, linux, cron, alpinelinux, node, nodejs, javascript]
date: 2017-09-30T09:30:10+02:00
comments: true
image:
  feature: posts/cover-cron.jpg
---

When you need to setup a cron job, you can do it using Docker. The best way to do it with Docker is using an Alpine Linux image, as it has only 5MB initial size.

Don't know what is Docker? Check more about it [here](https://www.docker.com/what-docker).

In this post I expect you to already have Docker and docker-compose installed on your OS.

## Creating the image

In the case below I needed an Alpine Linux container with Node.js inside, as I wanted cron to run a Node.js script.

First we create a file called Dockerfile with the following content (yes, with no file extension):

{% highlight bash linenos %}
FROM node:8-alpine
MAINTAINER Jonathas Ribeiro <contact@jonathas.com>

RUN apk update && apk add tzdata &&\ 
    cp /usr/share/zoneinfo/Europe/Prague /etc/localtime &&\ 
    echo "Europe/Prague" > /etc/timezone &&\ 
    apk del tzdata && rm -rf /var/cache/apk/*

CMD chown root:root /etc/crontabs/root && /usr/sbin/crond -f
{% endhighlight %}

Let's save this file inside a directory called cron.

The FROM command tells Docker which image we want to use for building our image.

The MAINTAINER command tells Docker who is maintaining this Dockerfile.

The RUN command tells Docker which commands to run on image creation.

It's very important for your Dockerfile to have the least number of commands possible, as another RUN command for example, would create another layer in the resulting image. That's why we concatenate the commands with && instead of writing a RUN command for each of them. Simplicity should be the goal.

In the example, I set the timezone to Prague (my current timezone), but you can change it to yours instead, of course.

The CMD command tells Docker what will run inside the container that will be created from this image, which is cron in this case. Exactly what we need.

## Configuring cron

Now we need to configure what this image will run using cron.

I created this silly example below just to illustrate what could be done.

Save the content below to a file called root in the same directory where you saved the Dockerfile (the cron directory you created before).

{% highlight bash linenos %}


# ┌───────────── minute (0 - 59)
# │ ┌───────────── hour (0 - 23)
# │ │ ┌───────────── day of month (1 - 31)
# │ │ │ ┌───────────── month (1 - 12)
# │ │ │ │ ┌───────────── day of week (0 - 6) (Sunday to Saturday;
# │ │ │ │ │                                       7 is also Sunday on some systems)
# │ │ │ │ │
# │ │ │ │ │
# * * * * *  command to execute

0 7,19 * * * /usr/local/bin/node /home/node/hello.js >> /var/log/crontest/hello.log 2>&1



{% endhighlight %}

Now create a directory called scripts on the same level of the cron directory, not inside of it.

Inside this scripts directory, create our hello.js script with the following content:

```javascript
console.log("Hello world");
```

In this example I told cron to execute a JavaScript file using the Node.js executable that comes inside the container and output the response to a hello.log file.
This cron command, as you can see in the explanation in the comments, will run every day at 7:00 and 19:00.

Ok, so how to add this to our container?

## Docker-compose

I like using docker-compose as it enables me to orchestrate many containers in a simple way, using only one docker-compose.yml file.

With docker-compose you can for example create a file with all the containers you need and then run docker-compose up for it to start all of them at once, instead of dealing with docker run and passing all parameters every time, etc.

Let's create a file called docker-compose.yml with the following content in same level as our scripts and cron directories:

{% highlight yaml linenos %}
version: "2"
services:
    cron:
        build: cron
        container_name: crontest
        volumes:
            - /var/log/crontest:/var/log/crontest
            - cron/root:/etc/crontabs/root
            - sripts:/home/node
{% endhighlight %}

Spacing is very important here. Visual Studio Code can help you with that, as it detects yml files.

The cron directory is informed in the build atribute there, so docker-compose will build the image using the Dockerfile we created inside the cron directory.

In "volumes", the crontest directory inside your host OS is being mounted inside the container using the same path there, so when our script outputs to hello.log as configured above, the log will be easily accessible from the host OS instead of having to enter the cron container to read it. If this directory doesn't exist on your host OS, docker-compose will create it.

For the next lines configured inside "volumes", the root file we created inside the cront directory is being mounted inside the container in the /etc/crontabs/root path. On the 3rd line, everything from the scripts directory is being mounted inside /home/node inside the container, as this is the home directory this Node.js container comes with.

Instead of mounting these files inside the container using docker-compose, you could have added them on image creation using the [ADD command](https://docs.docker.com/engine/reference/builder/#add), but then you'd have to rebuild the image if you needed to change any of these files and the resulting image would also have more layers. Because of that, I prefer to mount them from the host OS instead, so if I need to modify any of these files, they are not builtin with the image in any way.

After saving the docker-compose.yml file, you can run the command to build our docker image and start our container from it:

```bash
docker-compose up
```

This way all the commands will show up on your terminal and you won't be able to use this terminal for anything else. If you add -d in the end of the command, then it runs as a daemon and your terminal will be free, but you wont see any output about the image build process or when it's running.

If you run docker-compose witout the daemon option, you just need to use Ctrl + c when you want to stop the containers specified in your docker-compose.yml file. If you run as a daemon, you can run docker-compose stop to stop the containers and then if you want to remove them you can run docker-compose rm

In order to list the current containers:

```bash
docker ps -a
```

And the images:

```bash
docker images
```

## Running on startup

After creating the image and checking that everything works as expected, we need to create a startup script so if our OS gets restarted the cron container runs automatically.

The servers I configure are usually running [Debian Linux](https://www.debian.org/), my personal notebook runs [Arch Linux](https://www.archlinux.org/) and at work I run [Manjaro](https://manjaro.org/). All these Linux distributions come with systemd, so I'll show an example of how the startup could be configured for that.

Let's save a file called docker-infra.service inside /etc/systemd/system/ with the following content:

{% highlight bash linenos %}
[Unit]
Description=Docker infra
Requires=docker.service
After=docker.service

[Service]
Restart=always
ExecStart=/usr/local/bin/docker-compose -f /home/jon/crontest/docker-compose.yml up
ExecStop=/usr/local/bin/docker-compose -f /home/jon/crontest/docker-compose.yml stop

[Install]
WantedBy=default.target
{% endhighlight %}

You need to change the path to your docker-compose.yml file to the one where you saved yours inside your server, of course.

Then let's reload the system daemons:

```bash
sudo systemctl daemon-reload
```

and enable our newly created daemon to run on system startup:

```bash
sudo systemctl enable docker-infra
```

If you need to stop or check the status of the daemon that controls our docker container, just change the enable in the systemctl command above to stop or status and that's it.

## Conclusion

In this post we saw how to create a Docker image to use as cron to run Node.js scripts. Then we saw how to use docker-compose and create a daemon for it to run on system startup.

If you have any suggestion or doubt, let me know in the comments below :)
