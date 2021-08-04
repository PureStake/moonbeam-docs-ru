---
title: Как стейкать
description: Руководство, которое показывает, как вы можете стейкать свои токены в Moonbeam, номинируя коллаторов
---

# Как застейкать свои токены

![Staking Moonbeam Banner](/images/staking/staking-stake-banner.png)

## Вступление {: #introduction } 

Коллаторы (производители блоков) с наибольшей долей в сети присоединяются к активному пулу коллаторов, из которого они выбираются для предложения блока relay chain.

Держатели токенов могут добавить к ставке коллекторов, используя свои токены, процесс, называемый назначением (также называемый стейкингом). Когда они это делают, они ручаются за этот конкретный коллатором , и их назначение является признаком доверия.

Когда коллатор ведет себя ненадлежащим образом, его доля в сети сокращается, что влияет и на токены, назначенные пользователями (функция в настоящее время недоступна в Moonbase Alpha). Если коллаторы будут действовать соответствующим образом, они получат вознаграждение за блок как часть инфляционной модели. Они могут поделиться ими со своими номинантами в качестве вознаграждения за стекинг.

С выпуском [Moonbase Alpha v6](https://github.com/PureStake/moonbeam/releases/tag/v0.6.0), пользователи сети теперь могут ставить свои токены для назначения каллаторов. В этом руководстве описаны все шаги необходимые для этого.

## Общие определения {: #general-definitions } 

--8<-- 'text/staking/staking-definitions.md'

В настоящее время для Moonbase Alpha существует следующее:

|             Variable             |     |                         Value                         |
| :------------------------------: | :-: | :---------------------------------------------------: |
|     Minimum nomination stake     |     |     {{ networks.moonbase.staking.min_nom_stake }}     |
|        Minimum nomination        |     |     {{ networks.moonbase.staking.min_nom_amount}}     | | Maximum nominators per collators |     |     {{ networks.moonbase.staking.max_nom_per_col }}   |
| Maximum collators per nominator  |     |     {{ networks.moonbase.staking.max_col_per_nom }}   |
|              Round               |     | {{ networks.moonbase.staking.round_blocks }} blocks ({{ networks.moonbase.staking.round_hours }} hours) |
|          Bond duration           |     |     {{ networks.moonbase.staking.bond_lock }} rounds  |

## Определения {: #extrinsics-definitions } 

Есть много внешних элементов, связанных с палетой для стекинга, поэтому все они не рассматриваются в этом руководстве. Однако этот список определяет все внешние факторы, связанные с процессом номинации:

!!! примечание 
    Внешний вид может измениться в будущем по мере обновления паллеты для стейкинга.

 - **nominate** — два входа: адрес коллатора для номинирования и сумма. Extrinsic, чтобы номинировать коллатор. Сумма должна быть не менее {{ networks.moonbase.staking.min_nom_amount }} токенов
 - **leaveNominators** — входов нет. Extrinsic чтобы покинуть набор номинаторов. Следовательно, все текущие номинации будут отменены.
 - **nominatorBondLess** — два входа: адрес номинированного коллатора и сумма. Extrinsic, чтобы уменьшить количество поставленных токенов для уже назначенного коллатора. Сумма не должна уменьшать вашу общую ставку ниже {{ networks.moonbase.staking.min_nom_stake }} токенов.
 - **nominatorBondMore** — два входа: адрес номинированного коллатора и сумма. Внешний для увеличения количества поставленных токенов для уже назначенного коллатора.
 - **revokeNomination** — один вход: адрес назначенного коллатора. Внешний, чтобы удалить существующую номинацию.

## Получение списка коллаторов {: #retrieving-the-list-of-collators } 

Перед тем, как начать ставку токенов, важно получить список коллаторов, доступных в сети. Для этого перейдите в «Состояние чейна» на вкладке «Разработчик».

![Staking Account](/images/staking/staking-stake-10.png)

Здесь укажите следующую информацию:

 1. Head to the "Developer" tab 
 2. Click on "Chain State"
 3. Choose the pallet to interact with. In this case, it is the `parachainStaking` pallet
 4. Choose the state to query. In this case, it is the `selectedCandidates` or `candidatePool` state
 5. Send the state query by clicking on the "+" button

Каждый extrinsic ответ дает свой ответ:

 - **selectedCandidates** — возвращает текущий активный набор коллаторов, то есть {{ networks.moonbase.staking.max_collators }} лучших коллаторов по общему количеству поставленных токенов (включая номинации).
 - **candidatePool** — возвращает в текущий список все коллаторы, в том числе те, которых нет в активном наборе.

![Staking Account](/images/staking/staking-stake-2.png)

## Get the Collator Nominator Count {: #get-the-collator-nominator-count }

First, you need to get the `collator_nominator_count` as you'll need to submit this parameter in a later transaction. To do so, you'll have to run the following JavaScript code snippet from within [PolkadotJS](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Fwss.testnet.moonbeam.network#/js):

```js
// Simple script to get collator_nominator_count
// Remember to replace COLLATOR_ADDRESS with the address of desired collator.
const collatorAccount = 'COLLATOR_ADDRESS'; 
const collatorInfo = await api.query.parachainStaking.collatorState2(collatorAccount);
console.log(collatorInfo.toHuman()["nominators"].length);
```

 1. Head to the "Developer" tab 
 2. Click on "JavaScript"
 3. Copy the code from the previous snippet and paste it inside the code editor box 
 4. (Optional) Click the save icon and set a name for the code snippet, for example, "Get collator_nominator_count". This will save the code snippet locally
 5. Click on the run button. This will execute the code from the editor box
 6. Copy the result, as you'll need it when initiating a nomination

![Get collator nominator count](/images/staking/staking-stake-3.png)

## Get your Number of Existing Nominations {: #get-your-number-of-existing-nominations }
If you've never made a nomination from your address you can skip this section. However, if you're unsure how many existing nominations you have, you'll want to run the following JavaScript code snippet to get `nomination_count` from within [PolkadotJS](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Fwss.testnet.moonbeam.network#/js):

```js
// Simple script to get your number of existing nominations.
// Remember to replace YOUR_ADDRESS_HERE with your nominator address.
const yourNominatorAccount = 'YOUR_ADDRESS_HERE'; 
const nominatorInfo = await api.query.parachainStaking.nominatorState(yourNominatorAccount);
console.log(nominatorInfo.toHuman()["nominations"].length);
```

 1. Head to the "Developer" tab 
 2. Click on "JavaScript"
 3. Copy the code from the previous snippet and paste it inside the code editor box 
 4. (Optional) Click the save icon and set a name for the code snippet, for example, "Get existing nominations". This will save the code snippet locally
 5. Click on the run button. This will execute the code from the editor box
 6. Copy the result, as you'll need it when initiating a nomination

![Get existing nomination count](/images/staking/staking-stake-4.png)

## Как номинировать коллатора {: #how-to-nominate-a-collator } 

В этом разделе рассматривается процесс назначения коллаторов. В этом руководстве в качестве справочника будут использоваться следующие колаторы :

|  Variable  |     |                      Address                       |
| :--------: | :-: | :------------------------------------------------: |
| Collator 1 |     | {{ networks.moonbase.staking.collators.address1 }} |
| Collator 2 |     | {{ networks.moonbase.staking.collators.address2 }} |

Чтобы получить доступ к функциям стейкинга, Вам необходимо использовать интерфейс приложений PolkadotJS. Для этого Вам необходимо сначала импортировать / создать учетную запись в стиле Ethereum (адрес H160), что Вы можете сделать, следуя [этому руководству](/integrations/wallets/polkadotjs/#creating-or-importing-an-h160-account).

В этом примере учетная запись была импортирована и названа супер-оригинальным именем: Алиса.

В настоящее время все, что связано со стейкингом необходимо получить через меню «Внешние компоненты» на вкладке «Разработчик»:

![Staking Account](/images/staking/staking-stake-5.png)

Чтобы проверить номинацию, Вы можете перейти в «Состояние цепочки» на вкладке «Разработчик».

 1. Выберите аккаунт с которого Вы хотите застейкать свои токены.
 2. Выберите палет с которым хотите взаимодействовать. В данном случае это `parachainStaking`
 3. Выберите внешний метод, который будет использоваться для транзакции. Это определит поля, которые необходимо заполнить в следующих шагах. В данном случае это `nominate`
 4. Укажите адрес коллатора которого Вы хотите номинировать. В данном случае это `{{ networks.moonbase.staking.collators.address1 }}`
 5. Укажите количество токенов которые Вы хотите застейкать.
 6. Input the `collator_nominator_count` you [retrieved above from the JavaScript console](/staking/stake/#get-the-collator-nominator-count)
 7. Input the `nomination_count` [you retrieved from the Javascript console](/staking/stake/#get-your-number-of-existing-nominations). This is `0` if you haven't yet nominated a collator
 8. Click the "Submit Transaction" button and sign the transaction

![Staking Join Nominators Extrinsics](/images/staking/staking-stake-6.png)

!!! note
    The parameters used in steps 6 and 7 are for gas estimation purposes and do not need to be exact. However, they should not be lower than the actual values. 

После подтверждения транзакции Вы можете подтвердить свою новую номинацию в опции «Состояние чейна» на вкладке «Разработчик»:

Чтобы проверить номинацию, Вы можете перейти в «Состояние цепочки» на вкладке «Разработчик».

![Staking Account and Chain State](/images/staking/staking-stake-7.png)

Здесь укажите следующую информацию:

 1. Выберите палет с которым хотите взаимодействовать. В данном случае это `parachainStaking` 
 2. Choose the state to query. In this case, it is the `nominatorState`
 3. Make sure to enable the "include option" slider
 4. Отправьте запрос состояния, нажав кнопку «+»

![Staking Chain State Query](/images/staking/staking-stake-8.png)

В ответе Вы должны увидеть свою учетную запись (в данном случае учетную запись Алисы) со списком номинаций. Каждая номинация содержит целевой адрес коллатора и сумму.

Чтобы назначить следующего коллатора Вам нужно повторить тот же процесс, что и раньше. К примеру, Алиса так же номинировала `{{ networks.moonbase.staking.collators.address2 }}`

## Как остановить номинации {: #how-to-stop-nominations } 

Если Вы уже являетесь номинатором, у Вас есть два варианта остановить свои номинации: использовать внешнюю функцию `revokeNomination`, чтобы вывести свои токены из определенного коллатора или использовать внешнюю функцию `leaveNominators` чтобы отозвать все текущие номинации.

Этот пример является продолжением предыдущего раздела и подразумевает, что у Вас есть как минимум две активные номинации.

Вы можете удалить свою номинацию из определенного коллатора, перейдя в меню «Внешние элементы» на вкладке «Разработчик». Здесь укажите следующую информацию:

 1. Выберите аккаунт, из которого Вы хотите удалить свою номинацию
 2. Выберите палет, с которой хотите взаимодействовать. В данном случае это `parachainStaking`
 3. Выберите внешний метод, который будет использоваться для транзакции. Это определит поля, которые необходимо заполнить в следующих шагах. В данном случае это внешний вид `revokeNomination`
 4. Задайте адрес подборщика, из которого Вы хотите удалить свою кандидатуру. В данном случае это `{{ networks.moonbase.staking.collators.address2 }}`
 5. Нажмите кнопку «Отправить транзакцию» и подпишите транзакцию.

![Staking Revoke Nomination Extrinsic](/images/staking/staking-stake-9.png)

После подтверждения транзакции Вы можете убедиться, что Ваша номинация была удалена, в опции «Состояние чейна» на вкладке «Разработчик».

Здесь укажите следующую информацию:

 1. Выберите палет, с которой хотите взаимодействовать. В данном случае это `parachainStaking`
 2. Выберите состояние для запроса. В данном случае это `nominatorState`
 3. Make sure to enable the "include options" slider
 4. Отправьте запрос состояния, нажав кнопку "+".

![Staking Revoke Nomination Chain State](/images/staking/staking-stake-8.png)

В ответе вы должны увидеть свою учетную запись (в данном случае учетную запись Алисы) со списком номинаций. Каждая номинация содержит целевой адрес коллатора и сумму.

Как упоминалось ранее, Вы также можете удалить все текущие номинации с помощью функции `leaveNominators` (на шаге 3 инструкций "Extrinsics"). Этот extrinsic не требует ввода:

![Staking Leave Nominatiors Extrinsic](/images/staking/staking-stake-10.png)

После подтверждения транзакции Ваша учетная запись не должна быть указана в списке номинаторов при запросе, и у Вас не должно быть зарезервированного баланса (связанного с размещением ставок).

## Награды за стейкинг {: #staking-rewards } 

Поскольку коллаторы получают вознаграждение за производство блоков, номинаторы также получают вознаграждение. Краткий обзор того, как рассчитываются вознаграждения, можно найти на [этой странице](/staking/overview/#reward-distribution).

Таким образом, номинаторы будут получать вознаграждение в зависимости от их доли от общего числа номинаций для присуждаемого коллатора(включая долю сборщика).

В предыдущем примере Алиса была вознаграждена `0.0044`токенов после двух раундов выплат: 

![Staking Reward Example](/images/staking/staking-stake-1.png)
