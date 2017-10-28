---
layout: post
title: "Token based authentication in Node.js with Passport, JWT and bcrypt"
excerpt: "Forget about the cookies and sessions. Time to go stateless!"
tags: [nodejs, javascript, passport, jwt, bcrypt, api, token]
date: 2017-10-21T13:14:08+02:00
comments: true
image:
  feature: posts/cover-authentication.jpg
---

When you develop an API, most of the times you'll need part or all of its endpoints to require authentication. How to do that using Node.js?

A combination of [passport.js](http://passportjs.org/) with JWT and bcrypt is one of the best ways to implement it. Time to go [stateless](https://www.jbspeakr.cc/purpose-jwt-stateless-authentication/)!

This post is based on the authentication implemented in the TODO List API project that you can find [here](https://github.com/jonathas/todo-api). In this post, all the code is in TypeScript and I expect you to have Node.js and MongoDB already configured on your OS.

## What is Passport

As its website states: "Passport is an authentication middleware for Node.js. Extremely flexible and modular, Passport can be unobtrusively dropped in to any Express-based web application. A comprehensive set of strategies support authentication using a username and password, Facebook, Twitter, and more".

So Passport allows us to integrate login strategies for many kinds of services and they have currently more than 300 strategies that can be just plugged in, ready to be used.

## What is JWT

JWT (JSON Web Token) is an open, industry standard [RFC 7519](https://tools.ietf.org/html/rfc7519) method for representing claims securely between two parties. If we use Passport with a strategy for JWT, then it generates tokens that look for example like this:

```bash
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
```

A token contains its expiration date and can also contain data we need for checking the user.

The token goes in the Authorization header of the HTTP method call, so the Passport middleware extracts and validates it.

## What is bcrypt

In order to save the user in the database and later compare the password he/she enters for generating the authentication token, we need to encrypt his/her password, as it's not safe to keep passwords with no encryption in the database. Also, it would be a joke to use md5 for that and [sha1 recently became unsafe](https://www.computerworld.com/article/3173616/security/the-sha1-hash-function-is-now-completely-unsafe.html). Enter bcrypt.

From Wikipedia: "bcrypt is a password hashing function designed by Niels Provos and David Mazières, based on the Blowfish cipher, and presented at USENIX in 1999".
[Here is more on why](https://medium.com/@danboterhoven/why-you-should-use-bcrypt-to-hash-passwords-af330100b861) you should use bcrypt to hash passwords.

## Configuring the project

In order to implement authentication in our API, we need to install the following packages:

```bash
npm i bcryptjs dotenv moment express bluebird body-parser mongoose express-validator@2.20.8 jwt-simple passport passport-jwt@2.2.1 --save
```

And the dev dependencies (As I've been using TypeScript, I also install some types):

```bash
npm i @types/bcryptjs @types/jwt-simple @types/passport @types/passport-jwt @types/node @types/mongoose @types/superagent chai gulp gulp-apidoc gulp-sourcemaps gulp-typescript gulp-util istanbul@1.0.0-alpha.2 mocha@3.2.0 supertest tslint typescript vinyl-fs --save-dev
```

The passport-jwt package is Passport's strategy for JWT.

Inside the package.json file, in "scripts", let's leave it like this so istanbul runs integrated with mocha for generating the code coverage report:

{% highlight javascript linenos %}
"scripts": {
    "pretest": "gulp",
    "test": "istanbul cover node_modules/mocha/bin/_mocha",
    "prestart": "gulp",
    "start": "nodemon bin/server.js"
}
{% endhighlight %}

Inside the test directory, create a file called mocha.opts with the following content:

```bash
bin/test/*_test.js
--reporter spec
```

Let's copy these files and save them to the root of our project:

- [.istanbul.yml](https://github.com/jonathas/todo-api/blob/master/api/.istanbul.yml)
- [gulpfile.js](https://github.com/jonathas/todo-api/blob/master/api/gulpfile.js)
- [tsconfig.json](https://github.com/jonathas/todo-api/blob/master/api/tsconfig.json)
- [tslint.json](https://github.com/jonathas/todo-api/blob/master/api/tslint.json)

In the root of our project, let's create a file called .env if it isn't there yet. This file needs to have the JWT_SECRET environment variable, with a random hash we need to generate.
It's very important not to repeat this hash among your projects, as it has to be something unique and secure, because this is what is used for generating and comparing the token.

You can generate a hash using the [LastPass password generator](https://lastpass.com/generatepassword.php), for example. I recommend you to use all kinds of characters and to have the length of at least 25. An example of an .env file then would be:

```bash
API_BASE='/api/v1/'
DEFAULT_TIMEZONE='Europe/Prague'

JWT_SECRET='ogA9ppB$S!dy!hu3Rauvg!L96'

DB_HOST='127.0.0.1'
DB_PORT='27017'
DB_AUTH='false'
DB_USER=''
DB_PASS=''
```

In the config directory, let's create a db.ts file for the database connection:

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

## Auth tests

Now let's write some tests for what we need our authentication to do.

Create a file named common.ts inside the test directory:

{% highlight typescript linenos %}
process.env.NODE_ENV = "test";

import "mocha";
import { IUser, model as User } from "../models/user";

const express = require("../config/express")();

export const request = require("supertest")(express);

export const chai = require("chai");
export const should = chai.should();

const testUser = { "username": "testuser", "password": "mytestpass" };

const createUser = async (): Promise<void> => {
    const UserModel = new User(testUser);
    await UserModel.save();
};

const getUser = async (): Promise<IUser> => {
    let users = await User.find({});
    if (users.length === 0) {
        await createUser();
        return getUser();
    } else {
        return users[0];
    }
};

export const login = async (): Promise<any> => {
    let user = await getUser();
    return request.post(process.env.API_BASE + "login")
        .send({ "username": user.username, "password": testUser.password })
        .expect(200);
};
{% endhighlight %}

And then some tests for authentication in a file called auth_test.ts inside the test directory:

{% highlight typescript linenos %}
import { request, login } from "./common";
import { cleanCollection } from "../models/user";

describe("# Auth", () => {
    const endpoint = process.env.API_BASE + "login";

    it("should retrieve the token", () => {
        return cleanCollection().then(res => {
            return login().then(res => {
                res.status.should.equal(200);
                res.body.token.should.not.be.empty;
            });
        });
    });

    it("should not login with the right user but wrong password", () => {
        return request.post(endpoint)
            .send({ "username": "testuser", "password": "anythingGoesHere" })
            .expect(401);
    });

    it("should return invalid credentials error", () => {
        return request.post(endpoint)
            .send({ "username": "testuser", "password": "" })
            .expect(401)
            .then(res => {
                return request.post(endpoint)
                    .send({ "username": "anotherusername", "password": "mypass" })
                    .expect(401);
            });
    });

    it("should return token expired message", () => {
        return request.post(process.env.API_BASE + "tasks")
            .set("Authorization", "JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE0OTg5Mzk1MTksInVzZXJuYW1lIjoidGVzdHVzZXIifQ.FUJcVCzZTkjDr62MCJj5gvCFvmxewmz2jotiknuVbOg")
            .send({
                name: "Do the dishes"
            })
            .expect(res => res.body.message.should.equal("Your token has expired. Please generate a new one"))
            .expect(401);
    });
});
{% endhighlight %}

Now we have tests for some things related to our JWT implementation. We need then to configure Express to use Passport as a middleware.

## Auth middleware

Inside the config directory, let's create a file called express.ts to configure everything related to Express.

{% highlight typescript linenos %}
const dotenv = require("dotenv").config();
const express = require("express");
const auth = require("../controllers/auth").default;
const bodyParser = require("body-parser");
const expressValidator = require("express-validator");
const db = require("./db");

export = () => {
    let app = express();

    app.use(bodyParser.json());
    app.use(expressValidator());

    app.use(auth.initialize());

    app.all(process.env.API_BASE + "*", (req, res, next) => {
        if (req.path.includes(process.env.API_BASE + "login")) return next();

        return auth.authenticate((err, user, info) => {
            if (err) { return next(err); }
            if (!user) {
                if (info.name === "TokenExpiredError") {
                    return res.status(401).json({ message: "Your token has expired. Please generate a new one" });
                } else {
                    return res.status(401).json({ message: info.message });
                }
            }
            app.set("user", user);
            return next();
        })(req, res, next);
    });

    const routes = require("../routes")(app);

    return app;
};
{% endhighlight %}

In the above code, if our endpoint is "login", it will not check for authentication (return next()), as that is the endpoint the clients will use for sending the username and password for generating the token they need for the other endpoints. Any other endpoint will go through the authenticate method inside our Auth controller, which is using Passport.

As you see in the code, the Auth middleware must come before the routes are required, as the authentication process needs to happen before them.

## Auth controller and route

Inside the controllers directory, let's create a file called auth.ts with the following content:

{% highlight typescript linenos %}
import * as jwt from "jwt-simple";
import * as passport from "passport";
import * as moment from "moment";
import { Strategy, ExtractJwt } from "passport-jwt";
import { model as User, IUser } from "../models/user";

class Auth {

    public initialize = () => {
        passport.use("jwt", this.getStrategy());
        return passport.initialize();
    }

    public authenticate = (callback) => passport.authenticate("jwt", { session: false, failWithError: true }, callback);

    private genToken = (user: IUser): Object => {
        let expires = moment().utc().add({ days: 7 }).unix();
        let token = jwt.encode({
            exp: expires,
            username: user.username
        }, process.env.JWT_SECRET);

        return {
            token: "JWT " + token,
            expires: moment.unix(expires).format(),
            user: user._id
        };
    }

    public login = async (req, res) => {
        try {
            req.checkBody("username", "Invalid username").notEmpty();
            req.checkBody("password", "Invalid password").notEmpty();

            let errors = req.validationErrors();
            if (errors) throw errors;

            let user = await User.findOne({ "username": req.body.username }).exec();

            if (user === null) throw "User not found";

            let success = await user.comparePassword(req.body.password);
            if (success === false) throw "";

            res.status(200).json(this.genToken(user));
        } catch (err) {
            res.status(401).json({ "message": "Invalid credentials", "errors": err });
        }
    }

    private getStrategy = (): Strategy => {
        const params = {
            secretOrKey: process.env.JWT_SECRET,
            jwtFromRequest: ExtractJwt.fromAuthHeader(),
            passReqToCallback: true
        };

        return new Strategy(params, (req, payload: any, done) => {
            User.findOne({ "username": payload.username }, (err, user) => {
                /* istanbul ignore next: passport response */
                if (err) {
                    return done(err);
                }
                /* istanbul ignore next: passport response */
                if (user === null) {
                    return done(null, false, { message: "The user in the token was not found" });
                }

                return done(null, { _id: user._id, username: user.username });
            });
        });
    }

}

export default new Auth();
{% endhighlight %}

As you can see, here we set the Passport initialize method, configured the token generation with a validity of 7 days, implemented the login method to be used in the login endpoint and the strategy using JWT, extracting the token from the Authorization header.

We need now a route for the login endpoint, for this to work. Let's create a file called auth.ts inside the routes directory:

{% highlight typescript linenos %}
import Auth from "../controllers/auth";

export = (app) => {
    /**
    * @api {post} /api/v1/login Generate a token
    * @apiVersion 1.0.0
    * @apiName Login
    * @apiGroup Auth
    * @apiPermission public
    * @apiDescription In order to generate a token, you will need to already have a user in the database.
    *
    * @apiParam (Request body) {String} username The username
    * @apiParam (Request body) {String} password The password
    *
    * @apiExample {js} Example usage:
    * const data = {
    *   "username": "test@email.com",
    *   "password": "yourpassword"
    * };
    *
    * $http.post(url, data)
    *   .success((res) => doSomethingHere())
    *   .error((err) => doSomethingHere());
    *
    * @apiSuccess {String} token The token that must be used to access the other endpoints
    * @apiSuccess {String} expires The expiration datetime (YYYY-MM-DDTHH:mm:ssZ)
    * @apiSuccess {String} user The user id
    *
    * @apiSuccessExample {json} Success response:
     *     HTTPS 200 OK
     *     {
     *      "token": "JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9 ... and the rest of the token here",
     *      "expires": "2017-10-28T14:50:17+00:00",
     *      "user": "57e12cab65c0c892381b8b44"
     *    }
    */
    app.post(process.env.API_BASE + "login", Auth.login);
};
{% endhighlight %}

This example is ready to be used with apiDoc for generating the API documentation, explained in my [previous post](https://jonathas.com/documenting-your-nodejs-api-with-apidoc/).

We also need an index.ts file inside the routes directory, in case you don't have it:

{% highlight typescript linenos %}
export = (app) => {

    // Require the routes files in the routes directory
    require("./auth")(app);

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

Now that we have the project configured with the packages, the auth tests, auth middleware, auth controller and routes, we need to implement the bcrypt functions for checking the user password before generating the token.

## User model

Let's create the file called user.ts inside the models directory:

{% highlight typescript linenos %}
import * as mongoose from "mongoose";
import * as bcrypt from "bcryptjs";

export interface IUser extends mongoose.Document {
    name: string;
    username: string;
    password: string;
    comparePassword(candidatePassword: string): Promise<boolean>;
}

export const schema = new mongoose.Schema({
    name: String,
    username: {
        type: String,
        required: true,
        unique: true
    },
    password: {
        type: String,
        required: true
    }
}, { timestamps: { createdAt: "created_at", updatedAt: "updated_at" } });

schema.pre("save", function (next) {
    bcrypt.hash(this.password, 10, (err, hash) => {
        this.password = hash;
        next();
    });
});

schema.pre("update", function (next) {
    bcrypt.hash(this.password, 10, (err, hash) => {
        this.password = hash;
        next();
    });
});

schema.methods.comparePassword = function (candidatePassword: string): Promise<boolean> {
    let password = this.password;
    return new Promise((resolve, reject) => {
        bcrypt.compare(candidatePassword, password, (err, success) => {
            if (err) return reject(err);
            return resolve(success);
        });
    });
};

export const model = mongoose.model<IUser>("User", schema);

export const cleanCollection = () => model.remove({}).exec();

export default model;
{% endhighlight %}

Here we added bcrypt methods to hash a new password when saving or updating a user and implemented a method to compare the password the user will post to the login endpoint when he wants to generate a token. This comparePassword method is used in the Auth controller.

How to create a user for the first login?

The common.ts file inside the test directory does that using the User model we just created, so when our tests run and try to login, if no user exists, it creates a user before accessing the login endpoint for posting the credentials. For a production case you can create a users controller inside the controllers directory or [copy the one I have here](https://github.com/jonathas/todo-api/blob/master/api/controllers/users.ts), then create user routes inside the routes directory or [copy the one I have here](https://github.com/jonathas/todo-api/blob/master/api/routes/users.ts) to make the endpoints available for a web panel to create new users, for example. Or you could create a script that runs in the CLI and uses bcrypt and the User model to create new users in the database. I will not cover user management in this post, as the focus here is the authentication part.

## The server file

We need to run our Express config and create a server with it. Let's create a server.ts file in the root of our project:

{% highlight typescript linenos %}
const app = require("./config/express")();

const port = process.env.PORT || 3000;

const server = app.listen(port, () => {
    console.log(`Express server listening on port ${port}.\nEnvironment: ${process.env.NODE_ENV}`);
});
{% endhighlight %}

## Running the tests

Now that we configured everything that is used for requiring authentication for our endpoints, we can run the tests and check if the code is working (be sure to have MongoDB running before):

```bash
npm test
```

![Auth tests](/images/posts/auth_tests.jpg "Auth tests")

## Calling an endpoint from a client

After you integrate the authentication on your API, you can use your client (javascript on the browser, mobile, desktop, postman, etc) to call the login endpoint and get the token for your user.

Run the project with node, nodemon or pm2 (edit that in the package.json file):

```bash
npm start
```

then:

![Login](/images/posts/login.png "Login")

The response comes with "JWT" before the actual token. This is the token bearer and it is required when sending the token in the Authorization header.

If you are using the TODO List API example, you can use the tasks endpoint sending the Authorization header using your client ([Postman](https://www.getpostman.com/) in the example), with the token generated in the previous step:

![Using Authorization](/images/posts/using_authorization.png "Using Authorization")

## Conclusion

In this post we saw how to implement authentication with Passport, JWT and bcrypt, integrating it to our existing API, then calling it from a client. As our authentication is stateless, when we need to deploy the API behind a load balancer, we don't need to have sticky sessions and the authentication process is much simpler and easily scalable.

If you have any suggestion, doubt or use something else, leave a comment below :)
