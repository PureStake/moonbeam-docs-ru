---
title: Коллаторы
description: Инструкции о том, как стать коллатором в сети Moonbeam Network после того, как вы запустите ноду
---

# Запуск коллатора на Moonbeam

![Collator Moonbeam Banner](/images/fullnode/collator-banner.png)

## Введение {: #introduction } 

Коллаторы - это члены сети, которые поддерживают парачейны, в которых они участвуют. Они запускают полную ноду (как для своего конкретного парачайна, так и для relay chain) и производят доказательство перехода состояния для валидаторов relay chain.

Users can spin up full nodes on Moonbase Alpha and Moonriver and activate the `collate` feature to participate in the ecosystem as collators.

Moonbeam использует [Nimbus Parachain Consensus Framework](/learn/consensus/). Это обеспечивает двухступенчатый фильтр для распределения коллаторов в слот производства блока:
 - The parachain staking filter selects the top {{ networks.moonbase.staking.max_collators }} collators on Moonbase Alpha and the top {{ networks.moonriver.staking.max_collators }} collators on Moonriver in terms of tokens staked in each network. This filtered pool is called selected candidates, and selected candidates are rotated every round
 - Фильтр подмножества фиксированного размера выбирает псевдослучайное подмножество ранее выбранных кандидатов для каждого слота добычи блока
 
