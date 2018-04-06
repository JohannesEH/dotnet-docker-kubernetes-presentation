# .NET (Core) Sparringsgruppe d. 9/4/2018 - Intro til Dotnet Core, Docker & Kubernetes

## Tools
* [VS Code](https://code.visualstudio.com/)
* [.Net Core](https://www.microsoft.com/net/learn/get-started/windows)
* [Docker for Windows](https://www.docker.com/docker-windows)
* [Minikube](https://github.com/kubernetes/minikube)
* [Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

## Agenda

1. Dotnet
    1. Console App
    2. Web App (React)
2. Docker
    1. Intro til docker (https://docs.docker.com/)
    2. Dockerize console app
    3. Dockerize web app
3. Kubernetes via. minikube
    1. Intro til Kubernetes (https://k8s.io/)
    2. Start a local minikube "cluster"
    3. Install web app as a service
    4. Install console app as a cronjob

## .NET Core

First we'll create a couple of apps using the `dotnet new` cli command.

### Console App

To create the console app we'll first call:
```
dotnet new console -o console_app
```

After app creation we'll go into the app directory, build the app and run it to see the output and finally we'll go back to the root directory.

```
cd console_app
dotnet build
dotnet run
cd..
```

### Web App

To create the web app we'll call:
```
dotnet new react -o web_app
```

## Docker

## Kubernetes

### Demotime! 🤞

