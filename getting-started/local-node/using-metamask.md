---
title: Использование MetaMask
description: В этом руководстве вы узнаете, как взаимодействовать с локальной нодой Moonbeam, используя, установленный по умолчанию, плагин для браузера MetaMask.
---

# Взаимодействие ноды Moonbeam с помощью MetaMask

<style>.embed-container { position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; } .embed-container iframe, .embed-container object, .embed-container embed { position: absolute; top: 0; left: 0; width: 100%; height: 100%; }</style><div class='embed-container'><iframe src='https://www.youtube.com/embed//hrpBd2-a7as' frameborder='0' allowfullscreen></iframe></div>
<style>.caption { font-family: Open Sans, sans-serif; font-size: 0.9em; color: rgba(170, 170, 170, 1); font-style: italic; letter-spacing: 0px; position: relative;}</style><div class='caption'>
Вы можете найти весь необходимый код касающийся этого руководства в <a href="{{ config.site_url }}resources/code-snippets/">code snippets page</a></div>

## Введение

MetaMask можно использовать для подключения к Moonbeam через Moonbase Alpha TestNet или через автономную ноду Moonbeam, работающую локально.

В этом руководстве описаны все шаги для подключения MetaMask к автономной ноде Moonbeam для отправки токенов между учетными записями. Если вы еще не настроили собственную ноду разработчика, обратитесь к [этому руководству](/getting-started/local-node/setting-up-a-node/), или следуйте инструкции в [репозитории Github](https://github.com/PureStake/moonbeam/).

!!! note
    Это руководство было создано с использованием тега {{ networks.development.build_tag}}, который основан на {{ networks.moonbase.version }} версии [Moonbase Alpha](https://github.com/PureStake/moonbeam/releases/tag/{{ networks.moonbase.version }}). Платформа Moonbeam и компоненты [Frontier](https://github.com/paritytech/frontier), которые используются для совместимости Ethereum на основе Substrate, все еще находятся в стадии активной разработки. 
    --8<-- 'text/common/assumes-mac-or-ubuntu-env.md'

Вы можете взаимодействовать с Moonbeam двумя способами: используя конечные точки RPC Substrate или конечные точки RPC, совместимые с Web3. Последние конечные точки в настоящее время обслуживаются тем же сервером RPC, что и RPC Substrate. В этом руководстве мы будем использовать конечные точки Web3 RPC для взаимодействия с Moonbeam.

## Установка расширения Metamask

Во-первых, мы начнем с установки [MetaMask](https://metamask.io/) по умолчанию из магазина Chrome. После загрузки, установки и инициализации расширения следуйте руководству «Начало работы». Там вам нужно создать кошелек, установить пароль и сохранить секретную фразу (это дает прямой доступ к вашим средствам, поэтому обязательно храните их в безопасном месте). После завершения мы импортируем учетную запись разработчика:

![Импортируйте учетную запись разработчика](/images/metamask/using-metamask-1.png)

Подробные сведения об учетных записях разработчиков, которые поставляются предварительно пополненными для этой ноды разработчика, выглядят следующим образом:

--8<-- 'code/setting-up-node/dev-accounts.md'

--8<-- 'code/setting-up-node/dev-testing-account.md'

На экране импорта выберите «Приватный ключ» и вставьте один из ключей, перечисленных выше. В этом примере мы будем использовать ключ Джеральда:

![Вставьте ключ от вашего аккаунта в MetaMask](/images/metamask/using-metamask-2.png)

У вас должен получиться импортированный «Account 2», который выглядит следующим образом:

![MetaMask отображает ваш новый Account 2](/images/metamask/using-metamask-3.png)

## Подключение к локальной ноде Moonbeam

MetaMask можно настроить для подключения к локальной ноде разработчика или к Moonbase Alpha TestNet.

Чтобы подключить MetaMask к Moonbeam, перейдите в Настройки -> Сети -> Добавить сеть. Здесь вы можете настроить, к какой сети вы хотите подключить MetaMask, используя следующие конфигурации сети:

Нода разработчика Moonbeam:

--8<-- 'text/metamask-local/development-node-details.md'

Moonbase Alpha TestNet:

--8<-- 'text/testnet/testnet-details.md'

Для целей этого руководства давайте подключим MetaMask к нашей локальной ноде разработчика Moonbeam.

![Введите информацию о вашей новой сети в MetaMask](/images/metamask/using-metamask-4.png)

Когда вы нажмете «сохранить» и будете выходить из экрана сетевых настроек, MetaMask должен быть подключен к локальной ноде Moonbeam через его Web3 RPC, и вы должны увидеть учетную запись разработчика Moonbeam с балансом 1208925.8196 DEV.

![Баланс вашей учтеонй записи Moonbeam 1207925.8196](/images/metamask/using-metamask-5.png)

## Инициализация перевода

Давайте попробуем отправить несколько токенов с помощью MetaMask.

Для простоты мы будем делать перевод из этой учетной записи разработчика в учетную запись, созданную нами при настройке MetaMask. Следовательно, мы можем использовать опцию «Перевод между моими счетами». Давайте перенесем 100 токенов и оставим все остальные настройки как есть:

![Инициализация перевода токенов](/images/metamask/using-metamask-6.png)

После того, как вы отправили транзакцию, вы увидите, что статус изменился на «ожидает», пока не будет подтверждена транзакция, как показано на следующем изображении:

![Подтверждение транзакции](/images/metamask/using-metamask-7.png)

Учтите, что баланс “Account 2”  уменьшился на сумму отправления + плата за газ. Перейдя к учетной записи “Account 1”, мы видим, что 100 отправленных токенов прибыли:

![Новый баланс “Account 1](/images/metamask/using-metamask-8.png)

Если вы вернетесь к своему терминалу, где запущена ваша нода Moonbeam, вы увидете блоки, создаваемые по мере возникновения транзакций:

![Нода разработчика Moonbeam](/images/metamask/using-metamask-9.png)

!!! note
    Если вы в конечном итоге сбросите свою ноду с помощью команды Substrate purge-chain, вам нужно будет сбросить свою учетную запись MetaMask genesis, используя Настройки -> Дополнительно -> Сбросить учетную запись. Это очистит историю транзакций из ваших учетных записей и сбросит одноразовый номер. Убедитесь, что вы не стираете все, что хотите сохранить!
 
