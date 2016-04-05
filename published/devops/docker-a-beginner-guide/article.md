This tutorial covers the basics of Docker, a container that wraps code in a complete filesystem that contains code, runtime, system tools, and system libraries. Docker containers ensure that code will function the same, regardless of the environment -- in essence, a code incubator. 

This tool is a hot product on the market right now. 

Let me explain some terms used in Docker:

- **dockerfile** - a file that describes your steps in order to create a Docker image. It's like a recipe with all ingredients and steps necessary in making your dish.
- **image** - the snapshot of a virtual machine, but way more lightweight. Images are the building-blocks of the containers.
- **container** - the equivalent of creating a VM from a snapshot, but again, way more lightweight. Containers run the applications themselves.

These terms are the most used terms in the Docker world, and they are the main terms that I will use in this tutorial.

I will also cover the creation and packaging of a **Java** application that runs on [Tomcat Server](https://www.quora.com/What-is-the-function-of-Apache-Tomcat-and-how-do-I-use-it).

### 0. Kitematic Installation

Docker only runs above _Linux_ but you can use it with _Mac OS_ or _Windows OS_ using **Kitematic**. Kitematic creates **VirtualBox Machines** that let you run Docker on them. 

You can install Kitematic for Mac OS and Windows from here [https://kitematic.com/](https://kitematic.com/) and for Linux you can type this command:

```
curl -sSL https://get.docker.com/ | sh
```

_If you are using Mac or Windows, I suggest that you not use the Kitematic interface early on. It will be better if you start creating and manipulating everything from the command line. In this way you will learn more._

### 1. Creating a Dockerfile

You start by creating your Dockerfile that describes the steps in order to create the base image. Here is an example (comments on each line describe the code):

```
# All Dockerfiles need to start from a base Linux image.
# Docker has a hub (http://hub.docker.com) where you can see all Images submitted by users and 
# where you can submit your own images. This is like a github for Docker images.
# We will start from java:8-jre. This is an ubuntu machine with java 8 installed on it.
# The keyword here is FROM.
FROM java:8-jre

# The KEYWORD ENV it let us specify some Linux environment variables
# Here we will set CATALINA_HOME (the home of the tomcat server)
ENV CATALINA_HOME /usr/local/tomcat
ENV PATH $CATALINA_HOME/bin:$PATH


# RUN lets you run Linux commands inside the image.
# Here we will navigate into /usr/local then download tomcat from the server
# then decompress it and rename it to /tomcat.
# The final path to tomcat will be /usr/local/tomcat.
RUN 
     cd /usr/local/ && 
     wget http://mirrors.m247.ro/apache/tomcat/tomcat-8/v8.0.32/bin/apache-tomcat-8.0.32.tar.gz && 
     tar xzf apache-tomcat-8.0.32.tar.gz &&
     mv apache-tomcat-8.0.32/ tomcat/


# the EXPOSE command will tell your future container to expose 8080 port to the outside
# 8080 is the default port for tomcat.
EXPOSE 8080

# CMD sets the first command that will run when you create containers from the resulting image.
# Here, we will start tomcat once the container is called
CMD ["/usr/local/tomcat/catalina.sh", "run"]
```

### 2. Creating an Image

After you create this Dockerfile you need to create your image.
You need to navigate through your directory to where the Dockerfile was saved. Use this command line prompt: 

```
docker build -t tomcat .
```

A couple quick definitions:

**docker build** - _This command builds a new image from the source code at PATH_. In our case, the PATH is '.' (current folder). 
**t** _ this term refers to a Name of (or tag of) the image (format name:tag). In this case, just the name "tomcat".

After all steps have been executed, you can test your code's functionality by typing docker images and you will see your image there: tomcat.

### 3. Starting a container

You can now start a container from an image by running this:

```
docker run -p 8080:8080 --name=tomcat tomcat
```

Let's break down this simple, yet complex command.

**docker run** - _Runs a command in a new container_. 

**p** - an important term. Usually applications that use some ports in order to run, like tomcat 8080, mySQL 3306, Apache 80, and so on. In a docker container, those ports will be restricted to that container. This means that, if you start a container with mySQL (port 3306) and you'd like to access that mySQL instance from a Java application, you will not be able to access it. The mySQL port is forced to be open in the Docker container that you have started. 

But we solve this port issue by using p parameter.

**p parameter** - Establishes a connection between container's port(s) to the host_. By using **port1:port2** you make  **port1** match with **port2**. After that, you can use **port2** in order to access the service that is opened on **port1** inside the container. Thus, the container is now connected with outside environments.

### 4. Checking for containers

After you run a container you can check that it exists by typing the following sequence:

```
docker ps
```

This command will show you all running containers. If this commands returns nothing at all, then your container creation failed.

```
docker ps -a
```

The above command displays all containers (running or not).

### 5. Extras

After you started a container you can stop it by using **docker stop [name]**. You can restart the container by typing **docker start [name]**.

Another important command is **docker exec**, which lets you run a command inside your Docker **running** container. Note that this only works in runnign containers, not stopped ones.

For a complete list of commands you can go to [Docker references](https://docs.docker.com/engine/reference/commandline/cli/). I will recommend that you read up on many of those commands if you find Docker containers interesting.

I hope that this brief tutorial on Dockers has helped you! 

If you have any questions, please comment below or find me on Twitter, at [@grvmariobyn](https://twitter.com/grvmariobyn).

Thank you. 
Marius.
