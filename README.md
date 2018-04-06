# .NET Group Presentation - Intro to .NET Core, Docker & Kubernetes

This presentation/guide is a quick introduction to dotnet core, docker and kubernetes. In this guide we'll create a couple of simple dotnet apps to run in docker containers and as jobs and services in a kuberntes cluster using minikube.

This guide can be found on GitHub: https://github.com/JohannesEH/dotnet-docker-kubernetes-presentation (or bit.ly/dotnet-docker-k8s)

## Tools

To follow this guide you have to install these tools:

* [VS Code](https://code.visualstudio.com/)
* [.Net Core SDK](https://www.microsoft.com/net/learn/get-started/windows)
* [Docker for Windows](https://www.docker.com/docker-windows)
* [Minikube](https://github.com/kubernetes/minikube)
* [Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

## Agenda

1. Dotnet
    1. Console App
    2. Web App (React)
2. Docker
    1. Dockerize console app
    2. Dockerize web app
3. Kubernetes 
    1. Start a local minikube "cluster"
    2. Install web app as a service
    3. Install console app as a cronjob

## .NET Core

First we'll create a couple of apps using the `dotnet new` cli command.

### Console App

To create the console app we'll first call:
```shell
dotnet new console -o console-app
```

After app creation we'll go into the app directory, build the app and run it to see the output.

```shell
cd console-app
dotnet build
dotnet run
```
The app should output `Hello World!`.  
Ok, lets head back out to the root dir and move on to the web app.

```shell
cd ..
```

### Web App

In this presentation we'll create a dotnet web app based on react (other templates are available for angular, vue and others). The steps are almost identical to creating the console app.

To create the web app we'll call:

```shell
dotnet new react -o web-app
```

After the web app is created lets build and test it to see if it runs.

```shell
cd web-app
npm install
dotnet build
dotnet run
```

Open the web app in your browser using the default address of http://localhost:5000/ and verify that the sample app is running correctly. After testing the app go back to the console and close the server by pressing Ctrl+C. Then head back out to the root dir.

```shell
cd ..
```

That's it for the dotnet core part. Not much, I know, but enough for us to have something to build and run in docker and kubernetes.

## Docker (https://docs.docker.com/engine/docker-overview/ & https://hub.docker.com/)

Docker basics DEMO 🤞🔥

### Setting up a Dockerfile for the console app



## Kubernetes (https://kubernetes.io/docs/concepts/)

K8s basics DEMO 🤞🤞🔥🔥

### Setting up & running minikube
