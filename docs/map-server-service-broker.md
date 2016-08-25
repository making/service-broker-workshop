## 【演習】独自サービスのService Broker

[はじめてのService Broker](first-service-broker.md)で、Service Brokerの作り方の基礎を学びました。
学んだ内容を使って、次は既存のサービスに対するService Brokerを作成しましょう。

ここでは仮想既存サービスとして、REST APIでアクセスできる簡易KVS実装である[Map Server](https://github.com/Pivotal-Japan/map-server)を使用します。

この演習では、既存のMap Serverに対するService Brokerを実装して、サンプルアプリケーションからアクセスできるようにします。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/1b171dfe-ee21-dcbe-4d19-3489efc40fad.png)

#### 【演習1】 Map ServerのAPIを把握する

Map Serverは次のようなマルチテナント構成になっています。

```
<<Spaces>> -+- [Space] -+- <<Map>> -+- [Key] - [Value]
            |           |           +- [Key] - [Value]
            |           |           `- [Key] - [Value]
            |            `- <<Users>> -+- [User]
            |                          +- [User]
            |                          `- [User]
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

[Map Server Sevrice Broker](https://github.com/Pivotal-Japan/map-server-service-broker/tree/lab)の`ServiceInstanceService`、`ServiceInstanceBindingService`を実装してください。

`code/map-server-service-broker`をSTSにインポートしてください。「File」 -> 「Import」 -> 「Existing Maven Projects」

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/1e5d5d9c-cba8-11cb-6881-dbfaf753278e.png)

map-server-service-brokerフォルダを選択。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/083f6bbf-eb65-377c-ef75-f09fe401ba75.png)


「Window」 -> 「Show View」 -> 「Others」

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/5663b0c0-6f37-3d99-66d4-7c17f2182953.png)

「General」 -> 「Tasks」を選択してOKをクリック。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/0b99132f-b366-7569-850c-dcb52cc564ed.png)

TODO:01~07を埋める形になっています。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/0bbffa20-e98b-7092-3bd5-b1ed5e6d4177.png)

`MapServerClient`クラスに必要な処理が実装されているので、このクラスのメソッドを呼び出してください。

#### 【演習3】Map Server Service Brokerをデプロイする

`map-server`の`README`を参考に[Map Server Sevrice Broker](https://github.com/Pivotal-Japan/map-server-service-broker/tree/lab)をデプロイしてください。

またexampleフォルダにある[サンプルアプリケーション](https://github.com/Pivotal-Japan/map-server-service-broker/tree/lab/example)でデプロイして、Map Serverにアクセスしてください。


