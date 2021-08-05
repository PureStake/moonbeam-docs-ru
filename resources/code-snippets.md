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

=== "Moonbeam Development Node"

    - Network Name: `Moonbeam Dev`
    - RPC URL: `{{ networks.development.rpc_url }}`
    - ChainID: `{{ networks.development.chain_id }}`
    - Symbol (Optional): `DEV`
    - Block Explorer (Optional): `{{ networks.development.block_explorer }}`

=== "Moonbase Alpha TestNet"

    - Network Name: `Moonbase Alpha`
    - RPC URL: `{{ networks.moonbase.rpc_url }}`
    - ChainID: `{{ networks.moonbase.chain_id }}`
    - Symbol (Optional): `DEV`
    - Block Explorer (Optional): `{{ networks.moonbase.block_explorer }}`

=== "Moonriver"

    - Network Name: `Moonriver`
    - RPC URL: `{{ networks.moonriver.rpc_url }}`
    - ChainID: `{{ networks.moonriver.chain_id }}`
    - Symbol (Optional): `MOVR`
    - Block Explorer (Optional): `{{ networks.moonriver.block_explorer }}`