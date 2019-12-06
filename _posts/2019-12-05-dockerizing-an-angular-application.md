---
layout: post
published: true
title: Dockerizing an angular application
date: '2019-12-05'
---
In my last project, the UI team decided not to host its angular app in Docker... Challenge Accepted!!!.
I started looking into it, as proabbly for other projects this will be necessary.
### Things I need to have:
- Not need to rebuild the world every time. `npm i` usually takes too long.
- Be able to edit code and see it changed in the browser.
- Hit my APIs seamlessly. bonus point if I dont need to change my env, as I want to have a really easy to set up dev environment
- be able to access my app from my localhost, and my local browsers

### Nice things to have
- Run unit tests inside the container (or separate container) unsig `ng test`.
- Have a deployment Container with the angular AOT compilation.
- Multi-platform (win, mac, linux)

------------
## The proposed solution

In order to get this done, I created some docker files iside my project, that will drive the creation and deployment of the container.
### BUild the container
I created a `Dockerfile` just like this:
```
FROM node:12.2.0
WORKDIR /myApp
ENV PATH /myApp/node_modules/.bin:$PATH

COPY package.json /myApp/package.json
RUN npm install -g @angular/cli@7.3.9
RUN npm install

COPY . /myApp
CMD ng serve --watch=true --host=0.0.0.0
```
And the following `.dockerIgnore`
```
node_modules
.git
.gitignore
```
As Jack the Ripper would say: Lets take this piece by piece.

I am starting with a node 12 docker container, then I set my work dir to `myApp`, and add the `node_modules/.bin` to the path.

Then I copy the packake.json file to the container, in the myApp dir and run npm install to install first the angular CLI and then the app dependencies. I do this here so I can take advantage of cache.

Then I copy all the content of myApp to the /myApp directory, and run the `serve` command establishing the host as `0.0.0.0`, otherwise it would be localhost, and I don't think this would work when I expose the ports...

I set the .dockerignore with the node_modules and .git so they don't make their way to the container.

### Run the container
For running the container, I created a docker-compose file like this:
```
version: '3.4'
volumes:
  node_modules:

services:
  bmt-ui:
    image: ${DOCKER_REGISTRY}ng
    build:
      context: ./
      dockerfile: ./Dockerfile
    ports:
      - 4200:4200
    volumes:
      - ./src:/bmt-ui/src
      - node_modules:/bmt-ui/node_modules
```




