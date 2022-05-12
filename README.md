# Using --mount=type=cache with package managers
link to the official docker docs:  
https://github.com/moby/buildkit/blob/master/frontend/dockerfile/docs/syntax.md  
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
