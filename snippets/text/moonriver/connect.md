### HTTPS DNS {: #https-dns } 

Чтобы подключиться к Moonriver через HTTPS просто укажите своему провайдеру следующий RPC DNS:

```
https://rpc.moonriver.moonbeam.network
```

Для библиотеки web3.js Вы можете создать локальный экземпляр Web3 и настроить провайдера для подключения к Moonriver (поддерживаются как HTTP, так и WS):

```js
const Web3 = require('web3'); //Load Web3 library
.
.
.
//Create local Web3 instance - set Moonriver as provider
const web3 = new Web3("https://rpc.moonriver.moonbeam.network"); 
```
Для библиотеки ethers.js определите поставщика, используя `ethers.providers.StaticJsonRpcProvider(providerURL, {object})` и задав URL-адрес поставщика Moonriver:

```js
const ethers = require('ethers');


const providerURL = "https://rpc.moonriver.moonbeam.network";
// Define Provider
const provider = new ethers.providers.StaticJsonRpcProvider(providerURL, {
    chainId: 1285,
    name: 'moonriver'
});
```

Любой кошелек Ethereum должен иметь возможность генерировать действительный адрес для Moonbeam (например, [MetaMask](https://metamask.io/)).

### WSS DNS {: #wss-dns } 

Для подключений через WebSocket вы можете использовать следующий DNS:

```
wss://wss.moonriver.moonbeam.network
```

### Chain ID {: #chain-id } 

Для Moonriver идентификатор цепи: `1285`
