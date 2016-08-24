## 【演習】独自サービスのService Broker

[はじめてのService Broker](first-service-broker.md)で、Service Brokerの作り方の基礎を学びました。
学んだ内容を使って、次は既存のサービスに対するService Brokerを作成しましょう。

ここでは仮想既存サービスとして、簡易KVS実装である[Map Server](../code/map-server)を使用します。


#### 【演習1】 Map ServerのAPIを把握する

Map Serverは次のようなマルチテナント構成になっています。

```
[Spaces] -+- [Space] -+- [Map] -+- [Key] - [Value]
          |          |          +- [Key] - [Value]
          |          |          `- [Key] - [Value]
          |           `- [Users] -+- [User]
          |                       +- [User]
          |                       `- [User]
          +- [Space] -+- ...
          `- [Space] -+- ...
```

`map-server`の`README`を参考に、`map-server`をCloud Foundryにデプロイし、
以下のAPIを`curl`で実行してください。

* Create Space
* Delete Space
* Create User
* Delete User
* Put Key-Value
* Get Key-Value
* Delete Key-Value

#### 【演習2】 Map ServerのService Brokerを実装する

[Map Server Sevrice Broker](../code/map-server-service-broker)の`ServiceInstanceService`、`ServiceInstanceBindingService`を実装してください。

TODO:01~07を埋める形になっています。

#### 【演習3】Map Server Service Brokerをデプロイする

`map-server`の`README`を参考に[Map Server Sevrice Broker](../code/map-server-service-broker)をデプロイしてください。

またexampleフォルダにあるサンプルアプリケーションからMap Serverにアクセスしてください。
