# ADR 023:プロトコルバッファの命名とバージョン規則

## 変更ログ

-2020年4月27日:最初のドラフト
-2020年8月5日:ガイドラインを更新

## 状態

受け入れられました

## 環境

Protocol Buffersは、基本的な[スタイルガイド](https://developers.google.com/protocol-buffers/docs/style)を提供します
[Buf](https://buf.build/docs/style-guide)これに基づいています。到着
可能な場合は、業界で認められたガイドラインと知恵に従うことを望んでいます
protobufの効果的な使用は、明確な場合にのみそれらから逸脱します
ユースケースの基本原則。

### `Any`を採用する

推奨されるエンコード方法として「google.protobuf.Any」を使用してください
( `oneof`ではなく)インターフェースタイプにより、パッケージの命名がコア部分になります
完全修飾としてエンコードされたメッセージの名前がエンコードに表示されるようになりました
情報。

### 現在のディレクトリ編成

これまでのところ、主に[Buf's](https://buf.build)[DEFAULT](https://buf.build/docs/lint-checkers#default)に従います。
[`PACKAGE_DIRECTORY_MATCH`](https://buf.build/docs/lint-checkers#file_layout)の小さな偏差を無効にすることをお勧めします
コードを開発するのは便利ですが、警告があります
Bufから:

>そうしないと、さまざまな言語の多くのProtobufプラグインで非常に悪い時間を過ごすことになります

### gRPCを使用したクエリ

[ADR 021](adr-021-protobuf-query-encoding.md)では、ProtobufはgRPCを使用します
ローカルクエリ。したがって、完全なgRPCサービスパスはABCIクエリの重要な部分になっています
トレイル。将来的には、gRPCクエリが永続的なスクリプトで許可される可能性があります
CosmWasmなどのテクノロジーを通じて、これらのクエリルートは次の場所に保存されます。
スクリプトバイナリファイル。

## 決定

このADRの目標は、思慮深い命名規則を提供することです。

*ユーザーが直接操作するときに優れたユーザーエクスペリエンスを促進する
.protoファイルと完全修飾protobuf名
*単純さと過度の最適化の可能性のバランスを取ります(
名前が短すぎて不思議です)または最適化が不十分です(肥大化した名前のみが受け入れられます
冗長な情報がたくさんあります)

これらのガイドは、CosmosSDKおよび
サードパーティのモジュール。

開始点として、すべての[DEFAULT](https://buf.build/docs/lint-checkers#default)を使用する必要があります
[`PACKAGE_DIRECTORY_MATCH`](https://buf.build/docs/lint-checkers#file_layout)を含む[Buf's](https://buf.build)のチェッカー、
の他に: 

* [PACKAGE_VERSION_SUFFIX](https://buf.build/docs/lint-checkers#package_version_suffix)
* [SERVICE_SUFFIX](https://buf.build/docs/lint-checkers#service_suffix)

さらなるガイダンスを以下に説明する。

### 原則として

#### 簡潔でわかりやすい名前

名前は、その意味を伝え、区別するのに十分な説明的である必要があります
彼らは他の名前から来ています。

完全修飾名を使用したことを考えると
`google.protobuf.Any`とgRPCクエリルーティングでは、私たちの目標は
名前は簡潔に保ち、やりすぎないでください。一般的な経験則は
短い名前がより多くのまたは同じことを伝える場合は、短い名前を選択してください
名前。

たとえば、 `cosmos.bank.MsgSend`(19バイト)はほぼ同じ情報を伝達します
`cosmos_sdk.x.bank.v1.MsgSend`(28バイト)などですが、より簡潔です。

この簡潔さにより、名前が使いやすくなり、使用量が少なくなります。
取引とオンラインスペース。

また、名前を付けて過度に最適化する誘惑に抵抗する必要があります
略語付きのミステリアスなショート。たとえば、試してはいけません
`cosmos.bank.MsgSend`を` csm.bk.MSnd`に減らすことは、ほんの数バイトを節約することです。

目標は、名前を** _簡潔に、しかし神秘的ではない_ **にすることです。 

#### 名前はカスタマーサービス用です

パッケージとタイプの名前は、ユーザーの利益のために選択する必要があります。
これは、goコードベースに関連するレガシーの問題が原因である必要があります。

#### 長寿計画

長期的なサポートのために、私たちは私たちが作る名前を計画する必要があります
長期使用を選択するので、今が作成する機会です
将来のための最良の選択。

### バージョン管理

#### 安定したパッケージバージョンガイド

一般的に、モードの進化は、protobufモードを更新する方法です。これは新しい分野を意味し、
メッセージとRPCメソッドが既存のスキーマに追加され、古いフィールド、メッセージ、およびRPCメソッドが追加されます
できるだけ長く保管してください。

ブロックチェーンのシナリオでは、物事を壊すことは通常受け入れられません。たとえば、不変のスマートコントラクト
ホストチェーンの特定のデータパターンに依存する場合があります。ホストチェーンがこれらのパターンを破った場合、スマート
契約は不可逆的に破られる可能性があります。修正できる場合でも(たとえば、クライアントソフトウェアで)、
これは通常、高い価格を必要とします。

物事を破壊するのではなく、単に破壊するのではなく、モデルを開発するためにできる限りのことをする必要があります。
[Buf](https://buf.build)すべての安定した(非アルファまたはベータ)パッケージには、重大な変更の検出を使用する必要があります
そのような損傷を防ぐため。

これを念頭に置いて、パッケージのさまざまな安定したバージョン(つまり、 `v1`または` v2`)を多かれ少なかれ検討する必要があります
さまざまなパッケージで、これがprotobufモードをアップグレードする最後の手段になるはずです。作成したシーン
`v2`が意味をなすのは次のとおりです。

*既存のモジュールと同様の機能を持つ新しいモジュールを作成したいので、 `v2`を追加するのが最も自然です
これを行う方法。この場合、実際には、APIが異なる2つの異なるが類似したモジュールしかありません。
*既存のモジュールに新しく改善されたAPIを追加したいのですが、既存のパッケージに追加するのは面倒です。
したがって、 `v2`に入れる方がユーザーにとってよりクリーンです。この場合、の使用を放棄しないように注意する必要があります
不変のスマートコントラクトで積極的に使用されている場合は `v1`。

####不安定な(アルファおよびベータ)パッケージバージョンガイド

次のガイドラインに従って、パッケージをアルファまたはベータとしてマークすることをお勧めします。

*一部のコンテンツを `alpha`または` beta`としてマークすることは最後の手段であり、一部のコンテンツを
安定したパッケージ(つまり、 `v1`または` v2`)を優先する必要があります
*削除するアクティブなディスカッションがある場合にのみ、パッケージを「アルファ」としてマークする必要があります。
または、近い将来、パッケージを大幅に変更します
*活発な議論がある場合に限り、パッケージは「ベータ」としてマークされるべきです。
近い将来、大幅にリファクタリング/再設計された機能ですが、削除されることはありません
*モジュールは、安定した(つまり、 `v1`または` v2`)および不安定な( `alpha`または` beta`)パッケージタイプを持つことができます。

*互換性を維持する責任を回避するために、 `alpha`と` beta`を使用しないでください。 *
特にブロックチェーン上でコードが公開されるときはいつでも、物事を変更するコストは高くなります。いくつかの
たとえば、不変のスマートコントラクトの場合、主要な変更は修復できない場合があります。

何かを「アルファ」または「ベータ」としてマークする場合、メンテナは次の質問をする必要があります。

*他の人にコードを変更するように依頼するコストと、コードを変更するオプションを維持することの利点は何ですか？
*それを `v1`に移動する計画は何ですか？これはユーザーにどのように影響しますか？

`alpha`または` beta`は、実際には「計画された変更」を伝えるために使用する必要があります。

ケーススタディとして、gRPCリフレクションは `grpc.reflection.v1alpha`パッケージにあります。それ以来変わっていません
2017年には、gRPCurlなどの他の広く使用されているソフトウェアで使用されるようになりました。一部の人々はそれを生産サービスで使用するかもしれません
したがって、パッケージを `grpc.reflection.v1`に変更すると、一部のソフトウェアがクラッシュし、
彼らはそれをしたくないかもしれません...それで今 `v1alpha`パッケージは多かれ少なかれ実際の` v1`です。そんなことはしないでください。

以下は、不安定なパッケージを使用するためのガイドです。

* [Buf推奨バージョンのサフィックス](https://buf.build/docs/lint-checkers#package_version_suffix)
(例: `v1alpha1`)は不安定なパッケージに使用する必要があります
*不安定なパッケージは通常、破壊的な変更の検出から除外する必要があります
*不変のスマートコントラクトモジュール(つまり、CosmWasm)_should_blockスマートコントラクト/永続的
`alpha`.` beta`パッケージと相互作用するスクリプト

#### v1サフィックスを省略

[Buf推奨バージョンサフィックス](https://buf.build/docs/lint-checkers#package_version_suffix)を使用する代わりに、
実際に2番目のバージョンがないパッケージの場合、 `v1`を省略できます。この
`cosmos.bank.Send`などの一般的なユースケースのより簡潔な名前を許可します。
2番目または3番目のバージョンのパッケージは `.v2`で表すことができます
または `.v3`。

### パッケージの命名

#### 短くて一意のトップレベルのパッケージ名を使用する

トップレベルのパッケージでは、競合しないことがわかっている短い名前を使用する必要があります
コスモスエコシステムで一般的に使用される他の名前。近い将来、
使用される最上位のパッケージ名を保持および索引付けするために、レジストリを作成する必要があります
コスモスエコシステム。 CosmosSDKが提供することを目的としているため
Cosmosプロジェクトのトップレベルタイプ。トップレベルパッケージ名は `cosmos`です。
長い `cosmos_sdk`の代わりに、CosmosSDKで使用することをお勧めします。
[ICS](https://github.com/cosmos/ics)仕様は1つを考慮することができます
`ics23`などの標準番号に基づく短いトップレベルパッケージ。

#### サブパッケージの深さを制限する

下請けの深さを増すように注意する必要があります。一般的にシングル
モジュールまたはライブラリにはサブパッケージが必要です。 `x`または` modules`であっても
ソースコードでモジュールを表すために使用されます。これは通常、.protoには不要です。
モジュールとしてのファイルは、サブパッケージの主な目的です。それだけ
使用頻度が低いことがわかっているものは、サブパッケージの深さを深くする必要があります。

Cosmos SDKの場合、単に `cosmos.bank`と書くことをお勧めします。
`cosmos.x.bank`の代わりに` cosmos.gov`など。実際には、ほとんどの非モジュール
タイプは `cosmos`パッケージに直接配置することも、導入することもできます
必要に応じて、 `cosmos.base`をパッケージ化します。この命名は変更されないことに注意してください
goパッケージ名、つまり `cosmos.bank`protobufパッケージはまだ存在します
`x .bank`。

### メッセージの命名

メッセージタイプの名前は、わかりやすくするためにできるだけ簡潔にする必要があります。 `sdk.Msg`
トランザクションで使用されるタイプは、「Msg」プレフィックスを保持します。
便利なコンテキスト。

### サービスとRPCの命名

[ADR 021](adr-021-protobuf-query-encoding.md)は、モジュールが
gRPCクエリサービスを実現します。単純性の原則を考慮する必要があります
永続的なスクリプトから呼び出すことができるため、サービス名とRPC名のクエリに使用されます
CosmWasmおよびその他のモジュール。さらに、ユーザーは次のようなツールからこれらのクエリパスを使用できます。
[gRPCurl](https://github.com/fullstorydev/grpcurl)。たとえば、短縮できます
`.cosmos_sdk.x.bank.v1.QueryService .QueryBalance`から
`.cosmos.bank.Query .Balance`は多くの有用な情報を失うことはありません。

RPC要求および応答type_should_は `ServiceNameMethodNameRequest`.に従います
`ServiceNameMethodNameResponse`の命名規則。つまり、「Balance」という名前のRPCメソッドの場合
`Query`サービスでは、リクエストとレスポンスのタイプは` QueryBalanceRequest`になります
そして `QueryBalanceResponse`。これは、BalanceRequestよりも自明です。
そして「バランスの取れた反応」。

#### クエリサービスとしてのみ `Query`を使用する

[Bufのデフォルトのサービスサフィックスの推奨事項](https://github.com/cosmos/cosmos-sdk/pull/6033)の代わりに、
クエリサービスを提供するには、単に短い `Query`を使用する必要があります。

他のタイプのgRPCサービスについては、Bufに固執することを検討する必要があります
デフォルトで推奨されます。

#### クエリサービスのRPC名から `Get`と` Query`を省略します

`Get`と` Query`は `Query`サービス名から省略されるべきです。
完全修飾名では冗長です。たとえば、 `.cosmos.bank.Query .QueryBalance`
新しい情報なしで、「クエリ」を2回言うだけです。

## 将来の改善

ネーミングを調整するために、トップレベルのパッケージ名のレジストリを作成する必要があります
クロスエコロジカルで、競合を防ぎ、開発者が発見するのを助けます
便利なパターン。簡単な出発点はgitリポジトリです
コミュニティベースのガバナンス。
## 結果

### ポジティブ

*名前はより簡潔で読みやすく入力しやすくなります
* `Any`を使用するすべてのトランザクションが短縮されます(` _sdk.x`と `.v1`は削除されます)
* `.proto`ファイルのインポートはより標準的になります(` "third_party .proto" `はありません
道)
* .protoファイルが
単一の `proto.`ディレクトリでは、散在する代わりにコピーできます
CosmosSDK全体

### ネガティブ

### ニュートラル

* `.proto`ファイルを再編成して再構築する必要があります
*一部のモジュールはアルファまたはベータとしてマークする必要がある場合があります

## 参照 