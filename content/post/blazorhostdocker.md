---
title: "Steps for Deploying a Blazor as Static Site with Docker and Nginx"
date: 2020-06-11T18:52:19+11:00
draft: false
---

![Blazor meets nginx](https://dev-to-uploads.s3.amazonaws.com/i/57a76kbj8isstq42nlga.png)

## Step 1 Publish the Blazor WebAssembly project
Publish the project from Visual Studio,this ensures that the projects is linked which removes all the unwanted dependencies from the output, reducing the size of the assemblies created.  
## Step 2 Create a dockerfile
The docker file is very straightforward, pull the nginx image and copy the published Blazor WebAssembly file from the WWWRoot folder to the html folder in nginx  
```
FROM nginx:alpine
EXPOSE 80
COPY bin/Release/netcoreapp3.1/publish/wwwroot /usr/share/nginx/html
```
## Step 3 Build the docker image  
use the below command  
```
docker build --tag blazorstatic . 
```

## Step 4 Run the docker container  
Run the docker container mapping the port exposed from the container to a port on the host  
```
 docker run -p 8080:80 blazorstatic   
```
## Step 5 Access the Blazor WebApp from browser  

Access the URL https://localhost:8080 which loads the WebApp
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/0etothdadxr8h1v5mi6t.png)

Source code for both ASPNet hosting and Static Website Hosting is available in Github: https://github.com/gopkumr/BlazorTourOfHeroes.git
Branch: Perf