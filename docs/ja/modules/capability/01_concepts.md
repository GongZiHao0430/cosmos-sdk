# 概念

## 能力

機能は複数所有者です。スコープのキーパーは、 `NewCapability`を介して機能を作成できます
これは、新しい一意の偽造不可能なオブジェクト機能リファレンスを作成します。新着
作成された機能は自動的に保持されます。呼び出し元のモジュールを呼び出す必要はありません。
「主張する能力。」 `NewCapability`を呼び出すと、
呼び出し元のモジュールとタプルとしての名前は、関数の最初の所有者と見なされます。

他のモジュールは関数を宣言でき、これらのモジュールはそれらを所有者として追加します。 「主張する能力」
モジュールが別のモジュールから受け取った機能キーを宣言できるようにする
`GetCapability`への将来の呼び出しが成功するようにモジュール化します。 `ClaimCapability`は
受信モジュールが名前でアクセスしたい場合は、
将来。同様に、機能は複数所有者であるため、複数のモジュールに1つある場合
単一の能力リファレンス、それらはすべてそれを持っています。モジュールが機能を受け取った場合
別のモジュールからですが、 `ClaimCapability`を呼び出さないため、実行時に使用できます
トランザクションですが、後でアクセスすることはできません。

どのモジュールでもAuthenticateCapabilityを呼び出して、機能を確認できます
実際には特定の名前に対応しています（名前は信頼できないユーザー入力である可能性があります）
モジュールを呼び出す前に、モジュールに関連付けられています。

`GetCapability`を使用すると、モジュールは以前に持っていた機能を取得できます
名前で主張する。モジュールは、実行する関数を取得することを許可しません
所有していません。

## ストレージ

- メモリバンク