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


```
docker run --rm --name elastic --hostname elastic  -ti -p 9200:9200 romrsch/elastic:6.5
```
ответ elasticsearch 
```
curl -X GET localhost:9200
```

![альт](https://i.ibb.co/kJ8ZWm9/Screenshot-6.jpg)


![альт](https://i.ibb.co/M8wmmSq/Screenshot-15.jpg)

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

```
docker run --rm --name elastic --hostname elastic  -ti -p 9200:9200 romrsch/elastic:6.5
```
![альт](https://i.ibb.co/b6KtY72/Screenshot-4.jpg)

Создание индексов
```
curl -X PUT localhost:9200/ind-1 -H 'Content-Type: application/json' -d'{ "settings": { "number_of_shards": 1,  "number_of_replicas": 0 }}'
curl -X PUT localhost:9200/ind-2 -H 'Content-Type: application/json' -d'{ "settings": { "number_of_shards": 2,  "number_of_replicas": 1 }}'
curl -X PUT localhost:9200/ind-3 -H 'Content-Type: application/json' -d'{ "settings": { "number_of_shards": 4,  "number_of_replicas": 2 }}'    
```
![альт](https://i.ibb.co/rM8sgCX/Screenshot-7.jpg)


>Получите список индексов и их статусов, используя API

Список индексов:

```
curl -X GET 'http://localhost:9200/_cat/indices?v' 
```
![альт](https://i.ibb.co/nDMvgbd/Screenshot-8.jpg)

Статус индексов:
```
curl -X GET 'http://localhost:9200/_cluster/health/ind-1?pretty'
```
![альт](https://i.ibb.co/BKMRVXf/Screenshot-9.jpg)

```
curl -X GET 'http://localhost:9200/_cluster/health/ind-2?pretty'
```
![альт](https://i.ibb.co/RpLLjb6/Screenshot-10.jpg)

```
curl -X GET 'http://localhost:9200/_cluster/health/ind-3?pretty' 
```
![альт](https://i.ibb.co/1fjXdyD/Screenshot-11.jpg)

>Получите состояние кластера elasticsearch, используя API.

```
curl -XGET localhost:9200/_cluster/health/?pretty=true
```
![альт](https://i.ibb.co/28nTxK6/Screenshot-12.jpg)


>Как вы думаете, почему часть индексов и кластер находится в состоянии yellow?

Ответ: индексы `ind-2` и `ind-3` в статусе Yellow, т.к. мы их создали с указанием количеством реплик `1` и `2` , но у нас нет серверов куда можно эти данные синхронизировать. 



> Удалите все индексы.

```
curl -X DELETE 'http://localhost:9200/ind-1?pretty'
curl -X DELETE 'http://localhost:9200/ind-2?pretty'
curl -X DELETE 'http://localhost:9200/ind-3?pretty'
```
![альт](https://i.ibb.co/stJjkPH/Screenshot-13.jpg)

Проверка - Список индексов:
```
curl -X GET 'http://localhost:9200/_cat/indices?v'
```
![альт](https://i.ibb.co/QDGPVSF/Screenshot-14.jpg)




---

### задача 3

В данном задании вы научитесь:

создавать бэкапы данных
восстанавливать индексы из бэкапов
Создайте директорию {путь до корневой директории с elasticsearch в образе}/snapshots.

* Используя API [зарегистрируйте](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-register-repository.html#snapshots-register-repository) данную директорию как `snapshot repository` c именем `netology_backup`.

Приведите в ответе запрос API и результат вызова API для создания репозитория.

* Создайте индекс `test` с `0` реплик и `1` шардом и приведите в ответе список индексов.

* [Создайте snapshot](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-take-snapshot.html) состояния кластера `elasticsearch`.

Приведите в ответе список файлов в директории со snapshotами.

* Удалите индекс `test` и создайте индекс `test-2`. 
Приведите в ответе список индексов.

* [Восстановите](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-restore-snapshot.html) состояние кластера elasticsearch из snapshot, созданного ранее.

Приведите в ответе запрос к API восстановления и итоговый список индексов.

Подсказки:

* возможно вам понадобится доработать `elasticsearch.yml` в части директивы `path.repo` и перезапустить elasticsearch

***Ответ:***

> Используя API зарегистрируйте данную директорию как snapshot repository c именем netology_backup.

Итак, у нас конфигурация:
![альт](https://i.ibb.co/dcfqtZ0/Screenshot-16.jpg)

тогда
```
curl -XPOST localhost:9200/_snapshot/netology_backup?pretty -H 'Content-Type: application/json' -d'{"type": "fs", "settings": { "location":"/elasticsearch-7.14.0/snapshots" }}'
```
![альт](https://i.ibb.co/gSFqcWk/Screenshot-17.jpg)

> Приведите в ответе запрос API и результат вызова API для создания репозитория.

![альт](https://i.ibb.co/4p1yGb4/Screenshot-18.jpg)

![альт](https://i.ibb.co/TYfTjhx/Screenshot-19.jpg)


>Создайте индекс test с `0` реплик и `1` шардом и приведите в ответе список индексов.

```
curl -X PUT localhost:9200/test -H 'Content-Type: application/json' -d'{ "settings": { "number_of_shards": 1,  "number_of_replicas": 0 }}'
```
![альт](https://i.ibb.co/28PMC0x/Screenshot-20.jpg)

Результат:

![альт](https://i.ibb.co/x8VbSfD/Screenshot-21.jpg)


> Создайте snapshot состояния кластера elasticsearch.
```
curl -X PUT localhost:9200/_snapshot/netology_backup/elasticsearch?wait_for_completion=true
```
![альт](https://i.ibb.co/FDKZ8Cm/Screenshot-22.jpg)

> Приведите в ответе список файлов в директории со snapshotами.
```
docker exec -i -t 14655b8d9b85 bash

[elasticsearch@elastic elasticsearch]$ cd /elasticsearch-7.14.0/snapshots
[elasticsearch@elastic snapshots]$ pwd
/elasticsearch-7.14.0/snapshots
[elasticsearch@elastic snapshots]$ ls -la
total 56
drwxr-xr-x 1 elasticsearch elasticsearch  4096 Aug 16 20:42 .
drwxr-xr-x 1 elasticsearch elasticsearch  4096 Aug 16 16:56 ..
-rw-r--r-- 1 elasticsearch elasticsearch   831 Aug 16 20:42 index-0
-rw-r--r-- 1 elasticsearch elasticsearch     8 Aug 16 20:42 index.latest
drwxr-xr-x 4 elasticsearch elasticsearch  4096 Aug 16 20:42 indices
-rw-r--r-- 1 elasticsearch elasticsearch 27668 Aug 16 20:42 meta-Y7Kh2G2OTvCYhsPlZ7Q67g.dat
-rw-r--r-- 1 elasticsearch elasticsearch   440 Aug 16 20:42 snap-Y7Kh2G2OTvCYhsPlZ7Q67g.dat
[elasticsearch@elastic snapshots]$

```
>Удалите индекс test и создайте индекс test-2. Приведите в ответе список индексов.

Список индексов:
![альт](https://i.ibb.co/VQDC7Tm/Screenshot-23.jpg)

> Удаляем индекс `test`
```
curl -X DELETE 'http://localhost:9200/test?pretty'
```
![альт](https://i.ibb.co/bXwbM1N/Screenshot-24.jpg)

> создайте индекс test-2

```
curl -X PUT localhost:9200/test-2?pretty -H 'Content-Type: application/json' -d'{ "settings": { "number_of_shards": 1,  "number_of_replicas": 0 }}'
```
![альт](https://i.ibb.co/LYzvxj5/Screenshot-25.jpg)

> Список индексов
```
curl -X GET http://localhost:9200/_cat/indices?v
```

![альт](https://i.ibb.co/TRDTn30/Screenshot-26.jpg)

![альт](https://i.ibb.co/jbg3wgL/Screenshot-27.jpg)


---



