---
title: Контракты и Библиотеки
description:  Узнайте, как легко создавать общие контракты OpenZeppelin с помощью Мастера контрактов и как устанавливать их на Moonbeam с использованием функций совместимости с Ethereum.
---

# Контракты и библиотеки OpenZeppelin

![Баннер контрактов OpenZeppelin](/images/builders/interact/oz-remix/oz-contracts-banner.png)

## Введение {: #introduction } 

Контракты и библиотеки OpenZeppelin стали стандартом в отрасли. Они помогают разработчикам свести к минимуму риски, поскольку их открытые исходные шаблоны кода проверены в действии для Ethereum и других блокчейнов. Их код включает наиболее используемые варианты реализации стандартов и дополнений ERC и часто встречается в руководствах и инструкциях сообщества.

Поскольку Moonbeam полностью совместим с Ethereum, все контракты и библиотеки OpenZeppelin могут быть реализованы без каких-либо изменений.

Это руководство состоит из двух разделов. В первом разделе описывается Мастер контрактов OpenZeppelin - отличный онлайн-инструмент, который поможет вам создавать смарт-контракты с использованием кода OpenZeppelin. Второй раздел содержит пошаговое руководство по установке этих контрактов на Moonbeam.

## Мастер контрактов OpenZeppelin {: #openzeppelin-contract-wizard } 

