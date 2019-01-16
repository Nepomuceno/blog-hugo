---
title: Serverless, Serverless² and Serverless³
date: 2018-07-31 01:01:01
draft: true
---

There are many ways to use serverless in Azure. I will try them to combine all of then in one single request, Of course the pratical implications this are not so abundant but just seeing that this is possible shows how far we are in this new jorrney

<!--Should be one image here-->

<!--more-->

## Azure Fuctions

The first step that we will do will be create one azure function. For that you will need to have the function api running in your machine. Since with Azure functions ytou can run it locally to to test 😄.

First step if you dont have the functions 2.0 tools installed you would need to install it there is a set of inctructions on how to do it [here](https://docs.microsoft.com/en-us/azure/azure-functions/functions-run-local). But if you are on Windows the easy way to do it is to install it is to just use npm

```sh
npm install -g azure-functions-core-tools@core
```

Now now navigate to the folder that we want to create the function app

them type the following command.

```sh
func init --docker
```

> We are using the --docker to create the docker file that we are going to use in the future in this article.

Now lets create our first function

```sh
func new -n hello -t httptrigger
```

Simple as that you have your first serverless function created now to run it just type

```sh
func start --build
```

Now that we have our first function you can test it going to the browser on the address that was shown on your terminal in my case I did a get on [http://localhost:7071/api/amazing?name=Gabriel](http://localhost:7071/api/amazing?name=Gabriel) remember to include the name parameter.

We could go on about how to create or change functions but there are many good blog posts about that and the [documentation](https://docs.microsoft.com/en-us/azure/azure-functions/) it is really good on that too.

Now time to publish this function.
Use this command to create the app and seup a local git repo to push your code.

```sh
az group create --name function-demo -l northeurope
az storage account create -n funcstoragegabriel -l northeurope -g function-demo --sku Standard_LRS
az functionapp create -n demo-func -g function-demo -c northeurope -s funcstoragegabriel -l
az functionapp config appsettings set -n demo-func -g function-demo --settings FUNCTIONS_EXTENSION_VERSION=beta
```

Now you need to recover the credentials to deploy to this git repo

```sh
az functionapp deployment list-publishing-profiles --resource-group function-demo --name demo-func3 --query '{username:[0].userName, password:[0].userPWD}'
```

Them lets just create out git repo and push that to our function app.

```sh
git init
git add -A
git commit -m "Initial function deploy"
git remote add origin https://demo-func.scm.azurewebsites.net/demo-func.git
git push --set-upstream origin master
```

This should deploy your function app remember to use the username and password that you got in the previous step.



