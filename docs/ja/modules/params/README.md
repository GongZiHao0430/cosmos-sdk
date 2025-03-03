# `パラメータ`

## 概要

パッケージparamsは、グローバルに利用可能なパラメータストアを提供します。

キーパーとサブスペースの2つの主要なタイプがあります。 サブスペースは、パラメータストアの分離された名前空間であり、キーの前に事前設定されたスペース名が付けられます。 Keeperには、既存のすべてのスペースにアクセスする権限があります。

サブスペースは、他のキーパーが変更できないプライベートパラメーターストアを必要とする個々のキーパーが使用できます。 params Keeperを使用して、プロポーザルが通過した場合にパラメータを変更するために、 `x/gov`ルーターへのルートを追加できます。

以下の内容は、マスターモジュールとユーザーモジュールにparamsモジュールを使用する方法を説明しています。

## コンテンツ

1. **[Keeper](01_keeper.md)**
2. **[Subspace](02_subspace.md)**
    - [Key](02_subspace.md#key)
    - [KeyTable](02_subspace.md#keytable)
    - [ParamSet](02_subspace.md#paramset)
