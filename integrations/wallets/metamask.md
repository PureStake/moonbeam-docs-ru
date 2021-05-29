---
title: MetaMask
description: В этом руководстве вы узнаете, как подключить MetaMask - кошелек Ethereum на основе браузера, к Moonbeam.
---

# Взаимодействие с Moonbeam при помощи MetaMask

![Вступительная диаграмма](/images/integrations/integrations-metamask-banner.png)

## Вступление

Разработчики могут использовать функции совместимости Moonbeam с Ethereum для интеграции таких инструментов, как [MetaMask](https://metamask.io/), в свои DApps. Таким образом, они могут использовать встроенную библиотеку, которую предоставляет MetaMask, для взаимодействия с блокчейном.

В настоящее время MetaMask можно настроить для подключения к двум сетям: автономной ноде Moonbeam или к Moonbase Alpha TestNet.

Если у вас уже установлен MetaMask, вы можете легко подключить MetaMask к Moonbase Alpha TestNet:

<div class="button-wrapper">
    <a href="#" class="md-button connectMetaMask">Connect MetaMask</a>
</div>

!!! note
    Вы увидите всплывающее окно MetaMask с запросом разрешения на добавление Moonbase Alpha в качестве используемой сети. Как только вы утвердите разрешения, MetaMask переключит вашу текущую сеть на Moonbase Alpha.

Узнайте [как интегрировать кнопку Connect MetaMask](#integrate-metamask-into-a-dapp) в ваши DApps, чтобы пользователи могли подключаться к Moonbase Alpha простым нажатием кнопки.

## Подключение MetaMask к Moonbeam

После того, как вы установили [MetaMask](https://metamask.io/), вы можете подключить его к Moonbeam, нажав на верхний правый логотип, открыв настройки.

![MetaMask3](/images/testnet/testnet-metamask3.png)

Затем перейдите на вкладку Networks и нажмите кнопку "Add Network".

![MetaMask4](/images/testnet/testnet-metamask4.png)

Здесь вы можете настроить MetaMask для следующих сетей:

Автономная нода Moonbeam:

--8<-- 'text/metamask-local/development-node-details.md'

Moonbase Alpha TestNet:

--8<-- 'text/testnet/testnet-details.md'

## Пошаговые инструкции

Если вас интересует более подробное пошаговое руководство, вы можете перейти к нашим подробным руководствам:

 - MetaMask для [автономной ноды Moonbeam](/getting-started/local-node/using-metamask/)
 - MetaMask для [Moonbase Alpha](/getting-started/testnet/metamask/)

## Интегрируйте MetaMask в DApp

С выпуском [Custom Networks API](https://consensys.net/blog/metamask/connect-users-to-layer-2-networks-with-the-metamask-custom-networks-api/) MetaMask, пользователям может быть предложено добавить Moonbeam's Testnet и Moonbase Alpha.

Этот раздел проведет вас через процесс добавления кнопки "Connect to Moonbase Alpha", которая предлагает пользователям подключить свои учетные записи MetaMask к Moonbase Alpha. Вашим пользователям больше не нужно будет знать или беспокоиться о сетевых конфигурациях Moonbase Alpha и добавлении настраиваемой сети в MetaMask. Чтобы взаимодействовать с Moonbeam из вашего dApp, все, что пользователям нужно сделать, это нажать несколько кнопок, чтобы подключиться к Moonbase Alpha и начать работу.
 
MetaMask внедряет глобальный API Ethereum в веб-сайты, которые пользователи посещают в `window.ethereum`, что позволяет веб-сайтам читать и запрашивать данные блокчейна пользователей. Вы будете использовать провайдера Ethereum, чтобы проводить ваших пользователей через процесс добавления Moonbase Alpha в качестве настраиваемой сети. Вам нужно будет:

- Проверить, существует ли провайдер Ethereum и есть ли он в MetaMask
- Запросить адрес учетной записи пользователя
- Добавить Moonbase Alpha в качестве новой цепи

This guide is divided into two sections. First, it'll cover adding a button that will be used to trigger MetaMask to pop-up and connect to Moonbase Alpha. The second part of the guide will create the logic for connecting the user to MetaMask. This way when you click the button you can actually test the functionality as you go through the guide.

### Проверка предварительных условий

To add the Connect MetaMask button you'll need a JavaScript project and the MetaMask browser extension installed for local testing.

It's recommended to use MetaMask's `detect-provider` utility package to detect the provider injected at `window.ethereum`. The package handles detecting the provider for the MetaMask extension and MetaMask Mobile. To install the package in your JavaScript project, run:

```
npm install @metamask/detect-provider
```
### Add a Button

You'll start off by adding a button that will be used to connect MetaMask to Moonbase Alpha. You want to start with the button so when you create the logic in the next step you can test the code as you make your way through the guide. 

The function we will create in the next section of the guide will be called `configureMoonbaseAlpha`. So the button on click should call `configureMoonbaseAlpha`.

```html
<button onClick={configureMoonbaseAlpha()}>Connect to Moonbase Alpha</button>
```

### Add Logic

Now that you have created the button, you need to add the `configureMoonbaseAlpha` function that will be used on click. 

1. Detect the provider at `window.ethereum` and check if it's MetaMask. If you want a simple solution you can directly access `window.ethereum`. Or you can use MetaMask's `detect-provider` package and it will detect the provider for MetaMask extension and MetaMask Mobile for you.
```javascript
import detectEthereumProvider from '@metamask/detect-provider';

const configureMoonbaseAlpha = async () => {
    const provider = await detectEthereumProvider({ mustBeMetaMask: true });
    if (provider) {
        // Logic will go here    
    } else {
        console.error("Please install MetaMask");
    }
}
```

2. Request the user's accounts by calling the `eth_requestAccounts` method. This will prompt MetaMask to pop-up and ask the user to select which accounts they would like to connect to. Behind the scenes, permissions are being checked by calling `wallet_requestPermissions`. Currently the only permissions are for `eth_accounts`. So you're ultimately verifying that you have access to the user's addresses returned from `eth_accounts`. If you're interested in learning more about the permissions system, check out [EIP-2255](https://eips.ethereum.org/EIPS/eip-2255).
```javascript
import detectEthereumProvider from '@metamask/detect-provider';

const configureMoonbaseAlpha = async () => {
    const provider = await detectEthereumProvider({ mustBeMetaMask: true });
    if (provider) {
        try {
            await provider.request({ method: "eth_requestAccounts"});
        } catch(e) {
            console.error(e);
        }  
    } else {
        console.error("Please install MetaMask");
    }
}
```
<img src="/images/integrations/integrations-metamask-1.png" alt="Integrate MetaMask into a Dapp - Select account" style="width: 50%; display: block; margin-left: auto; margin-right: auto;"/>
<img src="/images/integrations/integrations-metamask-2.png" alt="Integrate MetaMask into a Dapp - Connect account" style="width: 50%; display: block; margin-left: auto; margin-right: auto;" />

3. Add Moonbase Alpha as a new chain by calling `wallet_addEthereumChain`. This will prompt the user to provide permission to add Moonbase Alpha as a custom network. 
```javascript
import detectEthereumProvider from '@metamask/detect-provider';

const configureMoonbaseAlpha = async () => {
    const provider = await detectEthereumProvider({ mustBeMetaMask: true });
    if (provider) {
        try {
            await provider.request({ method: "eth_requestAccounts"});
            await provider.request({
                method: "wallet_addEthereumChain",
                params: [
                    {
                        chainId: "0x507", // Moonbase Alpha's chainId is 1287, which is 0x507 in hex
                        chainName: "Moonbase Alpha",
                        nativeCurrency: {
                            name: 'DEV',
                            symbol: 'DEV',
                            decimals: 18
                        },
                       rpcUrls: ["https://rpc.testnet.moonbeam.network"],
                       blockExplorerUrls: ["https://moonbase-blockscout.testnet.moonbeam.network/"]
                    },
                ]
            })
        } catch(e) {
            console.error(e);
        }  
    } else {
        console.error("Please install MetaMask");
    }
}
```

<img src="/images/integrations/integrations-metamask-3.png" alt="Integrate MetaMask into a Dapp - Add network" style="width: 50%; display: block; margin-left: auto; margin-right: auto;"/>

Once the network has been successfully added, it will also prompt the user to then switch to Moonbase Alpha.

<img src="/images/integrations/integrations-metamask-4.png" alt="Integrate MetaMask into a Dapp - Switch to network" style="width: 50%; display: block; margin-left: auto; margin-right: auto;"/>

So, now you should have a button that, on click, walks users through the entire process of connecting their MetaMask accounts to Moonbase Alpha. 

<img src="/images/integrations/integrations-metamask-5.png" alt="Integrate MetaMask into a Dapp - Account connected to Moonbase Alpha"/>

### Confirm Connection

It's possible that you'll have logic that relies on knowing whether a user is connected to Moonbase Alpha or not. Perhaps you want to disable the button if the user is already connected. To confirm a user is connected to Moonbase Alpha, you can call `eth_chainId`, which will return the users current chain ID:

```javascript
    const chainId = await provider.request({
        method: 'eth_chainId'
    })
    // Moonbase Alpha's chainId is 1287, which is 0x507 in hex
    if (chainId === "0x507"){
        // At this point, you might want to disable the "Connect" button
        // or inform the user that they are already connected to the
        // Moonbase Alpha testnet
    }
```

### Listen to Account Changes

To ensure that your project or dApp is staying up to date with the latest account information, you can add the `accountsChanged` event listener that MetaMask provides. MetaMask emits this event when the return value of `eth_accounts` changes. If an address is returned, it is your user's most recent account that provided access permissions. If no address is returned, that means the user has not provided any accounts with access permissions.

```javascript
    provider.on("accountsChanged", (accounts) => {
        if (accounts.length === 0) {
            // MetaMask is locked or the user doesn't have any connected accounts
            console.log('Please connect to MetaMask.');
        } 
    })
```

### Listen to Chain Changes

To keep your project or dApp up to date with any changes to the connected chain, you'll want to subscribe to the `chainChanged` event. MetaMask emits this event every time the connected chain changes.

```javascript
    provider.on("chainChanged", () => {
        // MetaMask recommends reloading the page unless you have good reason not to
        window.location.reload();
    })
```

MetaMask recommends reloading the page whenever the chain changes, unless there is a good reason not to, as it's important to always be in sync with chain changes.
