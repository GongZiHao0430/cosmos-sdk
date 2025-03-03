# パラメーター

ガバナンスモジュールには、次のパラメータが含まれています。

| Key           | Type   | Example                                                                                            |
|---------------|--------|----------------------------------------------------------------------------------------------------|
| depositparams | object | {"min_deposit":[{"denom":"uatom","amount":"10000000"}],"max_deposit_period":"172800000000000"}     |
| votingparams  | object | {"voting_period":"172800000000000"}                                                                |
| tallyparams   | object | {"quorum":"0.334000000000000000","threshold":"0.500000000000000000","veto":"0.334000000000000000"} |

## SubKeys

| Key                | Type             | Example                                 |
|--------------------|------------------|-----------------------------------------|
| min_deposit        | array (coins)    | [{"denom":"uatom","amount":"10000000"}] |
| max_deposit_period | string (time ns) | "172800000000000"                       |
| voting_period      | string (time ns) | "172800000000000"                       |
| quorum             | string (dec)     | "0.334000000000000000"                  |
| threshold          | string (dec)     | "0.500000000000000000"                  |
| veto               | string (dec)     | "0.334000000000000000"                  |

__注意__：ガバナンスモジュールには、他のモジュールとは異なり、オブジェクトであるパラメータが含まれています。 パラメータのサブセットのみを変更する必要がある場合は、パラメータオブジェクト構造全体ではなく、パラメータのみを含める必要があります。
