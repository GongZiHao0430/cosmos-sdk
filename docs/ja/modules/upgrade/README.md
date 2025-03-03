# `upgrade`

## 概要

`x / upgrade`は、スムーズに処理できるCosmosSDKモジュールの実装です。
ライブのCosmosチェーンを新しい（最新の）ソフトウェアバージョンにアップグレードします。 それはこれを達成します
ブロックチェーンステートマシンが
事前定義されたアップグレードブロックの高さに達したら続行します。

このモジュールは、ガバナンスがどのように実行することを決定するかについては何も規定していません。
アップグレードしますが、アップグレードを安全に調整するためのメカニズムにすぎません。 ソフトウェアなし
アップグレードのサポート、ライブチェーンのアップグレードは、すべてのバリデーターが危険であるためリスクがあります
プロセスのまったく同じポイントでステートマシンを一時停止する必要があります。 もしも
これは正しく行われていません。状態の不整合が発生する可能性があります。
から回復する。

<!-- コンテンツ -->
1. **[概念](01_concepts.md)**
2. **[状態](02_state.md)**
3. **[イベント](03_events.md)**
4. **[クライアント](04_client.md)**
    - [CLI](04_client.md#cli)
    - [REST](04_client.md#rest)
    - [gRPC](04_client.md#grpc) 