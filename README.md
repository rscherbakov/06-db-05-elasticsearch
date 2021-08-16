# netology-6.5
## Домашнее задание к занятию "6.5. Elasticsearch"

### Задача 1

В этом задании вы потренируетесь в:

* установке elasticsearch
* первоначальном конфигурировании elastcisearch
* запуске elasticsearch в docker
* Используя докер образ [centos:7](https://hub.docker.com/_/centos) как базовый и [документацию по установке и запуску Elastcisearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/targz.html):

Составьте Dockerfile-манифест для elasticsearch

* соберите docker-образ и сделайте push в ваш docker.io репозиторий
* запустите контейнер из получившегося образа и выполните запрос пути `/` c хост-машины

Требования к elasticsearch.yml:

* данные *path* должны сохраняться в `/var/lib`
* имя ноды должно быть `netology_test`

В ответе приведите:

* текст Dockerfile манифеста
* ссылку на образ в репозитории dockerhub
* ответ elasticsearch на запрос пути `/` в `json` виде

Подсказки:

Возможно вам понадобится установка пакета `perl-Digest-SHA` для корректной работы пакета `shasum`

При сетевых проблемах внимательно изучите кластерные и сетевые настройки в `elasticsearch.yml`
при некоторых проблемах вам поможет `docker` директива `ulimit`

`elasticsearch` в логах обычно описывает проблему и пути ее решения.

Далее мы будем работать с данным экземпляром elasticsearch.

***Ответ:***

Dockerfile

```
FROM centos:7

LABEL Netology Homework 6.5:ElasticSearch
MAINTAINER Scherbakov Roman <romrsch@gmail.com>

ENV PATH=/usr/lib:/usr/lib/jvm/jre-11/bin:$PATH

RUN yum install java-11-openjdk -y
RUN yum install wget -y
RUN yum install perl-Digest-SHA -y

RUN  wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.14.0-linux-x86_64.tar.gz \
    &&  wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.14.0-linux-x86_64.tar.gz.sha512

RUN shasum -a 512 -c elasticsearch-7.14.0-linux-x86_64.tar.gz.sha512  \ 
    && tar -xzf elasticsearch-7.14.0-linux-x86_64.tar.gz \
    && yum upgrade -y
    
ADD elasticsearch.yml /elasticsearch-7.14.0/config/
ENV JAVA_HOME=/elasticsearch-7.14.0/jdk/
ENV ES_HOME=/elasticsearch-7.14.0
RUN groupadd elasticsearch \
    && useradd -g elasticsearch elasticsearch
    
RUN mkdir /var/lib/logs \
    && chown elasticsearch:elasticsearch /var/lib/logs \
    && mkdir /var/lib/data \
    && chown elasticsearch:elasticsearch /var/lib/data \
    && chown -R elasticsearch:elasticsearch /elasticsearch-7.14.0/
RUN mkdir /elasticsearch-7.14.0/snapshots &&\
    chown elasticsearch:elasticsearch /elasticsearch-7.14.0/snapshots
    
USER elasticsearch
CMD ["/usr/sbin/init"]
CMD ["/elasticsearch-7.14.0/bin/elasticsearch"]

```
![альт](https://i.ibb.co/Jk1XZPN/Screenshot-5.jpg)

```
docker build -t romrsch/elastic:6.5 .
docker login
docker image push romrsch/elastic:6.5
```
![альт](https://i.ibb.co/yFFRwgB/Screenshot-3.jpg)

Ссылка на образ в репозитории docker.hub

https://hub.docker.com/repository/docker/romrsch/elastic

![альт](https://i.ibb.co/C61fVJG/Screenshot-2.jpg)


---
### Задача 2

В этом задании вы научитесь:

* создавать и удалять индексы
* изучать состояние кластера
* обосновывать причину деградации доступности данных

Ознакомтесь с [документацией](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html) и добавьте в `elasticsearch` 3 индекса, в соответствии со таблицей:


| Имя       | Количество реплик   | Количество шард |
| ------------- |:---------------:| ---------------:|
| ind-1         | 0               |  1	    		|
| ind-2         | 1               |  2		    	|
| ind-3         | 2               |  4  </head>     |


Получите список индексов и их статусов, используя API и приведите в ответе на задание.

Получите состояние кластера elasticsearch, используя API.

Как вы думаете, почему часть индексов и кластер находится в состоянии yellow?

Удалите все индексы.

Важно

При проектировании кластера elasticsearch нужно корректно рассчитывать количество реплик и шард, иначе возможна потеря данных индексов, вплоть до полной, при деградации системы.

***Ответ:***

---

### задача 3

В данном задании вы научитесь:

создавать бэкапы данных
восстанавливать индексы из бэкапов
Создайте директорию {путь до корневой директории с elasticsearch в образе}/snapshots.

Используя API [зарегистрируйте](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-register-repository.html#snapshots-register-repository) данную директорию как `snapshot repository` c `именем netology_backup`.

Приведите в ответе запрос API и результат вызова API для создания репозитория.

Создайте индекс test с 0 реплик и 1 шардом и приведите в ответе список индексов.

[Создайте snapshot](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-take-snapshot.html) состояния кластера `elasticsearch`.

Приведите в ответе список файлов в директории со snapshotами.

Удалите индекс `test` и создайте индекс `test-2`. Приведите в ответе список индексов.

[Восстановите](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-restore-snapshot.html) состояние кластера elasticsearch из snapshot, созданного ранее.

Приведите в ответе запрос к API восстановления и итоговый список индексов.

Подсказки:

* возможно вам понадобится доработать `elasticsearch.yml` в части директивы `path.repo` и перезапустить elasticsearch

***Ответ:***

---



