---
title: Настройка ноды
description: Воспользуйтесь данным руководством, для настройки своего первого узла Moonbeam. Вы также узнаете, как его подключить и управлять им с помощью пользовательского графического интерфейса Polkadot JS.
---

# Настройка ноды Moonbeam и её подключение к графическому интерфейсу Polkadot JS

<style>.embed-container { position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; } .embed-container iframe, .embed-container object, .embed-container embed { position: absolute; top: 0; left: 0; width: 100%; height: 100%; }</style><div class='embed-container'><iframe src='https://www.youtube.com/embed//p_0OAHSlHNM' frameborder='0' allowfullscreen></iframe></div>
<style>.caption { font-family: Open Sans, sans-serif; font-size: 0.9em; color: rgba(170, 170, 170, 1); font-style: italic; letter-spacing: 0px; position: relative;}</style><div class='caption'>Вы можете найти весь необходимый код касающейся этого руководства в <a href="{{ config.site_url }}resources/code-snippets/">фрагментах кода</a></div>

## Вступление

В этом руководстве описаны шаги, по созданию ноды для тестирования совместимости с Ethereum и функциональности Moonbeam.

!!! Примечание
    Это руководство было создано с использованием {{ networks.development.build_tag }} тега [Moonbase Alpha](https://github.com/PureStake/moonbeam/releases/tag/{{ networks.development.build_tag }}). Платформа Moonbeam и компоненты [Frontier](https://github.com/paritytech/frontier) которые она использует для совместимости с Ethereum на основе Substrate, все еще находятся в стадии активной разработки.
    --8<-- 'text/common/assumes-mac-or-ubuntu-env.md'

Узел разработки Moonbeam - это Ваша собственная среда разработчика для создания и тестирования приложений на Moonbeam. Для разработчиков Ethereum это сравнимо с Ganache. Это позволяет Вам быстро и легко начать работу без каких-либо издержек на reley chain. Вы можете развернуть свой узел с помощью опции `--sealing`, для мгновенного создания блоков, вручную или с заданным интервалом после получения транзакций. По умолчанию при получении транзакции создается блок, что похоже на функцию Instamine в Ganache. 

Следуя данной инструкции от начала и до конца, Вы получите узел разработки Moonbeam, работающий в Вашей локальной среде, с 10-ю [предварительно профинансированными счетами разработки](#Предварительно-профинансированные-счета-разработки), и сможете подключить его к стандартному графическому интерфейсу Polkadot JS.

Есть два способа запустить узел Moonbeam: Вы можете использовать [докер для запуска предварительно созданного бинарного файла](#Запуск-Moonbeam-с-ипользованием-Docker) или Вы можете [скомпилировать бинарный файл локально](#Запуск-Moonbeam-из-исходного-кода) и самостоятельно настроить узел для разработки. Использование Docker - это быстрый и удобный способ для того чтобы начать работу, так как Вам не нужно устанавливать Substrate и все зависимости, и Вы также можете пропустить процесса создание ноды вручную. Это потребует от Вас [установить докер](https://docs.docker.com/get-docker/). С другой стороны, если Вы решите, пройти процесс создания собственного узла для разработки, это может занять около 30 минут или больше в зависимости от Вашего оборудования.

## Запуск Moonbeam с ипользованием Docker

Использование Docker позволяет развернуть ноду за считанные секунды. После того, как Вы установили Docker, Вы можете выполнить следующую команду, чтобы загрузить необходимый образ:

```
docker pull purestake/moonbeam:{{ networks.development.build_tag }}
```

Последние строки журнала событий в консоли должен выглядеть так:

![Docker - imaged pulled](/images/setting-up-a-node/setting-up-node-1.png)

После загрузки Docker-образа следующим шагом будет его запуск.

Вы можете запустить образ Docker, используя следующую команду:

=== "Ubuntu"
    ```
    docker run --rm --name {{ networks.development.container_name }} --network host \
    purestake/moonbeam:{{ networks.development.build_tag }} \
    --dev
    ```

=== "MacOS"
    ```
    docker run --rm --name {{ networks.development.container_name }} -p 9944:9944 -p 9933:9933 \
    purestake/moonbeam:{{ networks.development.build_tag }} \
    --dev --ws-external --rpc-external
    ```

=== "Windows"
    ```
    docker run --rm --name {{ networks.development.container_name }} -p 9944:9944 -p 9933:9933 \
    purestake/moonbeam:{{ networks.development.build_tag }} \
    --dev --ws-external --rpc-external
    ```

Данная команда запустит узел разработки Moonbeam в режиме локального тестирования, таким образом блоки создаются мгновенно по мере получения транзакций.
В случае положительного результата Вы должны увидеть вывод, отражающий режим ожидания для создания новых блоков:

![Docker - вывод показывает создаваемые блоки](/images/setting-up-a-node/setting-up-node-2.png)

Для получения дополнительной информации о некоторых флагах и параметрах, использованных в примере, ознакомьтесь с [общими флагами и параметрами](#Общие-команды-флаги-и-параметры). Если Вы хотите увидеть полный список всех флагов, параметров и sub-команд, откройте меню справки, выполнив:

```
docker run --rm --name {{ networks.development.container_name }} \
purestake/moonbeam \
--help
```

Чтобы продолжить изучение, Вам не нужен будет следующий раздел, так как Вы уже создали узел с помощью Docker. Вы можете перейти к [Подключение приложений Polkadot JS к локальному узлу Moonbeam](#Подключение-приложений-Polkadot-JS-к-локальному-узлу-Moonbeam).

## Запуск Moonbeam из исходного кода

!!! обратите внимание
    Если Вы знаете, что делаете, Вы можете напрямую загрузить предварительно скомпилированные бинарные файлы, поставляемые к каждому выпуску, на [Moonbeam-release page](https://github.com/PureStake/moonbeam/releases). Данные файлы не будут работать во всех системах. Например, двоичные файлы работают только в Linux x86-64 версиях и с определенными версиями зависимостей. Самый безопасный способ обеспечить совместимость - это скомпилировать бинарный файл непосредственно в той системе, в которой будет происходить запуск ноды.

Во-первых, начните с клонирования определенного тега репозитория Moonbeam, который Вы можете найти здесь:

[https://github.com/PureStake/moonbeam/](https://github.com/PureStake/moonbeam/)

```
git clone -b {{ networks.development.build_tag }} https://github.com/PureStake/moonbeam
cd moonbeam
```

Затем установите Substrate и все необходимые компоненты (включая Rust), выполнив:

```
--8<-- 'code/setting-up-node/substrate.md'
```

После того, как Вы выполнили все описанные выше шаги, Вы можете создать узел для разработки, запустив:

```
--8<-- 'code/setting-up-node/build.md'
```

Если в терминале появляется сообщение _cargo not found error_, Вам необходимо будет вручную добавить Rust в системные пути (или перезапустить сервер):

```
--8<-- 'code/setting-up-node/cargoerror.md'
```

!!! обратите внимание
    Первоначальная сборка продлится некоторое время. Время сборки будет зависить от Вашего оборудования, примерное время ожидания будет составлять около 30 минут.

Вот как должен выглядеть процесс окончания сборки:

![End of build output](/images/setting-up-a-node/setting-up-node-3.png)

Затем Вам необходимо будет запустить узел в режиме разработки, используя следующую команду:

```
--8<-- 'code/setting-up-node/runnode.md'
```

!!! обратите внимание
    Для людей, не знакомых с Substrate, `--dev` флаг позволяет запустить узел на основе substrate в режиме разработчика с одним узлом в целях тестирования. Вы можете узнать больше о `--dev` режиме в [этом реководстве по Substrate](https://substrate.dev/docs/en/tutorials/create-your-first-substrate-chain/interact).

Вы должны увидеть следующий вывод, который будет отображать состояние ожидания, до тех пор пока не будут созданы блоки:

![Output shows blocks being produced](/images/setting-up-a-node/setting-up-node-4.png)

Для получения дополнительной информации о некоторых флагах и параметрах, которые использовались в примерах, посетите страницу [Общие флаги и Опции](#common-flags-and-options). Если Вы хотите увидеть полный список всех флагов, параметров и sub-команд, откройте меню справки, выполнив:

```
./target/release/moonbeam --help
```
## Подключение приложений Polkadot JS к локальному узлу Moonbeam

Узел разработки - это узел на основе substrate, поэтому Вы можете взаимодействовать с ним, используя стандартные инструменты substrate. Ниже представлены два RPC адреса, которые Вы можете использовать для подключения: 

 - HTTP: `http://127.0.0.1:9933`
 - WS: `ws://127.0.0.1:9944` 

Начнем с подключения к нему с помощью Polkadot JS Apps. Откройте браузер: [https://polkadot.js.org/apps/#/explorer](https://polkadot.js.org/apps/#/explorer). Откроется приложение Polkadot JS Apps, которое автоматически подключается к Polkadot MainNet.

![Polkadot JS Apps](/images/setting-up-a-node/setting-up-node-5.png)

Щелкните в верхнем левом углу, чтобы открыть меню для настройки сетей, а затем перейдите вниз, чтобы открыть меню «Разработка» (Development sub-menu). ам вы должны переключить опцию «Локальная нода»(Local Node), которая указывает приложениям Polkadot JS на `ws://127.0.0.1:9944`. Затем нажмите кнопку «Переключить»(Switch), и сайт должен подключиться к вашей ноде Moonbeam.

![Select Local Node](/images/setting-up-a-node/setting-up-node-6.png)

При подключении Polkadot JS Apps Вы увидите ноду Moonbeam, создающую блоки.

![Select Local Node](/images/setting-up-a-node/setting-up-node-7.png)

## Запрос состояния учетной записи

С выпуском [Moonbase Alpha v3](https://www.purestake.com/news/moonbeam-network-upgrades-account-structure-to-match-ethereum/), Moonbeam теперь работает в формате единой учетной записи, который представляет собой H160 в стиле Ethereum и теперь также поддерживается в приложениях Polkadot JS. Чтобы проверить баланс адреса, вы можете просто импортировать свою учетную запись на вкладку «Учетные записи»(Accounts tab). Вы можете найти более подробную информацию в секции [Единые учетные записи](/learn/unified-accounts/).
 
Тем не менее, используя все возможности Ethereum RPC Moonbeam, Вы также можете использовать [MetaMask](/getting-started/local-node/using-metamask/) для проверки баланса этого адреса. Кроме того, Вы также можете использовать другие инструменты разработки, такие как [Remix](/getting-started/local-node/using-remix/) и [Truffle](/getting-started/local-node/using-truffle/).

## Общие команды, флаги и параметры

### Очистка цепи

При запуске узла с помощью бинарного файла, данные хранятся в директории, которая обычно расположена в `~/.local/shared/moonbeam/chains/development/db`. If you want to start a fresh instance of the node, you can either delete the content of the folder, or run the following command inside the `moonbeam` folder:

```
./target/release/moonbeam purge-chain --dev -y
```

Это приведет к удалению папки с данными, обратите внимание, что с этого момента все данные цепочки потеряны.

Если Вы использовали Docker, в этом случае папка данных связана с самим контейнером Docker.
### Флаги узлов

Флаги не принимают аргументы, чтобы использовать флаг, добавьте его в конец команды. Например:

```
--8<-- 'code/setting-up-node/runnode.md'
```

- `--dev`: Определяет цепь разработки
- `--no-telemetry`: Отключить подключение к серверу телеметрии Substrate. Для глобальных сетей, телеметрия включена по умолчанию. Телеметрия недоступна, если Вы работаете в   режиме разработке (`--dev`).
- `--tmp`: Запустить временный узел, в котором вся конфигурация будет удалена по окончанию работы процесса.
- `--rpc-external`: Прослушивать все RPC интерфейсы
- `--ws-external`: Прослушивать все Websocket интерфейсы 

### Параметры узла

Опции принимают аргумент справа от опции. Например:

```
--8<-- 'code/setting-up-node/runnodewithsealinginterval.md'
```

- `-l <log pattern>` или `--log <log pattern>`: Устанавливает настраиваемый фильтр ведения журнала. Синтаксис шаблона для журнала событий: `<target>=<level>`. Например, чтобы распечатать все RPC журналы, команда будет выглядеть так: `-l rpc=trace`.
- `--sealing <interval>`: Когда блоки должны быть запечатаны в сервисе разработки. Допустимые аргументы для значения интервала: `instant`, `manual`, или число, представляющее интервал таймера в миллисекундах (например, `6000` узел будет производить блоки каждые 6 секунд). По умолчанию `instant`.
- `--rpc-port <port>`: Устанавливает TCP-порт для HTTP RPC-сервера. Принимает порт в качестве аргумента.
- `--ws-port <port>`: Устанавливает TCP-порт для WebSockets RPC-сервера. Принимает порт в качестве аргумента.

Чтобы получить полный список всех флагов и параметров, для Вашей Moonbeam ноды, добавьте в конце команды `--help`.

## Расширенные флаги и параметры

--8<-- 'text/setting-up-node/advanced-flags.md'

Например, при запуске бинарного файла:

```
./target/release/moonbeam --dev --execution=Native --ethapi=debug,trace
```

## Предварительно профинансированные счета разработки

Ваш узел разработки Moonbeam поставляется с десятью предварительно профинансированными учетными записями для разработки. Адреса взяты из substrate в виде мнемоник фраз:

```
bottom drive obey lake curtain smoke basket hold race lonely fit walk
```

--8<-- 'code/setting-up-node/dev-accounts.md'

Ознакомьтесь с разделом [Использовать MetaMask](/getting-started/local-node/using-metamask/) для начала работы со своими аккаунтами.

Кроме того, в узел разработки входит предварительно профинансированная учетная запись, используемая в тестовых целях:

--8<-- 'code/setting-up-node/dev-testing-account.md'
