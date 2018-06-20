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
    "https://img.buzzfeed.com/buzzfeed-static/static/2013-10/enhanced/webdr05/15/9/anigif_enhanced-buzz-26388-1381844103-11.gif",
    "https://img.buzzfeed.com/buzzfeed-static/static/2013-10/enhanced/webdr01/15/9/anigif_enhanced-buzz-31540-1381844535-8.gif",
    "https://img.buzzfeed.com/buzzfeed-static/static/2013-10/enhanced/webdr05/15/9/anigif_enhanced-buzz-26390-1381844163-18.gif",
    "https://img.buzzfeed.com/buzzfeed-static/static/2013-10/enhanced/webdr06/15/10/anigif_enhanced-buzz-1376-1381846217-0.gif",
    "https://img.buzzfeed.com/buzzfeed-static/static/2013-10/enhanced/webdr03/15/9/anigif_enhanced-buzz-3391-1381844336-26.gif",
    "https://img.buzzfeed.com/buzzfeed-static/static/2013-10/enhanced/webdr06/15/10/anigif_enhanced-buzz-29111-1381845968-0.gif",
    "https://img.buzzfeed.com/buzzfeed-static/static/2013-10/enhanced/webdr03/15/9/anigif_enhanced-buzz-3409-1381844582-13.gif",
    "https://img.buzzfeed.com/buzzfeed-static/static/2013-10/enhanced/webdr02/15/9/anigif_enhanced-buzz-19667-1381844937-10.gif",
    "https://img.buzzfeed.com/buzzfeed-static/static/2013-10/enhanced/webdr05/15/9/anigif_enhanced-buzz-26358-1381845043-13.gif",
    "https://img.buzzfeed.com/buzzfeed-static/static/2013-10/enhanced/webdr06/15/9/anigif_enhanced-buzz-18774-1381844645-6.gif",
    "https://img.buzzfeed.com/buzzfeed-static/static/2013-10/enhanced/webdr06/15/9/anigif_enhanced-buzz-25158-1381844793-0.gif",
    "https://img.buzzfeed.com/buzzfeed-static/static/2013-10/enhanced/webdr03/15/10/anigif_enhanced-buzz-11980-1381846269-1.gif"
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
Flask==0.10.1
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
      <p><small>Courtesy: <a href="http://www.buzzfeed.com/copyranter/the-best-cat-gif-post-in-the-history-of-cat-gifs">Buzzfeed</a></small></p>
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

