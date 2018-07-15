# Chapter 2. Running a PHP Script in Docker

Before we start building our application, it's helpful to get an idea of how Docker works by simply running a PHP script in Docker. Let's start by writing a classic Hello World! script in PHP:

{title="hello.php", linenos=off, lang=php}
~~~~~~~
<?php
echo "Hello World!";
~~~~~~~

You can run this script in a VM or on your laptop (assuming you have PHP installed) by running php hello.php from your terminal. You should see the output Hello World! just as we typed it above.

## Introduction to Docker Images

Docker runs every process within a container. All these containers run on a "host" machine, which will be your computer in this book. Once your application is ready for a production environment, the server (or several servers) will act as your Docker host.

Behind each running container is an "image." Docker images are created and maintained by software developers using Dockerfiles. In other words, if you want to create your own Docker image from scratch, you first make a new Dockerfile, then *build* an image from it, then *run* a container from the image.

{width=100%}
![Diagram 2: Dockerfiles, Images, Containers](images/diagram2.png)

Fortunately for us, we usually don't have to build our own images from scratch. Most popular software platforms (including PHP) have images that are officially provided by the developers of the software, or by groups of interested users. You will rarely need to build a completely new image, but later we'll see how to extend an existing image by writing your own Dockerfile.

Docker images can be built and stored on your host machine, or they can live in a remote "registry". In addition to maintaining the core Docker platform, the Docker team maintains a large registry called [Docker Hub](https://hub.docker.com/), where public images can be stored for free. Most open source software teams host official images on Docker Hub, including [PHP](https://hub.docker.com/_/php/).

## Getting a PHP Docker Image

In order to run our hello.php script in a container, we first need to "pull" an image for PHP. Let's start with the latest stable version of PHP. In your terminal, run:

{linenos=off, lang=sh}
~~~~~~~
$ docker pull php:latest
~~~~~~~

You should see something like this in your terminal:

{linenos=off, lang=sh}
~~~~~~~
latest: Pulling from library/php
94ed0c431eb5: Pull complete
8d5410041793: Downloading   41.6MB/82.75MB
6ab2a160fa31: Download complete
247a60c07518: Download complete
b09bee244a51: Download complete
b808e084c9be: Downloading  7.222MB/9.858MB
1d362d99e847: Waiting
~~~~~~~

This indicates that Docker is pulling the version of the PHP image tagged latest. When it's done, Docker will indicate that it has pulled the latest version by showing you a status like this:

{linenos=off, lang=sh}
~~~~~~~
Status: Downloaded newer image for php:latest
~~~~~~~

Note: The "latest" tag is a standard convention that most Docker images use for the most up-to-date version of their software. Beware using “latest” indiscriminately as it will automatically track the “latest” version even when there is a major version change.

Since our hello.php script is simple, it doesn't matter which version of PHP we use, but what if we need to run an older version of PHP for an existing project? This is where Docker truly shines as we just need to specify the PHP version when we run docker pull. For example, to download the PHP 5.6 image, we just run:

{linenos=off, lang=sh}
~~~~~~~
$ docker pull php:5.6
~~~~~~~

We can use this method to get newer, and unreleased versions of PHP as well (assuming there's a at least a Beta version on the [PHP registry's list](https://hub.docker.com/_/php/)). This makes running PHP in Docker very helpful for developers who need to work with multiple versions of PHP on a regular basis.

## Getting Code Into a Container

In order to understand the next step, you have to know a little bit about how Docker accesses files on the host system. A running container can't just reach down and read or write files to your computer - that container is essentially its own isolated system. Instead, we have to run containers with data from the host system mounted in a [volume](https://docs.docker.com/engine/admin/volumes/volumes/) or add code while building the image.

Later in this book we'll cover building Docker images from Dockerfiles and adding code that way, but for this simple Hello World! example, we're just going to mount the hello.php file into our PHP container using a volume.

## Running our Hello World script in Docker

Now that we've pulled a couple PHP images from Docker Hub and we know a little about how Docker uses volumes, we can run our script in a container from the terminal:

{linenos=off, lang=sh}
~~~~~~~
$ docker run --rm -v $(pwd):/app php:latest php /app/hello.php
~~~~~~~

If everything was done correctly, you should see Hello World! in your command line. You just ran your first PHP script in Docker!

### What's going on here?

Let's go over that Docker command and what it all meant:

* `docker run` - This is Docker's command to [run a command within a new container](https://docs.docker.com/engine/reference/run/). There are a lot of options that you can pass in, but we'll just start with the basics.

* `--rm` - This tells Docker to "remove" the container after the command is run. Alternatively, you can save the container to run it again, but if you don't eventually remove the container, it will just sit around taking up space, so it's best to set the remove flag in most cases.

* `-v $(pwd):/app` - This is telling Docker to [mount a volume](https://docs.docker.com/engine/tutorials/dockervolumes/). You typically pass in a path to a folder on your host system, a colon, and then a path to the folder in the container. Volumes are a powerful tool, but for this simple example we're just mounting the current directory (using $(pwd)) from our terminal into the /app directory in the new Docker container.

* `php:latest` - This indicates the image we're using for this container. You could specify another PHP image (eg: php:7.0 or php:5.6) to use a specific version of the language.

* `php /app/hello.php` - Finally, this is the command that Docker will run in the container. Since we mounted our code in the /app directory on the container, we have to run our script from that directory.

Now that you have a basic understanding of Docker and can run a PHP script within containers, it's time to build something a little more useful and interesting. This might also be a good time to take a break and read up on some of the core Docker concepts [in their documentation](https://docs.docker.com/). When you're ready, read on to start building a PHP web application in Docker.