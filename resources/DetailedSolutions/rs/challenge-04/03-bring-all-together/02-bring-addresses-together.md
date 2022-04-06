---
ms.openlocfilehash: 9877ceaf24da6e80dca3f0b8723af2865a75554e
ms.sourcegitcommit: 3ff1164f4921cd2b423d97e7ff29cf3184026d57
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/22/2022
ms.locfileid: "140765966"
---
# <a name="31---bringing-all-addresses-together"></a>3.1 - すべてのアドレスをまとめる

3 つのソース システムからアドレスを読み込んだら、最後の手順として、次のことを実行する必要があります。

- 各データ フレームに、ソース システムを追跡するための列を追加します。
- 3 つのデータ フレームをすべて連結します。

次のコードでそれを実行します。

```python
sr_addresses['SourceSystem'] = 'southridge'
va_addresses['SourceSystem'] = 'vanarsdel'
fc_addresses['SourceSystem'] = 'fourthcoffee'
addresses_frame = [sr_addresses, va_addresses, fc_addresses]

all_addresses = pd.concat(addresses_frame)

display(all_addresses)
```

## <a name="detailing-the-code-above"></a>上記のコードの詳細

```python
sr_addresses['SourceSystem'] = 'southridge'
va_addresses['SourceSystem'] = 'vanarsdel'
fc_addresses['SourceSystem'] = 'fourthcoffee'
```

> 存在するアドレス データ フレームごとに、このコードでは `SourceSystem` という名前の新しい列を追加し、その値をソース システムの名前に設定しています。

```python
addresses_frame = [sr_addresses, va_addresses, fc_addresses]

all_addresses = pd.concat(addresses_frame)
```

> 後で連結し、結果を `all_addresses` 変数に割り当てることができるように、`addresses_frame` データ フレーム コレクションが作成されます。

```python
display(all_addresses)
```

> 検証の目的で、`all_addresses` データ フレームが表示されます。