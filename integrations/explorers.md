---
title: Обозреватель Блоков
description: Обзор доступных в настоящее время обозревателей блоков, которые можно использовать для навигации по уровням Substrate и Ethereum в Moonbeam TestNet.
---
# Обозреватель Блоков

![Обозреватель Блоков](/images/explorers/explorers-banner.png)

## Введение 

Обозреватель блоков можно рассматривать как поисковые системы для блокчейна. Они позволяют пользователям находить информацию о балансах, контрактах и транзакциях. Более продвинутые “поисковики” блоков даже предлагают возможности индексации, которые позволяют им предоставить полный набор информации, о токенах в сети ERC20. Они могут даже предлагать API службы для доступа к ним через внешние службы.

В настоящее время мы предлагаем два обозревателя блоков: один для Ethereum API и второй это Substrate API.

!!! note
    Если вы используете Brave Browser и ваш проводник не подключается к экземпляру Moonbeam, на который вы ссылаетесь, попробуйте отключить Brave Shield.

## Ethereum API

### Expedition (Dev Node - TestNet)

Используя эту [ссылку](https://moonbeam-explorer.netlify.app/) Вы можете найти версию [Expedition](https://github.com/etclabscore/expedition).

По умолчанию проводник подключен к сети Moonbase Alpha TestNet. Однако вы можете подключить его, выполнив следующие шаги:

 1. Нажмите на значок шестеренки в правом верхнем углу.
 2. Выберите пункт "Development", если у вас есть нода, работающая по следующему адресу `http://localhost:9933` (default RPC location of a Moonbeam node running with `--dev`flag). You can also switch back to "Moonbase Alpha"(по умолчанию RPC узел Moonbeam, запущен с опцией --devflag). Вы также можете вернуться в режим "Moonbase Alpha"
 3. In the case you want to connect to a specific RPC URL, select "Custom RPC" and enter the URL. For example, `http://localhost:9937`

![Expedition Explorer](/images/explorers/explorers-images-1.png)

### Blockscout (TestNet)

Blockscout provides an easy-to-use interface for users to inspect and confirm transactions on EVM blockchains, including Moonbeam. It allows you to search transactions, view accounts, and balances, and verify smart contracts. More information can be found in their [documentation site](https://docs.blockscout.com/).

As main features, Blockscout offers:

 - Open source development, meaning all code is open to the community to explore and improve. You can find the code [here](https://github.com/blockscout/blockscout)
 - Real-time transaction tracking
 - Smart contract interaction
 - ERC20 and ERC721 token support, listing all available token contract in a friendly interface
 - Full-featured API with GraphQL, where users can test API calls directly from a web interface

An instance of Blockscout running against the Moonbase Alpha TestNet can be found in [this link](https://moonbase-blockscout.testnet.moonbeam.network/).

![Blockscout Explorer](/images/explorers/explorers-images-2.png)

## Substrate API

### PolkadotJS (Dev Node - TestNet)

Polkadot JS Apps uses the WebSocket endpoint to interact with the Network. To connect it to a Moonbeam development node, you can follow the steps in [this tutorial](/getting-started/local-node/setting-up-a-node/#connecting-polkadot-js-apps-to-a-local-moonbeam-node). The default port for this is `9944`.

![Polkadot JS Local Node](/images/explorers/explorers-images-3.png)

To view and interact with Moonbase Alpha's substrate layer, go to [this URL](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Fwss.testnet.moonbeam.network#/explorer). This is the Polkadot JS Apps pointing to the TestNet. You can find more information in [this page](/integrations/wallets/polkadotjs/).

![Polkadot JS Moonbase Alpha](/images/explorers/explorers-images-4.png)

### Subscan

Subscan provides blockchain explorer capabilities for Substrate-based chains. It is capable of parsing standard or custom modules. For example, this is useful to display information regarding the Staking, Governance, and EVM pallets (or modules). Code is all open-source and can be found [here](https://github.com/itering/subscan-essentials).

An instance of Subscan running against the Moonbase Alpha TestNet can be found in [this link](https://moonbase.subscan.io/).

![Subscan Moonbase Alpha](/images/explorers/explorers-images-5.png)
