<!--[metadata]>
+++
title = "Getting Started"
description = "Getting started with Docker Compose"
keywords = ["documentation, docs,  docker, compose, orchestration, containers"]
[menu.main]
parent="smn_workw_compose"
weight=3
+++
<![end-metadata]-->


## Getting Started

Let's get started with a walkthrough of getting a simple Python web app running
on Compose. It assumes a little knowledge of Python, but the concepts
demonstrated here should be understandable even if you're not familiar with
Python.

### Installation and set-up

First, [install Docker and Compose](install.md).

Next, you'll want to make a directory for the project:

    $ mkdir composetest
    $ cd composetest

Inside this directory, create `app.py`, a simple Python web app that uses the Flask
framework and increments a value in Redis. Don't worry if you don't have Redis installed, docker is going to take care of that for you when we [define services](#define-services):

    from flask import Flask
    from redis import Redis

    app = Flask(__name__)
    redis = Redis(host='redis', port=6379)

    @app.route('/')
    def hello():
        redis.incr('hits')
        return 'Hello World! I have been seen %s times.' % redis.get('hits')

    if __name__ == "__main__":
        app.run(host="0.0.0.0", debug=True)

Next, define the Python dependencies in a file called `requirements.txt`:

    flask
    redis

### Create a Docker image

Now, create a Docker image containing all of your app's dependencies. You
specify how to build the image using a file called
[`Dockerfile`](http://docs.docker.com/reference/builder/):

    FROM python:2.7
    ADD . /code
    WORKDIR /code
    RUN pip install -r requirements.txt
    CMD python app.py

This tells Docker to:

* Build an image starting with the Python 2.7 image.
* Add the current directory `.` into the path `/code` in the image.
* Set the working directory to `/code`.
* Install the Python dependencies.
* Set the default command for the container to `python app.py`

For more information on how to write Dockerfiles, see the [Docker user guide](https://docs.docker.com/userguide/dockerimages/#building-an-image-from-a-dockerfile) and the [Dockerfile reference](http://docs.docker.com/reference/builder/).

You can build the image by running `docker build -t web .`.

### Define services

Next, define a set of services using `docker-compose.yml`:

    web:
      build: .
      ports:
       - "5000:5000"
      volumes:
       - .:/code
      links:
       - redis
    redis:
      image: redis

This template defines two services, `web` and `redis`. The `web` service:

* Builds from the `Dockerfile` in the current directory.
* Forwards the exposed port 5000 on the container to port 5000 on the host machine.
* Mounts the current directory on the host to `/code` inside the container allowing you to modify the code without having to rebuild the image.
* Links the web container to the Redis service.

The `redis` service uses the latest public [Redis](https://registry.hub.docker.com/_/redis/) image pulled from the Docker Hub registry.

### Build and run your app with Compose

Now, when you run `docker-compose up`, Compose will pull a Redis image, build an image for your code, and start everything up:

    $ docker-compose up
    Pulling image redis...
    Building web...
    Starting composetest_redis_1...
    Starting composetest_web_1...
    redis_1 | [8] 02 Jan 18:43:35.576 # Server started, Redis version 2.8.3
    web_1   |  * Running on http://0.0.0.0:5000/
    web_1   |  * Restarting with stat

If you're using [Docker Machine](https://docs.docker.com/machine), then `docker-machine ip MACHINE_VM` will tell you its address and you can open `http://MACHINE_VM_IP:5000` in a browser.

If you're using Docker on Linux natively, then the web app should now be listening on port 5000 on your Docker daemon host. If `http://0.0.0.0:5000` doesn't resolve, you can also try `http://localhost:5000`.

You should get a message in your browser saying:

`Hello World! I have been seen 1 times.`

Refreshing the page will increment the number.

If you want to run your services in the background, you can pass the `-d` flag
(for "detached" mode) to `docker-compose up` and use `docker-compose ps` to
see what is currently running:

    $ docker-compose up -d
    Starting composetest_redis_1...
    Starting composetest_web_1...
    $ docker-compose ps
	    Name                 Command            State       Ports
    -------------------------------------------------------------------
    composetest_redis_1   /usr/local/bin/run         Up
    composetest_web_1     /bin/sh -c python app.py   Up      5000->5000/tcp

The `docker-compose run` command allows you to run one-off commands for your
services. For example, to see what environment variables are available to the
`web` service:

    $ docker-compose run web env

See `docker-compose --help` to see other available commands. You can also install [command completion](completion.md) for the bash and zsh shell, which will also show you available commands.

If you started Compose with `docker-compose up -d`, you'll probably want to stop
your services once you've finished with them:

    $ docker-compose stop

At this point, you have seen the basics of how Compose works.

- Next, try the quick start guide for [Django](django.md),
  [Rails](rails.md), or [WordPress](wordpress.md).
- See the reference guides for complete details on the [commands](./reference/index.md), the
  [configuration file](compose-file.md) and [environment variables](env.md).

## More Compose documentation

- [User guide](/)
- [Installing Compose](install.md)
- [Get started with Django](django.md)
- [Get started with Rails](rails.md)
- [Get started with WordPress](wordpress.md)
- [Command line reference](./reference/index.md)
- [Compose file reference](compose-file.md)