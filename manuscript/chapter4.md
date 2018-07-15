# Chapter 4. Connecting to the Database

Before we can connect a database, we need to make sure that our PHP container has all the necessary extensions installed. By default, the PHP Docker image from Docker Hub is pretty lightweight, so it doesn't include many PHP extensions or Linux packages you might need. This tradeoff between smaller and more flexible images is one you'll have to make based on your project priorities, but since we know we'll need MySQL for this project, let's extend the base PHP image with the required extensions.

> The base PHP images are usually a good starting point, but I've also started keeping some PHP images with more common extensions enabled in Github. Check out [this repository for more](https://github.com/shiphp/dockerfiles).

## Creating a Custom Dockerfile

In the root of your `weather-app` project, next to the `index.php` file, create a new file called simply `Dockerfile`. Dockerfiles have no extension, and are used by Docker to configure and set up images. All the images on [Docker Hub](https://hub.docker.com/) have a corresponding Dockerfile, and usually you can find them on Github. [PHP's Dockerfiles are here](https://github.com/docker-library/php), but they may be a little confusing since they build on one-another. Our Dockerfile will be much simpler:

#### Dockerfile

{title="Dockerfile", linenos=off, lang=sh}
~~~~~~~
FROM php:apache

RUN docker-php-source extract && docker-php-ext-install mysqli && docker-php-source delete
~~~~~~~

### What's going on here?

This Dockerfile has two lines:

* `FROM php:apache` - This tells Docker where to start when building the image. You can start with just a Linux distribution (like [Ubuntu](https://hub.docker.com/_/ubuntu/) or the super light [Alpine](https://hub.docker.com/_/alpine/)), or you can extend a more specific container as we did in this case. Here we're extending the `php:apache` image we used to run our container before.

* `RUN docker-php-source extract...` - This is the line that adds the PHP extensions we need. In our case, we just want to use [mysqli functions](http://php.net/manual/en/book.mysqli.php) for our database. Since this is such a common extension, the PHP image has methods for adding it automatically when we extend the base image with these lines.

> Check out [the official Docker documentation](https://docs.docker.com/engine/reference/builder/) for more options for extending existing images.

## Building the Image

In order to create an image, we will build it from the Dockerfile we created above:

{linenos=off, lang=sh}
~~~~~~~
$ docker build . -t shiphp/weather-app
~~~~~~~

When you run this command, you will see a bunch of output as Docker builds your image, ending with something like:

{linenos=off, lang=sh}
~~~~~~~
Removing intermediate container 80b3f79714b2
Successfully built 8dc1f8c175a1
Successfully tagged shiphp/weather-app:latest
~~~~~~~

### What's going on here?

The [Docker build](https://docs.docker.com/engine/reference/commandline/build/) command reads your Dockerfile and creates an image that can then be used to run a container. In this case, we're passing two parameters to the command:

* `.` - The dot lets Docker know the "context" of our build - in this case, it's the current directory. You can also use an absolute path like `/Users/username/weather-app` if you know the exact path of your Dockerfile. Docker automatically builds this image in the context of the directory containing your Dockerfile, so make sure your Dockerfile is located at the root of your project.

* `-t shiphp/weather-app` - The -t flag sets a "tag" for your image. This tag will allow you to more easily create a container from the image or push the image to a registry to share it with others.

You can see a list of all the Docker images on your local machine by running `$ docker images`. When you run this command, you should see something like this in your terminal:

{linenos=off, lang=sh}
~~~~~~~
REPOSITORY         TAG    IMAGE ID     CREATED        SIZE
shiphp/weather-app latest 8dc1f8c175a1 27 minutes ago 391MB
~~~~~~~

## Running a MySQL Container

In the introduction, I mentioned that Docker containers could be "linked" through [Docker's network feature](https://docs.docker.com/engine/userguide/networking/). In order to have our PHP application get data from our database, we need to link it to an active database container. Let's start a new MySQL container so we can link our web application to it:

{linenos=off, lang=sh}
~~~~~~~
$ docker run -d --rm --name weather-db -e MYSQL_USER=admin -e MYSQL_DATABASE=weather -e MYSQL_PASSWORD=p23l%v11p -e MYSQL_RANDOM_ROOT_PASSWORD=true mysql:5.7
~~~~~~~

### What's going on here?

When you run the above docker run command, Docker will get the latest version of the [MySQL 5.7 image](https://hub.docker.com/_/mysql/) and start a container named weather-db. Like PHP, MySQL has [listed their Docker images on Docker Hub](https://hub.docker.com/_/mysql/), and you can use them in much the same way. Some of the flags above are new though, so let's take a closer look:

* `-d` - This runs the container in "detached" mode, meaning that you won't have to keep your terminal connected to the container.

* `--name weather-db` - Naming our database container is important because it's easier to connect to a named container. We can also use this name to connect to the MySQL database from within our PHP application as we'll see later.

* `-e MYSQL_USER=admin` - The [MySQL image](https://hub.docker.com/_/mysql/) includes several -e (environmental variable) options. This first one sets the username for the MySQL user we'll use in our PHP application. More about the environmental options is available on [the official Docker Hub image](https://hub.docker.com/_/mysql/).

* `-e MYSQL_DATABASE=weather` - By default, the container will not create a database. While you can create one manually by logging into the container (which will be covered in the next section), it's easier to create the database up front like this.

* `-e MYSQL_PASSWORD=p23l%v11p` - This sets our database password to something besides the default. Even though it isn't a huge security risk as we're just doing local development, you should set this to your own strong password.

* `-e MYSQL_RANDOM_ROOT_PASSWORD=true` - Your root user password should be set to something random as you're not going to be logging in with the root user anyway.

* `mysql:5.7` As with the PHP image, you can choose a version of MySQL to use. Here I've elected to use the version 5.7.

As with the `php:apache` image, the default image command is fine, so we don't need to add anything after the image.

## Logging into a Running Container

At this point, we want to check that the database in our new container is working, but how can we access a running Docker container? We will use the `docker exec` command to enter the terminal for our new container, and then once we're inside, we'll use the [MySQL command line interface](https://dev.mysql.com/doc/refman/5.7/en/mysql.html) to access the database we created.

We can enter a bash session on the running database container using:

{linenos=off, lang=sh}
~~~~~~~
$ docker exec -it weather-db bash
~~~~~~~

You should see a line like `root@35c7ad5a574d:/#` (with your container's ID) on your terminal, indicating that you're now logged in to the running container. From within the container (not on your host machine), run:

{linenos=off, lang=sh}
~~~~~~~
$ mysql --user=admin --password
Enter password:
~~~~~~~

Enter the password you chose for the MySQL container you created above (in this example it was p23l%v11p), and hit the "return" key. You'll see some information about MySQL like this:

{linenos=off, lang=sh}
~~~~~~~
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 10
Server version: 5.7.19 MySQL Community Server (GPL)

Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
~~~~~~~

Finally, let's just check that our database was created using the `show databases` command:

{linenos=off, lang=sh}
~~~~~~~
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| weather            |
+--------------------+
2 rows in set (0.01 sec)
~~~~~~~

### What's going on here?

In the section above, we logged into a terminal on the running database container, we then logged into MySQL via the command line interface, and finally, we took a look at the databases available to us. Since you may not be familiar with all these commands, we'll review them one-by-one.

#### Docker Exec

* `docker exec` - This is the [Docker command for executing commands](https://docs.docker.com/engine/reference/commandline/exec/) on a running container. Like Docker's `run` command, there are many options for the `exec` command, but we'll just cover the highlights for this example.

* `-it` - The `-it` flag is actually two flags which are commonly used together. The `-i` flag runs the session in "interactive" mode, meaning the STDIN and STDOUT from your terminal are attached to the container's terminal. The `-t` flag attaches a pseudoterminal to the connection. There's a little more explained in [this Stack Overflow question](https://stackoverflow.com/a/22287905/977192), but suffice it to say, when running `docker exec` bash commands, you will probably want to use the `-it` flags.

* `weather-db` - This is the name (you could also use the ID) of the running container we want to run our command on.

* `bash` - Finally, this is the command that we want to run in the container. [bash](https://en.wikipedia.org/wiki/Bash_(Unix_shell)) is a shell installed in most Linux distributions that allows us to run other programs on the container. If your container doesn't have bash installed, you can [try *sh* instead](http://pubs.opengroup.org/onlinepubs/009695399/utilities/sh.html).

#### MySQL CLI

The [MySQL CLI](https://dev.mysql.com/doc/refman/5.7/en/mysql.html) also provides many options, but here are the three we used:

* `mysql` - This is the command used to access MySQL from a terminal. If you've been using MySQL on a VM or your local machine, this is no different.

* `--user=admin` - We specify the username here so that MySQL knows we want access as a specific user and not `root`.

* `--password` - The password flag instructs MySQL to give us the password prompt. You can also enter the password directly into the command, but it's generally less secure.

#### Exiting MySQL and the Container

We're done with the MySQL container for now, so let’s exit and stop the container:

* To exit the MySQL CLI, type `\q` and then press "return".

* Exit the container by typing `exit` and then pressing "return".

* Stop the container by typing `docker stop weather-db` and then "return" again.

The whole command line output should look something like this when you're done:

{linenos=off, lang=sh}
~~~~~~~
mysql> \q
Bye
root@b01aff352dd7:/# exit
exit
$ docker stop weather-db
weather-db
~~~~~~~

We have now proven that our MySQL container runs and that we can log in to access the database, but how will we keep data saved in the container? What happens when we stop this container? Persistence is important in real-world applications, so in the next section, we'll dive into keeping our data even after our container is long gone.

## Retaining Data in Our Database Container

So far, our database container starts up and automatically creates a database and user. This is fine, but when we connect this database to our PHP application, we'll want to make sure that the database tables and values are saved even after our container stops. If not, we'll have to rebuild the database every time our database container restarts.

The best way to save data from containers for local development is by mounting a volume. We saw how effective this was for local development of our PHP code, and the process we follow for mounting our database's data is very similar. When working in a PHP project, I like to mount database volumes from the project directory (in our case, the weather-app/ directory we created earlier), but you can mount the volume from anywhere on your host system.

> It's important to note that when mounting volumes from a Mac host computer, there are [performance tradeoffs](https://docs.docker.com/docker-for-mac/osxfs-caching/). These should not be an issue in Linux environments, but I've found that when running mounted volumes in many linked containers, things can get painfully slow. Docker provides [some hints for tuning here](https://docs.docker.com/docker-for-mac/osxfs-caching/).

To start the MySQL container with the database saved to our host system, navigate to your new `weather-app` project directory and type:

{linenos=off, lang=sh}
~~~~~~~
$ docker run -d --rm --name weather-db -e MYSQL_USER=admin -e MYSQL_DATABASE=weather -e MYSQL_PASSWORD=p23l%v11p -e MYSQL_RANDOM_ROOT_PASSWORD=true -v $(pwd)/.data:/var/lib/mysql mysql:5.7
~~~~~~~

### What's going on here?

The only thing we added to this docker run command was -v $(pwd)/.data:/var/lib/mysql. This mounts a volume from our local directory into the MySQL container. A new directory will appear (.data/) and as the database container starts up, folders and files will appear. These are the files that MySQL uses to store your data, and you can view the files being created from your terminal:

{linenos=off, lang=sh}
~~~~~~~
$ ls -a .data/
~~~~~~~

You should see files and folders listed out like this:

{linenos=off, lang=text}
~~~~~~~
.                   ib_buffer_pool      private_key.pem
..                  ib_logfile0         public_key.pem
auto.cnf            ib_logfile1         server-cert.pem
ca-key.pem          ibdata1             server-key.pem
ca.pem              ibtmp1              sys
client-cert.pem     mysql               weather
client-key.pem      performance_schema
~~~~~~~

Don't worry about what each of those mean (although [you can read up on MySQL internals if you're interested](https://dev.mysql.com/doc/internals/en/)). At this point, we just care that MySQL is now using data from our host machine in the container, allowing us to keep the database data even after the container is shut down.

> Be sure to add the `.data` directory to your .gitignore file so that your database isn't added to version control.

## Creating a Database Table

Since this application just needs to cache the results from the MetaWeather API, the database will just have one table with three columns. We'll call the table `locations` and add columns `id`, `weather`, and `last_updated`.

{width=100%}
![Diagram 3: Weather Application Database](images/diagram3.png)

If the MySQL container is not running, then start it using the method above that mounts data in a volume:

{linenos=off, lang=sh}
~~~~~~~
$ docker run -d --rm --name weather-db -e MYSQL_USER=admin -e MYSQL_DATABASE=weather -e MYSQL_PASSWORD=p23l%v11p -e MYSQL_RANDOM_ROOT_PASSWORD=true -v $(pwd)/.data:/var/lib/mysql mysql:5.7
~~~~~~~

Log into the container and into the MySQL CLI. This time, we're going to do it in just one step:

{linenos=off, lang=sh}
~~~~~~~
$ docker exec -it weather-db mysql --user=admin --password=p23l%v11p weather
~~~~~~~

Next, we're going to run a SQL command to create the database table we outlined above:

{linenos=off, lang=sh}
~~~~~~~
mysql> CREATE TABLE locations (id VARCHAR(64) NOT NULL, weather JSON NULL, last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP);
~~~~~~~

It should tell you `Query OK`, but you can check by running `mysql> SHOW TABLES;`. Exit the container and MySQL CLI by typing `\q`.

### What's going on here?

Unlike the last time we logged in to the MySQL command line, this time we cut the process down to just one step. Remember that when running `docker run` or `docker exec`, the last part is the command we want to run in the container. This means that we don't actually have to log into a bash session and then log into MySQL, we can just directly log in to the database CLI.

This can be done with any command available on the container, so `docker exec` becomes very useful for seeing what's actually going on inside your containers.

## Saving to the Database from our PHP Application

Now that our database is ready to store weather data and it will keep that data even after the container is removed, we need to update our PHP application to connect to and store weather results in the database.

Initially, the `index.php` file was getting data directly from the MetaWeather API, but now that we have a storage mechanism in place (ie: our database), let's get the application to save data to the database for repeat requests.

{title="index.php", linenos=off, lang=php}
~~~~~~~
<?php

require 'vendor/autoload.php';

// Create a new Container
$container = new \Slim\Container([
    // Add Guzzle as 'http'
    'http' => function () {
        return new GuzzleHttp\Client();
    },
    // Add mysqli as 'mysql'
    'mysql' => function () {
        $mysqli = new mysqli(
            "weather-db",
            "admin",
            "p23l%v11p",
            "weather"
        );
        if ($mysqli->connect_errno) {
            echo "Failed to connect to MySQL: " . $mysqli->connect_error;
            exit;
        } else {
            return $mysqli;
        }
    },
]);
// Instantiate the App object
$app = new \Slim\App($container);

// Get weather by location ID
$app->get('/locations/{id}', function ($request, $response, $args) {

    // Get the location from the database
    $id = $this->mysql->real_escape_string($args['id']);
    $results = $this->mysql->query("SELECT * FROM locations WHERE id='{$id}'");

    // If location found, then show it from the DB, otherwise, query MetaWeather
    if ($results->num_rows > 0) {
        $result = $results->fetch_assoc()['weather'];
    } else {
        $result = $this->http->get("https://www.metaweather.com/api/location/{$id}")
            ->getBody()
            ->getContents();
        $cleanResult = $this->mysql->real_escape_string($result);
        if (!$this->mysql->query("INSERT into locations (id, weather) VALUES ('{$id}', '{$cleanResult}')")) {
            throw new Exception("Location could not be updated.");
        }
    }

    // Return the results as JSON
    return $response->withStatus(200)->withJson(json_decode($result));
});

// Run the application
$app->run();
~~~~~~~

### What's going on here?

We've added two important things to our `index.php` file above. First, we updated the `$container` variable:

{title="index.php", linenos=off, lang=php}
~~~~~~~
$container = new \Slim\Container([
    // Add Guzzle as 'http'
    'http' => function () {
        return new GuzzleHttp\Client();
    },
    // Add mysqli as 'mysql'
    'mysql' => function () {
        $mysqli = new mysqli(
            "weather-db",   // The database host
            "admin",        // The database user
            "p23l%v11p",    // The database password
            "weather"       // The database name
        );
        if ($mysqli->connect_errno) {
            echo "Failed to connect to MySQL: " . $mysqli->connect_error;
            exit;
        } else {
            return $mysqli;
        }
    },
]);
~~~~~~~

Guzzle is the same, but we've also added an element to the array named `'mysql'`. This callback instantiates [MySQLi](http://php.net/manual/en/book.mysqli.php) with hard-coded database credentials.

> Hard-coding your database credentials like this is not a good idea in a real application. We'll cover a more secure way to include credentials via environmental variables in Docker in the last section of this chapter. In the meantime, don't commit this to a public repository unless you plan on changing your database password.

The other major update to the file is the `GET /locations` endpoint:

{title="index.php", linenos=off, lang=php}
~~~~~~~
$app->get('/locations/{id}', function ($request, $response, $args) {

    // Get the location from the database
    $id = $this->mysql->real_escape_string($args['id']);
    $results = $this->mysql->query("SELECT * FROM locations WHERE id='{$id}'");

    // If location found, show it from DB, otherwise, query MetaWeather
    if ($results->num_rows > 0) {
        $result = $results->fetch_assoc()['weather'];
    } else {
        $result = $this->http->get("https://www.metaweather.com/api/location/{$id}")
            ->getBody()
            ->getContents();
        $cleanResult = $this->mysql->real_escape_string($result);
        if (!$this->mysql->query("INSERT into locations (id, weather) VALUES ('{$id}', '{$cleanResult}')")) {
            throw new Exception("Location could not be updated.");
        }
    }

    // Return the results as JSON
    return $response->withStatus(200)->withJson(json_decode($result));
});
~~~~~~~

While the comments in the code may help, it's worth adding some more detail in case you're not familiar with MySQLi:

* `$id = $this->mysql->real_escape_string($args['id']);` - First, we escape the `id` passed in through the endpoint's `$args` variable. This is a good idea anytime you're taking user or third-party input as they might accidentally (or intentionally) inject SQL statements that break your code.

* `$results = $this->mysql->query("SELECT * FROM locations WHERE id='{$id}'");` - While I would typically [use an ORM](https://stackoverflow.com/a/1279678) for a real application, this simple two-endpoint example didn't seem to warrant it. Instead, we're using [MySQLi - a PHP extension for accessing MySQL -](http://php.net/manual/en/book.mysqli.php) to query the database. This allows us to write raw SQL with PHP variables embedded.

* `if ($results->num_rows > 0) {...}` - If we find at least one result for this location ID, we should return the result from the database. If not, we'll continue on...

* `else { ... $this->mysql->query("INSERT into locations (id, weather) VALUES ('{$id}', '{$cleanResult}')") ...` - If we did not find a result for this location ID in the database, we're getting it from the MetaWeather API, escaping the JSON string, and then inserting the result into the database.

* `return $response->withStatus(200)->withJson(json_decode($result));` - Just as we did before, we're returning any result as JSON to the user.

## Linking the PHP Container

With the code in place, now we need to run a new PHP container linked to the MySQL database container we just started:

{linenos=off, lang=sh}
~~~~~~~
$ docker run -d --rm --name=weather-app -p 38000:80 -v $(pwd):/var/www/html --link weather-db shiphp/weather-app
~~~~~~~

### What's going on here?

There are two new parts to this docker run command:

* `--link weather-db` - This links the container named `weather-db` to this container. You can also [give the linked container an alias](https://docs.docker.com/engine/reference/run/#expose-incoming-ports) for use within the PHP container, but we're not going to need to do that here.

* `shiphp/weather-app` - Instead of using the `php:apache` image we used in the previous chapter, we're using the new Docker image we built at the beginning of this chapter. The reason for this is that we need the PHP MySQLi extension that we added in our custom Dockerfile.

Now that the PHP container is up and running, you should be able to navigate to a location ID like the following:

`GET http://localhost:38000/index.php/locations/2487956`

The first time you load this URL, it will probably take a second or two to load, but if you refresh the page, you should see a dramatically faster result. Mine loaded in less than 150 ms! That's thanks to the caching we just set up by saving the result in the database.

## Deleting Data

In order to remove data from the database, we'll add a second endpoint. Add the following just after the `$app->get(...);` endpoint to allow users to delete a location from the database:

#### index.php (partial)

{title="index.php", linenos=off, lang=php}
~~~~~~~
$app->delete('/locations/{id}', function ($request, $response, $args) {

    // Get the location from the database
    $id = $this->mysql->real_escape_string($args['id']);
    $results = $this->mysql->query("SELECT * FROM locations WHERE id='{$id}'");

    // If it exists, delete it, otherwise send a 404
    if (
        $results->num_rows > 0 &&
        $this->mysql->query("DELETE FROM locations WHERE id='{$id}'")
    ) {
        return $response->withStatus(200)->write("Location {$args['id']} deleted.");
    } else {
        return $response->withStatus(404)->write("Location {$args['id']} not found.");
    }
});

// Run the application
$app->run();
~~~~~~~

Now you can use [cURL](https://www.garron.me/en/bits/curl-delete-request.html) or [Postman](https://www.getpostman.com/) to make a `DELETE` call to the same location endpoint you did above. For example:

`DELETE http://localhost:38000/index.php/locations/2487956`

You should see a message `Location 2487956` deleted. and now when you make a `GET` request to that URL again, you'll wait longer as the result must come from the MetaWeather API.

## Managing Environmental Variables

The last topic we'll cover in this chapter is managing environmental variables in Docker. Docker allows you to specify environmental variables at runtime, so that your PHP code can access these variables, but you won't have to hard-code them as we did above.

First, we'll need to change the code that specifies our database credentials in the `index.php` file:

{title="index.php", linenos=off, lang=php}
~~~~~~~
// Create a new Container
$container = new \Slim\Container([
    // Add Guzzle as 'http'
    'http' => function () {
        return new GuzzleHttp\Client();
    },
    // Add mysqli as 'mysql'
    'mysql' => function () {
        $mysqli = new mysqli(
            getenv('DATABASE_HOST'),
            getenv('DATABASE_USER'),
            getenv('DATABASE_PASSWORD'),
            getenv('DATABASE_NAME')
        );
        if ($mysqli->connect_errno) {
            echo "Failed to connect to MySQL: " . $mysqli->connect_error;
            exit;
        } else {
            return $mysqli;
        }
    },
]);
~~~~~~~

Stop the PHP container, and re-run it using this command:

{linenos=off, lang=sh}
~~~~~~~
$ docker run -d --rm --name=weather-app -p 38000:80 -v $(pwd):/var/www/html --link weather-db -e DATABASE_HOST='weather-db' -e DATABASE_USER='admin' -e DATABASE_PASSWORD='p23l%v11p' -e DATABASE_NAME='weather' shiphp/weather-app
~~~~~~~

Now you can safely share your PHP code without revealing your database login credentials.

### What's going on here?

PHP can read environmental variables from the operating system [using the getenv function](http://php.net/manual/en/function.getenv.php), and when we use this function in a container, the effect is the same. Instead of reading environmental variables from the host operating system (ie: your computer), it read environmental variables from the Docker container running the code. This helps with security - different containers aren't sharing environmental variables - and it helps make sharing code easier.

The updated `docker run` command now includes four -e flags that translate to environmental variables in the container. Now, your PHP application will read the environmental variables you set when you run the container instead of hard-coded values or values from your host machine.

> If you don't want to have to type long commands like this in every time you run your container, you can use Docker Compose and/or an .env file instead. This is covered in [this blog post](https://www.shiphp.com/blog/2017/env-php-docker).

Finally, you can see the environmental variables your container is using:

{linenos=off, lang=sh}
~~~~~~~
$ docker inspect weather-app</td>
~~~~~~~

The JSON returned from this command gives you a lot of information about a running container, but for this example, we just care about the array at `Config.Env`:

{linenos=off, lang=php}
~~~~~~~
[
    "DATABASE_HOST=weather-app",
    "DATABASE_USER=admin",
    "DATABASE_PASSWORD=p23l%v11p",
    "DATABASE_NAME=weather",
    // ...
],

//...
~~~~~~~

If everything worked correctly, you should now be able to access the application as before. In the next chapter, we'll take a quick look at how we might improve this application using advanced Docker topics. If you want to take a look at the complete source code for this app, it's [available on Github](https://github.com/shiphp/weather-app).