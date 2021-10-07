---
title: MetaMask
description: В этом руководстве Вы узнаете, как подключить MetaMask - кошелек Ethereum на основе браузера, к Moonbeam.
---

# Взаимодействие с Moonbeam при помощи MetaMask

![Вступительная диаграмма](/images/tokens/connect/metamask/metamask-banner.png)

## Вступление {: #introduction } 

Разработчики могут использовать функции совместимости Moonbeam с Ethereum для интеграции таких инструментов, как [MetaMask](https://metamask.io/), в свои DApps. Таким образом, они могут использовать встроенную библиотеку, которую предоставляет MetaMask, для взаимодействия с блокчейном.

В настоящее время MetaMask можно настроить для подключения к двум сетям: автономной ноде Moonbeam или к Moonbase Alpha TestNet.

Если у Вас уже установлен MetaMask, Вы можете легко подключить MetaMask к Moonbase Alpha TestNet:

<div class="button-wrapper">
    <a href="#" class="md-button connectMetaMask" value="moonbase">Connect MetaMask</a>
</div>

!!! примечание 
    Вы увидите всплывающее окно MetaMask с запросом разрешения на добавление Moonbase Alpha в качестве настраиваемой сети. Как только вы утвердите разрешения, MetaMask переключит Вашу текущую сеть на Moonbase Alpha.

Узнайте [как интегрировать кнопку Connect MetaMask](https://medium.com/moonbeam-network/integrate-metamask-into-a-dapp-ea7528c5a786) в Ваши DApps, чтобы пользователи могли подключаться к Moonbase Alpha простым нажатием кнопки.

## Подключение MetaMask к Moonbeam {: #connect-metamask-to-moonbeam } 

После того, как Вы установили [MetaMask](https://metamask.io/), Вы можете подключить его к Moonbeam, нажав на верхний правый логотип, открыв настройки.

![MetaMask3](/images/tokens/connect/metamask/metamask-6.png)

Затем перейдите на вкладку Networks и нажмите кнопку "Add Network".

![MetaMask4](/images/tokens/connect/metamask/metamask-7.png)

Здесь Вы можете настроить MetaMask для следующих сетей:

Автономная нода Moonbeam:

--8<-- 'text/metamask-local/development-node-details.md'

Moonbase Alpha TestNet:

--8<-- 'text/testnet/testnet-details.md'

## Пошаговые инструкции {: #step-by-step-tutorials } 

Если Вас интересует более подробное пошаговое руководство, Вы можете перейти к нашим подробным руководствам:

 - MetaMask для [автономной ноды Moonbeam](/getting-started/local-node/using-metamask/)
 - MetaMask для [Moonbase Alpha](/getting-started/moonbase/metamask/)
