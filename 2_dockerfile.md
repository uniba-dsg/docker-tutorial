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
Sending build context to Docker daemon  6.656kB
Step 1/8 : FROM alpine:latest
latest: Pulling from library/alpine
df20fa9351a1: Pull complete
Digest: sha256:185518070891758909c9f839cf4ca393ee977ac378609f700f60a771a2dfe321
Status: Downloaded newer image for alpine:latest
 ---> a24bb4013296
Step 2/8 : RUN apk add --update py-pip
 ---> Running in a62262f5d681
fetch http://dl-cdn.alpinelinux.org/alpine/v3.12/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.12/community/x86_64/APKINDEX.tar.gz
(1/36) Installing libbz2 (1.0.8-r1)
(2/36) Installing expat (2.2.9-r1)
(3/36) Installing libffi (3.3-r2)
(4/36) Installing gdbm (1.13-r1)
(5/36) Installing xz-libs (5.2.5-r0)
(6/36) Installing ncurses-terminfo-base (6.2_p20200523-r0)
(7/36) Installing ncurses-libs (6.2_p20200523-r0)
(8/36) Installing readline (8.0.4-r0)
(9/36) Installing sqlite-libs (3.32.1-r0)
(10/36) Installing python3 (3.8.3-r0)
(11/36) Installing py3-appdirs (1.4.4-r1)
(12/36) Installing py3-ordered-set (4.0.1-r0)
(13/36) Installing py3-parsing (2.4.7-r0)
(14/36) Installing py3-six (1.15.0-r0)
(15/36) Installing py3-packaging (20.4-r0)
(16/36) Installing py3-setuptools (47.0.0-r0)
(17/36) Installing py3-chardet (3.0.4-r4)
(18/36) Installing py3-idna (2.9-r0)
(19/36) Installing py3-certifi (2020.4.5.1-r0)
(20/36) Installing py3-urllib3 (1.25.9-r0)
(21/36) Installing py3-requests (2.23.0-r0)
(22/36) Installing py3-msgpack (1.0.0-r0)
(23/36) Installing py3-lockfile (0.12.2-r3)
(24/36) Installing py3-cachecontrol (0.12.6-r0)
(25/36) Installing py3-colorama (0.4.3-r0)
(26/36) Installing py3-distlib (0.3.0-r0)
(27/36) Installing py3-distro (1.5.0-r1)
(28/36) Installing py3-webencodings (0.5.1-r3)
(29/36) Installing py3-html5lib (1.0.1-r4)
(30/36) Installing py3-pytoml (0.1.21-r0)
(31/36) Installing py3-pep517 (0.8.2-r0)
(32/36) Installing py3-progress (1.5-r0)
(33/36) Installing py3-toml (0.10.1-r0)
(34/36) Installing py3-retrying (1.3.3-r0)
(35/36) Installing py3-contextlib2 (0.6.0-r0)
(36/36) Installing py3-pip (20.1.1-r0)
Executing busybox-1.31.1-r16.trigger
OK: 65 MiB in 50 packages
Removing intermediate container a62262f5d681
 ---> d929a614524c
Step 3/8 : COPY requirements.txt /usr/src/app/
 ---> c331941296ff
Step 4/8 : RUN pip install --no-cache-dir -r /usr/src/app/requirements.txt
 ---> Running in f7bad223acb9
Collecting Flask==1.1.2
  Downloading Flask-1.1.2-py2.py3-none-any.whl (94 kB)
Collecting itsdangerous>=0.24
  Downloading itsdangerous-1.1.0-py2.py3-none-any.whl (16 kB)
Collecting Werkzeug>=0.15
  Downloading Werkzeug-1.0.1-py2.py3-none-any.whl (298 kB)
Collecting Jinja2>=2.10.1
  Downloading Jinja2-2.11.2-py2.py3-none-any.whl (125 kB)
Collecting click>=5.1
  Downloading click-7.1.2-py2.py3-none-any.whl (82 kB)
Collecting MarkupSafe>=0.23
  Downloading MarkupSafe-1.1.1.tar.gz (19 kB)
Using legacy setup.py install for MarkupSafe, since package 'wheel' is not installed.
Installing collected packages: itsdangerous, Werkzeug, MarkupSafe, Jinja2, click, Flask
    Running setup.py install for MarkupSafe: started
    Running setup.py install for MarkupSafe: finished with status 'done'
Successfully installed Flask-1.1.2 Jinja2-2.11.2 MarkupSafe-1.1.1 Werkzeug-1.0.1 click-7.1.2 itsdangerous-1.1.0
Removing intermediate container f7bad223acb9
 ---> 9e1edc94ac44
Step 5/8 : COPY app.py /usr/src/app/
 ---> 29da3e4ceeab
Step 6/8 : COPY templates/index.html /usr/src/app/templates/
 ---> b58b376c08ac
Step 7/8 : EXPOSE 5000
 ---> Running in 52fd80c57790
Removing intermediate container 52fd80c57790
 ---> f1ffb484e232
Step 8/8 : CMD ["python3", "/usr/src/app/app.py"]
 ---> Running in b791acbfbfd1
Removing intermediate container b791acbfbfd1
 ---> c581490d4902
Successfully built c581490d4902
```
> Note, the Alpine Linux CDN has been experiencing some trouble recently. If you encounter an error building this image, there's a workaround as outlined in [issue #104](https://github.com/docker/docker-birthday-3/issues/104). This is also reflected currently in the repo for the [Flask app](https://github.com/docker/docker-birthday-3/tree/master/flask-app)

If you don't have the `alpine:latest` image, the client will first pull the image and then create your image. Therefore, your output on running the command will look different from mine. If everything went well, your image should be ready! Run `docker image ls` and see if your image (`<YOUR_USERNAME>/myfirstapp`) shows.

The last step in this section is to run the image and see if it actually works.

```
$ docker container run -it --rm -p 8080:5000 --name myfirstapp YOUR_USERNAME/myfirstapp
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
```

Head over to `http://<DOCKER_HOST-IP-ADDRESS>:8080` and your app should be live. You may need to open up another terminal and determine the container ip address using `docker-machine ip default` or use localhost:port to run the website.
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
