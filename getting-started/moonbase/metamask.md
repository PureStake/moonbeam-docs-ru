---
title: Интеграция MetaMask
description: Узнайте, как использовать MetaMask с Moonbeam TestNet. Из этого туториала Вы узнаете, как подключить MetaMask, установленную по умолчанию, к Moonbase Alpha.
---

# Подключение MetaMask к Moonbase Alpha

## Вступление {: #introduction } 

This guide outlines the steps needed to connect MetaMask to Moonbase Alpha. In contrast to the MetaMask guide for a Moonbeam development node, this is much simpler because you don't need to connect to a local running Moonbeam node. Let's jump right into it.

Если у вас уже установлена MetaMask, вы можете легко подключить MetaMask к тестовой сети Moonbase Alpha:

<div class="button-wrapper">
    <a href="#" class="md-button connectMetaMask">Connect MetaMask</a>
</div>

!!! примечание :
    Появится всплывающее окно MetaMask с запросом разрешения на добавление Moonbase Alpha в качестве настраиваемой сети. Как только вы утвердите разрешения, MetaMask переключит вашу текущую сеть на Moonbase Alpha.

--8<-- 'text/common/create-metamask-wallet.md'

## Подключение к Moonbase Alpha {: #connecting-to-moonbase-alpha } 

После того как Вы закончите создание или импорт кошелька, Вы увидите основной интерфейс MetaMask. Здесь щелкните в правом верхнем углу логотипа и перейдите в Настройки(Settings).

![MetaMask3](/images/testnet/testnet-metamask3.png)

Перейдите на вкладку «Сети»(Networks) и нажмите кнопку «Добавить сеть»(Add Network).

![MetaMask4](/images/testnet/testnet-metamask4.png)

десь введите следующую информацию и нажмите Сохранить(Save):

  - Network Name: `Moonbase Alpha`
  - RPC URL: `{{ networks.moonbase.rpc_url }}`
  - ChainID: `{{ networks.moonbase.chain_id }}`
  - Symbol (Optional): `DEV`
  - Block Explorer: `{{ networks.moonbase.block_explorer }}`
  
![MetaMask5](/images/testnet/testnet-metamask5.png)

Вот и все! Вы успешно подключили MetaMask к Moonbeam TestNet, Moonbase.
