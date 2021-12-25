---
layout: post
title: "Docker для самых маленьких"
img: child2.jpg
tags: [DOCKER, PHP]
description: В этой статье мы познакомимся с основами технологии docker, научимся настраивать сети, пользоваться volume-мами, узнаем что такое docker-compose и развернём простое веб-приложение с использованием php-fpm + nginx + postgres.
comments: true
---



## Docker для самых маленьких. php-fpm + nginx + postgres
<br>
Представьте что вы начинаете работать над новым проектом и вам нужно его развернуть, а в современных реалиях это огромные приложения с множествами сервисов. И как было бы круто иметь магическую кнопку "установи мне всё и где угодно", неплохо верно?
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
sudo docker run -p 8080:80 nginx:latest
{% endhighlight %}

<br>
Идём в бразур, открываем `127.0.0.1:8080`. И уже видим страницу приветсвия nginx! И всё!

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
FROM ubuntu:20.04

COPY someFile.log /tmp/someFile.log

CMD tail -f /tmp/someFile.log
{% endhighlight %}


- `FROM` - задаёт базовый (родительский) образ, должен идти первой командой
- `COPY` - копирует в контейнер файлы и директории
- `CMD` - команда для запуска контейнера (должна быть только одна)
- `ADD` - копирует файлы, директории или скачивает по ссылке и добавляет их в файловую систему образа
- `RUN` - выполняет команду и создаёт слой образа. Используется для установки в контейнер пакетов
- `WORKDIR` - устанавливает рабочую директорию для последующих инструкций

[Подробнее об этих и других командах тут](https://docs.docker.com/engine/reference/builder/)

`tail -f /tmp/someFile.log` - выводит содержимое конца файла 

И давайте сбилдим наш докер образ из Dockerfile:

{% highlight bash %}
$ sudo docker build -t taillogfile -f dockerFiles/test/Dockerfile /opt/src/test_docker
Sending build context to Docker daemon  20.48kB
Step 1/3 : FROM ubuntu:20.04
---> ba6acccedd29
Step 2/3 : COPY someFile.log /tmp/someFile.log
---> 58029edbe9db
Step 3/3 : CMD tail -f /tmp/someFile.log
---> Running in 98beed0fec93
Removing intermediate container 98beed0fec93
---> 35847abc8606
Successfully built 35847abc8606
Successfully tagged taillogfile:latest

{% endhighlight %}

- `-t taillogfile` - это тэг будущего образа
- `-f dockerFiles/test/Dockerfile` - путь до Dockerfile
- `/opt/src/test_docker` - директория в контексте которого будет сбилжен образ

И запустим

{% highlight bash %}
$ docker run taillogfile
Hi
{% endhighlight %}

Замечательно, у нас удалось создать свой образ и запустить его!

Но что будет если в конец нашего лог файла someFile.log добавить что-то?
Например
{% highlight bash %}
echo "foobar" >> /opt/src/test_docker/someFile.log
{% endhighlight %}

Мы не увидим изменений, но почему?
{% highlight bash %}
$ docker run taillogfile
Hi
{% endhighlight %}

Потому что, лог файл someFile.log был скопирован во время билда единожды и теперь в контейнере он лежит в неизменном состояинии.
Как быть? В этом нам помогу **Volumes**

### Volumes

Volumes - это механизм для хранения данных вне контейнера, т.е на нашей хост машине.

<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>