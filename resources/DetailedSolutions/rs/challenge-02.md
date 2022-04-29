---
ms.openlocfilehash: 16fbe6839a0025975655b2c4fd319c298b494dee
ms.sourcegitcommit: 3ff1164f4921cd2b423d97e7ff29cf3184026d57
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/22/2022
ms.locfileid: "140765925"
---
# <a name="challenge-02---extract-data-from-azure-sql-and-cosmos-db"></a>課題 02 - Azure SQL と Cosmos DB からデータを抽出する

この課題では、次からデータをコピーすることが求められます

- Azure SQL データベース
    - CloudSales
    - CloudStreaming
- CosmosDB ドキュメント コレクション
    - ムービー

そのためには、Azure で抽出メカニズムをプロビジョニングする必要があります。 この詳細なソリューションでは、抽出メカニズムとして Azure Data Factory を使用して、このシナリオについて説明します。

Data Factory のプロビジョニングが完了したら、次のように、ソースからシンクにデータをコピーできるように、3 つのファクトリ リソースを構成する必要があります。

- リンク サービス: 接続先の場所を宣言します
- データセット: データ構造について説明します
- パイプライン: データ フローを構成します

## <a name="create-an-azure-data-factory"></a>Azure データ ファクトリを作成する

Azure Data Factory は、データの保管、移動、および処理のサービスを自動化データ パイプラインにまとめるのに役立つクラウド データ統合サービスです。

次の 2 つの方法で Azure Data Factory を作成できます。

- Azure portal
- Azure PowerShell

このチュートリアルでは、**PowerShell** を使用して Data Factory を作成します。

詳細な手順については、[こちら](https://docs.microsoft.com/ja-jp/azure/data-factory/quickstart-create-data-factory-powershell)を確認してください

> Windows 以外の OS を使用している場合は、Azure Cloud Shell を利用して次のコマンドを実行できます。

```powershell
# Create variables to have consistency throughout the tutorial
$resourceGroupName = "dryrunres"
$dataFactoryName = "<data factory name>"

# If you already have the Resource Group you will create the Data Factory in
$resourceGroup = Get-AzRmResourceGroup -Name $resourceGroupName

# If you will create the Resource Group as part of this tutorial
$resourceGroup = New-AzResourceGroup $resourceGroupName -location '<location>'

# Now to create the Data Factory
$DataFactory = Set-AzDataFactoryV2 `
    -ResourceGroupName $resourceGroup.ResourceGroupName `
    -Location $resourceGroup.Location -Name $dataFactoryName
```

## <a name="copying-sql-data-into-azure-data-lake-storage-gen2-using-powershell"></a>PowerShell を使用して SQL データを Azure Data Lake Storage Gen2 にコピーする

Data Lake Storage Gen2 が有効になっているストレージ アカウントを作成したら、データベースからデータ レイクにデータをコピーするための構成を開始できます。

PowerShell を使用してこれを実現するには、次の手順に従います。

このチュートリアルで作成するさまざまなリンク サービスの種類に対して、リンク サービス構成参照も必要になります。
構成するソースまたはシンクがサポートされているかどうかを確認し、それらのサンプルを確認する場合に備えて、[データセットの種類](https://docs.microsoft.com/ja-jp/azure/data-factory/concepts-datasets-linked-services#dataset-type) のドキュメントを手元に用意しておいてください。

- [01 - リンク サービスの作成](challenge-02/creating-linked-services.md)
    - データセットを作成する前に、データ ソースとシンクに接続するためのリンク サービスが必要です。
- [02 - データセットの作成](challenge-02/creating-datasets.md)
    - データセットを使用して、コピー元またはコピー先のデータ表現を定義します。
- [03 - パイプラインの作成](challenge-02/creating-pipelines.md)
    - これらのパイプラインは、データセットを調整して、ソースからシンクにデータを移動します。

## <a name="learning-experiences-and-road-blocks"></a>ラーニング エクスペリエンスと道路ブロック

- 出席者が MacOS および Azure CLI をローカルで使用している場合、bash を使用して ADF を作成することはできません。 次のいずれかの操作を行う必要があります。
    - Azure Cloud Shell を使用する
    - UI を使用してリソースを作成する
- Azure Cloud Shell を使用してファイルをアップロードする

## <a name="potential-feedbacks"></a>潜在的なフィードバック

### <a name="data-factory-creation"></a>Data Factory の作成

- 見た限りでは、Azure CLI を使用した ADF の管理はサポートされていません。
- [このリンク](https://docs.microsoft.com/ja-jp/azure/data-factory/#5-minute-quickstarts)の左側のメニューに、間違ったタイトルが表示されています
    - _"データ ファクトリの作成"_ には、次の 2 つのみが必要です。
        - ユーザー インターフェイス (UI) または
        - Azure PowerShell
    - 他のユーザーは、 _"パイプラインの作成 - (サブジェクト)"_ のようになります
- [このチュートリアル](https://docs.microsoft.com/ja-jp/azure/data-factory/quickstart-create-data-factory-powershell) では、Azure Data Factory を作成する方法を説明するだけでなく、完全な **データ移動** のサンプルを作成します。 **Data Factory の作成** に関する明確なサンプルを用意することをお勧めします。
    - これは、このチュートリアルでは、最初にコピーされるデータを保持するストレージ アカウントを作成し、次に Data Factory の作成方法について説明するためです。 そのため、混乱する可能性があります。チュートリアルに従っているという理由だけでストレージ アカウントを作成する可能性がありますが、実際には必要ない場合があります。

### <a name="linked-services"></a>リンクされたサービス

- [このチュートリアル](https://docs.microsoft.com/ja-jp/azure/data-factory/tutorial-bulk-copy)では、さまざまなリンク サービスの種類 (SQL DW の代わりに ADLS など) の他のサンプルを取得するためのユーザー向けのリファレンスを共有していません。
- 別のタブで実行すると、Azure Cloud Shell セッションの有効期限が短くなります。ユーザーがチュートリアル全体で名前の一貫性を維持するためにいくつかの変数を設定した場合に影響します。 これらの変数では、セッションが削除されると、値が緩やかになります。

### <a name="datasets"></a>データセット

- AzureRm モジュールを使用して PowerShell コマンドから CosmosDB 接続文字列を取得する方法が見つかりませんでした。
- Az PowerShell モジュールで、リンク サービスを作成するために、`-File` パラメーターを使用してファイルを参照します。 データセットを作成する場合、同じ目的を持つパラメーターには `-DefinitionFile` という名前が付けられます。 ここでは標準があります。
- Parquet ファイルの機能方法の学習
- データセットのドキュメントは、ソリューションをより堅牢にするために使用できる手法、つまりパラメーターの使用に関しては不十分です。

### <a name="pipelines"></a>パイプライン

- CosmosDB のパイプラインをソースとして作成するときに、データを *そのまま* コピーする場合、パラメーター `properties:activities:<Copy Activity>:typeProperties:source:nestingSeparator`
 は空白である **必要があります**。 それ以外の場合は、値として "." が自動的に割り当てられ、パイプラインは機能しません。