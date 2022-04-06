---
ms.openlocfilehash: 2248b634086756c34289c2293c1273c48339a9b8
ms.sourcegitcommit: 3ff1164f4921cd2b423d97e7ff29cf3184026d57
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/22/2022
ms.locfileid: "140765929"
---
# <a name="22---loading-vanarsdel-addresses"></a>2.2 - VanArsdel の住所の読み込み

次のコードでは、VanArsdel の住所を読み込みます。

```python
va_customers_filePath = mountPoint + "/raw/vanarsdel/Customers.json"

va_customers_raw = spark.read.option("multiLine", "true").options(header='true', inferschema='true').json(va_customers_filePath)
va_customers_pd = va_customers_raw.toPandas()

va_addresses = va_customers_pd[['CustomerID', 'AddressLine1', 'AddressLine2', 'City', 'State', 'ZipCode', 'CreatedDate', 'UpdatedDate']]

va_addresses['CreatedDate'] = pd.to_datetime(va_addresses['CreatedDate'], errors='coerce')
va_addresses['UpdatedDate'] = pd.to_datetime(va_addresses['UpdatedDate'], errors='coerce')

va_addresses.AddressLine2 = va_addresses.AddressLine2.astype(str)
va_addresses.ZipCode = va_addresses.ZipCode.astype(str)

va_addresses.insert(0, 'AddressID', 'None')

display(va_addresses)
```

## <a name="detailing-the-code-above"></a>前のコードの詳細な説明

このコードに関する詳細情報を次に示します。

```python
va_addresses_filePath = mountPoint + "/raw/vanarsdel/Addresses.json"

va_addresses_raw = spark.read.option("multiLine", "true").options(header='true', inferschema='true').json(va_addresses_filePath)
```

> VanArsdel データが JSON ファイルとしてデータ レイクに保存されたら、上で確認できるいくつかのオプションを指定し、`spark.read.json` を使用してそれを読み込みます。 それにより、読み込まれたデータは、すぐに使用できる PySpark データフレームになります。

```python
va_addresses_pd = va_addresses_raw.toPandas()
```

> 変換しやすくするために、PySpark データフレームを Pandas データフレームに変換します。

```python
va_addresses = va_addresses_pd[['CustomerID', 'LastName', 'FirstName', 'PhoneNumber', 'CreatedDate', 'UpdatedDate']]
```

> このデータフレームには顧客情報も含まれているため、このスコープで必要なものはその住所だけです。 そのため、Southridge データ フレームの場合と同様に、前のコードを使用して顧客情報に関連した列のみを選択します。

```python
va_addresses['CreatedDate'] = pd.to_datetime(va_addresses['CreatedDate'], errors='coerce')
va_addresses['UpdatedDate'] = pd.to_datetime(va_addresses['UpdatedDate'], errors='coerce')

va_addresses.AddressLine2 = va_addresses.AddressLine2.astype(str)
va_addresses.ZipCode = va_addresses.ZipCode.astype(str)
```

> 前のコードでは、データセットに、不整合の問題が発生する可能性があるフィールドのための標準を提供しています。
>
> - `CreatedDate` は、`UpdatedDate` と共に `datetime` 列に変換されます。
> - `AddressLine2` には `null` 値が含まれる可能性があるため、データの連結や型の推論中に問題が発生することがあります。 これを回避するために、この列を `string` 列に変換します。
> - `ZipCode` は `string` 列に変換されます。

```python
va_addresses.insert(0, 'AddressID', 'None')
```

> すべてのデータを同じ一連の列に一致させるために、このデータ フレームに `AddressID` を追加して、Southridge のスキーマに従うようにします。 *削除か、空の追加のどちらか* を決定する状況では、削除してそれを行うのではなく、空の列を追加してデータセットを一致させることになる可能性があります。 詳しくは、[こちらのリンク](../../../_TODOS/delete-or-not.md)を参照してください。

```python
display(va_addresses)
```

> このコードの最後の手順では、データ変換の結果を表として画面に `display` します。