---
title: Полноценная Нода Moonriver на MacOS
---

# Фрагменты кода Коллатор/Полноценная нода на MacOS

## Полноценная Нода Moonbase Alpha {: #moonbase-alpha-full-node } 

```
docker run -p 9933:9933 -p 9944:9944 -v "{{ networks.moonbase.node_directory }}:/data" \
-u $(id -u ${USER}):$(id -g ${USER}) \
purestake/moonbeam:{{ networks.moonbase.parachain_release_tag }} \
--base-path=/data \
--chain {{ networks.moonbase.chain_spec }} \
--name="YOUR-NODE-NAME" \
--execution wasm \
--wasm-execution compiled \
--pruning archive \
--state-cache-size 1 \
-- \
--pruning archive \
--name="YOUR-NODE-NAME (Embedded Relay)"
```

## Коллатор Moonbase Alpha {: #moonbase-alpha-collator } 

```
docker run -p 9933:9933 -p 9944:9944 -v "{{ networks.moonbase.node_directory }}:/data" \
-u $(id -u ${USER}):$(id -g ${USER}) \
purestake/moonbeam:{{ networks.moonbase.parachain_release_tag }} \
--base-path=/data \
--chain {{ networks.moonbase.chain_spec }} \
--name="YOUR-NODE-NAME" \
--validator \
--execution wasm \
--wasm-execution compiled \
--pruning archive \
--state-cache-size 1 \
-- \
--pruning archive \
--name="YOUR-NODE-NAME (Embedded Relay)"
```

## Полноценная Нода Moonriver {: #moonriver-full-node } 

```
docker run -p 9933:9933 -p 9944:9944 -v "{{ networks.moonriver.node_directory }}:/data" \
-u $(id -u ${USER}):$(id -g ${USER}) \
purestake/moonbeam:{{ networks.moonriver.parachain_release_tag }} \
--base-path=/data \
--chain {{ networks.moonriver.chain_spec }} \
--name="YOUR-NODE-NAME" \
--execution wasm \
--wasm-execution compiled \
--pruning archive \
--state-cache-size 1 \
-- \
--pruning archive \
--name="YOUR-NODE-NAME (Embedded Relay)"
```

## Коллатор Moonriver {: #moonriver-collator } 

```
docker run -p 9933:9933 -p 9944:9944 -v "{{ networks.moonriver.node_directory }}:/data" \
-u $(id -u ${USER}):$(id -g ${USER}) \
purestake/moonbeam:{{ networks.moonriver.parachain_release_tag }} \
--base-path=/data \
--chain {{ networks.moonriver.chain_spec }} \
--name="YOUR-NODE-NAME" \
--validator \
--execution wasm \
--wasm-execution compiled \
--pruning archive \
--state-cache-size 1 \
-- \
--pruning archive \
--name="YOUR-NODE-NAME (Embedded Relay)"
```
