---
title: Band Protocol
description: Как использовать данные запроса от Oracle Band Protocol в вашем DApp Moonbeam Ethereum с помощью смарт-контрактов или javascript
---
# Оракул Band Protocol

![Band Protocol Moonbeam Diagram](/images/band/band-banner.png)

## Вступление
У разработчиков есть два способа получить цены из инфраструктуры Oracle . С одной стороны, они могут использовать смарт-контракты Band на Moonbeam. При этом они получают доступ к данным, которые находятся в цепочке и обновляются либо через регулярные промежутки времени, либо когда проскальзывание цены превышает целевое значение (разное для каждого токена). С другой стороны, разработчики могут использовать вспомогательную библиотеку Javascript, которая использует конечную точку API для извлечения данных с использованием функций, аналогичных функциям смарт-контрактов, но эта реализация полностью обходит блокчейн. Это может быть полезно, если Вашему интерфейсу DApp требуется прямой доступ к данным.

Адрес контракта агрегатора можно найти в следующей таблице:

|     Network    | |         Aggregator Contract Address        |
|:--------------:|-|:------------------------------------------:|
| Moonbase Alpha | | 0xDA7a001b254CD22e46d3eAB04d937489c93174C3 |

## Поддерживаемый токен
Ценовые запросы любого номинала доступны при условии, что поддерживаются базовые символы и символы котировки (_основание_/_котировка_). Например:

 - `BTC/USD`
 - `BTC/ETH`
 - `ETH/EUR`

На момент написания статьи список поддерживаемых символов можно найти по [этой ссылке](https://data.bandprotocol.com). Для запроса доступно более 146 пар.

## Запрос цен
Как указывалось ранее, разработчики могут использовать два метода для запроса цен у оракула Band:

 - Смарт-контракт Band на Moonbeam (на данный момент развернут в Moonbase Alpha TestNet)
 - Вспомогательная библиотека Javascript

## Получение данных с помощью смарт-контрактов
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

### Пример контракта

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
 - getMultiPrices:  _view_ fфункция просмотра, которая запрашивает несколько баз. В этом примере цена `BTC` и `ETH`, оба указаны в долларах `USD`
 - savePrice:  _public_ function that queries the _base/quote_ pair. Each element is provided as separate strings, for example `_base = "BTC", _quotes = "USD"`. This sends a transaction and modifies the `price` variable stored in the contract
 - saveMultiPrices:  _public_  function that queries each _base/quote_ pair. Each element is provided as a string array. For example, `_bases = ["BTC","ETH"], _quotes = ["USD","USD"]`. This sends a transaction and modifies the `prices` array stored in the contract, which will hold the price of each pair in the same order as specified in the input

 When deployed, the constructor function needs the Aggregator Contract address for the target network.

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

### Try it in Moonbase Alpha

We've deployed a contract available in the Moonbase Alpha TestNet (at address `0xf15c870344c1c02f5939a5C4926b7cDb90dEc655`) so you can easily check the information fed from Band Protocol's oracle. To do so, you need the following interface contract:

```sol
pragma solidity 0.6.11;
pragma experimental ABIEncoderV2;

interface TestInterface {
    function getPrice(string memory _base, string memory _quote) external view returns (uint256);

    function getMultiPrices(string[] memory _bases, string[] memory _quotes) external view returns (uint256[] memory);
}
```

With it, you will have two view functions available - very similar to our previous examples:

 - getPrice: provides the price feed for a single base/quote pair that is given as input to the function, that is, "BTC", "USD"
 - getMultiPrices: provides the price feed for a multiple base/quote pairs that are given as input to the function, that is, ["BTC", "ETH", "ETH"], ["USD", "USD", "EUR"]

For example, using [Remix](/integrations/remix/), we can easily query the `BTC/USD` price pair using this interface.

After creating the file and compiling the contract, head to the "Deploy and Run Transactions" tab, enter the contract address (`0xf15c870344c1c02f5939a5C4926b7cDb90dEc655`) and click on "At Address." Make sure you have set the "Environment" to "Injected Web3" so you are connected to Moonbase Alpha. 

![Band Protocol Remix deploy](/images/band/band-demo1.png)

This will create an instance of the demo contract that you can interact with. Use the functions `getPrice()` and `getMultiPrices()` to query the data of the corresponding pair.

![Band Protocol Remix check price](/images/band/band-demo2.png)

## BandChain.js Javascript Helper Library

The helper library also supports a similar `getReferenceData` function. To get started, the library needs to be installed:

```
npm install @bandprotocol/bandchain.js
```

The library provides a constructor function that requires an endpoint to point to. This returns an instance that then enables all the necessary methods, such as the `getReferenceData` function.  When querying for information, the function accepts an array where each element is the _base/quote_ pair needed. For example:

```
getReferenceData(['BTC/USD', 'BTC/ETH', 'ETH/EUR'])
```

Then, it returns an array object with the following structure:

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
Where `lastUpdatedBase` and `lastUpdatedQuote` are the last time when the base and quote prices were updated respectively (since UNIX epoch).

### Example Usage

The following Javascript script provides a simple example of the `getReferenceData` function.

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

We can execute this code with a node, and the following `dataQuery` output should look like this:

![Band Protocol JavaScript Library](/images/band/band-console.png)

Note that compared to the request done via smart contracts, the result is given directly in the correct units.
