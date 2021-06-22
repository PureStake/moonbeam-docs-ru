---
title: Использование Truffle
description: С помощью Truffle можно невероятно легко разместить Solidity смарт-контракт на ноде Moonbeam. Узнайте как это сделать, в этом руководстве.
---

# Взаимодействие с Moonbeam с помощью Truffle

<style>.embed-container { position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; } .embed-container iframe, .embed-container object, .embed-container embed { position: absolute; top: 0; left: 0; width: 100%; height: 100%; }</style><div class='embed-container'><iframe src='https://www.youtube.com/embed/RD5MefSPNeo' frameborder='0' allowfullscreen></iframe></div>
<style>.caption { font-family: Open Sans, sans-serif; font-size: 0.9em; color: rgba(170, 170, 170, 1); font-style: italic; letter-spacing: 0px; position: relative;}</style><div class='caption'>
Вы можете найти весь необходимый код касающийся этого руководства в <a href="{{ config.site_url }}resources/code-snippets/">code snippets page</a></div>

## Введение

В этом руководстве рассматривается процесс размещения смарт-контракта на основе Solidity на ноде Moonbeam с использованием [Truffle](https://www.trufflesuite.com/), широко используемого инструмента разработки смарт-контрактов на Ethereum. Учитывая особенности совместимости Moonbeam с Ethereum, Truffle можно использовать напрямую с нодой Moonbeam.

!!! note
    Это руководство было создано с использованием тега {{ networks.development.build_tag}}, который основан на {{ networks.moonbase.version }} версии [Moonbase Alpha](https://github.com/PureStake/moonbeam/releases/tag/{{ networks.moonbase.version }}). Платформа Moonbeam и компоненты [Frontier](https://github.com/paritytech/frontier), которые используются для совместимости Ethereum на основе Substrate, все еще находятся в стадии активной разработки.
    --8<-- 'text/common/assumes-mac-or-ubuntu-env.md'

Для этого руководства Вам понадобится нода Moonbeam, работающая в режиме `--dev`. Это можно сделать, выполнив шаги, подробно описанные [здесь](/getting-started/local-node/setting-up-a-node/) или используя [Moonbeam Truffle plugin](/integrations/trufflebox/#the-moonbeam-truffle-plugin), который мы будем использовать в примерах этого руководства.

## Проверка предварительных условий

--8<-- 'text/common/install-nodejs.md'


Кроме того, Вы можете установить Truffle глобально, запустив:

```
npm install -g truffle
```

На дату публикации этого руководства использовались версии 15.12.10, 7.6.3 и 5.2.4 соответственно.

!!! note
    Для следующих примеров Вам не нужно устанавливать Truffle глобально, т.к. он включен как зависимость от Moonbeam Truffle box. Если хотите, Вы можете запустить `npx truffle` или `./node_modules/.bin/truffle` вместо `truffle`.

## Начало работы с Truffle

Чтобы облегчить процесс знакомства с Truffle, мы выпустили [Moonbeam Truffle box](https://moonbeam.network/announcements/moonbeam-truffle-box-available-solidity-developers/). Это обеспечивает стандартную настройку для ускорения процесса размещения контрактов на Moonbeam. Чтобы узнать больше о Moonbeam Truffle box, Вы можете перейти по [этой ссылке](/integrations/trufflebox/).

Чтобы загрузить Moonbeam Truffle box, следуйте [этим инструкциям](/integrations/trufflebox/#downloading-and-setting-up-the-truffle-box). Оказавшись внутри директории, давайте взглянем на файл `truffle-config.js` (для целей этого руководства некоторая информация была удалена):

```js
const HDWalletProvider = require('@truffle/hdwallet-provider');
// Moonbeam Development Node Private Key
const privateKeyDev =
   '99B3C12287537E38C90A9219D4CB074A89A16E9CDB20BF85728EBD97C343E342';
//...
module.exports = {
   networks: {
      dev: {
         provider: () => {
            ...
            return new HDWalletProvider(privateKeyDev, 'http://localhost:9933/')
         },
         network_id: 1281,
      },
      //...
   },
   plugins: ['moonbeam-truffle-plugin']
};
```

Обратите внимание, что мы используем `HD-Wallet-Provider` от Truffle в качестве иерархического детерминированного кошелька. Кроме того, мы определили `dev` сеть, которая указывает на URL-адрес поставщика ноды, и в нее включен закрытый ключ учетной записи разработчика, в которой хранятся все средства в ноде.

## Запуск ноды

Чтобы настроить ноду Moonbeam, Вы можете использовать [это руководство](/getting-started/local-node/setting-up-a-node/). В общей сложности, этот процесс занимает около 40 минут, также Вам необходимо установить Substrate и все его зависимости. Плагин Moonbeam Truffle позволяет гораздо быстрее начать работу с нодой, и единственное требование — установить Docker (на момент написания использовалась версия Docker 19.03.6).

Чтобы запустить ноду Moonbeam в Вашей локальной среде, нам нужно сначала загрузить соответствующий образ Docker:

```
truffle run moonbeam install
```

![Загрукза Docker](/images/truffle/using-truffle-1.png)

После загрузки мы можем приступить к запуску ноды с помощью следующей команды:

```
truffle run moonbeam start
```

Вы увидите сообщение о том, что нода запущена, а затем обе доступные конечные точки.

![Запущенная локальная нода Moonbeam](/images/truffle/using-truffle-2.png)

Когда Вы закончите использовать ноду Moonbeam, Вы можете запустить следующие команды, чтобы остановить ее и удалить образ Docker:

```
truffle run moonbeam stop && \
truffle run moonbeam remove
```

![Остановленная локальная нода Moonbeam](/images/truffle/using-truffle-3.png)

## Файл контракта

Также вместе с Truffle Box включен контракт с токеном ERC-20:

```solidity
pragma solidity ^0.7.5;

// Import OpenZeppelin Contract
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

// This ERC-20 contract mints the specified amount of tokens to the contract creator.
contract MyToken is ERC20 {
    constructor(uint256 initialSupply) ERC20("MyToken", "MYTOK")
    {
        _mint(msg.sender, initialSupply);
    }
}
```

Это простой контракт ERC-20, основанный на шаблоне OpenZepplin ERC-20. Он создает MyToken с символом MYTOK и стандартными 18 десятичными знаками. Кроме того он создает первичное предложение токенов для эмитента контракта.

Если мы посмотрим на скрипт миграции контракта Truffle в разделе `migrations/2_deploy_contracts.js`, мы увидим следующее:

```javascript
var MyToken = artifacts.require('MyToken');

module.exports = function (deployer) {
   deployer.deploy(MyToken, '8000000000000000000000000');
};
```

"8000000000000000000000000" это количество токенов, которые первоначально будут созданы контрактом, т.е. 8 миллионов MYTOK с 18 знаками после запятой.

## Размещение контракта на Moonbeam с использованием Truffle

Прежде чем мы сможем разместить наши контракты, мы должны их скомпилировать. (Мы говорим «контракты», потому что обычные размещения Truffle включают контракт `Migrations.sol`) Вы можете сделать это с помощью следующей команды:

```
truffle compile
```

В случае успеха, Вы должны увидеть следующее:

![Успешная компилляция Truffle](/images/truffle/using-truffle-4.png)

Теперь мы готовы разместить скомпилированные контракты. Это можно сделать с помощью следующей команды:

```
truffle migrate --network dev
```

В случае успеха, Вы увидите действия по размещению, включая адрес размещенного контракта:

![Успешное размещение контракта](/images/truffle/using-truffle-5.png)

После того, как Вы выполните руководство по использованию [MetaMask](/getting-started/local-node/using-metamask/) и [Remix](/getting-started/local-node/using-remix/), Вы сможете получить адрес размещенного контракта и загрузить его в MetaMask или Remix.

