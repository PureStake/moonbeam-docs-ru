---
title: Отправка транзакции
description: Узнайте как создавать и отправлять транзакции в сети Moonbeam, совместимой с Ethereum, с помощью простого скрипта с использованием web3.js, ethers.js или web3.py.
---

# Использование библиотек Ethereum для отправки транзакций в сети Moonbeam

![Ethereum Libraries Integrations Moonbeam](/images/builders/interact/eth-libraries/web3-libraries-banner.png)

## Вступление {: #introduction } 

В этом руководстве рассматривается использование трех разных библиотек Ethereum для ручной подписи и отправки транзакции на Moonbeam. Рассматриваются следующие три библиотеки:

 - [Web3.js](https://web3js.readthedocs.io/)
 - [Ethers.js](https://docs.ethers.io/)
 - [Web3.py](https://web3py.readthedocs.io/)

!!! примечание 
    --8<-- 'text/common/assumes-mac-or-ubuntu-env.md'

## Проверка предварительных условий {: #checking-prerequisites } 

Примеры, использующие как web3.js, так и ethers.js, требуют предварительной установки Node.js и NPM. В примере с использованием web3.py требуются Python и PIP. На момент написания этого руководства использовались следующие версии:

 - Node.js v15.10.0
 - NPM v7.5.3
 - Python v3.6.9 (web3 требуется Python >= 3.5.3 and < 4)
 - PIP3 v9.0.1

Затем создайте каталог для хранения всех соответствующих файлов:

```
mkdir transaction && cd transaction/
```

Для библиотек JavaScript Вы можете сначала создать простой файл package.json (не обязательно):

```
npm init --yes
```

В каталоге установите используемую библиотеку (web3.py устанавливается в каталог PIP3 по умолчанию):

=== "Web3.js"
    ```
    npm i web3
    ```

=== "Ethers.js"
    ```
    npm i ethers
    ```

=== "Web3.py"
    ```
    pip3 install web3
    ```

На момент публикации этого руководства использовались следующие версии:

 - Web3.js v1.33 (`npm ls web3`)
 - Ethers.js v5.0.31 (`npm ls ethers`)
 - Web3.py v5.17.0 (`pip3 show web3`)

## Файл транзакции {: #the-transaction-file } 

Для выполнения транзакции между учетными записями нужен только один файл. Сценарий, показанный в этом разделе, перенесет 1 токен с исходного адреса (на котором Вы храните закрытый ключ) на другой адрес. Вы можете найти фрагменты кода для каждой библиотеки здесь (они были произвольно названы `transaction. *`):

 - Web3.js: [_transaction.js_](/snippets/code/web3-tx-local/transaction.js)
 - Ethers.js: [_transaction.js_](/snippets/code/ethers-tx-local/transaction.js)
 - Web3.py: [_transaction.py_](/snippets/code/web3py-tx/transaction.py)

Каждый из файлов, независимо от используемой библиотеки, разделен на три раздела. В первом разделе ("Define Provider & Variables") импортируется используемая библиотека, а также определяются провайдер и другие переменные (переменные зависят от библиотеки). Обратите внимание, что `providerRPC` имеет как стандартную конечную точку RPC для автономной ноды, так и конечную точку для [Moonbase Alpha](/networks/moonbase/).

Во втором разделе ("Create and Deploy Transaction") функции, необходимые для отправки самой транзакции. Некоторые из основных выводов будут рассмотрены далее.

=== "Web3.js"
    ```
    --8<-- 'code/web3-tx-local/transaction.js'
    ```

=== "Ethers.js"
    ```
    --8<-- 'code/ethers-tx-local/transaction.js'
    ```

=== "Web3.py"
    ```
    --8<-- 'code/web3py-tx/transaction.py'
    ```

### Web3.js {: #web3js } 

В первом разделе [скрипта](/snippets/code/web3-tx-local/transaction.js), `web3` создается с помощью конструктора `Web3` с провайдером RPC. Изменяя RPC поставщика, предоставленный конструктору, Вы можете выбрать, в какую сеть Вы хотите отправить транзакцию.

Закрытый ключ и связанный с ним общедоступный адрес определяются для подписания транзакции и ведения журнала соответственно. Требуется только закрытый ключ.

Значение `addressTo`, куда будет отправлена транзакция, также определяется здесь и является обязательным.

Во втором разделе создается объект транзакции с полями `to`, `value` и `gas`. Они описывают получателя, сумму для отправки и газ, потребленный транзакцией (в данном случае 21000). Вы можете использовать функцию `web3.utils.toWei()` для ввода значения в ETH (например) и получения вывода в WEI. Транзакция подписывается закрытым ключом с помощью метода `web3.eth.accounts.signTransaction ()`. 

Затем, подписав транзакцию (Вы можете вопспользоваться `console.log (createTransaction)`, чтобы увидеть значения v-r-s), можно отправить ее с помощью метода `web3.eth.sendSignedTransaction ()`, предоставив подписанную транзакцию, расположенную в `createTransaction.rawTransaction`.

Наконец, запустите функцию асинхронного размещения.

### Ethers.js {: #ethersjs } 

В первом разделе [скрипта](/snippets/code/ethers-tx-local/transaction.js), можно указать разные сети с помощью имени, URL-адреса RPC (обязательно) и идентификатора цепочки. Провайдер (аналогичный экземпляру `web3`) создается с помощью метода `ethers.providers.StaticJsonRpcProvider`. Альтернативой является использование метода `ethers.providers.JsonRpcProvide (providerRPC)`, для которого требуется только адрес конечной точки RPC провайдера. Но это может создать проблемы совместимости с индивидуальными спецификациями проекта.

Закрытый ключ определяется для создания экземпляра кошелька, для которого также требуется провайдер из предыдущего шага. Экземпляр кошелька используется для подписи транзакций.

Значение `addressTo`, куда будет отправлена транзакция, также определяется здесь и является обязательным.

Во втором разделе асинхронная функция оборачивает метод `wallet.sendTransaction(txObject)`. Объект транзакции довольно простой. Требуется только адрес получателя и сумма для отправки. Обратите внимание, что можно использовать метод `ethers.utils.parseEther()`, который обрабатывает необходимые преобразования единиц из ETH в WEI - аналогично использованию `ethers.utils.parseUnits (value, 'ether')`.

После отправки транзакции Вы можете получить ответ транзакции (в этом примере с именем `createReceipt`), который имеет несколько свойств. Например, Вы можете вызвать метод `createReceipt.wait ()`, чтобы дождаться обработки транзакции (статус получения - ОК).

Наконец, запустите функцию асинхронного размещения.

### Web3.py {: #web3py } 

В первом разделе [скрипта](/snippets/code/web3py-tx/transaction.py), экземпляр `web3` (провайдер) создается с помощью метода `Web3(Web3.HTTPProvider (provider_rpc))` и указанием RPC провайдера. Изменяя RPC провайдера, Вы можете выбрать, в какую сеть Вы хотите отправить транзакцию.

Закрытый ключ и связанный с ним общедоступный адрес определяются для подписания транзакции и ведения журнала. Публичный адрес не требуется.

Значение `addressTo`, куда будет отправлена транзакция, также определяется здесь и является обязательным.

Во втором разделе объект транзакции создается с полями `nonce`, `gasPrice`, `gas`, `to`, и `value`.  Они описывают количество транзакций, цену на газ (0 для автономной версии и Moonbase Alpha), количество газа (в данном случае 21000), получателя и сумму отправки. Обратите внимание, что количество транзакций можно получить с помощью метода `web3.eth.getTransactionCount(address)`. Кроме того, Вы можете использовать функцию `web3.toWei()` для ввода значения в ETH (например) и получения вывода в WEI. Транзакция подписывается закрытым ключом с помощью метода `web3.eth.account.signTransaction ()`.

Затем, подписав транзакцию, Вы можете отправить ее с помощью метода `web3.eth.sendSignedTransaction()` предоставив подписанную транзакцию, расположенную в `createTransaction.rawTransaction`.

## Файл баланса {: #the-balance-file } 

Перед запуском скрипта, другой файл проверяет баланс обоих адресов до и после транзакции. Это легко сделать, просто запросив баланс счета.

Вы можете найти фрагменты кода для каждой библиотеки здесь (файлы были произвольно названы `balances.*`):

 - Web3.js: [_balances.js_](/snippets/code/web3-tx-local/balances.js)
 - Ethers.js: [_balances.js_](/snippets/code/ethers-tx-local/balances.js)
 - Web3.py: [_balances.py_](/snippets/code/web3py-tx/balances.py)

Для простоты, файл баланса состоит из двух разделов. Как и раньше, в первом разделе ("Define Provider & Variables") импортируется используемая библиотека, а также определяются провайдер и адрес от/до (для проверки баланса).

Во втором разделе ("Balance Call Function") описаны функции, необходимые для получения балансов по определенным ранее аккаунтам. Обратите внимание, что `providerRPC` имеет как стандартную конечную точку RPC автономной ноды, так и конечную точку для [Moonbase Alpha](/networks/moonbase/). Некоторые из основных выводов будут рассмотрены далее.

=== "Web3.js"
    ```
    --8<-- 'code/web3-tx-local/balances.js'
    ```

=== "Ethers.js"
    ```
    --8<-- 'code/ethers-tx-local/balances.js'
    ```

=== "Web3.py"
    ```
    --8<-- 'code/web3py-tx/balances.py'
    ```

### Web3.js {: #web3js } 

Первый раздел [скрипта](/snippets/code/web3-tx-local/balances.js) очень похож на раздел в [файле транзакции](/getting-started/local-node/send-transaction/#web3js). Основное отличие состоит в том, что закрытый ключ не требуется, потому что нет необходимости отправлять транзакцию.

Во втором разделе асинхронная функция оборачивает метод web3, используемый для получения баланса адреса - `web3.eth.getBalance(address)`. Вы можете использовать функцию `web3.utils.fromWei()`, чтобы преобразовать баланс в более читаемое число в ETH.

### Ethers.js {: #ethersjs } 

Первый раздел [скрипта](/snippets/code/ethers-tx-local/balances.js) очень похож на раздел в [файле транзакции](/getting-started/local-node/send-transaction/#ethersjs). Основное отличие состоит в том, что закрытый ключ не требуется, потому что нет необходимости отправлять транзакцию. Напротив, необходимо определить `addressFrom`.

Во втором разделе асинхронная функция оборачивает метод web3, используемый для получения баланса адреса - `provider.getBalance(address)`. Вы можете использовать функцию `ethers.utils.formatEther()` чтобы преобразовать баланс в более читаемое число в ETH.

### Web3.py {: #web3py } 

Первый раздел [скрипта](/snippets/code/web3py-tx/balances.py) очень похож на раздел в [файле транзакций](/getting-started/local-node/send-transaction/#web3py). Основное отличие состоит в том, что закрытый ключ не требуется, потому что нет необходимости отправлять транзакцию.

Во втором разделе для получения баланса целевого адреса, используется метод `web3.eth.getBalance(address)`. Вы можете использовать функцию `eb3.fromWei()`, чтобы преобразовать баланс в более читаемое число в ETH.

## Запуск скриптов {: #running-the-scripts } 

В этом разделе показанный ранее код был адаптирован для работы с автономной нодой, которую Вы можете запустить, следуя [этому руководству](/getting-started/local-node/setting-up-a-node/). Кроме того, каждая транзакция была отправлена с предварительно пополненной учетной записи, которая поставляется с нодой:

--8<-- 'text/metamask-local/dev-account.md'

Сначала проверьте баланс обоих адресов перед транзакцией, запустив (обратите внимание, что каталог был переименован для каждой библиотеки):

=== "Web3.js"
    ```
    node balances.js
    ```

=== "Ethers.js"
    ```
    node balances.js
    ```

=== "Web3.py"
    ```
    python3 balances.py
    ```

Затем запустите сценарий _transaction.\*_ для выполнения транзакции:

=== "Web3.js"
    ```
    node transaction.js
    ```

=== "Ethers.js"
    ```
    node transaction.js
    ```

=== "Web3.py"
    ```
    python3 transaction.py
    ```

И на последок еще раз проверьте баланс, чтобы убедиться, что перевод прошел успешно. Выполнение должно выглядеть так:

=== "Web3.js"
    ![Send Tx Web3js](/images/builders/interact/eth-libraries/send-tx/sendtx-web3js.png)

=== "Ethers.js"
    ![Send Tx Etherjs](/images/builders/interact/eth-libraries/send-tx/sendtx-ethers.png)

=== "Web3.py"
    ![Send Tx Web3py](/images/builders/interact/eth-libraries/send-tx/sendtx-web3py.png)

