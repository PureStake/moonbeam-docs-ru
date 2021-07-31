---
title: Band Protocol
description: Как использовать данные запроса от Oracle Band Protocol в вашем DApp Moonbeam Ethereum с помощью смарт-контрактов или javascript
---
# Оракул Band Protocol

![Band Protocol Moonbeam Diagram](/images/band/band-banner.png)

## Вступление {: #introduction }
У разработчиков есть два способа получить цены из инфраструктуры Oracle . С одной стороны, они могут использовать смарт-контракты Band на Moonbeam. При этом они получают доступ к данным, которые находятся в цепочке и обновляются либо через регулярные промежутки времени, либо когда проскальзывание цены превышает целевое значение (разное для каждого токена). С другой стороны, разработчики могут использовать вспомогательную библиотеку Javascript, которая использует конечную точку API для извлечения данных с использованием функций, аналогичных функциям смарт-контрактов, но эта реализация полностью обходит блокчейн. Это может быть полезно, если Вашему интерфейсу DApp требуется прямой доступ к данным.

Адрес контракта агрегатора можно найти в следующей таблице:

|     Network    | |         Aggregator Contract Address        |
|:--------------:|-|:------------------------------------------:|
| Moonbase Alpha | | 0xDA7a001b254CD22e46d3eAB04d937489c93174C3 |

## Поддерживаемый токен {: #supported-token } 
Ценовые запросы любого номинала доступны при условии, что поддерживаются базовые символы и символы котировки (_основание_/_котировка_). Например:

 - `BTC/USD`
 - `BTC/ETH`
 - `ETH/EUR`

