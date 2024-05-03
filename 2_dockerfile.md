<a href="Readme.md#table-of-contents" class="top" id="preface">Top</a>
<a id="our-image"></a>
### 2.3 Our First Image

Now that you have a better understanding of images, it's time to create our own. Our goal in this section will be to create an image that sandboxes a small [Flask](http://flask.pocoo.org) application.
For the purposes of this workshop, we'll created a fun little Python Flask app that displays a random cat `.gif` every time it is loaded - because you know, who doesn't like cats?

<a id="dockerfiles"></a>
### 2.4 Dockerfile

A [Dockerfile](https://docs.docker.com/engine/reference/builder/) is a text-file that contains a list of commands that the Docker daemon calls while creating an image. It is simple way to automate the image creation process. The best part is that the [commands](https://docs.docker.com/engine/reference/builder/) you write in a Dockerfile are *almost* identical to their equivalent Linux commands. This means you don't really have to learn new syntax to create your own Dockerfiles.

**The goal of this exercise is to create a Docker image which will run a Flask app.**

Start by creating a folder ```flask-app``` where we'll create the following files:

```
- Dockerfile
- app.py
- requirements.txt
- templates/index.html
```

Create the **app.py** with the following content:

```
from flask import Flask, render_template
import random

app = Flask(__name__)

# list of cat images
images = [
    "https://media2.giphy.com/media/12PA1eI8FBqEBa/giphy.gif",
    "https://media0.giphy.com/media/C9x8gX02SnMIoAClXa/giphy.gif",
    "https://media0.giphy.com/media/1iu8uG2cjYFZS6wTxv/giphy.gif",
    "https://media3.giphy.com/media/JIX9t2j0ZTN9S/giphy.gif",
    "https://media3.giphy.com/media/JIX9t2j0ZTN9S/giphy.gif"
]

@app.route('/')
def index():
    url = random.choice(images)
    return render_template('index.html', url=url)

if __name__ == "__main__":
    app.run(host="0.0.0.0")
```

In order to install Python modules required for our app we need to add to **requirements.txt** file the following line:

```
Flask==1.1.4
markupsafe==2.0.1
```

Create a directory called `templates` and create a **index.html** file in that directory, to have the same content as below:

```
<html>
  <head>
    <style type="text/css">
      body {
        background: black;
        color: white;
      }
      div.container {
        max-width: 500px;
        margin: 100px auto;
        border: 20px solid white;
        padding: 10px;
        text-align: center;
      }
      h4 {
        text-transform: uppercase;
      }
    </style>
  </head>
  <body>
    <div class="container">
      <h4>Cat Gif of the day</h4>
      <img src="{{url}}" />
      <p><small>Courtesy: <a href="https://giphy.com/explore/cat">Giphy</a></small></p>
    </div>
  </body>
</html>
```

The next step now is to create a Docker image with this web app. As mentioned above, all user images are based off a base image. Since our application is written in Python, we will build our own Python image based on [Alpine](https://hub.docker.com/_/alpine/). We'll do that using a **Dockerfile**.

Create a file **Dockerfile**.
Start by specifying our base image. Use the `FROM` keyword to do that

```
FROM alpine:latest
```

The next step usually is to write the commands of copying the files and installing the dependencies.
But first we will install the Python pip package to the alpine linux distribution. This will not just install the pip package but any other dependencies too, which includes the python interpreter. Add the following [RUN](https://docs.docker.com/engine/reference/builder/#run) command next:
```
RUN apk add --update py-pip
```

Additionally, we create a virtual environment for Python and source add it to the path variable
```
RUN python3 -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"
```

Next, let us add the files that make up the Flask Application.

Install all Python requirements for our app to run. This will be accomplished by adding the lines:

```
COPY requirements.txt /usr/src/app/
RUN pip install --no-cache-dir -r /usr/src/app/requirements.txt
```

Copy the files you have created earlier our image by using [COPY](https://docs.docker.com/engine/reference/builder/#copy)  command.

```
COPY app.py /usr/src/app/
COPY templates/index.html /usr/src/app/templates/
```

The next thing you need to specify is the port number which needs to be exposed. Since our flask app is running on `5000` that's what we'll expose.
```
EXPOSE 5000
```

The last step is the command for running the application which is simply - `python3 ./app.py`. Use the [CMD](https://docs.docker.com/engine/reference/builder/#cmd) command to do that -

```
CMD ["python3", "/usr/src/app/app.py"]
```

The primary purpose of `CMD` is to tell the container which command it should run by default when it is started. With that, our `Dockerfile` is now ready. This is how it looks:

```
# our base image
FROM alpine:latest

# Install python and pip
RUN apk add --update py-pip

# create a virtual environment for Python
RUN python3 -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

# install Python modules needed by the Python app
COPY requirements.txt /usr/src/app/
RUN pip install --no-cache-dir -r /usr/src/app/requirements.txt

# copy files required for the app to run
COPY app.py /usr/src/app/
COPY templates/index.html /usr/src/app/templates/

# tell the port number the container should expose
EXPOSE 5000

# run the application
CMD ["python3", "/usr/src/app/app.py"]
```

Now that you finally have your `Dockerfile`, you can now build your image. The `docker image build` command does the heavy-lifting of creating a docker image from a `Dockerfile`.

While running the `docker image build` command given below, make sure to replace `<YOUR_USERNAME>`  with your username. This username should be the same on you created when you registered on [Docker hub](https://hub.docker.com). If you haven't done that yet, please go ahead and create an account. The `docker image build` command is quite simple - it takes an optional tag name with `-t` and a location of the directory containing the `Dockerfile` - the `.` indicates the current directory:

```
$ docker image build -t <YOUR_USERNAME>/myfirstapp .
[+] Building 23.5s (13/13) FINISHED                               docker:default
 => [internal] load build definition from Dockerfile              0.0s
 => => transferring dockerfile: 663B                              0.0s
 => [internal] load metadata for docker.io/library/alpine:latest  0.0s
 => [internal] load .dockerignore                                 0.0s
 => => transferring context: 2B                                   0.0s
 => [1/8] FROM docker.io/library/alpine:latest                    0.0s
 => [internal] load build context                                 0.1s
 => => transferring context: 1.37kB                               0.0s
 => [2/8] RUN apk add --update py-pip                            11.1s
 => [3/8] COPY requirements.txt /usr/src/app/                     0.1s
 => [4/8] RUN python3 -m venv /opt/venv                           6.9s
 => [5/8] WORKDIR /usr/src/app                                    0.1s
 => [6/8] RUN pip install --no-cache-dir -r requirements.txt      4.3s
 => [7/8] COPY app.py /usr/src/app/                               0.1s
 => [8/8] COPY templates/index.html /usr/src/app/templates/       0.1s
 => exporting to image                                            0.6s
 => => exporting layers                                           0.5s
 => => writing image sha256:1e02501e29451f993b9767117e101db46d447e284cba6e20f7aad44910bbae93        0.0s
 => => naming to docker.io/<YOUR_USERNAME>/myfirstapp
```
> Note, the Alpine Linux CDN has been experiencing some trouble recently. If you encounter an error building this image, there's a workaround as outlined in [issue #104](https://github.com/docker/docker-birthday-3/issues/104). This is also reflected currently in the repo for the [Flask app](https://github.com/docker/docker-birthday-3/tree/master/flask-app)

If you don't have the `alpine:latest` image, the client will first pull the image and then create your image. Therefore, your output on running the command will look different from mine. If everything went well, your image should be ready! Run `docker image ls` and see if your image (`<YOUR_USERNAME>/myfirstapp`) shows.

The last step in this section is to run the image and see if it actually works.

```
$ docker container run -it --rm -p 8080:5000 --name myfirstapp <YOUR_USERNAME>/myfirstapp
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
```

Head over to `http://localhost:8080` and your app should be live.
_Hint: In AWS Cloud9 use `Tools -> Preview -> Preview running applications` to open the browser on the appropriate remote address._

<img src="https://raw.githubusercontent.com/docker/Docker-Birthday-3/master/tutorial-images/catgif.png" title="static">

Hit the Refresh button in the web browser to see a few more cat images.

OK, now that you are done with the this container, stop and remove it since you won't be using it again.

Open another terminal window and execute the following commands:

```
$ docker container stop myfirstapp
$ docker container rm myfirstapp
```

Yay, know you know how to create images from Dockerfiles! Let's take a look at a more complex application orchestration. <a href="3_compose.md">Goto next section.</a>
