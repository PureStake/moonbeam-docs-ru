---
title: Remix IDE
description: Узнайте, как использовать один из самых популярных инструментов разработчика Ethereum, Remix IDE, для взаимодействия с Moonbeam.
---

# Взаимодействие с Moonbeam с помощью Remix

![Вступительная диаграмма](/images/integrations/integrations-remix-banner.png)

## Введение {: #introduction } 

Еще один инструмент, который разработчики могут использовать для взаимодействия с Moonbeam, — это [Remix IDE](https://remix.ethereum.org/), часто используемая среда разработки для смарт-контрактов на Ethereum. Remix IDE предоставляет веб-решение для быстрой компиляции и размещения кода на основе Solidity и Vyper либо на локальной виртуальной машине, или же, что более интересно, у внешнего провайдера Web3, такого как MetaMask. Комбинируя оба инструмента, можно очень быстро начать работу с Moonbeam.

## Размещение контракта на Moonbeam {: #deploying-a-contract-to-moonbeam } 

Чтобы продемонстрировать, как Вы можете использовать [Remix](https://remix.ethereum.org/) для размещения смарт-контрактов в Moonbeam, мы используем следующий базовый контракт:

```solidity
pragma solidity ^0.7.5;

contract SimpleContract{
    string public text;
    
    constructor(string memory _input) {
        text = _input;
    }
}
```

После компиляции, мы можем перейти на вкладку "Deploy & Run Transactions". Сначала нам нужно настроить нашу среду на "Injected Web3". Для этого используется провайдер, внедренный в MetaMask, что позволяет нам размещать контракты в сети, к которой он подключен — в данном случае Moonbase Alpha TestNet.

В этом примере мы будем размещать контракт из финансируемой учетной записи MetaMask. Вы можете использовать [кран TestNet](/getting-started/moonbase/faucet/) чтобы получить тестовые токены для размещений на Moonbase Alpha. Затем измените значение на `Test Contract` в качестве входных данных для нашей функции конструктора и нажмите deploy. Всплывающее окно MetaMask покажет информацию о транзакции, которую нам нужно будет подписать, нажав "confirm".


![Размещение контракта](/images/remix/integrations-remix-1.png)

Как только транзакция будет включена, контракт появится в разделе "Deployed Contracts" в Remix. Там мы можем взаимодействовать с функциями, доступными в нашем контракте.

![Взаимодействовать с контрактом](/images/remix/integrations-remix-2.png)

## Пошаговые инструкции {: #stepbystep-tutorials } 

Если Вас интересует более подробные пошаговые инструкции, перейдите к нашим руководствам по использованию [Remix на автономной ноде Moonbeam](/getting-started/local-node/using-remix/). Эти шаги также можно адаптировать для размещения на Moonbase Alpha TestNet [подключив к ней MetaMask](/getting-started/moonbase/metamask/).

