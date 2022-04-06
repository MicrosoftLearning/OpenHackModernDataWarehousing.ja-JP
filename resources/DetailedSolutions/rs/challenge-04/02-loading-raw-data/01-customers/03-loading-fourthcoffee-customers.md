---
ms.openlocfilehash: 7246f53f9c1618cf0fa50e03a326dfb76a5473a1
ms.sourcegitcommit: 3ff1164f4921cd2b423d97e7ff29cf3184026d57
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/22/2022
ms.locfileid: "140765910"
---
# <a name="23---loading-fourthcoffee-customers"></a>2.3 - FourthCoffee の顧客の読み込み

次のコードでは、FourthCoffee の顧客を読み込みます。

```python
fc_customers_filePath = "/dbfs" + mountPoint + "/raw/fourthcoffee/Customers.csv"

fc_customers_pd = pd.read_csv(fc_customers_filePath)

fc_customers = fc_customers_pd[['CustomerID', 'LastName', 'FirstName', 'PhoneNumber', 'CreatedDate', 'UpdatedDate']]

fc_customers['CreatedDate'] = pd.to_datetime(fc_customers_pd['CreatedDate'], errors='coerce')
fc_customers['UpdatedDate'] = pd.to_datetime(fc_customers_pd['UpdatedDate'], errors='coerce')
fc_customers.PhoneNumber = fc_customers_pd.PhoneNumber.astype(str)

display(fc_customers)
```

## <a name="detailing-the-code-above"></a>前のコードの詳細な説明

このコードに関する詳細情報を次に示します。

### <a name="loading-the-data"></a>データを読み込む

```python
fc_customers_filePath = "/dbfs" + mountPoint + "/raw/fourthcoffee/Customers.csv"

fc_customers_pd = pd.read_csv(fc_customers_filePath)
```

> FourthCoffee データが CSV ファイルとしてデータ レイクに保存されたら、`pandas.read_csv` を使用してそれを読み込みます。 読み込まれたデータは既に Pandas データフレームになっているため、他の形式から変換するためのいくつかの手順が節約されます。

### <a name="selecting-the-relevant-data"></a>関連データの選択

```python
fc_customers = fc_customers_pd[['CustomerID', 'LastName', 'FirstName', 'PhoneNumber', 'CreatedDate', 'UpdatedDate']]
```

> このデータフレームには住所も含まれているため、このスコープで必要なものは顧客情報だけです。 そのため、Southridge データ フレームの場合と同様に、前のコードを使用して顧客情報に関連した列のみを選択します。

### <a name="conforming-the-data-types"></a>データ型を一致させる

```python
fc_customers['CreatedDate'] = pd.to_datetime(fc_customers['CreatedDate'], errors='coerce')
fc_customers['UpdatedDate'] = pd.to_datetime(fc_customers['UpdatedDate'], errors='coerce')
fc_customers.PhoneNumber = fc_customers.PhoneNumber.astype(str)
```

> 前のコードでは、データセットに、不整合の問題が発生する可能性があるフィールドのための標準を提供しています。
>
> - `CreatedDate` は、`UpdatedDate` と共に `datetime` 列に変換されます。
> - `PhoneNumber` は `string` 列に変換されます。

```python
display(fc_customers)
```

> このコードの最後の手順では、データ変換の結果を表として画面に `display` します。