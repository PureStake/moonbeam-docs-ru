---
title: Скомпилируйте бинарный файл
description: Как скомпилировать бинарный файл Moonbeam для запуска полного узла Parachain, получения доступа к конечным точкам RPC и создания блоков для сети Moonbeam.
---

# Скомпилируйте бинарный файл Moonbeam

![Full Node Moonbeam Banner](/images/node-operators/networks/compile-binary/compile-binary-banner.png)

## Вступление {: #introduction } 

Есть несколько способов запустить полный узел в сети Moonbeam. В этом руководстве рассматривается процесс компиляции бинарного файла Moonbeam из исходного кода Rust. Чтобы получить более общий обзор работающих узлов или начать работу с Docker, ознакомьтесь с [Запуск Ноды](/node-operators/networks/full-node) страница нашей документации.

Это руководство предназначено для людей с опытом составления [Substrate](https://substrate.dev/) на основе узлов блокчейна. Узел парачейна похож на типичный узел Substrate, но есть некоторые отличия. Узел парачейна Substrate будет более крупной сборкой, потому что он содержит код для запуска самого парачейна, а также код для синхронизации цепочки реле и облегчения связи между ними. Таким образом, эта сборка довольно велика, может занять более 30 минут и потребовать 32 ГБ памяти.

## Компиляция бинарного файла {: #compiling-the-binary } 

Следующие команды построят последнюю версию парачейна Moonbeam.

Во-первых, давайте начнем с клонирования репозитория Moonbeam.

```
git clone https://github.com/PureStake/moonbeam
cd moonbeam
```

Давайте посмотрим на последний выпуск:

```
git checkout tags/$(git tag | tail -1)
```

Затем установите среду разработки Substrate, включая Rust, выполнив:

```
--8<-- 'code/setting-up-node/substrate.md'
```

Наконец, создайте бинарный файл парачейна:

```
cargo build --release
```

![Компиляция бинарого файла](/images/node-operators/networks/compile-binary/compile-binary-1.png)

Если в терминале появляется _cargo not found error_, вручную добавьте Rust в системный путь или перезапустите систему:

```
--8<-- 'code/setting-up-node/cargoerror.md'
```

Теперь вы можете использовать бинарный файл Moonbeam для [запуска службы systemd](/node-operators/networks/full-node/#running-the-systemd-service).
