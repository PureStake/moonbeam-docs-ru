---
title: Фрагменты кода
description: Для того чтобы было легче начать работу с Moonbeam, здесь приведены фрагменты кода для каждого из созданных нами учебных пособий.
---

# Фрагменты кода

## Настройка локальной ноды Moonbeam {: #setting-up-a-local-moonbeam-node } 

**Выполните клонирование “moonbeam-tutorials” репозитория:**

```
git clone -b {{ networks.development.build_tag }} https://github.com/PureStake/moonbeam
cd moonbeam
```

**Выполните установку substrate и необходимых компонентов для его правильной работы:**

```
--8<-- 'code/setting-up-node/substrate.md'
```

**Укажите системный путь для утилиты Rust:**

```
--8<-- 'code/setting-up-node/cargoerror.md'
```

**Создайте автономную ноду:**

```
--8<-- 'code/setting-up-node/build.md'
```

**Запустите ноду в “dev” режиме:**

```
--8<-- 'code/setting-up-node/runnode.md'
```

**Очистите цепь, удалите старые данные запуска ноды в ‘dev’ режиме которые могли остаться с прошлого раза:**

```
./target/release/moonbeam-development purge-chain --dev
```

**Запустите ноду в ‘dev’ режиме, скрывая информацию о блоках и выводя на экран только ошибки:**

```
./target/release/moonbeam-development --dev -lerror
```

## Учетная запись Genesis {: #genesis-account } 

--8<-- 'text/metamask-local/dev-account.md'

## Учетная запись Development {: #development-accounts } 

--8<-- 'code/setting-up-node/dev-accounts.md'

--8<-- 'code/setting-up-node/dev-testing-account.md'

## MetaMask {: #metamask } 

**Детали ноды Moonbeam Development:**

--8<-- 'text/metamask-local/development-node-details.md'

**Moonbase Alpha TestNet:**

--8<-- 'text/testnet/testnet-details.md'
