---
title: Waffle & Mars
description: Узнайте, как использовать Waffle и Mars для написания, компиляции, тестирования и развертывания смарт-контрактов Ethereum на Moonbeam.
---

# Использование Waffle & Mars на Moonbeam

![Waffle and Mars on Moonbeam](/images/builders/interact/waffle-mars/waffle-mars-banner.png)

## Вступление {: #introduction } 

[Waffle](https://getwaffle.io/) это библиотека для компиляции и тестирования смарт-контрактов, и [Mars](https://github.com/EthWorks/Mars) это менеджер по развертыванию. Waffle и Mars можно использовать вместе для написания, компилирования, тестирования, и создания Ethereum смарт-контрактов. Поскольку Moonbeam совместим с Ethereum, Waffle и Mars можно использовать для развертывания смарт-контрактов на сервере разработки Moonbeam или в Moonbase Alpha TestNet.

Для работы с Waffle требуются минимальные зависимости, он имеет простой синтаксис, который легко изучить и расширить, а также обеспечивает быстрое выполнение при компиляции и тестировании смарт-контрактов. Кроме того, он совместим с [TypeScript](https://www.typescriptlang.org/) и использует [Chai matchers](https://ethereum-waffle.readthedocs.io/en/latest/matchers.html) делая тесты удобными для чтения и записи.

Mars предоставляет простой, совместимый с TypeScript фреймворк для создания расширенных сценариев развертывания и синхронизации с изменениями состояния. Mars использует подход инфраструктура-как-код, позволяя разработчикам определять, как необходимо разворачивать свои смарт-контракты, а затем использовать эти спецификации для автоматической обработки изменений состояния и развертывания.

В этом руководстве Вы создадите проект TypeScript, напишите, скомпилируете и протестируете смарт-контракт с помощью Waffle, а затем развернете его в Moonbase Alpha TestNet опять же с помощью Mars.

## Проверка требований {: #checking-prerequisites } 

--8<-- 'text/common/install-nodejs.md'

На момент написания этого руководства использовались версии 15.12.0 и 7.6.3 соответственно.

Waffle и Mars можно использовать на лоакльной ноде разработки Moonbeam, но в рамках данного руководства Вы будете выполнять все работы на Moonbase Alpha. Следовательно, для разработки Вам понадобится счет. Вы можете выбрать [создание аккаунта через MetaMask](/getting-started/moonbase/metamask/#creating-a-wallet) или [создание аккаунта через PolkadotJS Apps](/integrations/wallets/polkadotjs/#creating-or-importing-an-h160-account). 

После того, как Вы создали учетную запись, Вам необходимо будет экспортировать закрытый ключ, который будет использоваться в данном руководстве. Прежде чем двигаться дальше, убедитесь, что на Вашем счету есть средства и, при необходимости, получите `DEV` токены здесь [faucet](/getting-started/moonbase/faucet/).

## Создайте проект TypeScript с помощью Waffle & Mars {: #create-a-typescript-project-with-waffle-mars } 

Для начала Вы создадите проект TypeScript, а также установите и настроите несколько зависимостей.

1. Создайте каталог проекта и перейдите в него:
```
mkdir waffle-mars && cd waffle-mars
```

2. Выполните инициализацию проекта. Это создаст файл `package.json` в текущем аккаунте:
```
npm init -y
```

3. Установите следующие зависимости:
```
npm install ethereum-waffle ethereum-mars ethers \
@openzeppelin/contracts typescript ts-node chai \
@types/chai mocha @types/mocha
```
    - [Waffle](https://github.com/EthWorks/Waffle) - for writing, compiling, and testing smart contracts
    - [Mars](https://github.com/EthWorks/Mars) - for deploying smart contracts to Moonbeam
    - [Ethers](https://github.com/ethers-io/ethers.js/) - for interacting with Moonbeam's Ethereum API
    - [OpenZeppelin Contracts](https://github.com/OpenZeppelin/openzeppelin-contracts) - the contract you'll be creating will use OpenZeppelin's ERC20 base implementation
    - [TypeScript](https://github.com/microsoft/TypeScript) - the project will be a TypeScript project
    - [TS Node](https://github.com/TypeStrong/ts-node) - for executing the deployment script you'll create later in this guide
    - [Chai](https://github.com/chaijs/chai) - an assertion library used alongside Waffle for writing tests
    - [@types/chai](https://github.com/DefinitelyTyped/DefinitelyTyped/tree/HEAD/types/chai) - contains the type definitions for chai
    - [Mocha](https://github.com/mochajs/mocha) - a testing framework for writing tests alongside Waffle
    - [@types/mocha](https://github.com/DefinitelyTyped/DefinitelyTyped/tree/HEAD/types/mocha) - contains the type definitions for mocha

4. Создайте файл [конфигурации TypeScript](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html):
```
touch tsconfig.json
```

5. Затем добавьте базовую конфигурацию TypeScript:
```
{
  "compilerOptions": {
    "strict": true,
    "target": "ES2019",
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "esModuleInterop": true,
    "module": "CommonJS",
    "composite": true,
    "sourceMap": true,
    "declaration": true,
    "noEmit": true
  }
}
```

После чего у Вас должен быть базовый проект TypeScript с необходимыми зависимостями, чтобы начать сборку Waffle и Mars.

## Добавление контракта {: #add-a-contract } 

В этом руководстве Вы создадите контракт ERC-20, с указанным количеством монет для создателя контракта. Он основан на шаблоне Open Zeppelin ERC-20.

1. Создайте каталог для хранения Ваших контрактов и отдельный файл для смарт-контракта:
```
mkdir contracts && cd contracts && touch MyToken.sol
```

2. Добавьте в `MyToken.sol` следующий контракт:
```
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract MyToken is ERC20 {
    constructor() ERC20("MyToken", "MYTOK") {}

    function initialize(uint initialSupply) public {
      _mint(msg.sender, initialSupply);
    }
}
```

В этом контракте Вы создаете токен ERC20 под названием `MyToken` с символом MYTOK, который позволяет Вам, как создателю контракта, производить столько MYTOK, сколько Вам понадобится.

## Используйте Waffle для компиляции и тестирования {: #use-waffle-to-compile-and-test } 

### Компиляция с помощью Waffle {: #compile-with-waffle } 

Теперь, когда Вы написали смарт-контракт, следующим шагом будет его компиляция с помощью Waffle. Прежде чем перейти к процессу компиляции контракта, Вам необходимо настроить Waffle.

1. Вернитесь в корневой каталог проекта и создайте файл `waffle.json` для настройки Waffle:
```
cd .. && touch waffle.json
```

2. Отредактируйте файл `waffle.json`, указав конфигурацию компилятора, каталог, содержащий Ваши контракты, и многое другое. В этом примере мы будем использовать `solcjs` и версию Solidity, которую Вы использовали для контракта, то есть` 0.8.0`:
```json
{
  "compilerType": "solcjs", // Specifies compiler to use
  "compilerVersion": "0.8.0", // Specifies version of the compiler
  "compilerOptions": {
    "optimizer": { // Optional optimizer settings
      "enabled": true, // Enable optimizer
      "runs": 20000 // Optimize how many times you want to run the code
    }
  },
  "sourceDirectory": "./contracts", // Path to directory containing smart contracts
  "outputDirectory": "./build", // Path to directory where Waffle saves compiler output
  "typechainEnabled": true // Enable typed artifact generation
}
```

3. Добавьте скрипт для запуска Waffle в `package.json`:
```json
"scripts": {
  "build": "waffle"
},
```

Это все, что Вам нужно сделать для настройки Waffle, теперь Вы можете скомпилировать контракт `MyToken` с помощью скрипта `build`:

```
npm run build
```

![Waffle compiler output](/images/builders/interact/waffle-mars/waffle-mars-1.png)

После компиляции Ваших контрактов, Waffle сохраняет вывод JSON в каталоге `build`. Контракт в этом руководстве основан на шаблоне ERC-20 Open Zeppelin, поэтому соответствующие файлы JSON ERC-20 также появятся в каталоге `build`.

### Тест с Waffle {: #test-with-waffle } 

Перед тем, как развернуть свой контракт и отправить его в "дикую природу", Вы должны сначала протестировать его. Waffle предоставляет расширенную среду тестирования и множество инструментов, которые помогут Вам с тестированием. 

Вы будете запускать тесты для Moonbase Alpha TestNet, и для подключения к ней Вам потребуется соответствующий URL-адрес RPC: `https://rpc.testnet.moonbeam.network`. Поскольку Вы будете запускать тесты в TestNet, запуск всех тестов может занять пару минут. Если вам нужен более эффективный опыт тестирования, Вы можете [развернуть среду разработки Moonbeam](/getting-started/local-node/setting-up-a-node/) используя [`instant seal`](/getting-started/local-node/setting-up-a-node/#node-options). Запуск локальной среды разработки Moonbeam с функцией `instant seal` аналогичен быстрому и итеративному опыту, который Вы получили бы с [Ganache](https://www.trufflesuite.com/ganache).

1. Создайте каталог, в котором будут храниться Ваши тесты, и файл для тестирования Вашего контракта `MyToken`:
```
mkdir test && cd test && touch MyToken.test.ts
```

2. Откройте файл `MyToken.test.ts` и настройте свой тестовый файл, чтобы использовать плагин Waffle Solidity и использовать настраиваемый поставщик JSON-RPC Ethers для подключения к Moonbase Alpha:
```typescript
import { use, expect } from 'chai';
import { Provider } from '@ethersproject/providers';
import { solidity } from 'ethereum-waffle';
import { ethers, Wallet } from 'ethers';
import { MyToken, MyTokenFactory } from '../build/types';

// Tell Chai to use Waffle's Solidity plugin
use(solidity);

describe ('MyToken', () => {
  // Use custom provider to connect to Moonbase Alpha
  let provider: Provider = new ethers.providers.JsonRpcProvider('https://rpc.testnet.moonbeam.network');
  let wallet: Wallet;
  let walletTo: Wallet;
  let token: MyToken;

  beforeEach(async () => {
    // Logic for setting up the wallet and deploying MyToken will go here
  });

  // Tests will go here
})
```

3. Перед запуском каждого теста Вы захотите создать кошелек и подключить его к провайдеру, используйте кошельки для развертывания экземпляра контракта `MyToken`, а затем вызовите функцию `initialize` один раз с первоначальным количеством 10 монет:
```typescript
  beforeEach(async () => {
    const PRIVATE_KEY = '<insert-your-private-key-here>'
    // Create a wallet instance using your private key & connect it to the provider
    wallet = new Wallet(PRIVATE_KEY).connect(provider);

    // Create a random account to transfer tokens to & connect it to the provider
    walletTo = Wallet.createRandom().connect(provider);

    // Use your wallet to deploy the MyToken contract
    token = await new MyTokenFactory(wallet).deploy();

    // Mint 10 tokens to the contract owner, which is you
    let contractTransaction = await token.initialize(10);

    // Wait until the transaction is confirmed before running tests
    await contractTransaction.wait();
  });
```

4. Теперь Вы можете создать свой первый тест. Первый тест проверит Ваш первоначальный баланс, чтобы убедиться, что Вы получили необходимый запас в 10 токенов. Однако, чтобы следовать хорошей практике тестирования, сначала напишите неудачный тест:
```typescript
it('Mints the correct initial balance', async () => {
  expect(await token.balanceOf(wallet.address)).to.equal(1); // This should fail
});
```

5. Прежде чем Вы сможете запустить свой первый тест, Вам нужно вернуться в корневую директорию и добавить файл конфигурации Mocha `.mocharc.json`:
```
cd .. && touch .mocharc.json
```

6. Теперь отредактируйте файл `.mocharc.json`, чтобы настроить Mocha:
```json
{
  "require": "ts-node/register/transpile-only", // Use ts-node to transpile the code for tests
  "timeout": 600000, // Set timeout to 10 minutes
  "extension": "test.ts" // Specify extension for test files
}
```

7. Вам также нужно будет добавить скрипт в `package.json` для запуска Ваших тестов:
```json
"scripts": {
  "build": "waffle",
  "test": "mocha",
},
```

8. Наконец все готово к запуску тестов, для этого просто используйте только что созданный и исполняемый скрипт `test`:
```
npm run test
```
Обратите внимание, что обработка может занять несколько минут, потому что тесты выполняются на Moonbase Alpha, но если все прошло, как и ожидалось, у Вас должен быть один неудачный тест.

9. Затем Вы можете вернуться и отредактировать тест, чтобы проверить наличие 10 токенов:
```typescript
it('Mints the correct initial balance', async () => {
  expect(await token.balanceOf(wallet.address)).to.equal(10); // This should pass
});
```
10. Если Вы снова запустите тесты, Вы должны увидеть один пройденный тест:
```
npm run test
```

11. Вы проверили способность создавать монеты, затем Вы проверите возможность передачи созданных монет. Если Вы хотите с нуля написать неудачный тест, то окончательный вариант должен выглядеть так:
```typescript
it('Should transfer the correct amount of tokens to the destination account', async () => {
  // Send the destination wallet 7 tokens
  await (await token.transfer(walletTo.address, 7)).wait();

  // Expect the destination wallet to have received the 7 tokens
  expect(await token.balanceOf(walletTo.address)).to.equal(7);
});
```

Поздравляем, теперь у Вас должно быть два пройденных теста! В целом Ваш тестовый файл должен выглядеть так:
```typescript
import { use, expect } from 'chai';
import { Provider } from '@ethersproject/providers';
import { solidity } from 'ethereum-waffle';
import { ethers, Wallet } from 'ethers';
import { MyToken, MyTokenFactory } from '../build/types';

use(solidity);

describe ('MyToken', () => {
  let provider: Provider = new ethers.providers.JsonRpcProvider('https://rpc.testnet.moonbeam.network');
  let wallet: Wallet;
  let walletTo: Wallet;
  let token: MyToken;

  beforeEach(async () => {
    const PRIVATE_KEY = '<insert-your-private-key-here>'
    wallet = new Wallet(PRIVATE_KEY).connect(provider);
    walletTo = Wallet.createRandom().connect(provider);
    token = await new MyTokenFactory(wallet).deploy();
    let contractTransaction = await token.initialize(10);
    await contractTransaction.wait();
  });

  it('Mints the correct initial balance', async () => {
    expect(await token.balanceOf(wallet.address)).to.equal(10);
  });

  it('Should transfer the correct amount of tokens to the destination account', async () => {
    await (await token.transfer(walletTo.address, 7)).wait();
    expect(await token.balanceOf(walletTo.address)).to.equal(7);
  });
})
```

Если Вы хотите написать больше тестов самостоятельно, Вы можете рассмотреть возможность тестирования переводов со счетов без каких-либо средств или переводов со счетов без достаточного количества средств.

## Используйте Mars для развертывания на Moonbase Alpha {: #use-mars-to-deploy-to-moonbase-alpha } 

После компиляции контрактов и перед развертыванием Вам нужно будет сгенерировать артефакты контрактов для Mars. Mars использует артефакты контрактов для проверки типов в процессе развертывания. Затем Вам нужно будет создать сценарий развертывания и развернуть смарт-контракт `MyToken`.

Помните, что Вы будете выполнять развертывание на Moonbase Alpha, и Вам нужно будет использовать конфигурацию TestNet:

--8<-- 'text/testnet/testnet-details.md'

Развертывание будет разбито на три части: [генерация артифактов](#generate-artifacts), [создание сценария развертывания](#create-a-deployment-script), и [и развёртывание с Mars](#deploy-with-mars). 

### Создание артефактов {: #generate-artifacts } 

Для Mars необходимо сгенерировать артефакты, чтобы включить проверку типов в сценариях развертывания.

1. Обновите существующий скрипт для запуска Waffle в `package.json`, чтобы включить Mars:
```json
"scripts": {
  "build": "waffle && mars",
  "test": "mocha",
},
```

2. Сгенерируйте артефакты и создайте файл `artifacts.ts`, необходимый для развертывания:
```
npm run build
```
![Waffle and Mars compiler output](/images/builders/interact/waffle-mars/waffle-mars-2.png)

Если Вы откроете каталог `build`, Вы должны увидеть файл `artifacts.ts`, содержащий данные об артефактах, необходимые для развертывания. Чтобы продолжить процесс развертывания, Вам необходимо написать сценарий развертывания. Сценарий развертывания будет использоваться, чтобы сообщить Mars, какой контракт разворачивать, в какой сети и какую учетную запись следует использовать для запуска развертывания.

### Создание сценария развертывания {: #create-a-deployment-script } 

Теперь Вам нужно настроить развертывание контракта `MyToken` в Moonbase Alpha TestNet.

На этом этапе Вы создадите сценарий развертывания, который определит, как следует развертывать контракт. Mars предлагает функцию `deploy`, в которую Вы можете передавать параметры, такие как закрытый ключ учетной записи для создания контракта, сеть для развертывания и многое другое. Внутри функции `deploy` определяются создаваемые контракты. Mars имеет функцию `contract`, которая принимает значения `name`, `artifact` и` constructorArgs`. Эта функция будет использоваться для развертывания контракта `MyToken` с первоначальным количеством 10 MYTOK.

1. Создайте каталог `src` для хранения Ваших сценариев развертывания и создайте сценарий для развертывания контракта` MyToken`:
```
mkdir src && cd src && touch deploy.ts
```

2. В `deploy.ts` используйте функцию Mars `deploy`, чтобы создать сценарий для развертывания на Moonbase Alpha с использованием закрытого ключа Вашей учетной записи:
```javascript
import { deploy } from 'ethereum-mars';

const privateKey = "<insert-your-private-key-here>";
deploy({network: 'https://rpc.testnet.moonbeam.network', privateKey},(deployer) => {
  // Deployment logic will go here
});
```

3. Настройте функцию `deploy` для развертывания контракта` MyToken`, созданного на предыдущем этапе:
```javascript
import { deploy, contract } from 'ethereum-mars';
import { MyToken } from '../build/artifacts';

const privateKey = "<insert-your-private-key-here>";
deploy({network: 'https://rpc.testnet.moonbeam.network', privateKey}, () => {
  contract('myToken', MyToken, [10]);
});
```

4. Добавьте сценарий развертывания к объекту `scripts` в` package.json`:
```json
  "scripts": {
    "build": "waffle && mars",
    "test": "mocha",
    "deploy": "ts-node src/deploy.ts"
  },
```

Пока что Вы должны были создать сценарий развертывания в `deploy.ts`, который будет развертывать контракт `MyToken` в Moonbase Alpha, и добавить возможность простого вызова сценария и развертывания контракта.

### Deploy с использованием Mars {: #deploy-with-mars } 

Вы настроили развертывание, теперь пришло время запуска на Moonbase Alpha.

1. Разверните контракт, используя только что созданный скрипт:
```
npm run deploy
```

2. В вашем терминале Mars Вам будет предложено нажать `ENTER`, чтобы отправить транзакцию:
![Mars confirm deployment](/images/builders/interact/waffle-mars/waffle-mars-3.png)

В случае успеха Вы должны увидеть подробную информацию о своей транзакции, включая ее хэш, блок, в который она была включена, и его адрес.

![Mars deployment output](/images/builders/interact/waffle-mars/waffle-mars-4.png)

Поздравлем! Вы развернули контракт с Moonbase Alpha, используя Waffle и Mars!

## Пример проекта {: #example-project } 

Если Вы хотите увидеть завершенный пример проекта Waffle и Mars на Moonbeam, ознакомьтесь с [moonbeam-waffle-mars-example](https://github.com/EthWorks/moonbeam-waffle-mars-example) созданным командой разработчиков Waffle, Mars и EthWorks.
