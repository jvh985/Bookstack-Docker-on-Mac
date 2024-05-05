# Setting up Bookstack with Docker on macOS
How to setup a running Bookstack instance in a Docker container on MacOS using a compose file

## Why make this?
I ran into a few issues learning Docker and it wasn't as straightforward on Mac as it was for other implementations.

Also, updates to Docker caused video tutorials to be of little help in this process. This is to help anyone that finds this guide as it 
explains why certain things will be done the way they are instead of just blindly following along.

This is meant for people who are trying to learn a bit about Docker using compose files. I myself learn more through teaching a concept
so it has the added benefit of solidifying the information for myself.

## What is Docker?
Docker is essentially a software package that wraps around your application and **ALL** of its dependencies. For example, say your application needs
a database and various other packages. Then all of that will be wrapped in the Docker container so that any system anywhere can have everything it needs
to run your application. 

## Installing Bookstack with a compose file
I followed the instructions from [Bookstack's site](https://www.bookstackapp.com/docs/admin/installation/)

I initially tried to install manually but kept running into issues. So I went with the compose file route. Scroll down to the header: **Docker Containers**

The top repository is from LinuxServer.io. They have a fantastic explaination.
Here there is a link to [LinuxServer.io repository](https://github.com/linuxserver/docker-bookstack)

Read through the README file and you'll come accross a compose file. The file itself it a bit outdated and video tutorials don't account for this either.

As of now "docker-compose" is depreciated and no longer used with your terminal/command line. The word docker is now the active application in your PATH and it 
has the 'compose' command associated with it. Also the use of 'version' inside the .yml file is now depreciated and isn't required. The naming convention for the 
docker compose file is now just 'compose.yml' instead of 'docker-compose.yml'

The other issue I ran into was the APP_URL. On mac, you need to use your full localhost address **__AND__** the port you have assigned to the connect to the container. 
Every Docker container has a port and you need to assign an unused port to that Docker port on your machine so it will let you through. This port you've assinged
has to be attached to the localhost address.

The ports work as such: 'your assigned port':'Docker port' so you change the first port to an unassigned port for your system. In the example below the computer port is
changed to 6875. Making the ports section look like: - 6875:80

The APP_URL for mac should look like this: 'https :// localhost:6875'

To start create and navigate to your application folder directory and create your compose file. It is a yaml file so 'compose.yml' is what Docker will look for.

The updated compose file should look like:

```python
services:
  bookstack:
    image: lscr.io/linuxserver/bookstack
    container_name: bookstack
    environment:
      - PUID=1000
      - PGID=1000
      - APP_URL=https://localhost:6875
      - DB_HOST=bookstack_db
      - DB_PORT=3306
      - DB_USER=bookstack
      - DB_PASS=<yourdbpass>
      - DB_DATABASE=bookstackapp
    volumes:
      - ./bookstack_app_data:/config
    ports:
      - 6875:80
    restart: unless-stopped
    depends_on:
      - bookstack_db
  bookstack_db:
    image: lscr.io/linuxserver/mariadb
    container_name: bookstack_db
    environment:
      - PUID=1000
      - PGID=1000
      - MYSQL_ROOT_PASSWORD=<yourdbpass>
      - TZ=Europe/London
      - MYSQL_DATABASE=bookstackapp
      - MYSQL_USER=bookstack
      - MYSQL_PASSWORD=<yourdbpass>
    volumes:
      - ./bookstack_db_data:/config
    restart: unless-stopped
```

Now, if you are brand new to Docker, then a lot of this will seem confusing. But take the time to look through this file and get a feel for why things are the
way they are and the indention importance.

This file shows that there are 2 services that will be started inside your Docker container: Bookstack itself, and a database for Bookstack. The PUID and PGID
can be left alone completely if you are just spinning up a local instance of this application, however they need to be changed in accordance to different 
companies user group id's if used in a workplace setting.

The usernames and passwords should be changed to be something much more secure that what is shown, just make sure that they are correct. There is some
overlap in the naming between the services. 

The volumes are where the information is stored inside your application directory and if you aren't too familiar with terminal syntax ./ means the path to
the current directory you are within, inside the command line/terminal. So there is no need to type out the full path of your folder. Everything after the 
/ is creating new folders for the data of each service and the :/config is storing the configuration in those new folders for each service.

---

## Actually using the compose file
Now you will need to actually use the compose file.

The command to read the yml file, start a container and run the contents of the file are:

> docker compose up

There are a few options to add to the docker command in terminal to make things easier for yourself:

> -p <*your custom container name*>

> -d

The -p option allows you to create a custom name for your container instead of a random string.

The -d option runs everything in the background instead of locking that terminal window to the running container, which allows you to
continue using your terminal window for checking your containers and viewing logs.

The full command would look something like:

> docker compose -p test_container up -d

**__NOTE__**: You may receive an error: "error getting credentials - err docker-credential-desktop"

This is common and you need to edit the docker config.json file. A simple fix, just open the file in an editor:

> sudo nano ~/.docker/config.json

Inside you will see a line containing credsStore. Just change it to credStore and save. Everything will work fine afterwards.

To see what containers you have running just use:

> docker ps

This will show you **__running__** containers. If you want to see every container created use:

> docker ps -a

---

## Very useful tip for new users

Get used to looking at the logs of everything, it will give you really useful information and lead you to fix issues on your own
without the need to search the web and find all sorts of potential solutions that may or may not work.

Use the command:

> docker logs <*container-name*>

Depending on what you are running in your container or if it's endlessly restarting you may need to ctrl+c a few times to exit the 
log.

---

# Hopefully this helps anyone who stumbles across it. Again, please refer to the main bookstackapp website and linuxserver.io for extra
documentation and support. They've done all of the hard work.



