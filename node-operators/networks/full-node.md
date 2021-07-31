---
title: Запуск Ноды
description: Как запустить полноценную Parachain ноду для сети Moonbeam, для получения собственного RPC узла или для создания блоков.
---

# Запуск Moonbeam Узла

![Full Node Moonbeam Banner](/images/fullnode/fullnode-banner.png)

## Вступление {: #introduction } 

Запуск полноценного узла в сети Moonbeam, даёт возможность подключения к сети, выполнять синхронизацию с загрузочным узлом, получить локальный доступ к RPC узлам, создавать авторские блоки на парачейне и многое другое.

Есть несколько вариантов реализации Moonbeam, в том числе Moonbase Alpha TestNet, Moonriver на Кусаме, и в конечном итоге Moonbeam на Polkadot (поддержка последнего будет реализована позже). Вот названия этих окружений и соответствующие им имена [файл спецификации цепи] (https://substrate.dev/docs/en/knowledgebase/integrate/chain-spec):

|    Сеть        |     | Хостинг   |     |   Имя цепи      |
| :------------: | :-: | :-------: | :-: | :-------------: |
| Moonbase Alpha |     | PureStake |     |    {{ networks.moonbase.chain_spec }}     |
|   Moonriver    |     |  Kusama   |     |    {{ networks.moonriver.chain_spec }}    |
|    Moonbeam    |     | Polkadot  |     | _not available_ |

Это руководство предназначено для людей с опытом компиляции блокчейн узлов на основе [Substrate](https://substrate.dev/). Узел парачейна похож на типичный узел Substrate, но имеет некоторые отличия. Узел парачейна на Substrate ялвяется более сложным в сборке, потому что он содержит код для запуска самого парачейна, а также код синхронизации цепи для передачи данных, что в свою очередь упрощает связь между ними. Таким образом, данная сборка довольно объёмная, поэтому может занять более 30 минут и потребовать 32ГБ оперативной памяти.

!!! примечание
    Moonbase Alpha по-прежнему считается Alphanet, и поэтому _не будет иметь_ 100% времени безотказной работы. Парачейн _будет_ время от времени очищаться. Во время разработки Вашего приложения, убедитесь, что Вы реализовали метод быстрого переноса Ваших контрактов и учетных записей в новый парачейн. Об очистке цепи будет объявлено на нашем [канале Discord](https://discord.gg/PfpUATX) как минимум за 24 часа.

## Требования {: #requirements } 

Минимальные характеристики, рекомендуемые для запуска узла, показаны в следующей таблице. Для создная MainNet на Kusama и Polkadot требования к дискам будут выше по мере роста сети.

=== "Moonbase Alpha"
    |  Компонент   |     | Требования                                                                                                                 |
    | :----------: | :-: | :------------------------------------------------------------------------------------------------------------------------- |
    |   **CPU**    |     | 8 ядер (Самая высокая скорость на ядро)                                                                      |
    |   **RAM**    |     | 16 GB                                                                         |
    |   **SSD**    |     | 50 GB (для начала)                                                                                            |
    | **Firewall** |     | P2P-порт должен быть открыт для входящего трафика:<br>&nbsp; &nbsp; - Source: Any<br>&nbsp; &nbsp; - Destination: 30333, 30334 TCP |

=== "Moonriver"
    |  Компонент   |     | Требования                                                                                                                 |
    | :----------: | :-: | :------------------------------------------------------------------------------------------------------------------------- |
    |   **CPU**    |     | 8 ядер (Самая высокая скорость на ядро)                                                                          |
    |   **RAM**    |     | 16 GB                                                                         |
    |   **SSD**    |     | 300 GB (для начала)                                                                              |
    | **Firewall** |     | P2P-порт должен быть открыт для входящего трафика:<br>&nbsp; &nbsp; - Source: Any<br>&nbsp; &nbsp; - Destination: 30333, 30334 TCP |


!!! примечание
    Если Вы не видите сообщение `Импортировать` (без тега `[Relaychain]`) при запуске узла, Вам может потребоваться перепроверить конфигурацию Вашего порта.

## Активные порты {: #running-ports } 

Как мы до этого обсуждали, узлы передачи/парачейна будут прослушивать несколько портов. Порты Substrate по умолчанию используются в парачейне, в то время как цепь передачи данных будет прослушивать следующий более высокий порт.

Единственные порты, которые должны быть открыты для входящего трафика, - это те, которые предназначены для P2P.

### Порты по умолчанию для полноценной Parachain ноды {: #default-ports-for-a-parachain-fullnode } 

|    Описание    |     |                Порт                 |
| :------------: | :-: | :---------------------------------: |
|    **P2P**     |     | {{ networks.parachain.p2p }} (TCP)  |
|    **RPC**     |     |    {{ networks.parachain.rpc }}     |
|     **WS**     |     |     {{ networks.parachain.ws }}     |
| **Prometheus** |     | {{ networks.parachain.prometheus }} |

### Порты по умолчанию встроенной цепи передачи {: #default-ports-of-embedded-relay-chain } 

|    Описание    |     |                 Порт                  |
| :------------: | :-: | :-----------------------------------: |
|    **P2P**     |     | {{ networks.relay_chain.p2p }} (TCP)  |
|    **RPC**     |     |    {{ networks.relay_chain.rpc }}     |
|     **WS**     |     |     {{ networks.relay_chain.ws }}     |
| **Prometheus** |     | {{ networks.relay_chain.prometheus }} |

## Инструкции по установке - Docker {: #installation-instructions-docker } 

Узел Moonbeam можно быстро развернуть с помощью Docker. Для получения дополнительной информации об установке Docker посетите [эту страницу](https://docs.docker.com/get-docker/). На момент написания статьи использовалась версия Docker 19.03.6. При подключении к Moonriver на Кусаме, полная синхронизация встроенной цепи передачи Kusama займет несколько дней. Убедитесь, что Ваша система соответствует [требованиям](#требования).

Создайте локальный каталог для хранения данных цепи:

=== "Moonbase Alpha"
    ```
    mkdir {{ networks.moonbase.node_directory }}
    ```

=== "Moonriver"
    ```
    mkdir {{ networks.moonriver.node_directory }}
    ```

Затем убедитесь, что Вы правильно установили владельца и разрешения для локального каталога, в котором хранятся данные цепи. В этом случае установите необходимые разрешения либо для конкретного, либо для текущего пользователя (замените `DOCKER_USER` на фактического пользователя, который будет запускать команду `docker`):

=== "Moonbase Alpha"
    ```
    # chown для конкретного пользователя
    chown DOCKER_USER {{ networks.moonbase.node_directory }}

    # chown для текущего пользователя
    sudo chown -R $(id -u):$(id -g) {{ networks.moonbase.node_directory }}
    ```

=== "Moonriver"
    ```
    # chown для конкретного пользователя
    chown DOCKER_USER {{ networks.moonriver.node_directory }}

    # chown для текущего пользователя
    sudo chown -R $(id -u):$(id -g) {{ networks.moonriver.node_directory }}
    ```

Теперь выполните команду docker run. Если Вы настраиваете узел коллатора, обязательно следуйте фрагментам кода для "Коллатора". Обратите внимание, что Вам нужно заменить `YOUR-NODE-NAME` в двух разных местах.

### Полноценная нода {: #full-node } 

=== "Moonbase Alpha"
    ```
    docker run --network="host" -v "{{ networks.moonbase.node_directory }}:/data" \
    -u $(id -u ${USER}):$(id -g ${USER}) \
    purestake/moonbeam:{{ networks.moonbase.parachain_release_tag }} \
    --base-path=/data \
    --chain {{ networks.moonbase.chain_spec }} \
    --name="YOUR-NODE-NAME" \
    --execution wasm \
    --wasm-execution compiled \
    --pruning archive \
    --state-cache-size 1 \
    -- \
    --pruning archive \
    --name="YOUR-NODE-NAME (Embedded Relay)"
    ```

=== "Moonriver"
    ```
    docker run --network="host" -v "{{ networks.moonriver.node_directory }}:/data" \
    -u $(id -u ${USER}):$(id -g ${USER}) \
    purestake/moonbeam:{{ networks.moonriver.parachain_release_tag }} \
    --base-path=/data \
    --chain {{ networks.moonriver.chain_spec }} \
    --name="YOUR-NODE-NAME" \
    --execution wasm \
    --wasm-execution compiled \
    --pruning archive \
    --state-cache-size 1 \
    -- \
    --pruning archive \
    --name="YOUR-NODE-NAME (Embedded Relay)"
    ```

### Коллатор {: #collator } 

=== "Moonbase Alpha"
    ```
    docker run --network="host" -v "{{ networks.moonbase.node_directory }}:/data" \
    -u $(id -u ${USER}):$(id -g ${USER}) \
    purestake/moonbeam:{{ networks.moonbase.parachain_release_tag }} \
    --base-path=/data \
    --chain {{ networks.moonbase.chain_spec }} \
    --name="YOUR-NODE-NAME" \
    --validator \
    --execution wasm \
    --wasm-execution compiled \
    --pruning archive \
    --state-cache-size 1 \
    -- \
    --pruning archive \
    --name="YOUR-NODE-NAME (Embedded Relay)"
    ```

=== "Moonriver"
    ```
    docker run --network="host" -v "{{ networks.moonriver.node_directory }}:/data" \
    -u $(id -u ${USER}):$(id -g ${USER}) \
    purestake/moonbeam:{{ networks.moonriver.parachain_release_tag }} \
    --base-path=/data \
    --chain {{ networks.moonriver.chain_spec }} \
    --name="YOUR-NODE-NAME" \
    --validator \
    --execution wasm \
    --wasm-execution compiled \
    --pruning archive \
    --state-cache-size 1 \
    -- \
    --pruning archive \
    --name="YOUR-NODE-NAME (Embedded Relay)"
    ```

Если Вы используете MacOS, Вы можете найти все фрагменты кода [здесь](/snippets/text/full-node/macos-node/).

Как только Docker извлечет необходимые образы, Ваш полноценный узел Moonbeam (или Moonriver) запустится, будет показано большое количество информации, такой как спецификация цепи, имя узла, роль, статус генезиса и многое другое:

![Запуск Полноценного Узла](/images/fullnode/fullnode-docker1.png)

!!! примечание
    Если Вы хотите запустить конечную RPC точку, для подключения polkadot.js.org или для запуска собственного приложения, используйте флаги `--unsafe-rpc-external` и/или `--unsafe-ws-external` для запуска полноценного узла с внешним доступом к RPC портам.  Более подробную информацию можно получить, выполнив команду `moonbeam --help`.  

!!! примечание
    Вы можете указать собственный порт Prometheus с помощью флага `--prometheus-port XXXX` (заменив `XXXX` на конкретный номер порта). Это возможно как для парачейна, так и для встроенной цепи передачи данных.

Приведенная выше команда включит все открытые порты, необходимые для базовой работы, включая порты P2P и Prometheus (телеметрия). Эта команда совместима с телеметрией Gantree Node Watchdog. Если Вы хотите открыть определенные порты, включите их в командной строке Docker run, как показано ниже. Однако это заблокирует контейнер Gantree Node Watchdog (телеметрия) от доступа к контейнеру moonbeam, поэтому не делайте этого при работе с коллатором, если Вы не разбираетесь в [докер сети](https://docs.docker.com/network/).

```
docker run -p {{ networks.relay_chain.p2p }}:{{ networks.relay_chain.p2p }} -p {{ networks.parachain.p2p }}:{{ networks.parachain.p2p }} -p {{ networks.parachain.rpc }}:{{ networks.parachain.rpc }} -p {{ networks.parachain.ws }}:{{ networks.parachain.ws }} #rest of code goes here
```

Во время процесса синхронизации Вы увидите сообщения как от встроенной цепи передачи данных, так и от парачейна (без тега). Эти сообщения отображают блок назначения (текущее состояние сети) и лучший блок (синхронизированное состояние локального узла).

![Полноценный запуск узла](/images/fullnode/fullnode-docker2.png)

!!! примечание
    Полная синхронизация встроенной цепи передачи Kusama займет несколько дней. Убедитесь, что Ваша система соответствует [требованиям](#требования). 

Если Вы следовали инструкции по установке Moonbase Alpha, после синхронизации у Вас будет локально запущен узел тестовой сети Moonbase Alpha TestNet!

Если Вы следовали инструкции по установке Moonriver, после синхронизации Вы будете подключены к одноранговым узлам и увидите блоки, производимые в сети Moonriver! Обратите внимание, что в этом случае Вам также необходимо выполнить синхронизацию с цепью передачи данных Kusama, что может занять несколько дней.

## Инструкции по установке - из исходного кода {: #installation-instructions-binary } 

В этом разделе рассматривается процесс использования двоичного файла и запуска полноценного узла Moonbeam в качестве службы systemd. Следующие шаги были протестированы на системе Ubuntu 18.04. Moonbeam также может работать и с другими версиями Linux, но Ubuntu в настоящее время является единственной протестированной версией.

Чтобы вручную собрать двоичные файлы самостоятельно, ознакомьтесь с руководством [компиляция бинарного файл Moonbeam](/node-operators/networks/compile-binary).

### Использование двоичного файла {: #use-the-release-binary } 

Есть несколько способов начать работу с двоичным файлом Moonbeam. Вы можете самостоятельно скомпилировать двоичный файл, но весь процесс может занять около 30 минут, чтобы установить необходимые зависимости и собрать двоичный файл. Если Вам интересно использовать данный метод, посетите страницу нашей документации [Компиляция бинарного файла](/) 

Или Вы можете использовать [готовый бинарный файл](https://github.com/PureStake/moonbeam/releases) чтобы сразу приступить к работе.

Используйте утилиту `wget`, для загрузки последней версии двоичного файла:


=== "Moonbase Alpha"
    ```
    wget https://github.com/PureStake/moonbeam/releases/download/{{ networks.moonbase.parachain_release_tag }}/moonbeam
    ```

=== "Moonriver"
    ```
    wget https://github.com/PureStake/moonbeam/releases/download/{{ networks.moonriver.parachain_release_tag }}/moonbeam
    ``` 

Чтобы убедиться, в том, что Вы загрузили правильную версию, Вы можете запустить следующую команду `sha256sum moonbeam` в своем терминале, Вы должны получить следующий результат:

=== "Moonbase Alpha"
    ```
    {{ networks.moonbase.parachain_sha256sum }}
    ```

=== "Moonriver"
    ```
    {{ networks.moonriver.parachain_sha256sum }}
    ```

Загрузив двоичный файл, Вы можете использовать его для запуска службы systemd.

### Запуск Systemd Службы {: #running-the-systemd-service } 

Следующие команды выполнят необходимые действия для запуска службы.

Во-первых, давайте создадим учетную запись для запуска службы:

=== "Moonbase Alpha"
    ```
    adduser moonbase_service --system --no-create-home
    ```

=== "Moonriver"
    ```
    adduser moonriver_service --system --no-create-home
    ```

Затем создайте каталог в котором будет хранится наш бинарный файл и данные. Убедитесь, что Вы правильно установили владельца и права для локальной директории, в которой хранятся данные цепи:

=== "Moonbase Alpha"
    ```
    mkdir {{ networks.moonbase.node_directory }}
    chown moonbase_service {{ networks.moonbase.node_directory }}
    ```

=== "Moonriver"
    ```
    mkdir {{ networks.moonriver.node_directory }}
    chown moonriver_service {{ networks.moonriver.node_directory }}
    ```

Теперь скопируйте бинарный файл, созданный в последнем разделе, в уже созданную папку. Если Вы самостоятельно [скомпилировали двоичный файл](/node-operators/networks/compile-binary/), Вам необходимо скопировать его в целевой каталог (`./target/release/{{ networks.moonbase.binary_name }}`). В противном случае скопируйте двоичный файл Moonbeam в "root" директорию:

=== "Moonbase Alpha"
    ```
    cp ./{{ networks.moonbase.binary_name }} {{ networks.moonbase.node_directory }}
    ```

=== "Moonriver"
    ```
    cp ./{{ networks.moonriver.binary_name }} {{ networks.moonriver.node_directory }}
    ```

Следующим шагом будет создание конфигурационного файла systemd. Если Вы настраиваете узел коллатора, обязательно следуйте фрагментам кода для "Коллатора". Обратите внимание, что Вам необходимо:

 - Замените `YOUR-NODE-NAME` в двух разных местах
 - Дважды проверьте, что двоичный файл находится по правильному пути, как описано ниже (_ExecStart_)
 - Дважды проверьте стандартный путь, если Вы использовали другой каталог
 - Имя файла `/etc/systemd/system/moonbeam.service`

#### Полноценный узел {: #full-node } 

=== "Moonbase Alpha"
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
         --port {{ networks.parachain.p2p }} \
         --rpc-port {{ networks.parachain.rpc }} \
         --ws-port {{ networks.parachain.ws }} \
         --pruning=archive \
         --state-cache-size 1 \
         --base-path {{ networks.moonbase.node_directory }} \
         --chain {{ networks.moonbase.chain_spec }} \
         --name "YOUR-NODE-NAME" \
         -- \
         --port {{ networks.relay_chain.p2p }} \
         --rpc-port {{ networks.relay_chain.rpc }} \
         --ws-port {{ networks.relay_chain.ws }} \
         --pruning=archive \
         --name="YOUR-NODE-NAME (Embedded Relay)"

    [Install]
    WantedBy=multi-user.target
    ```

=== "Moonriver"
    ```
    [Unit]
    Description="Moonriver systemd service"
    After=network.target
    StartLimitIntervalSec=0
    
    [Service]
    Type=simple
    Restart=on-failure
    RestartSec=10
    User=moonriver_service
    SyslogIdentifier=moonriver
    SyslogFacility=local7
    KillSignal=SIGHUP
    ExecStart={{ networks.moonriver.node_directory }}/{{ networks.moonriver.binary_name }} \
         --port {{ networks.parachain.p2p }} \
         --rpc-port {{ networks.parachain.rpc }} \
         --ws-port {{ networks.parachain.ws }} \
         --pruning=archive \
         --state-cache-size 1 \
         --base-path {{ networks.moonriver.node_directory }} \
         --chain {{ networks.moonriver.chain_spec }} \
         --name "YOUR-NODE-NAME" \
         -- \
         --port {{ networks.relay_chain.p2p }} \
         --rpc-port {{ networks.relay_chain.rpc }} \
         --ws-port {{ networks.relay_chain.ws }} \
         --pruning=archive \
         --name="YOUR-NODE-NAME (Embedded Relay)"
    
    [Install]
    WantedBy=multi-user.target
    ```
#### Collator {: #collator } 

=== "Moonbase Alpha"
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
         --validator \
         --port {{ networks.parachain.p2p }} \
         --rpc-port {{ networks.parachain.rpc }} \
         --ws-port {{ networks.parachain.ws }} \
         --pruning=archive \
         --state-cache-size 1 \
         --base-path {{ networks.moonbase.node_directory }} \
         --chain {{ networks.moonbase.chain_spec }} \
         --name "YOUR-NODE-NAME" \
         -- \
         --port {{ networks.relay_chain.p2p }} \
         --rpc-port {{ networks.relay_chain.rpc }} \
         --ws-port {{ networks.relay_chain.ws }} \
         --pruning=archive \
         --name="YOUR-NODE-NAME (Embedded Relay)"

    [Install]
    WantedBy=multi-user.target
    ```

=== "Moonriver"
    ```
    [Unit]
    Description="Moonriver systemd service"
    After=network.target
    StartLimitIntervalSec=0

    [Service]
    Type=simple
    Restart=on-failure
    RestartSec=10
    User=moonriver_service
    SyslogIdentifier=moonriver
    SyslogFacility=local7
    KillSignal=SIGHUP
    ExecStart={{ networks.moonriver.node_directory }}/{{ networks.moonriver.binary_name }} \
         --validator \
         --port {{ networks.parachain.p2p }} \
         --rpc-port {{ networks.parachain.rpc }} \
         --ws-port {{ networks.parachain.ws }} \
         --pruning=archive \
         --state-cache-size 1 \
         --base-path {{ networks.moonriver.node_directory }} \
         --chain {{ networks.moonriver.chain_spec }} \
         --name "YOUR-NODE-NAME" \
         -- \
         --port {{ networks.relay_chain.p2p }} \
         --rpc-port {{ networks.relay_chain.rpc }} \
         --ws-port {{ networks.relay_chain.ws }} \
         --pruning=archive \
         --name="YOUR-NODE-NAME (Embedded Relay)"
    
    [Install]
    WantedBy=multi-user.target
    ```

!!! примечание
    Вы можете указать собственный порт Prometheus с помощью флага `--prometheus-port XXXX` (заменив `XXXX` на необходимый номер порта). Это возможно как для парачейна, так и для встроенной цепи передачи.

Почти готово! Зарегистрируйтесь и запустите службу, выполнив команды:

```
systemctl enable moonbeam.service
systemctl start moonbeam.service
```

И, наконец, убедитесь, что служба запущена:

```
systemctl status moonbeam.service
```

![Service Status](/images/fullnode/fullnode-binary1.png)

Вы также можете проверить журнал событий, выполнив:

```
journalctl -f -u moonbeam.service
```

![Service Logs](/images/fullnode/fullnode-binary2.png)

## Дополнительные флаги и параметры {: #advanced-flags-and-options } 

--8<-- 'text/setting-up-node/advanced-flags.md'

## Обновление клиента {: #updating-the-client } 

Поскольку разработка Moonbeam продолжается, иногда будет необходимо обновить программное обеспечение на Вашем сервере. Операторы узлов будут уведомлены на нашем [Discord канале](https://discord.gg/PfpUATX) когда обновления будут доступны и необходимо ли их применять (некоторые обновления клиента не являются обязательными). Процесс обновления достаточно простой и одинаков как для полноценной ноды так и для коллатора.

Сначала остановите докер контейнер или службу systemd:

```
sudo docker stop `CONTAINER_ID`
# или
sudo systemctl stop moonbeam
```

Затем установите новую версию, повторив шаги, описанные ранее, и убедитесь, что Вы используете последний доступный тег. После обновления Вы можете снова запустить службу.

### Очистка цепи {: #purging-the-chain } 

Время от времени Moonbase Alpha может быть очищена и сброшена после крупных обновлений. Как всегда, операторы узлов будут уведомлены заранее (через наш [Discord канал](https://discord.gg/PfpUATX)) если это обновление будет требовать очистки. Вы также можете очистить свой узел, если Ваш индивидуальный каталог данных будет поврежден.

Для этого сначала остановите докер контейнер или службу systemd:

```
sudo docker stop `CONTAINER_ID`
# или
sudo systemctl stop moonbeam
```

Затем удалите содержимое папки, в которой хранятся данные цепи (как для парачейна, так и для цепи передачи данных):

```
sudo rm -rf {{ networks.moonbase.node_directory }}/*
```

Наконец, установите самую новую версию, выполнив шаги, описанные ранее, и убедитесь, что Вы используете последний доступный тег. Если это так, Вы можете запустить новый узел со свежим каталогом данных.

## Телеметрия {: #telemetry } 

Чтобы включить сервер телеметрии Вашего узла Moonbase Alpha или Moonriver, Вы можете следовать [этому руководству](/node-operators/networks/telemetry/).

Запускать телеметрию на полноценном узле не обязательно. Однако это необходимо для коллаторов.

Кроме того, Вы можете проверить текущие данные в [телеметрии Moonbase Alpha](https://telemetry.polkadot.io/#list/Moonbase%20Alpha) и [Moonriver телеметрии](https://telemetry.polkadot.io/#list/Moonriver).

## Журналирование и устранение неполадок {: #logs-and-troubleshooting } 

Вы увидите журнал событий как из цепочки передачи данных, так и из парачейна. Цепь передачи будет иметь префикс «[Relaychain]», в то время как парачейн не имеет префикса.

### P2P Ports Not Open {: #p2p-ports-not-open } 

Если Вы не видите сообщение `Imported` (без тега `[Relaychain]`), Вам необходимо проверить конфигурацию порта P2P. P2P-порт должен быть открыт для входящего трафика.

### В синхронизации {: #in-sync } 

Обе цепи должны быть постоянно синхронизированы, и Вы должны видеть сообщения `Imported` или `Idle` и иметь подключенные одноранговые узлы.

### Несоответствие генезиса {: #genesis-mismatching } 

Сеть тестирования Moonbase Alpha TestNet часто обновляется. Следовательно, Вы можете увидеть следующее сообщение:

```
DATE [Relaychain] Bootnode with peer id `ID` is on a different
chain (our genesis: GENESIS_ID theirs: OTHER_GENESIS_ID)
```

Обычно это означает, что Вы используете устаревшую версию и Вам необходимо ее обновить.

Мы сообщаем об обновлениях (и об необходимости очистки сети) через наш [Discord канал](https://discord.gg/PfpUATX) минимум за 24 часа.

