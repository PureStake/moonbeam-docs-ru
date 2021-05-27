---
title: Moonbeam Truffle Box
description: Начните использовать Moonbeam Truffle Box для быстрого, предварительно настроенного способа развертывания Ваших смарт-контрактов Solidity на Moonbeam с помощью Truffle.
---
# Moonbeam Truffle Box

![Intro diagram](/images/integrations/integrations-trufflebox-banner.png)

##Вступление
В рамках постоянной работы по оказанию помощи разработчикам, которые хотят начать работу над Moonbeam, мы выпустили [Moonbeam Truffle box](https://moonbeam.network/announcements/moonbeam-truffle-box-available-solidity-developers/). С его помощью разработчики найдут шаблонную настройку, чтобы быстро приступить к размещение смарт-контрактов на базе Moonbeam. В Moonbeam Truffle Box мы также включили плагин Moonbeam Truffle, который вводит некоторые команды для запуска автономного узла в Вашей локальной среде в виде образа Docker. Это исключает процесс настройки локального узла (который может занять до 40 минут при создании бинарных файлов) является быстрым и простым решением для начала разработки в Вашей локальной среде.

Это руководство проведет вас через процесс настройки Truffle Box с использованием плагина Moonbeam Truffle и размещение контрактов как на автономном узле Moonbeam, так и на Moonbase Alpha с использованием базовой конфигурацией Truffle Box.

!!! Обратите внимание
    Это руководство основано на установке Ubuntu 18.04. На момент написания использовались версии Node.js и npm 15.2.1 и 7.0.8 соответственно. Требуются версии Node.js выше 10.23.0. Также мы заметили ошибку при установке пакетов с npm версии 7.0.15. Вы можете понизить версию npm, запустив команду `npm install -g npm@version`, установив желаемую версию.
   
## Проверка предварительных условий

--8<-- 'text/common/install-nodejs.md'

Для этого урока нам нужно установить Node.js (мы установим на v15.x) и менеджер пакетов npm. Вы можете сделать это, запустив в своем терминале:


```
npm install -g truffle
```

На момент написания этого руководства использовалась версия 5.1.51.

## Загрузка и настройка Truffle Box

Чтобы начать работу с Moonbeam Truffle box, если Truffle установлен глобально, Вы можете выполнить:

```
mkdir moonbeam-truffle-box && cd moonbeam-truffle-box
truffle unbox PureStake/moonbeam-truffle-box
```

![Unbox Moonbeam Truffle box](/images/trufflebox/trufflebox-07.png)

Тем не менее, в коробке также есть Truffle ​на случай, если Вы не хотите, чтобы он устанавливался глобально. В таком случае Вы можете напрямую клонировать следующий репозиторий:

```
git clone https://github.com/PureStake/moonbeam-truffle-box
cd moonbeam-truffle-box
``` 

Следующим шагом с файлами в Вашей локальной системе является установка всех зависимостей, запустив:

```
npm install
```

!!! Обратите внимание
    Мы заметили ошибку при установке пакетов с npm версии 7.0.15. Вы можете понизить `npm install -g npm@version` версию npm, запустив и установив желаемую версию. Например, 7.0.8 или 6.14.9.

На этом завершены все предварительные условия, необходимые для использования Moonbeam Truffle box.

## Основные функции

Коробка предварительно сконфигурирована для работы с двумя сетями: `dev` (для автономного узла) и `moonbase` (Moonbeam TestNet). Также включены, в качестве примера, контракт токена ERC20 и простой тестовый скрипт. По умолчанию установлен компилятор Solidity `^0.7.0`, но при необходимости его можно изменить. Если у Вас есть опыт работы с Truffle, эта установка покажется Вам знакомой.

```js
const HDWalletProvider = require('@truffle/hdwallet-provider');
// Moonbeam Development Node Private Key
const privateKeyDev =
   '99B3C12287537E38C90A9219D4CB074A89A16E9CDB20BF85728EBD97C343E342';
// Moonbase Alpha Private Key --> Please change this to your own Private Key with funds
// NOTE: Do not store your private key in plaintext files
//       this is only for demostration purposes only
const privateKeyMoonbase =
   'YOUR_PRIVATE_KEY_HERE_ONLY_FOR_DEMOSTRATION_PURPOSES';

module.exports = {
   networks: {
      // Standalode Network
      dev: {
         provider: () => {
            //...
            return new HDWalletProvider(
               privateKeyDev,
               'http://localhost:9933/'
            );
         },
         network_id: 1281,
      },
      // Moonbase Alpha TestNet
      moonbase: {
         provider: () => {
            //...
            return new HDWalletProvider(
               privateKeyMoonbase,
               'https://rpc.testnet.moonbeam.network'
            );
         },
         network_id: 1287,
      },
   },
   // Solidity 0.7.0 Compiler
   compilers: {
      solc: {
         version: '^0.7.0',
      },
   },
   // Moonbeam Truffle Plugin
   plugins: ['moonbeam-truffle-plugin'],
};
```

Файл  `truffle-config.js` также включает в себя закрытый ключ генезиса аккаунта для автономного узла. Адрес, связанный с этим ключом, содержит все токены в этой среде разработки. Для размещения в Moonbase Alpha TestNet Вам необходимо предоставить закрытый ключ адреса, на котором хранятся средства. Для этого Вы можете создать учетную запись в MetaMask, пополнить ее с помощью [TestNet faucet](https://docs.moonbeam.network/getting-started/testnet/faucet/) крана и экспортировать ее закрытый ключ.

Как и при использовании Truffle в любой сети Ethereum, Вы можете запускать обычные команды для компиляции, тестирования и размещения смарт-контрактов в Moonbeam. Например, Вы можете попробовать следующие команды, используя включенный контракт токена ERC20:

```
truffle compile # compiles the contract
truffle test #run the tests included in the test folder
truffle migrate --network network_name  #deploys to the specified network
```

В зависимости от сети, в которой Вы хотите разместить контракты, Вам необходимо заменить network_name на dev (для автоматного узла) или moonbase (для TestNet).

!!! Обратите внимание
   Если у Вас нет глобального Truffle, Вы можете использовать `npx truffle` или `./node_modules/.bin/truffle` вместо `truffle`.

## Плагин Moonbeam Truffle

Чтобы настроить автономный узел Moonbeam, Вы можете следовать [этому руководству](/getting-started/local-node/setting-up-a-node/). В общей сложности процесс занимает около 40 минут, и Вам необходимо установить Substrate и все его зависимости. Плагин Moonbeam Truffle позволяет гораздо быстрее приступить к работе с автономным узлом, и единственное требование — установить Docker (на момент написания использовалась версия Docker 19.03.6). Для получения дополнительной информации об установке Docker посетите [эту страницу](https://docs.docker.com/get-docker/). Чтобы загрузить образ Docker, выполните следующую строку:

```
truffle run moonbeam install
``` 

![Install Moonbeam Truffle box](/images/trufflebox/trufflebox-01.png)

 
Затем у Вас есть набор команд, доступных для управления узлом, включенным в образ Docker:
 
```
truffle run moonbeam start
truffle run moonbeam status
truffle run moonbeam pause
truffle run moonbeam unpause
truffle run moonbeam stop
truffle run moonbeam remove
```

Каждая из показанных выше команд выполняет следующие действия:

-  Start: запускает автономный узел Moonbeam. Это обеспечивает две конечные точки RPC: — HTTP: 
      - HTTP: `http://127.0.0.1:9933` 
      - WS: `ws://127.0.0.1:9944`
-  Status: сообщает пользователю, запущен ли автономный узел Moonbeam.
-  Pause: приостанавливает работу автономного узла, если он работает.
-  Unpause: возобновляет работу автономного узла, если он приостановлен.
-  Stop: останавливает автономный узел, если он работает. Это также удаляет контейнер Docker.
-  Remove: удаляет образ Docker purestake / moonbase.

Вы можете увидеть результат выполнения этих команд на следующем изображении:

![Install Moonbeam Truffle box](/images/trufflebox/trufflebox-02.png)

Если Вы знакомы с Docker, Вы можете пропустить команды плагина и напрямую взаимодействовать с образом Docker.

## Тестирование Moonbeam Truffle Box

На коробке указаны минимальные требования, которые помогут Вам начать работу. Давайте сначала скомпилируем контракты, запустив:

```
truffle compile
``` 
![Compile Contracts](/images/trufflebox/trufflebox-03.png)

Помните, что если Truffle установлен глобально, Вы можете пропустить часть ./node_modules/.bin/ в командах. С скомпилированным контрактом мы можем запустить базовый тест, включенный в комплект (обратите внимание, что для этих тестов используется Ganache, а не автономный узел Moonbeam):

```
truffle test
```

![Test Contract Moonbeam Truffle box](/images/trufflebox/trufflebox-04.png)

После выполнения команды установки плагина, которая загружает образ Docker автономного узла Moonbeam, давайте запустим локальный узел и разместим контракт токена в нашей локальной среде:

```
truffle run moonbeam start
truffle migrate --network dev
```

![Deploy on Dev Moonbeam Truffle box](/images/trufflebox/trufflebox-05.png)

Наконец, мы можем разместить наш токен-контракт на Moonbase Alpha, но сначала убедитесь, что Вы установили закрытый ключ в файле truffle-config.js. Как только закрытый ключ установлен, мы можем выполнить команду migrate, указывающую на TestNet.

```
truffle migrate --network moonbase
```

![Deploy on Moonbase Moonbeam Truffle box](/images/trufflebox/trufflebox-06.png)

Вот и все, Вы использовали Moonbeam Truffle box для размещения простого контракта с токеном ERC20 как в Вашем автономном узле Moonbeam, так и в Moonbase Alpha.
 
