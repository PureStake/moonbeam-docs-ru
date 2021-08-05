---
title: Web3.py
description: Следуйте этому руководству, чтобы узнать, как использовать библиотеку Ethereum Web3 Python для развертывания смарт-контрактов Solidity на Moonbeam.
---
# Библиотека Python Web3.py

![Intro diagram](/images/integrations/integrations-web3py-banner.png)

## Вступление {: #introduction } 

[Web3.py](https://web3py.readthedocs.io/) это набор библиотек, которые позволяют разработчикам взаимодействовать с нодами Ethereum, используя протоколы HTTP, IPC или WebSocket с JavaScript. Moonbeam имеет API-интерфейс, подобный Ethereum, который полностью совместим с вызовами JSON RPC в стиле Ethereum. Поэтому разработчики могут использовать эту совместимость и использовать библиотеку web3.py для взаимодействия с нодой Moonbeam, как если бы они делали это в Ethereum.

## Настройтка Web3.py с помощью Moonbeam {: #setup-web3py-with-moonbeam } 

Чтобы начать работу с библиотекой web3.py, нам сначала нужно установить ее, используя следующую команду:

```
pip3 install web3
```

После этого простейшая настройка для начала использования библиотеки и ее методов будет следующая:

```py
from web3 import Web3

web3 = Web3(Web3.HTTPProvider('RPC_URL'))
```

В зависимости от того, к какой сети Вы хотите подключиться, Вы можете установить для `RPC_URL` следующие значения:

 - Нода Moonbeam: `http://127.0.0.1:9933`
 - Moonbase Alpha TestNet: `https://rpc.testnet.moonbeam.network`
 - Moonriver: `{{ networks.moonriver.rpc_url }}`
## Пошаговые инструкции {: #step-by-step-tutorials } 

Если вас интересует более подробное пошаговое руководство, перейдите к нашим конкретным руководствам по использованию web3.py на Moonbeam для [отправки транзакции](/getting-started/local-node/send-transaction/) или [развертывания контракта.](/getting-started/local-node/deploy-contract/).

