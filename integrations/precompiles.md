---
title: Предварительно скомпилированные контракты
description:  Узнайте, как использовать предварительно скомпилированные контракты на Moonbase Alpha, в сети Moonbeam Network TestNet, которая уникальна своей полной совместимостью с Ethereum.
---

# Предварительно скомпилированные контракты на Moonbase Alpha

## Вступление

Еще одна функция, добавленная в [релиз Moonbase Alpha v2](https://moonbeam.network/announcements/moonbase-alpha-v2-contract-events-pub-sub-capabilities/) это включение некоторых [предварительно скомпилированных контрактов](https://docs.klaytn.com/smart-contract/precompiled-contracts) которые изначально доступны в Ethereum.

В настоящее время включено пять предварительно скомпилированных функций: ecrecover, sha256, ripemd-160, функция идентичности, и модульное возведение в степень.

В этом руководстве мы объясним, как использовать и/или проверять эти предварительно скомпилированные функции.

## Проверка предварительных условий

--8<-- 'text/common/install-nodejs.md'

На момент написания этого руководства использовались версии 15.2.1 и 7.0.8 соответственно. Нам также потребуется установить пакет Web3, выполнив:

```
npm install --save web3
```

Чтобы узнать установленную версию Web3, Вы можете использовать команду ls:

```
npm ls web3
```
На момент написания этого руководства использовалась версия 1.3.0. Мы также будем использовать [Remix](/integrations/remix/), подключив его к Moonbase Alpha TestNet через [MetaMask](/integrations/wallets/metamask/).

## Проверка подписи с помощью ECRECOVER

Основная задача этой предварительно скомплированной функции - проверить подпись сообщения. В общих чертах, Вы подставляете `ecrecover` значения подписи транзакции, и она возвращает адрес. Подпись проверяется, если возвращаемый адрес совпадает с публичным адресом, по которому была отправлена транзакция.

Давайте перейдем к небольшому примеру, чтобы продемонстрировать, как использовать эту предварительно скомпилированную функцию. Для этого нам нужно получить значения подписи транзакции (v, r, s). Поэтому мы выполним подпись и получим подписанное сообщение, в котором есть следующие значения:
```solidity
const Web3 = require('web3');

// Provider
const web3 = new Web3('https://rpc.testnet.moonbeam.network');

// Address and Private Key
const address = '0x6Be02d1d3665660d22FF9624b7BE0551ee1Ac91b';
const pk1 = '99B3C12287537E38C90A9219D4CB074A89A16E9CDB20BF85728EBD97C343E342';
const msg = web3.utils.sha3('supercalifragilisticexpialidocious');

async function signMessage(pk) {
   try {
   // Sign and get Signed Message
      const smsg = await web3.eth.accounts.sign(msg, pk);
      console.log(smsg);
   } catch (error) {
      console.error(error);
   }
}

signMessage(pk1);
```

Этот код вернет на экран следующий объект:

```js
{
  message: '0xc2ae6711c7a897c75140343cde1cbdba96ebbd756f5914fde5c12fadf002ec97',
  messageHash: '0xc51dac836bc7841a01c4b631fa620904fc8724d7f9f1d3c420f0e02adf229d50',
  v: '0x1b',
  r: '0x44287513919034a471a7dc2b2ed121f95984ae23b20f9637ba8dff471b6719ef',
  s: '0x7d7dc30309a3baffbfd9342b97d0e804092c0aeb5821319aa732bc09146eafb4',
  signature: '0x44287513919034a471a7dc2b2ed121f95984ae23b20f9637ba8dff471b6719ef7d7dc30309a3baffbfd9342b97d0e804092c0aeb5821319aa732bc09146eafb41b'
}
```
С необходимыми значениями мы можем перейти в Remix, чтобы протестировать предварительно скомпилированный контракт.Обратите внимание, что это также можно проверить с помощью библиотеки Web3 JS, но в нашем случае мы перейдем к Remix, чтобы убедиться, что он использует предварительно скомпилированный контракт на блокчейне. Код Solidity, который мы можем использовать для проверки подписи, имеет следующий вид:

```solidity
pragma solidity ^0.7.0;

contract ECRECOVER{
    address addressTest = 0x12Cb274aAD8251C875c0bf6872b67d9983E53fDd;
    bytes32 msgHash = 0xc51dac836bc7841a01c4b631fa620904fc8724d7f9f1d3c420f0e02adf229d50;
    uint8 v = 0x1b;
    bytes32 r = 0x44287513919034a471a7dc2b2ed121f95984ae23b20f9637ba8dff471b6719ef;
    bytes32 s = 0x7d7dc30309a3baffbfd9342b97d0e804092c0aeb5821319aa732bc09146eafb4;
    
    
    function verify() public view returns(bool) {
        // Use ECRECOVER to verify address
        return (ecrecover(msgHash, v, r, s) == (addressTest));
    }
}
```

Используя [Remix компилятор и его равёзртывание](/getting-started/local-node/using-remix/) с [MetaMask указывающим на Moonbase Alpha](/getting-started/testnet/metamask/), мы можем развернуть контракт и вызвать метод `verify()` который возвращает значение _true_ если адрес, возвращаемый функцией `ecrecover` равен адресу, используемому для подписи сообщения (связан с закрытым ключом и должен быть вручную установлен в контракте).

## Хеширование с помощью SHA256

Эта функция хеширования возвращает хэш SHA256 из заданных данных. Чтобы протестировать данную прекомпиляцию, Вы можете использовать эту [онлайн утилиту](https://md5calc.com/hash/sha256) для вычисления хэша SHA256 любой строки, которая Вам необходима. В нашем случае мы сделаем это с помощью `Hello World!`. Мы можем перейти непосредственно к Remix и развернуть следующий код, в котором вычисляемый хэш будет установлен для переменной `expectedHash`:

```solidity
pragma solidity ^0.7.0;

contract Hash256{
    bytes32 public expectedHash = 0x7f83b1657ff1fc53b92dc18148a1d65dfc2d4b1fa3d677284addd200126d9069;

    function calculateHash() internal pure returns (bytes32) {
        string memory word = 'Hello World!';
        bytes32 hash = sha256(bytes (word));
        
        return hash;        
    }
    
    function checkHash() public view returns(bool) {
        return (calculateHash() == expectedHash);
    }
}

```
После развертывания контракта мы можем вызвать `checkHash()` метод который вернёт _true_ значение если хэш, возвращаемый `calculateHash ()`, равен предоставленному хешу.

## Хеширование с помощью RIPEMD-160

Эта функция хеширования возвращает хэш RIPEMD-160 из указываемых данных. Чтобы протестировать данную прекомпиляцию, Вы можете воспользоватся следующей [онлайн утилитой](https://md5calc.com/hash/ripemd160) для вычисления хэша любой строки при помощи RIPEMD-160 функции. В нашем случае снова будет использоваться код `Hello World!`. Мы будем повторно использовать тот же код, что и раньше, но использовать функцию `ripemd160`. Обратите внимание, что он возвращает переменную типа `bytes20`:

```solidity
pragma solidity ^0.7.0;

contract HashRipmd160{
    bytes20 public expectedHash = hex'8476ee4631b9b30ac2754b0ee0c47e161d3f724c';

    function calculateHash() internal pure returns (bytes20) {
        string memory word = 'Hello World!';
        bytes20 hash = ripemd160(bytes (word));
        
        return hash;        
    }
    
    function checkHash() public view returns(bool) {
        return (calculateHash() == expectedHash);
    }
}
```
После развертывания контракта мы можем вызвать `checkHash()` метод, который возвращает _true_, если хэш, возвращаемый `calculateHash ()`, равен предоставленному хешу.

## Функция идентичности

Эта функция, также известная как datacopy, служит более дешевым способом копирования данных в память. Компилятор Solidity не поддерживает его, поэтому его нужно вызывать с помощью готового бинарного файла. [Следующий код](https://docs.klaytn.com/smart-contract/precompiled-contracts#address-0x-04-datacopy-data) (адаптированный к Solidity), можно использовать для вызова этого предварительно скомпилированного контракта. Мы можем воспользоваться этой [онлайн утилитой](https://web3-type-converter.onbrn.com/) чтобы получить байты из любой строки, так как мы используем `callDataCopy()` метод.

```solidity
pragma solidity ^0.7.0;

contract Identity{
    
    bytes public memoryStored;

    function callDatacopy(bytes memory data) public returns (bytes memory) {
    bytes memory result = new bytes(data.length);
    assembly {
        let len := mload(data)
        if iszero(call(gas(), 0x04, 0, add(data, 0x20), len, add(result,0x20), len)) {
            invalid()
        }
    }
    
    memoryStored = result;

    return result;
    }
}
```
После развертывания контракта мы можем вызвать метод `callDataCopy()` и проверить, соответствует ли `memoryStored` байтам, которые Вы передаете в качестве входных данных функции.

## Модульное возведение в степень

Этот пятая предварительно скомпилированя функция которая вычисляет остаток, когда целое число _b_ (основание) возводится в _e_(ю) степень (показатель степени) и делится на положительное целое число _m_ (модуль).

Компилятор Solidity не поддерживает его, поэтому его нужно вызывать ввиде готового бинарного файла. [Следующий код](https://docs.klaytn.com/smart-contract/precompiled-contracts#address-0x05-bigmodexp-base-exp-mod) был упрощен, чтобы показать работу данной функции.

```solidity
pragma solidity ^0.7.0;

contract ModularCheck {

    uint public checkResult;

    // Function to Verify ModExp Result
    function verify( uint _base, uint _exp, uint _modulus) public {
        checkResult = modExp(_base, _exp, _modulus);
    }

    function modExp(uint256 _b, uint256 _e, uint256 _m) public returns (uint256 result) {
        assembly {
            // Free memory pointer
            let pointer := mload(0x40)
            // Define length of base, exponent and modulus. 0x20 == 32 bytes
            mstore(pointer, 0x20)
            mstore(add(pointer, 0x20), 0x20)
            mstore(add(pointer, 0x40), 0x20)
            // Define variables base, exponent and modulus
            mstore(add(pointer, 0x60), _b)
            mstore(add(pointer, 0x80), _e)
            mstore(add(pointer, 0xa0), _m)
            // Store the result
            let value := mload(0xc0)
            // Call the precompiled contract 0x05 = bigModExp
            if iszero(call(not(0), 0x05, 0, pointer, 0xc0, value, 0x20)) {
                revert(0, 0)
            }
            result := mload(value)
        }
    }
}
```

Вы можете попробовать это в [Remix](/integrations/remix/). Используйте функцию `verify ()`, передав в неё основание, показатель степени и модуль. Функция сохранит значение в переменной `checkResult`.

