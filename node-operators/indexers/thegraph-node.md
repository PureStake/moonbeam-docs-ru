---
title: Нода Graph
description: Создание API с использованием протокола индексирования Graph на Moonbeam
---

# Запуск ноды Graph на Moonbeam

![The Graph Node on Moonbeam](/images/node-operators/indexer-nodes/the-graph/the-graph-node-banner.png)

## Вступление {: #introduction } 

Узел Graph получает инофрмацию о событиях из блокчейна для детерминированного обновления хранилища данных, которые затем могут быть запрошены через конечную точку GraphQL.

Есть два способа настроить ноду Graph: Вы можете использовать Docker, для запуска образа, который хранит в себе весь необходимый набор программного обеспечения, или Вы можете запустить её [используя Rust](https://github.com/graphprotocol/graph-node). Шаги, описанные в этом руководстве, относятся только к использованию Docker, так как это более удобно, и Вы можете очень быстро настроить Graph ноду используя этот метод установки.

!!! примечание  внимание
    Шаги, описанные в этом руководстве, были протестированы в Ubuntu 18.04, и в среде MacOs, поэтому их необходимо будет соответствующим образом адаптировать если Вы планируете использовать другие системы.

## Проверка требований {: #checking-prerequisites } 

Прежде чем приступить к настройке Graph ноды, в Вашей системе должны быть установлены следующие компоненты:

 - [Docker](https://docs.docker.com/get-docker/)
 - [Docker Compose](https://docs.docker.com/compose/install/)
 - [JQ](https://stedolan.github.io/jq/download/)

Кроме того, у Вас должен быть узел, работающий с включенной опцией`--ethapi=trace`. В настоящее время Вы можете запускать два разных типа узлов:

 - **Узел разработки Moonbeam** — запустите свой собственный экземпляр Moonbeam в своей частной среде. Для этого Вы можете следовать [этому руководству](/getting-started/local-node/setting-up-a-node/). Обязательно проверьте [раздел расширенных флагов](/getting-started/local-node/setting-up-a-node/#advanced-flags-and-options)
 - **Узел Moonbase Alpha** — запустить полноценную TestNet ноду и получить доступ к своим собственным частным конечным точкам. Для этого Вы можете использовать [это руководство](/node-operators/networks/full-node/). Обязательно проверьте [раздел расширенных флагов](/node-operators/networks/full-node/#advanced-flags-and-options)

В этом руководстве нода Graph работает с полноценной нодой Moonbase Alpha с флагом `--ethapi=trace`.

## Запуск ноды Graph {: #running-a-graph-node } 

В первом шаге Вам необходимо будет склонировать [репозиторий Graph](https://github.com/graphprotocol/graph-node/):

```
git clone https://github.com/graphprotocol/graph-node/ \
&& cd graph-node/docker
```

Затем, запустить файл `setup.sh`. Это команда извлечет все необходимые образы Docker и запишет необходимую информацию в файл `docker-compose.yml`.

```
./setup.sh
```

В конце выполнения предыдущей команды Вы должны увидеть примерно следующее:

![Graph Node setup](/images/node-operators/indexer-nodes/the-graph/the-graph-node-1.png)

После того, как все настроено, Вам нужно изменить "среду Ethereum" внутри файла `docker-compose.yml`, чтобы он ссылался на ноду, на котором Вы запускаете непосредственно узел Graph. Обратите внимание, что файл `setup.sh` определяет IP-адрес Вашего сервера и записывает его значение, поэтому Вам нужно будет соответствующим образом изменить его.

```
ethereum: 'mbase:http://localhost:9933'
```

Весь файл `docker-compose.yml` должен выглядеть примерно так:

```
version: '3'
services:
  graph-node:
    image: graphprotocol/graph-node
    ports:
      - '8000:8000'
      - '8001:8001'
      - '8020:8020'
      - '8030:8030'
      - '8040:8040'
    depends_on:
      - ipfs
      - postgres
    environment:
      postgres_host: postgres
      postgres_user: graph-node
      postgres_pass: let-me-in
      postgres_db: graph-node
      ipfs: 'ipfs:5001'
      ethereum: 'mbase:http://localhost:9933'
      RUST_LOG: info
  ipfs:
    image: ipfs/go-ipfs:v0.4.23
    ports:
      - '5001:5001'
    volumes:
      - ./data/ipfs:/data/ipfs
  postgres:
    image: postgres
    ports:
      - '5432:5432'
    command: ["postgres", "-cshared_preload_libraries=pg_stat_statements"]
    environment:
      POSTGRES_USER: graph-node
      POSTGRES_PASSWORD: let-me-in
      POSTGRES_DB: graph-node
    volumes:
      - ./data/postgres:/var/lib/postgresql/data
```

Наконец, чтобы запустить Graph ноду, просто выполните следующую команду:

```
docker-compose up
```

![Graph Node compose up](/images/node-operators/indexer-nodes/the-graph/the-graph-node-2.png)

Через некоторое время Вы должны увидеть информацию, связанную с синхронизацией Graph ноды с последним доступным блоком в сети:

![Graph Node logs](/images/node-operators/indexer-nodes/the-graph/the-graph-node-3.png)

На этом всё! Теперь у Вас есть Graph нода, работающая с Moonbase Alpha TestNet. Не стесняйтесь адаптировать этот пример и к узлу разработки Moonbeam.
