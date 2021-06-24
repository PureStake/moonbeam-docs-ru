---
title: Нода Chainlink
description: Как настроить ноду Chainlink Oracle для Moonbeam Network, в целях передачи данных в цепочке, для использования смарт-контрактами
---

# Запуск ноды Chainlink Oracle на Moonbeam

![Chainlink Moonbeam](/images/chainlink/chainlinknode-banner.png)

## Вступление

Для предоставления данных смарт-контрактов, работающих на Moonbeam может быть использован оракул, как открытая сеть, не требующая дополнительных разрешений.

В этой статье представлен гайд по настройке Chainlink Oracle на Moonbase Alpha.

!!! примечание 
    Примеры приведены только в демонстрационных целях. Управление паролями **ДОЛЖНО** отвечать стандартам безопасности, а сами пароли никогда не должны храниться в простых текстовых файлах. В данных примерах используется среда на базе Ubuntu 18.04, но включены также и примеры для macOS. Это руководство предназначено только для разработки, не используйте его для продакшена.

## Базовая модель запроса

Прежде чем мы приступим к началу работы, важно понять основы "базовой модели запроса".

--8<-- 'text/chainlink/chainlink-brm.md'

## Продвинутые пользователи

Если Вы знакомы с запуском нод Chainlink Oracle, эта информация поможет Вам быстро начать работу с Moonbase Alpha TestNet:

 - Документацию Chainlink Вы можете найти [здесь](https://docs.chain.link/docs/running-a-chainlink-node)
 - Moonbase Alpha WSS EndPoint: `wss://wss.testnet.moonbeam.network`
 - Moonbase Alpha ChainId: `1287`
 - LINK Token в Moonbase Alpha: `0xa36085F69e2889c224210F603D836748e7dC0088`
 - Получите токены Moonbase Alpha через [наш кран](/getting-started/moonbase/faucet/)

## Приступаем к работе

В этом руководстве будет описан процесс настройки ноды Oracle. Его можно кратко представить следующим образом:

 - Натсройка ноды Chainlink, подключенной к Moonbase Alpha
 - Пополнение ноды
 - Размещение контракта Oracle
 - Создание job на ноде Chainlink
 - Связывание ноды и Oracle
 - Тестирование с использованием клиентского контракта

Основные требования:

 - Докер для запуска контейнеров нод Postgres DB и ChainLink. Для получения дополнительной информации по установке докера, пожалуйста, посетите [эту страницу](https://docs.docker.com/get-docker/)
 - Учетная запись с денежными средствами. Вы можете создать ее с помощью [Metamask](/integrations/wallets/metamask/), который можно пополнить через [этот кран](https://docs.moonbeam.network/getting-started/moonbase/faucet/)
 - Доступ к Remix IDE на случай, если Вы захотите использовать его для размещения контракта Oracle. Вы можете найти более подробную информацию о Remix на Moonbeam [здесь](/integrations/remix/)

## Настройка ноды

Во-первых, давайте создадим новый каталог для размещения всех необходимых файлов. Например:

```
mkdir -p ~/.chainlink-moonbeam //
cd ~/.chainlink-moonbeam
```

Далее, давайте создадим Postgres DB с помощью докера. Для этого выполните следующую команду (MacOs пользователи могут заменить `--network host \` на `-p 5432:5432`):

```
docker run -d --name chainlink_postgres_db \
    --volume chainlink_postgres_data:/var/lib/postgresql/data \
    -e 'POSTGRES_PASSWORD={YOU_PASSWORD_HERE}' \
    -e 'POSTGRES_USER=chainlink' \
    --network host \
    -t postgres:11
```

Обязательно замените `{YOU_PASSWORD_HERE}` актуальным паролем.

!!! примечание 
    Напоминаем, что хранить пароли в текстовом файле небезопасно. Примеры приведены только в демонстрационных целях.

Docker продолжит скачивание необходимых файлов, если они недоступны. Теперь нам нужно создать файл среды для Chainlink во вновь созданном каталоге. Этот файл считывается при создании контейнера Chainlink. Пользователи MacOs могут заменить `localhost` на `host.docker.internal`.

```
echo "ROOT=/chainlink
LOG_LEVEL=debug
ETH_CHAIN_ID=1287
MIN_OUTGOING_CONFIRMATIONS=2
LINK_CONTRACT_ADDRESS={LINK TOKEN CONTRACT ADDRESS}
CHAINLINK_TLS_PORT=0
SECURE_COOKIES=false
GAS_UPDATER_ENABLED=false
ALLOW_ORIGINS=*
ETH_URL=wss://wss.testnet.moonbeam.network
DATABASE_URL=postgresql://chainlink:{YOUR_PASSWORD_HERE}@localhost:5432/chainlink?sslmode=disable
MINIMUM_CONTRACT_PAYMENT=0" > ~/.chainlink-moonbeam/.env
```

Здесь, помимо пароля (`{YOUR_PASSWORD_HERE}`), нам необходимо предоставить контракт LINK токена (`{LINK TOKEN CONTRACT ADDRESS}`). После создания файла среды нам также понадобится файл `.api`, в котором хранятся пользователь и пароль, используемые для доступа к API ноды, пользовательскому интерфейсу оператора ноды и командной строке Chainlink.

```
echo "{AN_EMAIL_ADDRESS}" >  ~/.chainlink-moonbeam/.api
echo "{ANOTHER_PASSWORD}"   >> ~/.chainlink-moonbeam/.api
```

Задайте адрес электронной почты и новый пароль. Теперь, нам нужен еще один файл, в котором хранится пароль кошелька для адреса ноды:

```
echo "{THIRD_PASSWORD}" > ~/.chainlink-moonbeam/.password
```

Теперь, когда мы закончили создавать все необходимые файлы, мы можем запустить контейнеры следующей командой (пользователи macOS могут заменить `--network host \` на `-p 6688:6688`):

```
docker run -d --name chainlink_oracle_node \
    --volume $(pwd):/chainlink \
    --env-file=.env \
    --network host \
    -t smartcontract/chainlink:0.9.2 \
        local n \
        -p /chainlink/.password \
        -a /chainlink/.api
```

Чтобы убедиться, что все работает и что логи пишутся, используйте:

```
docker ps #Containers Running
docker logs --tail 50 {container_id} #Logs progressing
```

![логи Docker](/images/chainlink/chainlinknode-image1.png)

## Настройка контракта

Когда нода Oracle запущена, давайте настроим смарт-контракт.

Во-первых, нам нужно получить адрес, который нода Оракула будет использовать для отправки транзакций и записи данных в чейн. Чтобы получить адрес, войдите в [пользовательский интерфейс ноды ChainLink](http://localhost:6688/) (расположенный по адресу `http://localhost:6688/`) используя учетные данные из файла `.api`.

![вход в Chainlink](/images/chainlink/chainlinknode-image2.png)

Перейдите на "Configuration Page" и скопируйте адрес ноды. Используйте [кран Moonbeam](https://docs.moonbeam.network/getting-started/moonbase/faucet/) для пополнения.

![адрес Chainlink](/images/chainlink/chainlinknode-image3.png)

Затем нам нужно разместить контракт Oracle, который является промежуточным ПО между чейном и нодой. Контракт отдает событие со всей необходимой информацией, которая считывается нодой Oracle. Затем нода выполняет запрос и записывает запрашиваемые данные в контракт вызывающего.

Исходный код контракта Oracle можно найти в официальном репозитории Chainlink на GitHub [здесь](https://github.com/smartcontractkit/chainlink/tree/develop/evm-contracts/src/v0.6). В этом примере мы будем использовать Remix для взаимодействия с Moonbase Alpha и размещения контракта. В Remix мы можем скопировать следующий код:

```
pragma solidity ^0.6.6;

import "https://github.com/smartcontractkit/chainlink/evm-contracts/src/v0.6/Oracle.sol";
```

После компиляции контракта перейдите на вкладку "Deploy and Run Transactions", введите адрес LINK токена и разместите контракт. После размещения скопируйте адрес контракта.

![размещение Oracle с использованием Remix](/images/chainlink/chainlinknode-image4.png)

Теперь, мы должны связать ноду Oracle со смарт-контрактом. Нода может отлавливать запросы, отправленные в определенный контракт Oracle, но только авторизованные (или связанные) ноды могут выполнить запрос с предоставлением результата.

Для авторизации мы можем использовать функцию `setFulfillmentPermission()` из контракта Oracle. Для этого нужны два параметра:

 - Адрес ноды, которую мы хотим связать с контрактом (что мы сделали на предыдущем шаге)
 - Логическое значение, указывающее статус привязки. В этом случае мы устанавливаем значение в `true`

Для этого мы можем использовать экземпляр контракта, развернутого на Remix, и проверить, авторизована ли нода Oracle с помощью функции `getAuthorizationStatus()`, передав адрес ноды Oracle.

![Авторизация ноды Chainlink Oracle](/images/chainlink/chainlinknode-image5.png)

## Создание Job на ноде Oracle

Последним шагом настройки Оракула Chainlink является создание Job. Ссылаясь на [официальную документацию Chainlink](https://docs.chain.link/docs/job-specifications):

> Спецификации Job или спеки - содержат последовательные задачи, которые нода должна выполнить для получения конечного результата. Спецификация содержит по крайней мере один инициатор и одну задачу, которые подробно описаны ниже. Спецификации определяются с использованием стандартного JSON, для удобства чтения и легкости анализа нодой Chainlink.

Рассматривая Оракул как сервис API, Job здесь будет одной из функций, которую мы можем вызвать, и которая вернет результат. Чтобы создать нашу первую Job, перейдите в разделы [Jobs вашей ноды](http://localhost:6688/jobs) и нажмите на "New Job."

![Jobs ноды Chainlink](/images/chainlink/chainlinknode-image6.png)

Затем вставьте следующий JSON. Это создаст Job, которая запросит текущую цену ETH в долларах США. Убедитесь, что Вы ввели свой адрес контракта Oracle (`YOUR_ORACLE_CONTRACT_ADDRESS`).

```
{
  "initiators": [
    {
      "type": "runlog",
      "params": { "address": "YOUR_ORACLE_CONTRACT_ADDRESS" }
    }
  ],
  "tasks": [
    {
      "type": "httpget",
      "params": { "get": "https://min-api.cryptocompare.com/data/price?fsym=ETH&tsyms=USD" }
    },
    {
      "type": "jsonparse",
      "params": { "path": [ "USD" ] }
    },
    {
      "type": "multiply",
      "params": { "times": 100 }
    },
    { "type": "ethuint256" },
    { "type": "ethtx" }
  ]
}
```

![Chainlink New Job JSON Blob](/images/chainlink/chainlinknode-image7.png)

Вот и все! Вы полностью настроили ноду Chainlink Oracle, работающую на Moonbase Alpha.

## Test the Oracle

Чтобы убедиться, что Oracle работает и отвечает на запросы, то следуйте нашему руководству [с использованием Oracle](/integrations/oracles/chainlink/). Основная идея состоит в том, чтобы разместить клиентский контракт, который обращается к Oracle, а Oracle записывает запрошенные данные в хранилище контракта.

