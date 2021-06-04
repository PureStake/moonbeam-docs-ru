---
title: Moonbase Alpha
description: Обзор текущей конфигурации Moonbeam TestNet, Moonbase Alpha и информация о том, как начать строить на ней с помощью Solidity.
---

# Moonbeam Альфа, Moonbeam TestNet

_Обновленно Апрель 5, 2021_

## Цель

Первая сеть Moonbeam TestNet, получившая название Moonbase Alpha, призвана предоставить разработчикам возможность начать экспериментировать и строить на Moonbeam в общей среде. Поскольку Moonbeam будет развернут как парачейн на Kusama и Polkadot, мы хотим, чтобы наш TestNet отражал нашу рабочую конфигурацию. По этой причине мы решили, что это должна быть конфигурация на основе парачейна, а не отдельная установка Substrate.

Чтобы собрать как можно больше отзывов и быстро решить проблемы, мы создали [Discord с каналом для обсуждения Moonbase AlphaNet.](https://discord.gg/PfpUATX).

## Первоначальная конфигурация

Moonbase Alpha имеет следующую конфигурацию:

 - Moonbeam работает как парачейн, подключенный к цепи передачи
 - Парачейн имеет два сортировщика (организованной PureStake), которые объединяют блоки. Внешние подборщики могут присоединиться к сети. Только верх {{ networks.moonbase.staking.max_collators }} коллаторы по ставке выбираются в активном наборе
 - В релэй чейне размещены три валидатора (размещенные на PureStake) для финализации блоков цепочки ретрансляции. Один из них выбирается для завершения каждого блока, сопоставленного подборщиками Moonbeam. Эта установка дает возможность расширения до конфигурации с двумя парачейнами в будущем.
 - Есть две конечные точки RPC (размещенные на PureStake). Люди могут запускать полные узлы для доступа к своим собственным частным конечным точкам RPC.

![TestNet Diagram](/images/testnet/Moonbase-Alpha-v7.png)

## Функции

Доступны следующие функции:

??? release v1 "_Сентябрь  2020_"
    - Полностью эмулированное производство блоков Ethereum в Substrate (Ethereum pallet)
    - Управляемые функции для взаимодействия с реализацией Rust EVM ([EVM pallet](https://docs.rs/pallet-evm/2.0.1/pallet_evm/))
    - Встроенная поддержка Ethereum RPC (Web3) в субстрате ([Frontier](https://github.com/paritytech/frontier)). Это обеспечивает совместимость с инструментами разработчика Ethereum, такими как MetaMask, Truffle и Remix

??? release v2 "_Октябрь  2020_"
    - Поддержка подписки на события (pub / sub), которая отсутствует на стороне Web3 RPC и обычно используется разработчиками dApp. Вы можете найти руководство о том, как подписаться на события [здесь](/integrations/pubsub/)
    - Поддержка следующих контрактов предварительной компиляции: [ecrecover](https://docs.klaytn.com/smart-contract/precompiled-contracts#address-0x-01-ecrecover-hash-v-r-s), [sha256](https://docs.klaytn.com/smart-contract/precompiled-contracts#address-0x-02-sha-256-data), [ripemd160](https://docs.klaytn.com/smart-contract/precompiled-contracts#address-0x-03-ripemd-160-data) and the [identity function](https://docs.klaytn.com/smart-contract/precompiled-contracts#address-0x-04-datacopy-data) (или datacopy)

??? release v3 "_Ноябрь 2020_"
    - Унификация учетных записей Substrate и Ethereum в формате H160. Эту функцию мы называем [Unified Accounts](https://medium.com/moonbeam-network/moonbase-alpha-v3-introducing-unified-accounts-88fae3564cda). Таким образом, в системе будет только один вид учетной записи, представленный одним адресом.
    - Обновление поддержки подписки на события, добавление возможности использования подстановочных знаков и условного форматирования для тем. Вы можете найти больше информации [здесь](https://docs.moonbeam.network/integrations/pubsub/#using-wildcards-and-conditional-formatting)
    - Приложения Polkadot JS изначально поддерживают адреса H160 и ключи ECDSA. Вы можете использовать свой адрес в стиле Ethereum для функций субстрата (если они доступны), таких как размещение, баланс и управление. Вы можете найти больше информации здесь [здесь](/integrations/wallets/polkadotjs/)

??? release v4 "_Декабрь 2020_"
    - Обновление до самой новой версии протокола парачейна Polkadot ([Parachains V1](https://w3f.github.io/parachain-implementers-guide/)), в котором исправлено несколько проблем с синхронизацией узлов, что открывает путь к синхронизации нескольких коллаторов в одном парачейне
    - Несколько улучшений наших функций совместимости с Ethereum:
        * Идентификатор подписки на событие теперь возвращает идентификатор подписки в стиле Ethereum
        * Исправлены проблемы с оценкой газа для конкретных сценариев использования
        * Добавлена поддержка сообщения о причине возврата
        * Поддержка транзакций Ethereum без ChainId

??? release v5 "_Январь 2021_"      
    - Добавлена пользовательская версия [Staking pallet](https://wiki.polkadot.network/docs/en/learn-staking) (только для целей тестирования и разработки)
    - Добавлена поддержка запросов ожидающих транзакций, пока они находятся в пуле.
    - Исправлены некоторые проблемы при получении прошлых событий и другие мелкие исправления, связанные с событиями смарт-контрактов.
    - Множественные внутренние улучшения, которые включают оптимизацию времени выполнения EVM, что делает его в 15–50 раз быстрее.
    - Поддержка [modexp](https://docs.klaytn.com/smart-contract/precompiled-contracts#address-0x05-bigmodexp-base-exp-mod) контрактов предварительной компиляции

??? release v6 "_Февраль 2021_"      
    - Публичный выпуск пользовательского [Staking pallet](https://wiki.polkadot.network/docs/en/learn-staking). Теперь держатели токенов могут назначать подборщиков и получать вознаграждения.
    - Добавлен [Democracy pallet](https://github.com/paritytech/substrate/tree/HEAD/frame/democracy). TДержатели токенов теперь могут [подавать предложения](/governance/proposals/) и [голосовать за них](/governance/voting/)
    - Обновление до последней версии [Frontier RPC](https://github.com/paritytech/frontier), которая увеличивает эффективность выполнения EVM в 5 раз.
    - Лимит газа был увеличен до 15 миллионов на блок и с лимитом 13 миллионов на транзакцию.

??? release v7 "_Апрель 2021_"      
    - Добавлена поддержка модулей отладки / трассировки Ethereum. По умолчанию они отключены, чтобы использовать их, вам нужно развернуть полный узел и включить эту функцию.
    - Fixed block propagation issues so that is not longer limited to collators, improving network stability
    - Исправлены проблемы с распространением блоков, поэтому они больше не ограничиваются подборщиками, улучшая стабильность сети.
    - Модуль стекинга был переработан с новыми именами, чтобы улучшить взаимодействие с конечным пользователем.
    - Добавлены три новых прекомпилятора: [Bn128Add](https://eips.ethereum.org/EIPS/eip-196), [Bn128Mul](https://eips.ethereum.org/EIPS/eip-196) и [Bn128Pairing](https://eips.ethereum.org/EIPS/eip-197)

### Примечание к релизу

Для получения дополнительных сведений об обновлениях Moonbase Alpha смотрите следующие примечания к релизу:

 - [Moonbase Alpha v2](https://github.com/PureStake/moonbeam/releases/tag/v0.2.0)
 - [Moonbase Alpha v3](https://github.com/PureStake/moonbeam/releases/tag/v0.3.0)
 - [Moonbase Alpha v4](https://github.com/PureStake/moonbeam/releases/tag/v0.4.0)
 - [Moonbase Alpha v5](https://github.com/PureStake/moonbeam/releases/tag/v0.5.0)
 - [Moonbase Alpha v6](https://github.com/PureStake/moonbeam/releases/tag/v0.6.0)
 - [Moonbase Alpha v7](https://github.com/PureStake/moonbeam/releases/tag/v0.7.0)

### Будущие релизы

Возможности, которые могут быть реализованы в будущем:

 - Особенности казначейства ([Treasury pallet](https://github.com/paritytech/substrate/tree/master/frame/treasury))

## Начнём

--8<-- 'text/testnet/connect.md'

## Телеметрия

Вы можете увидеть текущую информацию о телемитрии Moonbase Alpha, перейдя по [этой ссылке](https://telemetry.polkadot.io/#list/Moonbase%20Alpha).

## Токены

--8<-- 'text/testnet/faucet.md'

## Ранняя стадия Proof of Stake

С выпуском Moonbase Alpha v6 TestNet теперь работает с системой Proof of Stake на ранней стадии. Это означает, что в целях тестирования партнерам Moonbeam будет предложено стать первыми подборщиками в сети.

По мере развития Moonbase Alpha мы планируем превратиться в полностью децентрализованную сеть Proof of Stake.

## Ограничения

Это первый TestNet Moonbeam поэтому он содержит некоторые ограничения.

Некоторые [прекомпиляторы](https://docs.klaytn.com/smart-contract/precompiled-contracts) еще не включены в этот выпуск. Вы можете проверить список поддерживаемых прекомпиляций [здесь](/integrations/precompiles/). Однако доступны все встроенные функции.

С момента выпуска Moonbase Alpha v6 максимальный лимит газа на блок был установлен на уровне 15 миллионов, а максимальный лимит газа на транзакцию — 13 миллионов.

Пользователи имеют доступ только к парачейну Moonbeam. В будущих сетях мы добавим доступ к цепочке ретрансляции, чтобы пользователи могли тестировать передачу токенов.

## Очистка сети (Chain Purge)

Данная сеть находится в стадии активного развития. Иногда может потребоваться очистка цепочки, чтобы вернуть цепочку блоков в исходное состояние. Это необходимо при выполнении крупных обновлений или обслуживания TestNet. Мы сообщим, когда произойдет чистка цепочки, на нашем канале в [Discord](https://discord.gg/PfpUATX) не менее чем за 24 часа.

Обратите внимание, что PureStake не будет переносить состояние цепочки. Таким образом, все данные, хранящиеся в цепочке блоков, будут потеряны при выполнении очистки цепочки. Однако, поскольку предела по газу нет, пользователи могут легко восстановить состояние до очистки.