The last step is the command for running the application which is simply - `python ./app.py`. Use the [CMD](https://docs.docker.com/engine/reference/builder/#cmd) command to do that -

```
CMD ["python", "/usr/src/app/app.py"]
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
CMD ["python", "/usr/src/app/app.py"]
```

Now that you finally have your `Dockerfile`, you can now build your image. The `docker image build` command does the heavy-lifting of creating a docker image from a `Dockerfile`.

While running the `docker image build` command given below, make sure to replace `<YOUR_USERNAME>`  with your username. This username should be the same on you created when you registered on [Docker hub](https://hub.docker.com). If you haven't done that yet, please go ahead and create an account. The `docker image build` command is quite simple - it takes an optional tag name with `-t` and a location of the directory containing the `Dockerfile` - the `.` indicates the current directory:

```
$ docker image build -t <YOUR_USERNAME>/myfirstapp .
Sending build context to Docker daemon 9.728 kB
Step 1 : FROM alpine:latest
 ---> 0d81fc72e790
Step 2 : RUN apk add --update py-pip
 ---> Running in 8abd4091b5f5
fetch http://dl-4.alpinelinux.org/alpine/v3.3/main/x86_64/APKINDEX.tar.gz
fetch http://dl-4.alpinelinux.org/alpine/v3.3/community/x86_64/APKINDEX.tar.gz
(1/12) Installing libbz2 (1.0.6-r4)
(2/12) Installing expat (2.1.0-r2)
(3/12) Installing libffi (3.2.1-r2)
(4/12) Installing gdbm (1.11-r1)
(5/12) Installing ncurses-terminfo-base (6.0-r6)
(6/12) Installing ncurses-terminfo (6.0-r6)
(7/12) Installing ncurses-libs (6.0-r6)
(8/12) Installing readline (6.3.008-r4)
(9/12) Installing sqlite-libs (3.9.2-r0)
(10/12) Installing python (2.7.11-r3)
(11/12) Installing py-setuptools (18.8-r0)
(12/12) Installing py-pip (7.1.2-r0)
Executing busybox-1.24.1-r7.trigger
OK: 59 MiB in 23 packages
 ---> 976a232ac4ad
Removing intermediate container 8abd4091b5f5
Step 3 : COPY requirements.txt /usr/src/app/
 ---> 65b4be05340c
Removing intermediate container 29ef53b58e0f
Step 4 : RUN pip install --no-cache-dir -r /usr/src/app/requirements.txt
 ---> Running in a1f26ded28e7
Collecting Flask==0.10.1 (from -r /usr/src/app/requirements.txt (line 1))
  Downloading Flask-0.10.1.tar.gz (544kB)
Collecting Werkzeug>=0.7 (from Flask==0.10.1->-r /usr/src/app/requirements.txt (line 1))
  Downloading Werkzeug-0.11.4-py2.py3-none-any.whl (305kB)
Collecting Jinja2>=2.4 (from Flask==0.10.1->-r /usr/src/app/requirements.txt (line 1))
  Downloading Jinja2-2.8-py2.py3-none-any.whl (263kB)
Collecting itsdangerous>=0.21 (from Flask==0.10.1->-r /usr/src/app/requirements.txt (line 1))
  Downloading itsdangerous-0.24.tar.gz (46kB)
Collecting MarkupSafe (from Jinja2>=2.4->Flask==0.10.1->-r /usr/src/app/requirements.txt (line 1))
  Downloading MarkupSafe-0.23.tar.gz
Installing collected packages: Werkzeug, MarkupSafe, Jinja2, itsdangerous, Flask
  Running setup.py install for MarkupSafe
  Running setup.py install for itsdangerous
  Running setup.py install for Flask
Successfully installed Flask-0.10.1 Jinja2-2.8 MarkupSafe-0.23 Werkzeug-0.11.4 itsdangerous-0.24
You are using pip version 7.1.2, however version 8.1.1 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.
 ---> 8de73b0730c2
Removing intermediate container a1f26ded28e7
Step 5 : COPY app.py /usr/src/app/
 ---> 6a3436fca83e
Removing intermediate container d51b81a8b698
Step 6 : COPY templates/index.html /usr/src/app/templates/
 ---> 8098386bee99
Removing intermediate container b783d7646f83
Step 7 : EXPOSE 5000
 ---> Running in 31401b7dea40
 ---> 5e9988d87da7
Removing intermediate container 31401b7dea40
Step 8 : CMD python /usr/src/app/app.py
 ---> Running in 78e324d26576
 ---> 2f7357a0805d
Removing intermediate container 78e324d26576
Successfully built 2f7357a0805d
```
> Note, the Alpine Linux CDN has been experiencing some trouble recently. If you encounter an error building this image, there's a workaround as outlined in [issue #104](https://github.com/docker/docker-birthday-3/issues/104). This is also reflected currently in the repo for the [Flask app](https://github.com/docker/docker-birthday-3/tree/master/flask-app)

If you don't have the `alpine:latest` image, the client will first pull the image and then create your image. Therefore, your output on running the command will look different from mine. If everything went well, your image should be ready! Run `docker image ls` and see if your image (`<YOUR_USERNAME>/myfirstapp`) shows.

The last step in this section is to run the image and see if it actually works.

```
$ docker container run -p 8888:5000 --name myfirstapp YOUR_USERNAME/myfirstapp
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
```

Head over to `http://<DOCKER_HOST-IP-ADDRESS>:8888` and your app should be live. You may need to open up another terminal and determine the container ip address using `docker-machine ip default` or use localhost:port to run the website.

<img src="https://raw.githubusercontent.com/docker/Docker-Birthday-3/master/tutorial-images/catgif.png" title="static">

Hit the Refresh button in the web browser to see a few more cat images.

OK, now that you are done with the this container, stop and remove it since you won't be using it again.

Open another terminal window and execute the following commands:

```
$ docker container stop myfirstapp
$ docker container rm myfirstapp
```

Yay, know you know how to create images from Dockerfiles! Let's take a look at a more complex application orchestration. <a href="3_compose.md">Goto next section.</a>
