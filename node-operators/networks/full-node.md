---
title: Запуск Ноды
description: Как запустить полную парачейн ноду для сети Moonbeam, для получения конечной точки RPC или для производства блоков
---

# Запуск Ноды на Moonbeam

![Full Node Moonbeam Banner](/images/fullnode/fullnode-banner.png)

## Введение

С запуском Moonbase Alpha v6 Вы можете разворачивать ноду, которая подключается к тестовой сети Moonbase Alpha TestNet, синхронизируется с загрузочным узлом, обеспечивает локальный доступ к вашим конечным точкам RPC (remote procedure call) и даже создает блоки в парачейн (parachain).

В нашей тестовой сети хостинг и управление ретранслирующей цепью обеспечивает компания PureStake.  По мере развития, также будет организовано размещение на блокчейнах Kusama и Polkadot. Ниже представлены названия сетей и соответствующие им имена [файлов спецификации цепи](https://substrate.dev/docs/en/knowledgebase/integrate/chain-spec):

|    Сеть     |     | Расположение |     |   Название цепи   |
| :------------: | :-: | :-------: | :-: | :-------------: |
| Moonbase Alpha |     | PureStake |     |    alphanet     |
|   Moonriver    |     |  Kusama   |     | _нет данных_ |
|    Moonbeam    |     | Polkadot  |     | _нет данных_ |

Это руководство предназначено для людей, имеющих опыт работы с сетями на основе Substrate. Запуск парачейн аналогичен запуску ноды Substrate с небольшими отличиями. Нода парачейна Substrate будет запускать два процесса: один для синхронизации ретранслирующей цепи, а другой - для синхронизации парачейн. Таким образом, многие вещи дублируются, например, каталог базы данных, используемые порты, строки записи и многое другое.

!!! Примечание
    Moonbase Alpha по-прежнему считается Alphanet и поэтому не обеспечивает 100% безотказной работы. Парачейн время от времени будет очищаться. Во время разработки Вашего приложения убедитесь, что Вы реализуете метод быстрого повторного развертывания Ваших контрактов и учетных записей в новом парачейне. Уведомление об очистке цепи будет опубликовано в нашем [Discord канале](https://discord.gg/PfpUATX) как минимум за 24 часа.

## Требования

Ниже, в таблице,  представлены минимальные требования, рекомендуемые для запуска ноды. Для размещения нод основной сети (MainNet) на блокчейнах Kusama и Polkadot, требования к дискам будут больше, по мере роста сети.

|  Компоненты   |     | Требования                                                                                                                |
| :----------: | :-: | :------------------------------------------------------------------------------------------------------------------------- |
|   **CPU**    |     | 8 Cores (на ранних стадиях размещения - еще не оптимизированы)                                                                      |
|   **RAM**    |     | 16 GB (на ранних стадиях размещения - еще не оптимизированы)                                                                        |
|   **SSD**    |     | 50 GB (Для запуска нашей TestNet)                                                                                            |
| **Firewall** |     | P2P порт должен быть открыт для входящего трафика:<br>&nbsp; &nbsp; - Источник: Любой<br>&nbsp; &nbsp; - Получатель: 30333, 30334 TCP |

!!! Примечание
    Если Вы не видите сообщение Imported (без тега [Relaychain]), когда запускаете ноду, Вам может понадобиться перепроверить настройки портов.

## Порты запуска

Как указывалось ранее, ретранслирующая сеть  и парачейн ноды будут обращаться ко многим портам. По умолчанию, порты Substrate используются в парачейн, в то время как ретранслирующая сеть обращается к следующему в порядке возрастания  порту.

Для входящего трафика должны быть открыты только те порты, которые назначены для Р2Р.

### Порты по умолчанию для полной парачейн ноды

|  Описание   |     |                Порт                 |
| :------------: | :-: | :---------------------------------: |
|    **P2P**     |     | {{ networks.parachain.p2p }} (TCP)  |
|    **RPC**     |     |    {{ networks.parachain.rpc }}     |
|     **WS**     |     |     {{ networks.parachain.ws }}     |
| **Prometheus** |     | {{ networks.parachain.prometheus }} |

### Порты по умолчанию встроенной основной ретранслирующей сети

|  Описание   |     |                 Порт                  |
| :------------: | :-: | :-----------------------------------: |
|    **P2P**     |     | {{ networks.relay_chain.p2p }} (TCP)  |
|    **RPC**     |     |    {{ networks.relay_chain.rpc }}     |
|     **WS**     |     |     {{ networks.relay_chain.ws }}     |
| **Prometheus** |     | {{ networks.relay_chain.prometheus }} |

## Инструкция по установке - Docker

Нода Moonbase Alpha может быть быстро запущена с использованием Docker. Для большей информации по установке Docker, пожалуйста перейдите на [эту страницу](https://docs.docker.com/get-docker/). На момент написания, актуальная версия Docker - 19.03.6.

Сначала создайте локальную папку для хранения данных цепи:

```
mkdir {{ networks.moonbase.node_directory }}
```

Затем установите необходимые разрешения для каждого пользователя, определенного или текущего (замените `DOCKER_USER` на имя того пользователя, который будет запускать команду `docker`):

```
# chown to a specific user
chown DOCKER_USER {{ networks.moonbase.node_directory }}

# chown to current user
sudo chown -R $(id -u):$(id -g) {{ networks.moonbase.node_directory }}
```

!!! Примечание
    Убедитесь, что Вы установили соответствующие права  и необходимые разрешения для локальной папки, в которой хранятся данные цепи.

Теперь выполните Docker командой запуска. Имейте в виду, что Вы должны:

 - Заменить `YOUR-NODE-NAME` в двух различных местах.
 - Для коллаторов, замените `PUBLIC_KEY` открытым адресом, который привязан к деятельности коллатора.

!!! Примечание
    Если Вы устанавливаете ноду коллатора, убедитесь что вы используете фрагменты кода для “Коллатора”.

### Полная Нода

=== "Ubuntu"
    ```
    docker run --network="host" -v "{{ networks.moonbase.node_directory }}:/data" \
    -u $(id -u ${USER}):$(id -g ${USER}) \
    purestake/moonbeam:{{ networks.moonbase.parachain_docker_tag }} \
    --rpc-cors all \
    --base-path=/data \
    --chain alphanet \
    --name="YOUR-NODE-NAME" \
    --execution wasm \
    --wasm-execution compiled \
    --in-peers 200 \
    --out-peers 200 \
    --pruning archive \
    --state-cache-size 1 \
    -- \
    --pruning archive \
    --name="YOUR-NODE-NAME (Embedded Relay)"
    ```

=== "MacOS"
    ```
    docker run -p {{ networks.parachain.rpc }}:{{ networks.parachain.rpc }} -p {{ networks.parachain.ws }}:{{ networks.parachain.ws }} -v "{{ networks.moonbase.node_directory }}:/data" \
    -u $(id -u ${USER}):$(id -g ${USER}) \
    purestake/moonbeam:{{ networks.moonbase.parachain_docker_tag }} \
    --ws-external \
    --rpc-external \
    --rpc-cors all \
    --base-path=/data \
    --chain alphanet \
    --name="YOUR-NODE-NAME" \
    --execution wasm \
    --wasm-execution compiled \
    --in-peers 200 \
    --out-peers 200 \
    --pruning archive \
    --state-cache-size 1 \
    -- \
    --pruning archive \
    --name="YOUR-NODE-NAME (Embedded Relay)"
    ```
### Коллатор

=== "Ubuntu"
    ```
    docker run --network="host" -v "{{ networks.moonbase.node_directory }}:/data" \
    -u $(id -u ${USER}):$(id -g ${USER}) \
    purestake/moonbeam:{{ networks.moonbase.parachain_docker_tag }} \
    --base-path=/data \
    --chain alphanet \
    --name="YOUR-NODE-NAME" \
    --collator \
    --author-id PUBLIC_KEY \
    --execution wasm \
    --wasm-execution compiled \
    --in-peers 200 \
    --out-peers 200 \
    --pruning archive \
    --state-cache-size 1 \
    -- \
    --pruning archive \
    --name="YOUR-NODE-NAME (Embedded Relay)"
    ```

=== "MacOS"
    ```
    docker run -p {{ networks.parachain.rpc }}:{{ networks.parachain.rpc }} -p {{ networks.parachain.ws }}:{{ networks.parachain.ws }} -v "{{ networks.moonbase.node_directory }}:/data" \
    -u $(id -u ${USER}):$(id -g ${USER}) \
    purestake/moonbeam:{{ networks.moonbase.parachain_docker_tag }} \
    --ws-external \
    --rpc-external \
    --base-path=/data \
    --chain alphanet \
    --name="YOUR-NODE-NAME" \
    --collator \
    --author-id PUBLIC_KEY \
    --execution wasm \
    --wasm-execution compiled \
    --in-peers 200 \
    --out-peers 200 \
    --pruning archive \
    --state-cache-size 1 \
    -- \
    --pruning archive \
    --name="YOUR-NODE-NAME (Embedded Relay)"
    ```

Как только Docker получит необходимые данные, Ваша полная нода Moonbase Alpha запустится, выдавая большое количество информации, такой как, спецификация цепи, имя ноды, ее роль, прогресс работы и так далее:

![Запуск Полной Ноды](/images/fullnode/fullnode-docker1.png)

!!! Примечание
    Если Вы столкнулись со сложностями с телеметрией по умолчанию, Вы можете добавить метку `--no-telemetry` для запуска основной ноды без активирования телеметрии.

!!! Примечание
    Вы можете задать порт Prometheus по собственному выбору, добавив метку `--prometheus-port XXXX`  (заменив `XXXX` актуальным номером порта). Это возможно и для парачейн и для встроенной ретранслирующей сети.

Команда, указанная выше, сделает доступными все открытые порты, включая P2P, RPC и Prometheus ( для телеметрии) порты. Эта команда совместима с телеметрией Gantree Node Watchdog. Если Вы хотите открыть определенные порты, сделайте их доступными в Docker, запустив командную строку как указано ниже. Тем не менее, это действие блокирует контейнеру Gantree Node Watchdog (телеметрия) доступ к контейнеру Moonbeam. Поэтому не делайте этого, когда запускаете ноду коллатора, если Вы не понимаете принципы сетевой работы Docker [docker networking](https://docs.docker.com/network/).

```
docker run -p {{ networks.relay_chain.p2p }}:{{ networks.relay_chain.p2p }} -p {{ networks.parachain.p2p }}:{{ networks.parachain.p2p }} -p {{ networks.parachain.rpc }}:{{ networks.parachain.rpc }} -p {{ networks.parachain.ws }}:{{ networks.parachain.ws }} #rest of code goes here
```

Во время процесса синхронизации Вы увидите сообщения от встроенной ретранслирующей сетью и парачейна (без тегов). Эти сообщения отображают целевой блок (TestNet) и лучший блок (состояние синхронизации локальной ноды).

![Запуск Полной Ноды](/images/fullnode/fullnode-docker2.png)

После синхронизации, у Вас будет нода тестовой сети Moonbase Alpha  работающая локально!

## Инструкция по установке - Бинарный файл

Этот раздел показывает процесс компиляции бинарного файла и запуск основной ноды Moonbeam в качестве сервиса systemd. Представленные шаги были протестированы на версии Ubuntu 18.04. Moonbase Alpha может работать и на других версиях Linux, но на данный момент, Ubuntu единственная протестированная версия.

### Компилирование бинарного файла

Последующие команды установят самую последнюю версию парачейн Moonbeam.

Начнем с клонирования репозитория Moonbeam.

```
git clone https://github.com/PureStake/moonbeam
cd moonbeam
```

Давайте проверим последние обновления:

```
git checkout tags/$(git tag | tail -1)
```

Далее, инсталлируем Substrate и все его реквизиты, включая Rust, для этого выполним команду:

```
--8<-- 'code/setting-up-node/substrate.md'
```

И в завершение компилируем бинарный файл:

```
cargo build --release
```

![Компилирование бинарного файла](/images/fullnode/fullnode-binary1.png)

Если в терминале появилось сообщение _cargo not found error_ , необходимо вручную добавить Rust в вашу систему или перезагрузить ее:

```
--8<-- 'code/setting-up-node/cargoerror.md'
```

### Запуск сервиса Systemd

Нижеприведенные команды выполнят необходимую установку настроек для запуска сервиса.

Для начала создаем аккаунт для запуска сервиса:

```
adduser moonbase_service --system --no-create-home
```

Далее создаем папку для хранения бинарного файла и установки необходимых разрешений:

```
mkdir {{ networks.moonbase.node_directory }}
chown moonbase_service {{ networks.moonbase.node_directory }}
```

!!! Примечание
    Убедитесь, что вы установили соответствующие права владения и необходимые разрешения для локальной папки, в которой хранятся данные цепи.

Теперь копируем созданный бинарный файл в эту папку:

```
cp ./target/release/{{ networks.moonbase.binary_name }} {{ networks.moonbase.node_directory }}
```

Следующий шаг это создание файла конфигурации для systemd. Имейте в виду, что вам необходимо:

 - Заменить `YOUR-NODE-NAME` в двух различных местах
 - Перепроверить путь к бинарному файлу, он должен быть таким как указано ниже (_ExecStart_)
 - Перепроверить основной путь, если вы используете другую папку
 - Для коллаторов, замените `PUBLIC-KEY` открытым ключом вашего адреса Ethereum  H160, который был создан в соответствии с инструкциями выше.
 - Присвойте имя файлу `/etc/systemd/system/moonbeam.service`

!!! Примечание
    Если вы устанавливаете ноду коллатора, убедитесь что вы используете фрагменты кода для “Коллатора”.

=== "Полная Нода"
    ```
    [Unit]
    Description="Moonbase Alpha systemd service"
    After=network.target
    StartLimitIntervalSec=0

    [Service]
    Type=simple
    Restart=on-failure
    RestartSec=10
    User=moonbase_service
    SyslogIdentifier=moonbase
    SyslogFacility=local7
    KillSignal=SIGHUP
    ExecStart={{ networks.moonbase.node_directory }}/{{ networks.moonbase.binary_name }} \
         --parachain-id 1000 \
         --port {{ networks.parachain.p2p }} \
         --rpc-port {{ networks.parachain.rpc }} \
         --ws-port {{ networks.parachain.ws }} \
         --pruning=archive \
         --state-cache-size 1 \
         --unsafe-rpc-external \
         --unsafe-ws-external \
         --rpc-methods=Safe \
         --rpc-cors all \
         --log rpc=info \
         --base-path {{ networks.moonbase.node_directory }} \
         --chain alphanet \
         --name "YOUR-NODE-NAME" \
        --in-peers 200 \
        --out-peers 200 \
         -- \
         --port {{ networks.relay_chain.p2p }} \
         --rpc-port {{ networks.relay_chain.rpc }} \
         --ws-port {{ networks.relay_chain.ws }} \
         --pruning=archive \
         --name="YOUR-NODE-NAME (Embedded Relay)"

    [Install]
    WantedBy=multi-user.target
    ```

=== "Нода Коллатора"
    ```
    [Unit]
    Description="Moonbase Alpha systemd service"
    After=network.target
    StartLimitIntervalSec=0

    [Service]
    Type=simple
    Restart=on-failure
    RestartSec=10
    User=moonbase_service
    SyslogIdentifier=moonbase
    SyslogFacility=local7
    KillSignal=SIGHUP
    ExecStart={{ networks.moonbase.node_directory }}/{{ networks.moonbase.binary_name }} \
         --parachain-id 1000 \
         --collator \
         --author-id PUBLIC_KEY \
         --port {{ networks.parachain.p2p }} \
         --rpc-port {{ networks.parachain.rpc }} \
         --ws-port {{ networks.parachain.ws }} \
         --pruning=archive \
         --state-cache-size 1 \
         --unsafe-rpc-external \
         --unsafe-ws-external \
         --rpc-methods=Safe \
         --log rpc=info \
         --base-path {{ networks.moonbase.node_directory }} \
         --chain alphanet \
         --name "YOUR-NODE-NAME" \
         --in-peers 200 \
         --out-peers 200 \
         -- \
         --port {{ networks.relay_chain.p2p }} \
         --rpc-port {{ networks.relay_chain.rpc }} \
         --ws-port {{ networks.relay_chain.ws }} \
         --pruning=archive \
         --name="YOUR-NODE-NAME (Embedded Relay)"

    [Install]
    WantedBy=multi-user.target
    ```

!!! Примечание
    Если Вы испытываете сложности с телеметрией по умолчанию, Вы можете добавить метку `--no-telemetry` для запуска основной ноды без активирования телеметрии.

!!! Примечание
    Вы можете задать порт Prometheus по собственному выбору, добавив метку `--promethues-port XXXX`  (заменив `XXXX` ктуальным номером порта). Это возможно и для парачейн и для встроенной ретранслирующей ноды.

Почти готово! Регистрируем и запускаем сервис при помощи:

```
systemctl enable moonbeam.service
systemctl start moonbeam.service
```

И последнее, убедитесь что сервис запущен:

```
systemctl status moonbeam.service
```

![Статус сервиса](/images/fullnode/fullnode-binary2.png)

Вы также можете проверить логи, выполнив команду:

```
journalctl -f -u moonbeam.service
```

![Логи сервиса](/images/fullnode/fullnode-binary3.png)

## Дополнительные флаги и опции

--8<-- 'text/setting-up-node/advanced-flags.md'

## Обновление Клиента

Поскольку Moonbeam находится в разработке, время от времени необходимо обновлять Вашу ноду. По мере того, как обновления станут доступны и в зависимости от того насколько они необходимы (некоторые обновления клиентов не обязательны), операторы нод будут уведомлены в нашем [Discord канале](https://discord.gg/PfpUATX) . Процесс обновления прост и одинаков для основных нод и нод коллаторов.

Для начала остановите контейнер Docker или сервис systemd:

```
sudo docker stop `CONTAINER_ID`
# or
sudo systemctl stop moonbeam
```

Затем инсталлируйте новую версию путем повторения шагов указанных выше, убедившись что Вы используете последние доступные теги. После обновления, Вы можете запустить сервис снова.

### Очистка цепи

Изредка Moonbase Alpha может быть очищена и обнулена в связи со значительными обновлениями. Если это обновление сопровождается очисткой, операторы нод будут уведомлены заблаговременно (через наш [Discord канал](https://discord.gg/PfpUATX)) Вы также можете очистить Вашу ноду, если данные в вашей папке повреждены.

Чтобы это сделать, для начала остановите контейнер Docker или сервис systemd:

```
sudo docker stop `CONTAINER_ID`
# or
sudo systemctl stop moonbeam
```

Затем удалите содержимое папки, в которой хранятся данные цепи (как для парачейна, так и для ретранслирующей цепи):

```
sudo rm -rf {{ networks.moonbase.node_directory }}/*
```

Далее инсталлируйте последнюю версию путем повторения шагов,описанных выше, убедитесь что вы используете последние доступные теги. Если это все выполнено, вы можете запустить новую ноду с чистой папкой данных.

## Телеметрия

Для включения сервера телеметрии Вашей ноды Moonbase Alpha, вы можете воспользоваться [этим руководством](/node-operators/networks/telemetry/).

Запуск телеметрии для полных нод не является обязательным. Однако, коллаторы должны выполнить это требование.

Также Вы можете увидеть текущую информацию по телеметрии Moonbase Alpha воспользовавшись [этой ссылкой](https://telemetry.polkadot.io/#list/Moonbase%20Alpha).

## Логи и исправление проблем

Вы увидите логи как от ретранслирующей сети, так и от парачейна. Ретранслирующая сеть  имеет приставку `[Relaychain]`, в то время как парачейн не имеет приставки.

### P2P порты не открыты

Если Вы не видите сообщения `Imported` (без тега `[Relaychain]`), Вам необходимо проверить конфигурацию порта P2P. P2P порт должен быть открыт для входящего трафика.

### Синхронизация

Обе цепи должны быть синхронизированы все время и Вы должны увидеть сообщение `Imported` или `Idle`  и иметь подключенные пиры.

### Несоответствие генезиса

Тестовая сеть Moonbase Alpha часто обновляется. Как следствие, вы можете видеть такое сообщение:

```
DATE [Relaychain] Bootnode with peer id `ID` is on a different
chain (our genesis: GENESIS_ID theirs: OTHER_GENESIS_ID)
```

Обычно это означает, что у Вас запущена старая версия и она требует обновления.

Мы уведомляем об обновлениях (и соответствующих очистках цепи) через наш [Discord канал](https://discord.gg/PfpUATX) как минимум за 24 часа.

