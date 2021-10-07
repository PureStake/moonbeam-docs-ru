---
title: Телеметрия
description: Как запустить телеметрию для полноценного узла Parachain в сети Moonbeam
---

# Телеметрия для полноценного узла

![Telemetry Moonbeam Banner](/images/node-operators/networks/telemetry-banner.png)

## Вступление {: #introduction } 

Начиная с Moonbase Alpha v6 и недавнего запуска Moonriver, Вы можете развернуть узел, который подключается к Moonbase Alpha TestNet или Moonriver на Кусаме. Вы можете проверить эти шаги в [данном руководстве](/node-operators/networks/full-node/).

Это руководство предоставит необходимые шаги для активации телеметрии-сервера для Вашего узла на базе Moonbeam.

!!! примечание
    Шаги, описанные в этом руководстве, относятся к экземпляру телеметрии, отличному от стандартной телеметрии Polkadot, включенной по умолчанию. (Вы можете запускать узлы без телеметрии, используя флаг `--no-telemetry`). Шаги, описанные в этом руководстве, являются обязательными только для узлов коллаторов.

## Общее резюме по Телеметрии {: #telemetry-exporter-summary } 

Moonbeam будет запускать сервер телеметрии, который собирает метрики Prometheus со всех парачейн узлов в Moonbeam сети. Запуск данной функции будет большой помощью для нас на этапе разработки.

"Экспортер" метрик может работать либо как сопроводительный элемент Kubernetes, либо как локальный двоичный файл, если Вы используете виртуальную машину. Он будет отправлять данные на наши сервера, поэтому Вам не нужно включать какие-либо входящие порты для этой службы.

