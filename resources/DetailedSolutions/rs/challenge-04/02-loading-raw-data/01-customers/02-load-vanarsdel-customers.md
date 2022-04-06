---
ms.openlocfilehash: a92b11c8be6b20d5344f1c001b4176aeba647a0c
ms.sourcegitcommit: 3ff1164f4921cd2b423d97e7ff29cf3184026d57
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/22/2022
ms.locfileid: "140765974"
---
# <a name="22---loading-vanarsdel-customers"></a>2.2 - VanArsdel の顧客の読み込み

次のコードでは、VanArsdel の顧客を読み込みます。

```python
va_customers_filePath = mountPoint + "/raw/vanarsdel/Customers.json"

va_customers_raw = spark.read.option("multiLine", "true").options(header='true', inferschema='true').json(va_customers_filePath)
va_customers_pd = va_customers_raw.toPandas()

va_customers = va_customers_pd[['CustomerID', 'LastName', 'FirstName', 'PhoneNumber', 'CreatedDate', 'UpdatedDate']]

va_customers['CreatedDate'] = pd.to_datetime(va_customers_pd['CreatedDate'], errors='coerce')
va_customers['UpdatedDate'] = pd.to_datetime(va_customers_pd['UpdatedDate'], errors='coerce')

va_customers.PhoneNumber = va_customers_pd.PhoneNumber.astype(str)

display(va_customers)
```

## <a name="detailing-the-code-above"></a>上記のコードの詳細

このコードに関する詳細情報を次に示します。

### <a name="loading-the-data"></a>データを読み込む

```python
va_customers_filePath = mountPoint + "/raw/vanarsdel/Customers.json"

va_customers_raw = spark.read.option("multiLine", "true").options(header='true', inferschema='true').json(va_customers_filePath)
```

> VanArsdel データが JSON ファイルとしてデータ レイクに保存されたら、上で確認できるいくつかのオプションを指定し、`spark.read.json` を使用してそれを読み込みます。 その後、読み込まれたデータは、すぐに使用できる PySpark データフレームになります。

```python
va_customers_pd = va_customers_raw.toPandas()
```

> 変換しやすくするために、PySpark データフレームを Pandas データフレームに変換します。

### <a name="selecting-the-relevant-data"></a>関連データの選択

```python
va_customers = va_customers_pd[['CustomerID', 'LastName', 'FirstName', 'PhoneNumber', 'CreatedDate', 'UpdatedDate']]
```

> このデータフレームにはアドレスも含まれているため、このスコープで必要なものは顧客情報だけです。 そのため、Southridge データ フレームの場合と同様に、前のコードを使用して顧客情報に関連した列のみを選択します。

### <a name="conforming-the-data-types"></a>データ型を一致させる

```python
va_customers['CreatedDate'] = pd.to_datetime(va_customers['CreatedDate'], errors='coerce')
va_customers['UpdatedDate'] = pd.to_datetime(va_customers['UpdatedDate'], errors='coerce')

va_customers.PhoneNumber = va_customers.PhoneNumber.astype(str)
```

> 上記のコードでは、不整合の問題がある可能性のあるフィールドに対する基準をデータセットに付与しています。
>
> - `UpdatedDate` と共に `CreatedDate` は `datetime` 列に変換されます
> - `PhoneNumber` は `string` 列に変換されます

```python
display(va_customers)
```

> このコードの最後の手順では、データ変換の結果をテーブルとしてスクリーンに `display` します。