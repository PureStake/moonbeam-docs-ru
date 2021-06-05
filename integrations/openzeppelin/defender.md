---
title: Defender
description:  Узнайте, как использовать OpenZeppelin Defender для безопасного управления смарт-контрактами на Moonbeam с использованием функций совместимости с Ethereum
---

# Defender OpenZeppelin

![OpenZeppelin Defender Banner](/images/openzeppelin/ozdefender-banner.png)

## Введение

OpenZeppelin Defender - это веб-приложение, которое позволяет разработчикам безопасно выполнять и автоматизировать операции со смарт-контрактами. Защитник предлагает различные компоненты:

 - [**Admin**](https://docs.openzeppelin.com/defender/admin) — создание автоматизации и обеспечения безопасности всех операций смарт-контракта, таких как контроль доступа, обновление и приостановка.
 - [**Relay**](https://docs.openzeppelin.com/defender/relay) — создание частную и безопасную инфраструктуру транзакций с внедрением частных ретрансляторов
 - [**Autotasks**](https://docs.openzeppelin.com/defender/autotasks) — создание автоматизированных скриптов для взаимодействия с вашими смарт-контрактами
 - [**Sentinel**](https://docs.openzeppelin.com/defender/sentinel) — отслеживание событий, функций и транзакций вашего смарт-контракта и получение уведомлений по электронной почте
 - [**Advisor**](https://docs.openzeppelin.com/defender/advisor) — изучение и внедрение лучшего опыта в области разработки, тестирования, мониторинга и эксплуатации

Defender от OpenZeppelin теперь можно использовать в тестовой сети Moonbase Alpha. Это руководство расскажет вам, как начать работу с Defender и продемонстрирует компонент Admin для приостановки смарт-контракта. Более подробную информацию о других компонентах вы можете найти в вышеупомянутых ссылках.

Для получения дополнительной информации команда OpenZeppelin написала отличный [сайт документации](https://docs.openzeppelin.com/defender/) для Defender.

## Начало работы с Defender на Moonbase Alpha

В этом разделе описаны шаги по началу работы с OpenZeppelin Defender на Moonbase Alpha.
 
### Проверка необходимых условий

Шаги, описанные в этом разделе, предполагают, что у вас установлен [MetaMask](https://metamask.io/) и подключен к Moonbase Alpha TestNet. Если вы еще не подключили MetaMask к TestNet, ознакомьтесь с нашим руководством по интеграции [MetaMask](/integrations/wallets/metamask/).

Кроме того, необходимо зарегистрировать бесплатную учетную запись OpenZeppelin Defender, что можно сделать на главном сайте [Defender](https://defender.openzeppelin.com/).

Контракт, используемый в этом руководстве, является расширением контракта `Box.sol`, используемого в  [руководстве по обновлению смарт-контрактов](https://docs.openzeppelin.com/learn/upgrading-smart-contracts), из документации OpenZeppelin. Кроме того, контракт был сделан обновляемым и [приостанавливаемым ](https://docs.openzeppelin.com/contracts/4.x/api/security#Pausable), чтобы использовать все преимущества компонента Admin. Вы можете разместить свой контракт, используя следующий код и следуя  [руководству по обновлению смарт-контрактов](https://docs.openzeppelin.com/learn/upgrading-smart-contracts):

```sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol";
import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";

contract PausableBox is Initializable, PausableUpgradeable, OwnableUpgradeable {
    uint256 private value;
 
    // Emitted when the stored value changes
    event ValueChanged(uint256 newValue);

    // Initialize
    function initialize() initializer public {
        __Ownable_init();
        __Pausable_init_unchained();
    }
 
    // Stores a new value in the contract
    function store(uint256 newValue) whenNotPaused public {
        value = newValue;
        emit ValueChanged(newValue);
    }
 
    // Reads the last stored value
    function retrieve() public view returns (uint256) {
        return value;
    }
    
    function pause() public onlyOwner {
        _pause();
    }

    function unpause() public onlyOwner {
        _unpause();
    }
}
```

### Подключение Defender к Moonbase Alpha

После того как у вас есть учетная запись OpenZeppelin Defender, войдите в [Приложение Defender](https://defender.openzeppelin.com/). На главном экране, с MetaMask [подключенным к Moonbase Alpha](/getting-started/moonbase/metamask/) нажмите на кнопку "Connect wallet" в правом верхнем углу:

![Подключение OpenZeppelin Defender](/images/openzeppelin/ozdefender-images1.png)

В случае успешного завершения вы увидите свой адрес и сообщение " Connected to Moonbase Alpha".

## Использование компонента Admin

В этом разделе описаны шаги по началу работы с компонентом OpenZeppelin Defender Admin для управления смарт-контрактами на Moonbase Alpha.

### Импорт вашего контракта

Первым шагом в использовании Defender Admin является добавление контракта, которым вы хотите управлять. Для этого нажмите на кнопку " Add contract" в правом верхнем углу. Это откроет окно " Import contract", где вам необходимо:

 1. Set a contract name. This is only for display purposes
 2. Выберите сеть, в которой размещен контракт, которым вы хотите управлять. Это особенно удобно, когда контракт размещен с одним и тем же адресом в нескольких сетях. В данном примере введите `Moonbase Alpha`.
 3. Введите адрес контракта
 4. Вставьте ABI контракта. Его можно получить либо в [Remix](/integrations/remix/), либо в файле `.json`, обычно создаваемом после процесса компиляции (например, в Truffle или HardHat).
 5. Проверьте, правильно ли были определены функции контракта.
 6. После проверки всей информации нажмите на кнопку " Add".

![OpenZeppelin Defender Admin Добавить контракт](/images/openzeppelin/ozdefender-images2.png)

Если все было успешно импортировано, вы должны увидеть свой контракт на главном экране компонента Admin:

![OpenZeppelin Defender Admin Contract Added](/images/openzeppelin/ozdefender-images3.png)

### Создание предложения по контракту

Предложения - это действия, которые должны быть выполнены в договоре. На момент написания существует три основных предложения/действия, которые могут иметь место:

- **Pause** — доступна при обнаружении функции паузы. Приостанавливает перевод, минтинга и сжигание жетонов
- **Upgrade** — доступна, если обнаружена функция обновления. Позволяет [обновить контракт через прокси-контракт](https://docs.openzeppelin.com/learn/upgrading-smart-contracts)
- **Admin action** — вызов любой функции в управляемом контракте

В этом случае создается новое предложение, чтобы приостановить действие контракта. Для этого выполните следующие шаги:

 1. Нажмите на кнопку " New proposal", чтобы увидеть все доступные варианты
 2. Нажмите на " Pause" (Пауза)

![OpenZeppelin Defender Admin Contract New Pause Proposal](/images/openzeppelin/ozdefender-images4.png)

Откроется страница предложения, где необходимо заполнить все детали, касающиеся предложения. В данном примере вам необходимо предоставить следующую информацию:

 1. Адрес учетной записи администратора. Вы также можете оставить это поле пустым, если хотите запустить действие с вашего текущего кошелька (если он имеет все необходимые разрешения).
 2. Наименование предложения
 3. Описание предложения. Здесь вы должны указать как можно больше подробностей для других участников/управляющих контрактом (если используется кошелек MultiSig)
 4. Нажмите " Create pause proposal"

![OpenZeppelin Defender Admin Contract Pause Proposal Details](/images/openzeppelin/ozdefender-images5.png)

После успешного создания предложения оно должно появиться в списке на панели администратора контракта.

![OpenZeppelin Defender Admin Contract Proposal List](/images/openzeppelin/ozdefender-images6.png)

### Одобрение предложения по контракту

После создания предложения по контракту следующим шагом будет его одобрение и исполнение. Для этого перейдите к предложению и нажмите " Approve and Execute". 

![OpenZeppelin Defender Admin Contract Proposal Pause Approve](/images/openzeppelin/ozdefender-images7.png)


Это инициирует транзакцию, которая должна быть подписана с помощью MetaMask, после чего состояние предложения должно измениться на " Executed ( подтверждение)". Как только транзакция будет обработана, статус должен показать " Executed".

![OpenZeppelin Defender Admin Contract Proposal Pause Executed](/images/openzeppelin/ozdefender-images8.png)

Вы также видите, что статус контракта изменился с " Running" на "Paused". Отлично! Теперь вы знаете, как использовать компонент Admin для управления смарт-контрактами. 
 