Компания OpenZeppelin разработала интерактивный веб-инструмент для создания контрактов, который, вероятно, является самым простым и быстрым способом написания смарт-контракта с использованием кода OpenZeppelin. Инструмент называется Contracts Wizard, и вы можете найти его на их [сайте в разделе документация](https://docs.openzeppelin.com/contracts/4.x/wizard).

В настоящее время Мастер контрактов поддерживает следующие стандарты ERC:

 - [**ERC20**](https://ethereum.org/en/developers/docs/standards/tokens/erc-20/) — стандарт взаимозаменяемых токенов, который соответствует требованиям [EIP-20](https://eips.ethereum.org/EIPS/eip-20). Взаимозаменяемость означает, что все токены эквивалентны и взаимозаменяемы, то есть имеют одинаковую стоимость. Одним из типичных примеров взаимозаменяемых жетонов являются фиатные валюты, где каждая купюра равного номинала имеет одинаковую стоимость.
 - [**ERC721**](https://ethereum.org/en/developers/docs/standards/tokens/erc-721/) — невзаимозаменяемый контракт на токены, который соответствует требованиям [EIP-721](https://eips.ethereum.org/EIPS/eip-721). Не взаимозаменяемость означает, что каждый токен отличается от других, а значит, уникален. Токен ERC721 может представлять право собственности на этот уникальный предмет, будь то коллекционный предмет в игре, реальное состояние и так далее.
 - [**ERC1155**](https://docs.openzeppelin.com/contracts/4.x/erc1155) — также известен как мульти-токен контракт, поскольку он может представлять как взаимозаменяемые, так и не взаимозаменяемые токены в одном смарт-контракте. Из него следует [EIP-1155](https://eips.ethereum.org/EIPS/eip-1155)

Мастер состоит из следующих разделов:

 1. **Выбор стандарта токена** — показывает все различные стандарты, поддерживаемые мастером
 2. **Настройки** — предоставляет базовые настройки для каждого стандарта токенов, такие как имя токена, символ, pre-mint (поставка токенов при запуске контракта) и URI (для невзаимозаменяемых токенов)
 3. **Характеристики** — список всех возможностей, доступных для каждого стандарта токенов. Более подробную информацию о различных возможностях можно найти в следующих ссылках:
     - [ERC20](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20)
     - [ERC721](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721)
     - [ERC1155](https://docs.openzeppelin.com/contracts/4.x/api/token/erc1155)
 4. **Контроль доступа** — список всех доступных [механизмов контроля доступа](https://docs.openzeppelin.com/contracts/4.x/access-control) для каждого стандарта токенов
 5. **Интерактивное отображение кода** — показывает код смарт-контракта с конфигурацией, заданной пользователем

![Мастер контрактов OpenZeppelin](/images/builders/interact/oz-remix/oz-wizard-1.png)

После того как вы настроите свой контракт со всеми параметрами и функциями, вам останется только скопировать и вставить код в файл контракта.

## Установка контрактов OpenZeppelin на Moonbeam {: #deploying-openzeppelin-contracts-on-moonbeam } 

В этом разделе описаны шаги по установке контрактов OpenZeppelin на Moonbeam. В нем рассматриваются следующие контракты:

 - ERC20 (взаимозаменяемые жетоны)
 - ERC721 (невзаимозаменяемые жетоны)
 - ERC1155 (стандарт мульти-токенов)

Весь код контрактов был получен с помощью OpenZeppelin [Contract Wizard](https://docs.openzeppelin.com/contracts/4.x/wizard).
 
### Проверка предварительных условий {: #checking-prerequisites } 

The steps described in this section assume you have [MetaMask](https://metamask.io/) installed and connected to the Moonbase Alpha TestNet. Contract deployment is done using the [Remix IDE](https://remix.ethereum.org/) via the "Injected Web3" environment. You can find corresponding tutorials in the following links:

Действия, описанные в этом разделе, подразумевают, что у вас есть [MetaMask](https://metamask.io/) установлен и подключен к сети Moonbase Alpha TestNet. Установка контракта выполняется с помощью [Remix IDE](https://remix.ethereum.org/) через среду "Injected Web3". Соответствующие руководства можно найти по следующим ссылкам:
 - [Взаимодействие с Moonbeam с помощью MetaMask](/integrations/wallets/metamask/)
 - [Взаимодействие с Moonbeam с помощью Remix](/integrations/remix/)

### Размещение токена ERC20 {: #deploying-an-erc20-token } 

В данном примере токен ERC20 будет размещен на Moonbase Alpha. Финальный используемый код сочетает в себе различные контракты из OpenZeppelin:

 - **ERC20.sol** — Внедрение токена ERC20 с дополнительными функциями из базового интерфейса. Включает механизм поставки с функцией `mint`, но должен быть специально вызван из основного контракта
 - **Ownable.sol** — расширение для ограничения доступа к определенным функциям

Контракт майнинг токена  ERC20 OpenZeppelin реализует функцию `mint`, которую может вызвать только владелец контракта. По умолчанию владельцем является адрес разработчика контракта. Существует также предварительная оплата в размере `1000` токенов, отправляемая внедряющему контракт лицу, настроенному в функции `constructor`.

Первый шаг - перейдите к [Remix](https://remix.ethereum.org/) и выполните следующие действия:

 1. Нажмите на значок " Create New File" и задайте имя файла. Для данного примера оно было задано как `ERC20.sol`.
 2. Убедитесь, что файл был создан успешно. Кликните по файлу, чтобы открыть его в текстовом редакторе
 3. Напишите свой смарт-контракт с помощью редактора файлов. Для данного примера был использован следующий код:

```sol
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract MyToken is ERC20, Ownable {
    constructor() ERC20("MyToken", "MTK") {
        _mint(msg.sender, 1000 * 10 ** decimals());
    }

    function mint(address to, uint256 amount) public onlyOwner {
        _mint(to, amount);
    }
}
```

Этот смарт-контракт на токены ERC20 был создан с помощью [Мастера Контрактов](#openzeppelin-contract-wizard), установив предварительную сумму в `1000` токенов и активировав функцию `Mintable`.

![Начало работы с Remix](/images/builders/interact/oz-remix/oz-contracts-1.png)

После того как ваш смарт-контракт написан, вы можете скомпилировать его, выполнив следующие действия:

 1. Перейдите в раздел "Solidity Compiler'
 2. Нажмите на кнопку компиляции
 3. Кроме того, вы можете выбрать функцию " Auto compile ".

![Компиляция контракта ERC20 с помощью Remix](/images/builders/interact/oz-remix/oz-contracts-2.png)

Когда контракт создан, вы готовы к его внедрению, выполнив следующие действия: 

 1. Перейдите на вкладку " Deploy & Run Transactions" (Развертывание и запуск транзакций)
 2. Измените среду на "Injected Web3". Это позволит использовать MetaMask . Следовательно, контракт будет размещен в любой сети, к которой подключен MetaMask. MetaMask может отобразить всплывающее окно о том, что Remix пытается подключиться к вашему кошельку.
 3. Выберите нужный контракт для установки. В данном примере это контракт `MyToken` в файле `ERC20.sol`.
 4. Если все готово, нажмите на кнопку " Deploy". Просмотрите информацию о транзакции в MetaMask и подтвердите ее
 5. Через несколько секунд транзакция должна быть подтверждена, и вы увидите свой контракт в разделе " Deployed Contracts ".

![Размещение контракта ERC721 с помощью Remix](/images/builders/interact/oz-remix/oz-contracts-3.png)

Вот и все! Вы установили контракт на токен ERC20, используя контракты и библиотеки OpenZeppelin. Далее вы можете взаимодействовать с вашим контрактом токенов через Remix или добавить его в MetaMask.

### Размещение токена ERC721 {: #deploying-an-erc721-token } 

В данном примере токен ERC721 будет размещен на Moonbase Alpha. В финальном коде используются различные контракты из OpenZeppelin:

 - **ERC721** — Реализация токена ERC721 с дополнительными функциями из базового интерфейса. Включает механизм предложения с функцией `_mint`, но должен быть специально вызван из основного контракта
 - **Burnable** — расширение, позволяющее уничтожать токены их владельцами (или утвержденными адресами)
 - **Enumerable** — eрасширение, позволяющее перечислять токены в сети 
 - **Ownable.sol** — расширение для ограничения доступа к определенным функциям

Контракт токена mintable ERC721 OpenZeppelin предоставляет функцию `mint`, которая может быть вызвана только владельцем контракта. По умолчанию владельцем является адрес разработчика контракта.

Как и в случае с [ERC20-контрактом](#deploying-an-erc20-token), первым шагом будет переход на  [Remix](https://remix.ethereum.org/) и создание нового файла. Для данного примера имя файла будет `ERC721.sol`.

Далее вам нужно будет написать смарт-контракт и скомпилировать его. Для данного примера используется следующий код:

```sol
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721Enumerable.sol";
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721Burnable.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract MyToken is ERC721, ERC721Enumerable, ERC721Burnable, Ownable {
    constructor() ERC721("MyToken", "MTK") {}

    function safeMint(address to, uint256 tokenId) public onlyOwner {
        _safeMint(to, tokenId);
    }

    function _baseURI() internal pure override returns (string memory) {
        return "Test";
    }

    function _beforeTokenTransfer(address from, address to, uint256 tokenId)
        internal
        override(ERC721, ERC721Enumerable)
    {
        super._beforeTokenTransfer(from, to, tokenId);
    }

    function supportsInterface(bytes4 interfaceId)
        public
        view
        override(ERC721, ERC721Enumerable)
        returns (bool)
    {
        return super.supportsInterface(interfaceId);
    }
}
```

Этот смарт-контракт с токенами ERC721 был получен из  [Мастера Контрактов] (#openzeppelin-contract-wizard), установив `Base URI` как `Test` и активировав функции `Mintable`, `Burnable` и `Enumerable`.

Скомпилировав контракт, перейдите на вкладку "Deploy & Run Transactions". Здесь вам необходимо:

 1. Измените среду на "Injected Web3". Это позволит использовать  MetaMask. Следовательно, контракт будет развернут в любой сети, к которой подключен MetaMask. MetaMask может отобразить всплывающее окно о том, что Remix пытается подключиться к вашему кошельку.
 2. Выберите нужный контракт для размещения. В данном примере это контракт `MyToken` в файле `ERC721.sol`.
 3. Если все готово, нажмите на кнопку " Deploy". Просмотрите информацию о транзакции в MetaMask и подтвердите ее
 4. Через несколько секунд транзакция должна быть подтверждена, и вы увидите свой контракт в разделе " Deployed Contracts ".

![Размещение контракта ERC721 с помощью Remix](/images/builders/interact/oz-remix/oz-contracts-4.png)

Вот и все! Вы разместили контракт на токен ERC721, используя контракты и библиотеки OpenZeppelin. Далее вы можете взаимодействовать с вашим токен-контрактом через Remix или добавить его в MetaMask.

### Размещение токена ERC1155 {: #deploying-an-erc1155-token } 

В данном примере токен ERC1155 будет развернут на Moonbase Alpha. Используемый финальный код объединяет различные контракты из OpenZeppelin:

 - **ERC1155** — Реализация токена ERC1155 с дополнительными функциями из базового интерфейса. Включает механизм поставки с функцией `_mint`, но должен быть специально вызван из основного контракта
 - **Pausable** — расширение, позволяющее приостанавливать передачу токенов, минтинье и сжигание
 - **Ownable.sol** — расширение для ограничения доступа к определенным функциям

Контракт токенов OpenZeppelin ERC1155 предоставляет функцию `_mint`, которая может быть вызвана только в функции `constructor`. Поэтому в данном примере создается 1000 токенов с идентификатором `0` и 1 уникальный токен с идентификатором `1`.

Первый шаг - зайти в [Remix](https://remix.ethereum.org/) и создайте новый файл. Для данного примера имя файла будет `ERC1155.sol`.

Как показано в примере с [ERC20-токеном](#deploying-an-erc20-token), вам нужно будет написать смарт-контракт и скомпилировать его. Для данного примера используется следующий код:

```sol
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC1155/ERC1155.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/security/Pausable.sol";

contract MyToken is ERC1155, Ownable, Pausable {
    constructor() ERC1155("Test") {
        _mint(msg.sender, 0, 1000 * 10 ** 18, "");
        _mint(msg.sender, 1, 1, "");
    }

    function setURI(string memory newuri) public onlyOwner {
        _setURI(newuri);
    }

    function pause() public onlyOwner {
        _pause();
    }

    function unpause() public onlyOwner {
        _unpause();
    }

    function _beforeTokenTransfer(address operator, address from, address to, uint256[] memory ids, uint256[] memory amounts, bytes memory data)
        internal
        whenNotPaused
        override
    {
        super._beforeTokenTransfer(operator, from, to, ids, amounts, data);
    }
}
```

Этот смарт-контракт на токены ERC1155 был создан с помощью [Мастера Контрактов](#openzeppelin-contract-wizard), не устанавливая `Base URI` и активируя функцию `Pausable`. Функция конструктора была модифицирована, чтобы включить майнинг как взаимозаменяемого, так и не взаимозаменяемого токена.

Скомпилировав контракт, перейдите на вкладку "Deploy & Run Transactions". Здесь вам необходимо:

 1. Измените среду на "Injected Web3". Это позволит использовать  MetaMask. Следовательно, контракт будет размещен в любой сети, к которой подключен MetaMask. MetaMask может показать всплывающее окно о том, что Remix пытается подключиться к вашему кошельку.
 2. Выберите нужный контракт для размещения. В данном примере это контракт `MyToken` в файле `ERC1155.sol`.
 3. Если все готово, нажмите на кнопку " Deploy". Просмотрите информацию о транзакции в MetaMask и подтвердите ее
 4. Через несколько секунд транзакция должна быть подтверждена, и вы увидите свой контракт в разделе " Deployed Contracts ".

![Размещение контракта ERC1155 с помощью Remix](/images/builders/interact/oz-remix/oz-contracts-5.png)

Вот и все! Вы разместили контракт токена ERC1155, используя контракты и библиотеки OpenZeppelin. Теперь вы можете взаимодействовать с вашим контрактом токенов через Remix.
