---
title: Прекомпиляция Стейкинга
description: Демонстрации интерфейса прекомпиляции Парачейн-Стейкинга Moonbeam на основе Ethereum Solidity 
---

# Прекомпиляция Стейкинга

![Staking Moonbeam Banner](/images/tokens/staking/precompiles/precompile-banner.png)

## Введение {: #introduction } 

Недавно дебютировал набор модулей с делегированным proof of stake под названием [Парачейн-Стейкинг] (https://github.com/PureStake/moonbeam/tree/master/pallets/parachain-staking/src), позволяющий держателям токенов (номинаторам) точно выразить, каких кандидатов в коллаторы они хотели бы поддержать и с каким количеством доли. Дизайн модуля Парачейн-Стейкинга таков, что он обеспечивает разделение риска/вознаграждения по цепи между делегирующими и коллаторами.

Модуль Стейкинг написан на языке Rust и является частью набора модулей, который обычно недоступен со стороны Ethereum в Moonbeam. Однако прекомпиляция Стейкинга позволяет разработчикам получить доступ к функциям Стейкинг с помощью Ethereum API в предварительно скомпилированном контракте, расположенном по адресу `{{networks.moonbase.staking.precompile_address}}. Впервые прекомпиляция стейкинга была выпущена в [релизе Moonbase Alpha v8] (https://moonbeam.network/announcements/testnet-upgrade-moonbase-alpha-v8/).


## Интерфейс Парачейн-Стейкинга на Solidity {: #the-parachain-staking-solidity-interface } 

[StakingInterface.sol](https://github.com/PureStake/moonbeam/blob/master/precompiles/parachain-staking/StakingInterface.sol) - это интерфейс, через который контракты solidity могут взаимодействовать с Парачейн-Стейкинга. Преимущество заключается в том, что разработчикам solidity не нужно изучать Substrate API. Вместо этого они могут взаимодействовать с функциями стейкинга, используя знакомый им интерфейс Ethereum.

Переведено с помощью www.DeepL.com/Translator (бесплатная версия)

Интерфейс включает следующие функции:

  - **is_nominator**(*address* collator) - функция только для чтения, которая проверяет, является ли указанный адрес в настоящее время номинантом для стейкинга
 - **is_candidate**(*address* collator) - функция только для чтения, которая проверяет, является ли указанный адрес в настоящее время кандидатом в коллаторы
 - **min_nomination**() - функция только для чтения, которая получает минимальную сумму номинации
 - **join_candidates**(*uint256* amount) - позволяет учетной записи присоединиться к набору кандидатов-коллаторов с указанной суммой взноса.
 - **leave_candidates**() - немедленно удаляет счет из пула кандидатов, чтобы другие не могли выбрать его в качестве коллатора, и запускает открепление после истечения раундов BondDuration
 - **go_offline**() - временно отменить набор кандидатов в коллаторы без отмены взноса
 - **go_online**() - снова вернуться к набору кандидатов в коллаторы после предварительного вызова go_offline()
 - **candidate_bond_more**(*uint256* more) - кандидат в коллаторы увеличивает взнос на указанную величину
 - **candidate_bond_less**(*uint256* less) - кандидат в коллаторы уменьшает взнос на указанную величину
 - **nominate**(*address* collator, *uint256* amount) - если вызывающий пользователь не является номинатором, эта функция добавляет его в набор номинаторов. Если абонент уже является номинатором, то корректируется сумма номинации.
 - **leave_nominators**() - покинуть набор номинаторов и отозвать все текущие номинации
 - **revoke_nomininations**(*address* collator) - отменить конкретную номинацию
 - **nominator_bond_more**(*address* collator, *uint256* more) - номинатор увеличивает взнос  коллатору на указанную сумму
 - **nominator_bond_less**(*address* collator, *uint256* less) - номинатор уменьшает взнос  коллатору на указанную сумму

## Взаимодействие с прекомпиляцией стейкинга {: #interacting-with-the-staking-precompile } 

### Проверка необходимых условий {: #checking-prerequisites } 
Приведенный ниже пример демонстрируется на Moonbase Alpha, однако он совместим со всеми сетями, включая Moonriver и Moonbeam.

 - У вас должен быть установлен MetaMask и [подключен к Moonbase Alpha](/getting-started/moonbase/metamask/)
 - Иметь аккаунт с более чем `{{networks.moonbase.staking.min_nom_stake}}` токенов. Вы можете получить их в [Mission Control](/getting-started/moonbase/faucet/)

!!! примечание
    Приведенный ниже пример требует больше, чем `{{networks.moonbase.staking.min_nom_stake}} токенов из-за минимальной суммы номинации плюс плата за газ. Если вам нужно больше, чем выдает кран, пожалуйста, свяжитесь с нами в Discord, и ы с радостью вам поможем. 

### Настройка Remix {: #remix-set-up } 
1. Получите копию [StakingInterface.sol](https://github.com/PureStake/moonbeam/blob/master/precompiles/parachain-staking/StakingInterface.sol).
2. Скопируйте и вставьте содержимое файла в файл Remix с именем StakingInterface.sol

![Copying and Pasting the Staking Interface into Remix](/images/tokens/staking/precompiles/precompile-1.png)

### Компиляция Контракта {: #compile-the-contract } 
1. Перейдите на вкладку Compile, вторую сверху.
2. Скомпилируйте [Staking Interface.sol](https://github.com/PureStake/moonbeam/blob/master/precompiles/parachain-staking/StakingInterface.sol)

![Compiling StakingInteface.sol](/images/tokens/staking/precompiles/precompile-2.png)

### Доступ к контракту {: #access-the-contract } 
1. Перейдите на вкладку " Deploy and Run" (Развертывание и запуск), расположенную непосредственно под вкладкой "Compile" (Компиляция) в Remix. **Примечание**: здесь мы не развертываем контракт, вместо этого мы получаем доступ к уже развернутому предварительно скомпилированному контракту.
2. Убедитесь, что "Injected Web3" выбран в выпадающем списке Environment
3. Убедитесь, что в выпадающем списке Contract выбран "ParachainStaking - StakingInterface.sol". Поскольку это прекомпилированный контракт, нет необходимости его развертывать, вместо этого мы укажем адрес прекомпиляции в поле "At Address".
4. Укажите адрес прекомпиляции Staking: `{{networks.moonbase.staking.precompile_address}} и нажмите "At Address".

![Provide the address](/images/tokens/staking/precompiles/precompile-3.png)

### Номинирование Коллатора {: #nominate-a-collator } 
В данном примере мы будем номинировать коллатора. Номинаторы - это держатели токенов, которые стейкают свои токены, через конкретных коллаторов. Номинатором может стать любой пользователь, имеющий на свободном балансе минимальное количество токенов {{networks.moonbase.staking.min_nom_stake}}.

1. Разверните панель с адресом контракта. Найдите функцию nominate и разверните панель, чтобы увидеть параметры.
2. Укажите адрес коллатора, например `{{ networks.moonbase.staking.collators.address1 }}`.
3. Укажите сумму для номинации в WEI. Существует минимальное количество токенов `{{networks.moonbase.staking.min_nom_stake}} для номинации, поэтому наименьшая сумма в WEI составляет `5000000000000000000`.
4. Нажмите "transact" и подтвердите транзакцию в Metamask

![Nominate a Collator](/images/tokens/staking/precompiles/precompile-4.png)

### Проверка номинации {: #verify-nomination } 
Чтобы убедиться, что ваша номинация прошла успешно, вы можете проверить состояние сети в Polkadot.js Apps. Сначала добавьте адрес вашего Мetamask в [адресную книгу в Polkadot.js Apps](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Fwss.testnet.moonbeam.network#/addresses). Если вы уже выполнили этот шаг, вы можете перейти к следующему разделу.

#### Добавить адрес Metamask в адресную книгу {: #add-metamask-address-to-address-book } 
1. Перейдите в раздел Accounts -> Address Book 
2. Нажмите на " Add contact"
3. Добавьте свой адрес Metamask
4. Укажите псевдоним для учетной записи

![Add to Address Book](/images/tokens/staking/precompiles/precompile-5.png)

#### Проверить сведения о номинации {: #verify-nominator-state } 
1. Чтобы проверить, что ваша номинация прошла успешно, зайдите на [Polkadot.js Apps](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Fwss.testnet.moonbeam.network#/chainstate) и перейдите в раздел Developer -> Chain State.
2. Выберите набор модулей "parachainStaking".
3. Выберите запрос "nominatorState"
4. Нажмите кнопку "Plus" для получения результатов и проверки вашей номинации

![Verify Nomination](/images/tokens/staking/precompiles/precompile-6.png)

### Аннулирование номинации {: #revoking-a-nomination } 
Чтобы отозвать номинацию и получить свои токены обратно, вызовите метод `revoke_nomination`, указав тот же адрес, с которого вы начали номинацию. Для подтверждения вы можете еще раз проверить состояние номинации на Polkadot.js Apps.

![Revoke Nomination](/images/tokens/staking/precompiles/precompile-7.png)
