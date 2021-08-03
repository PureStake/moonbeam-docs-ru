---
title: Hardhat
description: Используйте Hardhat для компиляции, развертывания и отладки смарт-контрактов Ethereum на Moonbeam.
---

# Разработка на Moonbeam с помощью Hardhat

![Hardhat Create Project](/images/hardhat/hardhat-banner.png)

## Вступление {: #introduction } 

Hardhat — это среда разработки Ethereum, которая помогает разработчикам управлять рутинными задачами, относящимися к созданию смарт-контрактов и dApps, и автоматизировать их. Hardhat может напрямую взаимодействовать с Ethereum API Moonbeam, поэтому его также можно использовать для размещения смарт-контрактов в Moonbeam.

В этом руководстве будет рассказано о том, как использовать Hardhat для компиляции, размещения и отладки смарт-контрактов Ethereum в Moonbase Alpha TestNet.

## Проверка необходимых условий {: #checking-prerequisites } 

--8<-- 'text/common/install-nodejs.md'

На момент написания этого руководства использовались версии 15.7.0 и 7.4.3 соответственно.

Кроме того, Вам также понадобится следующее:

 - Установить MetaMask и [подключиться к Moonbase](/getting-started/moonbase/metamask/)
 - Иметь учетную запись с денежными средствами, которые Вы можете получить от [Mission Control](/getting-started/moonbase/faucet/)

Как только все требования будут выполнены, Вы будете готовы производить блоки с помощью Hardhat.

## Запуск Hardhat проекта {: #starting-a-hardhat-project } 

Чтобы начать новый проект, создайте для него каталог:

```
mkdir hardhat && cd hardhat
```

Затем инициализируйте проект, запустив:

```
npm init -y
```

Вы заметите недавно созданный `package.json`, который будет продолжать расти по мере установки зависимостей проекта.

Чтобы начать работу с Hardhat, мы установим его в наш недавно созданный каталог проекта:

```
npm install hardhat
```

После установки запустите:

```
npx hardhat
```

Это создаст конфигурационный файл Hardhat (hardhat.config.js) в каталоге нашего проекта.

!!! примечание  внимание
    `npx` используется для запуска исполняемых файлов, установленных локально в Вашем проекте. Хотя Hardhat может быть установлен глобально, мы рекомендуем устанавливать его локально, чтобы Вы могли контролировать версию в каждом проекте.

После выполнения команды выберите `Создать пустой hardhat.config.js` файл:

![Hardhat Create Project](/images/hardhat/hardhat-images-1.png)

## Файл контракта {: #the-contract-file } 

Мы будем хранить наш контракт в каталоге `contracts`. Создайте его с помощью команды:

```
mkdir contracts && cd contracts
```

Смарт-контракт, который мы разместим в качестве примера, будет называться Box — он позволит хранить значение, которое позже можно будет вернуть.

Мы сохраним этот файл как `contracts/Box.sol`:

```solidity
// contracts/Box.sol
pragma solidity ^0.8.1;

contract Box {
    uint256 private value;

    // Emitted when the stored value changes
    event ValueChanged(uint256 newValue);

    // Stores a new value in the contract
    function store(uint256 newValue) public {
        value = newValue;
        emit ValueChanged(newValue);
    }

    // Reads the last stored value
    function retrieve() public view returns (uint256) {
        return value;
    }
}
```

## Конфигурационный файл Hardhat {: #hardhat-configuration-file } 

Давайте изменим наш конфигурационный файл Hardhat, чтобы можно было скомпилировать и разместить этот контракт на Moonbase Alpha.

Создайте учетную запись MetaMask (если вы еще не сделали этого), [подключитесь к Moonbase Alpha](/getting-started/moonbase/metamask/), и пополните ее через [Mission Control](/getting-started/moonbase/faucet/). Мы будем использовать приватный ключ учетной записи, созданной для размещения контракта.

