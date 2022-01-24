---
title: BrowserSync with Laravel Mix and Docker
date: 2021-07-04 20:40:39
tags:
- Docker
- BrowserSync
- Javascript
- Ecmascript
- Tooling

autoThumbnailImage: yes
coverImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_1400/v1555352082/posts/south-africa-penguins.jpg
coverSize: partial
coverMeta: in
metaAlignment: center
thumbnailImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_280/v1555352082/posts/south-africa-penguins.jpg
thumbnailImagePosition: right
---

I've pretty much switched to using docker for everything when doing local development now. 

My set up involves using a `docker-compose.yml` file for the core application dependencies that can be run in production and a `docker-compose.override.yml` file for development. This override file is mostly a load of "support" containers that get used in development, things like php, artisan, npm as well as things like mysql and redis that would be handled elsewhere in production.

Since switching to this development model I've just accepted not getting around to getting BrowserSync working· This weekend whilst battling some other issues my fingers got tired of pressing ctrl+r every few seconds so I figured I'd give myself a win by trying to tackle this.

The [Laravel mix docs](https://laravel-mix.com/docs/main/browsersync) have a brief section on BrowserSync but theres not enough to get BrowserSync working for our config.

<!--more-->

### Docker Compose Config

Lets start by looking the support container im using for npm commands:

```yaml
// docker-compose.override.yml
  npm:
    image: node:14-alpine
    container_name: my-app-npm
    volumes:
      - .:/var/www/html
    working_dir: /var/www/html
    entrypoint: [ 'npm' ]
    networks:
      - default
```

This allows me to run commands like:

```bash
docker-compose run  --rm npm run dev
```

I actually have the first part aliased to save some keystrokes:

```
// .zshrc / .bashrc
alias dcr='docker-compose run --rm '
```

The first change we need to make here is to get this container to expose BrowserSync's default port (3000) and its UI (3001).


```yaml
// docker-compose.override.yml
  npm:
    image: node:14-alpine
    container_name: my-app-npm
    volumes:
      - .:/var/www/html
    working_dir: /var/www/html
    entrypoint: [ 'npm' ]
    networks:
      - default
    # Add these lines:
    ports:
      - 3000:3000
      - 3001:3001
```

### Laravel Mix Config

We'll need to pass an config object to Mix's BrowserSync plugin. However before we do lets quickly review the `app` container definition from the `docker-compose.yml` file:

```yaml
services:
  app:
    image: my-app
    container_name: my-app-api
    networks:
      - default
    build:
      context: .
      dockerfile: docker/Dockerfile
    env_file:
      - .env
    ports:
      - ${APP_PORT:-80}:80
    volumes:
      - .:/var/www/html
```

Here we can see a service called "app" with a `container_name` of "my-app-api". Normally when referencing other services within the docker network we'd use the service name but for this we'll be using the `container_name`.

Back to the Laravel Mix config, add the following:

```javascript
// webpack.mix.js
	mix.browserSync({
         // fixes pagination urls otherwise they get re-written to use the service `container_name`...
		host: 'localhost',
        // service container_name...
		proxy: 'my-app-api', 
        // matches the port number exposed earlier...
		port: 3000, 
        open: false,
	});
```

### Running

We're almost done. Lets restart our application so the new ports are exposed:

```bash
docker-compose down
docker-compose up -d
```

Before running `npm run watch` we need to pass an additional argument to the docker-compose run command - "--service-ports".

When start containers using "run" the ports defined actually don't get mapped. This is so there aren't port collisions when multiple containers are "run". The `--serivice-ports` flag does map those ports which in this case we do want since we will be opening the browser at localhost:3000 [Reference.](https://docs.docker.com/compose/reference/run/)

So our final commands look like this:


```bash
docker-compose up -d
docker-compose run --rm --service-ports npm run watch
```

Or for lazy me:

```bash
dc up -d
dcr --service-ports npm run watch
```

I actually got even lazier and made a new `dcrs` alias ¯\\_(ツ)_/¯.