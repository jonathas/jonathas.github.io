---
layout: post
title: "One year working with Node.js after PHP and .NET - Challenges, comparisons and improvements"
excerpt: "Challenges, comparisons and improvements. A bit of what I've been experiencing with Node.js"
tags: [nodejs, javascript, async, dotnet, php, deployment, passenger, pm2, mongodb, vagrant, docker]
date: 2017-09-16T10:20:08+02:00
comments: true
image:
  feature: posts/cover-nodejs.jpg
---

In my software development career I've worked so far with both frontend and backend development, also with Desktop apps development (.NET) and Hybrid mobile development (Ionic Framework), but always enjoyed and focused more on the backend. I started my career in 2008 with PHP, so around this time JavaScript was mostly for creating alert messages, form validation and a bit of user interaction. Who could tell it would become what it is today?

In 2010 I've worked with Java web for a while, then PHP again, then around 2013 I changed to .NET for desktop and web, then AngularJS (Ionic Framework), then PHP again and then Node.js (since 2016).
So this year (2017) I complete 1 year working with Node.js and related technologies. It has been a great journey so far, full of ups and downs.

## Asynchronous vs Synchronous

When you write your code in PHP, .NET or Python, the instructions you write are followed line by line when the code is executed. That doesn't happen with JavaScript when you write a function that reads a file or fetches something from the database, for example. That function is called and the code continues its execution on the next line before that function is finished, as it works asynchronously. This can be mind blowing sometimes for someone who is coming from other languages, so it was the biggest change for me when starting to work with Node.js. Even though Node.js has many methods that have synchonous versions, it's not recommended to use them, as the Node process would be waiting for the method execution to finish before doing anything else. So it's always recommended to use async functions in order to allow the event loop to work in an optimized way.

![Event loop](/images/posts/eventloop.jpg "Event loop")

You get used to it after a while, but the callback hell was probably what made all look more confusing.

## Callbacks vs Promises vs async/await

As I started learning more about Node.js, all the courses I took online and books I read were showing how to do everything with callbacks. Sometimes there were callbacks inside callbacks, creating what is called the "[callback hell](http://callbackhell.com/)", code with many levels of indentation.

{% highlight javascript linenos %}
const fs = require("fs");

fs.readFile("a.txt", "utf8", (err, a) => {
     fs.readFile("b.txt", "utf8", (err, b) => {
           fs.writeFile("ab.txt", a + b, () => {
               console.log("we are done");
           });
     });
});
{% endhighlight %}

