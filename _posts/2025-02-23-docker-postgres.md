---
layout: post
title: A Quick Guide to PostgreSQL with Docker
description: "A easy way to use a PostgreSQL database in a container to run tests and development"
tags: programming english
categories: misc
image: https://live.staticflickr.com/65535/51936373085_d70f7e8117_b.jpg
---

A month ago, my Ubuntu [24.04.2 LTS](https://ubuntu.com/download/desktop) system had a problem during a system update and wouldn't boot.  While troubleshooting in recovery mode, I discovered the issue was with [PostgreSQL](https://www.postgresql.org/). I removed PostgreSQL, performed some updates and upgrades, and the system was working again. However, I encountered some problems when trying to reinstall PostgreSQL. It was a Monday morning, and I had a lot of work to do. While considering my options, I remembered [Docker](https://www.docker.com/).

Using a PostgreSQL database in a container is an easy way to run tests and development environments. I primarily use it for running automated tests, experimenting with changes in the console, running APIs locally to test code modifications, and so on.  Here are the steps to use a local PostgreSQL database in a container:

1. Install Docker if you haven't already. Follow the instructions on the [Docker website](https://docs.docker.com/get-docker/).
2. Choose the PostgreSQL image you want to use. You can find the images on [Docker Hub](https://hub.docker.com/_/postgres). Pay attention to the version you select, ensuring it's compatible with your project. Also, consider if you need any specific extensions like [PostGIS](https://postgis.net/), [pg_trgm](https://www.postgresql.org/docs/current/pgtrgm.html), [pgcrypto](https://www.postgresql.org/docs/current/pgcrypto.html) etc.
3. Prepare the command to run the container with the image. Be sure to set the necessary environment variables, such as the password, the port you want to use, and so on. You can find the environment variables on [Docker Hub](https://hub.docker.com/_/postgres) as well. Frameworks like Django or Ruby on Rails typically have a configuration file for database settings. Double-check that you're using the correct configuration in both the Docker command and the configuration file.

Here is an example of a command to run a PostgreSQL container with the PostGIS extension:

```bash
docker run -d --hostname localhost -e POSTGRES_PASSWORD=postgres -p 5432:5432 postgis/postgis:16-3.5
```

In the example above, the container will run in the background (`-d`). The password is `postgres`, and the port is `5432` (both in the container and on the host). The hostname is `localhost`, and the image is `postgis/postgis:16-3.5`.

You can use the `docker ps` command (or `docker container list`) to see the running containers.  You'll see something like this:

```bash
CONTAINER ID   IMAGE                    COMMAND                  CREATED         STATUS         PORTS                                       NAMES
d8f6f57868cb   postgis/postgis:16-3.5   "docker-entrypoint.s…"   2 seconds ago   Up 2 seconds   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   objective_turtle
```

Now it's time to use the container. Try some commands to create a database, a table, insert some data, etc. You can use commands from your framework to access the database. In Django, you can use `python manage.py dbshell` command to access the database. In Ruby on Rails, you can use `rails db` command to access the database. You can also use the `psql` command. To access the database from within the container, you can use `docker exec -it <container_name> bash` to enter in the container and then use `psql`.

When you're finished, you can stop the container with `docker stop <container_name>` command. You can remove the container with the `docker rm <container_name>` command.  You can remove the container with `docker rm <container_name>` and the image with `docker rmi <image_name>`.

Here the list of useful commands:

- `docker ps` or `docker container list`: list the running containers
- `docker stop <container_name>`: stop the container
- `docker rm <container_name>`: remove the container
- `docker rmi <image_name>`: remove the image
- `docker exec -it <container_name> bash`: enter in the container
- `psql`: access the database

And that's it! It's easy and fast and you can follow the same steps to use other databases like MySQL, MariaDB, etc. Have fun!

<table cellpadding="0" cellspacing="0" border="0" width="100%">
<tr><td align="center">
  <img src="https://live.staticflickr.com/65535/51936373085_d70f7e8117_b.jpg" width="450">
</td></tr>
</table>

>Image: [Container City ~ Explore #15 14-03-2022](https://openverse.org/image/f7b56405-8eb1-4122-b9b5-e3eab0a4564d). Source: OpenVerse - license Creative Commons 2.0.
