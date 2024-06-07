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
- [Terraform](https://learn.hashicorp.com/tutorials/terraform/install-cli): **hereya packages** are reusable 
  infrastructure as code components and need the appropriate runtime to be installed. In this tutorial, we will need 
  terraform. ***You will not write any terraform code***.
  - You can install it by following the instructions on the [official website](https://learn.hashicorp.com/tutorials/terraform/install-cli).

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

