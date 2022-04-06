---
ms.openlocfilehash: ac2fca1d5a874c57bb13ce9be13ce684a1c654c8
ms.sourcegitcommit: 3ff1164f4921cd2b423d97e7ff29cf3184026d57
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/22/2022
ms.locfileid: "140766028"
---
## <a name="challenge-5-a-star-is-born"></a>課題 5: スターの形成

この課題では、当座のビジネス インテリジェンスの目的を達成するために、前の課題からの "ワンストップ ショップ" を活用することに焦点を当てます。 チームは、PBIX ファイルが提供されますが、これらのレポートを提供するスキーマを作成し、データを設定する必要があります。

Polybase を使用して、SQL DW ソリューションのドキュメントを作成します。 他にもさまざまな方式があります。
チームはデータのスケールを考慮して、SQL DW の代わりに Azure SQL でスター スキーマを作成することもできます。
その上に Analysis Services を配置することもできます。
Polybase ではなく Spark コネクタを使用して、スター スキーマにデータをプッシュすることもできます。

### <a name="sql-dw-polybase-solution"></a>SQL DW Polybase ソリューション

[star-schema](./star-schema/) ディレクトリを参照してください。
これとコーチ ガイドの組み合わせにより、このソリューションの重要なポイントについて順を追って説明します。
この作業は保留中で、ソリューションは未完成ですが、この課題の残りの部分では、同じパターンに従って残りのディメンションおよびファクト テーブルを処理することになるでしょう。

スクリプトは、番号順に並んだディレクトリに分けられています。
各ステージのディレクトリ内でも、スクリプトは番号順に並んでいます。