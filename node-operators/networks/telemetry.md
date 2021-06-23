---
title: Телеметрия
description: Как запустить телеметрию для полной Парачейн ноды для Сети Moonbeam
---

# Телеметрия для полной ноды

![Telemetry Moonbeam Banner](/images/fullnode/telemetry-banner.png)

## Вступление

С выходом Moonbase Alpha v6 вы можете запустить ноду, которая подключается к Moonbase Alpha TestNet. Пошаговое руководство можно найти в  [этой инструкции](/node-operators/networks/full-node/).

Это руководство предоставит необходимые шаги для запуска сервера телеметрии для вашей Moonbase Alpha ноды.

!!! примечание 
    Шаги, описанные в этом руководстве, предназначены для варианта телеметрии, отличного от стандартной телеметрии Polkadot, активированной по умолчанию (вы можете запускать ноды) без телеметрии, используя флаг `--no-telemetry`). Шаги, описанные в этом руководстве, обязательны только для узлов коллатора.

## Сводка экспортера телеметрии

Moonbeam будет запускать сервер телеметрии, который собирает метрики Prometheus со всех нод парачейна Moonbeam в сети. Такой запуск будет помощью для нас на этапе разработки.  

Экспортер метрик может работать как сопроводительный файл, либо как локальный двоичный файл, если вы используете виртуальную машину. Он будет отправлять данные на наши серверы, поэтому вам не нужно включать какие-либо входящие порты для этого сервиса.

