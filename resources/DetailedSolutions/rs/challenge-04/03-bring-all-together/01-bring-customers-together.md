---
ms.openlocfilehash: 7362a7338ee78639b13c481f725e2e86b645fe05
ms.sourcegitcommit: 3ff1164f4921cd2b423d97e7ff29cf3184026d57
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/22/2022
ms.locfileid: "140765908"
---
# <a name="31---bringing-all-customers-together"></a>3.1 - すべての顧客を 1 つにまとめる

3 つのソース システムから顧客を読み込んだら、最後の手順として、次のことを実行する必要があります。

- 各データ フレームにソース システムを追跡するための列を追加します。
- 3 つのデータ フレームをすべて連結します。

次のコードがそれを実行します。

```python
sr_customers['SourceSystem'] = 'southridge'
va_customers['SourceSystem'] = 'vanarsdel'
fc_customers['SourceSystem'] = 'fourthcoffee'
customers_frame = [sr_customers, va_customers, fc_customers]

all_customers = pd.concat(customers_frame)

display(all_customers)
```

## <a name="detailing-the-code-above"></a>前のコードの詳細な説明

```python
sr_customers['SourceSystem'] = 'southridge'
va_customers['SourceSystem'] = 'vanarsdel'
fc_customers['SourceSystem'] = 'fourthcoffee'
```

> 存在する顧客データ フレームごとに、このコードでは `SourceSystem` という名前の新しい列を追加し、その値をソース システムの名前に設定しています。

```python
customers_frame = [sr_customers, va_customers, fc_customers]

all_customers = pd.concat(customers_frame)
```

> 後で連結が可能になるように `customers_frame` データ フレーム コレクションが作成され、その結果が `all_customers` 変数に割り当てられます。

```python
display(all_customers)
```

> 検証のために、`all_customers` データ フレームが表示されます。