Это руководство поможет вам выполнить следующие шаги:

 - **[Технические требования](#technical-requirements)** - показывает критерии, которым вы должны соответствовать с технической точки зрения
 - **[Учетные записи и требования к стекингу](#accounts-and-staking-requirements)** - описывается процесс создания учетной записи и получения токенов для участия в качестве кандидата в коллаторы.
 - **[Генерация сессионных ключей](#generate-session-keys)** - объясняется, как генерировать сессионные ключи, используемые для сопоставления вашего ID автора с вашей учетной записью H160
 - **[ID автора сопоставимого с вашим аккаунтом](#map-author-id-to-your-account)** - описывает шаги по сопоставлению вашего публичного сессионного ключа с вашим аккаунтом H160, на который будут выплачиваться вознаграждения за блокчейн


## Технические требования {: #technical-requirements } 

С технической точки зрения коллаторы должны отвечать следующим требованиям:

 - Иметь полную ноду, работающую с опциями коллатора. Для этого следуйте руководству по созданию полной ноды [руководство по созданию полной ноды] (/node-operators/networks/full-node/), учитывая специфические фрагменты кода для коллаторов
 - Включите сервер телеметрии для вашего полной ноды. Для этого следуйте руководству [руководство по телеметрии](/node-operators/networks/telemetry/)

## Требования к учетной записи и стейкингу {: #accounts-and-staking-requirements } 

Similar to Polkadot validators, you need to create an account. For Moonbeam, this is an H160 account or basically an Ethereum style account from which you hold the private keys. In addition, you will need a minimum amount of tokens staked to be considered eligible (become a candidate). Only a certain amount of the top collators by nominated stake will be in the active set.

=== "Moonbase Alpha"
    |    Variable     |                          Value                          |
    |:---------------:|:-------------------------------------------------------:|
    |   Bond Amount   | {{ networks.moonbase.staking.collator_bond_min }} DEV   |
    | Active set size | {{ networks.moonbase.staking.max_collators }} collators |

=== "Moonriver"
    |    Variable     |                          Value                           |
    |:---------------:|:--------------------------------------------------------:|
    |   Bond Amount   | {{ networks.moonriver.staking.collator_bond_min }} MOVR  |
    | Active set size | {{ networks.moonriver.staking.max_collators }} collators |

### Учетная запись PolkadotJS {: #account-in-polkadotjs } 

У коллатора есть учетная запись, связанная с его деятельностью. Этот аккаунт сопоставляется с ID автора для идентификации его как производителя блоков и отправки выплат от вознаграждений за блоки. 

В настоящее время у вас есть два способа действий в отношении наличия учетной записи в [PolkadotJS](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Fwss.testnet.moonbeam.network#/accounts):

 - Импортировать существующий (или создать новый) аккаунт H160 из внешних кошельков или сервисов, таких как [MetaMask](/integrations/wallets/metamask/) и [MathWallet](/integrations/wallets/mathwallet/).
 - Создайте новый аккаунт H160 с помощью [PolkadotJS](/integrations/wallets/polkadotjs/)

Как только вы импортировали аккаунт H160 в PolkadotJS, вы должны увидеть его на вкладке "Accounts". Убедитесь, что у вас под рукой есть публичный адрес (`PUBLIC_KEY`), так как он необходим для настройки [deploy your full node](/node-operators/networks/full-node/) с параметрами collation.

![Account in PolkadotJS](/images/fullnode/collator-polkadotjs1.png)

## Стать кандидатом в коллаторы {: #become-a-collator-candidate } 

Before getting started, it's important to note some of the timings of different actions related to collation activities:

=== "Moonbase Alpha"
    |               Variable                |       Value        |
    |:-------------------------------------:|:------------------:|
    |    Join/leave collator candidates     | {{ networks.moonbase.collator_timings.join_leave_candidates.rounds }} rounds ({{ networks.moonbase.collator_timings.join_leave_candidates.hours }} hours) |
    |        Add/remove nominations         | {{ networks.moonbase.collator_timings.add_remove_nominations.rounds }} rounds ({{ networks.moonbase.collator_timings.add_remove_nominations.hours }} hours) |
    | Rewards payouts (after current round) | {{ networks.moonbase.collator_timings.rewards_payouts.rounds }} rounds ({{ networks.moonbase.collator_timings.rewards_payouts.hours }} hours) |

=== "Moonriver"
    |               Variable                |       Value        |
    |:-------------------------------------:|:------------------:|
    |    Join/leave collator candidates     | {{ networks.moonriver.collator_timings.join_leave_candidates.rounds }} rounds ({{ networks.moonriver.collator_timings.join_leave_candidates.hours }} hours) |
    |        Add/remove nominations         | {{ networks.moonriver.collator_timings.add_remove_nominations.rounds }} rounds ({{ networks.moonriver.collator_timings.add_remove_nominations.hours }} hours) |
    | Rewards payouts (after current round) | {{ networks.moonriver.collator_timings.rewards_payouts.rounds }} rounds ({{ networks.moonriver.collator_timings.rewards_payouts.hours }} hours) |


!!! note 
    The values presented in the previous table are subject to change in future releases.

### Узнать размер пула кандидатов {: #get-the-size-of-the-candidate-pool } 

Сначала вам нужно получить размер `candidatePool` (он может меняться в процессе управления), так как вам нужно будет отправить этот параметр в последующей транзакции. Для этого нужно выполнить следующий фрагмент кода JavaScript из [PolkadotJS](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Fwss.testnet.moonbeam.network#/js):

```js
// Simple script to get candidate pool size
const candidatePool = await api.query.parachainStaking.candidatePool();
console.log(`Candidate pool size is: ${candidatePool.length}`);
```

 1. Перейдите на вкладку " Developer" (Разработчик) 
 2. Нажмите на "JavaScript"
 3. Скопируйте код из предыдущего фрагмента и вставьте его в поле редактора кода 
 4. (Необязательно) Нажмите на значок сохранения и задайте имя фрагмента кода, например, "Get candidatePool size". Это позволит сохранить фрагмент кода локально.
 5. Нажмите на кнопку выполнить. Это приведет к выполнению кода из окна редактора.
 6. Скопируйте результат, так как он понадобится вам при присоединении к пулу кандидатов

![Get Number of Candidates](/images/fullnode/collator-polkadotjs2.png)

### Присоединяйтесь к пулу кандидатов {: #join-the-candidate-pool } 

Once your node is running and in sync with the network, you become a collator candidate (and join the candidate pool). Depending on which network you are connected to, head to PolkadotJS for [Moonbase Alpha](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Fwss.testnet.moonbeam.network#/accounts) or [Moonriver](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Fwss.moonriver.moonbeam.network#/accounts) and take the following steps:

 1. Перейдите на вкладку " Developers" и нажмите на "Extrinsics".
 2. Выберите учетную запись, которую вы хотите ассоциировать с вашей деятельностью по коллации.
 3. Confirm your collator account is funded with at least the [minimum stake required](#accounts-and-staking-requirements) plus some extra for transaction fees
 4. Выберите паллет `parachainStaking` в меню "submit the following extrinsics".
 5. Откройте выпадающее меню, в котором перечислены все возможные варианты экстринсиков, связанных со стэкингом, и выберите функцию `joinCandidates()`.
 6. Set the bond to at least the [minimum amount](#accounts-and-staking-requirements) to be considered a collator candidate. Only collator bond counts for this check. Additional nominations do not count
 7. Установите количество кандидатов в качестве размера пула кандидатов. Чтобы узнать, как получить это значение, ознакомьтесь с [этим разделом](#get-the-size-of-the-candidate-pool).
 8. Отправьте транзакцию. Следуйте указаниям мастера и подпишите транзакцию, используя пароль, который вы установили для учетной записи

![Join Collators pool PolkadotJS](/images/fullnode/collator-polkadotjs3.png)

!!! Примечание
    Названия функций и минимальные требования к залогу могут быть изменены в будущих версиях.

As mentioned before, only the top {{ networks.moonbase.staking.max_collators }} collators on Moonbase Alpha and the top {{ networks.moonriver.staking.max_collators }} collators on Moonriver by nominated stake will be in the active set. 
### Прекращение коллаторства {: #stop-collating } 

Подобно Polkadot функции chill(), тобы выйти из пула кандидатов коллаторов, выполните те же шаги, что и раньше, но выберите функцию leaveCandidates() на шаге 5.

## Ключи сессии {: #session-keys } 

С выходом [Moonbase Alpha v8](/networks/testnet/) коллаторы будут подписывать блоки, используя идентификатор автора, который, по сути, является [ключом сессии](https://wiki.polkadot.network/docs/learn-keys#session-keys). Чтобы соответствовать стандарту Substrate, сессионными ключами коллаторов Moonbeam являются [SR25519](https://wiki.polkadot.network/docs/learn-keys#what-is-sr25519-and-where-did-it-come-from). Это руководство покажет вам, как можно создать/поменять сессионные ключи, связанные с нодой коллатора.

Сначала убедитесь, что у вас [запущен нода коллатора](/node-operators/networks/full-node/) и вы открыли порты RPC. После запуска ноды коллатора ваш терминал должен вывести похожие записи:

![Collator Terminal Logs](/images/fullnode/collator-terminal1.png)

Далее, сессионные ключи могут быть изменены путем отправки RPC вызова на конечную точку HTTP с методом `author_rotateKeys`. Для справки, если конечная точка HTTP вашего коллатора находится на порту `9933`, вызов JSON-RPC может выглядеть следующим образом:

```
curl http://127.0.0.1:9933 -H \
"Content-Type:application/json;charset=utf-8" -d \
  '{
    "jsonrpc":"2.0",
    "id":1,
    "method":"author_rotateKeys",
    "params": []
  }'
```

Нода коллатора должна ответить соответствующим открытым ключом нового идентификатора автора (ключ сессии).

![Collator Terminal Logs RPC Rotate Keys](/images/fullnode/collator-terminal2.png)

Обязательно запишите этот открытый ключ идентификатора автора. Далее он будет сопоставлен с адресом H160 в формате Ethereum, на который выплачиваются вознаграждения за блоки.

## Сопоставление авторского идентификатора с вашей учетной записью {: #map-author-id-to-your-account } 

После того как вы сгенерировали свой идентификатор автора (сессионные ключи), следующим шагом будет привязка его к учетной записи H160 (адрес, стилизованный под Ethereum). Убедитесь, что у вас есть приватные ключи от этого аккаунта, поскольку именно на него выплачиваются вознаграждения за блоки.

There is a bond that is sent when mapping your author ID with your account. This bond is per author ID registered. The bond set is as follows:

 - Moonbase Alpha - {{ networks.moonbase.staking.collator_map_bond }} DEV tokens 
 - Moonriver - {{ networks.moonriver.staking.collator_map_bond }} MOVR tokens. 

В модуле `authorMapping` запрограммированы следующие дополнительные функции:

 - **addAssociation**(*address* authorID) — maps your author ID to the H160 account from which the transaction is being sent, ensuring is the true owner of its private keys. It requires a [bond](#accounts-and-staking-requirements)
 - **clearAssociation**(*address* authorID) — clears the association of an author ID to the H160 account from which the transaction is being sent, which needs to be the owner of that author ID. Also refunds the bond
 - **updateAssociation**(*address* oldAuthorID, *address* newAuthorID) - обновляет связку со старого ID автора на новый. Применяется после ротации или миграции ключей. Выполняет атомарно оба экстринсика ассоциации `add` и `clear`, позволяя производить ротацию ключа без необходимости использования второй связи.

Модуль также добавляет следующие вызовы RPC (состояние цепи):

- **mapping**(*address* optionalAuthorID) - отображает все отображения, хранящиеся в цепи, или только те, которые относятся к вводу, если они были предоставлены
- 
### Составление карты Extrinsic {: #mapping-extrinsic } 

To map your author ID to your account, you need to be inside the [candidate pool](#become-a-collator-candidate). Once you are a collator candidate, you need to send a mapping extrinsic (transaction). Note that this will bond tokens per author ID registered. To do so, take the following steps:

 1. Перейдите на вкладку " Developer ".
 2. Выберите опцию "Extrinsics"
 3. Выберите учетную запись, с которой вы хотите связать свой идентификатор автора, с которой вы будете подписывать эту транзакцию
 4. Выберите `authorMapping`.
 5. Установите метод `addAssociation()`.
 6. Введите идентификатор автора. В данном случае он был получен через RPC вызов `author_rotateKeys` в предыдущем разделе
 7. Нажмите на кнопку " Submit Transaction".

![Author ID Mapping to Account Extrinsic](/images/fullnode/collator-polkadotjs4.png)

Если операция прошла успешно, на вашем экране появится уведомление о подтверждении. В противном случае убедитесь, что вы присоединились к [пулу кандидатов](#become-a-collator-candidate).

![Author ID Mapping to Account Extrinsic Successful](/images/fullnode/collator-polkadotjs5.png)

### Проверка сопоставлений {: #checking-the-mappings } 

Вы также можете проверить текущие сопоставления в цепи, проверив состояние цепи. Для этого выполните следующие действия:

 1. Перейдите на вкладку " Developer" 
 2. Выберите опцию " Chain state"
 3. Выберите `authorMapping` в качестве состояния для запроса
 4. Выберите метод `mappingWithDeposit`.
 5. Укажите ID автора для запроса. Оптимально, вы можете отключить ползунок для получения всех отображений. 
 6. Нажмите на кнопку "+", чтобы отправить вызов RPC

![Author ID Mapping Chain State](/images/fullnode/collator-polkadotjs6.png)

Вы должны увидеть аккаунт H160, связанный с предоставленным ID автора. Если ID автора не был указан, это вернет все отображения, хранящиеся в цепи.
