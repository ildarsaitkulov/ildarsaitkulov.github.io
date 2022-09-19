---
layout: post
title: "Docker для самых маленьких. php-fpm + nginx + postgres"
img: child2.jpg
tags: [DOCKER, PHP]
description: В этой статье мы познакомимся с основами технологии docker, научимся настраивать сети, пользоваться volume-мами, узнаем что такое docker-compose и развернём простое веб-приложение с использованием php-fpm + nginx + postgres.
comments: true
---


## Docker для самых маленьких. php-fpm + nginx + postgres


Представьте что вы приходите в новый проект, и для начала вам нужно его развернуть. И как было бы круто иметь магическую команду "установи мне всё и где угодно", неплохо верно?
Решение есть - docker! В этой статье мы разберёмся с основами работы docker, и в конце развернём простое веб-приложение с использованием php-fpm + nginx + postgres.


**Docker** - это открытая платформа для разработки, доставки и запуска приложений. Базовым элементом которого являются контейнеры.

### Контейнер

Простыми словами **контейнер** - это некая песочница для запуска одного процесса на вашей машине которая изолирована от других процессов.

### Установка

Для начала нам нужно установить:
- [docker](https://docs.docker.com/engine/install/)
- [docker-compose](https://docs.docker.com/compose/install/)


### Сразу в бой!

Концепцию докера легче понять на практике. Например, давайте попробуем запустить http-сервер `nginx` на нашем компьютере.

```bash
docker run -p 8080:80 nginx:latest
```

Идём в браузер, открываем `127.0.0.1:8080`. И уже видим страницу приветствия nginx! И всё!

![enter image description here](https://ildarsaitkulov.github.io/assets/img/posts/docker-for-kids/nginx-hello.png "enter image title here")

Теперь разберёмся подробнее, что тут происходит.

Команда `docker run nginx:latest` делает следующее: скачивает докер образ `nginx:latest` (если ранее не скачивался) из [Docker Hub](https://hub.docker.com/) и запускает контейнер используя его.

### Докер образы и Docker Hub

**Докер образ (Docker image)** - шаблон для создания контейнеров. Представляет собой исполняемый пакет, содержащий все необходимое для запуска приложения: код, среду выполнения, библиотеки, переменные окружения и файлы конфигурации.

**Docker Hub** - это публичный репозиторий образов, куда может любой желающий загрузить его или скачать. На [странице nginx в Docker Hub](https://hub.docker.com/_/nginx?tab=tags) как раз можно найти тот самый наш образ [nginx:latest](https://hub.docker.com/layers/nginx/library/nginx/latest/images/sha256-b6a3554b020680898ad2d36f2211e03154766cb9841bf46f64d6259b12c3af5c?context=explore), latest - это tag который ссылается на самый свежий образ

Теперь мы разобрались что `nginx:latest` - это образ, который содержит всё для запуска nginx, и он лежит на Docker Hub. А что за опция `-p 8080:80` ? Ранее уже упоминалось, что процессы в контейнерах запускаются в изоляции от других процессов. И для того чтобы мы смогли обратиться к nginx, нужно пробросить порт на нашу машину `8080` - порт на нашей машине, `80` - это порт nginx внутри контейнера.

### Создание собственных образов

А давайте попробуем создать свой образ, взяв за основу `nginx:latest`?

Докер умеет создавать образ читая текстовые команды записанный в файл, этот файл называется **Dockerfile**

Пример простейшего Dockerfile:

{% highlight Dockerfile %}
FROM nginx:latest

RUN echo 'Hi, we are building custom docker image from nginx:latest!'

COPY nginx-custom-welcome-page.html /usr/share/nginx/html/index.html
{% endhighlight %}

- `FROM` - задаёт базовый (родительский) образ, должен идти первой командой
- `RUN` - выполняет команду и создаёт слой образа
- `COPY` - копирует в контейнер файлы

С помощью команды `COPY` мы заменяем стандартную welcome-страницу nginx-а на:
```html
<!DOCTYPE html>
<html>
<body>
<h1>Welcome to custom nginx page!</h1>
</body>
</html>
```

[Подробнее об этих и других командах тут](https://docs.docker.com/engine/reference/builder/)

И давайте создадим наш докер образ из Dockerfile:


```bash
$ docker build -t nginx_custom:latest -f /opt/src/docker-for-kids/dockerFiles/nginx-custom/Dockerfile /opt/src/docker-for-kids
Sending build context to Docker daemon  139.3kB
Step 1/3 : FROM nginx:latest
latest: Pulling from library/nginx
31b3f1ad4ce1: Pull complete 
fd42b079d0f8: Pull complete 
30585fbbebc6: Pull complete 
18f4ffdd25f4: Pull complete 
9dc932c8fba2: Pull complete 
600c24b8ba39: Pull complete 
Digest: sha256:0b970013351304af46f322da1263516b188318682b2ab1091862497591189ff1
Status: Downloaded newer image for nginx:latest
 ---> 2d389e545974
Step 2/3 : RUN echo 'Hi, we are building custom docker image from nginx:latest!'
 ---> Running in 05ffd060061f
Hi, we are building custom docker image from nginx:latest!
Removing intermediate container 05ffd060061f
 ---> 9ac62be4252a
Step 3/3 : COPY nginx-custom-welcome-page.html /usr/share/nginx/html/index.html
 ---> 704121601a45
Successfully built 704121601a45
Successfully tagged nginx_custom:latest

```

- `-t nginx_custom:latest` - это имя будущего образа, latest - это tag
- `-f /opt/src/docker-for-kids/dockerFiles/nginx-custom/Dockerfile` - путь до Dockerfile
- `/opt/src/docker-for-kids` - директория в контексте которого будет создан образ

И запустим:

```bash
$ docker run -p 8080:80 nginx_custom:latest
```
![enter image description here](https://ildarsaitkulov.github.io/assets/img/posts/docker-for-kids/nginx-hello-custom.png "enter image title here")

Замечательно, у нас удалось создать свой образ и запустить его!

### Docker compose

С ростом количества контейнеров поддержка их становится затруднительным. И справится с этим нам поможет Docker compose.

**Docker compose** - это инструмент для описания и запуска многоконтейнерных приложений. Для описания используется YAML файл.

{% highlight YAML %}
version: '3'

services:
  nginx:
    container_name: nginx-test # имя контейнера
    build: # создать образа из dockerFile
      context: . # путь в контексте которого будет создан образ
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

Из этого всего, думаю, стоит пояснить про volumes.

**Volumes** - это механизм для хранения данных вне контейнера, т.е на нашей хост машине.  И решает проблему совместного использования файлов.

И про **networks**. Подключая контейнеры к общей сети `test-network`, мы получаем возможность обращаться к нужному контейнеру по имени сервиса.

Все примеры, а так же исходники dockerFile-ов можно взять из репозитория на [https://github.com/ildarsaitkulov/docker-for-kids](https://github.com/ildarsaitkulov/docker-for-kids)

### Простое веб-приложение

Создадим самое простое веб-приложение, которое показывает нам сообщение об успешном подключении к базе данных.
Вместо адреса базы данных мы используем host=`postgres`, такое же имя нашего сервиса как и в YAML-файле

`index.php`
```php
<?php

try {
    $pdo = new \PDO("pgsql:host=postgres;dbname=postgres", 'postgres', 'mysecretpass');
    echo "Подключение к базе данных установлено! <br>";

    return;
} catch (PDOException $exception) {
    echo "Ошибка при подключении к базе данных<br><b>{$exception->getMessage()}</b><br>";
}
```


Теперь, для того чтобы создать все образы и запустить контейнеры нужно выполнить:

```bash
docker-compose up --build
```

<br>  
Выполним наш `index.php`

![enter image description here](https://ildarsaitkulov.github.io/assets/img/posts/docker-for-kids/index-pdo.png "enter image title here")

Увидим успешное соединение с базой данных!

Как и обещал одной лишь командой мы развернули все сервисы, и это можно сделать где угодно, нужен только докер, и везде у вас будет единое окружение!

В этой статье мы разобрались с основами работы с docker, развернули простое веб-приложение с использованием php-fpm + nginx + postgres.

Докер отличный инструмент для быстрого развертывания, деплоя, тестирования. Подробнее можно почитать на [официальном сайте](https://www.docker.com/).

Веб-приложение для самостоятельного запуска можно найти по ссылке [https://github.com/ildarsaitkulov/docker-for-kids](https://github.com/ildarsaitkulov/docker-for-kids)