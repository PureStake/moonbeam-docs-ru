---
title: Обозреватель Блоков
description: Обзор доступных в настоящее время обозревателей блоков, которые можно использовать для навигации по уровням Substrate и Ethereum в Moonbeam TestNet.
---
# Обозреватель Блоков

![Обозреватель Блоков](/images/explorers/explorers-banner.png)

## Введение 

Обозреватель блоков можно рассматривать как поисковые системы для блокчейна. Они позволяют пользователям находить информацию о балансах, контрактах и транзакциях. Более продвинутые "обозреватели" блоков даже предлагают возможности индексации, которые позволяют им предоставить полный набор информации, о токенах в сети ERC20. Они могут даже предлагать API службы для доступа к ним через внешние службы.

В настоящее время мы предлагаем два обозревателя блоков: один для Ethereum API и второй для Substrate API.

!!! Примечание
    Если Вы используете Brave Browser и Ваш проводник не подключается к экземпляру Moonbeam, на который Вы ссылаетесь, попробуйте отключить Brave Shield.

## Ethereum API

### Expedition (Dev Node - TestNet)

Используя эту [ссылку](https://moonbeam-explorer.netlify.app/) Вы можете найти версию [Expedition](https://github.com/etclabscore/expedition).

По умолчанию проводник подключен к сети Moonbase Alpha TestNet. Однако Вы можете подключить его, выполнив следующие шаги:

 1. Нажмите на значок шестеренки в правом верхнем углу.
 2. Выберите пункт "Development", если у Вас есть нода, работающая по следующему адресу `http://localhost:9933` (по умолчанию RPC узел Moonbeam, запущен с опцией `--dev`flag). Вы также можете вернуться в режим "Moonbase Alpha".
 3. В случае если вы хотите подключиться к определенному RPC URL-адресу выберите «Custom RPC» и введите URL-адрес. Например, `http://localhost:9937`

![Обозреватель Expedition](/images/explorers/explorers-images-1.png)

### Blockscout (TestNet)

Blockscout предоставляет пользователям простой в использовании интерфейс для проверки и подтверждения транзакций в блокчейнах EVM, включая Moonbeam. Это дает Вам возможность искать транзакции, просматривать аккаунты, балансы и проверять смарт контракты. Более подробную информацию Вы можете найти в [документации Blockscout](https://docs.blockscout.com/).

Blockscout предлагает следующие основные функции:

 - Разработка с использованием открытого исходного кода, данный код является открытым для изучения и может быть доработан любым членом сообщества. Информацию о коде Вы можете найти [здесь](https://github.com/blockscout/blockscout)
 - Отслеживание транзакций в реальном времени
 - Взаимодействие со смарт-контрактом
 - Поддержка токенов сети ERC20 и ERC721, список всех доступных контрактов для токенов находится в удобном интерфейсе
 - Полнофункциональный API с GraphQL, где пользователи могут тестировать вызовы API прямо из веб-интерфейса

Экземпляр Blockscout, работающий против Moonbase Alpha TestNet, можно найти по [этой ссылке](https://moonbase-blockscout.testnet.moonbeam.network/).

![Обозреватель Blockscout](/images/explorers/explorers-images-2.png)

## Substrate API

### PolkadotJS (Dev Node - TestNet)

Polkadot JS Apps использует конечную точку WebSocket для взаимодействия с сетью. Чтобы подключить его к автономной ноде Moonbeam, Вы можете выполнить действия, описанные [в этом руководстве](/getting-started/local-node/setting-up-a-node/#connecting-polkadot-js-apps-to-a-local-moonbeam-node). Порт по умолчанию - `9944`.

![Polkadot JS Local Node](/images/explorers/explorers-images-3.png)

Для просмотра и взаимодействия с уровнем Substrate Moonbase Alpha, воспользуйтесь следующей [ссылкой](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Fwss.testnet.moonbeam.network#/explorer). Это приложение Polkadot JS, ссылающееся на TestNet. Более подробную информацию Вы можете найти на этой странице](/integrations/wallets/polkadotjs/).

![Polkadot JS Moonbase Alpha](/images/explorers/explorers-images-4.png)

### Subscan

Subscan предоставляет возможности обозревателя блокчейнов для цепочек на основе Substrate. Он способен анализировать стандартные или настраиваемые модули. Например, это полезно для отображения информации, которая касается стейкинга, управления, EVM "модулей". Данный код полностью открыт и его можно найти [здесь](https://github.com/itering/subscan-essentials).

Используя [эту ссылку](https://moonbase.subscan.io/) Вы можете найти экземпляр Subscan, работающий с Moonbase Alpha TestNet.

![Subscan Moonbase Alpha](/images/explorers/explorers-images-5.png)

