# ADR 028:公開鍵アドレス

## 変更ログ

-2020/08/18:初期バージョン
-2021 .01/15:分析とアルゴリズムの更新

## 状態

提案

## 概要

このADRは、アドレス可能なすべてのCosmosSDKアカウントのアドレス形式を定義します。これには、新しい公開鍵アルゴリズム、マルチ署名公開鍵、およびモジュールアカウントが含まれます。

## 環境

問題[\#3685](https://github.com/cosmos/cosmos-sdk/issues/3685)で公開鍵が確認されました
アドレス空間は現在重複しています。 CosmosSDKのセキュリティが大幅に低下することを確認しました。

### 問題

攻撃者はアドレス生成機能の入力を制御できます。これは誕生日攻撃につながる可能性があり、セキュリティスペースが大幅に減少します。
この問題を克服するには、さまざまな種類の口座の入力を分離する必要があります。
1つのアカウントタイプのセキュリティの中断は、他のアカウントタイプのセキュリティに影響を与えるべきではありません。

### 予備的な提案

最初の提案は、住所の長さを延長し、
さまざまなタイプのアドレスにプレフィックスを追加します。

@ethanfreyは、https://github.com/iov-one/weaveで最初に使用された代替方法について説明しました。

>ウィーブを構築するときに、これについて考えることに多くの時間を費やしました...別のユニバースSDK。
>基本的に、条件をタイプとフォーマット、人間が読める文字列として定義し、いくつかのバイナリデータを追加します。この条件は、アドレス(これも20バイト)にハッシュされます。このプレフィックスを使用すると、条件が異なる特定のアドレスのプレイメージを見つけることができなくなります(たとえば、ed25519とsecp256k1)。
>ここに詳細な説明がありますhttps://weave.readthedocs.io/en/latest/design/permissions.html
>そして、コードはここにあり、主に処理条件の最上位にあります。 https://github.com/iov-one/weave/blob/master/conditions.go

また、この方法がどのように衝突に対して十分に耐性があるべきかについても説明します。

>はい、AFAIK、元の画像が一意で拡張不可能な場合、20バイトは衝突耐性が必要です。 2 ^ 160スペースでは、2 ^ 80要素の近くで衝突が発生すると予想されます(誕生日のパラドックス)。データベース内の既存の要素の競合を見つけたい場合でも、2 ^ 160です。 2 ^ 80これらすべての要素が状態で書き込まれる場合のみ。
>あなたの良い例は例えばです。公開鍵バイトは、コーデックでサポートされている2つのアルゴリズムの有効な公開鍵です。これは、それらのいずれかが侵害された場合、それらがより安全な亜種によって保護されていても、アカウントを侵害することを意味します。これは、元の画像に識別可能なタイプ情報がない場合にのみ問題になります(アドレスにハッシュする前)。
> 20バイトのスペースが実際のセキュリティの問題である場合、編み物中にアドレスサイズを増やして喜んでいるので、議論を聞きたいと思います。コスモス、イーサリアム、ビットコインはすべて20バイトを使用していると思いますが、これで十分です。そして、上記の議論は私にそれが安全であると感じさせます。しかし、私はこれ以上詳細な分析はしませんでした。

これが最初の提案につながりました(私たちはそれが十分ではないことを証明しました):
キータイプを公開キーと連結し、ハッシュしてハッシュの最初の20バイトを取得し、合計して `sha256(keyTypePrefix || keybytes)[:20]`とします。

### レビューとディスカッション

[\#5694](https://github.com/cosmos/cosmos-sdk/issues/5694)では、さまざまなソリューションについて説明しました。
20バイトは将来の証拠ではなく、アドレス長を延長することが、さまざまなタイプ、さまざまな署名タイプなどのアドレスを許可する唯一の方法であることに同意します。
これにより、元の提案は失格となりました。

この質問では、さまざまな変更について説明しました。

+ハッシュ関数の選択。
+プレフィックスをハッシュ関数から移動します: `keyTypePrefix + sha256(keybytes)[:20]` [post-hash-prefix-proposal]。
+ダブルハッシュを使用します: `sha256(keyTypePrefix + sha256(keybytes)[:20])`。
+キーバイトのハッシュスライスを20バイトから32バイトまたは40バイトに増やします。優れたハッシュ関数によって生成された32バイトは、将来的に安全であると結論付けました。

### 要件

+現在使用されているツールをサポートする-エコシステムに損害を与えたり、適応期間を長くしたりしたくありません。参照:https://github.com/cosmos/cosmos-sdk/issues/8041
+アドレスの長さをできるだけ短くするようにしてください。アドレスは、キーとオブジェクトの値の一部として状態で広く使用されています。

### スコープ

ADRは、アドレスバイトを生成するためのプロセスのみを定義します。エンドユーザーとアドレス間の対話(APIやCLIなどを介した)では、引き続きbech32を使用してこれらのアドレスを文字列にフォーマットします。 ADRはこれを変更しません。
文字列エンコーディングにBech32を使用すると、チェックサムエラーコードとユーザースペルエラーの処理がサポートされます。

## 決定

次のアカウントタイプを定義し、それらのアドレス関数を定義しました。

1.単純なアカウント:共通の公開鍵で表されます(例:secp256k1、sr25519)
2. naive multisig:他のアドレス可能なオブジェクトで構成されるアカウント(例:naive multisig)
3.ローカルアドレスキーの組み合わせアカウント(例:bls、グループモジュールアカウント)を使用します
4.モジュールアカウント:基本的に、トランザクションに署名できず、モジュールによって内部的に管理されるすべてのアカウント 

###レガシー公開鍵アドレスは変更されません

現在(2021年1月)、公式にサポートされているCosmos SDKユーザーアカウントは、「secp256k1」基本アカウントと従来のアミ​​ノマルチシグニチャのみです。
これらは、既存のCosmosSDKエリアで使用されます。次のアドレス形式を使用します。

-secp256k1: `ripemd160(sha256(pk_bytes))[:20]`
-従来のアミ​​ノマルチ署名: `sha256(aminoCdc.Marshal(pk))[:20]`

既存の住所を変更したくありません。したがって、これら2つのキータイプのアドレスは変更されません。

現在のマルチ署名公開鍵は、アミノシリアル化を使用してアドレスを生成します。維持します
それらの公開鍵とそのアドレス形式、およびそれらを「従来のアミ​​ノ」マルチ署名公開鍵と呼びます
protobufで。また、以下に説明するように、アミノアドレスのないマルチ署名公開鍵を作成します。

###ハッシュ関数の選択

Cosmos SDKの他の部分と同様に、sha256を使用します。

### ベースアドレス

まず、アドレスの生成に使用される基本的なハッシュアルゴリズムを定義します。単一のキーペアで表されるアカウントに使用されることは注目に値します。公開鍵モードごとに、関連付けられた `typ`文字列が必要です。これについては、次のセクションで説明します。 `hash`は、前のセクションで定義した暗号化ハッシュ関数です。  

```go
const A_LEN = 32

func Hash(typ string, key []byte) []byte {
    return hash(hash(typ) + key)[:A_LEN]
}
```

`+`は、区切り文字のないバイト連結です。

アルゴリズムは、プロの暗号学者との協議の結果です。
動機:アルゴリズムはアドレスを比較的小さく保ちます( `typ`の長さは最終的なアドレスの長さに影響しません)
また、[post-hash-prefix-proposal]よりも安全です(公開鍵ハッシュの最初の20バイトを使用して、アドレス空間を大幅に削減します)。
さらに、暗号技術者は、スイッチテーブル攻撃を防ぐためにハッシュに「typ」を追加することを選択するよう促しました。

`address.Hash`関数を使用して、単一のキーで表されるすべてのアカウントのアドレスを生成します。

*単純な公開鍵: `address.Hash(keyType、pubkey)`

+集約キー(例:BLS): `address.Hash(keyType、aggregatedPubKey)`
+モジュール: `address.Hash(" module "、moduleName)`

### 結合されたアドレス

単純な組み合わせアカウント(新しい単純なマルチ署名など)の場合、「address.Hash」を一般化します。 アドレスは、サブアカウントのアドレスを再帰的に作成し、アドレスを並べ替えて、それらを1つのアドレスに結合することによって作成されます。 キーの順序が結果のアドレスに影響を与えないようにします。 

```go
/.We don't need a PubKey interface - we need anything which is addressable.
type Addressable interface {
    Address() []byte
}

func Composed(typ string, subaccounts []Addressable) []byte {
    addresses = map(subaccounts, \a -> LengthPrefix(a.Address()))
    addresses = sort(addresses)
    return address.Hash(typ, addresses[0] + ... + addresses[n])
}
```

`typ`パラメータは、決定論的なシリアル化を伴うすべての重要な属性を含むパターン記述子である必要があります(例:utf8文字列)。
`LengthPrefix`は、アドレスの1バイト前に追加する関数です。 このバイトの値は、フロントの前のアドレスビットの長さです。 アドレスの長さは最大255ビットです。
競合を排除するために `LengthPrefix`を使用します-2つのアドレスリストに対して` as =(a1、a2、...、an} `と` bs =(b1、b2、...、bm) `すべての `bi`と` ai`の長さは最大255です。`concatenate(map(as、\ a-> LengthPrefix(a)))= map(bs、\ b-> LengthPrefix(b)) `iff` as = bs`。

実装のヒント:アカウントの実装では、アドレスをキャッシュする必要があります。 

#### マルチシグニチャアドレス

新しいマルチ署名公開鍵では、エンコードスキーム(aminoまたはprotobuf)に基づかない `typ`パラメーターを定義します。 これにより、コーディングスキームの不確実性の問題が回避されます。

例:

```proto
package cosmos.crypto.multisig;

message PubKey {
  uint32 threshold = 1;
  repeated google.protobuf.Any pubkeys = 2;
}
```

```go
func (multisig PubKey) Address() {
 ..first gather all nested pub keys
  var keys []address.Addressable ..cryptotypes.PubKey implements Addressable
  for _, _key := range multisig.Pubkeys {
    keys = append(keys, key.GetCachedValue().(cryptotypes.PubKey))
  }

 ..form the type from the message name (cosmos.crypto.multisig.PubKey) and the threshold joined together
  prefix := fmt.Sprintf("%s/%d", proto.MessageName(multisig), multisig.Threshold)

 ..use the Composed function defined above
  return address.Composed(prefix, keys)
}
```

#### モジュールアカウントアドレス

注:このセクションはまだ完了しておらず、活発に議論されています。

基本アドレスの部分では、モジュールアカウントアドレスを次のように定義します。

```go
address.Hash("module", moduleName)
```

すべてのモジュール派生アドレスのパターンタイプとして「モジュール」を使用します。 モジュールアカウントはサブアカウントを持つことができます。 派生プロセスには、モジュール名、サブモジュールキー、サブサブモジュールキーの定義されたシーケンスがあります。
モジュールアカウントアドレスはCosmosSDKで広く使用されているため、派生プロセスを最適化することは理にかなっています。モジュール名として `LengthPrefix`ではなく、区切り文字としてnullバイト(` '\ x00'`)を使用します。 nullバイトは有効なモジュール名の一部ではないため、これは有効です。 

```go
func Module(moduleName string, key []byte) []byte{
	return Hash("module", []byte(moduleName) + 0 + key)
}
```

**Example**  A lending BTC pool address would be:

```
btcPool := address.Module("lending", btc.Addrress()})
```

複数のキーに基づいてモジュールアカウントのアドレスを作成する場合は、それらを接続できます。 

```
btcAtomAMM := address.Module("amm", btc.Addrress() + atom.Address()})
```

#### 派生アドレス

パスワードを使用して、別のアドレスからアドレスを取得できる必要があります。 派生プロセスはハッシュ属性を保証する必要があるため、すでに定義されている `Hash`関数を使用します。

```go
func Derive(address []byte, derivationKey []byte) []byte {
    return Hash(addres, derivationKey)
}
```

注: `モジュール`は、より普通家族_derivative_稼の特別なケースであり、_fromaddress_に文字列 `"モジュール "`を設定します。

**例** cosmwasmの場合、時間の構造であるスマートコントラクトを使用します。  

```
smartContractAddr := Derived(Module("cosmwasm", smartContractsNamespace), []{smartContractKey})
```

### スキーマタイプ

`Hash`関数で使用される` typ`パラメータは、アカウントタイプごとに一意である必要があります。
すべてのCosmosSDKアカウントタイプは状態でシリアル化されるため、protobufメッセージ名文字列を使用することをお勧めします。

例:すべての公開鍵タイプには、次のような一意のprotobufメッセージタイプがあります。 

```proto
package cosmos.crypto.sr25519;

message PubKey {
  bytes key = 1;
}
```

すべてのprotobufメッセージには、一意の完全修飾名があります。この例では、「cosmos.crypto.sr25519.PubKey」です。
これらの名前は、標準化された方法で.protoファイルから直接派生して使用されます
`Any`sのタイプURLなどの他の場所。次の方法で簡単に名前を取得できます
`proto.MessageName(msg)`。

## 結果

###下位互換性

このADRは、Cosmos SDKリポジトリで送信され、直接サポートされているコンテンツと互換性があります。

### ポジティブ

-新しい公開鍵、複雑なアカウント、モジュールのアドレスを生成するためのシンプルなアルゴリズム
-アルゴリズムは_ネイティブキーの組み合わせ_を要約します
-アドレスのセキュリティと衝突防止を改善します
-この方法は、将来のユースケースに拡張できます。-ここで指定されたアドレス長(20または32バイト)と競合しない限り、他のアドレスタイプを使用できます。
-新しいアカウントタイプをサポートします。

### ネガティブ

-アドレスはキータイプを伝達しません。プレフィックスメソッドはこれを実行できます
-アドレスが60％長くなり、より多くのストレージスペースを消費します
-可変長アドレスを処理するためにKVStoreストレージキーを再構築する必要があります

### ニュートラル

-protobufメッセージ名はキータイププレフィックスとして使用されます

##さらなる議論

一部のアカウントには固定名を付けることも、他の方法で作成することもできます(例:モジュール)。組織で使用できる事前定義された名前(例: `me.regen`)を持つアカウントのアイデアについて話し合っています。
言うまでもなく、これらのタイプのアドレスは、長さが異なる限り、ここで説明するハッシュベースのアドレスと互換性があります。
より具体的には、特別なアカウントアドレスの長さは20バイトまたは32バイトに等しくてはなりません。

##付録:コンサルティングセッション

2020年12月末に、[Alan Szepieniec](https://scholar.google.be/citations?user=4LyZn8oAAAAJ&hl=en)とミーティングを行い、上記の方法について相談しました。

アランの一般的なコメント:

+2つのプリイメージ抵抗は必要ありません
+衝突に抵抗するために32バイトのアドレス空間が必要です
+攻撃者がアドレスを介してオブジェクトの入力を制御できる場合、誕生日攻撃の問題が発生します
+ハッシュに使用されるスマートコントラクトに問題があります
+ sha2マイニングを使用して、元のアドレスを解読できます

ハッシュアルゴリズム

+ blake3を破壊する攻撃は、blake2を破壊します
+ Alanは、ブレイクハッシュアルゴリズムの現在のセキュリティ分析に非常に自信を持っています。決勝戦の最終候補に挙げられた著者は、証券分析でよく知られています。

アルゴリズム:

+ Alanは、プレフィックスをハッシュすることを提案しています: `address(pub_key)= hash(hash(key_type)+ pub_key)[:32]`、主な利点:
    +任意の長いプレフィックス名を自由に使用できます
    +私たちはまだ衝突の危険を冒しません
    +スイッチテーブル
+罰についての議論->プレフィックスを追加した後のハッシュについて
+ Aaronは、ハッシュ後のプレフィックス( `address(pub_key)= key_type + hash(pub_key)`)とその違いについて質問しました。 Alanは、この方法はアドレススペースが長く、より強力であると指摘しました。

複雑な/結合されたキーのアルゴリズム:

+同じアルゴリズムを使用して類似したアドレスを持つツリーをマージするのは良いことです

モジュールアドレス:モジュールアドレスは、それらを区別するために異なるサイズにする必要がありますか？

+モジュールアドレスのプレイメージプレフィックスを設定して、32バイトのスペースに保持する必要があります: `hash(hash( 'module')+ module_key)`
+アーロンの観察:(secp256k1キーを破壊しないために)可変長を処理する必要があります。

ZKP算術ハッシュ関数に関する議論

+ポセイドン/レスキュー
+問題:暗号解読技術と算術構造の歴史についてほとんど知らないため、リスクははるかに大きくなります。これはまだ新しい分野であり、活発な研究の分野です。

ポスト量子署名のサイズ

+アランの提案:ファルコン:速度/体のサイズの比率-とても良い。
+アーロン-私たちはそれについて考えるべきですか？
  アラン:初期の推論によると、これは2050年にEC暗号を解読できるようになるでしょう。しかし、多くの不確実性があります。しかし、魔法のようなことが再帰/リンク/シミュレーションで起こり、進行をスピードアップすることができます。

その他のアイデア

+2つの異なるユースケースに同じキーと2つの異なるアドレスアルゴリズムを使用するとします。それでも安全に使用できますか？アラン:公開鍵を非表示にしたい場合(これは私たちのユースケースではありません)、そのセキュリティは低下しますが、それを修正する方法があります。

### 参照

+ [メモ](https://hackmd.io/_NGWI4xZSbKzj1BkCqyZMw) 