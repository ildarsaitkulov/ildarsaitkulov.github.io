---
layout: post
title: "Docker для самых маленьких. php-fpm + nginx + postgres"
img: child2.jpg
tags: [DOCKER, PHP]
description: В этой статье мы познакомимся с основами технологии docker, научимся настраивать сети, пользоваться volume-мами, узнаем что такое docker-compose и развернём простое веб-приложение с использованием php-fpm + nginx + postgres.
comments: true
---



## Docker для самых маленьких. php-fpm + nginx + postgres
<br>
Представьте что вы начинаете работать над новым проектом и вам нужно его развернуть, а в современных реалиях это огромные приложения с множествами сервисов. И как было бы круто иметь магическую команду "установи мне всё и где угодно", неплохо верно?
Решение есть - docker!


**Docker** - это открытая платформа для разработки, доставки и запуска приложений. Базовым элементом которого являются контейнеры.
<br>
### Контейнер 

Простыми словами **контейнер** - это некая песочница для запуска одного процесса на вашей машине которая изолирована от других процессов.

### Установка

Для начала нам нужно установить:
- [docker](https://docs.docker.com/engine/install/)
- [docker-compose](https://docs.docker.com/compose/install/)


### Сразу в бой!

Концепцию докера легче понять на практике. Например, давайте попробуем запустить `nginx` на нашем компьютере:

{% highlight bash %}
docker run -p 8080:80 nginx:latest
{% endhighlight %}

<br>
Идём в браузер, открываем `127.0.0.1:8080`. И уже видим страницу приветствия nginx! И всё!

<p align="center">
    <img src="/assets/img/posts/docker-for-kids/nginx-hello.png" alt="nginx hello" style="max-width:100%;">  
</p>
<br>
Теперь разберёмся подробнее что тут происходит. Команда `docker run nginx:latest` запускает контейнер, а `nginx:latest` - это название **докер образа (Docker image)**.


### Докер образы и Docker Hub

**Докер образ** - шаблон для создания контейнеров. Представляет собой исполняемый пакет, содержащий все необходимое для запуска приложения: код, среду выполнения, библиотеки, переменные окружения и файлы конфигурации.

Откуда мы можем взять этот образ? Верно, из [Docker Hub](https://hub.docker.com/) - это публичный репозиторий образов, куда может любой желающий загрузить его и скачать. На [странице nginx в Docker Hub](https://hub.docker.com/_/nginx?tab=tags) как раз можно найти тот самый наш образ [`nginx:latest`](https://hub.docker.com/layers/nginx/library/nginx/latest/images/sha256-b6a3554b020680898ad2d36f2211e03154766cb9841bf46f64d6259b12c3af5c?context=explore)

Теперь мы разобрались что `nginx:latest` - это образ, который содержит всё для запуска nginx, и он лежит на Docker Hub. А что за опция `-p 8080:80` ? Ранее уже упоминалось, что процессы в контейнерах запускаются в изоляции от других процессов. И для того чтобы мы смогли обратиться к nginx, нужно пробросить порт на нашу машину `8080` - порт на нашей машине, `80` - это порт nginx внутри контейнера.

А давайте попробуем создать свой образ? Докер умеет создавать образ читая текстовые команды записанный в файл, этот файл называется **Dockerfile**

Пример простейшего Dockerfile:


{% highlight Dockerfile %}
FROM php:8.1.1-cli          #базовый образ

WORKDIR /var/www/hello.dev/ #рабочая директория

COPY hello.php ./hello.php  #скопировать файл hello.php из директории указанной при билде в рабочую директорию

CMD php -S 0.0.0.0:8080     #запустить встроенный веб-сервер php
{% endhighlight %}



- `FROM` - задаёт базовый (родительский) образ, должен идти первой командой
- `COPY` - копирует в контейнер файлы и директории
- `CMD` - команда для запуска контейнера (должна быть только одна)
- `ADD` - копирует файлы, директории или скачивает по ссылке и добавляет их в файловую систему образа
- `RUN` - выполняет команду и создаёт слой образа. Используется для установки в контейнер пакетов
- `WORKDIR` - устанавливает рабочую директорию для последующих инструкций

[Подробнее об этих и других командах тут](https://docs.docker.com/engine/reference/builder/)

И давайте сбилдим наш докер образ из Dockerfile:

{% highlight bash %}
$ docker build -t php-hello-server -f /opt/src/docker-for-kids/dockerFiles/php-hello-server/Dockerfile /opt/src/docker-for-kids
Sending build context to Docker daemon    105kB
Step 1/4 : FROM php:8.1.1-cli
8.1.1-cli: Pulling from library/php
a2abf6c4d29d: Pull complete
c5608244554d: Pull complete
2d07066487a0: Pull complete
1b6dfaf1958c: Pull complete
40f5e6ee20ce: Pull complete
718b027f9905: Pull complete
3bf01f3e893c: Pull complete
af85a153f85f: Pull complete
e052a88c20f6: Pull complete
Digest: sha256:444ba13f11741642a2692430f6678d47fb028442160ec9a5cfa9da7d3c0a9e07
Status: Downloaded newer image for php:8.1.1-cli
---> 13b9b1961ba3
Step 2/4 : WORKDIR /var/www/hello.dev/
---> Running in 266b47648946
Removing intermediate container 266b47648946
---> 69d85039cbd9
Step 3/4 : COPY hello.php ./hello.php
---> a58ebaeb4dbf
Step 4/4 : CMD php -S 0.0.0.0:8080
---> Running in 8258159599b7
Removing intermediate container 8258159599b7
---> 82c701bf6aea
Successfully built 82c701bf6aea
Successfully tagged php-hello-server:latest
{% endhighlight %}

- `-t php-hello-server` - это тэг будущего образа
- `-f /opt/src/docker-for-kids/dockerFiles/php-hello-server/Dockerfile` - путь до Dockerfile
- `/opt/src/docker-for-kids` - директория в контексте которого будет сбилжен образ

И запустим:

{% highlight bash %}
$ docker run -p 8080:8080   php-hello-server
[Sun Dec 26 12:44:35 2021] PHP 8.1.1 Development Server (http://0.0.0.0:8080) started
{% endhighlight %}

<p>
    <img src="/assets/img/posts/docker-for-kids/php-server-hello.png" alt="php-server-hello" style="max-width:100%;">  
</p>

Замечательно, у нас удалось создать свой образ и запустить его!

### Docker compose

С ростом количества контейнеров поддержка их становится затруднительным. И справится с этим нам поможет Docker compose.

**Docker compose** - это инструмент для описания и запуска многоконтейнерных приложений. Для описания используется YAML файл.

{% highlight YAML %}
version: '3'

services:
  nginx:
    container_name: nginx-test # имя контейнера
    build: # билд образа из dockerFile
      context: . # путь в контексте которого будет сбилжен образ
      dockerfile: ./dockerFiles/nginx/Dockerfile # путь до dockerFile из которого будет собран образ
    ports: # проброс портов
      - "80:80"
    networks: # имя сети к котором будет подключен контейнер
      - test-network
    depends_on: # запуск контейнера зависит от
      - php-fpm
    volumes: #  монтирование директорий, директория-на-хост-машине:директория-в-докере
      - ./:/var/www/hello.dev/
  php-fpm:
    container_name: php-fpm-test
    build:
      context: .
      dockerfile: ./dockerFiles/php-fpm/Dockerfile
    networks:
      - test-network
    volumes:
      - ./:/var/www/hello.dev/
  postgres:
    container_name: postgres-test
    image: postgres:14.1-alpine # тэг образа из https://hub.docker.com/
    environment:
      POSTGRES_PASSWORD: mysecretpass # переменные окружения которые использует контейнер
    networks:
      - test-network
networks: # явно объявленные сети
  test-network:
    driver: bridge
{% endhighlight %}

Из этого всего думаю стоит пояснить про volumes.

**Volumes** - это механизм для хранения данных вне контейнера, т.е на нашей хост машине.  И решает проблему совместного использования файлов.

И про **networks**. Подключая контейнеры к общей сети `test-network`, мы получаем возможность обращаться к нужному контейнеру имени сервиса.

К примеру в `index.php`
{% highlight php %}
<?php
$pdo = new \PDO("pgsql:host=postgres;dbname=postgres", 'postgres', 'mysecretpass');
var_dump($pdo);
{% endhighlight %}

Вместо адреса базы данных мы используем host=`postgres`


Теперь для того чтобы сбилдить все образы и запустить контейнеры нужно выполнить:

{% highlight bash %}
docker-compose up --build
{% endhighlight %}


Выполним наш `index.php`
<p>
    <img src="/assets/img/posts/docker-for-kids/index-pdo.png" alt="nginx hello" style="max-width:100%;">  
</p>

И увидим успешное соединение с базой данных!

Как и обещал в одной лишь командой мы развернули все сервисы, и это можно сделать где угодно, нужен только докер, и везде у вас будет единое окружение!

Докер отличный инструмент для быстрого развертывания, деплоя, тестирования. Подробнее можно почитать на [официальном сайте](https://www.docker.com/).

Примеры из статьи, а так же исходники dockerFile-ов можно взять из репозитория на [https://github.com/ildarsaitkulov/docker-for-kids](https://github.com/ildarsaitkulov/docker-for-kids)