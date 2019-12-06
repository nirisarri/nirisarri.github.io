---
layout: post
published: true
title: Dockerizing an angular application
date: '2019-12-05'
tags:
  - Angular
  - Docker
  - WSL
---
In my last project, the UI team decided not to host its angular app in Docker...

![](https://media.giphy.com/media/d4zHnLjdy48Cc/giphy.gif)

I started looking into it, as probably for other projects this will be necessary.
### Things I need to have:
1. Not need to rebuild the world every time. `npm i` usually takes too long.
1. Be able to edit code and see it changed in the browser.
1. Hit my APIs seamlessly. bonus point if I don't need to change my env, as I want to have a really easy to set up dev environment
1. be able to access my app from my localhost, and my local browsers

### Nice things to have
1. Run unit tests inside the container (or separate container) using `ng test`.
1. Have a deployment Container with the angular AOT compilation.
1. Multi-platform (win, mac, linux)

------------
## The proposed solution

In order to get this done, I created some docker files inside my project, that will drive the creation and deployment of the container.
### Build the container
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

I am starting with a node V.12 docker container, then I set my work dir to `myApp`, and add the `node_modules/.bin` to the path.

Then I copy the `packake.json` file to the container, in the myApp dir and run `npm install` to install first the angular CLI and then the app dependencies. I do this here so I can take advantage of Docker's build cache.

Then I copy all the content of myApp to the `/myApp` directory, and run the `serve` command establishing the host as `0.0.0.0`, otherwise it would be `localhost`, and I don't think this would work when I expose the ports.
I also add the `--watch=true` flag so changes in the code will trigger new angular build. This will allow me to see real-time in the browser changes that I make in the code, without having to manually trigger a  container build. (more on this later)

I set the `.dockerignore` with the `node_modules` and `.git` so these heavy dirs don't make their way to the container.

### Run the container
For running the container, I created a docker-compose file like this:
```
version: '3.4'
volumes:
  node_modules:

services:
  ui:
    image: ${DOCKER_REGISTRY}ng
    build:
      context: ./
      dockerfile: ./Dockerfile
    ports:
      - 4200:4200
    volumes:
      - ./src:/myApp/src
      - node_modules:/myApp/node_modules
```

First I establish an anonymous volume where the `node_modules` will be stored. I do this so when recreating the container I don't lose them.
Then in the services, I establish the image name, the Dockerfile to use, expose the 4200 port and set the volumes I will use:
- The first one will map my `src` dir to the containers `src` dir, so I don't need to change the files inside the container, I can do that from the exterior.
-  The second maps the `node_modules` to the anonymous volume so they get persisted when the container stops.

## The results
Well... mixed results: the container gets created, and the node modules installed alright; the app gets built and server fine. I can check  #1, #3 and #4, but #2 is not working... I edit my file and the container does not see it changed 😒...

### More investigation
So, I found a couple of github issues that state that this is a docker for windows  [known issue](https://github.com/moby/moby/issues/30105). 
This is a bummer.
But wait! I have a WSL2 Ubuntu on windows distro!! (pretty awesome I must say). I also installed the latest edge Docker for windows that will use WSL2 instead of a hyper-V VM So, I copy the project into my Linux instance and rebuild everything from bash and do `docker-compose build && docker-compose up`... then I modify the files in my Linux (use VS code remote WSL to do that) and VOILA! 😎 changes in my code get tracked down in my container and the code is rebuilt! So I can *sorta* check #2 as well, and #3 from nice-to-have!
I know this i comes with a big payload, as you need
- Windows 10 Insiders (slow ring)
- WSL upgraded to V2
- Docker Edge Channel, with the experimental `Docker for WSL2 Engine` turned on.

But this will be out in the open really soon and from where I stand all is working really well!!

I will try to get the other 2 missing points and will post my results.
