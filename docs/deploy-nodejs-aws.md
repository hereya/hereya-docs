---
sidebar_position: 2
---

# Deploy an API to AWS

In this tutorial, we will develop and deploy a simple Node.js REST Api with mongodb to AWS using Hereya.

In the following steps, you will learn how to:

- Add mongodb database to your project with a single command.
- Develop a simple Node.js REST Api.
- Deploy your app to AWS with a single command.

## Prerequisites

To follow this tutorial, you need to have the following:

- An AWS account and [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) 
  installed on your machine and configured with your AWS account credentials. For this tutorial we recommend using a 
  development AWS account and not the account you use for production. Make sure that you have admin access to the 
  AWS account.
- [Node.js](https://nodejs.org/en/download/) version 18.0 or above.
- [Docker](https://docs.docker.com/get-docker/): *hereya* will use docker to run a mongodb database locally.
  - You can install it by following the instructions on the [official website](https://docs.docker.com/get-docker/).

## What you will build

You will build a simple Node.js REST Api that connects to a mongodb database. The app is a simple movies REST Api 
with the following endpoints:

- `GET /movies`: Get all movies.
- `POST /movies`: Create a new movie.
- `GET /movies/:id`: Get a movie by id.
- `PUT /movies/:id`: Update a movie by id.
- `DELETE /movies/:id`: Delete a movie by id.

## Install Hereya

You can install **Hereya** by running the following command:

```bash
npm install -g hereya-cli 
```

## Set up the Node.js project

Create a new directory for your project and navigate to it:

```bash
mkdir hereya-tutorial-aws # You can name it whatever you want
cd hereya-tutorial-aws
```

Initialize a new Node.js project:

```bash
npm init -y
```

Set the type of your project to `module` in the `package.json` file:

```json
{
  "type": "module"
}
```

Your `package.json` file should look like this:

```json
{
  "name": "hereya-tutorial-aws",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "type": "module",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```

## Initialize Hereya in your project

When hereya adds packages to your app, it installs them in the context of a **workspace**. A workspace is like an
execution environment for your app. You might have for example `dev`, `staging`, and `production` workspaces.

Create a new workspace named `dev`:

```bash
hereya workspace create dev
```

Initialize Hereya in your project:

```bash
hereya init hereya-tutorial-aws -w dev # Replace hereya-tutorial-aws with the name of your project
```
This command will create a `hereya.yaml` file in the root of your project with the following content:

```yaml
name: hereya-tutorial-aws
workspace: dev
```

## Write the Node.js REST Api

First, install the required packages:

```bash
npm install express body-parser
```

Create a new file named `index.js` in the root of your project and add the following code:

```javascript
import express from 'express';
import bodyParser from 'body-parser';

const app = express();

app.use(bodyParser.json());

app.get('/', (req, res) => {
    res.send(
        `Welcome to Movies API. You can use this API to get and add movies at /movies.`
    );
});

app.get('/movies', async (req, res) => {
    res.json([]);
});

app.get('/movies/:id', async (req, res) => {
    const id = req.params.id;
    // todo: get movie by id
    res.json({});
})

app.post('/movies', async (req, res) => {
    const {name, year} = req.body;
    // todo: create movie
    res.json({})
});


app.put('/movies/:id', async (req, res) => {
    const {name, year} = req.body;
    const id = req.params.id;
    // todo: update movie
    res.end();
});

app.delete('/movies/:id', async (req, res) => {
    const id = req.params.id;
    // todo: delete movie
    res.end();
});


const port = process.env.PORT || 8080;


app.listen(port, () => {
    console.log(`Server is running on port ${port}`);
});
```
In this code snippet, we created a simple express server with `/movies` endpoints. 
You can run `node index.js` to start the server and test the endpoints.

We will implement the logic for storing and retrieving movies from a mongodb database in the next steps.

## Add mongodb to your project

In this step, we will add a mongodb database to our project with a single command.

Without *hereya*, you would need to set up a mongodb database, configure the connection string, and provide it as 
an environment variable to your app.

With *hereya*, you can do all of these steps and more with a single command. *hereya* manage infrastructure through reusable infrastructure as code modules called *packages*. 


Let's add the package [hereya/mongo](https://github.com/hereya/mongo) to our project. Make sure *Docker* is running on 
your machine.

```bash
hereya add hereya/mongo
```

This command deploys a mongodb database in a docker container and save the connection string to be used in your app.

if you open the file `hereya.yaml`, you will see that the `hereya/mongo` package has been added to your project:

```yaml
name: hereya-tutorial-aws
workspace: dev
packages:
  hereya/mongo:
    version: ""
    onDeploy:
     pkg: hereya/aws-documentdb
     version: ""
```

`hereya.yaml` should be checked into your version control system. It is similar to `package.json` in a Node.js project.

You can notice that the `hereya/mongo` package has an `onDeploy` section. This section specifies that when you 
deploy your project with the command `hereya deploy`, the package [hereya/aws-documentdb](https://github.com/hereya/aws-documentdb) will be deployed instead of `hereya/mongo` which can only be used in a local environment. We will see how to deploy the project to AWS later.

## Connect to the mongodb database

`hereya/mongo` exports the following variables: `mongoUrl`, `mongoDbname`.
`hereya/aws-documentdb`, which will be used in the AWS environment, exports additional variables: `mongoUsername` and `mongoPassword`.

You can print the environment variables exported by all installed packages by running:

```bash
hereya env -l
```

You don't need to manage these environment variables manually. *Hereya* will automatically set them in your app when 
you run it with the `hereya run` command.

In order to connect to mongodb, we need a mongodb client. You can use any client you want, but in this tutorial, we 
will use the `mongodb` npm package.

Install the `mongodb` package:

```bash
npm install mongodb
```

Now create a new file named `db.js` in the root of your project and add the following code:

```javascript
import {MongoClient, ObjectId} from "mongodb";

const dbName = process.env.mongoDbname;
let mongoUrl = process.env.mongoUrl;
const username = process.env.mongoUsername;
const password = process.env.mongoPassword;


export const mongoClient = new MongoClient(mongoUrl, password ? {
    auth: {username, password},
} : {})


export async function createMovie({name, year}) {
    await mongoClient.connect();
    const result = await mongoClient.db(dbName).collection('movies').insertOne({name, year});
    return result.insertedId;
}

export async function getMovies() {
    return mongoClient.db(dbName).collection('movies').find({}).toArray();
}

export async function getMovie(id) {
    return mongoClient.db(dbName).collection('movies').findOne(
        {_id: new ObjectId(id)}
    );
}

export async function updateMovie({id, name, year}) {
    return mongoClient.db(dbName)
    .collection('movies')
    .updateOne(
        {_id: new ObjectId(id)},
        {
            $set: {name, year}
        }
    );
}


export async function deleteMovie(id) {
    return mongoClient.db(dbName)
    .collection('movies')
    .deleteOne({_id: new ObjectId(id)});
}
```

In this code snippet, we created a mongodb client and implemented functions to interact with the `movies` collection.

Update the `index.js` file to use the `db.js` functions. The updated `index.js` file should look like this:

```javascript
import express from 'express';
import bodyParser from 'body-parser';
import {createMovie, deleteMovie, getMovie, getMovies, updateMovie} from "./db.js";

const app = express();

app.use(bodyParser.json());

app.get('/', (req, res) => {
    res.send(
        `Welcome to Movies API. You can use this API to get and add movies at /movies.`
    );
});

app.get('/movies', async (req, res) => {
    const movies = await getMovies();
    res.json(movies);
});

app.get('/movies/:id', async (req, res) => {
    const id = req.params.id;
    const movie = await getMovie(id);
    res.json(movie);
})

app.post('/movies', async (req, res) => {
    const {name, year} = req.body;
    const id = await createMovie({name, year});
    res.json({id})
});


app.put('/movies/:id', async (req, res) => {
    const {name, year} = req.body;
    const id = req.params.id;
    await updateMovie({id, name, year});
    res.end();
});

app.delete('/movies/:id', async (req, res) => {
    const id = req.params.id;
    await deleteMovie(id);
    res.end();
});


const port = process.env.PORT || 8080;


app.listen(port, () => {
    console.log(`Server is running on port ${port}`);
});
```

## Run the app

You can run the app with the `hereya run` command:

```bash
hereya run node index.js
```

Now you can test the endpoints using a tool like `curl` or `Postman` or any `http` client.
For example, with `curl`:

- Get all movies:
```bash
curl http://localhost:8080/movies

# []
```

- Create a new movie:
```bash
curl -X POST -H "Content-Type: application/json" -d '{"name": "Inception", "year": 2010}' http://localhost:8080/movies

# {"id":"6662dfd7491f79189f82b0b6"} # The id will be different
```

- Get a movie by id:
```bash
curl http://localhost:8080/movies/<id> # Replace <id> with the id returned in the previous step

# {"_id":"6662dfd7491f79189f82b0b6","name":"Inception","year":2010} # The id will be different
``` 

- Update a movie:
```bash
curl -X PUT -H "Content-Type: application/json" -d '{"name": "Inception", "year": 2010}' http://localhost:8080/movies/<id> # Replace <id> with the id returned in the previous step
```

- Delete a movie:
```bash
curl -X DELETE http://localhost:8080/movies/<id> # Replace <id> with the id returned in the previous step
```

## Deploy the app to AWS

> **Warning**: In this step, you will create actual resources in your AWS account. These resources may incur costs.
> It should not cost more than a few dollars, but make sure to clean up the resources after you finish the tutorial 
> by reading the [clean up](#clean-up) section.

Now that you have developed your app, you can deploy it to AWS with a single command.

First of all, we need to bootstrap the AWS environment. This step is required only once. It creates the necessary 
resources for `hereya` to manage deployments in your AWS account. 
This step requires admin access to your AWS account. Make sure `AWS CLI` is installed and configured with your AWS account credentials.

* Run the following command to bootstrap your AWS environment. It invokes the [hereya/bootstrap-aws-stack](https://github.com/hereya/bootstrap-aws-stack.git) package, which uses the AWS CDK to set up an S3 bucket and a DynamoDB table for managing [openTofu](https://opentofu.org) package states. Both *AWS CDK* and *openTofu* are infrastructure-as-code frameworks used by Hereya packages.

```bash
hereya bootstrap aws
```

* Create a workspace named `staging`:
```bash
hereya workspace create staging
```

* Add a deployment package to your project. In this tutorial, we will deploy to `AWS AppRunner` using the package 
  [hereya/aws-apprunner-deploy](https://github.com/hereya/aws-apprunner-deploy):

```bash
hereya add hereya/aws-apprunner-deploy
```

A docker image will be built and deployed to AWS. Create a Dockerfile in the root of your project with the following content:
```Dockerfile
# Use the official Node.js image as the base image
FROM --platform=linux/amd64 node:20

# Set the working directory inside the container
WORKDIR /app

# Copy package.json and package-lock.json files to the working directory
COPY package*.json ./

# Install dependencies
RUN npm install --only=production

# Copy the rest of the application code to the working directory
COPY . .

# Expose the port the app runs on
EXPOSE 8080

# Command to run the app
CMD ["node", "index.js"]

```

* Deploy to AWS:
```bash
hereya deploy -w staging
```
This command deploys your application to AWS AppRunner, using AWS DocumentDB for the MongoDB-compatible database. Because this is the first time you’re running the deployment, it may take a few minutes to complete. Once the process finishes, you’ll receive a URL for your application. 

You can then use this AWS AppRunner-provided URL to test your app’s endpoints.

## Clean up

To avoid incurring costs, make sure to clean up the resources created in your AWS account.

* Undeploy the app:
```bash
hereya undeploy -w staging
```

* If you no longer want to use `hereya` in your AWS account, you can remove the resources created by `hereya bootstrap aws` by running:
```bash
hereya unbootstrap aws
```
