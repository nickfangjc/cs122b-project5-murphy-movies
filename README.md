# CS 122B Project 5 Murphy movies example

1. This example application allows you to log in, see a movie by Eddie Murphy and leave comments.
2. In branch `Dockerized`, you will see how this application is turned into a docker image and deployed to AWS using Docker
3. In branch `Kubernetes-Compatible`, you will see how this application is modified to be deployed using Kubernetes pods

## This README.md file is used for the Dockerized branch

## Brief Explanation

- The `Deckerized` branch extends the `main` branch to enable this application to run in Docker containers.
- There are two main changes compared to the `main` branch: a new `Dockerfile` in the root folder and a modified `context.xml`.

### Dockerfile
- This file provides instructions to the Docker engine on how to package the application into a docker image;
- The docker image will be used to "boot up" docker containers, which can run the application.

### context.xml
- Since the Tomcat server will run inside Docker containers, the url of the MySQL server changes.
- They used to be running on the same machine (using `localhost`);
- Now the MySQL server can be found using `host.docker.internal:3306`


## Running this example

We will build a docker image on an AWS instance, push it to Docker Hub, and run a Docker container on the AWS instance to serve the website.

### Setup an AWS instance
- Follow the instructions from the README file in the `main` branch to set up the `murphymovies` database on an AWS instance;
- On the AWS instance, edit the `/etc/mysql/mysql.conf.d/mysqld.cnf` file and set `bind-address` to `0.0.0.0`.
- Restart MySQL by running `sudo service mysql restart`.
- Install Docker on the AWS instance by following the instructions on https://docs.docker.com/engine/install/ubuntu/. Use the instructions under "Install using the apt repository".
- Register a Docker Hub account.
- Log into our Docker Hub account by running `sudo docker login` on the AWS instance;

### Build a Docker image on the AWS instance
`git clone` this repository on the AWS instance, run `git checkout Dockerized` to switch to the `Dockerized` branch
Go to the root folder of this application. Run `sudo docker build . --build-arg MVN_PROFILE="login" --platform linux/amd64 -t <DockerHub-user-name>/cs122b-p5-murphy-login:v1 `
- `-t` means giving this image a tag 
- `--build-arg MVN_PROFILE` specifics the value of MVN_PROFILE in Dockerfile
- replace `<DockerHub-user-name>` with the username you just registered
- `cs122b-p5-murphy` is the DockerHub repo name, you may change it to whatever, be consistent in below steps if you do.
- `v1` is the tag name. The naming convention is `v1` ..`v2` for incremental version number.
- `--platform linux/amd64` ensures that the image built will be compatible with the CPU architecture of the AWS machines 

Check the created image by running `sudo docker images`. Note the tag and ID of the image `cs122b-p5-murphy-login`.

Repeat the same step for the `star` profile. Run `sudo docker build . --build-arg MVN_PROFILE="star" --platform linux/amd64 -t <DockerHub-user-name>/cs122b-p5-murphy-star:v1 `

### Push the image to Docker Hub
- Push the image to Docker Hub by running the following command: `sudo docker push <DockerHub-user-name>/cs122b-p5-murphy-<profilename>:v1`
- Log in our Docker Hub web page. We should be able to see the newly pushed image.

### Use the image to start a Docker container on the AWS instance

In the security group of the AWS instance, open the "8080" port to our local machine.

On the AWS instance, use the image to start a container by running `sudo docker run --add-host host.docker.internal:host-gateway -p 8080:8080 <image ID>` to start a docker container to run your application.  In the command, `-p 8080:8080` means we bind the port 8080 (first parameter) of the host instance to the port 8080 (second parameter) of the container.  When the host machine (the AWS instance) receives a request to the port 8080, the request will be relayed to the container's port 8080.

Use our browser to access the website via `<AWS_PUBLIC_IP>:8080/cs122b-project5-murphy-movies/login.html`.

Congratulations!  We have successfully run the "Murphy Movies" application in a Docker container on the AWS instance!

The `-d` flag means we run the container in the "detached mode" so that it runs in the background.

The following commands are useful to manage containers on the AWS instance:

- `sudo docker ps -a`: list all the containers
- `sudo docker logs <container ID>` : see the logs of a container
- `sudo docker stop <container ID>` : stop a running container
- `sudo docker rm <container ID>` : delete a stopped container

The following commands are useful to manage images on the AWS instance:

- `sudo docker images` : list the images

- `sudo docker rmi <image ID>` : delete an image

If we prefer NOT to use `sudo` to run docker commands, we can read this page [manager Docker as a non-root user](https://docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user).
