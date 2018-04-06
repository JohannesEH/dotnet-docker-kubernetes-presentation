# .NET Group Presentation - Intro to .NET Core, Docker & Kubernetes

This presentation/guide is a quick introduction to dotnet core, docker and kubernetes. In this guide we'll create a couple of simple dotnet apps to run in docker containers and as jobs and services in a kuberntes cluster using minikube.

This guide can be found on GitHub: https://github.com/JohannesEH/dotnet-docker-kubernetes-presentation (or bit.ly/dotnet-docker-k8s)

## Tools

To follow this guide you have to install these tools:

* [VS Code](https://code.visualstudio.com/) (optional but recommended)
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

## Docker Intro (https://docs.docker.com/engine/docker-overview/ & https://hub.docker.com/)

Docker concepts and CLI (`pull`, `push`, `prune`, `run`, `build`) DEMO 🤞🔥

Example docker run command: `docker run --rm -it ubuntu /bin/bash`

### Setting up a Dockerfile for the console app

First well dockerize the console app to do this well need to create an empty file called "Dockerfile" in the console-app dir. Copy-paste the following code into the file:

```dockerfile
# Restore dependencies and build the app
FROM microsoft/dotnet:2.0-sdk AS build
WORKDIR /src
COPY *.csproj .
RUN dotnet restore
COPY . .
RUN dotnet build -c Release

# Publish the app based on the build step
FROM build as publish
RUN dotnet publish -c Release -o /app

# Run tests
# FROM publish AS test
# RUN dotnet test <some test project>

# Package the published code with the dotnet runtime image
FROM microsoft/dotnet:2.0-runtime AS final
WORKDIR /app
COPY --from=publish /app .
ENTRYPOINT ["dotnet", "console-app.dll"]
```

After creating the dockerfile build using:

```shell
docker build console-app -t console-app
```

If successful this command will output a docker image tagged console-app:latest to your local docker repository. To run the image use the following command:

```shell
docker run --rm -it console-app
```

This command should output `Hello World!` just as the `dotnet run` command did earlier.

### Setting up a Dockerfile for the web app

Next we'll setup a Dockerfile for the web app. Again create an empty Dockerfile in the web-app directory and fill in the following code:

```dockerfile
# Restore dependencies and build the app
FROM microsoft/aspnetcore-build:2.0 AS build
WORKDIR /src
COPY *.csproj .
COPY package.json .
RUN dotnet restore
RUN npm install
COPY . .
RUN dotnet build -c Release

# Publish the app based on the build step
FROM build as publish
RUN dotnet publish -c Release -o /app

# Run tests
# FROM publish AS test
# RUN dotnet test <some test project>

# Package the published code with the dotnet runtime image
FROM microsoft/aspnetcore:2.0 AS final
WORKDIR /app
COPY --from=publish /app .
EXPOSE 80
ENTRYPOINT ["dotnet", "web-app.dll"]
```

After creating the dockerfile build using:

```shell
docker build web-app -t web-app
```

If successful this command will output a docker image tagged web-app:latest to your local docker repository. To run the image use the following command:

```shell
docker run --rm -it -p 80:80 web-app
```

This will start an interactive session just like running `dotnet run` from the console. Now you can visit the app on the exposed and mapped port 80. If you want to run the web app in the background, and keep the docker container around for later use, you can run the following command:

```shell
docker run --name web-app -d -p 80:80 web-app
```

So now we have two working docker images lets setup kubernetes and configure the apps to run in the cluster.

## Kubernetes

### Setting up & running minikube

First of all we need to get the cluster up and running. We need minikube to run in hyper-v, which is a little tricky. So first you should follow [this guide](https://medium.com/@JockDaRock/minikube-on-windows-10-with-hyper-v-6ef0f4dc158c) to configure hyper-v correctly, and then finally you should start start minikube with this command from an elevated prompt:

```shell
minikube start --vm-driver hyperv --hyperv-virtual-switch "Primary Virtual Switch"
```

### Kubernetes Intro (https://kubernetes.io/docs/concepts/)

K8s concepts, CLI and dashboard DEMO 🤞🤞🔥🔥

### Setting up the console app as a cron job

Let's pretend our console app is job we need to run every minute. Create a new file called console-app-job.yaml and copy/paste this content into the file:

```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: console-app-job
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: console-app-container
            image: console-app
          restartPolicy: OnFailure
```

### Setting up the web-app as a external service