As a result, the first system I wrote in Node.js was full of callbacks. When I had to use a for loop with callbacks inside, then it was an even bigger problem. So I found out I could use the [async module](https://www.npmjs.com/package/async) for that, to orchestrate these callbacks and loop through them and that way the code would wait instead of going to the next line and ignoring the loop.

{% highlight javascript linenos %}
const async = require("async");
const fs = require("fs");

const concatFiles = (callback) => {
    let directories = ["dir01", "dir02", "dir03", "dir04"];

    async.each(directories, (dir, loopCallback) => {
        fs.readFile(`${dir}/a.txt`, "utf8", (err, a) => {
            fs.readFile(`${dir}/b.txt`, "utf8", (err, b) => {
                fs.writeFile(`${dir}/ab.txt`, a + b, () => {
                    console.log(`We are done for directory ${dir}`);
                    loopCallback();
                });
            });
        });
    }, (err) => {
        if (err) {
            callback(err);
        } else {
            callback(null, true);
        }
    });
}

concatFiles((err, res) => {
    if (err) {
        // Do something with the error
    } else {
        console.log("All done!");
        // Do something else here that depends on the ab.txt files
    }
});
{% endhighlight %}

Promises looked like a cleaner alternative for the callbacks, so I started refactoring parts of the code to use them and they looked indeed much cleaner.

{% highlight javascript linenos %}
const fs = require("fs");

const readFileContent = (filePath) => {
    return new Promise((resolve, reject) => {
        fs.readFile(filePath, "utf8", (err, data) => {
            if (err) return reject(err);
            return resolve(data);
        });
    });
}

const writeFileContent = (filePath, content) => {
    return new Promise((resolve, reject) => {
        fs.writeFile(filePath, content, (err) => {
            if (err) return reject(err);
            return resolve(true);
        });
    });
}

const concatFiles = () => {
    let concatContent = "";
    return readFileContent(`a.txt`)
        .then(a => {
            concatContent = a;
            return readFileContent(`b.txt`);
        })
        .then(b => {
            concatContent += b;
            return writeFileContent(`ab.txt`, concatContent);
        });
}

concatFiles().then(res => /* etc etc */ );
{% endhighlight %}

Then on ES7 we finally were allowed to make our code look normal like in some other languages, with instruction below instruction, instead of instruction inside instruction. When you have less levels of indentation, your code looks clearer and simpler. So I started refactoring all code to async/await and using util.promisify. Now it is much shorter and looks much simpler and easier to understand.
(Code refactored by [alsiola](https://www.reddit.com/user/alsiola) on reddit)

{% highlight javascript linenos %}
const fs = require("fs");
const util = require("util");
const readFile = util.promisify(fs.readFile);
const writeFile = util.promisify(fs.writeFile);

const concatDir = async (dir) => {
    const [a, b] = await Promise.all([
        readFile(`${dir}/a.txt`, "utf-8"),
        readFile(`${dir}/b.txt`, "utf-8")
    ]);

    await writeFile(`${dir}/ab.txt`, a + b);
}

const concatFiles = async () => {
    try {
        const dirs = ["dir01", "dir02", "dir03", "dir04"];
        await Promise.all(
                dirs.map(concatDir)
        );
        return "All Done!";
    } catch (err) {
        return err;
    }
}

const doSomethingElse = async () => {
    await concatFiles();
    // Do something else here that depends on the ab.txt files inside every directory
    // Return a promise
}
{% endhighlight %}

It is easier to work with that on Node.js than on the frontend, as you'll only need to update Node version to 8 or later, while on the frontend you'd have to use Babel to transpile to older ECMAScript versions in order to support older browsers that don't have async/await implemented in their old versions.

## MongoDB and the NoSQL paradigm

![MongoDB](/images/posts/mongodb.jpg "MongoDB")

This database has nothing an everything to do with Node.js. It can be used with many other languages as well, but it's the most used one by Node.js developers, forming the MEAN (MongoDB, Express, AngularJS, Node.js) or MERN (MongoDB, Express, React, Node.js) stack. Before starting to work with Node.js, I had used MySQL, SQL Server, Oracle and studied about PostgreSQL, which are all relational databases. So this was also a change of paradigm for me, as MongoDB is a NoSQL, non relational database. It doesn't have queries, tables nor relationships. In the beggining I ended up modelling the database with the relational paradigm in mind, which didn't work well for MongoDB (got really slow), so then I learned better and had to model it again to use the NoSQL way. Instead of queries with joins, I had to learn about the Aggregation framework, which works with pipelines filtering the results from the database. In my opinion that looks more confusing than any SQL query.

## Vagrant and Docker

Some years ago while working with PHP, I started using Vagrant with Puppet for provisioning development environments, but it would use a full virtual machine with Virtualbox for that with everything in one place and most of times it would become huge. Since starting to work with Node.js I also started using Docker, which I wanted to start using long before already. Docker is much lighter and not a full virtual machine, but a [container](https://www.docker.com/what-container), so it can be used for development and I started using it even for production in order to make the deployment process more automatic and much simpler. The same environment you have on development you have on production, so this takes away the "works on my machine" excuse.

## Deployment challenges

This part is where I had the biggest challenge. Some years ago when I was studying about Ruby on Rails, I wanted to learn how to deploy it to production, as what I had done the most before was LAMP (Linux, Apache, MySQL, PHP). I found out about [Phusion Passenger](https://www.phusionpassenger.com/) at that time, and that many people were using it to deploy their Ruby on Rails systems.

When I started thinking about how I would deploy Node.js, my company wanted to deploy to a Debian Linux server (IaaS) instead of to a managed platform like [Heroku](https://www.heroku.com/) or [Openshift](https://www.openshift.com/) (PaaS). I saw no problem with that, as I had been using Linux since 2006 and had already configured many LAMP servers before. So I rememberd about Phusion Passanger, went to look for it and saw that they also supported Node.js. I found out they had a Docker image as well, with everything installed and ready to be used on production, so I went with that. I read through their whole documentation for Node.js and the best way to deploy Passenger integrated with Nginx for Node.js.

Their documentation stated that in order to deploy in an optimized way for thousands of simultaneous requests, several Passenger instances should be started, according to the amount of RAM available on the server. So following a calculation they had on their documentation, I configured the number of Passenger instances according to the amount of RAM to handle the load. Then it's when the problems started to happen. When we started testing it with thousands of simultaneous clients, the Passenger instances would start handling the load but would also start bloating the RAM. After a while all the RAM was full and the server started swapping, until it couldn't take it anymore and would shutdown. We tried increasing the amount of available RAM several times, but it would use as much RAM as it had available.

Looking into the problem and discussing it on some IRC channels, I realised the issue was that Passenger's deployment documentation was written for several languages and technologies (As Passenger can be used for Rails and Python as well), not specifically for a Node.js system, so that's why it had that kind of calculation there to discover the number of instances that should be spinned in cluster mode. Because of that configuration, the CPU was becoming crazy with the context switching because of so many requests and the RAM was getting bloated. For Node.js, a non multi-thread technology, it should be maximum 1 process per CPU core.

After researching more and more about Node.js deployment, I decided to change from Passenger integrated with Nginx (all inside one Docker container) to a [pm2](http://pm2.keymetrics.io/) Docker container with a separate container for Nginx in front of it as a reverse proxy. Passenger is written in C++ and its [Docker image](https://github.com/phusion/passenger-docker) is based on Ubuntu with Nginx and even Redis or Memcached inside, which is a wrong way of using Docker, putting everything inside the same container as if it was a virtual machine with everything you need. The [pm2 Docker image](https://hub.docker.com/r/keymetrics/pm2/) comes only with pm2 and is based on [Alpine Linux](https://alpinelinux.org/), which has only 5mb size, so it's much more optimized.

After this change of process manager for Node.js, I also decided to split the monolith API into some microservices and run a pm2 docker container for each of these microservices, with Nginx proxying to the right address inside the network according to the accessed endpoint. The amount of pm2 instances to handle the load was configured not to be bigger than the amount of CPU cores the server has, so this time the server handled the load beautifully and consumed not even 10% of the available RAM, even under heavy load.

## Resources I used for learning Node.js

Some of the resources I've used for learning Node.js are listed below:

- Courses on [alura.com.br](https://www.alura.com.br/) (Brazilian Portuguese). If you understand Portuguese, I always recommend these guys.
- Lynda's course [Node.js Essential Training](https://www.lynda.com/Node-js-tutorials/Node-js-Essential-Training/417077-2.html)
- Udemy's course [Master NodeJS: The Complete Front-End Developer Course](https://www.udemy.com/the-complete-nodejs-developer-course-2/)
- Pluralsight - Building Web Apps With Node.js
- Chalkstreet - Node.js: Learn Node.js from scratch
- Coursera - [Server-side Development with NodeJS: The Hong Kong University of Science and Technology](https://www.coursera.org/learn/server-side-development/home/welcome)
- Book from Casa do Codigo: [Aplicações web real-time com Node.js - Caio Ribeiro Pereira](https://www.casadocodigo.com.br/products/livro-nodejs) (Brazilian Portuguese)
- Node.js documentation
- [Google](http://lmgtfy.com/?q=Node.js)

I can recommend Alura (if you know Portuguese) or Pluralsight for anything you need to learn, actually. They are absolutely amazing.
I also strongly recommend using TypeScript, as it helps you organize your code much better than pure Javascript.

## Conclusion

These were some of my experiences so far with Node.js and some comparisons with other technologies I've worked with.

In my opinion Node.js courses could cover more about deployment, but perhaps that's more related to insfrastructure/DevOps.

Do you work with Node.js or are planning to? Leave your comment below :)
