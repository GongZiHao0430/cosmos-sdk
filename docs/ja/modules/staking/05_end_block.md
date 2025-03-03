# エンドブロック

各abciエンドブロック呼び出し、キューとバリデーターセットを更新する操作
変更は実行するように指定されています。

## バリデーターセットの変更

ステーキングバリデーターセットは、このプロセス中に、すべてのブロックの最後で実行される状態遷移によって更新されます。 このプロセスの一部として、更新されたバリデーターもTendermintに戻され、コンセンサスレイヤーでTendermintメッセージの検証を担当するTendermintバリデーターセットに含まれます。 操作は次のとおりです。

- 新しいバリデーターセットは、
  `ValidatorsByPower`インデックスから取得したバリデーター
- 以前のバリデーターセットが新しいバリデーターセットと比較されます。
     - 欠落しているバリデーターは結合を解除し始め、それらの`トークン`はから転送されます
    `BondedPool`から`NotBondedPool``ModuleAccount`
     - 新しいバリデーターは即座に結合され、それらの「トークン」は
    `NotBondedPool`から`BondedPool``ModuleAccount`

すべての場合において、結合されたバリデーターセットを出入りするか、残高を変更し、結合されたバリデーターセット内にとどまるバリデーターは、Tendermintに返される新しいコンセンサスパワーを報告する更新メッセージを受け取ります。

`LastTotalPower`と`LastValidatorsPower`は総電力の状態を保存します。そして、チェックのための最後のブロックの終わりからのバリデーターパワー
`ValidatorsByPower`の変更と新しい総電力
「EndBlock」中に計算されます。

## 列

ステーキング内では、特定の状態遷移は瞬間的ではなく、一定期間(通常は非結合期間)にわたって発生します。 これらの遷移が成熟すると、状態操作を完了するために特定の操作を実行する必要があります。 これは、各ブロックの最後でチェック/処理されるキューを使用することで実現されます。

### 結合解除バリデーター

バリデーターが結合されたバリデーターセットから追い出されると(投獄されるか、十分な結合トークンがないため)、すべての委任が結合解除を開始するとともに、結合解除プロセスが開始されます(この検証子に委任されたままです)。 この時点で、バリデーターは「非結合バリデーター」と呼ばれ、非結合期間が経過すると、成熟して「非結合バリデーター」になります。

各ブロックのバリデーターキューは、成熟した非結合バリデーターについてチェックされます(つまり、完了時間<=現在の時間と完了の高さ<=現在のブロックの高さ)。 この時点で、委任が残っていない成熟したバリデーターは州から削除されます。 まだ委任が残っている他のすべての成熟した非結合バリデーターの場合、`validator.Status`は`types.Unbonding`から`types.Unbonded`に切り替えられます。

### 委任の解除

次の手順で、`UnbondingDelegations`キュー内のすべての成熟した`UnbondingDelegations.Entries`の結合解除を完了します。

- 残高コインを委任者のウォレットアドレスに転送します
-`UnbondingDelegation.Entries`から成熟したエントリを削除します
- エントリが残っていない場合は、ストアから`UnbondingDelegation`オブジェクトを削除します。。

### 再委任

次の手順で、`Redelegations`キュー内のすべての成熟した`Redelegation.Entries`の結合解除を完了します。

-成熟したエントリを`Redelegation.Entries`から削除します
-エントリが残っていない場合は、ストアから`Redelegation`オブジェクトを削除します。