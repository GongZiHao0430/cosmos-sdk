### IBCチェーンのアップグレードの概要

このカタログには、カウンターパーティのクライアントと接続を中断せずにIBCチェーンをアップグレードする方法に関する情報が含まれています。

IBCによって接続されているチェーンは、他のチェーンとの接続を中断することなくアップグレードできる必要があります。 そうしないと、価値の高いIBC接続のアップグレードと中断が大幅に妨げられ、IBCエコシステムのチェーン開発と改善が妨げられます。 多くのチェーンアップグレードはIBCとは関係がない場合がありますが、適切に処理されない場合、一部のアップグレードはカウンターパーティの顧客に損害を与える可能性があります。 したがって、IBCクライアント割り込みアップグレードを実行するIBCチェーンは、カウンターパーティクライアントが新しいライトクライアントに安全にアップグレードできるように、IBCアップグレードを実行する必要があります。

1. [quick-guide]（./ quick-guide.md）は、IBC接続チェーンがクライアント割り込みアップグレードを実行する方法と、リピーターがCosmosSDKを使用してカウンターパーティクライアントを安全にアップグレードする方法について説明しています。
2. [developer-guide]（./ developer-guide.md）は、アップグレードされたIBCクライアント実装を開発する予定の開発者向けのガイドです。