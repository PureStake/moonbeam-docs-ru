---
title: MathWallet
description: В этом руководстве вы узнаете, как подключить Mathwallet, браузерный кошелек, работающий с Ethereum, к Moonbeam.
---

# Взаимодействие с Moonbeam при помощи MathWallet
 
![Intro banner](/images/mathwallet/mathwallet-banner.png)

## Вступление {: #introduction } 

MathWallet [announced](https://mathwallet.org/moonbeam-wallet/en/) that is now natively supporting the [Moonbase Alpha TestNet](/networks/moonbase/) and [Moonriver](/networks/moonriver/). This means that you are now able to interact with Moonbase Alpha and Moonriver using another wallet besides MetaMask.

In this tutorial, we'll go through how to setup MathWallet to connect to our [TestNet](#connect-to-moonbase-alpha) and [Moonriver](#connect-to-moonriver). We'll also present a brief example of using MathWallet as a Web3 provider for other tools such as [Remix](/integrations/remix/).

## Checking Prerequisites {: #checking-prerequisites } 

Во-первых, Вам необходимо установить расширение браузера MathWallet, которое вы можете скачать с их [веб-сайта](https://mathwallet.org/en-us/).

Установив расширение для браузера, откройте его и установите пароль.

![Установите пароль кошелька](/images/mathwallet/mathwallet-images-1.png)
## Подключение к Moonbase Alpha {: #connect-to-moonbase-alpha } 

In this part, we'll go through the process of connecting MathWallet to Moonbase Alpha. Enable Moonbase Alpha under Settings (top right gear icon) -> Networks -> Ethereum.

![Enable Moonbase Alpha](/images/mathwallet/mathwallet-images-2.png)

И, наконец, на главном экране нажмите Switch Network и выберите Moonbase Alpha.

![Connect to Moonbase Alpha](/images/mathwallet/mathwallet-images-3.png)

Теперь у Вас есть MathWallet, подключенный к Moonbase Alpha TestNet! Ваш кошелек должен выглядеть так:

![Wallet Connected to Moonbase Alpha](/images/mathwallet/mathwallet-images-4.png)

## Подключение к Moonriver {: #connect-to-moonriver } 

Getting started with Moonriver is a straightforward process. All you have to do is click Switch Network and select Moonriver

![Connect to Moonriver](/images/mathwallet/mathwallet-images-5.png)

And that is it, you now have MathWallet connected to Moonriver! Your wallet should look like this:

![Wallet Connected to Moonriver](/images/mathwallet/mathwallet-images-6.png)

## Добавление кошелька {: #adding-a-wallet } 

The following steps will show you how to interact with the Moonbase Alpha TestNet, but can also be used to interact with Moonriver.

After you are connected to Moonbase Alpha, you can now create a wallet to get an account and start interacting with the TestNet. Currently, there are three ways to add a wallet:

 - Создать кошелек
 - Импортировать существующий кошелек с помощью мнемонического или закрытого ключа
 - Подключить аппаратный кошелек (_пока не поддерживается_)

### Создание кошелька {: #create-a-wallet } 

The following steps for creating a wallet can be modified for Moonriver.

Чтобы создать новый кошелек, щелкните значок :heavy_plus_sign: рядом с «Moonbase Alpha» и выберите «Создать кошелек».

![MathWallet create a wallet](/images/mathwallet/mathwallet-images-7.png)

Установите и подтвердите имя кошелька. Затем убедитесь, что Вы безопасно сохранили “мнемоник фразу”, поскольку она обеспечивает прямой доступ к вашим средствам. После завершения процесса Вы должны увидеть новый созданный кошелек с соответствующим публичным адресом.

![MathWallet wallet created](/images/mathwallet/mathwallet-images-8.png)

### Импортирование кошелька {: #import-a-wallet } 

Чтобы создать новый кошелек, щелкните значок :heavy_plus_sign: рядом с «Moonbase Alpha» и выберите «Импортировать кошелек».

![MathWallet import a wallet](/images/mathwallet/mathwallet-images-9.png)

Затем выберите вариант импорта с использованием мнемонического или закрытого ключа. Для первого варианта введите мнемоническую фразу (каждое слово через пробел). Для второго варианта введите закрытый ключ (либо с префиксом `0x`, либо без него, оба варианта рабочие).

![MathWallet private key or mnemonic import](/images/mathwallet/mathwallet-images-10.png)

После нажатия кнопки «Далее» задайте имя кошелька, на этом импорт кошелька завершён! Вы должны увидеть импортированный кошелек со связанным публичным адресом.

![MathWallet imported wallet](/images/mathwallet/mathwallet-images-11.png)

## Использование MathWallet {: #using-mathwallet } 

MathWallet serves as a Web3 provider in tools such as [Remix](/integrations/remix/). By having MathWallet connected to Moonbase Alpha or Moonriver, you can deploy contracts as you would like using MetaMask, signing the transactions with MathWallet instead.

For example, in Remix, when deploying a smart contract to Moonbase Alpha, make sure you select the "Injected Web3" option in the "Environment" menu. If you have MathWallet connected, you will see the TestNet chain ID just below the box (_1287_) and your MathWallet account injected into Remix as well. When sending a transaction, you should see a similar pop-up from MathWallet:

![MathWallet sign transaction](/images/mathwallet/mathwallet-images-12.png)

Нажимая «Принять», Вы подписываете эту транзакцию, и контракт будет развернут на Moonbase Alpha TestNet.

