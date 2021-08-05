---
title: Moonriver
description: Обзор текущей конфигурации внедрения Moonbeam на Kusama, Moonriver и информация о том, как начать создавать на ее основе, используя Solidity.
---

# Moonriver

_Обновлено 9 июля 2021 года_

## Цель {: #goal } 

В июне 2021 года Moonriver впервые был запущен как парачейн в сети Kusama. Moonriver является дочерней сетью Moonbeam и предоставляет среду для тестирования нового кода в реальных экономических условиях. Теперь у разработчиков есть возможность начать экспериментировать и создавать в стимулируемой канареечной сети, подключенной к Kusama.

Чтобы собрать как можно больше отзывов и обеспечить быстрое решение вопросов, мы создали [Discord с выделенным каналом Moonriver](https://discord.gg/5TaUvbRvgM).

## Первоначальные конфигурации {: #initial-configurations } 

Moonriver is scheduled to follow a [5-phase launch process](https://moonbeam.network/networks/moonriver/launch/). Currently, Moonriver is in Phase 3 of the launch process and has the following configurations:

- Работает как парачейн, подключенный к Relay Chain Кусамы.
- Has an active set of {{ networks.moonriver.staking.max_collators }} collators
- Есть две конечные точки RPC (размещенные PureStake). Люди могут запускать полные ноды для доступа к своим собственным частным конечным точкам RPC

![Moonriver Diagram](/images/moonriver/moonriver-diagram.png)

Некоторые важные переменные/конфигурации, на которые следует обратить внимание, включают:

=== "Общие"
    |       Переменные        |                                               Значение                                           |
    |:---------------------:|    :-----------------------------------------------------------------------------------------:|
    |   Минимальная цена за газ   | {{ networks.moonriver.min_gas_price }} Gsed*  |
    |   Время  до целевого блока   |          {{ networks.moonriver.block_time }} секунд  (ожидается 6          секунд)           |
    |    Лимит газа для блока    |         {{ networks.moonriver.gas_block }} (ожидается увеличение по меньшей мере в    4 раза)          |
    | Лимит газа на транзакцию |           {{ networks.moonriver.gas_tx }} (ожидается увеличение по меньшей мере в      4 раза)           |
    |     RPC конечная точка      |                             {{ networks.moonriver.rpc_url }}    }                              |
    |     WSS конечная точка      |                             {{ networks.moonriver.wss_url }}                              |

=== "Управление"
    |         Переменные         |                                                                  Значение                                                              |
    |:------------------------:|    :---------------------------------------------------------------------------------------------------    ----------------------------:|
    |      Период Голосования       |      {{ networks.moonriver.democracy.vote_period.blocks}} блоки ({{     networks.moonriver.democracy.vote_period.days}} days)      |
    | Ускоренный период голосования | {{ networks.moonriver.democracy.fast_vote_period.blocks}} блоки ({{     networks.moonriver.democracy.fast_vote_period.days}} days) |
    |     Период вступления в силу    |     {{ networks.moonriver.democracy.enact_period.blocks}} блоки ({{     networks.moonriver.democracy.enact_period.days}} days)      |
    |     Период ожидания      |      {{ networks.moonriver.democracy.cool_period.blocks}} блоки ({{     networks.moonriver.democracy.cool_period.days}} days)      |
    |     Минимальный Депозит      |                                       {{ networks.moonriver.democracy.    min_deposit }} MOVR                                       |
    |      Максимум Голосов       |                                          {{ networks.moonriver.    democracy.max_votes }}                                           |
    |    Муксимум предложений     |                                        {{ networks.moonriver.democracy.    max_proposals }}                                         |

=== "Стекинг"
    |             Переменные             |                                                       Значения                                                   |
    |:--------------------------------:|    :---------------------------------------------------------------------------------------------------    ------:|
    |     Минимальная доля стекинга     |                           {{ networks.moonriver.staking.    min_nom_stake }} токенов                           |
    |       Минимальная номинация        |                           {{ networks.moonriver.staking.    min_nom_amount}} токенов                           |
    | Максимум номинаций на коллатора |                             {{ networks.moonriver.staking.    max_nom_per_col }}                              |
    | Максимальное количество коллаторов на одного номинанта  |                             {{ networks.moonriver.staking.    max_col_per_nom }}                              |
    |              Раунд               | {{ networks.moonriver.staking.round_blocks }} blocks ({{     networks.moonriver.staking.round_hours }} hours) |
    |          Продолжительность размещения бондов           |                             {{ networks.moonriver.staking.    bond_lock }} раундов                             |

_*Подробнее о [деноминации токенов](#_6)_

## Начало работы {: #get-started } 

--8<-- 'text/moonriver/connect.md'

## Телеметрия {: #telemetry } 

Текущую информацию о телеметрии Moonriver можно посмотреть по [этой ссылке] (https://telemetry.polkadot.io/#list/Moonriver).

## Токены {: #tokens } 

Токены на Moonriver также будут называться Moonriver (MOVR). Посетите сайт Moonbeam Foundation для получения дополнительной информации о [токене Moonriver](https://moonbeam.foundation/moonriver-token/). 

### Номиналы токенов {: #token-denominations } 

Самая маленькая единица Moonriver называется Sediment (Sed). Для получения одного Moonriver требуется 10^18 Sediment. Номиналы следующие:

|      Еденица      |   Moonriver (MOVR)   |        Sediment (Sed)         |
|:--------------:|:--------------------:|:-----------------------------:|
| Sediment (Sed) | 0.000000000000000001 |               1               |
|    Kilosed     |  0.000000000000001   |             1,000             |
|    Megased     |    0.000000000001    |           1,000,000           |
|    Gigased     |     0.000000001      |         1,000,000,000         |
| Micromoonriver |       0.000001       |       1,000,000,000,000       |
| Millimoonriver |        0.001         |     1,000,000,000,000,000     |
|   Moonriver    |          1           |   1,000,000,000,000,000,000   |
| Kilomoonriver  |        1,000         | 1,000,000,000,000,000,000,000 |

## Proof of Stake {: #proof-of-stake } 

В течение 5 этапов запуска Moonriver сеть будет постепенно обновляться до полностью децентрализованной сети Proof of Stake. Подробную информацию о том, что будет происходить на каждом этапе, можно найти на странице [Статус запуска сети](https://moonbeam.network/networks/moonriver/launch/).

In Phase 1, there was an initial collator election to fill the active collator set with parties outside of the Moonbeam team. The number of collators in the active set will be subject to governance. The active set will consist of the top collators by stake, including nominations.

## Ограничения {: #limitations } 

Некоторые [прекомпиляции](https://docs.klaytn.com/smart-contract/precompiled-contracts) еще не включены. Список поддерживаемых прекомпиляций можно посмотреть [здесь](/integrations/precompiles/). Тем не менее, все встроенные функции доступны.

