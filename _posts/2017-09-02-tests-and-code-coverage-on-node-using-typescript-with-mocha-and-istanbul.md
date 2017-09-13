---
layout: post
title: "Tests and code coverage on Node.js using TypeScript with Mocha and Istanbul"
excerpt: "How to configure the environment, write the tests, write the API endpoints and generate the code coverage report"
tags: [tests, api, mocha, istanbul, nodejs, javascript, typescript]
date: 2017-09-02T12:15:08+02:00
comments: true
image:
  feature: posts/cover-tests.jpg
---

In this post we'll see how to configure the environment, write the tests, write the API endpoints and generate the code coverage report.
The steps below will be based on the TODO List API I developed and is available [here](https://github.com/jonathas/todo-api).
For this post I assume you already have Node configured and MongoDB running on localhost.

## Setting up our environment

Before we start, I recommend using [Visual Studio Code](https://code.visualstudio.com/nodejs), as it integrates really well with TypeScript and has amazing features, such as IntelliSense and debugging.

![VSCode website](/images/posts/vscode_20170315.png "VSCode website")

And of course, I'm on their official website for Node.js :)

I also recommend configuring the project by copying some files to its root directory, which will set how TypeScript and Istanbul will work.

These file are:

- [tsconfig.json](https://github.com/jonathas/todo-api/blob/master/api/tsconfig.json)
- [tslint.json](https://github.com/jonathas/todo-api/blob/master/api/tslint.json)
- [.istanbul.yml](https://github.com/jonathas/todo-api/blob/master/api/.istanbul.yml)

I could paste them here but then the post would be even bigger for no reason, so you can just click on the links above and copy them.

We will need a test directory, where our test files will be created. Inside this directory called test, copy the content below to a file called mocha.opts

```bash
bin/test/*_test.js
--reporter spec
```

This is usually the directory structure I have when creating a Node.js project.

![Directory structure](/images/posts/directory_structure.jpg "Directory structure")

As you may notice after opening the gulpfile.js file, I keep there a task that transpiles the TypeScript code to JavaScript in a bin directory and generates sourcemaps to be used later by Istanbul when generating the code coverage. Because of these sourcemaps, Istanbul is able to map correctly the TypeScript files instead of the ugly JavaScript ones that are generated.

The process.yml file you see there contains the settings for pm2 to work when we deploy our server, but I'll leave that for another post.

The .env file contains environment variables that can be accessed throughout the system, such as the timezone, some keys, etc. It is loaded by the dotenv npm package and accessed via process.env.API_BASE (for example, if you declared API_BASE inside the .env file).

Example of a dot env file:

{% highlight bash linenos %}
API_BASE='/api/v1/'
DEFAULT_TIMEZONE='Europe/Prague'

DATETIME_FORMAT='YYYY-MM-DDTHH:mm:ssZ'
DATE_FORMAT='YYYY-MM-DD'
TIME_FORMAT='HH:mm:ss'

DB_IP='127.0.0.1'
{% endhighlight %}

Let's start our project with npm:

```bash
npm init
```

For our build process to work, we need to install the required packages:

```bash
npm i express express-validator@2.20.8 mongoose dotenv body-parser bluebird --save
```

and for the dev dependencies:

```bash
npm i tslint typescript vinyl-fs supertest mocha istanbul@1.0.0-alpha.2 gulp-typescript gulp-sourcemaps gulp chai @types/node @types/mocha @types/express @types/mongoose --save-dev
```

Inside the package.json file used by npm, I usually leave "main" as "server.js" and set some values inside the "scripts" part:

{% highlight javascript linenos %}
"scripts": {
    "pretest": "gulp",
    "test": "istanbul cover --report cobertura --report lcov node_modules/mocha/bin/_mocha",
    "prestart": "gulp",
    "start": "nodemon bin/server.js"
}
{% endhighlight %}

Inside "start" it can be nodemon, pm2 or any other tool you use to run it for development. I recommend nodemon, as it is well suited for that. The way I deploy it, it doesn't matter what command is written there.

This command inside "test" runs istanbul, which generates the coverage report using the tests that are run by mocha.

You can find an example of a package.json file I use [here](https://github.com/jonathas/todo-api/blob/master/api/package.json).

Create a file called gulpfile.js in the root of the project with the following content:

{% highlight javascript linenos %}
const gulp = require("gulp");
const sourcemaps = require("gulp-sourcemaps"); // So Istanbul is able to map the code
const tsProject = require("gulp-typescript").createProject("tsconfig.json");
const vfs = require("vinyl-fs");

const paths = {
    tocopy: ["./**", "!./bin", "!./bin/**", "!./**/*.ts", "!./.vscode", "!./.vscode/**", "!./gulpfile.js", "!./*.log", "!./*.lock", "!./*.md", "!./*.json", "!./*.yml", "!./LICENSE",
        "!./test/unit/*.ts", "!./test/mocha.opts", "!./coverage", "!./coverage/**", "!./routes/**apidoc**", "!./tasks", "!./tasks/**", "!./node_modules", "!./node_modules/**", "!./dist", "!./dist/**"
    ],
    tsfiles: ["./config", "./controllers", "./models", "./routes/*.ts", "./test"]
};

gulp.task("copy", () => {
    return vfs.src(paths.tocopy, {
            dot: true
        })
        .pipe(vfs.dest("./bin"));
});

gulp.task("transpile", () => {
    return tsProject.src()
        .pipe(sourcemaps.init())
        .pipe(tsProject()).js
        .pipe(sourcemaps.write("./maps"))
        .pipe(gulp.dest("./bin"));
});

gulp.task("default", ["copy", "transpile"], (done) => done());

gulp.task("watch", () => {
    gulp.watch(paths.tsfiles, ["transpile"]);
});
{% endhighlight %}

You will need to have a MongoDB instance up and running, so you can install it directly on your OS or use a Docker image. This part will not be covered by this post.

In [this link](https://github.com/jonathas/todo-api/blob/master/.gitignore) you can find the .gitignore file I usually use for Node.js projects. It's important to have it so when you push your code to a repository the node_modules directory and other files that should not be versioned don't get sent there.

## Writing the tests

I usually create inside the test directory a common.ts file that is imported inside every test file. Each test file represents a controller and has a _test suffix.
Inside this common.ts file, we declare the environment as test, import mocha, initialize supertest, etc.

{% highlight typescript linenos %}
process.env.NODE_ENV = "test";

import "mocha";
import * as Task from "../models/task";

const express = require("../config/express")();

export const request = require("supertest")(express);

export const chai = require("chai");
export const should = chai.should();

export const cleanCollections = (): Promise<any> => {
    const cleanUp = [
        Task.cleanCollection()
        // Add more
    ];
    return Promise.all(cleanUp);
};
{% endhighlight %}

Our TODO API will have tasks, which are items in our TODO list. We can write some tests for our tasks so we assure they work the way we need them to.
Let's create a file called tasks_test.ts inside the test directory and add some tests to it.
I commented out the login and authentication parts of it, as this post is not about authentication and I plan to write about this subject later, but you can see now already how it could be implemented.

{% highlight typescript linenos %}
import { request, chai } from "./common";

describe("# Tasks", () => {
    const endpoint = process.env.API_BASE + "tasks";
    let token;
    let taskId;

    /*
        We could configure this "before" to run before all tests here, login and get a valid token for authentication in the following API calls.
        For that to work, I usually implement a login function inside the common.ts file and import it along with request and chai above.
        before(() => {
            return login().then(res => {
                token = res.body.token;
                return res;
            })
        });
    */

    it("should add some tasks", () => {
        return request.post(endpoint)
            // This is how we can configure authentication once we implement it
            // .set("Authorization", token)
            .send({ "name": "Do the dishes" })
            .expect(res => res.body.message.should.equal("Task saved successfully!"))
            .expect(201)
            .then(res => {
                return request.post(endpoint)
                .send({ "name": "Run in the park" })
                .expect(res => {
                    taskId = res.body.id;
                    res.body.message.should.equal("Task saved successfully!");
                })
                .expect(201);
            });
    });

    it("should not add a task when no data is sent", () => {
        return request.post(endpoint)
            .send({})
            .expect(res => {
                res.body.message.should.equal("Missing parameters");
            })
            .expect(400);
    });

    it("should update a task", () => {
        return request.put(endpoint + "/" + taskId)
            .send({ name: "Take out the trash" })
            .expect(res => res.body.message.should.equal("Task updated successfully!"))
            .expect(200);
    });

    it("should not update a task when no data is sent", () => {
        return request.put(endpoint + "/" + taskId)
            .send({})
            .expect(400);
    });

    it("should return bad request for trying to update a task with a malformed id", () => {
        return request.put(endpoint + "/anything")
            .send({ name: "something else" })
            .expect(400);
    });

    it("should retrieve a task", () => {
        return request.get(endpoint + "/" + taskId)
            .set("Accept", "application/json")
            .expect("Content-Type", /json/)
            .expect(res => res.body._id.should.equal(taskId))
            .expect(200);
    });

    it("should retrieve all tasks", () => {
        return request.get(endpoint)
            .set("Accept", "application/json")
            .expect("Content-Type", /json/)
            .expect(res => chai.expect(res.body).to.have.length.of.at.least(1))
            .expect(200);
    });

    it("should return bad request for trying to retrieve a task with a malformed id", () => {
        return request.get(endpoint + "/anything")
            .set("Accept", "application/json")
            .expect("Content-Type", /json/)
            .expect(400);
    });

    it("should return not found for a non existent task", () => {
        return request.get(endpoint + "/57d6e440b80470c440b3401f")
            .set("Accept", "application/json")
            .expect("Content-Type", /json/)
            .expect(404);
    });

    it("should return bad request for trying to delete a task with a malformed id", () => {
        return request.delete(endpoint + "/anything")
            .set("Accept", "application/json")
            .expect("Content-Type", /json/)
            .expect(400);
    });

    it("should delete a task", () => {
        return request.delete(endpoint + "/" + taskId)
            .expect(res => res.body.message.should.equal("Task deleted successfully!"))
            .expect(200);
    });

});
{% endhighlight %}

As you can notice, now we have some tests accesssing our API and the responses they expect to get for each call. We still have no routes configured for that, so nothing would work so far. Let's get to it.

## The Config directory

Before configuring the routes, we need to configure Express and the database.
Inside the config directory I create files to configure express, the database, the cache when it's being used, the logging tool, etc.

An express.ts file would contain the following code:

{% highlight typescript linenos %}
const dotenv = require("dotenv").config();
const express = require("express");
const bodyParser = require("body-parser");
const expressValidator = require("express-validator");
const db = require("./db");

export = () => {
    let app = express();

    app.use(bodyParser.json());
    app.use(expressValidator());

    const routes = require("../routes")(app);

    return app;
};
{% endhighlight %}

The db.ts file:

{% highlight typescript linenos %}
import * as mongoose from "mongoose";
(<any>mongoose).Promise = require("bluebird");
let dbName;

switch (process.env.NODE_ENV) {
    case "test":
        dbName = "todo_test";
        break;
    case "production":
        dbName = "todo";
        break;
    default:
        dbName = "todo_dev";
}

const dbAddress = process.env.DB_HOST || "127.0.0.1";
const dbPort = process.env.DB_PORT || 27017;

let options = {
    useMongoClient: true
};

if (process.env.DB_AUTH === "true") {
    options["user"] = process.env.DB_USER;
    options["pass"] = process.env.DB_PASS;
}

mongoose.connect(`mongodb://${dbAddress}:${dbPort}/${dbName}`, options).catch(err => {
    if (err.message.indexOf("ECONNREFUSED") !== -1) {
        console.error("Error: The server was not able to reach MongoDB. Maybe it's not running?");
        process.exit(1);
    } else {
        throw err;
    }
});
{% endhighlight %}

The process.env.DB_IP value is coming from what was declared in the .env file

## Routes

As you could notice above in the express.ts file, the routes directory was required. When require is called like that, if there is an index file in the directory, that's the file that gets required.

We can now create an index.ts file inside the routes directory with the following content:

{% highlight typescript linenos %}
export = (app) => {

    // Add here the routes for the controllers
    require("./tasks")(app);

    app.get("/", (req, res) => res.status(200).json({ message: "Welcome to the TODO API. Check the documentation for the list of available endpoints" }));

    // If no route is matched by now, it must be a 404
    app.use((req, res, next) => {
        res.status(404).json({ "error": "Endpoint not found" });
        next();
    });

    app.use((error, req, res, next) => {
        if (process.env.NODE_ENV === "production") {
            return res.status(500).json({ "error": "Unexpected error: " + error });
        }
        next(error);
    });

};
{% endhighlight %}

There we require only the file with the routes for the tasks endpoints, but we could also require other files with other endpoints as well, as our system grows and we add more functionality.

The tasks.ts file inside the routes directory will have the following content:

{% highlight typescript linenos %}
import Task from "../controllers/tasks";

export = (app) => {

    const endpoint = process.env.API_BASE + "tasks";

    app.post(endpoint, Task.create);

    app.delete(endpoint + "/:id", Task.delete);

    app.get(endpoint + "/:id", Task.getOne);

    app.get(endpoint, Task.getAll);

    app.put(endpoint + "/:id", Task.update);

};
{% endhighlight %}

It's important here to use the correct HTTP methods. POST for creating, PUT for updating, GET for retrieving and DELETE for deleting data.

Documentation for these endpoints can be automatically generated by running the gulp task called apidoc that I have in this [gulpfile.js example](https://github.com/jonathas/todo-api/blob/master/api/gulpfile.js) file. In order to generate this documentation, each endpoint will need to have commments using a specific format. You can find [here](https://github.com/jonathas/todo-api/blob/master/api/routes/tasks.ts) the tasks.ts file with these comments if you want to generate this documentation. I simplified it for this post as we're now focusing on tests instead.

## Controllers

Now we must declare the controller methods we called in the router. Let's create a tasks.ts file inside the controllers directory.

{% highlight typescript linenos %}
import { model as Task } from "../models/task";

class Tasks {

    public getAll = async (req, res) => {
        try {
            const tasks = await Task.find({}).exec();
            res.status(200).json(tasks);
        } catch (err) {
            /* istanbul ignore next */
            res.status(400).json(err);
        }
    }

    public getOne = async (req, res) => {
        try {
            let task = await Task.findById(req.params.id).exec();

            if (task === null) {
                return res.status(404).json({ message: "This task doesn't exist" });
            }

            res.status(200).json(task);
        } catch (err) {
            res.status(400).json(err);
        }
    }

    public create = async (req, res) => {
        try {
            this.validateRequest(req);

            let Data = new Task(req.body);
            await Data.save();

            res.status(201).json({ "message": "Task saved successfully!", "id": Data._id });
        } catch (err) {
            res.status(400).json({ "message": "Missing parameters", errors: err });
        }
    }

    public update = async (req, res) => {
        try {
            this.validateRequest(req);

            await Task.findByIdAndUpdate(req.params.id, req.body);

            res.status(200).json({ "message": "Task updated successfully!" });
        } catch (err) {
            res.status(400).json({ "message": "Missing parameters", errors: err });
        }
    }

    private validateRequest = (req) => {
        req.checkBody("name", "The name cannot be empty").notEmpty();

        let errors = req.validationErrors();
        if (errors) throw errors;
    }

    public delete = async (req, res) => {
        try {
            await Task.findByIdAndRemove(req.params.id);
            res.status(200).json({ "message": "Task deleted successfully!" });
        } catch (err) {
            res.status(400).json({ "message": `Error delete task: ${err}` });
        }
    }

}

export default new Tasks();
{% endhighlight %}

Some notes about it:

- The parameters req and res are the request and response objects from Express.
- I'm using async/await instead of callbacks or promises, as it makes our code look much cleaner.
- In order to keep the context of "this", [TypeScript recommends using the fat arrow for the methods](https://github.com/Microsoft/TypeScript/wiki/%27this%27-in-TypeScript).
- req.checkBody comes from our express-validator middleware. Check more about it [here](https://github.com/ctavan/express-validator).
- We instantiate an object with our Tasks class as soon as it is imported inside the tasks routes file.

## Models

Now we need to define a model for our tasks. Let's create the file task.ts inside the models directory, as it was imported above in the tasks controller.

{% highlight typescript linenos %}
import * as mongoose from "mongoose";

export interface ITask extends mongoose.Document {
    name: {
        type: string,
        require: true
    };
    scheduled_date: Date;
}

export const schema = new mongoose.Schema({
        name: {
        type: String,
        require: true
    },
    scheduled_date: {
        type: Date,
        default: new Date()
    }
}, { timestamps: { createdAt: "created_at", updatedAt: "updated_at" } });

export const model = mongoose.model<ITask>("Task", schema);

export const cleanCollection = (): Promise<any> => model.remove({}).exec();

export default model;
{% endhighlight %}

## The server file

Let's create the file that will start our API. Create the file server.ts in the root of your project:

{% highlight typescript linenos %}
const app = require("./config/express")();

const port = process.env.PORT || 3000;

const server = app.listen(port, () => {
    console.log(`Server listening on port ${port}. Environment: ${process.env.NODE_ENV}`);
});{% endhighlight %}

## Running the tests and checking the code coverage

We are ready to run our tests and check how much of the code is covered by them.

Run the following command configured in the package.json file:

```bash
npm test
```

This will run gulp to transpile our TypeScript files to JavaScript (es6), generate the sourcemaps and then run Mocha with supertest, which in turn will access our endpoints and generate the test results.

![Tests running](/images/posts/tests.jpg "Tests running")

If all tests pass, the code coverage is generated by Istanbul and stored inside the coverage directory. Open the index.html file inside coverage/lcov-report to see it.

![Code coverage report](/images/posts/coverage.png "Code coverage report")

My next post will be about how to configure Visual Studio Code for debugging and transpiling TypeScript to JavaScript automatically.

## Conclusion

In this post we saw how to configure an environment for Visual Studio Code with TypeScript for tests and how to generate code coverage using Istanbul for them.
More tests and functionalities can be added later using this base.

Suggestions? Leave a comment :)
