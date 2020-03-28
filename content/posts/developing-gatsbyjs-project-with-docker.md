---
title: "Developing GatsbyJS Project with Docker"
date: 2020-03-28T14:35:17+09:00
lastmod: 2020-03-28T14:35:17+09:00
summary: "Developing GatsbyJS Project with Docker"
categories:
    - "Programming"
tags:
    - "GatsbyJS"
    - "docker"
    - "docker-compose"
author: "STomoya"
---

# Developing GatsbyJS Project with Docker

I started to play around with GatsbyJS so I'll write down some of the things I learned when developing projects with docker.  
The [repository is here](https://github.com/STomoya/gatsby-playground).

## Initualization

In GatsbyJS, there are ways that makes it simple to creat a project.
Follow the [official tutorial](https://www.gatsbyjs.org/docs/) for making a project.
I used `https://github.com/gatsbyjs/gatsby-starter-hello-world` for initualizing my project.

Then I erased the created `node-modules` folder because I will do the development on a docker container.
`node-modules` folder is *very* heavy so if you aren't using it, you should erase it...

## Docker

Next I started creating a Docker container which lets me develop my website.  
[This](https://yopinoji.com/docker-for-gatsby-js) is the site I referenced. (Its Japanese. Sorry...)

I created a `./docker/Dockerfile` and `docker-compose.yml` inside the project folder.

In my `Dockerfile`:

```
FROM node:13-alpine

RUN apk add --no-cache python make g++ && \
    apk add vips-dev fftw-dev --no-cache --repository http://dl-3.alpinelinux.org/alpine/edge/community --repository http://dl-3.alpinelinux.org/alpine/edge/main && \
    rm -fR /var/cache/apk/*

WORKDIR /usr/src

COPY ./package.json package.json

RUN npm install

EXPOSE 8000
```

The base image I am using is `node:13-alpine`.
If you use the alpine linux, you need to install vips-dev.
So the first `RUN` is for installing dependencies.
Then, after changing the working directory I copy the package.json and install the dependencies of the website by `npm install`.
Finally, expose the `8000` port, which is the default port for the development server of GatsbyJS.
If you set your development server port to something else, you should expose the specific port.

*You should expose the port in the `Dockerfile` and not in the `docker-compose.yml` file. It will not work.*

Then in my `docker-compose.yml`:

```
version: '3'

volumes:
    node_modules:

services:
  web:
    build:
      context: .
      dockerfile: ./docker/Dockerfile
    ports:
      - "8000:8000"
    volumes:
      - node_modules:/usr/src/node_modules
      - .:/usr/src
    environment:
      - NODE_ENV=development
    command: npm run develop
```

The service `web` is the main part.
I put the Dockerfile in a folder call `docker`, so I declared that in the `build`.
The `8000` port is forward to the container's `8000` port.
Then the volumes current folder is binded to the container's `/usr/src` folder.
Next I have the `enviornment` for the NodeJS.
And the command to run the development server.

I didn't need the `node_modules` to be in my local folder so I did things to exclude this folder, which is the `volumes` on the top and first line of the `volumes` in the `web`.
This is not necessary if you don't care much about having heavy folders on your local folders.

So, now you can run

```console
$ docker-compose build
```

to build the container, and

```console
$ docker-compose up
```

to start the development server.  
Visit `localhost:8000` to see it running.

To add dependencies, run

```console
$ docker-compose run --rm npm install <package>
```

And to build, run

```console
$ docker-compose run --rm npm run build
```

and the files will be built inside the `public` folder.

## Development

In GatsbyJS, the changes you made to your files are detected automatically and they will run the build again to update the contents.
In order to find the changes, the development server keeps on running unless you hit `Ctrl+C` to stop the development server.

Also, you don't need to care about the server stopping when you have a bug in your program.
It wont stop even if you have a bug that makes your site unable to move.
It even shows the error message on the browser!
Wonderful:)

I recognized that there are several tasks in the build process, and they automatically detect where to build from by the changes that were made.
Inteligent:)

For the actual coding, see the [official tutorial](https://www.gatsbyjs.org/docs/).

## Overall

Thats it.  
I might write about the GatsbyJS itself, but many articles are on the Internet so it's better looking at those pages.