---
ms.openlocfilehash: ce6a43729bf1e77733545bf55454117f05c3cf0f
ms.sourcegitcommit: 3ff1164f4921cd2b423d97e7ff29cf3184026d57
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/22/2022
ms.locfileid: "140766030"
---
# <a name="23---loading-fourthcoffee-addresses"></a>2.3 - FourthCoffee のアドレスの読み込み

次のコードでは、FourthCoffee の顧客を読み込みます。

```python
fc_customers_filePath = "/dbfs" + mountPoint + "/raw/fourthcoffee/Customers.csv"

fc_customers_pd = pd.read_csv(fc_customers_filePath)

fc_addresses = fc_customers_pd[['CustomerID', 'AddressLine1', 'AddressLine2', 'City', 'State', 'ZipCode', 'CreatedDate', 'UpdatedDate']]

fc_addresses['CreatedDate'] = pd.to_datetime(fc_addresses['CreatedDate'], errors='coerce')
fc_addresses['UpdatedDate'] = pd.to_datetime(fc_addresses['UpdatedDate'], errors='coerce')
fc_addresses.AddressLine2 = fc_addresses.AddressLine2.astype(str)
fc_addresses.ZipCode = fc_addresses.ZipCode.astype(str)

fc_addresses.loc[fc_addresses.AddressLine2 == 'nan', 'AddressLine2'] = 'None'

fc_addresses.insert(0, 'AddressID', 'None')

display(fc_addresses)
```

## <a name="detailing-the-code-above"></a>上記のコードの詳細

このコードに関する詳細情報を次に示します。

```python
fc_customers_filePath = "/dbfs" + mountPoint + "/raw/fourthcoffee/Customers.csv"

fc_customers_pd = pd.read_csv(fc_customers_filePath)
```

> FourthCoffee データが CSV ファイルとしてデータ レイクに保存されたら、`pandas.read_csv` を使用してそれを読み込みます。 読み込まれたデータは既に Pandas データフレームになっているため、他の形式から変換するためのいくつかの手順が不要になります。

```python
fc_addresses = fc_customers_pd[['CustomerID', 'AddressLine1', 'AddressLine2', 'City', 'State', 'ZipCode', 'CreatedDate', 'UpdatedDate']]
```

> このデータフレームには顧客データも含まれているため、このスコープで必要なものはそのアドレスだけです。 そのため、Southridge データ フレームの場合と同様に、前のコードを使用して顧客情報に関連した列のみを選択します。

```python
fc_addresses['CreatedDate'] = pd.to_datetime(fc_addresses['CreatedDate'], errors='coerce')
fc_addresses['UpdatedDate'] = pd.to_datetime(fc_addresses['UpdatedDate'], errors='coerce')
fc_addresses.AddressLine2 = fc_addresses.AddressLine2.astype(str)
fc_addresses.ZipCode = fc_addresses.ZipCode.astype(str)
```

> 上記のコードでは、不整合の問題がある可能性のあるフィールドに対する基準をデータセットに付与しています。
>
> - `UpdatedDate` と共に `CreatedDate` は `datetime` 列に変換されます
> - `AddressLine2` には `null` 値が含まれる可能性があるため、データの連結と型の推論の際に問題が発生するおそれがあります。 これを回避するために、この列を `string` 列に変換します
> - `ZipCode` は `string` 列に変換されます

```python
fc_addresses.insert(0, 'AddressID', 'None')
```

> すべてのデータを同じ一連の列に一致させるために、このデータ フレームに `AddressID` を追加して、Southridge のスキーマに従うようにします。 *削除か、空の追加のどちらか* を決定する状況では、削除してそれを行うのではなく、空の列を追加してデータセットを一致させることになる可能性があります。 詳しくは、[こちらのリンク](../../../_TODOS/delete-or-not.md)を参照してください。

```python
display(fc_addresses)
```

> このコードの最後の手順では、データ変換の結果をテーブルとしてスクリーンに `display` します。