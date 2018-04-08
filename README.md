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
#Configure build envrionment
FROM microsoft/dotnet:2.0-sdk AS build-env
WORKDIR /src

ENV NODE_VERSION 8.11.1
ENV NODE_DOWNLOAD_SHA 0e20787e2eda4cc31336d8327556ebc7417e8ee0a6ba0de96a09b0ec2b841f60

RUN curl -SL "http://unencrypted.nodejs.org/download/release/v${NODE_VERSION}/node-v${NODE_VERSION}-linux-x64.tar.gz" --output nodejs.tar.gz \
    && echo "$NODE_DOWNLOAD_SHA nodejs.tar.gz" | sha256sum -c - \
    && tar -xzf "nodejs.tar.gz" -C /usr/local --strip-components=1 \
    && rm nodejs.tar.gz \
    && ln -s /usr/local/bin/node /usr/local/bin/nodejs \
    && echo Node Version && node -v \
    && echo NPM Version && npm -v

# Restore dependencies and build the app
FROM build-env AS build
RUN npm -v
COPY *.csproj .
RUN dotnet restore
COPY package.json .
RUN npm install
COPY . .
RUN dotnet build -c Release

FROM build AS publish
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
docker run --rm -it -p 8080:80 web-app
```

This will start an interactive session just like running `dotnet run` from the console. Now you can visit the app on the exposed and mapped port 80. If you want to run the web app in the background (detached mode), and keep the docker container around for later use, you can run the following command:

```shell
docker run --name web-app -d -p 8080:80 web-app
```

So now we have two working docker images let us publish them to dockerhub so we can run them in our kubernetes cluster.

## Dockerhub

This step is fairly easy. First you have to go to hub.docker.com and create an account and 2 repositories called "console-app" and "web-app". My username on dockerhub is "johanneseh" so I'll use that name in the following steps.

First you need to login the docker client to dockerhub by calling:

```shell
docker login
```

After you've logged in we need to tag our local images so they know what repo they should go to when you push. So lets tag our app images:

```shell
docker tag console-app johanneseh/console-app
docker tag web-app johanneseh/web-app
```

Now we're ready to push our images to dockerhub. The default behavior of the docker client is to push images to dockerhub if you don't choose another docker registry. So to push our images to dockerhub we simply use:

```shell
docker push johanneseh/console-app
docker push johanneseh/web-app
```

Good. Now our images are online and ready to be used in our kubernetes cluster so lets go ahead and set that up.

## Kubernetes

### Setting up & running minikube

First of all we need to get the cluster up and running. We need minikube to run in hyper-v, which is a little tricky. So first you should follow [this guide](https://medium.com/@JockDaRock/minikube-on-windows-10-with-hyper-v-6ef0f4dc158c) to configure hyper-v correctly, and then finally you should start start minikube with this command from an elevated prompt:

```shell
minikube start --vm-driver hyperv --hyperv-virtual-switch "Primary Virtual Switch"
```

### Kubernetes Intro (https://kubernetes.io/docs/concepts/)

K8s concepts, CLI and dashboard DEMO 🤞🤞🔥🔥

### Setting up the console app as a cron job

Let's pretend our console app is job we need to run every minute. To do this we create a cronjob with the following command:

```shell
kubectl run console-app-job --schedule="*/1 * * * *" --restart=OnFailure --image=johanneseh/console-app
```

Check that the job is running using the `minkube dashboard` command.

### Setting up the web-app and expose it

Now let's configure our web app to run as a service in our cluster with the following command:

```shell
kubectl run web-app --image=johanneseh/web-app --port=80 --replicas=3
```

Check that the deployment is running in the dashboard.

Finally let's expose the service using a kubernetes LoadBalancer service.

```shell
kubectl expose deployment web-app --type=LoadBalancer --name=web-app-service
```

This should create the LoadBalancer and let us visit the running web app. In a normal kubernetes cluster we would get an external ip to go to or point our dns at, but in minikube this is a bit different. Fortunately minikube has a command to get the address for the service:

```shell
minikube service web-app-service
```

This should open up your browser and let you see the running web page.

## We're done!
That's it... We've created to apps, dockerized them and made the run in a kubernetes cluster.