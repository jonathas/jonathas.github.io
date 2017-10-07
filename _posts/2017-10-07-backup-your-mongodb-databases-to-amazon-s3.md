---
layout: post
title: "Backup your MongoDB databases to Amazon S3"
excerpt: "How to do it using a backup script I developed and Docker"
tags: [nodejs, javascript, docker, mongodb, amazon, aws, s3, amazons3, backup, cron]
date: 2017-10-07T11:55:08+02:00
comments: true
image:
  feature: posts/cover-mongodb.jpg
---

I've developed a backup script for MongoDB using TypeScript. It's available as GPL3 [here](https://github.com/jonathas/mongo-s3-backup).

If you download or clone the repository, you can choose to run it with Docker (so cron is already configured) or run it without Docker (If you want to configure [cron on your host OS by yourself](https://corenominal.org/2016/05/12/howto-setup-a-crontab-file/)).

The cron example there is the same I wrote about on my [previous post about cron with Docker](https://jonathas.com/scheduling-tasks-with-cron-on-docker/).

In any case, you'll need to have Node.js and yarn installed.

This script generates a dump from your MongoDB databases, compresses this dump as tar.bz2, generates a hash file for integrity check and then uploads both files to your Amazon S3 bucket.

## Configuring

I expect you to have your Amazon Web Services (AWS) account ready and know your access key and secret access key before, also to have a bucket created on S3 in order to store the backup files.

In order to configure the backup script, you need to open the .env file and enter your data there, such as your timezone and AWS credentials.

Cron is configured to run the backup script every day at midnight. If you want to change the time there, change the file called root inside /infra/cron. If you need to change the timezone of the cron container, then open /infra/cron/Dockerfile

## Building

In order to build the script and start the cron process with Docker, run the following command:

```bash
make
```

This will run the following steps:

- Install the required packages
- Run the build script
- Build the cron Docker image
- Start the cron Docker container
- Run all the tests with npm

If you chose not to use Docker, but configure cron yourself on your host OS, then run this instead:

```bash
make no-docker
```

This will run the following steps:

- Install the required packages
- Run all the tests with npm

## Enabling daemon on system startup

In order to start this container automatically if your server reboots:

Change the path inside the file infra/host/etc/systemd/system/mongo-backup.service to be the path where you put the mongo-s3-backup source code.

As root, copy this file to /etc/systemd/system in your server

Reload the daemons:
```bash
sudo systemctl daemon-reload
```

Enable and start it:

```bash
sudo systemctl enable mongo-backup
sudo systemctl start mongo-backup
```

## Conclusion

Today we saw how to configure and run the MongoDB backup script. 

If you have any idea or suggestion, leave a comment or send a pull request.
