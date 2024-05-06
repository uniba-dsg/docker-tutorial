<a href="Readme.md#table-of-contents" class="top" id="preface">Top</a>
<a id="compose"></a>
# 4. Docker Compose and WordPress

You can use Docker Compose to easily run WordPress in an isolated environment built
with Docker containers. This quick-start guide demonstrates how to use Compose to set up and run WordPress. Before starting, you'll need to have
[Compose installed](https://docs.docker.com/compose/install/).

<a id="project"></a>
### 4.1 Define the project

1. Create an empty project directory.

    You can name the directory something easy for you to remember. This directory is the context for your application image. The directory should only contain resources to build that image.

    This project directory will contain a `docker-compose.yml` file which will be complete in itself for a good starter wordpress project.

2. Change directories into your project directory.

    For example, if you named your directory `my-wordpress`:
    ```
    $ cd my-wordpress/
    ```
3. Create a `docker-compose.yml` file that will start your `Wordpress` blog and a separate `MySQL` instance:
   ```
        networks:
           backend:

        volumes:
           db_data: {}

        services:
           db:
             image: mysql:5.7
             volumes:
               - db_data:/var/lib/mysql
             networks:
               - backend
             restart: always
             environment:
               MYSQL_ROOT_PASSWORD: somewordpress
               MYSQL_DATABASE: wordpress
               MYSQL_USER: wordpress
               MYSQL_PASSWORD: wordpress

           wordpress:
             depends_on:
               - db
             image: wordpress:latest
             networks:
               - backend
             ports:
               - "8080:80"
             restart: always
             environment:
               WORDPRESS_DB_HOST: db:3306
               WORDPRESS_DB_USER: wordpress
               WORDPRESS_DB_PASSWORD: wordpress
               WORDPRESS_DB_NAME: wordpress        
   ```

<a id="build"></a>
### 4.2 Build the project

Now, run `docker compose up -d` from your project directory.

This pulls the needed images, and starts the wordpress and database containers, as shown in the example below.

```
$ docker compose up -d
[+] Running 34/2
 ✔ wordpress Pulled                                 34.2s
 ✔ db Pulled                                        19.4s
[+] Running 4/4
 ✔ Network wordpress_backend        Created         0.1s
 ✔ Volume "wordpress_db_data"       Created         0.0s
 ✔ Container wordpress-db-1         Started         102.0s
 ✔ Container wordpress-wordpress-1  Started         0.8s
```

<a id="run"></a>
### 4.3 Bring up WordPress in a web browser

At this point, WordPress should be running on port `8080` of your Docker Host, and you can complete the "famous five-minute installation" as a WordPress administrator.

_Hint: In AWS Cloud9 use `Tools -> Preview -> Preview running applications` to open the browser on the appropriate remote address._

**NOTE**: The Wordpress site will not be immediately available on port `8080` because the containers are still being initialized and may take a couple of minutes before the first load.

![Choose language for WordPress install](images/wordpress-lang.png)

![WordPress Welcome](images/wordpress-welcome.png)

Bam, you got Wordpress running with a database in two separate containers. That was easy, wasn't it?
At this point, you've completed the tutorial. Now it's time to explore the capabilities of Docker on your on. Have fun!
Source: [Wordpress on Docker](https://docs.docker.com/compose/wordpress/)
