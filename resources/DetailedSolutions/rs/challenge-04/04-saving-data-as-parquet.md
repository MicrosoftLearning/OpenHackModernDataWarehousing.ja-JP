---
ms.openlocfilehash: 6dfbf1f20e20d04bb9ebbe80b8a11543210bf1b1
ms.sourcegitcommit: 3ff1164f4921cd2b423d97e7ff29cf3184026d57
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/22/2022
ms.locfileid: "140765972"
---
# <a name="4---saving-the-data-as-parquet-to-the-data-lake"></a>4 - データを Parquet としてデータ レイクに保存する

Pandas データフレームを Parquet ファイルとしてデータ レイクに保存するこの方法は、3 つのビジネス オブジェクト (`Addresses`、`Customers`、`Movies and Actors`) すべてで同様に機能します。

その前提で、次のコードには、これらの各ビジネス オブジェクトのプレースホルダーが含まれています。

```python
parquet_path = mountPoint + '/parquet/<Business Object name>.parquet'

saveAsParquet(all_<business object name>, parquet_path)
```

## <a name="detailing-the-code-above"></a>上記のコードの詳細

```python
parquet_path = mountPoint + '/parquet/<Business Object name>.parquet'
```

> このコード行では、2 つの文字列を組み合わせて、データ レイクに保存する Parquet ファイルの完全なパスを作成します。

```python
saveAsParquet(all_<business object name>, parquet_path)
```

> 上記のコード行では、最初のセルで定義された `saveAsParquet` 関数を使用して、変換結果を Parquet ファイルとしてデータ レイクに保存します。