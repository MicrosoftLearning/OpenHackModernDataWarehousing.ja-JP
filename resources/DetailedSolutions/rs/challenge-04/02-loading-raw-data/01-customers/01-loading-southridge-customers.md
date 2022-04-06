---
ms.openlocfilehash: 7de1cb3c951ee7e2cbe1245aaccb6aa9af168e17
ms.sourcegitcommit: 3ff1164f4921cd2b423d97e7ff29cf3184026d57
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/22/2022
ms.locfileid: "140765887"
---
# <a name="21---loading-southridge-customers"></a>2.1 - Southridge の顧客を読み込む

次のコードでは、Southridge の顧客が読み込まれます。

```python
sr_sales_customers_parquet = mountPoint + "/raw/cloudsales/Customers.parquet"
sr_streaming_customers_parquet = mountPoint + "/raw/cloudstreaming/Customers.parquet"

sr_sales_customers = sqlContext.read.parquet(sr_sales_customers_parquet)
sr_streaming_customers = sqlContext.read.parquet(sr_streaming_customers_parquet)

sr_sales_customers = sr_sales_customers.toPandas()
sr_streaming_customers = sr_streaming_customers.toPandas()

sr_customers_frame = [sr_sales_customers, sr_streaming_customers]
sr_customers = pd.concat(sr_customers_frame)

sr_customers['CreatedDate'] = pd.to_datetime(sr_customers['CreatedDate'], errors='coerce')
sr_customers['UpdatedDate'] = pd.to_datetime(sr_customers['UpdatedDate'], errors='coerce')
sr_customers.PhoneNumber = sr_customers.PhoneNumber.astype(str)

display(sr_customers)
```

## <a name="detailing-the-code-above"></a>上のコードの詳細な説明

このコードに関する詳細情報を次に示します。

### <a name="loading-data"></a>データの読み込み

```python
sr_sales_customers_parquet = mountPoint + "/raw/cloudsales/Customers.parquet"
sr_streaming_customers_parquet = mountPoint + "/raw/cloudstreaming/Customers.parquet"

sr_sales_customers = sqlContext.read.parquet(sr_sales_customers_parquet)
sr_streaming_customers = sqlContext.read.parquet(sr_streaming_customers_parquet)
```

> Southridge には、Cloud Sales と Cloud Streaming という 2 つの顧客のソースがあります。後のマージのためには、その両方を読み込む必要があります。 上記のコードでは、両方のファイルのパスを宣言し、さらに **parquet** からそれらを DataFrame として読み込みます。

```python
sr_sales_customers = sr_sales_customers.toPandas()
sr_streaming_customers = sr_streaming_customers.toPandas()
```

> 両方の DataFrame を、簡単に変換するために Pandas DataFrame に変換します。

### <a name="concatenating-the-data-frames"></a>データ フレームの連結

```python
sr_customers_frame = [sr_sales_customers, sr_streaming_customers]
sr_customers = pd.concat(sr_customers_frame)
```

> 上記のコードでは、DataFrame の新しいコレクション (`sr_customers_frame`) を作成し、次に連結して `sr_customers` に割り当てることができるようにします。

### <a name="conforming-the-data-types"></a>データ型を一致させる

```python
sr_customers['CreatedDate'] = pd.to_datetime(sr_customers['CreatedDate'], errors='coerce')
sr_customers['UpdatedDate'] = pd.to_datetime(sr_customers['UpdatedDate'], errors='coerce')
sr_customers.PhoneNumber = sr_customers.PhoneNumber.astype(str)
```

> 上記のコードでは、不整合の問題がある可能性のあるフィールドに対する基準をデータセットに付与しています。
>
> - `UpdatedDate` と共に `CreatedDate` は `datetime` 列に変換されます
> - `PhoneNumber` は `string` 列に変換されます

```python
display(sr_customers)
```

> このコードの最後の手順では、データ変換の結果をテーブルとしてスクリーンに `display` します。