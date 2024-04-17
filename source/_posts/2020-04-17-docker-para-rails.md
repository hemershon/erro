---
layout: post
title: "Docker para Rails"
permalink: docker-para-rails
date: 2020-04-17 13:44:04
comments: true
description: "Docker para Rails"
keywords: ""
image: "/images/img/docker.png"
categories:
  - Rails
  - Docker
  - Ruby

tags:

  - iniciante
  - docker
  - rails
---
# Docker com Ruby on Rails
*Escrito por [Hemershon Silva](https://hemershon.com/)*

## Como dockerizar sua aplicação

Esse tutorial é para ajudar quem tem dificuldade em montar seu ambiente rails no docker, aqui vou abordar técnicas de como usar o docker e docker-compose que ambos trabalham em plena harmonia.

**Resumo sobre o docker**

Docker é um software contêiner da empresa Docker, Inc, que fornece uma camada de abstração e automação para virtualização de sistema operacional no Windows, Macos e Linux.

Requisitos:

Docker instalado na sua maquina [Instalação do docker](https://www.edivaldobrito.com.br/docker-no-ubuntu/)
Docker-compose instalado[Instalação do docker-compose](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-ubuntu-20-04-pt)


## Definindo um projeto

Comece configurando os arquivos necessários para construir o aplicativo que será executado dentro de um [contêiner](https://www.redhat.com/pt-br/topics/containers/what-is-docker) do Docker suas dependências.

Os arquivos são:
>Dockerfile

>docker-compose.yml

>Gemfile

>Gemfile.lock

Primeiro passo vamos criar o arquivo [Dockerfile](https://www.alura.com.br/artigos/desvendando-o-dockerfile)

```
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
```
Isso colocará o código do seu aplicativo dentro de uma imagem que constrói um contêiner com Ruby, bundler e todas as suas dependências dentro dele.

agora vamos criar o [Gemfile](https://jtemporal.com/gemfile/)

```

source 'https://rubygems.org'
gem 'rails', '~>6'
```

como todo projeto rails precisamos também do Gemfile.lock
```
touch Gemfile.lock
```

[Docker-compose](https://imasters.com.br/banco-de-dados/docker-compose-o-que-e-para-que-serve-o-que-come#:~:text=Docker%20Compose%20%C3%A9%20o%20orquestrador,mas%20os%20maestros%20somos%20n%C3%B3s!)

```

version: "3.9"
services:
  db:
    image: postgres
    volumes:
      - ./tmp/db:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: password
  web:
    build: .
    command: bash -c "rm -f tmp/pids/server.pid && bundle exec rails s -p 3000 -b '0.0.0.0'"
    volumes:
      - .:/seuapp
    ports:
      - "3000:3000"
    depends_on:
      - db
```

Agora vamos montar o projeto 

```
docker-compose run --no-deps web rails new . --force --database=postgresql
```

Primeiro, o Compose cria a imagem para o webserviço usando o Dockerfile. O --no-deps diz ao Compose para não iniciar os serviços vinculados. Em seguida, ele é executado rails newdentro de um novo contêiner, usando essa imagem. Uma vez feito isso, você deve ter gerado um novo aplicativo.

Você irá precisar de uma permissão para manipular os arquivos

```
sudo chown -R $USER:$USER .
```

Agora você está com a estrutura montada com todos os arquivos do rails, agora precisamos reconstruir a estrutura para usamos com o docker.

```
docker-compose build
```

Configurações de Banco de dados

```

default: &default
  adapter: postgresql
  encoding: unicode
  host: db
  username: postgres
  password: password
  pool: 5

development:
  <<: *default
  database: myapp_development


test:
  <<: *default
  database: myapp_test
```

Agora você pode inicializar o aplicativo com

```
docker-compose up
```

se estiver tudo ok, essa é a mensagem que vai aparecer no terminal

```

rails_db_1 is up-to-date
Creating rails_web_1 ... done
Attaching to rails_db_1, rails_web_1
db_1   | PostgreSQL init process complete; ready for start up.
db_1   |
db_1   | 2018-03-21 20:18:37.437 UTC [1] LOG:  listening on IPv4 address "0.0.0.0", port 5432
db_1   | 2018-03-21 20:18:37.437 UTC [1] LOG:  listening on IPv6 address "::", port 5432
db_1   | 2018-03-21 20:18:37.443 UTC [1] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
db_1   | 2018-03-21 20:18:37.726 UTC [55] LOG:  database system was shut down at 2018-03-21 20:18:37 UTC
db_1   | 2018-03-21 20:18:37.772 UTC [1] LOG:  database system is ready to accept connections
```

Agora vamos criar o banco de dados

```

docker-compose run web rake db:create
```

mensagem que vai aparecer

```

Starting rails_db_1... done
Created database 'seuapp_development'
Created database 'seuapp_test'
```

depois vá para o navegador e digite 
```

localhost:3000 
```

Referência estão nos links

[O que é Docker](https://www.redhat.com/pt-br/topics/containers/what-is-docker)

[Docker](https://docs.docker.com/get-docker/)

[Docker-compose](https://imasters.com.br/banco-de-dados/docker-compose-o-que-e-para-que-serve-o-que-come#:~:text=Docker%20Compose%20%C3%A9%20o%20orquestrador,mas%20os%20maestros%20somos%20n%C3%B3s!)

[Gemfile](https://jtemporal.com/gemfile/)

[Dockerfile](https://www.alura.com.br/artigos/desvendando-o-dockerfile)