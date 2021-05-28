---
title: Размещение контракта
description: Узнайте  как разместить немодифицированные и неизмененные смарт-контракты на основе Solidity, на ноде Moonbeam с помощью простого скрипта с использованием Web3.js, Ethers.js или Web3.py.
---

# Использование библиотек Ethereum для размещения Smart-контрактов на Moonbeam

![Ethereum Libraries Integrations Moonbeam](/images/sendtx/web3-libraries-banner.png)

## Введение

В этом руководстве рассматривается использование компилятора Solidity и трех различных библиотек Ethereum для подписания и отправки транзакции в Moonbeam вручную. Данными библиотеками являются: 

 - [Web3.js](https://web3js.readthedocs.io/)
 - [Ethers.js](https://docs.ethers.io/)
 - [Web3.py](https://web3py.readthedocs.io/)

Кроме того, для составления смарт-контракта будут использованы еще две другие библиотеки:

 - [Solc-js](https://www.npmjs.com/package/solc) to compile Solidity smart contracts using JavaScript
 - [Py-solc-x](https://pypi.org/project/py-solc-x/) to compile Solidity smart contracts using Python

!!! note
    --8<-- 'text/common/assumes-mac-or-ubuntu-env.md'

## Проверяем необходимые требования

К примеру, тем кто использует web3.js, ethers.js необходимо предварительно установить Node.js и NPM. Для web3.py Вам понадобятся Python и PIP. На момент написания этого руководства использовались следующие версии:

 - Node.js v15.10.0
 - NPM v7.5.3
 - Python v3.6.9 (web3 requires Python >= 3.5.3 and < 4)
 - PIP3 v9.0.1

Дальше, создайте папку для хранения всех соответствующих файлов: 

```
mkdir incrementer && cd incrementer/
```

Для библиотек JavaScript сначала Вы можете создать простой файл package.json (не обязательно):

```
npm init --yes
```

В каталоге установите соответствующую библиотеку и компилятор Solidity (_web3.py_ и _py-solc-x_ устанавливаются по умолчанию в каталог PIP3):

=== "Web3.js"
    ```
    npm i web3 npm i solc@0.8.0
    ```

=== "Ethers.js"
    ```
    npm i ethers npm i solc@0.8.0
    ```

=== "Web3.py"
    ```
    pip3 install web3 pip3 install py-solc-x
    ```

На момент публикации этого руководства использовались следующие версии:

 - Web3.js v1.33 (`npm ls web3`)
 - Ethers.js v5.0.31 (`npm ls ethers`)
 - Solc (JS) v0.8.0 (`npm ls solc`)
 - Web3.py v5.17.0 (`pip3 show web3`)
 - Py-solc-x v1.1.0 (`pip3 show py-solc-x`)

Настройка для этого примера будет простой и будет содержать следующие файлы:

 - **_Incrementer.sol_** — the file with our Solidity code
 - **_compile.\*_** — compiles the contract with the Solidity compiler
 - **_deploy.\*_**: it will handle the deployment to our local Moonbeam node
 - **_get.\*_** — it will make a call to the node to get the current value of the number
 - **_increment.\*_** — it will make a transaction to increment the number stored on the Moonbeam node
 - **_reset.\*_** — the function to call that will reset the number stored to zero

## Файл контракта

Используемый контракт представляет собой простой инкрементатор, произвольно названный _Incrementer.sol_, который вы можете найти [здесь](/snippets/code/web3-contract-local/Incrementer.sol). Код Solidity будет выглядеть следующим образом:

```solidity
--8<-- 'code/web3-contract-local/Incrementer.sol'
```

Функция `constructor` которая запускается при размещении контракта, устанавливает начальное значение числовой переменной, хранящейся в цепочке (по умолчанию 0). Функция `increment` добавляет предоставленное значение `_value` к текущему числу, для этого необходимо отправить транзакцию, которая изменяет сохраненные данные. Наконец, функция `reset` возвращает сохраненное значение к нулю.

!!! note
    Этот контракт представляет собой только пример и не обрабатывает значения, оборачивающиеся вокруг.


## Компиляция смарт-контракта

Единственная цель файла компиляции - использовать компилятор Solidity для вывода байт-кода и интерфейса (ABI) нашего контракта. Вы можете найти фрагменты кода для каждой библиотеки здесь (они были произвольно названы `compile.*`):

 - Web3.js: [_compile.js_](/snippets/code/web3-contract-local/compile.js)
 - Ethers.js: [_compile.js_](/snippets/code/web3-contract-local/compile.js)
 - Web3.py: [_compile.py_](/snippets/code/web3py-contract/compile.py)

!!! note
    Файл компиляции для обеих библиотек JavaScript такой же, поскольку они используют привязки JavaScript для компилятора Solidity (один и тот же пакет)

=== "Web3.js"
    ```
    --8<-- 'code/web3-contract-local/compile.js'
    ```

=== "Ethers.js"
    ```
    --8<-- 'code/web3-contract-local/compile.js'
    ```

=== "Web3.py"
    ```
    --8<-- 'code/web3py-contract/compile.py'
    ```

### Web3.js и Ethers.js

В первой части [скрипта](/snippets/code/web3-contract-local/compile.js) выбирается путь к контракту и читается его содержимое.

Затем создается входной объект компилятора Solidity, который передается в качестве входных данных функции `solc.compile`.

Наконец, извлеките данные контракта `Incrementer` из файла `Incrementer.sol`, затем экспортируйте их, чтобы сценарий размещения мог их использовать.

### Web3.py

В первой части [скрипта](/snippets/code/web3py-contract/compile.py), файл контракта компилируется с помощью функции `solcx.compile_files`. Обратите внимание, что файл контракта находится в том же каталоге, что и сценарий компиляции.

!!! note
    При запуске `compile.py` вы можете получить сообщение об ошибке, в котором говорится, что необходимо установить `Solc`. В таком случае, раскомментируйте строку в файле, которая выполняет `solcx.install_solc()` и повторно запустите файл компиляции с помощью `python3 compile.py`. Более подробную информацию можно найти по этой [ссылке](https://pypi.org/project/py-solc-x/).

Затем, после завершения скрипта данные контракта экспортируются. В этом примере были определены только интерфейс (ABI) и байт-код.

## Размещение смарт-контракта

Независимо от библиотеки, стратегия размещения скомпилированного смарт-контракта имеет схододство. Экземпляр контракта создается с использованием его интерфейса (ABI) и байт-кода. В этом экземпляре, функция размещения используется для отправки подписанной транзакции, которая размещает контракт. Здесь вы можете найти фрагменты кода для каждой библиотеки (они были произвольно названы `deploy.*`):

 - Web3.js: [_deploy.js_](/snippets/code/web3-contract-local/deploy.js)
 - Ethers.js: [_deploy.js_](/snippets/code/ethers-contract-local/deploy.js)
 - Web3.py: [_deploy.py_](/snippets/code/web3py-contract/deploy.py)

Файл размещения состоит из двух разделов. В первом разделе ("Define Provider & Variables") импортируется используемая библиотека, а также ABI и байт-код контракта. Также определяются провайдер и учетная запись (с закрытым ключом). Обратите внимание, что `providerRPC` имеет как стандартную конечную точку RPC узла разработки, так и конечную точку для [Moonbase Alpha](/networks/testnet/).

Во втором разделе ("Deploy Contract") описывается фактическая часть размещения контракта. Обратите внимание, что для этого примера начальное значение числовой переменной было установлено на 5. Некоторые из ключевых выводов будут рассмотрены далее.

=== "Web3.js"
    ```
    --8<-- 'code/web3-contract-local/deploy.js'
    ```

=== "Ethers.js"
    ```
    --8<-- 'code/ethers-contract-local/deploy.js'
    ```

=== "Web3.py"
    ```
    --8<-- 'code/web3py-contract/deploy.py'
    ```

!!! note
    Скрипт _deploy.\*_ предоставляет адрес контракта в качестве вывода. Это очень удобно, поскольку он используется для взаимодействия с контрактом.


### Web3.js

In the first part of [the script](/snippets/code/web3-contract-local/deploy.js), the `web3` instance (or provider) is created using the `Web3` constructor with the provider RPC. By changing the provider RPC given to the constructor, you can choose which network you want to send the transaction to.

The private key, and the public address associated with it, are defined for signing the transaction and logging purposes. Only the private key is required. Also, the contract's bytecode and interface (ABI) are fetched from the compile's export.

In the second section, a contract instance is created by providing the ABI. Next, the `deploy` function is used, which needs the bytecode and arguments of the constructor function. This will generate the constructor transaction object.

Afterwards, the constructor transaction can be signed using the `web3.eth.accounts.signTransaction()` method. The data field corresponds to the bytecode, and the constructor input arguments are encoded together. Note that the value of gas is obtained using `estimateGas()` option inside the constructor transaction.

Lastly, the signed transaction is sent, and the contract's address is displayed in the terminal.

### Ethers.js

In the first part of [the script](/snippets/code/ethers-contract-local/deploy.js), different networks can be specified with a name, RPC URL (required), and chain ID. The provider (similar to the `web3` instance) is created with the `ethers.providers.StaticJsonRpcProvider` method. An alternative is to use the `ethers.providers.JsonRpcProvide(providerRPC)` method, which only requires the provider RPC endpoint address. But this might created compatibility issues with individual project specifications.

The private key is defined to create a wallet instance, which also requires the provider from the previous step. The wallet instance is used to sign transactions. Also, the contract's bytecode and interface (ABI) are fetched from the compile's export.

In the second section, a contract instance is created with `ethers.ContractFactory()`, providing the ABI, bytecode, and wallet. Thus, the contract instance already has a signer. Next, the `deploy` function is used, which needs the constructor input arguments. This will send the transaction for contract deployment. To wait for a transaction receipt you can use the `deployed()` method of the contract deployment transaction.

Lastly, the contract's address is displayed in the terminal.

### Web3.py

In the first part of [the script](/snippets/code/web3py-contract/deploy.py), the `web3` instance (or provider) is created using the `Web3(Web3.HTTPProvider(provider_rpc))` method with the provider RPC. By changing the provider RPC, you can choose which network you want to send the transaction to.

The private key and the public address associated with it are defined for signing the transaction and establishing the from address.

In the second section, a contract instance is created with `web3.eth.contract()`, providing the ABI and bytecode imported from the compile file. Next, the constructor transaction can be built using the `constructor().buildTransaction()` method of the contract instance. Note that inside the `constructor()`, you need to specify the constructor input arguments. The `from` account needs to be outlined as well. Make sure to use the one associated with the private key. Also, the transaction count can be obtained with the `web3.eth.getTransactionCount(address)` method.

The constructor transaction can be signed using `web3.eth.account.signTransaction()`, passing the constructor transaction and the private key.

Lastly, the signed transaction is sent, and the contract's address is displayed in the terminal.

## Reading from the Contract (Call Methods)

Call methods are the type of interaction that don't modify the contract's storage (change variables), meaning no transaction needs to be sent.

Let's overview the _get.\*_ file (the simplest of them all), which fetches the current value stored in the contract. You can find the code snippet for each library here (they were arbitrarily named `get.*`):

 - Web3.js: [_get.js_](/snippets/code/web3-contract-local/get.js)
 - Ethers.js: [_get.js_](/snippets/code/ethers-contract-local/get.js)
 - Web3.py: [_get.py_](/snippets/code/web3py-contract/get.py)

For simplicity, the get file is composed of two sections. In the first section ("Define Provider & Variables"), the library to use and the ABI of the contract are imported. Also, the provider and the contract's address are defined. Note that `providerRPC` has both the standard development node RPC endpoint and the one for [Moonbase Alpha](/networks/testnet/).

The second section ("Call Function") outlines the actual call to the contract. Regardless of the library, a contract instance is created (linked to the contract's address), from which the call method is queried. Some of the key takeaways are discussed next.

=== "Web3.js"
    ```
    --8<-- 'code/web3-contract-local/get.js'
    ```

=== "Ethers.js"
    ```
    --8<-- 'code/ethers-contract-local/get.js'
    ```

=== "Web3.py"
    ```
    --8<-- 'code/web3py-contract/get.py'
    ```

### Web3.js

In the first part of [the script](/snippets/code/web3-contract-local/get.js), the `web3` instance (or provider) is created using the `Web3` constructor with the provider RPC. By changing the provider RPC given to the constructor, you can choose which network you want to send the transaction to.

The contract's interface (ABI) and address are needed as well to interact with it.

In the second section, a contract instance is created with `web3.eth.Contract()` by providing the ABI and address. Next, the method to call can be queried with the `contract.methods.methodName(_input).call()` function, replacing `contract`, `methodName` and `_input` with the contract instance, function to call, and input of the function (if necessary). This promise, when resolved, will return the value requested.

Lastly, the value is displayed in the terminal.

### Ethers.js

In the first part of [the script](/snippets/code/ethers-contract-local/get.js), different networks can be specified with a name, RPC URL (required), and chain ID. The provider (similar to the `web3` instance) is created with the `ethers.providers.StaticJsonRpcProvider` method. An alternative is to use the `ethers.providers.JsonRpcProvide(providerRPC)` method, which only requires the provider RPC endpoint address. But this might created compatibility issues with individual project specifications.

The contract's interface (ABI) and address are needed as well to interact with it.

In the second section, a contract instance is created with `ethers.Contract()`, providing its address, ABI, and the provider. Next, the method to call can be queried with the `contract.methodName(_input)` function, replacing `contract` `methodName`, and `_input` with the contract instance, function to call, and input of the function (if necessary). This promise, when resolved, will return the value requested.

Lastly, the value is displayed in the terminal.

### Web3.py

In the first part of [the script](/snippets/code/web3py-contract/get.py), the `web3` instance (or provider) is created using the `Web3(Web3.HTTPProvider(provider_rpc))` method with the provider RPC. By changing the provider RPC, you can choose which network you want to send the transaction to.

The contract's interface (ABI) and address are needed as well to interact with it.

In the second section, a contract instance is created with `web3.eth.contract()` by providing the ABI and address. Next, the method to call can be queried with the `contract.functions.method_name(input).call()` function, replacing `contract`, `method_name` and `input` with the contract instance, function to call, and input of the function (if necessary). This returns the value requested.

Lastly, the value is displayed in the terminal.

## Interacting with the Contract (Send Methods)

Send methods are the type of interaction that modify the contract's storage (change variables), meaning a transaction needs to be signed and sent.

First, let's overview the _increment.\*_ file, which increments the current number stored in the contract by a given value. You can find the code snippet for each library here (they were arbitrarily named `increment.*`):

 - Web3.js: [_increment.js_](/snippets/code/web3-contract-local/increment.js)
 - Ethers.js: [_increment.js_](/snippets/code/ethers-contract-local/increment.js)
 - Web3.py: [_increment.py_](/snippets/code/web3py-contract/increment.py)

For simplicity, the increment file is composed of two sections. In the first section ("Define Provider & Variables"), the library to use and the ABI of the contract are imported. The provider, the contract's address, and the value of the `increment` function are also defined. Note that `providerRPC` has both the standard development node RPC endpoint and the one for [Moonbase Alpha](/networks/testnet/).

The second section ("Send Function") outlines the actual function to be called with the transaction. Regardless of the library, a contract instance is created (linked to the contract's address), from which the function to be used is queried.

=== "Web3.js"
    ```
    --8<-- 'code/web3-contract-local/increment.js'
    ```

=== "Ethers.js"
    ```
    --8<-- 'code/ethers-contract-local/increment.js'
    ```

=== "Web3.py"
    ```
    --8<-- 'code/web3py-contract/increment.py'
    ```

The second file to interact with the contract is the _reset.\*_ file, which resets the number stored in the contract to zero. You can find the code snippet for each library here (they were arbitrarily named `reset.*`):

 - Web3.js: [_reset.js_](/snippets/code/web3-contract-local/reset.js)
 - Ethers.js: [_reset.js_](/snippets/code/ethers-contract-local/reset.js)
 - Web3.py: [_reset.py_](/snippets/code/web3py-contract/reset.py)

Each file's structure is very similar to his _increment.\*_ counterpart for each library. The main difference is the method being called.

=== "Web3.js"
    ```
    --8<-- 'code/web3-contract-local/reset.js'
    ```

=== "Ethers.js"
    ```
    --8<-- 'code/ethers-contract-local/reset.js'
    ```

=== "Web3.py"
    ```
    --8<-- 'code/web3py-contract/reset.py'
    ```

### Web3.js

In the first part of the script ([increment](/snippets/code/web3-contract-local/increment.js) or [reset](/snippets/code/web3-contract-local/reset.js) files), the `web3` instance (or provider) is created using the `Web3` constructor with the provider RPC. By changing the provider RPC given to the constructor, you can choose which network you want to send the transaction to.

The private key, and the public address associated with it, are defined for signing the transaction and logging purposes. Only the private key is required. Also, the contract's interface (ABI) and address are needed to interact with it. If necessary, you can define any variable required as input to the function you are going to interact with.

In the second section, a contract instance is created with `web3.eth.Contract()` by providing the ABI and address. Next, you can build the transaction object with the `contract.methods.methodName(_input)` function, replacing `contract`, `methodName` and `_input` with the contract instance, function to call, and input of the function (if necessary).

Afterwards, the transaction can be signed using the `web3.eth.accounts.signTransaction()` method. The data field corresponds to the transaction object from the previous step. Note that the value of gas is obtained using `estimateGas()` option inside the transaction object.

Lastly, the signed transaction is sent, and the transaction hash is displayed in the terminal.

### Ethers.js

In the first part of the script ([increment](/snippets/code/ethers-contract-local/increment.js) or [reset](/snippets/code/ethers-contract-local/reset.js) files), different networks can be specified with a name, RPC URL (required), and chain ID. The provider (similar to the `web3` instance) is created with the `ethers.providers.StaticJsonRpcProvider` method. An alternative is to use the `ethers.providers.JsonRpcProvide(providerRPC)` method, which only requires the provider RPC endpoint address. But this might created compatibility issues with individual project specifications.

The private key is defined to create a wallet instance, which also requires the provider from the previous step. The wallet instance is used to sign transactions. Also, the contract's interface (ABI) and address are needed to interact with it. If necessary, you can define any variable required as input to the function you are going to interact with.

In the second section, a contract instance is created with `ethers.Contract()`, providing its address, ABI, and wallet. Thus, the contract instance already has a signer. Next, transaction corresponding to a specific function can be send with the `contract.methodName(_input)` function, replacing `contract`, `methodName` and `_input` with the contract instance, function to call, and input of the function (if necessary). To wait for a transaction receipt, you can use the `wait()` method of the contract deployment transaction.

Lastly, the transaction hash is displayed in the terminal.

### Web3.py

In the first part of the script ([increment](/snippets/code/web3py-contract/increment.py) or [reset](/snippets/code/web3py-contract/reset.py) files), the `web3` instance (or provider) is created using the `Web3(Web3.HTTPProvider(provider_rpc))` method with the provider RPC. By changing the provider RPC, you can choose which network you want to send the transaction to.

The private key and the public address associated with it are defined for signing the transaction and establishing the from address. Also, the contract's interface (ABI) and address are needed as well to interact with it.

In the second section, a contract instance is created with `web3.eth.contract()` by providing the ABI and address. Next, you can build the transaction object with the `contract.functions.methodName(_input).buildTransaction` function, replacing `contract`, `methodName` and `_input` with the contract instance, function to call, and input of the function (if necessary). Inside `buildTransaction()`, the `from` account needs to be outlined. Make sure to use the one associated with the private key. Also, the transaction count can be obtained with the `web3.eth.getTransactionCount(address)` method.

The transaction can be signed using `web3.eth.account.signTransaction()`, passing the transaction object of the previous step and the private key.

Lastly, the transaction hash is displayed in the terminal.

## Running the Scripts

For this section, the code shown before was adapted to target a development node, which you can run by following [this tutorial](/getting-started/local-node/setting-up-a-node/). Also, each transaction was sent from the pre-funded account that comes with the node:

--8<-- 'text/metamask-local/dev-account.md'

First, deploy the contract by running (note that the directory was renamed for each library):

=== "Web3.js"
    ```
    node deploy.js
    ```

=== "Ethers.js"
    ```
    node deploy.js
    ```

=== "Web3.py"
    ```
    python3 deploy.py
    ```

This will deploy the contract and return the address:

=== "Web3.js"
    ![Deploy Contract Web3js](/images/deploycontract/contract-deploy-web3js.png)

=== "Ethers.js"
    ![Deploy Contract Etherjs](/images/deploycontract/contract-deploy-ethers.png)

=== "Web3.py"
    ![Deploy Contract Web3py](/images/deploycontract/contract-deploy-web3py.png)

Next, run the increment file. You can use the get file to verify the value of the number stored in the contract before and after increment it:

=== "Web3.js"
    ```
    # Get value
    node get.js 
    # Increment value
    increment.js
    # Get value
    node get.js
    ```

=== "Ethers.js"
    ```
    # Get value
    node get.js 
    # Increment value
    increment.js
    # Get value
    node get.js
    ```

=== "Web3.py"
    ```
    # Get value
    python3 get.py 
    # Increment value
    python3 increment.py
    # Get value
    python3 get.py
    ```

This will display the value before the increment transaction, the hash of the transaction, and the value after:

=== "Web3.js"
    ![Increment Contract Web3js](/images/deploycontract/contract-increment-web3js.png)

=== "Ethers.js"
    ![Increment Contract Etherjs](/images/deploycontract/contract-increment-ethers.png)

=== "Web3.py"
    ![Increment Contract Web3py](/images/deploycontract/contract-increment-web3py.png)

Lastly, run the reset file. Once again, you can use the get file to verify the value of the number stored in the contract before and after resetting it:

=== "Web3.js"
    ```
    # Get value
    node get.js 
    # Reset value
    node reset.js 
    # Get value
    node get.js
    ```

=== "Ethers.js"
    ```
    # Get value
    node get.js 
    # Reset value
    node reset.js 
    # Get value
    node get.js
    ```

=== "Web3.py"
    ```
    # Get value
    python3 get.py 
    # Reset value
    python3 reset.py
    # Get value
    python3 get.py
    ```

This will display the value before the reset transaction, the hash of the transaction, and the value after:

=== "Web3.js"
    ![Reset Contract Web3js](/images/deploycontract/contract-reset-web3js.png)

=== "Ethers.js"
    ![Reset Contract Etherjs](/images/deploycontract/contract-reset-ethers.png)

=== "Web3.py"
    ![Reset Contract Web3py](/images/deploycontract/contract-reset-web3py.png)