Начнем с установки [ethers плагина](https://hardhat.org/plugins/nomiclabs-hardhat-ethers.html), который добавит библиотеку [ethers.js][/integrations/ethers/] позволяющую Вам легко взаимодействовать с блокчейном. Мы можем установить плагин `ethers` запустив:

```
npm install @nomiclabs/hardhat-ethers ethers
```

Затем мы импортируем приватный ключ, полученный из MetaMask, и сохраняем его в `.json` файле.

!!! примечание  внимание
    Пожалуйста, храните свои приватные ключи с помощью менеджера паролей или аналогичного сервиса. Никогда не сохраняйте и не коммитьте свои приватные ключи внутри ваших репозиториев.

В `module.exports`, нам нужно добавить версию Solidity (`0.8.1` в соответствии с нашим файлом контракта) и данные сети:

 - Network name: `moonbase`
 - URL: `{{ networks.moonbase.rpc_url }}`
 - ChainID: `{{ networks.moonbase.chain_id }}`

Если Вы хотите выполнить размещение на локальной автономной ноде Moonbeam, Вы можете использовать следующие данные сети:

 - Network name: `dev`
 - URL: `{{ networks.development.rpc_url }}`
 - ChainID: `{{ networks.development.chain_id }}`

Конфигурационный файл Hardhat должен выглядеть следующим образом:

```js
// ethers plugin required to interact with the contract
require('@nomiclabs/hardhat-ethers');

// private key from the pre-funded Moonbase Alpha testing account
const { privateKey } = require('./secrets.json');

module.exports = {
  // latest Solidity version
  solidity: "0.8.1",

  networks: {
    // Moonbase Alpha network specification
    moonbase: {
      url: `{{ networks.moonbase.rpc_url }}`,
      chainId: {{ networks.moonbase.chain_id }},
      accounts: [privateKey]
    }
  }
};
```

Далее создадим файл `secrets.json`, в котором мы будем хранить приватный ключ, упомянутый ранее. Не забудьте добавить к файл Вашему проекту `.gitignore`, и не делитесь приватными ключами. Файл `secrets.json` должен содержать запись `privateKey` например:

```js
{
    "privateKey": "YOUR-PRIVATE-KEY-HERE"
}
```

Поздравляем! Теперь, мы готовы к развертыванию!

## Компилирование Solidity {: #compiling-solidity } 

Наш контракт, `Box.sol`, использует Solidity 0.8.1. Убедитесь, что конфигурационный файл Hardhat правильно настроен в соответствии с текущей версией solidity. В таком случае мы можем скомпилировать контракт, запустив:

```
npx hardhat compile
```

![Hardhat Contract Compile](/images/hardhat/hardhat-images-2.png)

После компиляции создается каталог `artifacts` он содержит байт-код и метаданные контракта, которые являются файлами `.json`. Хорошей идеей будет добавить этот каталог в Ваш `.gitignore`.

## Размещение контракта {: #deploying-the-contract } 

Для того чтобы разместить смарт-контракт Box, нам нужно будет написать простой `deployment script`. Во-первых, давайте создадим новый каталог (`scripts`). . Внутрь созданного каталога добавьте новый файл `deploy.js`.

```
mkdir scripts && cd scripts
touch deploy.js
```

Далее нам нужно написать сценарий размещения с использованием `ethers`. Поскольку мы будем запускать его с помощью Hardhat, нам не нужно импортировать какие-либо библиотеки. Скрипт представляет собой упрощенную версию того, что используется [в этом руководстве](/getting-started/local-node/deploy-contract/#deploying-the-contract).

Мы начинаем с создания локального экземпляра контракта с помощью метода `getContractFactory()`. Затем используем метод `deploy()` который используется в данном экземпляре, чтобы инициировать смарт-контракт. Наконец, мы размещаем его, используя `deployed()`. После размещения мы можем получить адрес контракта в экземпляр box.

```js
// scripts/deploy.js
async function main() {
   // We get the contract to deploy
   const Box = await ethers.getContractFactory('Box');
   console.log('Deploying Box...');

   // Instantiating a new Box smart contract
   const box = await Box.deploy();

   // Waiting for the deployment to resolve
   await box.deployed();
   console.log('Box deployed to:', box.address);
}

main()
   .then(() => process.exit(0))
   .catch((error) => {
      console.error(error);
      process.exit(1);
   });
```
Воспользовавшись командой `run`, мы можем разместить контракт `Box` на `Moonbase Alpha`:

```
  npx hardhat run --network moonbase scripts/deploy.js
```

!!! примечание  внимание
    Чтобы выполнить размещение на автономной ноде Moonbeam, замените `moonbase` на `dev` в команде `run` moonbase.

Через несколько секунд контракт будет размещен, и Вы увидите адрес в терминале.

![Hardhat Contract Deploy](/images/hardhat/hardhat-images-3.png)

Поздравляем, контракт заработал! Сохраните адрес, так как мы будем использовать его для взаимодействия с этим экземпляром контракта на следующем шаге.

## Interacting with the Contract {: #interacting-with-the-contract } 

Давайте используем Hardhat для взаимодействия с нашим недавно размещенным контрактом на Moonbase Alpha. Для этого включите консоль `hardhat console`, запустив:

```
npx hardhat console --network moonbase
```

!!! примечание  внимание
    Чтобы выполнить размещение на автономной ноде Moonbeam, замените `moonbase` на `dev` в `console`.

Затем добавьте следующие строки кода — по одной строке. Сначала мы снова создаем локальный экземпляр контракта `Box.sol`. Не беспокойтесь, если Вы получили `undefined` после выполнения каждой строки:

```js
const Box = await ethers.getContractFactory('Box');
```

Далее давайте подключим этот экземпляр к существующему, передав адрес, полученный при размещении контракта:

```js
const box = await Box.attach('0x425668350bD782D80D457d5F9bc7782A24B8c2ef');
```
После присоединения к контракту мы готовы взаимодействовать с ним. Пока консоль все еще находится в данной сессии, давайте вызовем метод `store` и сохраним простое значение:

```
await box.store(5)
```

Транзакция будет подписана Вашей учетной записью Moonbase и передана в сеть. Результат должен выглядеть примерно так:

![Transaction output](/images/hardhat/hardhat-images-4.png)

Обратите внимание на свой адрес с пометкой  `from`, адрес контракта и передаваемые `данные`. Теперь давайте получим это значение, выполнив:

```
(await box.retrieve()).toNumber()
```

Мы должны увидеть `5` или значение, которое сохранили изначально.

Поздравляем, Вы прошли урок по Hardhat! 🤯 🎉

Для получения дополнительной информации о Hardhat, плагинах hardhat и других интересных функциях, пожалуйста, посетите сайт [hardhat.org](https://hardhat.org/).