На момент написания статьи список поддерживаемых символов можно найти по [этой ссылке](https://data.bandprotocol.com). Для запроса доступно более 146 пар.

## Запрос цен {: #querying-prices } 
Как указывалось ранее, разработчики могут использовать два метода для запроса цен у оракула Band:

 - Смарт-контракт Band на Moonbeam (на данный момент развернут в Moonbase Alpha TestNet)
 - Вспомогательная библиотека Javascript

## Получение данных с помощью смарт-контрактов {: #get-data-using-smart-contracts }
Контракты могут запрашивать данные в цепочке, такие как цены токенов, из оракула Band, реализуя интерфейс контракта `StdReference` который предоставляет функции `getReferenceData` и `getReferenceDataBulk`

Первая функция, `getReferenceData`, принимает в качестве входных данных две строки (основание и символ кавычки). Функция запрашивает у контракта `StdReference` последние доступные ставки для этих двух токенов. Он возвращает структуру `ReferenceData`

Структура `ReferenceData` имеет следующие элементы:

 - Курс: обменный курс с точки зрения базы / котировки. Возвращаемое значение умножается на 10<sup>18 степень</sup>
 - Последняя обновленная база: последний раз, когда обновлялась базовая цена (с эпохи UNIX)
 - Последнее обновление котировки: последний раз, когда котировочная цена была обновлена ​​(с эпохи UNIX)
 
```
struct ReferenceData {
   uint256 rate; 
   uint256 lastUpdatedBase; 
   uint256 lastUpdatedQuote;
}
```

Вторая функция, `getReferenceDataBulk`, принимает информацию в виде массивов данных. Например, если мы передадим `['BTC','BTC','ETH']` в качестве базы и `['USD','ETH','EUR']` в качестве цитаты, массив будет `ReferenceData` содержать информацию о следующих парах:

 - `BTC/USD`
 - `BTC/ETH`
 - `ETH/EUR`

### Пример контракта {: #example-contract } 

Следующий код смарт-контракта предоставляет несколько простых примеров контракта `StdReference` и функции `getReferenceData`  они не предназначены для производства. Интерфейс `IStdReference.sol` определяет структуру ReferenceData и функции, доступные для выполнения запросов.

```sol
pragma solidity 0.6.11;
pragma experimental ABIEncoderV2;

interface IStdReference {
    /// A structure returned whenever someone requests for standard reference data.
    struct ReferenceData {
        uint256 rate; // base/quote exchange rate, multiplied by 1e18.
        uint256 lastUpdatedBase; // UNIX epoch of the last time when base price gets updated.
        uint256 lastUpdatedQuote; // UNIX epoch of the last time when quote price gets updated.
    }

    /// Returns the price data for the given base/quote pair. Revert if not available.
    function getReferenceData(string memory _base, string memory _quote)
        external
        view
        returns (ReferenceData memory);

    /// Similar to getReferenceData, but with multiple base/quote pairs at once.
    function getReferenceDataBulk(string[] memory _bases, string[] memory _quotes)
        external
        view
        returns (ReferenceData[] memory);
}
```
Далее мы можем использовать следующий скрипт `DemoOracle` Он предоставляет четыре функции:

 - getPrice:  _view_ функция просмотра, которая запрашивает одну базу. В этом примере цена `BTC` указана в долларах `USD`
 - getMultiPrices:  _view_ функция просмотра, которая запрашивает несколько баз. В этом примере цена `BTC` и `ETH`, оба указаны в долларах `USD`
 - savePrice:  _public_ oбщедоступная функция, которая запрашивает пару _base/quotes_ . Каждый элемент представлен в виде отдельных строк, например `_base = "BTC", _quotes = "USD"`. Это отправляет транзакцию и изменяет `ценовую`  переменную, хранящуюся в контракте.
 - saveMultiPrices:  _public_  общедоступная функция, которая запрашивает каждую пару _base/quotes_. Каждый элемент представлен в виде массива строк. Например, `_bases = ["BTC","ETH"], _quotes = ["USD","USD"]`. Это отправляет транзакцию и изменяет массив цен, хранящийся в контракте, который будет содержать `цену` каждой пары в том же порядке, как указано во входных данных.

При развертывании функции конструктора требуется адрес контракта агрегатора для целевой сети.

```sol
pragma solidity 0.6.11;
pragma experimental ABIEncoderV2;

import "./IStdReference.sol";

contract DemoOracle {
    IStdReference ref;
    
    uint256 public price;
    uint256[] public pricesArr;

    constructor(IStdReference _ref) public {
        ref = _ref; // Aggregator Contract Address
                    // Moonbase Alpha 0xDA7a001b254CD22e46d3eAB04d937489c93174C3

    }

    function getPrice(string memory _base, string memory _quote) external view returns (uint256){
        IStdReference.ReferenceData memory data = ref.getReferenceData(_base,_quote);
        return data.rate;
    }

    function getMultiPrices(string[] memory _bases, string[] memory _quotes) external view returns (uint256[] memory){
        IStdReference.ReferenceData[] memory data = ref.getReferenceDataBulk(_bases,_quotes);

        uint256 len = _bases.length;
        uint256[] memory prices = new uint256[](len);
        for (uint256 i = 0; i < len; i++) {
            prices[i] = data[i].rate;
        }

        return prices;
    }
    
    function savePrice(string memory _base, string memory _quote) external {
        IStdReference.ReferenceData memory data = ref.getReferenceData(_base,_quote);
        price = data.rate;
    }
    
    function saveMultiPrices(
        string[] memory _bases,
        string[] memory _quotes
    ) public {
        require(_bases.length == _quotes.length, "BAD_INPUT_LENGTH");
        uint256 len = _bases.length;
        IStdReference.ReferenceData[] memory data = ref.getReferenceDataBulk(_bases,_quotes);
        delete pricesArr;
        for (uint256 i = 0; i < len; i++) {
            pricesArr.push(data[i].rate);
        }
        
    }
}
```

### Протестируйте в Moonbase Alpha {: #try-it-in-moonbase alpha } 

Мы развернули контракт, доступный в Moonbase Alpha TestNet (по адресу `0xf15c870344c1c02f5939a5C4926b7cDb90dEc655`) поэтому Вы можете легко проверить информацию, поступающую из оракула Band Protocol. Для этого вам понадобится следующий интерфейсный контракт:

```sol
pragma solidity 0.6.11;
pragma experimental ABIEncoderV2;

interface TestInterface {
    function getPrice(string memory _base, string memory _quote) external view returns (uint256);

    function getMultiPrices(string[] memory _bases, string[] memory _quotes) external view returns (uint256[] memory);
}
```

С его помощью Вам будут доступны две функции просмотра — очень похожие на наши предыдущие примеры:

 - getPrice: предоставляет ценовой поток для одной пары базовая цена / котировка, которая передается в качестве входных данных функции, то есть "BTC", "USD"
 - getMultiPrices: предоставляет поток цен для нескольких пар базовая цена / котировка, которые передаются в качестве входных данных функции, то есть, ["BTC", "ETH", "ETH"], ["USD", "USD", "EUR"]

Например, используя [Remix](/integrations/remix/), мы можем легко запросить ценовую пару `BTC/USD` с помощью этого интерфейса.

После создания файла и компиляции контракта перейдите на вкладку «Развертывание и выполнение транзакций», введите адрес контракта (`0xf15c870344c1c02f5939a5C4926b7cDb90dEc655`) и нажмите «At Address». Убедитесь, что вы установили «Environment» на «Injected Web3», чтобы Вы подключились к Moonbase Alpha.

![Band Protocol Remix deploy](/images/band/band-demo1.png)

Это создаст экземпляр демонстрационного контракта, с которым Вы можете взаимодействовать. Используйте функции `getPrice()` и `getMultiPrices()` для запроса данных соответствующей пары.

![Band Protocol Remix check price](/images/band/band-demo2.png)

## BandChain.js вспомогательная библиотека Javascript {: #bandchainjs-javascript-helper-library } 

Вспомогательная библиотека также поддерживает аналогичную функцию 'getReferenceData'. Для начала необходимо установить библиотеку:

```
npm install @bandprotocol/bandchain.js
```

Библиотека предоставляет функцию-конструктор, для которой требуется указать конечную точку. Это возвращает экземпляр, который затем включает все необходимые методы, такие как функция `getReferenceData`.  При запросе информации функция принимает массив, каждый элемент которого является необходимой парой _base/quote_. Например:

```
getReferenceData(['BTC/USD', 'BTC/ETH', 'ETH/EUR'])
```

Затем он возвращает объект массива со следующей структурой:

```
[
  {
    pair: 'BTC/USD',
    rate: rate,
    updated: { base: lastUpdatedBase, quote: lastUpdatedQuote}
  },
  {
    pair: 'BTC/ETH',
    rate: rate,
    updated: { base: lastUpdatedBase, quote: lastUpdatedQuote}
  },
  {
    pair: 'ETH/EUR',
    rate: rate,
    updated: { base: lastUpdatedBase, quote: lastUpdatedQuote}
  }
]
```
Где `lastUpdatedBase` и `lastUpdatedQuote` это последний раз, когда базовая цена и котировка обновлялись соответственно (с эпохи UNIX).

### Пример использования {: #example-usage } 

Следующий сценарий Javascript предоставляет простой пример функции `getReferenceData`.

```js
const BandChain = require('@bandprotocol/bandchain.js');

const queryData = async () => {
  const endpoint = 'https://poa-api.bandchain.org';

  const bandchain = new BandChain(endpoint);
  const dataQuery = await bandchain.getReferenceData(['BTC/USD', 'BTC/ETH', 'ETH/EUR']);
  console.log(dataQuery);
};

queryData();
```

Мы можем выполнить этот код с помощью ноды, и следующий вывод `dataQuery` должен выглядеть следующим образом:


![Band Protocol JavaScript Library](/images/band/band-console.png)

Обратите внимание, что по сравнению с запросом, выполненным с помощью смарт-контрактов, результат отображается непосредственно в правильных единицах.
