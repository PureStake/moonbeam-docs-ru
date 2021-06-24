---
title: MetaMask
description: В этом руководстве Вы узнаете, как подключить MetaMask - кошелек Ethereum на основе браузера, к Moonbeam.
---

# Взаимодействие с Moonbeam при помощи MetaMask

![Вступительная диаграмма](/images/integrations/integrations-metamask-banner.png)

## Вступление

Разработчики могут использовать функции совместимости Moonbeam с Ethereum для интеграции таких инструментов, как [MetaMask](https://metamask.io/), в свои DApps. Таким образом, они могут использовать встроенную библиотеку, которую предоставляет MetaMask, для взаимодействия с блокчейном.

В настоящее время MetaMask можно настроить для подключения к двум сетям: автономной ноде Moonbeam или к Moonbase Alpha TestNet.

Если у Вас уже установлен MetaMask, Вы можете легко подключить MetaMask к Moonbase Alpha TestNet:

<div class="button-wrapper">
    <a href="#" class="md-button connectMetaMask">Connect MetaMask</a>
</div>

!!! примечание 
    Вы увидите всплывающее окно MetaMask с запросом разрешения на добавление Moonbase Alpha в качестве настраиваемой сети. Как только вы утвердите разрешения, MetaMask переключит Вашу текущую сеть на Moonbase Alpha.

Узнайте [как интегрировать кнопку Connect MetaMask](#integrate-metamask-into-a-dapp) в Ваши DApps, чтобы пользователи могли подключаться к Moonbase Alpha простым нажатием кнопки.

## Подключение MetaMask к Moonbeam

После того, как Вы установили [MetaMask](https://metamask.io/), Вы можете подключить его к Moonbeam, нажав на верхний правый логотип, открыв настройки.

![MetaMask3](/images/testnet/testnet-metamask3.png)

Затем перейдите на вкладку Networks и нажмите кнопку "Add Network".

![MetaMask4](/images/testnet/testnet-metamask4.png)

Здесь Вы можете настроить MetaMask для следующих сетей:

Автономная нода Moonbeam:

--8<-- 'text/metamask-local/development-node-details.md'

Moonbase Alpha TestNet:

--8<-- 'text/testnet/testnet-details.md'

## Пошаговые инструкции

Если Вас интересует более подробное пошаговое руководство, Вы можете перейти к нашим подробным руководствам:

 - MetaMask для [автономной ноды Moonbeam](/getting-started/local-node/using-metamask/)
 - MetaMask для [Moonbase Alpha](/getting-started/moonbase/metamask/)

## Интегрируйте MetaMask в DApp

С выпуском [MetaMask Custom Networks API](https://consensys.net/blog/metamask/connect-users-to-layer-2-networks-with-the-metamask-custom-networks-api/), пользователям может быть предложено добавить Moonbeam's Testnet и Moonbase Alpha.

Этот раздел проведет Вас через процесс добавления кнопки "Connect to Moonbase Alpha", которая предлагает пользователям подключить свои учетные записи MetaMask к Moonbase Alpha. Вашим пользователям больше не нужно будет знать или беспокоиться о сетевых конфигурациях Moonbase Alpha и добавлении настраиваемой сети в MetaMask. Чтобы взаимодействовать с Moonbeam из Вашего dApp, все, что пользователям нужно сделать, это нажать несколько кнопок, чтобы подключиться к Moonbase Alpha и начать работу.
 
MetaMask внедряет глобальный API Ethereum в веб-сайты, которые пользователи посещают в `window.ethereum`, что позволяет веб-сайтам читать и запрашивать данные блокчейна пользователей. Вы будете использовать провайдера Ethereum, чтобы провести Ваших пользователей через процесс добавления Moonbase Alpha в качестве настраиваемой сети. Вам нужно будет:

- Проверить, существует ли провайдер Ethereum и есть ли он в MetaMask
- Запросить адрес учетной записи пользователя
- Добавить Moonbase Alpha в качестве новой цепочки

Это руководство разделено на два раздела. Первая часть руководства объяснит процесс добавление кнопки, которая будет использоваться для запуска MetaMask для всплывающего окна и подключения к Moonbase Alpha. Вторая часть руководства создаст логику подключения пользователя к MetaMask. Таким образом, при нажатии на кнопку, Вы можете фактически протестировать функциональность,  следуя руководству.

### Проверка предварительных условий

Чтобы добавить кнопку Connect MetaMask, Вам понадобится проект JavaScript и расширение браузера MetaMask, установленное для локального тестирования.

Рекомендуется использовать служебный пакет MetaMask `detect-provider` для обнаружения провайдера, внедренного в `window.ethereum`. Пакет обрабатывает обнаружение провайдера для расширения MetaMask и MetaMask Mobile. Чтобы установить пакет в свой проект JavaScript, запустите:

```
npm install @metamask/detect-provider
```
### Добавить кнопку

Вы начнете с добавления кнопки, которая будет использоваться для подключения MetaMask к Moonbase Alpha. Мы начнем с добавления кнопки, чтобы при создании логики на следующем шаге можно было протестировать код по мере продвижения по руководству.

Функция, которую мы создадим в следующем разделе руководства, будет называться `configureMoonbaseAlpha`. Таким образом, кнопка при нажатии должна вызывать `configureMoonbaseAlpha`.

```html
<button onClick={configureMoonbaseAlpha()}>Connect to Moonbase Alpha</button>
```

### Создать логику

Теперь, когда Вы создали кнопку, Вам нужно добавить функцию `configureMoonbaseAlpha` которая будет использоваться при нажатии.

1. Найдите провайдера в `window.ethereum` и проверьте, есть ли он MetaMask. Если Вы предпочитаете простое решение, Вы можете получить прямой доступ к `window.ethereum`. Или вы можете использовать пакет MetaMask `detect-provider` и он найдет провайдера для расширения MetaMask и MetaMask Mobile за вас.
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

2. Запросите учетные записи пользователя, вызвав метод `eth_requestAccounts`. Это вызовет всплывающее окно MetaMask и попросит пользователя выбрать, к каким учетным записям он хотел бы подключиться. В бэкенде, разрешения проверяются вызовом `wallet_requestPermissions`. В настоящее время разрешения доступны только для `eth_accounts`. В конечном итоге Вы подтверждаете, что у Вас есть доступ к адресам пользователя, возвращаемым из `eth_accounts`. Если Вы хотите узнать больше о политике разрешений, ознакомьтесь с [EIP-2255] (https://eips.ethereum.org/EIPS/eip-2255).
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

3. Добавьте Moonbase Alpha в качестве новой цепочки, с помощью вызова `wallet_addEthereumChain`. Это предложит пользователю предоставить разрешение на добавление Moonbase Alpha в качестве настраиваемой сети.
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

После успешного добавления сети, пользователю также будет предложено переключиться на Moonbase Alpha. 

<img src="/images/integrations/integrations-metamask-4.png" alt="Integrate MetaMask into a Dapp - Switch to network" style="width: 50%; display: block; margin-left: auto; margin-right: auto;"/>

Итак, теперь у Вас должна быть кнопка, которая при нажатии проводит пользователей через весь процесс подключения их учетных записей MetaMask к Moonbase Alpha.

<img src="/images/integrations/integrations-metamask-5.png" alt="Integrate MetaMask into a Dapp - Account connected to Moonbase Alpha"/>

### Подтвердить соединение

Вполне возможно, у Вас будет логика, основанная на знании того, подключен ли пользователь к Moonbase Alpha или нет. Возможно, Вы захотите отключить кнопку, если пользователь уже подключен. Чтобы подтвердить, что пользователь подключен к Moonbase Alpha, Вы можете вызвать метод `eth_chainId`, который вернет текущий идентификатор цепочки пользователя:

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

### Следить за изменениями учетных записей

Чтобы быть увереннными в том, что Ваш проект или приложение постоянно обновляются с последней информацией об учетной записи, Вы можете добавить прослушиватель событий `accountsChanged`, который предоставляет MetaMask. MetaMask генерирует это событие при изменении возвращаемого значения `eth_accounts`. Если адрес возвращается, это означает, что права доступа предоставлены последней учетной записью Вашего пользователя. Если адрес не возвращается, это означает, что пользователь не предоставил ни одной учетной записи с разрешениями на доступ.

```javascript
    provider.on("accountsChanged", (accounts) => {
        if (accounts.length === 0) {
            // MetaMask is locked or the user doesn't have any connected accounts
            console.log('Please connect to MetaMask.');
        } 
    })
```

### Следить за изменениями цепочки

Чтобы Ваш проект или dApp всегда был в курсе любых изменений в подключенной цепочке, Вам нужно подписаться на `chainChanged`. MetaMask генерирует это событие каждый раз при изменении подключенной цепочки.

```javascript
    provider.on("chainChanged", () => {
        // MetaMask recommends reloading the page unless you have good reason not to
        window.location.reload();
    })
```

MetaMask рекомендует перезагружать страницу при каждом изменении цепочки, если нет веской причины не делать этого, чтобы всегда быть синхронизированным с изменениями цепочки.

