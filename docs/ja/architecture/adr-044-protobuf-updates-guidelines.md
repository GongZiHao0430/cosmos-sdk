# ADR 044:Protobuf定義更新ガイド

## 変更ログ

-28.06.2021:最初のドラフト
-02.12.2021:新しいフィールドに `Since:`コメントを追加

## 状態

下書き

## 概要

このADRは、Protobufの定義を更新する際のガイドラインと推奨プラクティスを提供します。これらのガイドラインは、モジュール開発者を対象としています。

## 環境

Cosmos SDKは、一連の[Protobuf定義](https://github.com/cosmos/cosmos-sdk/tree/master/proto/cosmos)を維持しています。同じバージョンでの破壊的な変更を避けるために、Protobuf定義を正しく設計することが重要です。その理由は、ツール(インデクサーやリソースマネージャーを含む)、ウォレット、その他のサードパーティの統合を壊さないためです。

これらのProtobuf定義を変更する場合、Cosmos SDKは現在、[Buf](https://docs.buf.build/)の推奨事項のみに従います。ただし、場合によっては、Bufの提案によってSDKに大きな変更が加えられる可能性があることに気づきました。例えば:

-`Msg`にフィールドを追加します。フィールドの追加は、Protobuf仕様によって破られる操作ではありません。ただし、 `Msg`に新しいフィールドを追加すると、不明なフィールドの拒否により、新しい` Msg`が古いノードに送信されるときにエラーがスローされます。
-フィールドを「予約済み」としてマークします。 Protobufは、パッケージのバージョンを変更せずにフィールドを削除するには、 `reserved`キーワードを使用することをお勧めします。ただし、Protobufは「予約済み」フィールドに対して何も生成しないため、これを行うと、クライアントの下位互換性が失われます。この問題の詳細については、[#9446](https://github.com/cosmos/cosmos-sdk/issues/9446)を参照してください。

さらに、モジュール開発者は、「フィールドの名前を変更できますか？」や「フィールドを破棄できますか？」など、Protobufの定義に関する他の質問に直面することがよくあります。これらすべての質問に答えてください。

## 決定

次の場合を除いて、[Buf](https://docs.buf.build/)の提案を保持することにしました。

-`UNARY_RPC`:Cosmos SDKは現在、ストリーミングRPCをサポートしていません。
-`COMMENT_FIELD`:Cosmos SDKでは、コメントなしのフィールドを使用できます。
-`SERVICE_SUFFIX`: `Query`および` Msg`サービスの命名規則を使用し、 `-Service`サフィックスは使用しません。
-`PACKAGE_VERSION_SUFFIX`: `cosmos.crypto.ed25519`などの一部のパッケージは、バージョンサフィックスを使用しません。
-`RPC_REQUEST_STANDARD_NAME`: `Msg`サービスへのリクエストには、下位互換性を維持するための` -Request`サフィックスがありません。

Bufの提案に加えて、次のCosmosSDK固有のガイドを追加しました。

### 衝突バージョンなしでProtobuf定義を更新

#### 1.`Msg`sは新しいフィールドを持つことができません

`Msg`を処理する場合、Cosmos SDKの前処理手順は厳密であり、` Msg`で不明なフィールドを使用することはできません。これは、[`codec .unknownproto`パッケージ](https://github.com/cosmos/cosmos-sdk/blob/master/codec/unknownproto)の不明なフィールドを拒否することでチェックされます。

ここで、v0.43ノードが `MsgExample`トランザクションを受け入れ、v0.44で、チェーン開発者が` MsgExample`にフィールドを追加することを決定したと想像してください。 Protobuf定義のみを操作するクライアント開発者は、 `MsgExample`に新しいフィールドがあることを確認し、それを入力します。ただし、新しい「MsgExample」を古いv0.43ノードに送信すると、フィールドが不明であるため、v0.43ノードは「MsgExample」を拒否します。同じProtobufバージョンが複数のノードバージョンで使用できるという期待を保証する必要があります。 

このため、モジュール開発者は既存の `Msg`に新しいフィールドを追加してはなりません。

これは、 `Msg`へのフィールドの追加を制限するのではなく、すべてのネストされた構造と` Msg`の `Any`へのフィールドの追加も制限することに注意してください。

#### 2. `Msg`に関係のないProtobuf定義には新しいフィールドがあるかもしれませんが、` since: `コメントを追加する必要があります

一方、モジュール開発者は、ストレージに格納されている「クエリ」サービスまたはオブジェクトに関連するProtobuf定義に新しいフィールドを追加できます。 この推奨事項はProtobuf仕様に従いますが、わかりやすくするためにこのドキュメントに追加されています。

SDKでは、新しいフィールドのProtobufコメントに次の形式の1行が含まれている必要があります。 

```protobuf
/.Since: cosmos-sdk <version>{, <version>...}
```

各 `version`は、フィールドで使用可能なマイナー(" 0.45 ")またはパッチ(" 0.44.5 ")バージョンを表します。 これはクライアントライブラリに大いに役立ちます。クライアントライブラリは、リフレクションまたはカスタムコード生成を使用して、ターゲットノードのバージョンに基づいてこれらのフィールドを表示/非表示にすることを選択できます。

たとえば、次のコメントが有効です。

```protobuf
/.Since: cosmos-sdk 0.44

/.Since: cosmos-sdk 0.42.11, 0.44.5
```

and the following ones are NOT valid:

```protobuf
/.Since cosmos-sdk v0.44

/.since: cosmos-sdk 0.44

/.Since: cosmos-sdk 0.42.11 0.44.5

/.Since: Cosmos SDK 0.42.11, 0.44.5
```

#### 3.フィールドは「非推奨」としてマークでき、ノードはこれらのフィールドを処理するためにプロトコルに破壊的な変更を実装できます

Protobufは[`deprecated`フィールドオプション](https://developers.google.com/protocol-buffers/docs/proto#options)をサポートしており、このオプションは` Msg`フィールドを含むすべてのフィールドに使用できます。ノードが空でない非推奨フィールドを使用してProtobufメッセージを処理する場合、プロトコルに違反する方法であっても、ノードはメッセージの処理中に動作を変更する可能性があります。可能であれば、ノードはコンセンサスを壊すことなく下位互換性を処理する必要があります(プロトタイプバージョンを増やす場合を除く)。

たとえば、Cosmos SDK v0.42からv0.43へのアップデートには、以下に示すProtobufに対する2つの重大な変更が含まれています。 SDKチームはパッケージバージョンを `v1beta1`から` v1`にアップグレードしませんでしたが、重大な変更を元に戻し、これらの変更を非推奨としてマークし、非推奨フィールドを処理することにより、このガイドラインに従うことにしました。メッセージ中にノードの実装を変更します。さらに:

-Cosmos SDKは最近、[Time-Based Software Upgrade](https://github.com/cosmos/cosmos-sdk/pull/8849)のサポートをキャンセルしました。したがって、「time」フィールドは「cosmos.upgrade.v1beta1.Plan」で非推奨としてマークされています。さらに、ノードは、空でない「時間」フィールドを含むアップグレード計画の提案を拒否します。
-Cosmos SDKは、[Governance Split Voting](..adr-037-gov-split-vote.md)をサポートするようになりました。投票をクエリする場合、返された `cosmos.gov.v1beta1.Vote`メッセージの` option`フィールド(1投票オプションの場合)は非推奨になり、その `options`フィールド(複数の投票オプションを許可)に置き換えられます。可能な場合は常に、SDKは非推奨の `option`フィールドに入力します。つまり、` len(options)== 1`および `options [0] .Weight == 1.0`の場合に限ります。

#### 4。フィールドの名前を変更することはできません

公式のProtobufは、Protobufバイナリ表現を破壊しないため、フィールドの名前変更を禁止しないことを推奨していますが、SDKはProtobuf構造内のフィールドの名前変更を明示的に禁止しています。この選択の主な理由は、クライアントに破壊的な変更を導入しないようにすることです。これは通常、生成タイプのハードコードされたフィールドに依存します。さらに、フィールドの名前を変更すると、Protobufによって定義されたクライアントがRESTエンドポイントとCLIのJSON表現を壊します。

### Protobufパッケージバージョンをインクリメント

TODO、アーキテクチャのレビューが必要です。いくつかのトピック:

-衝突バージョンの頻度
-バージョンを衝突させる場合、Cosmos SDKは2つのバージョンをサポートする必要がありますか？
  -つまり、v1beta1-> v1であり、Cosmos SDKに2つのフォルダーと、2つのバージョンのハンドラーが必要ですか？
-メンションADR-023Protobufの命名

## 結果

>このセクションでは、決定を適用した後の結果コンテキストについて説明します。 「ポジティブ」な結果だけでなく、すべての結果をここにリストする必要があります。特定の決定は、ポジティブ、ネガティブ、ニュートラルな結果をもたらす可能性がありますが、これらはすべて、将来のチームやプロジェクトに影響を及ぼします。

### 下位互換性

>後方非互換性を導入するすべてのADRには、非互換性とその重大度を説明するセクションを含める必要があります。 ADRは、作成者がこれらの非互換性にどのように対処するつもりかを説明する必要があります。十分な下位互換性に関する書類がないADRの提出は、完全に拒否される可能性があります。

### ポジティブ

-ツール開発者の苦痛を軽減します
-エコシステムの互換性の向上
-..。

### ネガティブ

{否定的な結果}

### ニュートラル

-Protobufレビューの厳密性

## さらなる議論

このADRはまだドラフト段階にあります。正しく行う方法を決定したら、「インクリメンタルProtobufパッケージバージョン」に入力します。

## テストケース[オプション]

コンセンサスの変更に影響を与えるADRの場合、実装されたテストケースは必須です。該当する場合、他のADRがテストケースへのリンクを含めることを選択する場合があります。

## 参照する

-[#9445](https://github.com/cosmos/cosmos-sdk/issues/9445)プロト定義v1をリリース
-[#9446](https://github.com/cosmos/cosmos-sdk/issues/9446)v1beta1プロトの主要な変更を解決します 