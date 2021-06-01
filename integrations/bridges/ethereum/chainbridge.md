---
title: ChainBridge
description: Как использовать ChainBridge для соединения активов между Ethereum и Moonbeam с помощью смарт-контрактов
---
# Мост ChainBridge между Ethereum и Moonbeam

![ChainBridge Moonbeam banner](/images/chainbridge/chainbridge-banner.png)

## Введение

Мост позволяет двум экономически независимым и технологически различным цепям взаимодействовать друг с другом. Мосты могут варьироваться от централизованных и надежных до децентрализованных и с минимальным доверием. Одним из доступных решений на данный момент является [ChainBridge](https://github.com/ChainSafe/ChainBridge#installation), модульный разно-направленный мост на blockchain, разработанный командой [ChainSafe](https://chainsafe.io/). Интеграция ChainBridge теперь доступна и на Moonbeam. Данная интеграция соединяет нашу альфа тестовую сеть Moonbeam — Moonbase и Rinkeby/Kovan тестовые сети эфира.

Это руководство разбито на два основных раздела. В первой части мы объясним общий рабочий процесс моста. Во второй части рассмотрим несколько примеров использования моста для передачи ресурсов ERC-20 и ERC-721 между Moonbase Alpha и Rinkeby / Kovan. 

 - [Как работает мост](/integrations/bridges/ethereum/chainbridge/#how-the-bridge-works)
    - [Общие определения](/integrations/bridges/ethereum/chainbridge/#general-definitions)
 - [Использование моста в Moonbase Alpha](/integrations/bridges/ethereum/chainbridge/#try-it-on-moonbase-alpha)
    - [Перенести токены ERC-20](/integrations/bridges/ethereum/chainbridge/#erc-20-token-transfer)
    - [Перенести токены ERC-721](/integrations/bridges/ethereum/chainbridge/#erc-721-token-transfer)
    - [Универсальный обработчик](/integrations/bridges/ethereum/chainbridge/#generic-handler)
    
## Как работает мост

ChainBridge по своей сути является протоколом передачи сообщений. События в исходном чейне используются для отправки сообщения, которое направляется в цепочку назначения. В данном процессе есть 3 основные роли:

 - Слушатель: извлекает события из первоначального чейна и создает сообщение.
 - Роутер: передает сообщение от слушателя до писателя.
 - Писатель: интерпретирует сообщения и отправляет транзакцию на чейн-получатель.
 
В настоящее время ChainBridge полагается на доверенные ретрансляторы для выполнения этих ролей. Тем не менее, он имеет механизм, который предотвращает злоупотреблением полномочиями и ненадлежащее использование средств любым ретранслятором. На высоком уровне ретрансляторы создают предложения в целевой цепочке, которые отправляются на утверждение другим ретрансляторам. Утверждающее голосование также происходит в целевой цепочке, и каждое предложение выполняется только после того, как оно достигает определенного порога голосования.

По обе стороны моста есть набор смарт-контрактов, каждый из которых выполняет определенную функцию:

 - **Контракт Моста** — пользователи и ретрансляторы взаимодействуют с этим контрактом. Он делегирует вызовы контрактам обработчика для депозитов, запускает транзакцию в исходной цепочке и для выполнения предложений в целевой цепочке.
 - **Контракты обработчика** — проверяет параметры, предоставленные пользователем, создавая запись депозита / исполнения.
 - **Целевой контракт** — как следует из названия, это контракт, с которым мы собираемся взаимодействовать с каждой стороны моста.

### Общий рабочий процесс


Общий рабочий процесс следующий (от чейн A к чейн B):
 
  - Пользователь инициирует транзакцию с помощью функции _deposit()_ в мостовом контракте чейна A. Здесь пользователю необходимо ввести целевчй чейн, идентификатор ресурса и _calldata_ (определения после диаграммы). После нескольких проверок вызывается функция _deposit()_  контракта-обработчика, которая выполняет соответствующий вызов целевого контракта.
  - После того, как функция целевого контракта в чейне A выполнена, мостовым контрактом генерируется событие _Deposit_  которое содержит необходимые данные для выполнения в чейне B. Это называется предложением. Каждое предложение может иметь пять статусов (неактивно, активно, прошло, выполнено и отменено).
  - Ретрансляторы всегда слушают с обеих сторон чейна. Как только ретранслятор принимает событие, он инициирует голосование по предложению, которое происходит в мостовом контракте в чейне B. Это устанавливает состояние предложения с неактивного на активное.
  - Ретрансляторы должны проголосовать за предложение. Каждый раз, когда ретранслятор голосует, мостовой контракт генерирует событие, которое обновляет его статус. После достижения порога статус меняется с активного на пройденный. Затем ретранслятор выполняет предложение в чейне B через мостовой контракт.
  - После нескольких проверок мост выполняет предложение в целевом контракте через контракт обработчика в чейне B. Запускается другое событие, которое обновляет статус предложения с «передано» на «выполнено».

Этот рабочий процесс представлен на следующей диаграмме:

![ChainBridge Moonbeam diagram](/images/chainbridge/chainbridge-diagram.png)

Два целевых контракта на каждой стороне моста связаны путем выполнения серии регистраций в соответствующем контракте обработчика через мостовой контракт. Эти регистрации в настоящее время могут быть выполнены только администратором мостового контракта.

### Общие определения

Здесь мы собрали список концепций, применимых к реализации ChainBridge (от цепочки A до цепочки B):

 - **ChainBridge Chain ID** — не путать с идентификатором чейна сети. Это уникальный сетевой идентификатор, используемый протоколом для каждого чейна. Он может отличаться от фактического идентификатора чейна самой сети. Например, для Moonbase Alpha и Rinkeby мы установили идентификатор чейна ChainBridge на 43 и 4 соответственно (Кован был установлен на 42).
 - **Resource ID** — это 32-байтовое слово, предназначенное для однозначной идентификации актива в кросс-чейн среде. Обратите внимание, что младший байт зарезервирован для chainId, поэтому у нас будет 31 байт всего для представления актива чейна в нашем мосте. Например, это может быть выражение tokenX в чейне A эквивалентно tokenY в чейне B
 - **Calldata** — это параметр, необходимый для обработчика, который включает информацию, необходимую для выполнения предложения в чейне B. Точная сериализация определяется для каждого обработчика. Вы можете найти больше информации [здесь](https://chainbridge.chainsafe.io/chains/ethereum/#erc20-erc721-handlers)

## Протестируйте на Moonbase Alpha

Мы настроили ретранслятор с реализацией ChainBridge, который подключен к нашей тестовой сети Moonbase Alpha TestNet, а также к Rinkeby и Kovan TestNets Ethereum.

ChainSafe имеет [предварительно запрограммированные обработчики](https://chainbridge.chainsafe.io/chains/ethereum/#handler-contracts) специфичные для интерфейсов ERC-20 и ERC-721, и эти обработчики используются для передачи токенов ERC-20 и ERC-721 между чейнами. Более подробную информацию можно найти здесь. В общих чертах, это просто сужение общей схемы, которую мы описали ранее, поэтому обработчик работает только с конкретными функциями токена, такими как _lock/burn_, и _mint/unlock_. 

В этом разделе будут рассмотрены два различных примера использования моста для перемещения токенов ERC-20 и ERC-721 между цепочками. Для взаимодействия с Moonbase Alpha и Rinkeby/Kovan вам понадобится следующая информация:

```
# Kovan/Rinkeby - Moonbase Alpha bridge contract address:
    {{ networks.moonbase.chainbridge.bridge_address }}

 # Kovan/Rinkeby - Moonbase Alpha ERC-20 handler contract:
    {{ networks.moonbase.chainbridge.ERC20_handler }}
   
# Kovan/Rinkeby - Moonbase Alpha ERC-721 handler contract:
    {{ networks.moonbase.chainbridge.ERC721_handler }}
```

!!! Примечание
    Мостовой контракт, контракт обработчика ERC-20 и адрес контракта обработчика ERC-721, перечисленные выше, применимы как для Кована, так и для Ринкеби.

### ERC-20 Token Transfer

Токены ERC-20, которые нужно переместить через мост, должны быть зарегистрированы ретрансляторами в контрактах обработчиков. Поэтому для тестирования моста мы развернули токен ERC-20 (ERC20S), где любой пользователь может чеканить 5 токенов:

```
# Kovan/Rinkeby - Moonbase Alpha custom ERC-20 sample token:
    {{ networks.moonbase.chainbridge.ERC20S }}
```

Аналогичным образом, прямое взаимодействие с контрактом Bridge и вызов функции `deposit()`с правильными параметрами может быть пугающим. Чтобы упростить процесс использования моста, мы создали модифицированный контракт моста, который создает необходимые входные данные для функции `deposit()`:

```
# Kovan/Rinkeby - Moonbase Alpha custom bridge contract:
    {{ networks.moonbase.chainbridge.bridge_address }}
```


Проще говоря, в модифицированном контракте, используемом для инициирования передачи, для этого примера заранее определены _chainID_ и _resourceID_ . Следовательно, он создает объект _calldata_ из ввода пользователя, который является только адресом получателя и значением, которое нужно отправить.

Общий рабочий процесс для этого примера можно увидеть на этой диаграмме:

![ChainBridge ERC20 workflow](/images/chainbridge/chainbridge-erc20.png)

Чтобы попробовать мост с этим образцом токена ERC-20, мы должны выполнить следующие шаги (независимо от направления передачи):
 
 - Токены в исходном чейне (это подтверждает контракт обработчика источника в качестве спонсора на указанную сумму Токенов)
 - Используйте модифицированный мостовой контракт в исходном чейне для отправки токенов
 - Подождите, пока процесс завершится
 - Утвердите контракт обработчика целевого чейна в качестве спонсора для отправки токенов обратно
 - Используйте модифицированный мостовой контракт в целевом чейне для отправки токенов

!!! Примечание
    Помните, что токены будут переданы только в том случае, если в контракте обработчика будет достаточно разрешений на использование токенов от имени владельца. Если процесс не удался, проверьте разрешение.

Давайте отправим несколько токенов ERC20S с **Moonbase Alpha** на **Kovan**. Если вы хотите попробовать это с Rinkeby, шаги и адреса такие же. Для этого мы будем использовать [Remix](/integrations/remix/). Во-первых, мы можем использовать следующий интерфейс для взаимодействия с этим контрактом и создания токенов:

```solidity
pragma solidity ^0.8.1;

/**
    Interface for the Custom ERC20 Token contract for ChainBridge implementation
    Kovan/Rinkeby - Moonbase Alpha ERC-20 Address : 
        {{ networks.moonbase.chainbridge.ERC20S }}
*/

interface ICustomERC20 {

    // Mint 5 ERC20S Tokens
    function mintTokens() external;

    // Get Token Name
    function name() external view returns (string memory);
    
    /** 
        Increase allowance to Handler
        Kovan/Rinkeby - Moonbase Alpha ERC-20 Handler:
           {{ networks.moonbase.chainbridge.ERC20_handler}}
    */
    function increaseAllowance(address spender, uint256 addedValue) external returns (bool);
    
    // Get allowance
    function allowance(address owner, address spender) external view returns (uint256);
}
```

Обратите внимание, что функция контракта токена ERC-20 также была изменена для утверждения соответствующего контракта обработчика в качестве спонсора при выпуске токенов.

После добавления пользовательского контракта ERC20 в Remix и его компиляции, следующие шаги - чеканка токенов ERC20S:

1. Перейдите к **Deploy & Run Transactions** страница Remix
2. Выберите Injected Web3 из **Environment** 
3. Загрузите адрес контракта пользовательского токена ERC-20 и нажмите **At Address**
4. Вызовите функцию `mintTokens()` и подпишите транзакцию.
5. После подтверждения транзакции вы должны получить 5 токенов ERC20S. Вы можете проверить свой баланс, добавив токен в [MetaMask](/integrations/wallets/metamask/).

![ChainBridge ERC20 mint Tokens](/images/chainbridge/chainbridge-image1.png)

Когда у нас есть токены, мы можем приступить к их отправке через мост в целевую цепочку. В этом случае помните, что мы делаем это из **Moonbase Alpha** в **Kovan**. Есть единый интерфейс, который позволит вам передавать токены ERC20S и ERC721M. В этом примере вы будете использовать `sendERC20SToken()` функция для инициирования передачи ваших отчеканенных токенов ERC20S:

```solidity
pragma solidity 0.8.1;

/**
    Simple Interface to interact with bridge to transfer the ERC20S and ERC721M tokens
    Kovan/Rinkeby - Moonbase Alpha Bridge Address: 
        {{ networks.moonbase.chainbridge.bridge_address }}
 */

interface IBridge {

    /**
     * Calls the `deposit` function of the Chainbridge Bridge contract for the custom ERC20 (ERC20Sample) 
     * by building the requested bytes object from: the recipient, the specified amount and the destination
     * chainId.
     * @notice Use the destination `eth_chainId`.
     */
    function sendERC20SToken(uint256 destinationChainID, address recipient, uint256 amount) external;
    
    /**
     * Calls the `deposit` function for the custom ERC721 (ERC721Moon) that is only mintable in the
     * MOON side of the bridge. It builds the bytes object requested by the method from: the recipient,
     * the specified token ID and the destination chainId.
     * @notice Use the destination `eth_chainId`.
     */
    function sendERC721MoonToken(uint256 destinationChainID, address recipient, uint256 tokenId) external;
}
```

После добавления контракта Bridge в Remix и его компиляции для отправки токенов ERC20s через мост вам необходимо:

1. Загрузите адрес мостового контракта и нажмите **At Address**
2. Чтобы вызвать функцию `sendERC20SToken()` введите идентификатор цепочки назначения (в этом примере мы используем Kovan: `42`)
3. Введите адрес получателя на другой стороне моста
4. Добавьте сумму к переводу в WEI
5. Нажмите **transact** а затем появится MetaMask с просьбой подписать транзакцию. 

После подтверждения транзакции процесс может занять около 3 минут, после чего вы должны были получить токены в Kovan!

![ChainBridge ERC20 send Tokens](/images/chainbridge/chainbridge-image2.png)

Вы можете проверить свой баланс, добавив токен в [MetaMask](/integrations/wallets/metamask/) и подключение к целевой сети - в нашем случае Kovan.

![ChainBridge ERC20 balance](/images/chainbridge/chainbridge-image3.png)

Помните, что вы также можете чеканить токены ERC20S в Коване и отправить их в Moonbase Alpha. Чтобы утвердить спонсора или увеличить его размер, вы можете использовать `increaseAllowance()` функция предоставленного интерфейса. Чтобы проверить допустимость контракта обработчика в контракте токена ERC20, вы можете использовать `allowance()` функция интерфейса.

!!! Примечание:
    Токены будут переданы только в том случае, если в контракте с обработчиком будет достаточно разрешений для траты токенов от имени владельца. Если процесс не удался, проверьте припуск.

### Трансфер токена ERC-721

Как и в нашем предыдущем примере, контракты токенов ERC-721 должны быть зарегистрированы ретрансляторами, чтобы обеспечить передачу через мост. Поэтому мы настроили контракт токена ERC-721, чтобы любой пользователь мог создать токен для тестирования моста. Однако, поскольку каждый токен не является взаимозаменяемым и, следовательно, уникальным, функция доступна только в контракте токенов исходной цепочки, но не в целевом контракте. Как следствие, вам понадобится пара адресов контрактов ERC-721 для токенов в Rinkeby / Kovan и передачи их в Moonbase Alpha и еще одну пару для противоположного действия. Следующая диаграмма объясняет рабочий процесс для этого примера, где важно подчеркнуть, что идентификатор токена и метаданные сохраняются.

![ChainBridge ERC721 workflow](/images/chainbridge/chainbridge-erc721.png)

Чтобы произвести токены в Rinkeby / Kovan (ERC721E) и отправлять их туда и обратно на Moonbase Alpha, Вам понадобится следующий адрес:

```
# Kovan/Rinkeby - Moonbase Alpha ERC-721 Moon tokens (ERC721M),
# Mint function in Moonbase Alpha: 
    {{ networks.moonbase.chainbridge.ERC721M }}
```

Вместо взаимодействия с контрактом моста и вызова функции `deposit()`, мы изменили контракт моста, чтобы упростить процесс использования самого моста (тот же адрес, что и в предыдущем примере):

```
# Kovan/Rinkeby - Moonbase Alpha custom bridge contract:
    {{ networks.moonbase.chainbridge.bridge_address }}
```

Проще говоря, в модифицированном мостовом контракте, используемом для инициирования передачи, для этого примера заранее определены  _chainID_ и _resourceID_ . Следовательно, он создает объект _calldata_  из ввода пользователя, который является только адресом получателя и идентификатором отправляемого токена.

Давайте отправим токен ERC720E от **Kovan** на **Moonbase Alpha**. Для этого воспользуемся  [Remix](/integrations/remix/). Следующий интерфейс можно использовать для взаимодействия с исходными контрактами ERC721 и произведением токенов. Функцию `tokenOfOwnerByIndex()` также можно использовать для проверки идентификаторов токенов, принадлежащих определенному адресу, передавая адрес и индекс для запроса (каждый идентификатор токена сохраняется как элемент массива, связанный с адресом):

```solidity
pragma solidity ^0.8.1;

/**
    Interface for the Custom ERC721 Token contract for ChainBridge implementation:
    Kovan/Rinkeby - Moonbase Alpha:
        ERC721Moon: {{ networks.moonbase.chainbridge.ERC721M }}

    ERC721Moon tokens are only mintable on Moonbase Alpha
*/

interface ICustomERC721 {
    
    // Mint 1 ERC-721 Token
    function mintToken() external returns (uint256);
    
    // Query tokens owned by Owner
    function tokenOfOwnerByIndex(address _owner, uint256 _index) external view returns (uint256);

    // Get Token Name
    function name() external view returns (string memory);
    
    // Get Token URI
    function tokenURI(uint256 tokenId) external view returns (string memory);
    
    /**
        Set Approval for Handler 
        Kovan/Rinkeby - Moonbase Alpha ERC-721 Handler:
           {{ networks.moonbase.chainbridge.ERC721_handler }}
    */
    function approve(address to, uint256 tokenId) external;
    
    // Check the address approved for a specific token ID
    function getApproved(uint256 tokenId) external view returns (address);
}
```

Обратите внимание, что функция контракта токена ERC-721 также была изменена для утверждения соответствующего контракта обработчика в качестве спонсора при произведении токенов.

После добавления контракта в Remix и его компиляции, теперь мы хотим создать токен ERC721M:

1. Перейдите к **Deploy & Run Transactions** страницы на Remix
2. Выберите Injected Web3 с **Environment** 
3. Загрузите настраиваемый адрес контракта токена ERC721M и нажмите **At Address**
4. Вызовите функцию `mintTokens()` и подпишите транзакцию.
5. После подтверждения транзакции вы должны получить токен ERC721M. Вы можете проверить свой баланс, добавив токен в [MetaMask](/integrations/wallets/metamask/).

![ChainBridge ERC721 mint Tokens](/images/chainbridge/chainbridge-image4.png) 

Следующий интерфейс позволяет вам использовать функцию `sendERC721MoonToken()` для инициирования передачи токенов, изначально отчеканенных в Moonbase Alpha (ERC721M).

```solidity
pragma solidity 0.8.1;

/**
    Simple Interface to interact with bridge to transfer the ERC20S and ERC721M tokens
    Kovan/Rinkeby - Moonbase Alpha Bridge Address: 
        {{ networks.moonbase.chainbridge.bridge_address }}
 */

interface IBridge {

    /**
     * Calls the `deposit` function of the Chainbridge Bridge contract for the custom ERC20 (ERC20Sample) 
     * by building the requested bytes object from: the recipient, the specified amount and the destination
     * chainId.
     * @notice Use the destination `eth_chainId`.
     */
    function sendERC20SToken(uint256 destinationChainID, address recipient, uint256 amount) external;
    
    /**
     * Calls the `deposit` function for the custom ERC721 (ERC721Moon) that is only mintable in the
     * MOON side of the bridge. It builds the bytes object requested by the method from: the recipient,
     * the specified token ID and the destination chainId.
     * @notice Use the destination `eth_chainId`.
     */
    function sendERC721MoonToken(uint256 destinationChainID, address recipient, uint256 tokenId) external;
}
```

Теперь вы можете приступить к отправке токена ERC721M через мост в целевую цепочку. В этом случае помните, что мы сделаем это от Moonbase Alpha до Кована. Чтобы передать токен ERC721M через мост:

1. Загрузите адрес мостового контракта и нажмите **At Address**
2. Вызовите функцию `sendERC721MoonToken()` чтобы инициировать передачу токенов ERC721M, изначально отчеканенных в Moonbase Alpha, путем предоставления идентификатора целевой цепочки (в этом примере мы используем Kovan: `42`)
3. Введите адрес получателя на другой стороне моста
4. Добавьте идентификатор токена для переноса
5. Нажмите **transact** а затем появится MetaMask с просьбой подписать транзакцию.

После подтверждения транзакции процесс может занять около 3 минут, после чего вы должны получить тот же идентификатор токена в Kovan!

![ChainBridge ERC721 send Token](/images/chainbridge/chainbridge-image5.png)

Вы можете проверить свой баланс, добавив токен в [MetaMask](/integrations/wallets/metamask/) и подключив его к целевой сети, в нашем случае Kovan.

![ChainBridge ERC721 balance](/images/chainbridge/chainbridge-image6.png)

Помните, что токены ERC721M можно чеканить только в Moonbase Alpha, и тогда они будут доступны для отправки туда и обратно Ковану или Ринкеби. Важно всегда проверять допуск, предоставленный контракту обработчика в соответствующем контракте токена ERC721. Вы можете утвердить контракт обработчика на отправку токенов от вашего имени, используя функцию `approve()` предусмотренную в интерфейсе. Вы можете проверить одобрение каждого из ваших идентификаторов токенов с помощью функции `getApproved()`.

!!! Примечание:
    Токены будут переданы только в том случае, если контракт обработчика утвержден на передачу токенов от имени владельца. Если процесс не удался, проверьте утверждение.

### Универсальный обработчик

Универсальный обработчик предлагает возможность выполнения функции в цепочке A и создания предложения для выполнения другой функции в цепочке B (аналогично общей диаграмме рабочего процесса). Это обеспечивает удобный способ соединения двух независимых блоков цепочек.

Общий адрес обработчика:
```
{{ networks.moonbase.chainbridge.generic_handler }}
```

Если вы заинтересованы в реализации этой функции, вы можете связаться с нами напрямую через наш [сервер Discord](https://discord.com/invite/PfpUATX). Будем рады обсудить эту реализацию.

### Пользовательский интерфейс Moonbase Alpha ChainBridge UI

Если вы хотите поиграть с переносом токенов ERC20S из Moonbase Alpha в Kovan или Rinkeby без необходимости настраивать контракты в Remix, вы можете проверить наши [Пользовательский интерфейс Moonbase Alpha ChainBridge UI](https://moonbase-chainbridge.netlify.app/transfer).

