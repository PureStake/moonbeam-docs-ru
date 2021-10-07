---
title: Ethers.js
description: Следуйте этому руководству, чтобы узнать, как использовать библиотеку Ethereum EtherJS для развертывания смарт-контрактов Solidity на Moonbeam.
---
# Библиотека JavaScript Ethers.js

![Intro diagram](/images/builders/tools/eth-libraries/ethersjs-banner.png)

## Вступление {: #introduction } 

Библиотека [ethers.js](https://docs.ethers.io/) предоставляет набор инструментов для взаимодействия с нодами Ethereum с помощью JavaScript, аналогичных web3.js. Moonbeam имеет API-интерфейс, подобный Ethereum, который полностью совместим с вызовами JSON RPC в стиле Ethereum. Поэтому разработчики могут использовать эту совместимость и использовать библиотеку ethers.js для взаимодействия с нодой Moonbeam, как если бы они делали это в Ethereum. Вы можете узнать больше о ethers.js в этом [посте блога](https://medium.com/l4-media/announcing-ethers-js-a-web3-alternative-6f134fdd06f3).

## Установите Ethers.js с помощью Moonbeam {: #setup-ethersjs-with-moonbeam } 

Чтобы начать работу с библиотекой ethers.js, установите ее с помощью следующей команды:

```
npm install ethers
```

После этого простейшая настройка для начала использования библиотеки и ее методов следующая:

```js
const ethers = require('ethers');

// Variables definition
const privKey = '0xPRIVKEY';

// Define Provider
const provider = new ethers.providers.StaticJsonRpcProvider('RPC_URL', {
    chainId: ChainId,
    name: 'NETWORK_NAME'
});

// Create Wallet
let wallet = new ethers.Wallet(privKey, provider);
```

Внутри `provider` и `wallet` доступны разные методы. В зависимости от того, к какой сети Вы хотите подключиться, Вы можете установить для `RPC_URL` следующие значения:

Нода Moonbeam: 
 - RPC_URL: `http://127.0.0.1:9933`"
 - ChainId: `1281`
 - NETWORK_NAME: `moonbeam-development`
 
Moonbase Alpha TestNet: 
 - RPC_URL: `https://rpc.testnet.moonbeam.network`
 - ChainId: `1287`
 - NETWORK_NAME: `moonbase-alpha`

## Пошаговые инструкции {: #step-by-step-tutorials } 

Если Вас интересует более подробное пошаговое руководство, Вы можете перейти к нашим конкретным руководствам по использованию ethers.js на Moonbeam для [отправки транзакции](/getting-started/local-node/send-transaction/) или [развертывания контракта](/getting-started/local-node/deploy-contract/).
