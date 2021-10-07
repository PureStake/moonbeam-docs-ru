---
title: Chainlink
description: Как использовать данные запроса от Oracle Chainlink в вашем приложении Moonbeam Ethereum Dapp с помощью смарт-контрактов или Javascript
---

# Chainlink Oracle

![Chainlink Moonbeam Banner](/images/builders/integrations/oracles/chainlink/chainlink-banner.png)

## Вступление {: #introduction } 

Теперь разработчики могут использовать [децентрализованную сеть Оракулов Chainlink](https://chain.link/) для сбора данных в Moonbase Alpha TestNet. В этой инструкции рассматриваются два способа использования Оракулов Chainlink:

 - [Базовая модель запроса](https://docs.chain.link/docs/architecture-request-model) - конечный пользователь отправляет запрос провайдеру Оракулов, который извлекает данные по API и выполняет запрос, сохраняя эти данные в сети.
 - [Price Feeds](https://docs.chain.link/docs/architecture-decentralized-model) - данные постоянно обновляются операторами Оракулов в смарт-контракте так, чтобы другие смарт-контракты могли их получать.

## Базовая Модель Запроса {: #basic-request-model } 

Прежде чем мы перейдем к извлечению самих данных, важно понять основы "базовой модели запроса".

--8<-- 'text/chainlink/chainlink-brm.md'

### Клиентский Контракт {: #the-client-contract } 

Клиентский контракт — это элемент, который запускает интеграцию с Оракулом путем отправки запроса. Как показано на диаграмме, он вызывает метод _transferAndCall_ из контракта LINK токена, но для отслеживания запроса к Оракулу требуется дополнительная обработка. Для этого примера Вы можете использовать код из [данного файла](/snippets/code/chainlink/Client.sol), и разместить его на [Remix](/integrations/remix/) в качестве теста. Давайте рассмотрим основные функции контракта:

 - _constructor_: запускается при размещении контракта. Он устанавливает адрес LINK токена и владельца контракта.
 - _requestPrice_: требует адрес контракта в Оракуле, job ID и токены оплаты (LINK) исполнителя запроса. Создает новый запрос, который отправляется с помощью функции _sendChainlinkRequestTo_ function from the _ChainlinkClient.sol_ из импорта
 - _fulfill_: колбэк, используемый нодой Оракула для выполнения запроса путем сохранения запрашиваемой информации в контракте.

```solidity
pragma solidity ^0.6.6;

import "https://github.com/smartcontractkit/chainlink/evm-contracts/src/v0.6/ChainlinkClient.sol";

contract Client is ChainlinkClient {
  //... there is mode code here

  constructor(address _link) public {
    // Set the address for the LINK token for the network
    setChainlinkToken(_link);
    owner = msg.sender;
  }

  function requestPrice(address _oracle, string memory _jobId, uint256 _payment)
    public
    onlyOwner
  {
    // newRequest takes a JobID, a callback address, and callback function as input
    Chainlink.Request memory req = buildChainlinkRequest(stringToBytes32(_jobId), address(this), this.fulfill.selector);
    // Sends the request with the amount of payment specified to the oracle
    sendChainlinkRequestTo(_oracle, req, _payment);
  }

  function fulfill(bytes32 _requestId, uint256 _price)
    public
    recordChainlinkFulfillment(_requestId)
  {
    currentPrice = _price;
  }

  //... there is more code here
}
```

Обратите внимание, что Клиентский контракт должен иметь токены LINK на балансе, чтобы иметь возможность оплатить данный запрос. Однако при размещении сетапа можно установить значение LINK равным 0 в контракте `ChainlinkClient.sol`, но Вам все равно необходимо иметь размещенный в LINK токенах контракт.

### Протестируйте это в Moonbase Alpha {: #try-it-on-moonbase-alpha } 

Если Вы хотите минимизировать сложности, связанные с размещением контрактов, настройкой ноды Оракула, созданием job ID и т. д., то мы Вам поможем.

Для вас доступен кастомный Клиентский контракт на Moonbase Alpha, который делает все запросы к нашему контракту в Оракуле с 0 оплатой. Эти запросы исполняются нодой Оракула, которую мы запускаем. Вы можете попробовать выполнить это на следующем Клиентском контракте, размещенном по адресу `{{ networks.moonbase.chainlink.client_contract }}`:

```solidity
pragma solidity ^0.6.6;

/**
 * @title Simple Interface to interact with Universal Client Contract
 * @notice Client Address {{ networks.moonbase.chainlink.client_contract }}
 */
interface ChainlinkInterface {

  /**
   * @notice Creates a Chainlink request with the job specification ID,
   * @notice and sends it to the Oracle.
   * @notice _oracle The address of the Oracle contract fixed top
   * @notice _payment For this example the PAYMENT is set to zero
   * @param _jobId The job spec ID that we want to call in string format
   */
    function requestPrice(string calldata _jobId) external;

    function currentPrice() external view returns (uint);

}
```

Это обеспечивает нам две функции: `requestPrice()` функция, которой нужен только job ID данных, которые Вы хотите запросить. Эта функция запускает цепь событий, описанных ранее. `currentPrice()` функция представления, которая возвращает последнюю цену, хранящуюся в контракте.

В настоящее время в ноде Оракула имеется набор Job ID для различных прайсов по следующим парам:

|  Base/Quote  |     |                 Job ID Reference                  |
| :----------: | --- | :-----------------------------------------------: |
|  BTC to USD  |     |  {{ networks.moonbase.chainlink.basic.btc_usd }}  |
|  ETH to USD  |     |  {{ networks.moonbase.chainlink.basic.eth_usd }}  |
|  DOT to USD  |     |  {{ networks.moonbase.chainlink.basic.dot_usd }}  |
|  KSM to USD  |     |  {{ networks.moonbase.chainlink.basic.ksm_usd }}  |
| AAVE to USD  |     | {{ networks.moonbase.chainlink.basic.aave_usd }}  |
| ALGO to USD  |     | {{ networks.moonbase.chainlink.basic.algo_usd }}  |
| BAND to USD  |     | {{ networks.moonbase.chainlink.basic.band_usd }}  |
| LINK to USD  |     | {{ networks.moonbase.chainlink.basic.link_usd }}  |
| SUSHI to USD |     | {{ networks.moonbase.chainlink.basic.sushi_usd }} |
|  UNI to USD  |     |  {{ networks.moonbase.chainlink.basic.uni_usd }}  |

Далее используем интерфейсный контракт с Job ID `BTC в USD` в Job ID в [Remix](/integrations/remix/).

После создания файла и компиляции контракта перейдите на вкладку "Deploy and Run Transactions", введите адрес Клиентского контракта и нажмите кнопку "At Address". Убедитесь, что Вы установили "Environment" в "Injected Web3" для подключения к Moonbase Alpha. Это создаст экземпляр Клиентского контракта, с которым Вы сможете взаимодействовать. Используйте функцию `requestPrice()` для запроса данных соответствующего Job ID. После подтверждения транзакции нужно подождать, пока не завершится весь процесс, указанный ранее. Мы можем проверить цену с помощью функции `currentPrice()`.

![Chainlink Basic Request on Moonbase Alpha](/images/builders/integrations/oracles/chainlink/chainlink-1.png)

Если у Вас есть какая-то конкретная пара, которую Вы хотите, чтобы мы включили в список, не стесняйтесь обращаться к нам через наш [Discord server](https://discord.com/invite/PfpUATX).

### Запуск собственного Клиентского контракта {: #run-your-client-contract } 

Если Вы хотите запустить свой Клиентский контракт, но используете нашу ноду Оракула, Вы можете сделать это с помощью данных, указанных ниже:

|  Contract Type  |     |                      Address                      |
| :-------------: | --- | :-----------------------------------------------: |
| Oracle Contract |     | {{ networks.moonbase.chainlink.oracle_contract }} |
|   LINK Token    |     |  {{ networks.moonbase.chainlink.link_contract }}  |

Обратите внимание, что оплата в LINK токенах = 0.

### Other Requests {: #other-requests } 

Оракулы Chainlink могут предоставлять множество различных типов каналов данных с использованием внешних адаптеров. Однако для простоты наша нода Оракула настроена на предоставление только прайс-фидов.

Если вы заинтересованы в запуске собственной ноды Оракула в Moonbeam, пожалуйста, прочитайте [это руководство](/node-operators/oracles/node-chainlink/). Кроме того, мы рекомендуем ознакомиться с [документацией Chainlink](https://docs.chain.link/docs) на сайте.

## Прайс-фиды {: #price-feeds } 

Прежде чем мы перейдем к извлечению самих данных, важно понять основы прайс-фидов.

В стандартной конфигурации каждый прайс-фид обновляется децентрализованной сетью Оракула. Каждая нода Оракула получает вознаграждение за публикацию прайс-фидов в контракте Агрегатора. Однако информация обновляется только в том случае, если получено минимальное количество ответов от нод Оракулов (во время раунда агрегации).

Конечный пользователь может получать прайс-фиды с помощью read-only операций через Потребительский контракт, ссылаясь на правильный интерфейс агрегатора (Прокси-контракт). Прокси выступает в качестве промежуточного программного обеспечения для предоставления Потребителю наиболее актуального Агрегатора для конкретного прайс-фида.

![Price Feed Diagram](/images/builders/integrations/oracles/chainlink/chainlink-price-feed.png)

### Протестируйте это в Moonbase Alpha {: #try-it-on-moonbase-alpha } 

Если вы хотите минимизировать сложности, связанные с размещением контрактов, настройкой ноды Оракула, созданием job ID и т. д., то мы Вам поможем.

Мы разместили все необходимые контракты на Moonbase Alpha, чтобы упростить процесс запроса прайс-фидов. В нашей текущей конфигурации мы запускаем только одну ноду Оракула, которая получает цену из одного источника по API. Данные по ценам проверяются каждую минуту и обновляются в смарт-контрактах каждый час, если только отклонение цены не составляет 1%.

Данные хранятся в серии смарт-контрактов (по одному на прайс-фид) и могут быть извлечены с помощью следующего интерфейса:

```solidity
pragma solidity ^0.6.6;

interface ConsumerV3Interface {
    /**
     * Returns the latest price
     */
    function getLatestPrice() external view returns (int);

    /**
     * Returns the decimals to offset on the getLatestPrice call
     */
    function decimals() external view returns (uint8);

    /**
     * Returns the description of the underlying price feed aggregator
     */
    function description() external view returns (string memory);
}
```

Это обеспечивает нам три функции: `getLatestPrice()` будет считывать последние ценовые данные, доступные в контракте Агрегатора через Прокси. Мы добавили `decimals()` функцию, которая возвращает количество десятичных знаков данных и `description()` функция, которая возвращает краткое описание прайс-фида, доступного в запрашиваемом контракте Агрегатора.

В настоящее время существует Потребительский контракт для следующих ценовых пар:

|  Base/Quote  |     |                     Consumer Contract                     |
| :----------: | --- | :-------------------------------------------------------: |
|  BTC to USD  |     |  {{ networks.moonbase.chainlink.feed.consumer.btc_usd }}  |
|  ETH to USD  |     |  {{ networks.moonbase.chainlink.feed.consumer.eth_usd }}  |
|  DOT to USD  |     |  {{ networks.moonbase.chainlink.feed.consumer.dot_usd }}  |
|  KSM to USD  |     |  {{ networks.moonbase.chainlink.feed.consumer.ksm_usd }}  |
| AAVE to USD  |     | {{ networks.moonbase.chainlink.feed.consumer.aave_usd }}  |
| ALGO to USD  |     | {{ networks.moonbase.chainlink.feed.consumer.algo_usd }}  |
| BAND to USD  |     | {{ networks.moonbase.chainlink.feed.consumer.band_usd }}  |
| LINK to USD  |     | {{ networks.moonbase.chainlink.feed.consumer.link_usd }}  |
| SUSHI to USD |     | {{ networks.moonbase.chainlink.feed.consumer.sushi_usd }} |
|  UNI to USD  |     |  {{ networks.moonbase.chainlink.feed.consumer.uni_usd }}  |

Далее используем интерфейсный контракт для получения прайс-фида `BTC в USD` с помощью [Remix](/integrations/remix/).

После создания файла и компиляции контракта перейдите на вкладку "Deploy and Run Transactions", введите адрес Потребительского контракта, соответствующий `BTC to USD`, и нажмите кнопку "At Address". Убедитесь, что Вы установили "Environment" в "Injected Web3" для подключения к Moonbase Alpha.

Это создаст экземпляр Потребительского контракта, с которым Вы сможете взаимодействовать. Используйте функцию `getLatestPrice()` для запроса данных соответствующего прайс-фида.

![Chainlink Price Feeds on Moonbase Alpha](/images/builders/integrations/oracles/chainlink/chainlink-2.png)

Обратите внимание, что для получения реальной цены необходимо учитывать десятичные дроби прайс-фида, доступные при вызове метода `decimals()`.

Если у вас есть какая-то конкретная пара, которую Вы хотите, чтобы мы включили в список, не стесняйтесь обращаться к нам через наш [Discord сервер](https://discord.com/invite/PfpUATX).
