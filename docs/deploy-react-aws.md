---
sidebar_position: 3
---

# Deploy a React app to AWS

In this tutorial, you will learn how to deploy a React app to [AWS Cloudfront](https://aws.amazon.com/cloudfront/)
using [hereya](https://github.com/hereya/hereya-cli).

In the following steps, you will:

- Create a new React app using [Vite](https://vitejs.dev/)
- Initialize hereya in the project
- Add a hereya package to deploy the app to AWS Cloudfront
- Deploy the app to AWS Cloudfront

## Prerequisites

Before you begin, make sure you have the following:

- An AWS account and [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
  installed on your machine and configured with your AWS account credentials. For this tutorial we recommend using a
  development AWS account and not the account you use for production. Make sure that you have admin access to the
  AWS account.
- [Node.js](https://nodejs.org/en/download/) version 18.0 or above.

## What you will build

You will build a simple React app using Vite and deploy it to AWS Cloudfront. Learning React is not the focus of
this tutorial, so we will use the default Vite template.

## Create a new React app

Create a new React app using Vite:

```bash
npx create-vite@latest hereya-tutorial-react --template react
```

## Run the app locally

Navigate to the project directory:

```bash
cd hereya-tutorial-react
```

Install the dependencies:

```bash
npm install
```

Start the development server:

```bash
npm run dev
```

Navigate to the URL displayed in the terminal to see the app running.

Now that you have a React app running locally, let's deploy it to AWS Cloudfront.

## Install Hereya

You can install **Hereya** by running the following command:

```bash
npm install -g hereya-cli
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
hereya init hereya-tutorial-react --workspace dev
```

This command will create a `hereya.yaml` file in the root of your project with the following content:

```yaml
name: hereya-tutorial-react
workspace: dev
```

This file is similar to the `package.json` file in a Node.js project. It contains the configuration for your project.

## Deploy the app to AWS Cloudfront

> **Warning**: In this step, you will create actual resources in your AWS account. These resources may incur costs.
> It should not cost more than a few dollars, but make sure to clean up the resources after you finish the tutorial
> by reading the [clean up](#clean-up) section.

Now that you have developed your app, you can deploy it to AWS with a single command.

First of all, we need to bootstrap the AWS environment. This step is required only once. It creates the necessary
resources for `hereya` to manage deployments in your AWS account.
This step requires admin access to your AWS account. Make sure `AWS CLI` is installed and configured with your AWS
account credentials.

* Run the following command to bootstrap the AWS environment

```bash
hereya bootstrap aws
```

* Create a workspace named `staging`

```bash
hereya workspace create staging
```

* Add a deployment package to your project

In this tutorial, we will use the package [hereya/cloudfront-deploy](https://github.com/hereya/cloudfront-deploy) to deploy the app to AWS Cloudfront.

```bash
hereya add hereya/cloudfront-deploy
```

* Build the app:

```bash
npm run build
```

* Deploy the app to AWS Cloudfront

  In this step, you will deploy the app with the command `hereya deploy`. Your code will be uploaded and deployed by
  the package you added in the previous step right in your AWS account. The previous command `npm run build` has
  created a `dist` folder in your project. This folder contains the built React app that will be deployed to AWS
  Cloudfront.
  By default, hereya does not upload the files that are in the `.gitignore` file, which is the case for the `dist`
  folder.
  To include the `dist` folder in the deployment, create a `.hereyaignore` file in the root of your project and copy
  the content of the `.gitignore` file into it. Then remove the line that ignores the `dist` folder. 
  Also, add the folder `.hereya` to both `.gitignore` and `.hereyaignore` files.
  Your `.hereyaignore` file should look like this:

```gitignore
# Logs
logs
*.log
npm-debug.log*
yarn-debug.log*
yarn-error.log*
pnpm-debug.log*
lerna-debug.log*

node_modules
*.local

# Editor directories and files
.vscode/*
!.vscode/extensions.json
.idea
.DS_Store
*.suo
*.ntvs*
*.njsproj
*.sln
*.sw?
.hereya
```

* Now, issue the following command to deploy the app to AWS Cloudfront:

```bash
hereya deploy -w staging
```

Assuming everything went well, you should see the URL of your app in the terminal. Open the URL in your browser to see
your app running on AWS Cloudfront.

Every time you make changes to your app, you can run the `hereya deploy` command to deploy the changes to AWS
Cloudfront.

## Clean up

After you finish the tutorial, you can clean up the resources you created in your AWS account to avoid incurring costs.

* Undeploy the app:

```bash
hereya undeploy -w staging
```

The resources created by `hereya bootstrap aws` does not incur costs when not in use. If you no longer want to
use `hereya` in your AWS account, you can remove the resources by running:

```bash
hereya unbootstrap aws
```
