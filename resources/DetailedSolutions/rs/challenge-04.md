---
ms.openlocfilehash: ccf2ccbddf8a648a61ba072620667af464764aba
ms.sourcegitcommit: 3ff1164f4921cd2b423d97e7ff29cf3184026d57
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/22/2022
ms.locfileid: "140765885"
---
# <a name="challenge-04---transforming-all-the-data"></a>課題 04 - すべてのデータの変換

この課題の目的は、すべての共通データをマージし、一部のデータ型を調整することです。そのすべてで 1 つのソース管理ブランチのベスト プラクティスに従います。

この課題に合格するために、この詳細なソリューションでは Databricks オプションについて説明します。

## <a name="protecting-the-master-branch-with-pull-request-policies"></a>Pull Request ポリシーを使用したマスター ブランチの保護

目的は、コミットとプッシュがマスターに直接実行されるのを防ぐことです。
これにより、未承認のコードが環境に将来自動的にデプロイされるのを防ぎます。 ここでの考え方は、安定したコードのために `master` ブランチをコードベースに変えるというものです。

これを行うには、Microsoft ドキュメントとして使用できる非常に優れたガイドが[こちら](https://docs.microsoft.com/en-us/azure/devops/repos/git/branch-policies?view=azure-devops)にがあります。
具体的には、[この手順は](https://docs.microsoft.com/en-us/azure/devops/repos/git/branch-policies?view=azure-devops#require-a-minimum-number-of-reviewers)、だれかがマスターにコードを送信したい場合に少なくとも 1 人のレビュー担当者を持つのに役立ちます。

## <a name="creating-the-databricks-workspace"></a>Databricks ワークスペースの作成

[このリファレンス](https://azure.microsoft.com/en-us/resources/templates/101-databricks-workspace/)を使用して、次のプロパティを持つ新しい Databricks ワークスペースを作成します。

- `workspaceName`: ワークスペースの名前
- `pricingTier`: このソリューションのスコープの場合は Standard

### <a name="azure-cli"></a>Azure CLI

```cmd
RESOURCE_GROUP_NAME="<Name of the resource group you already have>"
WORKSPACE_NAME="<The name for your Databricks workspace>"
PRICING_TIER="standard"
TEMPLATE_URI="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-databricks-workspace/azuredeploy.json"

az group deployment create \
    --resource-group $RESOURCE_GROUP_NAME \
    --template-uri $TEMPLATE_URI
    --parameters \
    workspaceName=$WORKSPACE_NAME \
    pricingTier=$PRICING_TIER
```

### <a name="powershell"></a>PowerShell

> **警告**: このガイダンスのスコープでは、Azure CLI を使用することを強くお勧めします。 PowerShell Az モジュールでは、このシナリオの後半で必要になる、プレーン テキストのサービス プリンシパルのパスワードを取得できません

```powershell
$resourceGroupName = "<Name of the resource group you already have>"
$workspaceName = "<The name for your Databricks workspace>"
$pricingTier = "standard"
$templateUri = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-databricks-workspace/azuredeploy.json"

New-AzResourceGroupDeployment `
    -TemplateUri $templateUri
    -ResourceGroupName $resourceGroupName `
    -workspaceName $workspaceName `
    -pricingTier $pricingTier
```

## <a name="create-a-service-principal"></a>サービス プリンシパルを作成する

サービス プリンシパルを使用して、Databricks で Azure Storage アカウントの Data Lake Storage にアクセスできるようにします。 そのためには、次の手順に従います。

参照:

- [チュートリアル:Spark を使用して Azure Databricks で Data Lake Storage Gen2 のデータにアクセスする](https://docs.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-use-databricks-spark)
- [RBAC と Azure PowerShell を使用して Azure リソースへのアクセス権をユーザーに付与する](https://docs.microsoft.com/en-us/azure/role-based-access-control/role-assignments-powershell)
- [Azure portal で RBAC を使用して Azure BLOB とキューのデータへのアクセスを付与する](https://docs.microsoft.com/en-us/azure/storage/common/storage-auth-aad-rbac-portal#assign-rbac-roles-using-the-azure-portal)
- [**PowerShell** を使用して RBAC で Azure BLOB とキューのデータへのアクセスを付与する](https://docs.microsoft.com/en-us/azure/storage/common/storage-auth-aad-rbac-powershell#list-available-rbac-roles)

これは 1 つのコマンドで実行できます。

- 新しいサービス プリンシパルの作成
- ストレージ アカウントのこのサービス プリンシパルへの `Storage Blob Data Contributor` の割り当て

### <a name="powershell"></a>PowerShell

```powershell
$spDisplayName = "<The display name for the Service Principal>"
$scope = "/subscriptions/<subscription id>/resourceGroups/<resource group name>/providers/Microsoft.Storage/storageAccounts/<storage account name>"

New-AzADServicePrincipal `
    -DisplayName riserradsp `
    -Role "Storage Blob Data Contributor" `
    -Scope $scope
```

必ず次のプレースホルダーに入力してください。

- `<subscription id>`
- `<resource group name>`
- `<storage account name>`

次のような出力が表示されます。

```powershell
Secret                : System.Security.SecureString
ServicePrincipalNames : {<ApplicationId GUID>, http://<Service Principal Display Name>}
ApplicationId         : <ApplicationId GUID>
ObjectType            : ServicePrincipal
DisplayName           : <Service Principal Display Name>
Id                    : <Service Principal Object Id>
Type                  :
```

### <a name="azure-cli"></a>Azure CLI

```bash
$SP_DISPLAYNAME="<The display name for the Service Principal>"
$SCOPE="/subscriptions/<subscription id>/resourceGroups/<resource group name>/providers/Microsoft.Storage/storageAccounts/<storage account name>"

az ad sp create-for-rbac \
    -n $SP_DISPLAYNAME \
    --role "Storage Blob Data Contributor" \
    --scope $SCOPE
```

出力は次のようになります。

```bash
{
  "appId": "<ApplicationID GUID>",
  "displayName": "<Service Principal Display Name>",
  "name": "http://<Service Principal Display Name>",
  "password": "<Service Principal Password/Secret GUID>",
  "tenant": "<Tenant ID>"
}
```

## <a name="install-the-databricks-cli"></a>Databricks CLI をインストールする

[このリファレンス](https://github.com/databricks/databricks-cli#installation)を使用して、`pip` を使用して Databricks CLI をインストールします。 ここでの要件は次のとおりです。

- Python 2.7.9 以上または 3.6

```bash
pip install --upgrade databricks-cli
```

次のようなメッセージが表示された場合:

```bash
Cannot uninstall 'six'. It is a distutils installed project and thus we cannot accurately determine which files belong to it which would lead to only a partial uninstall.
```

`--ignore-installed` を使用してみてください。 次のリファレンスに注意してください。

**TODO**: わかりやすい説明となるよう以下のリンクを改善します。

- [pip 10 および apt: distutils パッケージで "X をアンインストールできない" エラーを回避する方法](https://stackoverflow.com/questions/49932759/pip-10-and-apt-how-to-avoid-cannot-uninstall-x-errors-for-distutils-packages)
- [pip は <package> をアンインストールできません: “これは distutils がインストールされているプロジェクトです”](https://stackoverflow.com/questions/53807511/pip-cannot-uninstall-package-it-is-a-distutils-installed-project/53807588#53807588)
- [pip 10 へのアップグレード: これは distutils がインストールされているプロジェクトであるため、プロジェクトに属しているファイルを正確に判断できず、部分的なアンインストールしか行われない可能性があります。](https://github.com/pypa/pip/issues/5247#issuecomment-443398741)

databricks-cli がインストールされた後、CLI を構成するには、Databricks ワークスペースからアクセス トークンを生成する必要があります。 これを行うには、[このリンク](https://docs.databricks.com/api/latest/authentication.html)に従います。

トークンを取得した後、次のコマンドを使用して、Databricks ワークスペースに接続するように CLI を構成します。

```bash
databricks configure --token
```

次の 2 つの情報の入力を求めるメッセージが表示されます。

- `Databricks Host`: たとえば、`East US` で Databricks ワークスペースを作成した場合、URL は `https://eastus.azuredatabricks.net` になる場合があります。 確実に行うには、Databricks ワークスペースを開き、ルート URL をコピーします
- `Token`: 前の手順で Databricks UI を使用して生成したトークン

CLI が正しく構成されていることをテストするには、次のコマンドを使用します。

```bash
databricks workspace ls
```

出力は次のようになります。

```bash
Shared
Users
```

## <a name="create-the-databricks-cluster"></a>Databricks クラスターを作成する

次の手順を使用して、新しい Databricks クラスターを作成します。

### <a name="create-the-json-metadata"></a>JSON メタデータを作成する

Databricks CLI を使用してクラスターを作成するには、JSON ファイルとしてそのメタデータが必要です。 以下の構造の、`cluster.json` という名前の新しい JSON ファイルを作成します。

```json
{
    "num_workers": null,
    "autoscale": {
        "min_workers": 2,
        "max_workers": 8
    },
    "cluster_name": "<cluster name>",
    "spark_version": "5.3.x-scala2.11",
    "spark_conf": {},
    "node_type_id": "Standard_DS3_v2",
    "ssh_public_keys": [],
    "custom_tags": {},
    "spark_env_vars": {
        "PYSPARK_PYTHON": "/databricks/python3/bin/python3"
    },
    "autotermination_minutes": 120,
    "init_scripts": []
}
```

`cluster_name`、`spark_version`、および `node_type_id` の固定値がある点に注意してください。 このソリューションを実行するときに、これらの値が引き続き有効である必要があります。

JSON ファイルが構成された後、次のコマンドを使用してクラスターを作成します。

```bash
databricks clusters create --json-file cluster.json
```

すぐに `cluster_id` を受け取ります。

```bash
{
    "cluster_id": "<cluster id>"
}
```

このクラスターの状態を確認するには、次のコマンドを使用します。

```bash
databricks clusters list
```

`RUNNING` と表示された場合、クラスターは使用する準備が整っています。

```bash
<cluster id>  <cluster name>  RUNNING
```

## <a name="create-databricks-notebooks-to-start-transforming"></a>Databricks ノートブックを作成して変換を開始する

Databricks でクラスターを適切に作成したら、ノートブックを作成して、取り込まれたデータの変換を開始します。

この時点では、実行できる多くの方法があります。 この詳細なソリューションのために、次の構造が提案されています。

- 以下のオブジェクトごとに 1 つのノートブック
    - 顧客
    - アドレス
    - 映画とアクターは結合されます
- 各ノートブックでは
    - データ レイク上のファイルからオブジェクト データを読み込む
    - さまざまなソース システムからそれらをまとめる
        - 列名、データ型を統一する
    - 統合データを Parquet としてデータ レイクに保存する

一部のセルは、この課題のために作成するノートブック間で共通のものとなります。 このため、各ノートブックの詳細なソリューションは、各ノートブックのステップごとに分割されます。

### <a name="notebooks-structure"></a>ノートブックの構造

変換ノートブックの構造は次のとおりです。

- 便利な関数を定義する
- 3 つのソース システムからデータを読み込む
- 連結されたデータを Parquet ファイルとして保存する

以下の詳細な手順を使用して、各ノートブックを構築します。

### <a name="notebooks-steps-details"></a>ノートブック ステップの詳細

#### <a name="customers-notebook"></a>顧客ノートブック

Databricks に新しいノートブックを作成し、`Customers` という名前を付けます。 作成したら、[それをソース管理に追加し](challenge-04/00-add-notebook-source-control.md)、次の手順に従います (セルごとに 1 つ):

- [1 - 便利な関数を定義する](challenge-04/01-define-useful-functions.md)
- 2 - 生データを読み込む
    - [2.1 - Southridge の顧客を読み込む](challenge-04/02-loading-raw-data/01-customers/01-loading-southridge-customers.md)
    - [2.2 - VanArsdel の顧客を読み込む](challenge-04/02-loading-raw-data/01-customers/02-load-vanarsdel-customers.md)
    - [2.3 - FourthCoffee の顧客を読み込む](challenge-04/02-loading-raw-data/01-customers/03-loading-fourthcoffee-customers.md)
- 3 - すべてをまとめる
    - [3.1 - すべての顧客を 1 つにまとめる](challenge-04/03-bring-all-together/01-bring-customers-together.md)
- [4 - データを Parquet としてデータ レイクに保存する](challenge-04/04-saving-data-as-parquet.md)

#### <a name="addresses-notebook"></a>アドレス ノートブック

Databricks に新しいノートブックを作成し、`Addresses` という名前を付けます。 作成したら、[それをソース管理に追加し](challenge-04/00-add-notebook-source-control.md)、次の手順に従います (セルごとに 1 つ):

- [1 - 便利な関数を定義する](challenge-04/01-define-useful-functions.md)
- 2 - 生データを読み込む
    - [2.1 - Southridge のアドレスを読み込む](challenge-04/02-loading-raw-data/02-addresses/01-loading-southridge-addresses.md)
    - [2.2 - VanArsdel アドレスを読み込む](challenge-04/02-loading-raw-data/02-addresses/02-load-vanarsdel-addresses.md)
    - [2.3 - FourthCoffee のアドレスを読み込む](challenge-04/02-loading-raw-data/02-addresses/03-loading-fourthcoffee-addresses.md)
- 3 - すべてをまとめる
    - [3.1 - すべてのアドレスをまとめる](challenge-04/03-bring-all-together/02-bring-addresses-together.md)
- [4 - データを Parquet としてデータ レイクに保存する](challenge-04/04-saving-data-as-parquet.md)

#### <a name="movies-and-actors-notebook"></a>映画とアクターのノートブック

Databricks に新しいノートブックを作成し、`MoviesAndActors` という名前を付けます。 作成したら、[それをソース管理に追加し](challenge-04/00-add-notebook-source-control.md)、次の手順に従います (セルごとに 1 つ):

- [1 - 便利な関数を定義する](challenge-04/01-define-useful-functions.md)
- 2 - 生データを読み込む
    - [2.1 - Southridge の映画とアクターを読み込む](challenge-04/02-loading-raw-data/03-movies-actors/01-loading-southridge-moviesandactors.md)
    - [2.2 - VanArsdel の映画とアクターを読み込む](challenge-04/02-loading-raw-data/03-movies-actors/02-loading-vanarsdel-moviesandactors.md)
    - [2.3 - FourthCoffee の映画とアクターを読み込む](challenge-04/02-loading-raw-data/03-movies-actors/03-loading-fourthcoffee-moviesandactors.md)
- 3 - すべてをまとめる
    - [3.1 - すべての映画とアクターをまとめる](challenge-04/03-bring-all-together/03-bring-movies-actors-together.md)
- [4 - データを Parquet としてデータ レイクに保存する](challenge-04/04-saving-data-as-parquet.md)

## <a name="learning-experiences-and-road-blocks"></a>学習エクスペリエンスと道路ブロック

このリファレンスは信用できないため、おそらく次の項目を追加できます。

- [DataFrame の概要 - Python](https://docs.databricks.com/spark/latest/dataframes-datasets/introduction-to-dataframes-python.html)

- CloudSales と CloudStreaming の顧客の間で重複を除去するための基準は何ですか?
- Southridge と FourthCoffee と VanArsdel の間には、共通する顧客が存在します。
これらを除去するための基準は何ですか?
- Southridge の映画の `ReleaseDate` について何をすべきかに関する意思決定ポイントがあります

## <a name="potential-feedbacks"></a>考えられるフィードバック

- PowerShell を使用したときに生成されるサービス プリンシパルのパスワードは表示されないようです。これは CLI を使用した場合に起きることとは異なっています。
この例では、ノートブックなどで使用するパスワードを表示してコピーする必要があります。PowerShell でパスワードを取得するオプションが提供されると便利です
- Databricks ワークスペースの価格レベルをアップグレードするには、既存のものと同じパラメーター値 (レベル以外) でそれを *再作成する* 必要があります。
[説明するドキュメントはこちらです](https://docs.azuredatabricks.net/administration-guide/account-settings/upgrade-downgrade.html)
- [このチュートリアル/リファレンス](https://docs.azuredatabricks.net/spark/latest/data-sources/azure/adls-passthrough.html#enable-passthrough-for-a-cluster)を課題 4 に追加することもできます。
    - 一方、このチュートリアルに従ってみましたが、資格情報パススルーを有効にするオプションは使用できませんでした。
    [スクリーンショットはこちらです](challenge-04/images/credential-passthrough-unavailable.jpg)。
