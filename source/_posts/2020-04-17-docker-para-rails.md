---
layout: post
title: "Docker para Rails"
permalink: docker-para-rails
date: 2020-04-17 13:44:04
comments: true
description: "Docker para Rails"
keywords: ""
categories:
  - Rails
  - Docker
  - Ruby

tags:

---

# Docker com Ruby on Rails
*Escrito por Hemershon Silva*

## Como dockerizar sua aplicação

Este tutorial é para ajudar quem tem dificuldade em montar seu ambiente Rails no Docker, aqui vou abordar técnicas de como usar o Docker e Docker Compose que ambos trabalham em plena harmonia.

### Resumo sobre o Docker

Docker é um software contêiner da empresa Docker, Inc, que fornece uma camada de abstração e automação para virtualização de sistema operacional no Windows, Macos e Linux.

### Requisitos:

- Docker instalado na sua maquina [Instalação do docker](https://www.edivaldobrito.com.br/docker-no-ubuntu/)
- Docker-compose instalado [Instalação do docker-compose](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-ubuntu-20-04-pt)

## Definindo um projeto

Comece configurando os arquivos necessários para construir o aplicativo que será executado dentro de um contêiner do Docker suas dependências.

Os arquivos são:

- Dockerfile
- docker-compose.yml
- Gemfile
- Gemfile.lock

Primeiro passo vamos criar o arquivo Dockerfile

```Dockerfile
FROM ruby:3.0.0
RUN apt-get update -qq && apt-get install -y nodejs postgresql-client
RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - && \
  echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list && \
  apt-get update -qq && apt-get install -y nodejs postgresql-client vim && \
  apt-get install -y yarn && \
  apt-get install -y imagemagick && \
  apt-get install -y libvips-tools && \
  apt-get install -y locales

# Instala nossas dependencias
RUN apt-get update && apt-get install -qq -y --no-install-recommends \
nodejs yarn build-essential libpq-dev imagemagick git-all nano

WORKDIR /seuapp
COPY Gemfile /seuapp/Gemfile
COPY Gemfile.lock /seuapp/Gemfile.lock
RUN bundle update mimemagic
RUN bundle install
COPY . /seuapp
