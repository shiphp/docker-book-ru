# Chapter 3. Creating a SlimPHP Application

In the rest of this book, we'll focus on our web application: a weather checker. We'll use the [MetaWeather API](https://www.metaweather.com/api/) to search for a location and get the weather there. When we find results, we'll save them to the database so that future requests will get the cached version of the results.

> I chose the MetaWeather API because it is free and does not require an API key. There are other weather APIs you can use for free, but most require you to sign up and get a key.

Our application will provide a very simple REST API that we'll use to interact with the service. We'll create two endpoints:

* `GET /locations/:location_id` - Gets the weather from the database (if it exists) or MetaWeather if a cached version does not.

* `DELETE /locations/:location_id` - Deletes the cached version of the MetaWeather results from the database. The next call to GET weather for this location will come from the MetaWeather API.

Before we get to the application logic, let's start simple and get a fresh installation of [SlimPHP](https://www.slimframework.com/) installed and running using Docker.

## Installing Slim Framework

Slim is a minimal PHP web development framework. Unlike more comprehensive frameworks like [Laravel](https://laravel.com/) or [CakePHP](https://cakephp.org/), [Slim](https://www.slimframework.com/) does not include much more than a router, middleware stack, and dependency injector. If you're familiar with [Node's Express framework](https://expressjs.com/), Slim will feel familiar to you, but even if you're not, it's such a small framework that it's pretty easy to grasp. That's why I've chosen it for this book: I don't want the framework to throw you off, but I didn't want to build everything from scratch either as the focus here is on Docker, not our PHP application.

Setting up Slim with Docker actually takes fewer commands than doing it with a locally installed version of PHP. We will use [Composer](https://getcomposer.org/), but thanks to Docker we don't even need to install it on our host machine.

From your command line, create a new directory for your Dockerized PHP application:

{linenos=off, lang=sh}
~~~~~~~
$ mkdir weather-app
~~~~~~~

And navigate to it:

{linenos=off, lang=sh}
~~~~~~~
$ cd weather-app
~~~~~~~

Now install Slim using [the official Composer Docker image](https://hub.docker.com/_/composer/):

{linenos=off, lang=sh}
~~~~~~~
$ docker run --rm -v $(pwd):/app composer:latest require slim/slim "^3.0"
~~~~~~~

If you view the `weather-app/` directory, you'll see two files (`composer.json` and `composer.lock`) and one directory (`vendor/`):

{linenos=off, lang=sh}
~~~~~~~
$ ls
composer.json   composer.lock   vendor
~~~~~~~

### What's going on here?

Our docker run command was similar to the one we ran in Chapter 2, but we used a different image. Instead of the PHP image we used to run the hello.php script, we used a Composer image. Let's take a look at what changed.

* `composer:latest` - This indicates the image we're using for this container. You can specify a specific version of Composer if you need to; just check the list of image tags supported on [Docker Hub](https://hub.docker.com/_/composer/).

* `require slim/slim "^3.0"` - This is the command that actually installs Slim PHP into the container's working directory. Because that working directory is a volume from our host directory, the files composer installs are now present on your host machine as well as in the container.

At this point, your application doesn't actually do anything, but Slim has been installed and we now have some idea how composer works with PHP in Docker.

## Stubbing Out Endpoints

In order to give our application life, we need to have a couple endpoints. As mentioned above, Slim is little more than a router, but that actually takes care of most of our application. Let's start out by creating an index.php file with a couple stubbed out endpoints that we can build upon throughout the rest of this tutorial.

{title="index.php", linenos=off, lang=php}
~~~~~~~
<?php

require 'vendor/autoload.php';

// Instantiate the App object
$app = new \Slim\App();

// Declare routes
$app->get('/locations/{id}', function ($request, $response, $args) {
    return $response->withStatus(200)->write("Location {$args['id']} retrieved.");
});

$app->delete('/locations/{id}', function ($request, $response, $args) {
    return $response->withStatus(200)->write("Location {$args['id']} deleted.");
});

// Run the application
$app->run();
~~~~~~~

### What's going on here?

This single file creates two API endpoints that simply return text for now. In case you're not familiar with Composer or Slim's routing system, here's what's going on:

* `require 'vendor/autoload.php';` - Any PHP project that uses Composer must include Composer's autoload.php file somewhere. Since our new application is just one file, we've added it as the first thing.

* `$app = new \Slim\App();` - This creates a new Slim PHP application instance. Slim can handle middleware and help with dependency injection as well, but for now we're just using its routing methods.

* `$app->get('/locations/{id}', function ($request, $response, $args) {...});` - This first endpoint handles GET requests to an endpoint that follows the pattern /locations/{id} where id will be a unique indicator for a location. We'll go into more detail about this endpoint and the callback function later in the tutorial.

* `$app->delete('/locations/{id}', function ($request, $response, $args) {...});` - This second endpoint handles DELETE requests to an endpoint that follows the pattern /locations/{id}. This will be for deleting the weather details for a specific location stored in our database.

* `$app->run();` - Finally, this line runs the Slim application, enabling any endpoints defined above. This is usually the last line in any Slim project.

## Running Our Application for the First Time

Now we can run our application in a Docker container just to try it out. There will be two endpoints that won't really do anything, but at least it will let us verify that we're on track before we continue expanding the application. From the terminal, run:

{linenos=off, lang=sh}
~~~~~~~
$ docker run --rm -p 38000:80 -v $(pwd):/var/www/html php:apache
~~~~~~~

Now if you navigate to http://localhost:38000/index.php/locations/1 in your web browser, you should see an empty page with the following text:

`Location 1 retrieved.`

You've just successfully run your first PHP web application in Docker!

### What's going on here?

The `docker run` command we used this time was different from the two we used to run our hello.php script and our `composer require...` command. The reason is that this time we wanted to get the latest version of PHP with [Apache](https://httpd.apache.org/) included so we could serve our web application. Let's take a look at the new parts of the command in more detail:

* `-p 38000:80` - Here we are defining [port mapping](https://docs.docker.com/engine/reference/commandline/port/) between our container and host system. This command says that Docker should map port 80 on the container to port 38000 on our host machine. You can pick any valid port on your host, but it's probably a good idea to use a high one (10000+) because many of the lower ones are reserved for built-in processes on most standard machines. For the container, you *must* use port 80 because that's the port the Docker image exposes and Apache runs on.

* `-v $(pwd):/var/www/html` - This time, we're mounting the code from our host machine to the container's `/var/www/html` directory. The reason is that the PHP Apache image serves code from this directory. This is a subtle but critical difference between the PHP scripts we ran before. Make sure to carefully read the [official PHP Docker image documentation](https://hub.docker.com/_/php/) to avoid missing things like this.

* `php:apache` - This image is the official PHP Apache image tag. It bundles PHP and Apache (a popular web server) into one container, allowing you to quickly serve your code for local development. While we won't cover it in this book, another option would be to use the `php:fpm` image and a linked `nginx` container.

* Finally, you'll notice that there's no command after the image name this time. This can trip new Docker users up, but by default, most images run some command even if you don't specify one. In the case of the PHP Apache image, the default command runs [a shell script](https://github.com/docker-library/php/blob/903540ea7918b5cabed6b32e81f8518f9e6f204f/7.1/apache/apache2-foreground) that starts Apache. This script is exactly what we want, so there's no reason to change it.

At this point, your terminal will show any Apache requests that come in, so when you loaded the first URL, you probably saw something like:

{linenos=off, lang=sh}
~~~~~~~
172.17.0.1 - - [21/Aug/2017:18:35:33 +0000] "GET /index.php/locations/1 HTTP/1.1" 200 250 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/60.0.3112.101 Safari/537.36"
~~~~~~~

You can stop and exit the terminal by sending a "break" signal to the terminal. On Macs, this is done by holding `control` and pressing `c`.

## More Docker Run Options

### Running in detached mode

Another option for running your new web application is to run the container in "[detached mode](https://docs.docker.com/engine/reference/run/#detached-vs-foreground)." This means you will not see terminal output from your container. This is done by adding the `-d` flag to our previous command:

{linenos=off, lang=sh}
~~~~~~~
$ docker run -d --rm -p 38000:80 -v $(pwd):/var/www/html php:apache
~~~~~~~

### Stopping a container

Immediately after starting the container in detached mode, your terminal will show the full ID of the new container - something like `a403bf1018222a42ba...` If you want to stop this container, you can tell Docker using the `docker stop` command with the container's ID. For example:

{linenos=off, lang=sh}
~~~~~~~
$ docker stop a403bf1018222a42ba
~~~~~~~

Because typing that whole ID is cumbersome, Docker allows you to just type the first three or more characters if you prefer:

{linenos=off, lang=sh}
~~~~~~~
$ docker stop a40
~~~~~~~

### Naming containers

Finally, I recommend [naming your containers](https://docs.docker.com/engine/reference/run/#container-identification). We will do this for many later examples in this book as it is much easier to remember a container by name than by the randomly assigned IDs, plus IDs are random, so each time you run a new version of the container, it gets a new ID. Names can be given out many times as long as there's not already a container with the same name. To name our new application container we could re-create it with the `--name` flag passed in:

{linenos=off, lang=sh}
~~~~~~~
$ docker run -d --rm --name=weather-app -p 38000:80 -v $(pwd):/var/www/html php:apache
~~~~~~~

There are many more options available when using the [docker run](https://docs.docker.com/engine/reference/run/) command, so you may want to read over the documentation in more detail. We will cover some of these options as we develop the rest of our application.

## Connecting to the MetaWeather API

The next step in building our API is to retrieve data from the [MetaWeather API](https://www.metaweather.com/api/). This is mostly a PHP problem, and running in Docker shouldn't really matter but let's start off by making sure all our existing containers are stopped:

{linenos=off, lang=sh}
~~~~~~~
$ docker ps
~~~~~~~

This command will [list all running containers](https://docs.docker.com/engine/reference/commandline/ps/). You can also see stopped containers by adding the `-a` flag.

If any containers are running, then use `docker stop <ID>` to stop them before we continue on.

### Adding Guzzle

While you can get data from MetaWeather's API using [PHP's built in cURL adapter](http://php.net/manual/en/book.curl.php), I prefer [Guzzle](http://docs.guzzlephp.org/en/stable/) as it takes less work to make API calls. Plus, this will give us an opportunity to install a new package using Composer. Open up the composer.json file that was automatically created when we set up Slim, and add a line to the require block for Guzzle:

{linenos=off, lang=json}
~~~~~~~
{
    "require": {
        "slim/slim": "^3.0",
        "guzzlehttp/guzzle": "~6.0"
    }
}
~~~~~~~

Now in your terminal, run:

{linenos=off, lang=sh}
~~~~~~~
$ docker run --rm -v $(pwd):/app composer:latest update
~~~~~~~

The container will install the new packages and update your composer.lock file. During this process, you should see some output like this in the terminal:

{linenos=off, lang=sh}
~~~~~~~
Loading composer repositories with package information
Updating dependencies (including require-dev)
Package operations: 3 installs, 0 updates, 0 removals
  - Installing guzzlehttp/promises (v1.3.1): Downloading (100%)
  - Installing guzzlehttp/psr7 (1.4.2): Downloading (100%)
  - Installing guzzlehttp/guzzle (6.3.0): Downloading (100%)
guzzlehttp/guzzle suggests installing psr/log (Required for using the Log middleware)
Writing lock file
Generating autoload files
~~~~~~~

Now Guzzle is installed, and ready to be used.

### Calling the MetaWeather API

The endpoint we'll be using takes a `woeid` ("[Where On Earth ID](https://developer.yahoo.com/geo/geoplanet/guide/concepts.html)") and returns the weather forecast in [JSON](https://www.json.org/) format. Initially, our application will just pass requests through to MetaWeather and return the same JSON data that they do. We'll add database caching in the next chapter. Here's our updated `index.php` file:

{title="index.php", linenos=off, lang=php}
~~~~~~~
<?php

require 'vendor/autoload.php';

// Create a new Container with Guzzle injected
$container = new \Slim\Container([
    'http' => function () {
        return new GuzzleHttp\Client();
    }
]);

// Instantiate the App object
$app = new \Slim\App($container);

// Get weather by location ID
$app->get('/locations/{id}', function ($request, $response, $args) {

    // Get the weather from MetaWeather
    $result = $this->http->get("https://www.metaweather.com/api/location/{$args['id']}")
        ->getBody()
        ->getContents();

    // Return the results as JSON
    return $response->withStatus(200)->withJson(json_decode($result));
});
// The delete endpoint is unchanged
$app->delete('/locations/{id}', function ($request, $response, $args) {
    return $response->withStatus(200)->write("Location {$args['id']} deleted.");
});

// Run the application
$app->run();
~~~~~~~

### What's going on here?

We've made some updates - mostly to the get('/locations/{id}', ...) endpoint. Let's take a look at what's new here:

* `$container = new \Slim\Container([...]);` - If you're not familiar with Slim's Dependency Injection system, you can [read up on it here](https://www.slimframework.com/docs/concepts/di.html). Since this book isn't focused on dependency injection, I'll just say that this injects Guzzle into the application so we can use it in any of our routes by calling $this->http. It makes our code more concise, and allows us to mock the library if we add tests to the application later.

* `$result = $this->http->get("https://www.metaweather.com/api/location/{$args['id']}")...` - This is where we make the call to MetaWeather through Guzzle. As you can see, we're simply passing the locationId from our Slim route into the API endpoint provided by MetaWeather. We also call getBody() and getContents() since we just want to get the response as a string rather than a [stream](http://docs.guzzlephp.org/en/stable/psr7.html#streams).

* `return $response->withStatus(200)->withJson(json_decode($result));` - Finally, we respond with a 200 response code and withJson(...). Slim provides this method to automatically set JSON response headers, but because the response from MetaWeather came back as a JSON string, we have to run json_decode on it before re-encoding it into JSON.

### Testing it out

Our application is ready to pass a real location ID to MetaWeather's API and return some data. First, find a location ID using [this WOEID lookup tool](http://woeid.rosselliot.co.nz/). I'm using my home town of Chicago's ID for testing: `2379574`.

Now, let's run the Docker container again:

{linenos=off, lang=sh}
~~~~~~~
$ docker run -d --rm --name=weather-app -p 38000:80 -v $(pwd):/var/www/html php:apache
~~~~~~~

And visit the running application in your browser at <http://localhost:38000/index.php/locations/2379574>. You should receive a response in JSON that looks something like this:

{linenos=off, lang=json}
~~~~~~~
{
  "consolidated_weather": [
    {
      "id": 5560109107249152,
      "weather_state_name": "Heavy Rain",
      "weather_state_abbr": "hr",
      "wind_direction_compass": "SSW",
      "created": "2017-08-21T17:23:20.408640Z",
      "applicable_date": "2017-08-21",
      "min_temp": 22.711666666666662,
      "max_temp": 29.093333333333334,
      "the_temp": 28.166666666666668,
      "wind_speed": 5.319549422227524,
      "wind_direction": 201.1261629987385,
      "air_pressure": 1012.975,
      "humidity": 74,
      "visibility": 14.841140240992603,
      "predictability": 77
    },
    ...
  ],
  "time": "2017-08-21T14:31:16.860660-05:00",
  "sun_rise": "2017-08-21T06:05:09.010298-05:00",
  "sun_set": "2017-08-21T19:42:46.959306-05:00",
  "timezone_name": "LMT",
  "parent": {
    "title": "Illinois",
    "location_type": "Region / State / Province",
    "woeid": 2347572,
    "latt_long": ""
  },
  "sources": [
    {
      "title": "BBC",
      "slug": "bbc",
      "url": "http://www.bbc.co.uk/weather/",
      "crawl_rate": 180
    },
    ...
  ],
  "title": "Chicago",
  "location_type": "City",
  "woeid": 2379574,
  "latt_long": "41.884151,-87.632408",
  "timezone": "US/Central"
}
~~~~~~~

Your Dockerized PHP application is now returning real data from an external data source and being served in Apache, but you'll probably notice that it's not exactly the speediest application in the world (mine clocked in at 2.9 seconds page load time!).

MetaWeather is a free service, and not intended to be blazingly fast around the world. In order to fix that, we're going to save responses from MetaWeather in our own MySQL database and serve those saved responses up when we can. This will greatly improve performance, but will also show us how to add a database to a PHP application with Docker.