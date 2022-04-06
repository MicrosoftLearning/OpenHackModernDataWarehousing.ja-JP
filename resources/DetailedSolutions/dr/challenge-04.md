---
ms.openlocfilehash: 07c90bf4f4ddc091584c13e865ba7d9b619bd69d
ms.sourcegitcommit: 3ff1164f4921cd2b423d97e7ff29cf3184026d57
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/22/2022
ms.locfileid: "140765944"
---
## <a name="challenge-4-more-than-meets-the-eye"></a>課題 4: 見た目ではわからない価値

この課題では、データ レイク内のデータの "ワンストップ ショップ" を作成することに焦点を絞ります。 多様なビジネス ニーズを満たそうとしている人びとはそれぞれ、そのダウンストリーム処理に対する要件は異なる可能性がありますが、**すべての人** にとってメリットがある 1 つのものとして、異種データ システムの荒削りな長所を滑らかにする中間データセットがあります。

特に、ダウンストリームの目標が BI または ML のどちらであっても、すべてのチームは次のことを行います。

- エンタープライズ データの完全なコーパスを表す統合されたデータセットを操作する
- 将来いくつのストアが必要になるかに関係なく、同じデータセットを使い続ける。つまり、新しいデータを完全なデータセットに結合するための作業の重複を回避する
- 共通の、合致したスキーマを用意する。たとえば、システムごとに名前が異なっていたり、同じ概念に対して異なるデータ型が使用されたりしている場合、ダウンストリーム チームはこの精神的な重みを引きずる必要はなくなる

### <a name="target-datasets"></a>ターゲット データセット

最後に、次の 5 つのデータセットのための機会を確認できます。

1. [カタログ](./notebooks/Conformed%20Catalog.ipynb): 3 つのすべてのソース システムのすべての映画と俳優
1. 顧客: 3 つのすべてのソース システムのすべての人びとと住所
1. 販売: Southridge CloudSales のすべての注文と注文の詳細
1. ストリーミング: Southridge CloudStreaming のすべてのトランザクション情報
1. レンタル: Fourth Coffee と Van Arsdel, Ltd. の両方のすべてのトランザクション情報

### <a name="to-join-cleanse-drop-duplicates-etc--or-not"></a>結合するために、クレンジング、重複の削除などを行うかどうか

この段階では、ダウンストリーム処理で例外を発生させる可能性のある致命的な異常 (一貫性のないデータ型や形式など) に焦点を絞ります。
このデータを最終的なレポート スキーマに直接読み込んだとすると、次のような追加のデータ クレンジングが適用される可能性があります (ただし、これには限定されません)。

- 入力ミスを探して解消する (たとえば、PG ではなく PGg)
- タイトルの大文字と小文字の区別、名前、レーティングなどを正規化する
- 対応する映画の矛盾を探して解決する (たとえば、Southridge では Mysterious Cube を G レーティングのファミリー映画と見なしているのに対して、VanArsdel, Ltd. では PG-13 レーティングのコメディーと見なしている)
- 俳優の名前のバリエーションを探し、レポート スキーマ全体にわたって一貫して使用する 1 つを選択する (たとえば、Vivica A. Fox か Vivica Fox か)
- 重複を削除する

ここでこれらの操作を実行すると、以前は認識されなかったデータの価値を発見する機会が失われる可能性があります。
不自然な例として、俳優や女優の中にはミドル ネームの頭文字を使う人も、使わない人もいるという可能性を考えてみてください。
ここで、データ サイエンティストが、映画はキャストがミドル ネームの頭文字を使っている方が売れるという傾向を発見したとします。
さらに、それがドラマのジャンルにのみ当てはまり、ファミリー映画には当てはまらないと想像してみてください。
人びとの "通常の" 課金に正規化するビジネス ルールが既に適用されていたとすると、データ サイエンティストがこれを発見することはできなくなるでしょう。

### <a name="establishing-a-branch-policy"></a>ブランチ ポリシーの確立

> TODO: これに関するリファレンスはかなり明確ですが、ここで直接のガイダンスをいくつか追加します。