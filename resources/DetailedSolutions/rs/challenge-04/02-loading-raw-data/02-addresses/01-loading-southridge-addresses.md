---
ms.openlocfilehash: b3880118cb83c85b0f5b91ce7e196296c81fbb81
ms.sourcegitcommit: 3ff1164f4921cd2b423d97e7ff29cf3184026d57
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/22/2022
ms.locfileid: "140765942"
---
# <a name="21---loading-southridge-addresses"></a>2.1 - Southridge のアドレスを読み込む

次のコードは、Southridge のアドレスを読み込みます。

```python
sr_sales_addresses_parquet = mountPoint + "/raw/cloudsales/Addresses.parquet"
sr_streaming_addresses_parquet = mountPoint + "/raw/cloudstreaming/Addresses.parquet"

sr_sales_addresses = sqlContext.read.parquet(sr_sales_addresses_parquet)
sr_streaming_addresses = sqlContext.read.parquet(sr_streaming_addresses_parquet)

sr_sales_addresses = sr_sales_addresses.toPandas()
sr_streaming_addresses = sr_streaming_addresses.toPandas()

sr_addresses_frame = [sr_sales_addresses, sr_streaming_addresses]
sr_addresses = pd.concat(sr_addresses_frame)

sr_addresses['CreatedDate'] = pd.to_datetime(sr_addresses['CreatedDate'], errors='coerce')
sr_addresses['UpdatedDate'] = pd.to_datetime(sr_addresses['UpdatedDate'], errors='coerce')

sr_addresses.AddressLine2 = sr_addresses.AddressLine2.astype(str)
sr_addresses.ZipCode = sr_addresses.ZipCode.astype(str)

display(sr_addresses)
```

## <a name="detailing-the-code-above"></a>上記のコードの詳細

このコードの詳細情報が以下に示されています。

```python
sr_sales_addresses_parquet = mountPoint + "/raw/cloudsales/Addresses.parquet"
sr_streaming_addresses_parquet = mountPoint + "/raw/cloudstreaming/Addresses.parquet"

sr_sales_addresses = sqlContext.read.parquet(sr_sales_addresses_parquet)
sr_streaming_addresses = sqlContext.read.parquet(sr_streaming_addresses_parquet)
```

> Southridge には、Cloud Sales と Cloud Streaming という 2 つのアドレスのソースがあります。後のマージのためには、その両方を読み込む必要があります。 上記のコードでは、両方のファイルのパスを宣言し、さらに **parquet** からそれらを DataFrame として読み込みます。

```python
sr_sales_addresses = sr_sales_addresses.toPandas()
sr_streaming_addresses = sr_streaming_addresses.toPandas()
```

> 両方の DataFrame を、簡単に変換するために Pandas DataFrame に変換します。

```python
sr_addresses_frame = [sr_sales_addresses, sr_streaming_addresses]
sr_addresses = pd.concat(sr_addresses_frame)
```

> 上記のコードでは、DataFrame の新しいコレクション (`sr_addresses_frame`) を作成し、次に連結して `sr_addresses` に割り当てることができるようにします。

```python
sr_addresses['CreatedDate'] = pd.to_datetime(sr_addresses['CreatedDate'], errors='coerce')
sr_addresses['UpdatedDate'] = pd.to_datetime(sr_addresses['UpdatedDate'], errors='coerce')

sr_addresses.AddressLine2 = sr_addresses.AddressLine2.astype(str)
sr_addresses.ZipCode = sr_addresses.ZipCode.astype(str)
```

> 上記のコードでは、不整合の問題がある可能性のあるフィールドに対する基準をデータセットに付与しています。
>
> - `UpdatedDate` と共に `CreatedDate` は `datetime` 列に変換されます
> - `AddressLine2` には `null` 値が含まれる可能性があるため、データの連結と型の推論の際に問題が発生するおそれがあります。 これを回避するために、この列を `string` 列に変換します
> - `ZipCode` は `string` 列に変換されます

```python
display(sr_addresses)
```

> このコードの最後の手順では、データ変換の結果をテーブルとしてスクリーンに `display` します。