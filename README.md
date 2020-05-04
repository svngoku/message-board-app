# Message Board App

## Django, Docker, and PostgreSQL Tutorial

<span class="italic text-secondary">Set up a new Django project using Docker and PostgreSQL.</span>

In this tutorial we will create a new Django project using Docker and PostgreSQL. Django ships with built-in SQLite support but even for local development you are better off using a "real" database like PostgreSQL that matches what is in production.

It's _possible_ to run PostgreSQL locally using a tool like [Postgres.app](https://postgresapp.com/), however the preferred choice among many developers today is to use [Docker](https://www.docker.com/), a tool for creating isolated operating systems. The easiest way to think of it is as a large virtual environment that contains everything needed for our Django project: dependencies, database, caching services, and any other tools needed.

A big reason to use Docker is that it completely removes any issues around local development set up. Instead of worrying about which software packages are installed or running a local database alongside a project, you simply run a Docker image of the entire project. Best of all, this can be shared in groups and makes team development much simpler.

## Install Docker

The first step is to install the desktop Docker app for your local machine:

*   [Docker for Mac](https://docs.docker.com/docker-for-mac/install/)
*   [Docker for Windows](https://docs.docker.com/docker-for-windows/install/)
*   [Docker for Linux](https://docs.docker.com/install/)

The initial download of Docker might take some time to download. It is a big file. Feel free to stretch your legs at this point!

Once Docker is done installing we can confirm the correct version is running. In your terminal run the command `docker --version`.

<div class="codehilite">

<pre><span></span>$ docker --version
Docker version 19.03.8, build afacb8b
</pre>

</div>

[Docker Compose](https://docs.docker.com/compose/) is an additional tool that is automatically included with Mac and Windows downloads of Docker. However if you are on Linux, you will need to add it manually. You can do this by running the command `sudo pip install docker-compose` after your Docker installation is complete.

## Django project

We will use the [Message Board app](https://djangoforbeginners.com/message-board/) from [Django for Beginners](http://djangoforbeginners.com/). It provides the code for a basic message board app using SQLite that can be updated in the admin.

Create a new directory on your Desktop and clone the repo into it.

<div class="codehilite">

<pre><span></span>$ <span class="nb">cd</span> ~/Desktop
$ git clone https://github.com/svngoku/message-board-app
$ <span class="nb">cd</span> message-board-app
</pre>

</div>

Then install the software packages specified by `Pipenv` and start a new shell. If you see `(message-board-app)` then you know the virtual environment is active.

<div class="codehilite">

<pre><span></span>$ pipenv install
$ pipenv shell
<span class="o">(</span>message-board-app<span class="o">)</span> $
</pre>

</div>

Make sure to `migrate` our database after these changes.

<div class="codehilite">

<pre><span></span><span class="err">(message-board-app) $ python manage.py migrate</span>
</pre>

</div>

If you now use the `python manage.py runserver` command you can see a working version of our application at [http://localhost:8000](http://localhost:8000).

![Message Board Homepage](https://learndjango.com/static/images/tutorials/docker_postgresql/mb-homepage.png)

## Docker part 2

## Images and Containers

There are two importance concepts to grasp in Docker: **images** and **containers**.

*   **Image**: the list of instructions for all the software packages in your projects
*   **Container**: a runtime instance of the image

In other words, an image _describes_ what will happen and a container _is_ what actually runs.

To configure Docker images and containers we use two files: `Dockerfile` and `docker-compose.yml`.

The `Dockerfile` contains the list of instructions for **the image**, aka, What actually goes on in the environment of the container.

Create a new `Dockerfile` file.

<div class="codehilite">

<pre><span></span><span class="err">(message-board-app) $ touch Dockerfile</span>
</pre>

</div>

Then add the following code in your text editor.

<div class="codehilite">

<pre><span></span><span class="o">#</span> <span class="n">Dockerfile</span>

<span class="o">#</span> <span class="n">Pull</span> <span class="n">base</span> <span class="n">image</span>
<span class="k">FROM</span> <span class="n">python</span><span class="p">:</span><span class="mi">3</span><span class="p">.</span><span class="mi">7</span>

<span class="o">#</span> <span class="k">Set</span> <span class="n">environment</span> <span class="n">variables</span>
<span class="n">ENV</span> <span class="n">PYTHONDONTWRITEBYTECODE</span> <span class="mi">1</span>
<span class="n">ENV</span> <span class="n">PYTHONUNBUFFERED</span> <span class="mi">1</span>

<span class="o">#</span> <span class="k">Set</span> <span class="k">work</span> <span class="n">directory</span>
<span class="n">WORKDIR</span> <span class="o">/</span><span class="n">code</span>

<span class="o">#</span> <span class="n">Install</span> <span class="n">dependencies</span>
<span class="n">RUN</span> <span class="n">pip</span> <span class="n">install</span> <span class="n">pipenv</span>
<span class="k">COPY</span> <span class="n">Pipfile</span> <span class="n">Pipfile</span><span class="p">.</span><span class="k">lock</span> <span class="o">/</span><span class="n">code</span><span class="o">/</span>
<span class="n">RUN</span> <span class="n">pipenv</span> <span class="n">install</span> <span class="c1">--system</span>

<span class="o">#</span> <span class="k">Copy</span> <span class="n">project</span>
<span class="k">COPY</span> <span class="p">.</span> <span class="o">/</span><span class="n">code</span><span class="o">/</span>
</pre>

</div>

On the top line we're using an [official Docker image](https://hub.docker.com/_/python/) for Python 3.7\. Next we create two environment variables. `PYTHONUNBUFFERED` ensures our console output looks familiar and is not buffered by Docker, which we don't want. [PYTHONDONTWRITEBYTECODE](https://docs.python.org/3/using/cmdline.html?highlight=pythondontwritebytecode#envvar-PYTHONDONTWRITEBYTECODE) means Python won't try to write `.pyc` files which we also do not desire.

The next line sets the `WORKDIR` to `/code`. This means the working directory is located at `/code` so in the future to run any commands like `manage.py` we can just use `WORKDIR` rather than need to remember where exactly on Docker our code is actually located.

Then we install our dependencies, making sure we have the latest version of `pip`, installing `pipenv`, copying our local `Pipfile` and `Pipfile.lock` into Docker and then running it to install our dependencies. The `RUN` command lets us run commands in Docker just as we would on the command line.

Next we need a new `docker-compose.yml` file. This tells Docker _how_ to run our Docker container.

<div class="codehilite">

<pre><span></span><span class="err">(message-board-app) $ touch docker-compose.yml</span>
</pre>

</div>

Then type in the following code.

<div class="codehilite">

<pre><span></span><span class="k">version</span><span class="p">:</span> <span class="s1">'3.7'</span>

<span class="n">services</span><span class="p">:</span>
  <span class="n">db</span><span class="p">:</span>
    <span class="n">image</span><span class="p">:</span> <span class="ss">"postgres:11"</span>
    <span class="n">environment</span><span class="p">:</span>
      <span class="o">-</span> <span class="ss">"POSTGRES_HOST_AUTH_METHOD=trust"</span>
    <span class="n">volumes</span><span class="p">:</span>
      <span class="o">-</span> <span class="n">postgres_data</span><span class="p">:</span><span class="o">/</span><span class="n">var</span><span class="o">/</span><span class="n">lib</span><span class="o">/</span><span class="n">postgresql</span><span class="o">/</span><span class="k">data</span><span class="o">/</span>
  <span class="n">web</span><span class="p">:</span>
    <span class="n">build</span><span class="p">:</span> <span class="p">.</span>
    <span class="n">command</span><span class="p">:</span> <span class="n">python</span> <span class="o">/</span><span class="n">code</span><span class="o">/</span><span class="n">manage</span><span class="p">.</span><span class="n">py</span> <span class="n">runserver</span> <span class="mi">0</span><span class="p">.</span><span class="mi">0</span><span class="p">.</span><span class="mi">0</span><span class="p">.</span><span class="mi">0</span><span class="p">:</span><span class="mi">8000</span>
    <span class="n">volumes</span><span class="p">:</span>
      <span class="o">-</span> <span class="p">.:</span><span class="o">/</span><span class="n">code</span>
    <span class="n">ports</span><span class="p">:</span>
      <span class="o">-</span> <span class="mi">8000</span><span class="p">:</span><span class="mi">8000</span>
    <span class="n">depends_on</span><span class="p">:</span>
      <span class="o">-</span> <span class="n">db</span>

<span class="n">volumes</span><span class="p">:</span>
  <span class="n">postgres_data</span><span class="p">:</span>
</pre>

</div>

On the top line we're using the [most recent version](https://docs.docker.com/compose/compose-file/compose-versioning/) of Compose which is `3.7`.

Under `db` for the database we want the Docker image for Postgres 10.1 and use `volumes` to tell Compose where the container should be located in our Docker container.

For `web` we're specifying how the web service will run. First Compose needs to build an image from the current directory and start up the server at `0.0.0.0:8000`. We use `volumes` to tell Compose to store the code in our Docker container at `/code/`. The `ports` config lets us map our own port 8000 to the port 8000 in the Docker container. This is the default Django port. And finally `depends_on` says that we should start the `db` first before running our web services.

The last section `volumes` is because Compose has a rule that you must list named volumes in a top-level `volumes` key.

We can `exit` the virtual environment now since we're about to switch over to running Docker instead.

<div class="codehilite">

<pre><span></span><span class="err">(message-board-app) $ exit</span>
<span class="err">$</span> 
</pre>

</div>

Now start the Docker container using the `up` command, adding the `-d` flag so it runs in detached mode, and the `--build` flag to build our initial image. If we did not add this flag, we'd need to open a separate command line tab to execute commands.

<div class="codehilite">

<pre><span></span>$ docker-compose up -d --build
</pre>

</div>

Docker is all set! You can visit the Message Board homepage at [http://127.0.0.1:8000/](http://127.0.0.1:8000/) and it will show, now running via Docker.

## Update to PostgreSQL

We need to update our _Message Board app_ to use PostgreSQL instead of SQLite. First install `psycopg2-binary` for our database bindings to PostgreSQL.

<div class="codehilite">

<pre><span></span>$ pipenv install psycopg2-binary
</pre>

</div>

Then update the `settings.py` file to specify we'll be using PostgreSQL not SQLite.

<div class="codehilite">

<pre><span></span><span class="c1"># settings.py</span>
<span class="n">DATABASES</span> <span class="o">=</span> <span class="p">{</span>
    <span class="s1">'default'</span><span class="p">:</span> <span class="p">{</span>
        <span class="s1">'ENGINE'</span><span class="p">:</span> <span class="s1">'django.db.backends.postgresql'</span><span class="p">,</span>
        <span class="s1">'NAME'</span><span class="p">:</span> <span class="s1">'postgres'</span><span class="p">,</span>
        <span class="s1">'USER'</span><span class="p">:</span> <span class="s1">'postgres'</span><span class="p">,</span>
        <span class="s1">'HOST'</span><span class="p">:</span> <span class="s1">'db'</span><span class="p">,</span> <span class="c1"># set in docker-compose.yml</span>
        <span class="s1">'PORT'</span><span class="p">:</span> <span class="mi">5432</span> <span class="c1"># default postgres port</span>
    <span class="p">}</span>
<span class="p">}</span>
</pre>

</div>

We should `migrate` our database at this point on Docker.

<div class="codehilite">

<pre><span></span>$ docker-compose <span class="nb">exec</span> web python manage.py migrate
</pre>

</div>

## Admin

Since the _Message Board app_ requires using the admin, create a superuser on Docker. Fill out the prompts after running the command below.

<div class="codehilite">

<pre><span></span>$ docker-compose <span class="nb">exec</span> web python manage.py createsuperuser
</pre>

</div>

Now go to [http://127.0.0.1:8000/admin](http://127.0.0.1:8000/admin) and login. You can add new posts via the admin and then seem them on the homepage just as described in [Django for Beginners](https://djangoforbeginners.com/message-board/).

![Message Board Admin](https://learndjango.com/static/images/tutorials/docker_postgresql/admin.png)

When you're done, don't forget to close down your Docker container.

<div class="codehilite">

<pre><span></span>$ docker-compose down
</pre>

</div>

## Quick Review

Here is a short version of the terms and concepts we've covered in this post:

*   **Image**: the "definition" of your project
*   **Container**: what your project actually runs in (an instance of the image)
*   **Dockerfile**: defines what your image looks like
*   **docker-compose.yml**: a [YAML](http://yaml.org/) file that takes the Dockerfile and adds additional instructions for how our Docker container should behave in production

We use the _Dockerfile_ to tell Docker how to build our _image_. Then we run our actual project within a _container_. The _docker-compose.yml_ file provides additional information for how our Docker container should behave in production.
