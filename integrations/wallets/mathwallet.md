---
title: MathWallet
description: В этом руководстве вы узнаете, как подключить Mathwallet, браузерный кошелек, работающий с Ethereum, к Moonbeam.
---

# Взаимодействие с Moonbeam при помощи MathWallet
 
![Intro banner](/images/mathwallet/mathwallet-banner.png)

## Вступление

MathWallet [анонсировал](https://mathwallet.org/moonbeam-wallet/en/) что теперь он поддерживает тестовую сеть [Moonbase Alpha TestNet](/networks/moonbase/). Это означает, что теперь Вы можете взаимодействовать с Moonbase Alpha, используя ещё один кошелек помимо MetaMask.

В этом руководстве мы рассмотрим, как настроить MathWallet для подключения к нашей “TestNet” сети. Мы также представим краткий пример использования MathWallet в качестве поставщика Web3 услуг для других инструментов, таких как [Remix](/integrations/remix/).

## Подключение MathWallet к Moonbeam

В этой части мы рассмотрим процесс подключения MathWallet к Moonbase Alpha.

Во-первых, Вам необходимо установить расширение браузера MathWallet, которое вы можете скачать с их [веб-сайта](https://mathwallet.org/en-us/).

Установив расширение для браузера, откройте его и установите пароль.

![Установите пароль кошелька](/images/mathwallet/mathwallet-images-1.png)

Затем давайте включим Moonbase Alpha в разделе «Настройки» (значок шестеренки вверху справа) -> Сети -> Ethereum.

![Enable Moonbase Alpha](/images/mathwallet/mathwallet-images-2.png)

И, наконец, на главном экране нажмите Switch Network и выберите Moonbase Alpha.

![Connect to Moonbase Alpha](/images/mathwallet/mathwallet-images-3.png)

Теперь у Вас есть MathWallet, подключенный к Moonbase Alpha TestNet! Ваш кошелек должен выглядеть так:

![Wallet Connected to Moonbase Alpha](/images/mathwallet/mathwallet-images-4.png)

## Добавление кошелька

Теперь, когда MathWallet подключен к Moonbase Alpha, мы можем создать кошелек, чтобы получить учетную запись и начать взаимодействие с TestNet. В настоящее время есть три способа добавить кошелек:

 - Создать кошелек
 - Импортировать существующий кошелек с помощью мнемонического или закрытого ключа
 - Подключить аппаратный кошелек (_пока не поддерживается_)

### Создание кошелька

Чтобы создать новый кошелек, щелкните значок :heavy_plus_sign: рядом с «Moonbase Alpha» и выберите «Создать кошелек».

![MathWallet create a wallet](/images/mathwallet/mathwallet-images-5.png)

Установите и подтвердите имя кошелька. Затем убедитесь, что Вы безопасно сохранили “мнемоник фразу”, поскольку она обеспечивает прямой доступ к вашим средствам. После завершения процесса Вы должны увидеть новый созданный кошелек с соответствующим публичным адресом.

![MathWallet wallet created](/images/mathwallet/mathwallet-images-6.png)

### Импортирование кошелька

Чтобы создать новый кошелек, щелкните значок :heavy_plus_sign: рядом с «Moonbase Alpha» и выберите «Импортировать кошелек».

![MathWallet import a wallet](/images/mathwallet/mathwallet-images-7.png)

Затем выберите вариант импорта с использованием мнемонического или закрытого ключа. Для первого варианта введите мнемоническую фразу (каждое слово через пробел). Для второго варианта введите закрытый ключ (либо с префиксом `0x`, либо без него, оба варианта рабочие).

![MathWallet private key or mnemonic import](/images/mathwallet/mathwallet-images-8.png)

После нажатия кнопки «Далее» задайте имя кошелька, на этом импорт кошелька завершён! Вы должны увидеть импортированный кошелек со связанным публичным адресом.

![MathWallet imported wallet](/images/mathwallet/mathwallet-images-9.png)

## Использование MathWallet

MathWallet служит поставщиком Web3 в таких инструментах, как [Remix](/integrations/remix/). Подключив MathWallet к Moonbase Alpha, Вы можете развертывать контракты так, так же как используя MetaMask, вместо этого подписывая транзакции с помощью MathWallet.

Например, в Remix, при развертывании смарт-контракта, убедитесь, что Вы выбрали опцию «Injected Web3» в меню «Environment». Если у Вас подключен MathWallet, вы увидите идентификатор цепочки TestNet чуть ниже поля (_1287_) а также вашу учетную запись MathWallet, введенную в Remix. При отправке транзакции Вы должны увидеть подобное всплывающее окно от MathWallet:

![MathWallet sign transaction](/images/mathwallet/mathwallet-images-10.png)

Нажимая «Принять», Вы подписываете эту транзакцию, и контракт будет развернут на Moonbase Alpha TestNet.

