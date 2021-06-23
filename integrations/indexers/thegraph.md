---
title: The Graph
description: Создавайте API с использованием протокола индексирования Graph на Moonbeam
---

# Использование The Graph на Moonbeam

![The Graph на Moonbeam](/images/thegraph/thegraph-banner.png)

## Вступление

Протоколы индексирования структурируют информацию таким образом, чтобы приложения могли получить к ней доступ наиболее эффективным способом. Например, Google выполняет индексацию всей сети Интернет, чтобы быстро предоставлять информацию, когда Вы что-то ищете. 

The Graph - это децентрализованный протокол индексации с открытым исходным кодом для выполнения запросов к таким сетям, как Ethereum. Простыми словами, он обеспечивает способ эффективного хранения данных, генерируемых событиями из смарт-контрактов, чтобы другие проекты или приложения могли легко получить к ним доступ. 

Кроме того, разработчики могут создавать API, именуемые как Subgraph(ы). Пользователи или другие разработчики могут использовать Subgraph(ы) для запроса данных, относящихся к набору смарт-контрактов. Данные извлекаются с помощью стандартных API запросов с использованием - GraphQL. Вы можете посетить страницу с [официальной документацией](https://thegraph.com/docs/introduction#what-the-graph-is), чтобы узнать больше о протоколе The Graph.

С внедрением модулей трассировки Ethereum в [Moonbase Alpha v7](https://github.com/PureStake/moonbeam/releases/tag/v0.7.0), The Graph может индексировать данные блокчейна в Moonbeam.

В этом руководстве мы рассмотрим процесс создания простого Subgraph для контракта "Lottery" на Moonbase Alpha.


## Проверка предварительных условий

У Вас есть два способа, чтобы использовать The Graph на Moonbase Alpha:

 - Запустить ноду The Graph с Moonbase Alpha, настроив свой Subgraph ссылкой на него. Чтобы это сделать используйте [данное руководство](/node-operators/indexers/thegraph-node/)
 - Привязать свой Subgraph к API The Graph через [Graph Explorer website](https://thegraph.com/explorer/). Для этого Вам необходимо создать учетную запись и получить ключ доступа.

## Lottery контракт

В качестве примера мы рассмотрим использование простого контракта Lottery. Вы можете найти файл Solidity используя [эту ссылку](https://github.com/PureStake/moonlotto-subgraph/blob/main/contracts/MoonLotto.sol). 

В контракте проводится лотерея, где игроки могут купить билеты для себя или подарить их другому пользователю. По прошествию 1-го часа, если набралось 10 участников, следующий игрок, который присоединяется к лотерее, вызовет функцию, которая выберет победителя. Все средства, хранящиеся в контракте, будут отправлены победителю, после чего начнется новый раунд.

Основные функции контракта:

 - **joinLottery** — нет вхождений. Функция для входа в текущий раунд лотереи, значение (количество токенов), отправленное на контракт, должно быть равно цене билета
 - **giftTicket** —  одно вхождение: адрес получателя билета. Функция аналогична `joinLottery`, но для владельца билета можно указать другой адрес
 - **enterLottery** — одно вхождение: адрес владельца билета. Внутренняя функция, которая обрабатывает логику лотерейных билетов. Если прошел час и есть не менее 10 участников, вызывается функция `pickWinner`
 - **pickWinner** — нет вхождений. Внутренняя функция, которая выбирает победителя лотереи с помощью генератора псевдослучайных чисел (небезопасно, только для демонстрационных целей). Она обрабатывает логику перевода средств и сброса переменной для следующего раунда лотереи.

### События контракта Lottery

The Graph - использует события, генерируемые контрактом, для индексации данных. Контракт Lottery включает в себя только два события:

 - **PlayerJoined** — вызывается в функции `enterLottery`. Предоставляет информацию, которая относится к последней лотерее, такую как адрес игрока, текущий раунд лотереи, был ли подарен билет и сумма приза в текущем раунде
 - **LotteryResult** — вызывается в функции `pickWinner`. Предоставляет информацию о розыгрыше текущего раунда, такую как адрес победителя, текущий раунд лотереи, был ли выигрышный билет подарком, сумму приза и отметку времени розыгрыша

## Создание Subgraph

В этом разделе рассматривается процесс создания Subgraph. Для Lottery Subgraph был подготовлен [репозиторий GitHub](https://github.com/PureStake/moonlotto-subgraph)со всем необходимым, чтобы помочь Вам начать работу. Репозиторий включает контракт Lottery, а также файл конфигурации Hardhat и сценарий размещения. Если Вы не знакомы с ним, Вы можете ознакомиться с нашим [руководством по интеграции Hardhat](/integrations/hardhat/). 

Для начала клонируйте репозиторий и установите необходимые зависимости:

```
git clone https://github.com/PureStake/moonlotto-subgraph \
&& cd moonlotto-subgraph && yarn
```

Теперь Вы можете создать типы TypeScript для The Graph, выполнив:

```
npx graph codegen --output-dir src/types/
```

!!! примечание 
    Для создания типов необходимо, чтобы файлы ABI были определены в файле  `subgraph.yaml`. В этом экземпляре репозитория уже присутствует файл, но обычно он создается после компиляции контракта. Дополнительную информацию Вы можете найти в [репозитории Moonlotto](https://github.com/PureStake/moonlotto-subgraph).

Вы можете выполнить команду `codegen` с помощью `yarn codegen`. 

В этом примере контракт был размещен на `{{ networks.moonbase.thegraph.lotto_contract }}`. Вы можете найти дополнительную информацию о том, как разместить контракт с Hardhat, в [руководстве по интеграциям](/integrations/hardhat/). Кроме того, файл "README"" в [репозитории Moonloto](https://github.com/PureStake/moonlotto-subgraph) содержит шаги, необходимые для компиляции и размещения контракта, если это потребуется. 

### Основная структура Subgraphs

В общих чертах, Subgraphs определяют данные, которые The Graph будет индексировать из цепочки блоков, а также способ их хранения. Subgraphs, как правило, содержат следующие файлы:

 - **subgraph.yaml** — это YAML файл, который содержит [манифест Subgraph(а)](https://thegraph.com/docs/define-a-subgraph#the-subgraph-manifest), т.е информацию, относящуюся к смарт-контрактам, индексируемым Subgraph.
 - **schema.graphql** — это файл [схемы GraphQL](https://thegraph.com/docs/define-a-subgraph#the-graphql-schema), который определяет хранилище данных для создаваемого Subgraph и его структуру. Он написан с использованием[схемы определения интерфейса GraphQL](https://graphql.org/learn/schema/#type-language)
 - **AssemblyScript mappings** — код в TypeScript (скомпилированный в [AssemblyScript](https://github.com/AssemblyScript/assemblyscript)), который используется для преобразования данных событий из контракта к субъектам, определенным в схеме.

При изменении файлов для создания Subgraph нет определенного порядка.

### Schema.graphql

Перед изменением `schema.graphql` важно подчеркнуть, какие данные необходимо извлечь из событий контракта. Схемы необходимо определять с учетом требований самого dApp. В этом примере, хотя приложение dApp не связано с лотереей, определены следующие объекты: 

 - **Round** — относится к розыгрышу лотереи. В нем хранится индекс раунда, присужденный приз, отметка времени начала раунда, отметка времени, когда был выбран победитель, и информация об участвующих билетах, полученная от объекта `Ticket`
 - **Player** — относится к игроку, который участвовал хотя бы в одном раунде. Он хранит свой адрес и информацию обо всех участвующих билетах, полученную от объекта `Ticket`
 - **Ticket** — относится к билетам для участия в розыгрыше лотереи. Он хранит информацию о том, был ли билет подарен, адрес владельца, раунд, с которого билет действителен, и был ли это выигрышный билет

Одним словом, `schema.graphql` должна выглядеть следующим образом:

```
type Round @entity {
  id: ID!
  index: BigInt!
  prize: BigInt! 
  timestampInit: BigInt!
  timestampEnd: BigInt
  tickets: [Ticket!] @derivedFrom(field: "round")
}

type Player @entity {
  id: ID!
  address: Bytes!
  tickets: [Ticket!] @derivedFrom(field: "player")
}

type Ticket @entity {
  id: ID!
  isGifted: Boolean!
  player: Player!
  round: Round!
  isWinner: Boolean!
}
```

### Манифест Subgraph

Файл `subgraph.yaml` или манифест Subgraph содержит информацию, относящуюся к индексируемому смарт-контракту, включая события, которые содержат данные, необходимые для сопоставления. Эти данные затем сохраняются на нодах The Graph, что позволяет приложениям запрашивать их. 

Некоторые из наиболее важных параметров в файле `subgraph.yaml`:

 - **repository** — относится к репозиторию Github Subgraph
 - **schema/file** — ссылается на расположение файла `schema.graphql`
 - **dataSources/name** — относится к имени Subgraph
 - **network** — относится к имени сети. Это значение **должно** быть установлено в `mbase`для любого Subgraph, размещаемого в Moonbase Alpha
 - **dataSources/source/address** — относится к адресу интересующего контракта
 - **dataSources/source/abi** — указывает, где хранится интерфейс контракта внутри папки `types`, созданной с помощью команды `codegen`
 - **dataSources/source/startBlock** — относится к начальному блоку, с которого начнется индексация. В идеале это значение должно быть как можно ближе к блоку, в котором был создан контракт. Вы можете использовать [Blockscout](https://moonbase-blockscout.testnet.moonbeam.network/) чтобы получить эту информацию, указав адрес контракта. В этом примере контракт был создан на блоке `{{ networks.moonbase.thegraph.block_number }}`
 - **dataSources/mapping/file** — указывает на расположение файла сопоставления
 - **dataSources/mapping/entities** — относится к определениям сущностей в файле`schema.graphql`
 - **dataSources/abis/name** — указывает, где интерфейс контракта хранится внутри `types/dataSources/name`
 - **dataSources/abis/file** — относится к месту, где хранится файл`.json` с ABI контракта
 - **dataSources/eventHandlers** — здесь нет необходимости определять значение, но этот раздел относится ко всем событиям, которые The Graph будет индексировать.
 - **dataSources/eventHandlers/event** — относится к структуре события, которое будет отслеживаться внутри контракта. Вам необходимо указать название события и тип его переменных
 - **dataSources/eventHandlers/handler** — относится к имени функции внутри файла `mapping.ts`, который обрабатывает данные события
 
Вкратце, `subgraph.yaml` должен выглядеть как следующий фрагмент:

```
specVersion: 0.0.2
description: Moonbeam lottery subgraph tutorial
repository: https://github.com/PureStake/moonlotto-subgraph
schema:
  file: ./schema.graphql
dataSources:
  - kind: ethereum/contract
    name: MoonLotto
    network: mbase
    source:
      address: '{{ networks.moonbase.thegraph.lotto_contract }}'
      abi: MoonLotto
      startBlock: {{ networks.moonbase.thegraph.block_number }}
    mapping:
      kind: ethereum/events
      apiVersion: 0.0.4
      language: wasm/assemblyscript
      file: ./src/mapping.ts
      entities:
        - Player
        - Round
        - Ticket
        - Winner
      abis:
        - name: MoonLotto
          file: ./artifacts/contracts/MoonLotto.sol/MoonLotto.json
      eventHandlers:
        - event: PlayerJoined(uint256,address,uint256,bool,uint256)
          handler: handlePlayerJoined
        - event: LotteryResult(uint256,address,uint256,bool,uint256,uint256)
          handler: handleLotteryResult
```

### Сопоставления

Файлы сопоставлений - это то, что преобразует данные цепочки блоков в объекты, которые определены в файле схемы. Каждый обработчик событий внутри файла `subgraph.yaml` должен содержать вспомогательные функции сопоставления. 

Файл сопоставления, используемый для примера с контрактом Lottery, можно найти [по этой ссылке](https://github.com/PureStake/moonlotto-subgraph/blob/main/src/mapping.ts).

В целом, стратегия каждой функции-обработчика состоит в том, чтобы загрузить данные о событии, проверить, существует ли уже запись, разместить данные по желанию и сохранить запись. Например, функция-обработчик для события `PlayerJoined`выглядит следующим образом:

```
export function handlePlayerJoined(event: PlayerJoined): void {
  // ID for the round:
  // round number
  let roundId = event.params.round.toString();
  // try to load Round from a previous player
  let round = Round.load(roundId);
  // if round doesn't exists, it's the first player in the round -> create round
  if (round == null) {
    round = new Round(roundId);
    round.timestampInit = event.block.timestamp;
  }
  round.index = event.params.round;
  round.prize = event.params.prizeAmount;

  round.save();

  // ID for the player:
  // issuer address
  let playerId = event.params.player.toHex();
  // try to load Player from previous rounds
  let player = Player.load(playerId);
  // if player doesn't exists, create it
  if (player == null) {
    player = new Player(playerId);
  }
  player.address = event.params.player;

  player.save();

  // ID for the ticket (round - player_address - ticket_index_round):
  // "round_number" + "-" + "player_address" + "-" + "ticket_index_per_round"
  let nextTicketIndex = event.params.ticketIndex.toString();
  let ticketId = roundId + "-" + playerId + "-" + nextTicketIndex;

  let ticket = new Ticket(ticketId);
  ticket.round = roundId;
  ticket.player = playerId;
  ticket.isGifted = event.params.isGifted;
  ticket.isWinner = false;

  ticket.save();  
}
```

## Размещение Subgraph

Если Вы собираетесь использовать API The Graph (размещенный сервис), Вам необходимо:

 - Создать учетную запись [Graph Explorer](https://thegraph.com/explorer/), для этого Вам понадобится учетная запись Github
 - Зайдите в личный кабинет для получения "access token"
 - Создайте свой Subgraph с помощью кнопки Add Subgraph на сайте Graph Explorer. Запишите название Subgraph

!!! примечание 
    Все шаги можно найти по [этой ссылке](https://thegraph.com/docs/deploy-a-subgraph).
 
Если Вы используете локальную ноду The Graph, Вы можете создать свой Subgraph, выполнив следующую команду:

```
npx graph create <username>/<subgraphName> --node <graph-node>
```

Где:

 - **username** — относится к имени пользователя, относящемуся к создаваемому Subgraph
 - **subgraphName** — относится к имени Subgraph
 - **graph-node** — ссылка на URL-адрес для управления сервисом. Обычно для локальной ноды The Graph используется адрес  `http://127.0.0.1:8020`

После этого Вы можете разместить свой Subgraph, выполнив следующую команду с теми же параметрами, что и раньше:

```
npx graph deploy <username>/<subgraphName> \
--ipfs <ipfs-url> \
--node <graph-node> \
--access-token <access-token>
```

Где:

 - **username** —  относится к имени пользователя, используемому при создании Subgraph
 - **subraphName** — относится к имени Subgraph, определенному при создании Subgraph
 - **ifps-url**  — относится к URL-адресу IFPS. Если Вы используете API The Graph, Вы можете использовать`https://api.thegraph.com/ipfs/`. For your local Graph Node, the default value is `http://localhost:5001`
 - **graph-node** — refers to the URL of the hosted service to use. If using The Graph API you can use `https://api.thegraph.com/deploy/`. Для Вашей локальной ноды The Graph значение по умолчанию - `http://localhost:8020`
 - **access-token** — относится к токену доступа для использования API The Graph. Если Вы используете локальную ноду The Graph, то этот параметр не нужен

Логи предыдущей команды должны выглядеть примерно так:

![Размещенный The Graph](/images/thegraph/thegraph-images1.png)

DApps теперь могут использовать конечные точки Subgraph для извлечения данных, проиндексированных протоколом The Graph.