Мы используем службу под названием [Gantree Node Watchdog](https://github.com/gantree-io/gantree-node-watchdog) для автоматической загрузки телеметрии. После включения телеметрии Вы также можете получить доступ к серверу Prometheus/Grafana из [Gantree App](https://app.gantree.io/). В репозитории GitHub на этот счёт есть подробные инструкции. Если Вам нужна дополнительная информация, вот с чего Вы можете начать.

На данный момент нам нужно запустить два watchdogs узла, один для парачейна и один для цепи передачи данных. Дополнительная информация будет добавлена в следующем выпуске.

Для получения помощи свяжитесь с нами через [Discord сервер](https://discord.gg/FQXm74UQ7V) или через [Gantree Discord](https://discord.gg/N95McPjHZ2). 
 
## Проверка требований {: #checking-prerequisites } 

Перед тем, как следовать этому руководству, Вам необходимо:

 1. Выполнить вход в [https://app.gantree.io](https://app.gantree.io) и создать учётную запись. Перейдите в меню "API ключи" и скопируйте Ваш API ключ. 
 2. Запросите ключ PCK на нашем [Discord сервере](https://discord.gg/FQXm74UQ7V)


Вы можете использовать один и тот же ключ PCK для всех наших сетей на основе Moonbeam, которые в настоящее время включают Moonbase Alpha и Moonriver.
   
## Телеметрия с Docker {: #telemetry-exporter-with-docker } 

Мы запустим два экземпляра watchdog узла Gantree с помощью Docker: один для парачейна и один для цепочки передачи данных.

### Необходимая информация о конфигурации {: #required-configuration-information } 

- GANTREE_NODE_WATCHDOG_API_KEY
- GANTREE_NODE_WATCHDOG_PROJECT_ID
- GANTREE_NODE_WATCHDOG_CLIENT_ID
- GANTREE_NODE_WATCHDOG_PCKRC
- GANTREE_NODE_WATCHDOG_METRICS_HOST

### Инструкции {: #instructions } 

Сначала клонируйте клиентский репозиторий содержащий экземпляр мониторинга и создайте образ докера:

```
git clone https://github.com/gantree-io/gantree-node-watchdog
cd gantree-node-watchdog
# checkout latest release
git checkout tags/$(git tag | tail -1)
docker build .  
# get the IMAGE-NAME for use below
docker images
```

Запустите докер-контейнер (узел парачейна watchdog Gantree). Обратите внимание, что Вам необходимо заменить следующие поля:

  - `IMAGE-NAME` с тем, что было получено на предыдущем шаге
  - `YOUR-API-KEY` с предоставленным [https://app.gantree.io](https://app.gantree.io)
  - `YOUR-SERVER-NAME`
  - `YOUR-PCK-KEY` используйте ключ, который был запрошен Вами на нашем сервере Discord (Вы можете использовать его для всех сетей на основе Moonbeam).

`PROJECT_ID` всегда будет иметь значение `moonbeam`, независимо от того, к какой сети Вы подключены. `CLIENT_ID` должен содержать название Вашей компании, чтобы мы могли легко идентифицировать Вас в панели инструментов Prometheus/Grafana.

```
docker run -it --network="host" \
-e GANTREE_NODE_WATCHDOG_API_KEY="ВАШ-API-КЛЮЧ" \
-e GANTREE_NODE_WATCHDOG_PROJECT_ID="moonbeam" \
-e GANTREE_NODE_WATCHDOG_CLIENT_ID="ИМЯ-ВАШЕГО-СЕРВЕРА-parachain" \
-e GANTREE_NODE_WATCHDOG_PCKRC="ВАШ-PCK-КЛЮЧ" \
-e GANTREE_NODE_WATCHDOG_METRICS_HOST="http://127.0.0.1:9615" \
--name gantree_watchdog_parachain ИМЯ-ОБРАЗА
```

Теперь нам нужно запустить ноду передачи данных - Gantree watchdog. Обратите внимание, что Вам нужно заменить ту же информацию, что и в предыдущем шаге.

```
docker run -it --network="host" \
-e GANTREE_NODE_WATCHDOG_API_KEY="ВАШ-API-КЛЮЧ" \
-e GANTREE_NODE_WATCHDOG_PROJECT_ID="moonbeam" \
-e GANTREE_NODE_WATCHDOG_CLIENT_ID="ИМЯ-ВАШЕГО-СЕРВЕРА-relay" \
-e GANTREE_NODE_WATCHDOG_PCKRC="ВАШ-PCK-КЛЮЧ" \
-e GANTREE_NODE_WATCHDOG_METRICS_HOST="http://127.0.0.1:9616" \
--name gantree_watchdog_relay ИМЯ-ОБРАЗА
```

В журнале событий Вы должны увидеть "ожидание подготовки". Если Вы впервые запускаете Gantree, он будет ждать, пока Вы снова зайдёте на портал и не нажмете "provision на панели управления", чтобы переключиться на "provisioning". Это переключение может занять несколько минут. Завершив предыдущие шаги Вы можете войти в [https://app.gantree.io] (https://app.gantree.io) и выбрать необходимую сеть. Вы увидите ссылку `Просмотр панели мониторинга` на свою индвидуальную панель мониторинга Prometheus/Grafana, которую Вы можете настроить в соответствии со своими потребностями.

Когда все начнёт работать правильно, Вы можете обновить команды для работы в режиме службы.  Удаление `-it` и добавление `-d` к команде выше.

## Телеметрия с Systemd {: #telemetry-exporter-with-systemd } 

Мы запустим два экземпляра watchdog таймера Gantree узла: один для парачейна и один для цепи передачи.

### Необходимая информация о конфигурации {: #required-configuration-information } 

- GANTREE_NODE_WATCHDOG_API_KEY
- GANTREE_NODE_WATCHDOG_PROJECT_ID
- GANTREE_NODE_WATCHDOG_CLIENT_ID
- GANTREE_NODE_WATCHDOG_PCKRC
- GANTREE_NODE_WATCHDOG_METRICS_HOST

### Инструкции {: #instructions } 


Во-первых, нам нужно загрузить двоичный код узла watchdog Gantree со [страницы выпуска](https://github.com/gantree-io/gantree-node-watchdog/releases), и распакуйте его в папку, например, `/usr/local/bin`.

Далее создадим две папки для файлов конфигурации:

```
mkdir -p /var/lib/gantree/parachain
mkdir -p /var/lib/gantree/relay
```

Теперь нам нужно сгенерировать файлы конфигурации, поместить каждый в папки, созданные на предыдущем шаге. Обратите внимание, что Вам необходимо заменить следующие поля:

  - `YOUR-API-KEY` с полученным на странице [https://app.gantree.io](https://app.gantree.io)
  - `YOUR-SERVER-NAME`
  - `YOUR-PCK-KEY` с ключом, который был запрошен ранее через наш Discord сервер

`Project_id` всегда будет иметь значение` moonbeam`, независимо от того, к какой сети Вы подключены. `client_id` должен содержать название Вашей компании, чтобы мы могли легко идентифицировать Вас в панели инструментов Prometheus/Grafana.

Парачейн:

```
# Contents of /var/lib/gantree/parachain/.gnw_config.json
{
  "api_key": "YOUR-API-KEY",
  "project_id": "moonbeam",
  "client_id": "YOUR-SERVER-NAME-parachain",
  "pckrc": "YOUR-PCK-KEY",
  "metrics_host": "http://127.0.0.1:9615"
}
```

Встроенная цепь передачи:

```
# Contents of /var/lib/gantree/relay/.gnw_config.json
{
  "api_key": "YOUR-API-KEY",
  "project_id": "moonbeam",
  "client_id": "YOUR-SERVER-NAME-relay",
  "pckrc": "YOUR-PCK-KEY",
  "metrics_host": "http://127.0.0.1:9616"
}
```

Следующим шагом будет создание файла конфигурации systemd.

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

Встроенная цепь передачи:

```
# Содержание /etc/systemd/system/gantree-relay.service

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

Мы почти у цели! Теперь давайте включим и запустим systemd службы, просмотрим журнал событий на наличие ошибок:

```
sudo systemctl enable gantree-parachain
sudo systemctl start gantree-parachain && journalctl -f -u gantree-parachain

sudo systemctl enable gantree-relay
sudo systemctl start gantree-relay && journalctl -f -u gantree-relay
```

Вы должны увидеть ожидание подготовки в журнале событий. По завершении Вы можете войти в [https://app.gantree.io](https://app.gantree.io) и выбрать необходимую сеть. Вы увидите ссылку `Просмотр панели мониторинга` на свою индивидуальную панель мониторинга Prometheus/Grafana, которую Вы можете настроить в соответствии со своими предпочтениями.
