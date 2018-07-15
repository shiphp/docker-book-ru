# Chapter 5. Next Steps

At this point, our application is working and we can retrieve weather results from the API or from our database cache, but there's still a lot more to learn. Building one demo application won't make you an expert with Docker, so let's take a look at some things we might do to continue to improve this application and learn more about Docker.

## Further Development

### Clean up long Docker commands

Right now, starting and stopping our container is an arduous process and quite prone to error. I'm guessing you don't want to type `docker run -d --rm --name=weather-app -p 38000:80 -v $(pwd):/var/www/html --link weather-db -e DATABASE_HOST='weather-db' -e DATABASE_USER='admin' -e DATABASE_PASSWORD='p23l%v11p' -e DATABASE_NAME='weather' shiphp/weather-app` every time you want to start editing your code, but you can pretty easily avoid this by writing [npm scripts](https://docs.npmjs.com/cli/run-script), [bash commands](http://tldp.org/LDP/abs/html/), or PHP scripts.

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