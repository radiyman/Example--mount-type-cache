# Using --mount=type=cache with package managers
link to the official docker docs:  
https://github.com/moby/buildkit/blob/master/frontend/dockerfile/docs/syntax.md  
## General requirements for using the --mount=type=cache command:  
1) You can't bulid Dockerfile without using docker buildkit.  
2) You need to make sure that cache usage is enabled in the application you want to cache.  
3) You need to know the folder where the application cache is stored.  
4) A cache that is mounted on the same path (target) will be common to all docker images. If you need to fix this, use the additional options from the documentation.  

## composer  
Get cache path:  
```
composer config --global --list | grep cache
```
example:  
```
FROM php:7.3
# Install composer from official docker image
COPY --from=composer:1.10 /usr/bin/composer /usr/bin/composer
RUN composer global require hirak/prestissimo --no-plugins --no-scripts -q

WORKDIR /app
COPY composer.json composer.lock /app

RUN --mount=type=cache,target=/root/.composer/cache \
  composer install --no-plugins --no-scripts --no-autoloader

```
## npm  
Get cache path:  
```
npm config get cache --global
```
example:  
```
FROM node
WORKDIR /usr/src/app

COPY package.json package-lock.json /usr/src/app

RUN --mount=type=cache,target=/usr/src/app/.npm \
    npm set cache /usr/src/app/.npm && \
    npm ci
```
## pip  
Get cache path:  
```
python -m pip cache dir
```
example:  
```
# syntax = docker/dockerfile:experimental
FROM python:3.6-alpine
RUN --mount=type=cache,target=/root/.cache/pip \
  pip --cache-dir /root/.cache/pip install pyyaml
```
## apt  
Get cache path:  
```
apt-config shell CACHE Dir::Cache
```
example:  
```
# syntax = docker/dockerfile:1.3
FROM ubuntu
RUN rm -f /etc/apt/apt.conf.d/docker-clean; echo 'Binary::apt::APT::Keep-Downloaded-Packages "true";' > /etc/apt/apt.conf.d/keep-cache
RUN --mount=type=cache,target=/var/cache/apt --mount=type=cache,target=/var/lib/apt \
  apt update && apt-get --no-install-recommends install -y gcc
```
## yarn  
Yarn is configured to use a local cache by default. This is done to make it easier for you to store it as part of your repository.  
Before using --mount=type=cache, you need to switch yarn to use the global cache.  
link to the official yarn docs:  
https://yarnpkg.com/features/offline-cache  
Get cache path:  
```
yarn cache dir
```
example:  
```
# syntax = docker/dockerfile:1.3
FROM node
WORKDIR /usr/src/app
COPY package.json yarn.lock /usr/src/app
RUN  yarn install --pure-lockfile
```
## how to build docker images with using buildkit:  
link to the official docker docs:  
https://docs.docker.com/develop/develop-images/build_enhancements/  
docker build example:  
```
DOCKER_BUILDKIT=1 docker build .
```
docker-compose build example:  
```
DOCKER_BUILDKIT=1 COMPOSE_DOCKER_CLI_BUILD=1 docker-compose build
```
Note that docker-compose can use buildkit only from 1.25.0 version or higher.  
