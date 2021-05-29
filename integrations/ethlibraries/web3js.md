---
title: Web3.js
description: Следуйте этому руководству, чтобы узнать, как использовать библиотеку JavaScript Ethereum Web3 для развертывания смарт-контрактов Solidity на Moonbeam.
---
# Библиотека JavaScript Web3.js

![Intro diagram](/images/integrations/integrations-web3js-banner.png)

## Вступление

[Web3.js](https://web3js.readthedocs.io/) это набор библиотек, которые позволяют разработчикам взаимодействовать с нодами Ethereum, используя протоколы HTTP, IPC или WebSocket с JavaScript. Moonbeam имеет API-интерфейс, подобный Ethereum, который полностью совместим с вызовами JSON RPC в стиле Ethereum. Поэтому разработчики могут использовать эту совместимость и использовать библиотеку web3.js для взаимодействия с нодой Moonbeam, как если бы они делали это в Ethereum.

## Настройтка Web3.js с помощью Moonbeam

Чтобы начать работу с библиотекой web3.js, нам сначала нужно установить ее, используя следующую команду:

```
npm install web3
```

После этого простейшая настройка для начала использования библиотеки и ее методов будет следующая:

```js
const Web3 = require('web3');

//Create web3 instance
const web3 = new Web3('RPC_URL');
```

В зависимости от того, к какой сети вы хотите подключиться, вы можете установить для `RPC_URL` следующие значения:

 -Нода Moonbeam: `http://127.0.0.1:9933`
 - Moonbase Alpha TestNet: `https://rpc.testnet.moonbeam.network`

## Пошаговые инструкции

сли вас интересует более подробное пошаговое руководство, перейдите к нашим конкретным руководствам по использованию web3.js на Moonbeam для [отправки транзакции](/getting-started/local-node/send-transaction/) или [развертывания контракта](/getting-started/local-node/deploy-contract/).

