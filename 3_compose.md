<a href="Readme.md#table-of-contents" class="top" id="preface">Top</a>
<a id="compose"></a>
# 3. Docker Compose

Make sure you have already installed both Docker Engine and the [Docker
Compose Plugin](https://docs.docker.com/compose/install/). You don't need to install Python or Redis, as both are provided by Docker images.

<a id="setup"></a>
## 3.1 Step 1: Setup

Define the application dependencies.

1.  Create a directory for the project:
```
$ mkdir composetest
$ cd composetest
```

2.  Create a file called `app.py` in your project directory and paste this in:
```
import time

import redis
from flask import Flask

app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)


def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)


@app.route('/')
def hello():
    count = get_hit_count()
    return 'Hello World! I have been seen {} times.\n'.format(count)
```

In this example, `redis` is the hostname of the redis container on the application's network. We use the default port for Redis, `6379`.


3.  Create another file called `requirements.txt` in your project directory and
    paste this in:

        flask
        redis

<a id="dockerfile"></a>
## 3.2 Step 2: Create a Dockerfile

In this step, you write a Dockerfile that builds a Docker image. The image
contains all the dependencies the Python application requires, including Python
itself.

In your project directory, create a file named `Dockerfile` and paste the
following:
```
FROM python:alpine
WORKDIR /code
ENV FLASK_APP app.py
ENV FLASK_RUN_HOST 0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
RUN python3 -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
COPY . .
CMD ["flask", "run"]
```

This tells Docker to:

* Build an image starting with the Python 3.7 image.
* Set the working directory to `/code`.
* Set environment variables used by the `flask` command.
* Install gcc so Python packages such as MarkupSafe and SQLAlchemy can compile speedups.
* Copy `requirements.txt` and install the Python dependencies.
* Copy the current directory `.` in the project to the workdir `.` in the image.
* Set the default command for the container to `flask run`.

<a id="composefile"></a>
## 3.3 Step 3: Define services in a Compose file

Create a file called `docker-compose.yml` in your project directory and paste
the following:
```
services:
  web:
    build: .
    ports:
      - "8080:5000"
  redis:
    image: "redis:alpine"
```

This Compose file defines two services: `web` and `redis`.

### Web service

The `web` service uses an image that's built from the `Dockerfile` in the current directory.
It then binds the container and the host machine port `8080` to the exposed port, `5000`. This example service uses the default port for
the Flask web server, `5000`.

### Redis service

The `redis` service uses a public [Redis](https://registry.hub.docker.com/_/redis/)
image pulled from the Docker Hub registry.

<a id="ship"></a>
## 3.4 Step 4: Build and run your app with Compose

1.  From your project directory, start up your application by running `docker compose up`.

```
$ docker compose up
[+] Building 3.0s (13/13) FINISHED                              docker:default
 => [web internal] load build definition from Dockerfile                  0.0s
 => => transferring dockerfile: 336B                                      0.0s
 => [web internal] load metadata for docker.io/library/python:alpine      0.4s
 => [web internal] load .dockerignore                                     0.0s
 => => transferring context: 2B                                           0.0s
 => [web 1/7] FROM docker.io/library/python:alpine@sha256:ef097620baf127  0.0s
 => => resolve docker.io/library/python:alpine@sha256:ef097620baf1272e38  0.0s
 => [web internal] load build context                                     0.0s
 => => transferring context: 236B                                         0.0s
 => CACHED [web 2/7] WORKDIR /code                                        0.0s
 => CACHED [web 3/7] RUN apk add --no-cache gcc musl-dev linux-headers    0.0s
 => CACHED [web 4/7] RUN python3 -m venv /opt/venv                        0.0s
 => CACHED [web 5/7] COPY requirements.txt requirements.txt               0.0s
 => CACHED [web 6/7] RUN pip install -r requirements.txt                  0.0s
 => [web 7/7] COPY . .                                                    0.1s
 => [web] exporting to docker image format                                2.2s
 => => exporting layers                                                   0.1s
 => => exporting manifest sha256:aa0bbfa9cc3dce81e470577862e6e031169dd91  0.0s
 => => exporting config sha256:4d7a24c23cf9b1f3b0db2f5ffad3b62ddaad6fa0d  0.0s
 => => sending tarball                                                    2.0s
 => [web] importing to docker                                             1.9s
 => => loading layer 4c9c2b9681ab 32.77kB / 619.60kB                      1.9s
 => => loading layer 80fef791f8cf 163.84kB / 13.96MB                      1.7s
 => => loading layer 6c673d8c5e6c 239B / 239B                             1.4s
 => => loading layer d46b5001af7f 32.77kB / 2.70MB                        1.4s
 => => loading layer 6cb4186d171a 94B / 94B                               1.2s
 => => loading layer 2674bd776a73 557.06kB / 60.16MB                      1.1s
 => => loading layer 1d71fffd8c04 65.54kB / 6.51MB                        0.5s
 => => loading layer d0fe50a6413c 159B / 159B                             0.3s
 => => loading layer 269722ddb98d 65.54kB / 3.33MB                        0.3s
 => => loading layer 30544cda00e9 735B / 735B                             0.1s
[+] Running 3/3
 ✔ Network composetest_default    Create...                               0.1s
 ✔ Container composetest-web-1    Create...                              18.2s
 ✔ Container composetest-redis-1  Crea...                                18.2s
Attaching to redis-1, web-1
redis-1  | 1:C 03 May 2024 17:39:52.803 * oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
redis-1  | 1:C 03 May 2024 17:39:52.803 * Redis version=7.2.4, bits=64, commit=00000000, modified=0, pid=1, just started
redis-1  | 1:C 03 May 2024 17:39:52.803 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
redis-1  | 1:M 03 May 2024 17:39:52.803 * monotonic clock: POSIX clock_gettime
redis-1  | 1:M 03 May 2024 17:39:52.804 * Running mode=standalone, port=6379.
redis-1  | 1:M 03 May 2024 17:39:52.804 * Server initialized
redis-1  | 1:M 03 May 2024 17:39:52.804 * Ready to accept connections tcp
web-1    |  * Serving Flask app 'app.py'
web-1    |  * Debug mode: off
web-1    | WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
web-1    |  * Running on all addresses (0.0.0.0)
web-1    |  * Running on http://127.0.0.1:5000
web-1    |  * Running on http://172.19.0.3:5000
web-1    | Press CTRL+C to quit

```

    Compose pulls a Redis image, builds an image for your code, and starts the
    services you defined. In this case, the code is statically copied into the image at build time.

2.  Enter http://localhost:8080/ in a browser to see the application running.

    If you're using Docker natively on Linux, Docker Desktop for Mac, or Docker Desktop for
    Windows, then the web app should now be listening on port 5000 on your
    Docker daemon host. Point your web browser to http://localhost:8080 to
    find the `Hello World` message. If this doesn't resolve, you can also try
    http://127.0.0.1:8080.

    _Hint: In AWS Cloud9 use `Tools -> Preview -> Preview running applications` to open the browser on the appropriate remote address._

    You should see a message in your browser saying:

```
Hello World! I have been seen 1 times.
```

3.  Refresh the page.

    The number should increment.

```
Hello World! I have been seen 2 times.
```

4.  Switch to another terminal window, and type `docker image ls` to
    list local images.

    Listing images at this point should return `redis` and `web`.

```
$ docker image ls
REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
composetest-web         latest              4d7a24c23cf9        About a minute ago  244MB
python                  3.4-alpine          84e6077c7ab6        7 days ago          82.5MB
redis                   alpine              9d8fa9aa0e5b        3 weeks ago         27.5MB
```

    You can inspect images with `docker inspect <tag or id>`.

5.  Stop the application, either by running `docker compose down`
from within your project directory in the second terminal, or by
hitting CTRL+C in the original terminal where you started the app.

<a id="mount"></a>
## 3.5 Step 5: Edit the Compose file to add a bind mount

Edit `docker-compose.yml` in your project directory to add a bind mount for the `web` service:

```
services:
  web:
    build: .
    ports:
      - "8080:5000"
    volumes:
      - .:/code
    environment:
      FLASK_ENV: development
  redis:
    image: "redis:alpine"
```

The new `volumes` key mounts the project directory (current directory) on the
host to `/code` inside the container, allowing you to modify the code on the
fly, without having to rebuild the image. The `environment` key sets the
`FLASK_ENV` environment variable, which tells `flask run` to run in development
mode and reload the code on change. This mode should only be used in development.

<a id="rebuild"></a>
## 3.6 Step 6: Re-build and run the app with Compose

From your project directory, type `docker compose up` to build the app with the updated Compose file, and run it.

```
$ docker compose up
[+] Running 3/3
 ✔ Network composetest_default    Create...                               0.1s
 ✔ Container composetest-web-1    Create...                               0.1s
 ✔ Container composetest-redis-1  Crea...                                 0.1s
Attaching to redis-1, web-1
redis-1  | 1:C 03 May 2024 17:42:50.726 * oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
redis-1  | 1:C 03 May 2024 17:42:50.726 * Redis version=7.2.4, bits=64, commit=00000000, modified=0, pid=1, just started
redis-1  | 1:C 03 May 2024 17:42:50.726 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
redis-1  | 1:M 03 May 2024 17:42:50.726 * monotonic clock: POSIX clock_gettime
redis-1  | 1:M 03 May 2024 17:42:50.726 * Running mode=standalone, port=6379.
redis-1  | 1:M 03 May 2024 17:42:50.727 * Server initialized
redis-1  | 1:M 03 May 2024 17:42:50.727 * Ready to accept connections tcp
web-1    |  * Serving Flask app 'app.py'
web-1    |  * Debug mode: off
web-1    | WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
web-1    |  * Running on all addresses (0.0.0.0)
web-1    |  * Running on http://127.0.0.1:5000
web-1    |  * Running on http://172.20.0.3:5000
web-1    | Press CTRL+C to quit
```

Check the `Hello World` message in a web browser again, and refresh to see the
count increment.

<a id="update"></a>
## 3.7 Step 7: Update the application

Because the application code is now mounted into the container using a volume,
you can make changes to its code and see the changes instantly, without having
to rebuild the image.

1.  Change the greeting in `app.py` and save it. For example, change the `Hello World!` message to `Hello from Docker!`:

```
return 'Hello from Docker! I have been seen {} times.\n'.format(count)
```

2.  Refresh the app in your browser. The greeting should be updated, and the
    counter should still be incrementing.


<a id="experiment"></a>
## 3.8 Step 8: Experiment with some other commands

If you want to run your services in the background, you can pass the `-d` flag
(for "detached" mode) to `docker compose up` and use `docker compose ps` to
see what is currently running:
```
$ docker compose up -d
[+] Running 2/2
 ✔ Container composetest-web-1    Starte...                               0.5s
 ✔ Container composetest-redis-1  Star...                                 0.5s

$ docker compose ps
Name                 Command            State       Ports
-------------------------------------------------------------------
composetest_redis_1   /usr/local/bin/run         Up
composetest_web_1     /bin/sh -c python app.py   Up      8080->5000/tcp
```

The `docker compose run` command allows you to run one-off commands for your
services. For example, to see what environment variables are available to the
`web` service:

```
$ docker compose run web env
```

See `docker compose --help` to see other available commands.

If you started Compose with `docker compose up -d`, stop
your services once you've finished with them:

```
$ docker compose stop
```

You can bring everything down, removing the containers entirely, with the `down`
command. Pass `--volumes` to also remove the data volume used by the Redis
container:

```
$ docker compose down --volumes
```

At this point, you have seen the basics of how Compose works. [Go to next section](4_wordpress.md)
