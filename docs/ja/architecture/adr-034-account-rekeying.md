# ADR 034:アカウントのキーの再生成

## 変更ログ

2020年9月30日:最初のドラフト

## 状態

提案

## 概要

アカウントの再暗号化は、アカウントが認証用の公開鍵を新しい公開鍵に置き換えることを可能にするプロセスです。

## 環境

現在、Cosmos SDKでは、auth`BaseAccount`のアドレスは公開鍵のハッシュ値に基づいています。アカウントが作成された後、アカウントの公開鍵は不変であり、変更することはできません。キーローテーションは便利なセキュリティプラクティスであるため、これはユーザーにとって問題になる可能性がありますが、現在は不可能です。また、マルチ署名は一種の公開鍵であるため、アカウントのマルチ署名を設定すると更新できなくなります。組織や企業はマルチシグニチャを使用することが多く、内部的な理由により、マルチシグニチャセットを変更する必要がある場合があるため、これは問題があります。

アカウントの一部の「参加」は簡単に転送できないため、更新された公開鍵を使用してアカウントのすべての資産を新しいアカウントに転送するだけでは不十分です。たとえば、ステーキングでは、バインドされたAtomを転送するために、アカウントはすべての委任を解放し、3週間のバインド解除期間を待つ必要があります。さらに重要なことに、検証者のオペレーターの場合、検証者の所有権は基本的に譲渡できません。つまり、検証者のオペレーターキーを更新できないため、検証者の運用上のセキュリティが低下します。

## 決定

`x .auth`に新機能を追加することをお勧めします。これにより、アカウントは、アドレスを変更せずに、アカウントに関連付けられた公開鍵を更新できます。

これが可能なのは、Cosmos SDKの `BaseAccount`が、他のブロックチェーン(ビットコインなど)のように公開鍵がトランザクション(明示的または暗黙的)に含まれていると想定するのではなく、アカウントの公開鍵を状態で保存するためです。 )およびEthereum。公開鍵はチェーンに格納されているため、アドレスは署名チェックプロセスとは関係がないため、公開鍵がアカウントアドレスにハッシュされていない可能性があります。

このシステムを構築するために、次のように新しいMsgタイプを設計しました。 

```protobuf
service Msg {
    rpc ChangePubKey(MsgChangePubKey) returns (MsgChangePubKeyResponse);
}

message MsgChangePubKey {
  string address = 1;
  google.protobuf.Any pub_key = 2;
}

message MsgChangePubKeyResponse {}
```

MsgChangePubKeyトランザクションは、州内の既存の公開鍵によって署名される必要があります。

承認されると、このメッセージタイプのハンドラー(AccountKeeperを受け入れる)は、アカウントの状態公開鍵を更新し、メッセージ内の公開鍵に置き換えます。

公開鍵が変更されたアカウントは、状態から自動的に削除することはできません。 これは、プルーニングされた場合、アカウントの元の公開鍵で同じアドレスを再作成する必要がありますが、アドレスの所有者が元の公開鍵を所有していない可能性があるためです。 現在、アカウントを自動的にトリミングすることはありませんが、このオプションを妨げないようにします(これがアカウントの目的です)。 この問題を解決するために、この外部性を補うために、この操作に追加のガス料金を請求します(バインドされたガス量はパラメーター `PubKeyChangeCost`として構成されます)。 `ConsumeGas`関数を使用して、ハンドラー内の余分なガスを充電します。 さらに、将来的には、新しいMsgタイプ(「MsgDeleteAccount」など)を使用するアカウントが、セルフプルーニング用のキーを手動で再生成できるようにすることができます。 アカウントの手動プルーニングは、操作を実行するためのインセンティブとしてガスの払い戻しを提供できます。

```go
	amount := ak.GetParams(ctx).PubKeyChangeCost
	ctx.GasMeter().ConsumeGas(amount, "pubkey change fee")
```

アドレスのキーが変更されるたびに、この変更のログがチェーンの状態に保存され、アドレスの以前のすべてのキーとそれらがアクティブである時間間隔のスタックが作成されます。これにより、dappsとクライアントは、アカウントの過去のキーを簡単に照会できます。これは、タイムスタンプ付きのオフチェーン署名付きメッセージの検証などの機能に役立つ場合があります。

## 結果

### ポジティブ

*ユーザーと検証者のオペレーターは、キーローテーションを通じてより優れた運用セキュリティ慣行を採用できるようになります。
*組織またはグループがマルチ署名を簡単に変更および追加/削除できるようになります。

### ネガティブ

H(pubkey)= addressであるため、アドレスと公開鍵の間で現在想定されている関係を解除します。これにはいくつかの結果があります。

*これにより、この機能をサポートするウォレットがより複雑になります。たとえば、チェーン上のアドレスが更新された場合、CLIウォレットの対応するキーも更新する必要があります。
*変更された公開鍵の残高が0のアカウントは、自動的にプルーニングできません。
### ニュートラル

*これの目的は、アカウント所有者が所有する新しい公開鍵に更新できるようにすることですが、技術的には、これを使用してアカウントの所有権を新しい所有者に譲渡することもできます。たとえば、これを使用して、拘束力を解除せずに、既得のトークンを持つ住宅ローンのポジションまたはアカウントを販売できます。ただし、基本的には非常に具体的な店頭取引として行われる必要があるため、この摩擦は非常に高くなります。さらに、属性トークンを持つアカウントがこの機能を使用できないようにするために、追加の制約を追加できます。
*ジェネシスエクスポートに含める必要があるアカウントの公開鍵を含めます。

## 参照

+ https://www.algorand.com/resources/blog/announcing-rekeying 