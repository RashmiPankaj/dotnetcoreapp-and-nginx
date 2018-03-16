# .Net Core App & Nginx with Docker compose

### Create a .Net Core App from Visual Studio Code

Create a new folder in file system. Open the folder in [Visual Studio Code](https://code.visualstudio.com/). Launch the command palette by ` ctrl + ~ `
Create two subfolders in the parent folder. One with `app` name and other with `nginx` name by using `mkdir app` and `mkdir nginx` on command terminal.

#### Steps:
* Change `app` as the current working directory `cd app`.
* On the command terminal type ``` dotnet new mvc ```. This step will create .Net Core MVC project in the current folder `app` and restores the nuget packages.
* You can try running this code by ``` dotnet run ```. This step starts the application. You can browse exposed URL in browser and see your app running.

### Create docker file

* Create a new file in the folder. Rename it as ``` Dockerfile ```. Place the content in the Dockerfile and replacing ```[your project name] ``` with your current app name.
 ``` dockerfile
      FROM microsoft/aspnetcore-build-nightly AS builder
      WORKDIR /source
      COPY *.csproj .
      RUN dotnet restore
      COPY . .
      RUN dotnet publish --output /app/ --configuration Release
      FROM microsoft/aspnetcore-nightly
      WORKDIR /app
      COPY --from=builder /app .
      ENTRYPOINT ["dotnet", "[]your project name].dll"]
  ```
* Before running docker commands please make sure you install Docker on your system. To download Docker for windows please refer:[https://docs.docker.com/docker-for-windows/install/](https://docs.docker.com/docker-for-windows/install/) 

* Launch the Terminal windows and build the docker image by typing the command ```docker build -t dotnetnginx . ```. You can specify any other tag name for image,  i have used dotnetnginx. This step will read the docker file and build the image as specified in docker file. 
* You can check the image created by using command ``` docker images ```
* Run the docker image by ``` docker run -d -p 8080:80 [image name] ```. ``` -d ``` option to run this container in background as daemon process.  ` -p ` option to specify the port of `host:container` format. Once the command is executed you can browse the container on browser by http://localhost:8080. However if you are testing this on Windows Containers, follow below steps to get ip of container:
    * Run ``` docker ps ``` to get the container id.
    * Run ``` docker inspect -f "{{ .NetworkSettings.Networks.nat.IPAddress }}" <container id> ``` to get the ip address of the container.
    * Browse to ip to test your application on windows container.
    * For this demo i am using linux container as Nginx multiarch is not available.

    ### Steps to create NGINX container
    * Change the current working directory to `nginx` folder by typing `cd nginx` on command terminal.
    * Create a `Dockerfile` and paste the below content
    ``` Dockerfile
        FROM nginx
        COPY nginx.conf /etc/nginx/nginx.conf
    ```
    * Create nginx configuration file with name  `nginx.conf` and paste the below content

    ```
    worker_processes 4;
 
    events { worker_connections 1024; }
 
    http {
        sendfile on;
 
        upstream app_servers {
        server dotnetnginx;
    }
 
    server {
        listen 80;
 
        location / {
            proxy_pass         http://app_servers;
            proxy_redirect     off;
            proxy_set_header   Host $host;
            proxy_set_header   X-Real-IP $remote_addr;
            proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header   X-Forwarded-Host $server_name;
                    }
            }
        }
```
* Change the current working directure to root folder and create `docker-compose.yml` file and put below contents

```
version: '2.1'
services:
  dotnetnginx:
    build: ./app
    image: app
    expose:
      - "5000"

  proxy:
    build: ./nginx
    image: nginx
    ports:
      - "80:80"
    links:
      - dotnetnginx
```
* Run `docker-compose build` to build the container images and `docker-compose up` to run the containers.
* As in the current config nginx is listening on port 80 you can browse to http://localhost to see your MVC site up.

Happy Coding !





