---
layout: post
title: "Documenting your Node.js API with apiDoc"
excerpt: "How to generate HTML documentation for your endpoints"
tags: [nodejs, javascript, apidoc, documentation, html, api]
date: 2017-10-14T11:10:08+02:00
comments: true
image:
  feature: posts/cover-apidoc.png
---

When you are developing an API and one or more developers (frontend, desktop, mobile, etc) will have to integrate their code with it, it's very important to have it well documented so they know what and how they can use.

For that, in Node.js projects I've been using [apiDoc](http://apidocjs.com/), as it is able to generate documentation in HTML from annotations in the source code.

For this post, I'll use the TODO List API I developed as an example, once more. So you can clone or download it from [here](https://github.com/jonathas/todo-api).

## Routes and annotations

On my post about [tests with mocha and code coverage with istanbul](https://jonathas.com/tests-and-code-coverage-on-node-using-typescript-with-mocha-and-istanbul/), I showed the example of the Task endpoints in our TODO List API:

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

This represents all our endpoints related to tasks in the system. How can we use them? What should the developers that will consume the API send to each of these endpoints?

So far there is no way for them to know other than looking at the code, which shouldn't be needed.

Because of apiDoc, we can achieve that with annotations. The way I configure it, is by writing them before each endpoint configured in the files inside the routes directory. If you are not sure of what I'm talking about, [check here](https://jonathas.com/tests-and-code-coverage-on-node-using-typescript-with-mocha-and-istanbul/) when I mentioned how I configure and organize my Node.js projects.

With annotations, our Task endpoints (inside routes/tasks.ts) would look like this:

{% highlight typescript linenos %}
import Task from "../controllers/tasks";

export = (app) => {

    const endpoint = process.env.API_BASE + "tasks";

    /**
    * @api {post} /api/v1/tasks Create a task
    * @apiVersion 1.0.0
    * @apiName Create
    * @apiGroup Task
    * @apiPermission authenticated user
    *
    * @apiParam (Request body) {String} name The task name
    *
    * @apiExample {js} Example usage:
    * const data = {
    *   "name": "Do the dishes"
    * }
    *
    * $http.defaults.headers.common["Authorization"] = token;
    * $http.post(url, data)
    *   .success((res, status) => doSomethingHere())
    *   .error((err, status) => doSomethingHere());
    *
    * @apiSuccess (Success 201) {String} message Task saved successfully!
    * @apiSuccess (Success 201) {String} id The campaign id
    *
    * @apiSuccessExample {json} Success response:
    *     HTTPS 201 OK
    *     {
    *      "message": "Task saved successfully!",
    *      "id": "57e903941ca43a5f0805ba5a"
    *    }
    *
    * @apiUse UnauthorizedError
    */
    app.post(endpoint, Task.create);

    /**
    * @api {delete} /api/v1/tasks/:id Delete a task
    * @apiVersion 1.0.0
    * @apiName Delete
    * @apiGroup Task
    * @apiPermission authenticated user
    *
    * @apiParam {String} id The task id
    *
    * @apiExample {js} Example usage:
    * $http.defaults.headers.common["Authorization"] = token;
    * $http.delete(url)
    *   .success((res, status) => doSomethingHere())
    *   .error((err, status) => doSomethingHere());
    *
    * @apiSuccess {String} message Task deleted successfully!
    *
    * @apiSuccessExample {json} Success response:
     *     HTTPS 200 OK
     *     {
     *      "message": "Task deleted successfully!"
     *    }
     *
     * @apiUse UnauthorizedError
    */
    app.delete(endpoint + "/:id", Task.delete);

    /**
    * @api {get} /api/v1/tasks/:id Retrieve a task
    * @apiVersion 1.0.0
    * @apiName GetOne
    * @apiGroup Task
    * @apiPermission authenticated user
    *
    * @apiParam {String} id The task id
    *
    * @apiExample {js} Example usage:
    * $http.defaults.headers.common["Authorization"] = token;
    * $http.get(url)
    *   .success((res, status) => doSomethingHere())
    *   .error((err, status) => doSomethingHere());
    *
    * @apiSuccess {String} _id The task id
    * @apiSuccess {String} name The task name
    *
    * @apiSuccessExample {json} Success response:
     *     HTTPS 200 OK
     *     {
     *        "_id": "57e8e94ea06a0c473bac50cc",
     *        "name": "Do the disehs",
     *        "__v": 0
     *      }
     *
     * @apiUse UnauthorizedError
    */
    app.get(endpoint + "/:id", Task.getOne);

    /**
    * @api {get} /api/v1/tasks Retrieve all tasks
    * @apiVersion 1.0.0
    * @apiName GetAll
    * @apiGroup Task
    * @apiPermission authenticated user
    *
    * @apiExample {js} Example usage:
    * $http.defaults.headers.common["Authorization"] = token;
    * $http.get(url)
    *   .success((res, status) => doSomethingHere())
    *   .error((err, status) => doSomethingHere());
    *
    * @apiSuccess {String} _id The task id
    * @apiSuccess {String} name The task name
    *
    * @apiSuccessExample {json} Success response:
    *     HTTPS 200 OK
    *     [{
    *       "_id": "57e8e94ea06a0c473bac50cc",
    *       "name": "Do the disehs"
    *      },
    *      {
    *       "_id": "57e903941ca43a5f0805ba5a",
    *       "name": "Take out the trash"
    *     }]
    *
    * @apiUse UnauthorizedError
    */
    app.get(endpoint, Task.getAll);

    /**
    * @api {put} /api/v1/tasks/:id Update a task
    * @apiVersion 1.0.0
    * @apiName Update
    * @apiGroup Task
    * @apiPermission authenticated user
    *
    * @apiParam {String} id The task id
    *
    * @apiParam (Request body) {String} name The task name
    *
    * @apiExample {js} Example usage:
    * const data = {
    *   "name": "Run in the park"
    * }
    *
    * $http.defaults.headers.common["Authorization"] = token;
    * $http.put(url, data)
    *   .success((res, status) => doSomethingHere())
    *   .error((err, status) => doSomethingHere());
    *
    * @apiSuccess {String} message Task updated successfully!
    *
    * @apiSuccessExample {json} Success response:
     *     HTTPS 200 OK
     *     {
     *      "message": "Task updated successfully!"
     *    }
     *
     * @apiUse UnauthorizedError
    */
    app.put(endpoint + "/:id", Task.update);

};
{% endhighlight %}

As you can see, we have the HTTP method type (post, put, get, delete), the endpoint address, the api version, the kind of permission it needs, the params it requires, the response and the error if the user is not authorized.

In the [official website](http://apidocjs.com/) you can check the documentation and available params for the annotations.

Where is this UnauthorizedError coming from?

## apiDoc settings

There are some settings that can be done for apiDoc and this UnauthorizedError is the one I usually implement.

Create a file called _apidoc.js inside the routes directory with the following content:

{% highlight javascript linenos %}
// ------------------------------------------------------------------------------------------
// General apiDoc documentation blocks and old history blocks.
// ------------------------------------------------------------------------------------------

// ------------------------------------------------------------------------------------------
// Current Success.
// ------------------------------------------------------------------------------------------


// ------------------------------------------------------------------------------------------
// Current Errors.
// ------------------------------------------------------------------------------------------


// ------------------------------------------------------------------------------------------
// Current Permissions.
// ------------------------------------------------------------------------------------------
/**
 * @apiDefine UnauthorizedError
 * @apiVersion 1.0.0
 *
 * @apiError Unauthorized Only authenticated users can access the endpoint.
 *
 * @apiErrorExample  Unauthorized response:
 *     HTTP 401 Unauthorized
 *     {
 *       "message": "Invalid credentials"
 *     }
 */

// ------------------------------------------------------------------------------------------
// History.
// ------------------------------------------------------------------------------------------
{% endhighlight %}

I also create another file, also inside the routes directory, called apidoc.json

This file contains the following content (example):

{% highlight typescript linenos %}
{
    "name": "Test API - This is my API",
    "version": "1.0.0",
    "description": "This is my very powerful API",
    "title": "Test API - This is my API",
    "url": "https://testapi.com"
}
{% endhighlight %}

These two files will be used by apiDoc when generating the documentation.

## Generating the documentation

After writing annotations for every endpoint and configuring our project, we need to configure a task to generate the documentation.

The way I do that is using gulp. Install the required dependencies:

```bash
npm i gulp gulp-apidoc --save-dev
```

Then create a file called gulpfile.js in the project's root directory. If it's there already, just add the parts related to apiDoc to it:

{% highlight javascript linenos %}
const gulp = require("gulp");
const apidoc = require("gulp-apidoc");

gulp.task("apidoc", (done) => {
    apidoc({
        src: "./routes",
        dest: "../docs/apidoc"
    }, done);
});

gulp.task("watch", () => {
    gulp.watch(["./routes/**"], ["apidoc"]);
});
{% endhighlight %}

You can change the "dest" there to another directory that suits you better. That's where you want the output to be generated to.

Now all you need to do is to run:

```bash
gulp apidoc
```

After that, you just need to open the index.html file inside the directory you configured in the "dest" above and you'll see something like this (Click on the image below to enlarge):

[![apidoc](/images/posts/thumbs/apidoc_tn.jpg)](/images/posts/apidoc.png)

So the other developers can generate the same with gulp as well or you can even serve this generated directory via Nginx, for example.

## Conclusion

In this post we saw how to document our APIs with annotations and generate the HTML for them using apiDoc.

Do you use another software for documenting your APIs or do you use apiDoc in a different way? Leave a comment below!