Мы используем сервис под названием [Gantree Node Watchdog](https://github.com/gantree-io/gantree-node-watchdog) to upload telemetry automatically.  для автоматической загрузки телеметрии. После включения телеметрии вы также можете получить доступ к серверу Prometheus / Grafana из приложения [Gantree App](https://app.gantree.io/).  В репозитории GitHub есть подробные инструкции. Если вам нужна дополнительная информация, тут краткая инструкция для быстрого старта.

На данный момент нам нужно запустить две “сторожевые” ноды: одна для Relay Chain, другая для парачейна. Все это будет обновлено в следующем выпуске.

 Если Вам понадобится помощь, звяжитесь с нами в [Discord](https://discord.gg/FQXm74UQ7V) или в [Gantree Discord](https://discord.gg/N95McPjHZ2). 
 
## Проверка предварительных условий

Перед тем как использовать это руководство, Вам необходимо:

 1. Войти в [https://app.gantree.io](https://app.gantree.io) и создать учетную запись.  Перейдите к ключам API и скопируйте свой ключ API.
 
 2. Запросить ключ PCK на нашем [Discord сервере](https://discord.gg/FQXm74UQ7V)
   
## Экспортер телеметрии с Docker

Мы запустим два экземпляра “сторожевых” нод Gantree с помощью Docker: одна для Relay Chain, другая для парачейна.  

### Необходимая информация о конфигурации

- GANTREE_NODE_WATCHDOG_API_KEY
- GANTREE_NODE_WATCHDOG_PROJECT_ID
- GANTREE_NODE_WATCHDOG_CLIENT_ID
- GANTREE_NODE_WATCHDOG_PCKRC
- GANTREE_NODE_WATCHDOG_METRICS_HOST

### Инструкции

Сначала склонируйте клиентский репозиторий мониторинга экземпляров и создайте образ докера:

```
git clone https://github.com/gantree-io/gantree-node-watchdog
cd gantree-node-watchdog
# checkout latest release
git checkout tags/$(git tag | tail -1)
docker build .  
# get the IMAGE-NAME for use below
docker images
```

Затем запустим докер-контейнер (“сторожевая” нода  парачейна Gantree). Обратите внимание, что вам необходимо заменить следующие поля:

  - `IMAGE-NAME` тем, что было получено на предыдущем шаге
  - `YOUR-API-KEY` тем, что предоставленно на  [https://app.gantree.io](https://app.gantree.io)
  - `YOUR-SERVER-NAME`
  - `YOUR-PCK-KEY` тем, что запрошено в нашем Discord

```
docker run -it --network="host" \
-e GANTREE_NODE_WATCHDOG_API_KEY="YOUR-API-KEY" \
-e GANTREE_NODE_WATCHDOG_PROJECT_ID="moonbase-alpha" \
-e GANTREE_NODE_WATCHDOG_CLIENT_ID="YOUR-SERVER-NAME-parachain" \
-e GANTREE_NODE_WATCHDOG_PCKRC="YOUR-PCK-KEY" \
-e GANTREE_NODE_WATCHDOG_METRICS_HOST="http://127.0.0.1:9615" \
--name gantree_watchdog_parachain IMAGE-NAME
```

Теперь нам нужно запустить “сторожевую” ноду парачейна Gantree. Обратите внимание, что вам нужно заменить ту же информацию, что и на предыдущем шаге.

```
docker run -it --network="host" \
-e GANTREE_NODE_WATCHDOG_API_KEY="YOUR-API-KEY" \
-e GANTREE_NODE_WATCHDOG_PROJECT_ID="moonbase-alpha" \
-e GANTREE_NODE_WATCHDOG_CLIENT_ID="YOUR-SERVER-NAME-relay" \
-e GANTREE_NODE_WATCHDOG_PCKRC="YOUR-PCK-KEY" \
-e GANTREE_NODE_WATCHDOG_METRICS_HOST="http://127.0.0.1:9616" \
--name gantree_watchdog_relay IMAGE-NAME
```

Вы должны увидеть ожидание подготовки в журналах. По завершении вы можете войти в [https://app.gantree.io](https://app.gantree.io) и выбрать сети. Вы увидите ссылку `View Monitoring Dashboard` на свою настраиваемую панель мониторинга Prometheus / Grafana, которую вы можете настроить в соответствии с вашими предпочтениями.  

Когда все заработает, вы можете обновить команды для работы в режиме daemon.  Удалите `-it` and add `-d` в приведенной выше команде.  

## Экспортер телеметрии с Systemd

Мы запустим два экземпляра сторожевых нод Gantree: одна для цепочки реле, другая для парачейна.  

### Необходимая информация о конфигурации

- GANTREE_NODE_WATCHDOG_API_KEY
- GANTREE_NODE_WATCHDOG_PROJECT_ID
- GANTREE_NODE_WATCHDOG_CLIENT_ID
- GANTREE_NODE_WATCHDOG_PCKRC
- GANTREE_NODE_WATCHDOG_METRICS_HOST

### Инструкции

Во-первых, нам нужно загрузить двоичный код сторожевой ноды Gantree со страницы выпуска  [release page](https://github.com/gantree-io/gantree-node-watchdog/releases), и извлечь ее в папку, например `/usr/local/bin`.

Далее создадим две папки для файлов конфигурации:

```
mkdir -p /var/lib/gantree/parachain
mkdir -p /var/lib/gantree/relay
```

Теперь нам нужно сгенерировать файлы конфигурации, поместить каждый файл в папки, созданные на предыдущем шаге. Обратите внимание, что вам необходимо заменить следующие поля:

  - `YOUR-API-KEY` тем, что предоставленно [https://app.gantree.io](https://app.gantree.io)
  - `YOUR-SERVER-NAME`
  - `YOUR-PCK-KEY` тем, что запрошено в нашем Discord

Парачейн:

```
# Contents of /var/lib/gantree/parachain/.gnw_config.json
{
  "api_key": "YOUR-API-KEY",
  "project_id": "moonbase-alpha",
  "client_id": "YOUR-SERVER-NAME-parachain",
  "pckrc": "YOUR-PCK-KEY",
  "metrics_host": "http://127.0.0.1:9615"
}
```

Встроенная Relay Chain:

```
# Contents of /var/lib/gantree/relay/.gnw_config.json
{
  "api_key": "YOUR-API-KEY",
  "project_id": "moonbase-alpha",
  "client_id": "YOUR-SERVER-NAME-relay",
  "pckrc": "YOUR-PCK-KEY",
  "metrics_host": "http://127.0.0.1:9616"
}
```

Следующим шагом будет создание файла конфигурации systemd:

Парачейн:

```
# Contents of /etc/systemd/system/gantree-parachain.service

[Unit]
Description=Gantree Node Watchdog Parachain
After=network.target

[Service]
WorkingDirectory=/var/lib/gantree/parachain
Type=simple
Restart=always
ExecStart=/usr/local/bin/gantree_node_watchdog

[Install]
WantedBy=multi-user.target
```

Встроенная Relay Chain:

```
# Contents of /etc/systemd/system/gantree-relay.service

[Unit]
Description=Gantree Node Watchdog Relay
After=network.target

[Service]
WorkingDirectory=/var/lib/gantree/relay
Type=simple
Restart=always
ExecStart=/usr/local/bin/gantree_node_watchdog

[Install]
WantedBy=multi-user.target
```

Мы почти на месте! Теперь давайте включим и запустим службы systemd, просмотрим журналы на наличие ошибок:

```
sudo systemctl enable gantree-parachain
sudo systemctl start gantree-parachain && journalctl -f -u gantree-parachain

sudo systemctl enable gantree-relay
sudo systemctl start gantree-relay && journalctl -f -u gantree-relay
```

Вы должны увидеть ожидание подготовки в журналах. По завершении вы можете войти в  [https://app.gantree.io](https://app.gantree.io) и выбрать сети. Вы увидите `View Monitoring Dashboard` на свою настраиваемую панель мониторинга Prometheus / Grafana, которую вы можете настроить в соответствии со своими предпочтениями.  

## Обратная связь

Если у вас есть какие-либо отзывы о запуске полной ноды с телеметрией или о любой другой теме, связанной с Moonbeam, свяжитесь с нами через наш официальный [Discord](https://discord.com/invite/PfpUATX).
