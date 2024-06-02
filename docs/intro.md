---
sidebar_position: 1
---

# Getting Started

Let's discover **Hereya in less than 15 minutes** by building a simple Node.js app with a postgres database.
In this tutorial, you will learn how to:

- Initialize Hereya in a project
- Use Hereya Cli to install and connect to a postgres database without managing environment variables


## Prerequisites

To follow this tutorial, you need to have the following installed on your machine:
- [Node.js](https://nodejs.org/en/download/) version 18.0 or above.
- [docker](https://docs.docker.com/get-docker/): *hereya* will use docker to run a postgres database.
  - You can install it by following the instructions on the [official website](https://docs.docker.com/get-docker/).
- [terraform](https://learn.hashicorp.com/tutorials/terraform/install-cli): **hereya packages** are reusable 
  infrastructure as code components and need the appropriate runtime to be installed. In this tutorial, we will need 
  terraform. ***You will not write any terraform code***.
  - You can install it by following the instructions on the [official website](https://learn.hashicorp.com/tutorials/terraform/install-cli).


## What you will build

You will build a simple Node.js commandline app that connects to a postgres database. The app will display a list of 
messages stored in the database or store a message given as an argument to the command like this:

```bash
node index.js # Display all messages
```

```bash
node index.js my message # Store my message in the database
```

## Install Hereya

You can install **Hereya** by running the following command:

```bash
npm install -g hereya-cli 
```

## Set up the Node.js project

Create a new directory for your project and navigate to it:

```bash
mkdir hereya-tutorial # You can name it whatever you want
cd hereya-tutorial
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
  "name": "hereya-tutorial",
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
hereya init hereya-tutorial -w dev # Replace hereya-tutorial with the name of your project
```

This command will create a `hereya.yaml` file in the root of your project with the following content:

```yaml
project: hereya-tutorial
workspace: dev
```

Add postgres package to your project. A package in hereya is a GitHub repository. Let's add the package 
[hereya/local-postgres](https://github.com/hereya/local-postgres) to your project:

```bash
hereya add hereya/local-postgres
```
This command deploys a postgres database in a docker container and exposes it on port `5432`. It will also add 
the necessary environment variables to the file `.hereya/env.dev.yaml`. Make sure to the add the directory `.hereya` 
to your `.gitignore` file when using git.

If you open the `hereya.yaml` file, you will see that the postgres package has been added to your project:

```yaml
project: hereya-tutorial
workspace: dev
packages:
  hereya/local-postgres:
    version: ""
```
`hereya.yaml` should be checked into your version control system. It contains the configuration of your project.

To customize the port, use the command:

```bash
hereya add hereya/local-postgres -p 'port=5433' # Replace 5433 with the port you want
```

`-p` is a flag that allows you to pass parameters to the package. In this case, we are changing the port to `5433`. 
If you use this command, *Hereya* will create the file `hereyavars/local-postgres.yaml` with the following content:

```yaml
port: 5433
```
Make sure to check this file into your version control system.

This package exports the following environment variables:
`POSTGRES_URL`, `POSTGRES_ROOT_URL`, `DBNAME`.

You can print the environment variables exported by all packages by running:

```bash
hereya env
```

You don't need to manage the environment variables yourself. **Hereya** will manage them for you.

## Write the Node.js app

To connect to the postgres database, you need a postgres client. You can use any client you want, but in this 
tutorial, we will use the `pg` package. Install it by running:

```bash
npm install pg
# optionally, you can install @types/pg
npm install @types/pg
```

Create a new file named `index.js` in the root of your project and add the following code:

```javascript
import pg from 'pg';

const client = new pg.Client({
    connectionString: process.env.POSTGRES_URL,
});
// Connect to the postgres database
await client.connect();

// Create the messages table if it doesn't exist
await client.query(`
    CREATE TABLE IF NOT EXISTS messages (
        id SERIAL PRIMARY KEY,
        message TEXT
    );
`)

// Get the message from the command line arguments
const message = process.argv.slice(2).join(' ');

// Save the message in the database if it exists
if (message) {
    await client.query(`
        INSERT INTO messages (message) VALUES ($1);
    `, [message]);
    console.log('Message saved!');
} else {
    // Display all messages
    const res = await client.query(`
        SELECT * FROM messages;
    `);
    console.table(res.rows);
}

// Close the connection
await client.end();
```

## Run the app with Hereya

**Hereya** manages the environment variables for you. You can access them in your code like any other environment variable.
You just need to run your app with the `hereya` command:

```bash
hereya run node index.js

# Running command "node index.js" ...
# ┌─────────┐
# │ (index) │
# ├─────────┤
# └─────────┘

```

You can also pass arguments to your app like this:

```bash
hereya run node index.js my message

# Running command "node index.js my message" ...
# Message saved!
```

Then, you can check the messages stored in the database by running the app without arguments:

```bash
hereya run node index.js

# Running command "node index.js" ...
# ┌────┬────────────────┐
# │ id │    message     │
# ├────┼────────────────┤
# │ 1  │ my message     │
# └────┴────────────────┘
```
