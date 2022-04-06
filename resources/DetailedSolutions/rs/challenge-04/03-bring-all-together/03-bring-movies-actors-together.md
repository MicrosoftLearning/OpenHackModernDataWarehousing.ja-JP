---
ms.openlocfilehash: 8144ebfae887df737b1ef38920465c064f1cc111
ms.sourcegitcommit: 3ff1164f4921cd2b423d97e7ff29cf3184026d57
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/22/2022
ms.locfileid: "140766026"
---
# <a name="31---bringing-all-movies-and-actors-together"></a>3.1 - すべての映画とアクターをまとめる

3 つのソース システムから映画とアクターを読み込んだら、最後の手順として、次のことを実行する必要があります。

- 各データ フレームに、ソース システムを追跡するための列を追加します。
- 3 つのデータ フレームをすべて連結します。

次のコードでそれを実行します。

```python
sr_all_moviesactors['SourceSystem'] = 'southridge'
va_all_moviesactors['SourceSystem'] = 'vanarsdel'
fc_all_moviesactors['SourceSystem'] = 'fourthcoffee'

moviesactors_frame = [sr_all_moviesactors, va_all_moviesactors, fc_all_moviesactors]

all_moviesactors = pd.concat(moviesactors_frame)

display(all_moviesactors)
```

## <a name="detailing-the-code-above"></a>上記のコードの詳細

```python
sr_all_moviesactors['SourceSystem'] = 'southridge'
va_all_moviesactors['SourceSystem'] = 'vanarsdel'
fc_all_moviesactors['SourceSystem'] = 'fourthcoffee'
```

> 存在する映画およびアクター データ フレームごとに、このコードでは、`SourceSystem` という新しい列を追加し、その値をソース システムの名前に設定します。

```python
moviesactors_frame = [sr_all_moviesactors, va_all_moviesactors, fc_all_moviesactors]

all_moviesactors = pd.concat(moviesactors_frame)
```

> 後で連結し、結果を `all_moviesactors` 変数に割り当てることができるように、`moviesactors_frame` データ フレーム コレクションが作成されます。

```python
display(all_moviesactors)
```

> 検証の目的で、`all_moviesactors` データ フレームが表示されます。