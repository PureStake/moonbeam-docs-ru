---
title: Razor Network
description: Как использовать данные запрос от Razor Network Oracle в вашем DApp Moonbeam Ethereum с помощью смарт-контрактов
---
# Оракул сети Razor

![Razor Network Moonbeam Diagram](/images/razor/razor-banner.png)

## Вступление {: #introduction } 

Теперь разработчики могут получать цены из оракула Razor Network, используя контракт моста, развернутый в Moonbase Alpha TestNet. Этот мост действует как промежуточное программное обеспечение, и генерируемые им события извлекаются инфраструктурой оракула Razor Network, отправляя цены в контракт моста.

Чтобы получить доступ к этим ценовым фидам, нам нужно взаимодействовать с адресом контракта моста, который можно найти в следующей таблице:

|     Network    | |         Contract Address        |
|:--------------:|-|:------------------------------------------:|
| Moonbase Alpha | | 0x53f7660Ea48289B5DA42f1d79Eb9d4F5eB83D3BE |

## Работа {: #jobs } 

К каждому каналу данных прикреплен идентификатор задания. Например:

|    Job ID    | |    Underlying Price [USD]  |
|:------------:|-|:--------------------------:|
|       1      | |            ETH             |
|       2      | |            BTC             |
|       3      | |      Microsoft Stocks      |

Вы можете проверить идентификаторы задачи для каждого потока данных по следующей [ссылке](https://razorscan.io/#/custom). Прайс-каналы обновляются каждые 5 минут. Дополнительную информацию можно найти на [веб-сайте документации Razor][https://docs.razor.network/].

## Получить данные из контракта моста {: #get-data-from-bridge-contract } 

Контракты могут запрашивать данные в цепочке, такие как цены на токены, из оракула сети Razor , реализуя интерфейс контракта Bridge, который предоставляет функции `getResult` и `getJob`.

```
pragma solidity 0.6.11;

interface Razor {
    
    function getResult(uint256 id) external view returns (uint256);
    
    function getJob(uint256 id) external view returns(string memory url, string memory selector, string memory name, bool repeat, uint256 result);
}
```

Первая функция, `getResult`, берет идентификатор задания, связанный с потоком данных, и получает цену. Например, если мы передадим 1, мы получим цену потока данных, связанную с идентификатором задачи.

Вторая функция, `getJob`, принимает идентификатор задания, связанный с фидом данных, и извлекает общую информацию о фиде данных, такую как имя фида данных, цена и URL-адрес, используемый для получения цен.

### Пример контракта {: #example-contract } 

Мы развернули контракт моста в Moonbase Alpha TestNet (по адресу `{{ networks.moonbase.razor.bridge_address }}`) чтобы Вы могли быстро проверить информацию, поступающую от оракула сети Razor .

Единственное требование — это интерфейс Bridge, который определяет структуру `getResult` и делает функции доступными контракту для запросов.


Мы можем использовать следующий `Demo` скрипт. Он предоставляет различные функции:

 - fetchPrice:  _view_ функция просмотра, которая запрашивает один идентификатор задания. Например, чтобы получить цену `ETH` и `USD`,  нам нужно будет отправить идентификатор задачи `1`
 - fetchMultiPrices:  _view_ функция просмотра, которая запрашивает несколько идентификаторов задач. Например, чтобы получить цену  `ETH` и `BTC` в `USD`, нам нужно будет отправить идентификаторы задач `[1,2]`
 - savePrice:  _public_ общедоступная функция, которая запрашивает один идентификатор задачи. Это отправляет транзакцию и изменяет `price` переменную, хранящуюся в контракте.
 - saveMultiPrices:  _public_ общедоступная функция, которая запрашивает несколько идентификаторов задач. Например, чтобы получить цену `ETH` и `BTC` в `USD`, нам нужно будет отправить идентификаторы задач `[1,2]`. Это отправляет транзакцию и изменяет массив `pricesArr` хранящийся в контракте, который будет содержать цену каждой пары в том же порядке, как указано во входных данных.

```sol
pragma solidity 0.6.11;

interface Razor {
    function getResult(uint256 id) external view returns (uint256);
    function getJob(uint256 id) external view returns(string memory url, string memory selector, string memory name, bool repeat, uint256 result);
}

contract Demo {
    // Interface
    Razor internal razor;
    
    // Variables
    uint256 public price;
    uint256[] public pricesArr;

    constructor(address _bridgeAddress) public {
        razor = Razor(_bridgeAddress); // Bridge Contract Address
                                       // Moonbase Alpha {{ networks.moonbase.razor.bridge_address }}
    }

    function fetchPrice(uint256 _jobID) public view returns (uint256){
        return razor.getResult(_jobID);
    }
    
    function fetchMultiPrices(uint256[] memory jobs) external view returns(uint256[] memory){
        uint256[] memory prices = new uint256[](jobs.length);
        for(uint256 i=0;i<jobs.length;i++){
            prices[i] = razor.getResult(jobs[i]);
        }
        return prices;
    }
    
    function savePrice(uint _jobID) public {
        price = razor.getResult(_jobID);
    }

    function saveMultiPrices(uint[] calldata _jobIDs) public {
        delete pricesArr;
        
        for (uint256 i = 0; i < _jobIDs.length; i++) {
            pricesArr.push(razor.getResult(_jobIDs[i]));
        }

    }
}
```

### Попробуйте на Moonbase Alpha {: #try-it-on-moonbase-alpha } 

Самый простой способ попробовать их реализацию Oracle — это указать интерфейс на контракт Bridge, развернутый по адресу `{{ networks.moonbase.razor.bridge_address }}`:

```sol
pragma solidity 0.6.11;

interface Razor {
    function getResult(uint256 id) external view returns (uint256);
    function getJob(uint256 id) external view returns(string memory url, string memory selector, string memory name, bool repeat, uint256 result);
}
```

С его помощью Вам будут доступны две функции просмотра, очень похожие на наши предыдущие примеры:

 - getPrice: предоставляет поток цен для одного идентификатора задачи, который вводится в функцию. Например, чтобы получить цену `ETH` в `USD`, нам нужно будет отправить идентификатор задачи `1`
 - getMultiPrices: предоставляет поток цен для нескольких идентификаторов задач, заданных как входной массив функции. Например, чтобы получить цену `ETH` и `BTC` в `USD`, нам нужно будет отправить идентификаторы задач `[1,2]`

Давайте воспользуемся [Remix](/integrations/remix/) чтобы получить цену `BTC` в `USD`.

После создания файла и компиляции контракта перейдите на вкладку «Deploy and Run Transactions», введите адрес контракта (`{{ networks.moonbase.razor.bridge_address }}`), и нажмите "At Address." Убедитесь, что Вы установили "Environment" на "Injected Web3" чтобы Вы подключились к Moonbase Alpha (через провайдера Web3 кошелька). 

![Razor Remix deploy](/images/razor/razor-demo1.png)

Это создаст экземпляр демонстрационного контракта, с которым Вы можете взаимодействовать. Используйте функции `getPrice()` и `getMultiPrices()` для запроса данных соответствующей пары.

![Razor check price](/images/razor/razor-demo2.png)

## Contact Us {: #contact-us } 
Если у Вас есть фидбек относительно реализации Razor Network Oracle в Vашем проекте или по любой другой теме, связанной с Moonbeam, не стесняйтесь обращаться к нам через наш официальный [Сервер Discord](https://discord.com/invite/PfpUATX).
