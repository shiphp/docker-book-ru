---
title: Copy of Shared: Building Your First PHP Application with Docker
layout: post
author: lex61rus
permalink: /copy-of-shared:-building-your-first-php-application-with-docker/
source-id: 13WqHLwiGkiN4Rn8MTHoqbrba08fnWZdxc-CQGm2oBAs
published: true
---
Building Your First PHP Application 

with Docker

A Step-by-Step Guide for PHP Developers Learning Docker

© 2017, Karl Hughes

Originally published October, 2017

This edition published January, 2018

All rights reserved. No part of this book may be reproduced in any form or by any electronic or mechanical means, without permission in writing from the copyright owner. Contact Karl Hughes ([karl@portablecto.com](mailto:karl@portablecto.com)) with any questions or comments on this work.

# Index

[[TOC]]

# Chapter 1. Introduction to Docker

Since its first release in 2013, Docker has become the container engine of choice for many developers, and it may be replacing a virtual machine near you soon. This book offers a step-by-step guide that will walk you through the process of building a real PHP web application using Docker while explaining the basics of the platform along the way.

The application we build in this book will do many of the key things that PHP developers need to do on a daily basis, including:

* Installing dependencies using [Composer](https://getcomposer.org/).

* Using a web framework ([SlimPHP](https://www.slimframework.com/)) for routing.

* Getting data from a third-party API.

* Saving data to a database.

* Securely setting environmental variables.

This book was written for seasoned PHP developers who want to learn to develop web applications with Docker, learn how Docker works, and like learning by building real, working applications. Once you've completed this book, you'll be ready for more advanced Docker topics, and I've included some resources to help you at the end.

All the code I include in this book is open source and [freely available on Github](https://github.com/shiphp/weather-app). If you see any problems or you want to suggest an improvement, feel free to make a pull request. Finally, check [www.shiphp.com](https://www.shiphp.com/) often for blog posts, training, and new books on both Docker and PHP.

## What is Docker?

Docker is a platform for managing and running containers. Containers are like [virtual machines](https://en.wikipedia.org/wiki/Virtual_machine), but they don't actually emulate the whole operating system. Instead, all of the containers you run share the same underlying kernel with the host machine, which means that they're much lighter-weight than virtual machines. Because of this, containers are very efficient, and most real-world applications run many containers at once. Docker helps you link these containers together using [networks](https://docs.docker.com/engine/userguide/networking/) of containers, and helps you define your containers using [Docker Compose](https://docs.docker.com/compose/) configuration files. While this book doesn't cover these topics in detail, it will get you started, and more in-depth material is available at [www.shiphp.com](https://www.shiphp.com/).

*Diagram 1: Docker Container vs. Virtual Machine*

## Why Docker?

PHP developers who are familiar with virtual machines or have used [Vagrant](https://www.sitepoint.com/5-easy-ways-getting-started-php-vagrant/) will most easily understand containers by comparing them with virtual machines. The important difference between containers and virtual machines in practice is that you will rarely use *one single* Docker container for your whole application. Instead, you will run your PHP code in one container, your web server in another container, and your database in third container. Some applications I've worked on use dozens of linked containers to operate!

If you haven't used a virtual machine before, Docker containers can still help you improve your development workflow. Developers who work on teams that run PHP and Apache natively on their operating system often run into the "works on my machine" problem. One team member updates the web application and everything seems to work fine, but then when the application is deployed to the server or run by another team member, it mysteriously breaks. Docker solves these problems by allowing developers to set up and share replicable development environments, and then *test and deploy applications in exactly the same state* using containers.

At this point you may want to read a little more about Docker. While this book will walk you through the process of building and running a PHP web application on Docker, it does not attempt to cover the inner-workings of Docker, containers, or operating system virtualization. You can read more on Docker and many of these related topics on [Docker's website](https://www.docker.com/what-docker).

## What can I expect from this book?

While the computer science behind containers is an interesting topic, this book takes a pragmatic approach. Many PHP developers (including myself) are hands-on learners, so I wrote this book to *help people who learn by doing*. Along the way, I've included some diagrams for visual learners, code samples (as well as [a complete code repo on Github](https://github.com/shiphp/weather-app)), and links to free and paid resources to learn more about each of the topics covered.

Reading the [Docker documentation](https://docs.docker.com/) is a good idea, but I find that starting out with a working application gives me better context for the docs, and deepens my understanding of what I read there. If you're a PHP developer, this book will take you on a journey to build your first application and give you that foundation you need to truly understand how Docker works and how you could use it to build better software.

The application we build in this book is simple [REST](https://stackoverflow.com/questions/671118/what-exactly-is-restful-programming) API, but it covers most of the typical problems that PHP developers will need to solve when using Docker. Throughout the course of building this application, I'll show you to install packages using [Composer](https://getcomposer.org/), get data from a third party API, save data to a database, and use environmental variables with Docker. Rather than explain everything up front, most details will be uncovered as you build the application. 

The best way to read this book would be to sit down with it and your computer for an afternoon and work through it. Hopefully by the end, you'll be ready to start using Docker for your next PHP project.

There are a few prerequisites to this book. You should have an understanding of [what PHP is](http://php.net/manual/en/intro-whatis.php) and the basics of its syntax, you should know how to open your computer's terminal and run PHP scripts from it, and you should have version 17 of the community edition of [Docker (available for Mac, Windows, or Linux)](https://www.docker.com/community-edition) installed on your computer. Assuming you've got those things, let's get started!

# Chapter 2. Running a PHP Script in Docker

Before we start building our application, it's helpful to get an idea of how Docker works by simply running a PHP script in Docker. Let's start by writing a classic Hello World! script in PHP:

*hello.php*

<table>
  <tr>
    <td><?phpecho "Hello World!";</td>
  </tr>
</table>


You can run this script in a VM or on your laptop (assuming you have PHP installed) by running php hello.php from your terminal. You should see the output Hello World! just as we typed it above.

## Introduction to Docker Images

Docker runs every process within a container. All these containers run on a "host" machine, which will be your computer in this book. Once your application is ready for a production environment, the server (or several servers) will act as your Docker host.

Behind each running container is an "image." Docker images are created and maintained by software developers using Dockerfiles. In other words, if you want to create your own Docker image from scratch, you first make a new Dockerfile, then *build* an image from it, then *run* a container from the image.

![image alt text]({{ site.url }}/public/WrhST5eBKSgLJSvgr54cg_img_0.png)

*Diagram 2: Dockerfiles, Images, Containers*

Fortunately for us, we usually don't have to build our own images from scratch. Most popular software platforms (including PHP) have images that are officially provided by the developers of the software, or by groups of interested users. You will rarely need to build a completely new image, but later we'll see how to extend an existing image by writing your own Dockerfile.

Docker images can be built and stored on your host machine, or they can live in a remote "registry". In addition to maintaining the core Docker platform, the Docker team maintains a large registry called [Docker Hub](https://hub.docker.com/), where public images can be stored for free. Most open source software teams host official images on Docker Hub, including [PHP](https://hub.docker.com/_/php/).

## Getting a PHP Docker Image

In order to run our hello.php script in a container, we first need to "pull" an image for PHP. Let's start with the latest stable version of PHP. In your terminal, run:

<table>
  <tr>
    <td>$ docker pull php:latest</td>
  </tr>
</table>


You should see something like this in your terminal:

<table>
  <tr>
    <td>latest: Pulling from library/php94ed0c431eb5: Pull complete 8d5410041793: Downloading   41.6MB/82.75MB6ab2a160fa31: Download complete 247a60c07518: Download complete b09bee244a51: Download complete b808e084c9be: Downloading  7.222MB/9.858MB1d362d99e847: Waiting </td>
  </tr>
</table>


This indicates that Docker is pulling the version of the PHP image tagged latest. When it's done, Docker will indicate that it has pulled the latest version by showing you a status like this:

<table>
  <tr>
    <td>Status: Downloaded newer image for php:latest</td>
  </tr>
</table>


Note: The "latest" tag is a standard convention that most Docker images use for the most up-to-date version of their software. Beware using “latest” indiscriminately as it will automatically track the “latest” version even when there is a major version change.

Since our hello.php script is simple, it doesn't matter which version of PHP we use, but what if we need to run an older version of PHP for an existing project? This is where Docker truly shines as we just need to specify the PHP version when we run docker pull. For example, to download the PHP 5.6 image, we just run:

<table>
  <tr>
    <td>$ docker pull php:5.6</td>
  </tr>
</table>


We can use this method to get newer, and unreleased versions of PHP as well (assuming there's a at least a Beta version on the [PHP registry's list](https://hub.docker.com/_/php/)). This makes running PHP in Docker very helpful for developers who need to work with multiple versions of PHP on a regular basis.

## Getting Code Into a Container

In order to understand the next step, you have to know a little bit about how Docker accesses files on the host system. A running container can't just reach down and read or write files to your computer - that container is essentially its own isolated system. Instead, we have to run containers with data from the host system mounted in a [volume](https://docs.docker.com/engine/admin/volumes/volumes/) or add code while building the image.

Later in this book we'll cover building Docker images from Dockerfiles and adding code that way, but for this simple Hello World! example, we're just going to mount the hello.php file into our PHP container using a volume.

## Running our Hello World script in Docker

Now that we've pulled a couple PHP images from Docker Hub and we know a little about how Docker uses volumes, we can run our script in a container from the terminal:

<table>
  <tr>
    <td>$ docker run --rm -v $(pwd):/app php:latest php /app/hello.php</td>
  </tr>
</table>


If everything was done correctly, you should see Hello World! in your command line. You just ran your first PHP script in Docker!

### What's going on here?

Let's go over that Docker command and what it all meant:

* docker run - This is Docker's command to [run a command within a new container](https://docs.docker.com/engine/reference/run/). There are a lot of options that you can pass in, but we'll just start with the basics.

* --rm - This tells Docker to "remove" the container after the command is run. Alternatively, you can save the container to run it again, but if you don't eventually remove the container, it will just sit around taking up space, so it's best to set the remove flag in most cases.

* -v $(pwd):/app - This is telling Docker to [mount a volume](https://docs.docker.com/engine/tutorials/dockervolumes/). You typically pass in a path to a folder on your host system, a colon, and then a path to the folder in the container. Volumes are a powerful tool, but for this simple example we're just mounting the current directory (using $(pwd)) from our terminal into the /app directory in the new Docker container.

* php:latest - This indicates the image we're using for this container. You could specify another PHP image (eg: php:7.0 or php:5.6) to use a specific version of the language.

* php /app/hello.php - Finally, this is the command that Docker will run in the container. Since we mounted our code in the /app directory on the container, we have to run our script from that directory.

Now that you have a basic understanding of Docker and can run a PHP script within containers, it's time to build something a little more useful and interesting. This might also be a good time to take a break and read up on some of the core Docker concepts [in their documentation](https://docs.docker.com/). When you're ready, read on to start building a PHP web application in Docker.

# Chapter 3. Creating a SlimPHP Application

In the rest of this book, we'll focus on our web application: a weather checker. We'll use the [MetaWeather API](https://www.metaweather.com/api/) to search for a location and get the weather there. When we find results, we'll save them to the database so that future requests will get the cached version of the results.

I chose the MetaWeather API because it is free and does not require an API key. There are other weather APIs you can use for free, but most require you to sign up and get a key.

Our application will provide a very simple REST API that we'll use to interact with the service. We'll create two endpoints:

* GET /locations/:location_id - Gets the weather from the database (if it exists) or MetaWeather if a cached version does not.

* DELETE /locations/:location_id - Deletes the cached version of the MetaWeather results from the database. The next call to GET weather for this location will come from the MetaWeather API.

Before we get to the application logic, let's start simple and get a fresh installation of [SlimPHP](https://www.slimframework.com/) installed and running using Docker.

## Installing Slim Framework

Slim is a minimal PHP web development framework. Unlike more comprehensive frameworks like [Laravel](https://laravel.com/) or [CakePHP](https://cakephp.org/), [Slim](https://www.slimframework.com/) does not include much more than a router, middleware stack, and dependency injector. If you're familiar with [Node's Express framework](https://expressjs.com/), Slim will feel familiar to you, but even if you're not, it's such a small framework that it's pretty easy to grasp. That's why I've chosen it for this book: I don't want the framework to throw you off, but I didn't want to build everything from scratch either as the focus here is on Docker, not our PHP application.

Setting up Slim with Docker actually takes fewer commands than doing it with a locally installed version of PHP. We will use [Composer](https://getcomposer.org/), but thanks to Docker we don't even need to install it on our host machine.

From your command line, create a new directory for your Dockerized PHP application:

<table>
  <tr>
    <td>$ mkdir weather-app</td>
  </tr>
</table>


And navigate to it:

<table>
  <tr>
    <td>$ cd weather-app</td>
  </tr>
</table>


Now install Slim using [the official Composer Docker image](https://hub.docker.com/_/composer/):

<table>
  <tr>
    <td>$ docker run --rm -v $(pwd):/app composer:latest require slim/slim "^3.0"</td>
  </tr>
</table>


If you view the weather-app/ directory, you'll see two files (composer.json and composer.lock) and one directory (vendor/):

<table>
  <tr>
    <td>$ lscomposer.json   composer.lock   vendor</td>
  </tr>
</table>


### What's going on here?

Our docker run command was similar to the one we ran in Chapter 2, but we used a different image. Instead of the PHP image we used to run the hello.php script, we used a Composer image. Let's take a look at what changed.

* composer:latest - This indicates the image we're using for this container. You can specify a specific version of Composer if you need to; just check the list of image tags supported on [Docker Hub](https://hub.docker.com/_/composer/).

* require slim/slim "^3.0" - This is the command that actually installs Slim PHP into the container's working directory. Because that working directory is a volume from our host directory, the files composer installs are now present on your host machine as well as in the container.

At this point, your application doesn't actually do anything, but Slim has been installed and we now have some idea how composer works with PHP in Docker.

## Stubbing Out Endpoints

In order to give our application life, we need to have a couple endpoints. As mentioned above, Slim is little more than a router, but that actually takes care of most of our application. Let's start out by creating an index.php file with a couple stubbed out endpoints that we can build upon throughout the rest of this tutorial.

*index.php*

<table>
  <tr>
    <td><?php require 'vendor/autoload.php';// Instantiate the App object$app = new \Slim\App();// Declare routes$app->get('/locations/{id}', function ($request, $response, $args) {    return $response->withStatus(200)->write("Location {$args['id']} retrieved.");});$app->delete('/locations/{id}', function ($request, $response, $args) {    return $response->withStatus(200)->write("Location {$args['id']} deleted.");});// Run the application$app->run();</td>
  </tr>
</table>


### What's going on here?

This single file creates two API endpoints that simply return text for now. In case you're not familiar with Composer or Slim's routing system, here's what's going on:

* require 'vendor/autoload.php'; - Any PHP project that uses Composer must include Composer's autoload.php file somewhere. Since our new application is just one file, we've added it as the first thing.

* $app = new \Slim\App(); - This creates a new Slim PHP application instance. Slim can handle middleware and help with dependency injection as well, but for now we're just using its routing methods.

* $app->get('/locations/{id}', function ($request, $response, $args) {...}); - This first endpoint handles GET requests to an endpoint that follows the pattern /locations/{id} where id will be a unique indicator for a location. We'll go into more detail about this endpoint and the callback function later in the tutorial.

* $app->delete('/locations/{id}', function ($request, $response, $args) {...}); - This second endpoint handles DELETE requests to an endpoint that follows the pattern /locations/{id}. This will be for deleting the weather details for a specific location stored in our database.

* $app->run(); - Finally, this line runs the Slim application, enabling any endpoints defined above. This is usually the last line in any Slim project.

## Running Our Application for the First Time

Now we can run our application in a Docker container just to try it out. There will be two endpoints that won't really do anything, but at least it will let us verify that we're on track before we continue expanding the application. From the terminal, run:

<table>
  <tr>
    <td>$ docker run --rm -p 38000:80 -v $(pwd):/var/www/html php:apache</td>
  </tr>
</table>


Now if you navigate to http://localhost:38000/index.php/locations/1 in your web browser, you should see an empty page with the following text:

<table>
  <tr>
    <td>Location 1 retrieved.</td>
  </tr>
</table>


You've just successfully run your first PHP web application in Docker!

### What's going on here?

The docker run command we used this time was different from the two we used to run our hello.php script and our composer require... command. The reason is that this time we wanted to get the latest version of PHP with [Apache](https://httpd.apache.org/) included so we could serve our web application. Let's take a look at the new parts of the command in more detail:

* -p 38000:80 - Here we are defining [port mapping](https://docs.docker.com/engine/reference/commandline/port/) between our container and host system. This command says that Docker should map port 80 on the container to port 38000 on our host machine. You can pick any valid port on your host, but it's probably a good idea to use a high one (10000+) because many of the lower ones are reserved for built-in processes on most standard machines. For the container, you *must* use port 80 because that's the port the Docker image exposes and Apache runs on.

* -v $(pwd):/var/www/html - This time, we're mounting the code from our host machine to the container's /var/www/html directory. The reason is that the PHP Apache image serves code from this directory. This is a subtle but critical difference between the PHP scripts we ran before. Make sure to carefully read the [official PHP Docker image documentation](https://hub.docker.com/_/php/) to avoid missing things like this.

* php:apache - This image is the official PHP Apache image tag. It bundles PHP and Apache (a popular web server) into one container, allowing you to quickly serve your code for local development. While we won't cover it in this book, another option would be to use the php:fpm image and a linked nginx container.

* Finally, you'll notice that there's no command after the image name this time. This can trip new Docker users up, but by default, most images run some command even if you don't specify one. In the case of the PHP Apache image, the default command runs [a shell script](https://github.com/docker-library/php/blob/903540ea7918b5cabed6b32e81f8518f9e6f204f/7.1/apache/apache2-foreground) that starts Apache. This script is exactly what we want, so there's no reason to change it.

At this point, your terminal will show any Apache requests that come in, so when you loaded the first URL, you probably saw something like:

<table>
  <tr>
    <td>172.17.0.1 - - [21/Aug/2017:18:35:33 +0000] "GET /index.php/locations/1 HTTP/1.1" 200 250 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/60.0.3112.101 Safari/537.36"</td>
  </tr>
</table>


You can stop and exit the terminal by sending a "break" signal to the terminal. On Macs, this is done by holding control and pressing c.

## More Docker Run Options

### Running in detached mode

Another option for running your new web application is to run the container in "[detached mode](https://docs.docker.com/engine/reference/run/#detached-vs-foreground)." This means you will not see terminal output from your container. This is done by adding the -d flag to our previous command:

<table>
  <tr>
    <td>$ docker run -d --rm -p 38000:80 -v $(pwd):/var/www/html php:apache</td>
  </tr>
</table>


### Stopping a container

Immediately after starting the container in detached mode, your terminal will show the full ID of the new container - something like a403bf1018222a42ba... If you want to stop this container, you can tell Docker using the docker stop command with the container's ID. For example:

<table>
  <tr>
    <td>$ docker stop a403bf1018222a42ba</td>
  </tr>
</table>


Because typing that whole ID is cumbersome, Docker allows you to just type the first three or more characters if you prefer:

<table>
  <tr>
    <td>$ docker stop a40</td>
  </tr>
</table>


### Naming containers

Finally, I recommend [naming your containers](https://docs.docker.com/engine/reference/run/#container-identification). We will do this for many later examples in this book as it is much easier to remember a container by name than by the randomly assigned IDs, plus IDs are random, so each time you run a new version of the container, it gets a new ID. Names can be given out many times as long as there's not already a container with the same name. To name our new application container we could re-create it with the --name flag passed in:

<table>
  <tr>
    <td>$ docker run -d --rm --name=weather-app -p 38000:80 -v $(pwd):/var/www/html php:apache</td>
  </tr>
</table>


There are many more options available when using the [docker run](https://docs.docker.com/engine/reference/run/) command, so you may want to read over the documentation in more detail. We will cover some of these options as we develop the rest of our application.

## Connecting to the MetaWeather API

The next step in building our API is to retrieve data from the [MetaWeather API](https://www.metaweather.com/api/). This is mostly a PHP problem, and running in Docker shouldn't really matter but let's start off by making sure all our existing containers are stopped:

<table>
  <tr>
    <td>$ docker ps</td>
  </tr>
</table>


This command will [list all running containers](https://docs.docker.com/engine/reference/commandline/ps/). You can also see stopped containers by adding the -a flag.

If any containers are running, then use docker stop <ID> to stop them before we continue on.

### Adding Guzzle

While you can get data from MetaWeather's API using [PHP's built in cURL adapter](http://php.net/manual/en/book.curl.php), I prefer [Guzzle](http://docs.guzzlephp.org/en/stable/) as it takes less work to make API calls. Plus, this will give us an opportunity to install a new package using Composer. Open up the composer.json file that was automatically created when we set up Slim, and add a line to the require block for Guzzle:

<table>
  <tr>
    <td>{    "require": {        "slim/slim": "^3.0",        "guzzlehttp/guzzle": "~6.0"    }}</td>
  </tr>
</table>


Now in your terminal, run:

<table>
  <tr>
    <td>$ docker run --rm -v $(pwd):/app composer:latest update</td>
  </tr>
</table>


The container will install the new packages and update your composer.lock file. During this process, you should see some output like this in the terminal:

<table>
  <tr>
    <td>Loading composer repositories with package informationUpdating dependencies (including require-dev)Package operations: 3 installs, 0 updates, 0 removals  - Installing guzzlehttp/promises (v1.3.1): Downloading (100%)           - Installing guzzlehttp/psr7 (1.4.2): Downloading (100%)           - Installing guzzlehttp/guzzle (6.3.0): Downloading (100%)         guzzlehttp/guzzle suggests installing psr/log (Required for using the Log middleware)Writing lock fileGenerating autoload files</td>
  </tr>
</table>


Now Guzzle is installed, and ready to be used.

### Calling the MetaWeather API

The endpoint we'll be using takes a woeid ("[Where On Earth ID](https://developer.yahoo.com/geo/geoplanet/guide/concepts.html)") and returns the weather forecast in [JSON](https://www.json.org/) format. Initially, our application will just pass requests through to MetaWeather and return the same JSON data that they do. We'll add database caching in the next chapter. Here's our updated index.php file:

*index.php*

<table>
  <tr>
    <td><?php require 'vendor/autoload.php';// Create a new Container with Guzzle injected$container = new \Slim\Container([    'http' => function () {        return new GuzzleHttp\Client();    }]);// Instantiate the App object$app = new \Slim\App($container);// Get weather by location ID$app->get('/locations/{id}', function ($request, $response, $args) {    // Get the weather from MetaWeather     $result = $this->http->get("https://www.metaweather.com/api/location/{$args['id']}")        ->getBody()        ->getContents();    // Return the results as JSON    return $response->withStatus(200)->withJson(json_decode($result));});// The delete endpoint is unchanged$app->delete('/locations/{id}', function ($request, $response, $args) {    return $response->withStatus(200)->write("Location {$args['id']} deleted.");});// Run the application$app->run();</td>
  </tr>
</table>


### What's going on here?

We've made some updates - mostly to the get('/locations/{id}', ...) endpoint. Let's take a look at what's new here:

* $container = new \Slim\Container([...]); - If you're not familiar with Slim's Dependency Injection system, you can [read up on it here](https://www.slimframework.com/docs/concepts/di.html). Since this book isn't focused on dependency injection, I'll just say that this injects Guzzle into the application so we can use it in any of our routes by calling $this->http. It makes our code more concise, and allows us to mock the library if we add tests to the application later.

* $result = $this->http->get("https://www.metaweather.com/api/location/{$args['id']}")... - This is where we make the call to MetaWeather through Guzzle. As you can see, we're simply passing the locationId from our Slim route into the API endpoint provided by MetaWeather. We also call getBody() and getContents() since we just want to get the response as a string rather than a [stream](http://docs.guzzlephp.org/en/stable/psr7.html#streams).

* return $response->withStatus(200)->withJson(json_decode($result)); - Finally, we respond with a 200 response code and withJson(...). Slim provides this method to automatically set JSON response headers, but because the response from MetaWeather came back as a JSON string, we have to run json_decode on it before re-encoding it into JSON.

### Testing it out

Our application is ready to pass a real location ID to MetaWeather's API and return some data. First, find a location ID using [this WOEID lookup tool](http://woeid.rosselliot.co.nz/). I'm using my home town of Chicago's ID for testing: 2379574.

Now, let's run the Docker container again:

<table>
  <tr>
    <td>$ docker run -d --rm --name=weather-app -p 38000:80 -v $(pwd):/var/www/html php:apache</td>
  </tr>
</table>


And visit the running application in your browser at http://localhost:38000/index.php/locations/2379574. You should receive a response in JSON that looks something like this:

<table>
  <tr>
    <td>{  "consolidated_weather": [    {      "id": 5560109107249152,      "weather_state_name": "Heavy Rain",      "weather_state_abbr": "hr",      "wind_direction_compass": "SSW",      "created": "2017-08-21T17:23:20.408640Z",      "applicable_date": "2017-08-21",      "min_temp": 22.711666666666662,      "max_temp": 29.093333333333334,      "the_temp": 28.166666666666668,      "wind_speed": 5.319549422227524,      "wind_direction": 201.1261629987385,      "air_pressure": 1012.975,      "humidity": 74,      "visibility": 14.841140240992603,      "predictability": 77    },    ...  ],  "time": "2017-08-21T14:31:16.860660-05:00",  "sun_rise": "2017-08-21T06:05:09.010298-05:00",  "sun_set": "2017-08-21T19:42:46.959306-05:00",  "timezone_name": "LMT",  "parent": {    "title": "Illinois",    "location_type": "Region / State / Province",    "woeid": 2347572,    "latt_long": ""  },  "sources": [    {      "title": "BBC",      "slug": "bbc",      "url": "http://www.bbc.co.uk/weather/",      "crawl_rate": 180    },    ...  ],  "title": "Chicago",  "location_type": "City",  "woeid": 2379574,  "latt_long": "41.884151,-87.632408",  "timezone": "US/Central"}</td>
  </tr>
</table>


Your Dockerized PHP application is now returning real data from an external data source and being served in Apache, but you'll probably notice that it's not exactly the speediest application in the world (mine clocked in at 2.9 seconds page load time!).

MetaWeather is a free service, and not intended to be blazingly fast around the world. In order to fix that, we're going to save responses from MetaWeather in our own MySQL database and serve those saved responses up when we can. This will greatly improve performance, but will also show us how to add a database to a PHP application with Docker.

# Chapter 4. Connecting to the Database

Before we can connect a database, we need to make sure that our PHP container has all the necessary extensions installed. By default, the PHP Docker image from Docker Hub is pretty lightweight, so it doesn't include many PHP extensions or Linux packages you might need. This tradeoff between smaller and more flexible images is one you'll have to make based on your project priorities, but since we know we'll need MySQL for this project, let's extend the base PHP image with the required extensions.

The base PHP images are usually a good starting point, but I've also started keeping some PHP images with more common extensions enabled in Github. Check out [this repository for more](https://github.com/shiphp/dockerfiles).

## Creating a Custom Dockerfile

In the root of your weather-app project, next to the index.php file, create a new file called simply Dockerfile. Dockerfiles have no extension, and are used by Docker to configure and set up images. All the images on [Docker Hub](https://hub.docker.com/) have a corresponding Dockerfile, and usually you can find them on Github. [PHP's Dockerfiles are here](https://github.com/docker-library/php), but they may be a little confusing since they build on one-another. Our Dockerfile will be much simpler:

#### Dockerfile

<table>
  <tr>
    <td>FROM php:apacheRUN docker-php-source extract && docker-php-ext-install mysqli && docker-php-source delete</td>
  </tr>
</table>


### What's going on here?

This Dockerfile has two lines:

* FROM php:apache - This tells Docker where to start when building the image. You can start with just a Linux distribution (like [Ubuntu](https://hub.docker.com/_/ubuntu/) or the super light [Alpine](https://hub.docker.com/_/alpine/)), or you can extend a more specific container as we did in this case. Here we're extending the php:apache image we used to run our container before.

* RUN docker-php-source extract... - This is the line that adds the PHP extensions we need. In our case, we just want to use [mysqli functions](http://php.net/manual/en/book.mysqli.php) for our database. Since this is such a common extension, the PHP image has methods for adding it automatically when we extend the base image with these lines.

Check out [the official Docker documentation](https://docs.docker.com/engine/reference/builder/) for more options for extending existing images.

## Building the Image

In order to create an image, we will build it from the Dockerfile we created above:

<table>
  <tr>
    <td>$ docker build . -t shiphp/weather-app</td>
  </tr>
</table>


When you run this command, you will see a bunch of output as Docker builds your image, ending with something like:

<table>
  <tr>
    <td>Removing intermediate container 80b3f79714b2Successfully built 8dc1f8c175a1Successfully tagged shiphp/weather-app:latest</td>
  </tr>
</table>


### What's going on here?

The [Docker build](https://docs.docker.com/engine/reference/commandline/build/) command reads your Dockerfile and creates an image that can then be used to run a container. In this case, we're passing two parameters to the command:

* . - The dot lets Docker know the "context" of our build - in this case, it's the current directory. You can also use an absolute path like /Users/username/weather-app if you know the exact path of your Dockerfile. Docker automatically builds this image in the context of the directory containing your Dockerfile, so make sure your Dockerfile is located at the root of your project.

* -t shiphp/weather-app - The -t flag sets a "tag" for your image. This tag will allow you to more easily create a container from the image or push the image to a registry to share it with others.

You can see a list of all the Docker images on your local machine by running $ docker images. When you run this command, you should see something like this in your terminal:

<table>
  <tr>
    <td>REPOSITORY         TAG    IMAGE ID     CREATED        SIZEshiphp/weather-app latest 8dc1f8c175a1 27 minutes ago 391MB</td>
  </tr>
</table>


## Running a MySQL Container

In the introduction, I mentioned that Docker containers could be "linked" through [Docker's network feature](https://docs.docker.com/engine/userguide/networking/). In order to have our PHP application get data from our database, we need to link it to an active database container. Let's start a new MySQL container so we can link our web application to it:

<table>
  <tr>
    <td>$ docker run -d --rm --name weather-db -e MYSQL_USER=admin -e MYSQL_DATABASE=weather -e MYSQL_PASSWORD=p23l%v11p -e MYSQL_RANDOM_ROOT_PASSWORD=true mysql:5.7</td>
  </tr>
</table>


### What's going on here?

When you run the above docker run command, Docker will get the latest version of the [MySQL 5.7 image](https://hub.docker.com/_/mysql/) and start a container named weather-db. Like PHP, MySQL has [listed their Docker images on Docker Hub](https://hub.docker.com/_/mysql/), and you can use them in much the same way. Some of the flags above are new though, so let's take a closer look:

* -d - This runs the container in "detached" mode, meaning that you won't have to keep your terminal connected to the container.

* --name weather-db - Naming our database container is important because it's easier to connect to a named container. We can also use this name to connect to the MySQL database from within our PHP application as we'll see later.

* -e MYSQL_USER=admin - The [MySQL image](https://hub.docker.com/_/mysql/) includes several -e (environmental variable) options. This first one sets the username for the MySQL user we'll use in our PHP application. More about the environmental options is available on [the official Docker Hub image](https://hub.docker.com/_/mysql/).

* -e MYSQL_DATABASE=weather - By default, the container will not create a database. While you can create one manually by logging into the container (which will be covered in the next section), it's easier to create the database up front like this.

* -e MYSQL_PASSWORD=p23l%v11p - This sets our database password to something besides the default. Even though it isn't a huge security risk as we're just doing local development, you should set this to your own strong password.

* -e MYSQL_RANDOM_ROOT_PASSWORD=true - Your root user password should be set to something random as you're not going to be logging in with the root user anyway.

* mysql:5.7 As with the PHP image, you can choose a version of MySQL to use. Here I've elected to use the version 5.7.

As with the php:apache image, the default image command is fine, so we don't need to add anything after the image.

## Logging into a Running Container

At this point, we want to check that the database in our new container is working, but how can we access a running Docker container? We will use the docker exec command to enter the terminal for our new container, and then once we're inside, we'll use the [MySQL command line interface](https://dev.mysql.com/doc/refman/5.7/en/mysql.html) to access the database we created.

We can enter a bash session on the running database container using:

<table>
  <tr>
    <td>$ docker exec -it weather-db bash</td>
  </tr>
</table>


You should see a line like root@35c7ad5a574d:/# (with your container's ID) on your terminal, indicating that you're now logged in to the running container. From within the container (not on your host machine), run:

<table>
  <tr>
    <td>$ mysql --user=admin --passwordEnter password: </td>
  </tr>
</table>


Enter the password you chose for the MySQL container you created above (in this example it was p23l%v11p), and hit the "return" key. You'll see some information about MySQL like this:

<table>
  <tr>
    <td>Welcome to the MySQL monitor.  Commands end with ; or \g.Your MySQL connection id is 10Server version: 5.7.19 MySQL Community Server (GPL)Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.Oracle is a registered trademark of Oracle Corporation and/or itsaffiliates. Other names may be trademarks of their respectiveowners.Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.mysql> </td>
  </tr>
</table>


Finally, let's just check that our database was created using the show databases command:

<table>
  <tr>
    <td>mysql> show databases;+--------------------+| Database           |+--------------------+| information_schema || weather            |+--------------------+2 rows in set (0.01 sec)</td>
  </tr>
</table>


### What's going on here?

In the section above, we logged into a terminal on the running database container, we then logged into MySQL via the command line interface, and finally, we took a look at the databases available to us. Since you may not be familiar with all these commands, we'll review them one-by-one.

#### Docker Exec

* docker exec - This is the [Docker command for executing commands](https://docs.docker.com/engine/reference/commandline/exec/) on a running container. Like Docker's run command, there are many options for the exec command, but we'll just cover the highlights for this example.

* -it - The -it flag is actually two flags which are commonly used together. The -i flag runs the session in "interactive" mode, meaning the STDIN and STDOUT from your terminal are attached to the container's terminal. The -t flag attaches a pseudoterminal to the connection. There's a little more explained in [this Stack Overflow question](https://stackoverflow.com/a/22287905/977192), but suffice it to say, when running docker exec bash commands, you will probably want to use the -it flags.

* weather-db - This is the name (you could also use the ID) of the running container we want to run our command on.

* bash - Finally, this is the command that we want to run in the container. [bash](https://en.wikipedia.org/wiki/Bash_(Unix_shell)) is a shell installed in most Linux distributions that allows us to run other programs on the container. If your container doesn't have bash installed, you can [try ](http://pubs.opengroup.org/onlinepubs/009695399/utilities/sh.html)[sh](http://pubs.opengroup.org/onlinepubs/009695399/utilities/sh.html)[ instead](http://pubs.opengroup.org/onlinepubs/009695399/utilities/sh.html).

#### MySQL CLI

The [MySQL CLI](https://dev.mysql.com/doc/refman/5.7/en/mysql.html) also provides many options, but here are the three we used:

* mysql - This is the command used to access MySQL from a terminal. If you've been using MySQL on a VM or your local machine, this is no different.

* --user=admin - We specify the username here so that MySQL knows we want access as a specific user and not root.

* --password - The password flag instructs MySQL to give us the password prompt. You can also enter the password directly into the command, but it's generally less secure.

#### Exiting MySQL and the Container

We're done with the MySQL container for now, so let’s exit and stop the container:

* To exit the MySQL CLI, type \q and then press "return".

* Exit the container by typing exit and then pressing "return".

* Stop the container by typing docker stop weather-db and then "return" again.

The whole command line output should look something like this when you're done:

<table>
  <tr>
    <td>mysql> \qByeroot@b01aff352dd7:/# exitexit$ docker stop weather-dbweather-db</td>
  </tr>
</table>


We have now proven that our MySQL container runs and that we can log in to access the database, but how will we keep data saved in the container? What happens when we stop this container? Persistence is important in real-world applications, so in the next section, we'll dive into keeping our data even after our container is long gone.

## Retaining Data in Our Database Container

So far, our database container starts up and automatically creates a database and user. This is fine, but when we connect this database to our PHP application, we'll want to make sure that the database tables and values are saved even after our container stops. If not, we'll have to rebuild the database every time our database container restarts.

The best way to save data from containers for local development is by mounting a volume. We saw how effective this was for local development of our PHP code, and the process we follow for mounting our database's data is very similar. When working in a PHP project, I like to mount database volumes from the project directory (in our case, the weather-app/ directory we created earlier), but you can mount the volume from anywhere on your host system.

It's important to note that when mounting volumes from a Mac host computer, there are [performance tradeoffs](https://docs.docker.com/docker-for-mac/osxfs-caching/). These should not be an issue in Linux environments, but I've found that when running mounted volumes in many linked containers, things can get painfully slow. Docker provides [some hints for tuning here](https://docs.docker.com/docker-for-mac/osxfs-caching/).

To start the MySQL container with the database saved to our host system, navigate to your new weather-app project directory and type:

<table>
  <tr>
    <td>$ docker run -d --rm --name weather-db -e MYSQL_USER=admin -e MYSQL_DATABASE=weather -e MYSQL_PASSWORD=p23l%v11p -e MYSQL_RANDOM_ROOT_PASSWORD=true -v $(pwd)/.data:/var/lib/mysql mysql:5.7</td>
  </tr>
</table>


### What's going on here?

The only thing we added to this docker run command was -v $(pwd)/.data:/var/lib/mysql. This mounts a volume from our local directory into the MySQL container. A new directory will appear (.data/) and as the database container starts up, folders and files will appear. These are the files that MySQL uses to store your data, and you can view the files being created from your terminal:

<table>
  <tr>
    <td>$ ls -a .data/</td>
  </tr>
</table>


You should see files and folders listed out like this:

<table>
  <tr>
    <td>.                   ib_buffer_pool      private_key.pem..                  ib_logfile0         public_key.pemauto.cnf            ib_logfile1         server-cert.pemca-key.pem          ibdata1             server-key.pemca.pem              ibtmp1              sysclient-cert.pem     mysql               weatherclient-key.pem      performance_schema</td>
  </tr>
</table>


Don't worry about what each of those mean (although [you can read up on MySQL internals if you're interested](https://dev.mysql.com/doc/internals/en/)). At this point, we just care that MySQL is now using data from our host machine in the container, allowing us to keep the database data even after the container is shut down.

Be sure to add the .data directory to your .gitignore file so that your database isn't added to version control.

## Creating a Database Table

Since this application just needs to cache the results from the MetaWeather API, the database will just have one table with three columns. We'll call the table locations and add columns id, weather, and last_updated.

![image alt text]({{ site.url }}/public/WrhST5eBKSgLJSvgr54cg_img_1.png)

*Diagram 3: Weather Application Database*

If the MySQL container is not running, then start it using the method above that mounts data in a volume:

<table>
  <tr>
    <td>$ docker run -d --rm --name weather-db -e MYSQL_USER=admin -e MYSQL_DATABASE=weather -e MYSQL_PASSWORD=p23l%v11p -e MYSQL_RANDOM_ROOT_PASSWORD=true -v $(pwd)/.data:/var/lib/mysql mysql:5.7</td>
  </tr>
</table>


Log into the container and into the MySQL CLI. This time, we're going to do it in just one step:

<table>
  <tr>
    <td>$ docker exec -it weather-db mysql --user=admin --password=p23l%v11p weather</td>
  </tr>
</table>


Next, we're going to run a SQL command to create the database table we outlined above:

<table>
  <tr>
    <td>mysql> CREATE TABLE locations (id VARCHAR(64) NOT NULL, weather JSON NULL, last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP);</td>
  </tr>
</table>


It should tell you Query OK, but you can check by running mysql> SHOW TABLES;. Exit the container and MySQL CLI by typing \q.

### What's going on here?

Unlike the last time we logged in to the MySQL command line, this time we cut the process down to just one step. Remember that when running docker run or docker exec, the last part is the command we want to run in the container. This means that we don't actually have to log into a bash session and then log into MySQL, we can just directly log in to the database CLI.

This can be done with any command available on the container, so docker exec becomes very useful for seeing what's actually going on inside your containers.

## Saving to the Database from our PHP Application

Now that our database is ready to store weather data and it will keep that data even after the container is removed, we need to update our PHP application to connect to and store weather results in the database.

Initially, the index.php file was getting data directly from the MetaWeather API, but now that we have a storage mechanism in place (ie: our database), let's get the application to save data to the database for repeat requests.

#### index.php

<table>
  <tr>
    <td><?php require 'vendor/autoload.php';// Create a new Container$container = new \Slim\Container([    // Add Guzzle as 'http'    'http' => function () {        return new GuzzleHttp\Client();    },    // Add mysqli as 'mysql'    'mysql' => function () {        $mysqli = new mysqli(            "weather-db",            "admin",            "p23l%v11p",            "weather"        );        if ($mysqli->connect_errno) {            echo "Failed to connect to MySQL: " . $mysqli->connect_error;            exit;        } else {            return $mysqli;        }    },]);// Instantiate the App object$app = new \Slim\App($container);// Get weather by location ID$app->get('/locations/{id}', function ($request, $response, $args) {    // Get the location from the database    $id = $this->mysql->real_escape_string($args['id']);    $results = $this->mysql->query("SELECT * FROM locations WHERE id='{$id}'");    // If location found, then show it from the DB, otherwise, query MetaWeather    if ($results->num_rows > 0) {        $result = $results->fetch_assoc()['weather'];    } else {        $result = $this->http->get("https://www.metaweather.com/api/location/{$id}")            ->getBody()            ->getContents();        $cleanResult = $this->mysql->real_escape_string($result);        if (!$this->mysql->query("INSERT into locations (id, weather) VALUES ('{$id}', '{$cleanResult}')")) {            throw new Exception("Location could not be updated.");        }    }    // Return the results as JSON    return $response->withStatus(200)->withJson(json_decode($result));});// Run the application$app->run();</td>
  </tr>
</table>


### What's going on here?

We've added two important things to our index.php file above. First, we updated the $container variable:

<table>
  <tr>
    <td>$container = new \Slim\Container([    // Add Guzzle as 'http'    'http' => function () {        return new GuzzleHttp\Client();    },    // Add mysqli as 'mysql'    'mysql' => function () {        $mysqli = new mysqli(            "weather-db",   // The database host            "admin",        // The database user            "p23l%v11p",    // The database password            "weather"       // The database name        );        if ($mysqli->connect_errno) {            echo "Failed to connect to MySQL: " . $mysqli->connect_error;            exit;        } else {            return $mysqli;        }    },]);</td>
  </tr>
</table>


Guzzle is the same, but we've also added an element to the array named 'mysql'. This callback instantiates [MySQLi](http://php.net/manual/en/book.mysqli.php) with hard-coded database credentials.

Hard-coding your database credentials like this is not a good idea in a real application. We'll cover a more secure way to include credentials via environmental variables in Docker in the last section of this chapter. In the meantime, don't commit this to a public repository unless you plan on changing your database password.

The other major update to the file is the GET /locations endpoint:

<table>
  <tr>
    <td>$app->get('/locations/{id}', function ($request, $response, $args) {    // Get the location from the database    $id = $this->mysql->real_escape_string($args['id']);    $results = $this->mysql->query("SELECT * FROM locations WHERE id='{$id}'");    // If location found, show it from DB, otherwise, query MetaWeather    if ($results->num_rows > 0) {        $result = $results->fetch_assoc()['weather'];    } else {        $result = $this->http->get("https://www.metaweather.com/api/location/{$id}")            ->getBody()            ->getContents();        $cleanResult = $this->mysql->real_escape_string($result);        if (!$this->mysql->query("INSERT into locations (id, weather) VALUES ('{$id}', '{$cleanResult}')")) {            throw new Exception("Location could not be updated.");        }    }    // Return the results as JSON    return $response->withStatus(200)->withJson(json_decode($result));});</td>
  </tr>
</table>


While the comments in the code may help, it's worth adding some more detail in case you're not familiar with MySQLi:

* $id = $this->mysql->real_escape_string($args['id']); - First, we escape the id passed in through the endpoint's $args variable. This is a good idea anytime you're taking user or third-party input as they might accidentally (or intentionally) inject SQL statements that break your code.

* $results = $this->mysql->query("SELECT * FROM locations WHERE id='{$id}'"); - While I would typically [use an ORM](https://stackoverflow.com/a/1279678) for a real application, this simple two-endpoint example didn't seem to warrant it. Instead, we're using [MySQLi - a PHP extension for accessing MySQL -](http://php.net/manual/en/book.mysqli.php) to query the database. This allows us to write raw SQL with PHP variables embedded.

* if ($results->num_rows > 0) {...} - If we find at least one result for this location ID, we should return the result from the database. If not, we'll continue on...

* else { ... $this->mysql->query("INSERT into locations (id, weather) VALUES ('{$id}', '{$cleanResult}')") ... - If we did not find a result for this location ID in the database, we're getting it from the MetaWeather API, escaping the JSON string, and then inserting the result into the database.

* return $response->withStatus(200)->withJson(json_decode($result)); - Just as we did before, we're returning any result as JSON to the user.

## Linking the PHP Container

With the code in place, now we need to run a new PHP container linked to the MySQL database container we just started:

<table>
  <tr>
    <td>$ docker run -d --rm --name=weather-app -p 38000:80 -v $(pwd):/var/www/html --link weather-db shiphp/weather-app</td>
  </tr>
</table>


### What's going on here?

There are two new parts to this docker run command:

* --link weather-db - This links the container named weather-db to this container. You can also [give the linked container an alias](https://docs.docker.com/engine/reference/run/#expose-incoming-ports) for use within the PHP container, but we're not going to need to do that here.

* shiphp/weather-app - Instead of using the php:apache image we used in the previous chapter, we're using the new Docker image we built at the beginning of this chapter. The reason for this is that we need the PHP MySQLi extension that we added in our custom Dockerfile.

Now that the PHP container is up and running, you should be able to navigate to a location ID like the following:

<table>
  <tr>
    <td>GET http://localhost:38000/index.php/locations/2487956</td>
  </tr>
</table>


The first time you load this URL, it will probably take a second or two to load, but if you refresh the page, you should see a dramatically faster result. Mine loaded in less than 150 ms! That's thanks to the caching we just set up by saving the result in the database.

## Deleting Data

In order to remove data from the database, we'll add a second endpoint. Add the following just after the $app->get(...); endpoint to allow users to delete a location from the database:

#### index.php (partial)

<table>
  <tr>
    <td>...$app->delete('/locations/{id}', function ($request, $response, $args) {    // Get the location from the database    $id = $this->mysql->real_escape_string($args['id']);    $results = $this->mysql->query("SELECT * FROM locations WHERE id='{$id}'");    // If it exists, delete it, otherwise send a 404    if (        $results->num_rows > 0 &&        $this->mysql->query("DELETE FROM locations WHERE id='{$id}'")    ) {        return $response->withStatus(200)->write("Location {$args['id']} deleted.");    } else {        return $response->withStatus(404)->write("Location {$args['id']} not found.");    }});// Run the application$app->run();</td>
  </tr>
</table>


Now you can use [cURL](https://www.garron.me/en/bits/curl-delete-request.html) or [Postman](https://www.getpostman.com/) to make a DELETE call to the same location endpoint you did above. For example:

<table>
  <tr>
    <td>DELETE http://localhost:38000/index.php/locations/2487956</td>
  </tr>
</table>


You should see a message Location 2487956 deleted. and now when you make a GET request to that URL again, you'll wait longer as the result must come from the MetaWeather API.

## Managing Environmental Variables

The last topic we'll cover in this chapter is managing environmental variables in Docker. Docker allows you to specify environmental variables at runtime, so that your PHP code can access these variables, but you won't have to hard-code them as we did above.

First, we'll need to change the code that specifies our database credentials in the index.php file:

<table>
  <tr>
    <td>// Create a new Container$container = new \Slim\Container([    // Add Guzzle as 'http'    'http' => function () {        return new GuzzleHttp\Client();    },    // Add mysqli as 'mysql'    'mysql' => function () {        $mysqli = new mysqli(            getenv('DATABASE_HOST'),            getenv('DATABASE_USER'),            getenv('DATABASE_PASSWORD'),            getenv('DATABASE_NAME')        );        if ($mysqli->connect_errno) {            echo "Failed to connect to MySQL: " . $mysqli->connect_error;            exit;        } else {            return $mysqli;        }    },]);</td>
  </tr>
</table>


Stop the PHP container, and re-run it using this command:

<table>
  <tr>
    <td>$ docker run -d --rm --name=weather-app -p 38000:80 -v $(pwd):/var/www/html --link weather-db -e DATABASE_HOST='weather-db' -e DATABASE_USER='admin' -e DATABASE_PASSWORD='p23l%v11p' -e DATABASE_NAME='weather' shiphp/weather-app</td>
  </tr>
</table>


Now you can safely share your PHP code without revealing your database login credentials.

### What's going on here?

PHP can read environmental variables from the operating system [using the getenv function](http://php.net/manual/en/function.getenv.php), and when we use this function in a container, the effect is the same. Instead of reading environmental variables from the host operating system (ie: your computer), it read environmental variables from the Docker container running the code. This helps with security - different containers aren't sharing environmental variables - and it helps make sharing code easier.

The updated docker run command now includes four -e flags that translate to environmental variables in the container. Now, your PHP application will read the environmental variables you set when you run the container instead of hard-coded values or values from your host machine.

If you don't want to have to type long commands like this in every time you run your container, you can use Docker Compose and/or an .env file instead. This is covered in [this blog post](https://www.shiphp.com/blog/2017/env-php-docker).

Finally, you can see the environmental variables your container is using:

<table>
  <tr>
    <td>$ docker inspect weather-app</td>
  </tr>
</table>


The JSON returned from this command gives you a lot of information about a running container, but for this example, we just care about the array at Config.Env:

<table>
  <tr>
    <td>[    "DATABASE_HOST=weather-app",    "DATABASE_USER=admin",    "DATABASE_PASSWORD=p23l%v11p",    "DATABASE_NAME=weather",    ...],...</td>
  </tr>
</table>


If everything worked correctly, you should now be able to access the application as before. In the next chapter, we'll take a quick look at how we might improve this application using advanced Docker topics. If you want to take a look at the complete source code for this app, it's [available on Github](https://github.com/shiphp/weather-app).

# Chapter 5. Next Steps

At this point, our application is working and we can retrieve weather results from the API or from our database cache, but there's still a lot more to learn. Building one demo application won't make you an expert with Docker, so let's take a look at some things we might do to continue to improve this application and learn more about Docker.

## Further Development

### Clean up long Docker commands

Right now, starting and stopping our container is an arduous process and quite prone to error. I'm guessing you don't want to type docker run -d --rm --name=weather-app -p 38000:80 -v $(pwd):/var/www/html --link weather-db -e DATABASE_HOST='weather-db' -e DATABASE_USER='admin' -e DATABASE_PASSWORD='p23l%v11p' -e DATABASE_NAME='weather' shiphp/weather-app every time you want to start editing your code, but you can pretty easily avoid this by writing [npm scripts](https://docs.npmjs.com/cli/run-script), [bash commands](http://tldp.org/LDP/abs/html/), or PHP scripts.

### Docker Compose

Another way to clean up our Docker workflow is to use [Docker Compose](https://docs.docker.com/compose/). Compose will allow us to write a single configuration file that starts all of our containers and links them together automatically. No more typing two or three commands just to get our app running - plus, a Docker Compose file can be used to host our containers in a [Docker Swarm](https://docs.docker.com/swarm/).

### Hosting

This walkthrough has focused on local development with Docker, but you can also host highly scalable and available web applications in Docker containers. There's no end to the number of ways you can configure applications to run in the cloud, but look into [Docker Swarm](https://docs.docker.com/engine/swarm/), [Kubernetes](https://kubernetes.io/), [Amazon EC2](https://aws.amazon.com/ec2/), and [Hyper.sh](https://hyper.sh/) for just a few options.

### Accessing our database externally

Right now, our application's database is only accessible from the container, but what if we want to browse it with a tool like [Datagrip](https://www.jetbrains.com/datagrip/), [phpMyAdmin](https://www.phpmyadmin.net/), or [Sequel Pro](https://www.sequelpro.com/)? We'll have to expose a port and consider the security implications of doing so.

### Testing and continuous integration

I'm a huge advocate for testing and continuous integration, and one of Docker's advantages is that it can make tests faster and more reproducible across environments. My favorite tool for continuous integration is [Codeship](https://codeship.com/), but Docker works with [Jenkins](https://jenkins.io/), [Travis](https://travis-ci.org/), and most other CI platforms as well.

### Reading and saving logs

Finally, debugging your PHP application in Docker via the logs is probably a necessary step for any real-world app. Docker has a powerful [logs](https://docs.docker.com/engine/reference/commandline/logs/) command that we haven't covered yet.

## Resources

While this book doesn't cover the above topics in detail, there are many great free resources available for learning Docker in more depth. Some of them include:

* [The official Docker Documentation](https://docs.docker.com/) - The documentation is a bit dry, but you should definitely bookmark it as a reference. I find myself keeping this in an open tab almost all the time.

* [Stack Overflow](https://stackoverflow.com/questions/tagged/docker) - Very specific questions can be found here, but some of the more popular questions will also reveal helpful best practices in their answers.

* [The New Stack's Docker Book Series](https://thenewstack.io/ebookseries/) - Good for an overview of terminology and tools available for developing with Docker. This book series is pretty high-level.

In addition, we regularly post Docker tips and tutorials on the [shiphp.com blog](https://www.shiphp.com/#blog). Posts there relate to building PHP applications on Docker using specific frameworks like Wordpress or Laravel as well as general PHP topics and updates on the language.

## Final Words

I hope that reading and working through this book has piqued your interest in Docker and given you the confidence to start using it regularly for local development. I have found that the best way for me to learn something new is to just start building things, and Docker has been no different. When I decided that Docker was how I would build PHP apps, I uninstalled my virtual machine and went all-in. While you may not be ready to do that just yet, forcing yourself to solve problems with Docker will make you get better at it.

Finally, if you have feedback for this book, questions for me, or you're interested in PHP or Docker books or training, my contact information is available at [shiphp.com](https://www.shiphp.com/). Thanks for reading!

## Quick Reference

Source: [WSargent's Docker Cheat Sheet](https://github.com/wsargent/docker-cheat-sheet)

### Images

**[docker image**s](https://docs.docker.com/engine/reference/commandline/images) shows all images.

**[docker buil**d](https://docs.docker.com/engine/reference/commandline/build) creates image from Dockerfile.

**[docker commi**t](https://docs.docker.com/engine/reference/commandline/commit) creates image from a container, pausing it if it's running.

**[docker rm**i](https://docs.docker.com/engine/reference/commandline/rmi) removes an image.

**[docker histor**y](https://docs.docker.com/engine/reference/commandline/history) shows history of image.

**[docker ta**g](https://docs.docker.com/engine/reference/commandline/tag) tags an image to a name (local or registry).

### Container Lifecycle

**[docker creat**e](https://docs.docker.com/engine/reference/commandline/create) creates a container but does not start it.

**[docker ru**n](https://docs.docker.com/engine/reference/commandline/run) creates and starts a container in one operation.

**[docker r**m](https://docs.docker.com/engine/reference/commandline/rm) deletes a container.

### Starting and Stopping Containers

**[docker star**t](https://docs.docker.com/engine/reference/commandline/start) starts a container so it is running.

**[docker sto**p](https://docs.docker.com/engine/reference/commandline/stop) stops a running container.

**[docker restar**t](https://docs.docker.com/engine/reference/commandline/restart) stops and starts a container.

**[docker paus**e](https://docs.docker.com/engine/reference/commandline/pause/) pauses a running container, "freezing" it in place.

**[docker unpaus**e](https://docs.docker.com/engine/reference/commandline/unpause/) will unpause a running container.

**[docker exe**c](https://docs.docker.com/engine/reference/commandline/exec) to execute a command in container.

**[docker attac**h](https://docs.docker.com/engine/reference/commandline/attach) will connect to a running container.

### Container Information

**[docker p**s](https://docs.docker.com/engine/reference/commandline/ps) shows running containers.

**[docker log**s](https://docs.docker.com/engine/reference/commandline/logs) gets logs from container.

**[docker inspec**t](https://docs.docker.com/engine/reference/commandline/inspect) looks at all the info on a container.

**[docker por**t](https://docs.docker.com/engine/reference/commandline/port) shows public facing port of container.

**[docker stat**s](https://docs.docker.com/engine/reference/commandline/stats) shows containers' resource usage statistics.

**[docker dif**f](https://docs.docker.com/engine/reference/commandline/diff) shows changed files in the container's FS.

## About the Author

[Karl Hughes](https://www.karllhughes.com/) has been building PHP web applications for education technology companies since 2011. He went to the University of Tennessee and graduated with a major in Mechanical Engineering, a minor in Business Administration, and a passion for writing both prose and software.

In addition to blogging at [www.karllhughes.com](http://www.karllhughes.com) and [www.shiphp.com](http://www.shiphp.com) he has contributed to Codeship's Blog, php[architect] magazine, and regularly speaks at meetups and conferences. He currently lives in Chicago and works as the Chief Technology Officer at [The Graide Network](https://www.thegraidenetwork.com/).

