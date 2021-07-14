### HTTPS DNS

Чтобы подключиться к Moonbase Alpha через HTTPS просто укажите своему провайдеру следующий RPC DNS:

```
https://rpc.testnet.moonbeam.network
```

Для библиотеки web3.js Вы можете создать локальный экземпляр Web3 и настроить провайдера для подключения к Moonbase Alpha (поддерживаются как HTTP, так и WS):

```js
const Web3 = require('web3'); //Load Web3 library
.
.
.
//Create local Web3 instance - set Moonbase Alpha as provider
const web3 = new Web3('https://rpc.testnet.moonbeam.network'); 
```
Для библиотеки ethers.js определите поставщика, используя `ethers.providers.StaticJsonRpcProvider(providerURL, {object})` и задав URL-адрес поставщика Moonbase Alpha:

```js
const ethers = require('ethers');


const providerURL = 'https://rpc.testnet.moonbeam.network';
// Define Provider
const provider = new ethers.providers.StaticJsonRpcProvider(providerURL, {
    chainId: 1287,
    name: 'moonbase-alphanet'
});
```

Любой кошелек Ethereum должен иметь возможность генерировать действительный адрес для Moonbeam (например, [MetaMask](https://metamask.io/)).

### WSS DNS

Для подключений через WebSocket вы можете использовать следующий DNS:

```
wss://wss.testnet.moonbeam.network
```

### Идентификатор цепи

Для Moonbase Alpha TestNet идентификатор цепи:  `1287`.

### Relay Chain

Для подключения к Relay Chain Moonbase Alpha, управляемой PureStake, можно использовать следующую конечную точку WS Endpoint:

```
wss://wss-relay.testnet.moonbeam.network
```
