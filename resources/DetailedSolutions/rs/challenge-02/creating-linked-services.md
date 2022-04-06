---
ms.openlocfilehash: 979d99a39ac1d17f41ac86baf14e386510ffacaf
ms.sourcegitcommit: 3ff1164f4921cd2b423d97e7ff29cf3184026d57
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/22/2022
ms.locfileid: "140765967"
---
# <a name="creating-the-linked-services-for-azure-data-factory"></a>Azure Data Factory 用のリンク サービスを作成する

## <a name="01---configure-linked-services-for-azure-sql-databases"></a>01 - Azure SQL データベース用のリンク サービスを構成する

[このリファレンス](https://docs.microsoft.com/en-us/azure/data-factory/connector-azure-sql-database#linked-service-properties)を利用して、Azure SQL をデータ ソースとして使用するための Data Factory リンク サービスを作成します。

以下の構造の、`CloudSales-LinkedService.json` という名前の新しい JSON ファイルを作成します。

```json
{
    "name": "<Name for the Linked Service>",
    "properties": {
        "type": "AzureSqlDatabase",
        "typeProperties": {
            "connectionString": {
                "type": "SecureString",
                "value": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
            }
        }
    }
}
```

置き換える必要がある次の値に注目してください。

- `<Name for the Linked Service>`
- SQL Server `<servername>`
- SQL `<databasename>`
- SQL Server `<username>`
- SQL SQL Server `<password>`

PowerShell を使用して以下のコマンドを実行し、このリンク サービスを作成します。

> これらのコマンドを Azure Cloud Shell で実行する場合は、必ず、続行する前に JSON ファイルをアップロードし、ファイルの場所をメモしてください。

```powershell
# Creating the values to avoid typing errors
$resourceGroupName = "<resource group name>"
$dataFactoryName = "<data factory name>"
$linkedServiceName = "<cloud sales linked service name>"

Set-AzDataFactoryV2LinkedService `
    -DataFactoryName $dataFactoryName `
    -ResourceGroupName $resourceGroupName `
    -Name $linkedServiceName `
    -File "./CloudSales-LinkedService.json"
```

Cloud Streaming データベースの場合は、前に作成した JSON ファイルを複製し、正しいデータベースを指すようにプロパティだけを変更し、別のリンク サービス名を作成することができます。 それで、以下のプロパティを注意深く更新してください。

- ファイル名 (例: `CloudStreaming-LinkedService.json`)
- `<Name for the Linked Service>`
- `<databasename>`

> 両方のデータベースが同じサーバー上にあるため、他のプロパティは厳密に更新する必要はありません。 実際のシナリオでは、別々のサーバーと資格情報が使用される可能性が高いです。

Cloud Streaming リンク サービスを作成するためのコマンドは、次のようになります。

```powershell
# Reassigning the linkedServiceName variable
$linkedServiceName = "<cloud streaming linked service name>"

Set-AzDataFactoryV2LinkedService `
    -DataFactoryName $dataFactoryName `
    -ResourceGroupName $resourceGroupName `
    -Name $linkedServiceName `
    -File "./CloudStreaming-LinkedService.json"
```

## <a name="02---configure-a-linked-service-for-the-cosmos-db-as-a-source"></a>02 - Cosmos DB をソースとするためのリンク サービスを構成する

[このリファレンス](https://docs.microsoft.com/en-us/azure/data-factory/connector-azure-cosmos-db#linked-service-properties)を利用して、Cosmos DB をデータ ソースとして使用するためのリンク サービスを作成します。

まず、Azure から CosmosDB の完全な `connectionString` を取得する必要があります。
これを行うには、次の Azure CLI コマンドを使用します。

```powershell
$resourceGroupName = "<resource group name>"
$cosmosDbAccountName = "<the CosmosDB account name>"

az cosmosdb list-connection-strings \
    -n $cosmosDbAccountName
    -g $resourceGroupName
```

> このアクションは現在、Az モジュールでサポートされていません。

次のような出力が表示されます。

```json
{
  "connectionStrings": [
    {
      "connectionString": "AccountEndpoint=https://<cosmosdb name>.documents.azure.com:443/;AccountKey=<account key1>",
      "description": "Primary SQL Connection String"
    },
    {
      "connectionString": "AccountEndpoint=https://<cosmosdb name>.documents.azure.com:443/;AccountKey=<account key2>",
      "description": "Secondary SQL Connection String"
    },
    {
      "connectionString": "AccountEndpoint=https://<cosmosdb name>.documents.azure.com:443/;AccountKey=<read-only account key1>",
      "description": "Primary Read-Only SQL Connection String"
    },
    {
      "connectionString": "AccountEndpoint=https://<cosmosdb name>.documents.azure.com:443/;AccountKey=<read-only account key2>",
      "description": "Secondary Read-Only SQL Connection String"
    }
  ]
}
```

> CosmosDB がデータ ソースとして使用されるため、いずれかの `Read-Only SQL Connection String` 接続文字列を使用することをお勧めします。

お気づきのように、いずれの接続文字列も文字列の末尾で `Database` が指定されていないため、それに留意し、データの抽出元になるデータベースを指定してください。

> Azure portal で Cosmos DB データ エクスプローラーから *データベース名* の一覧を取得できます。 Azure CLI を使用してそれを取得するには、次のコマンドを使用します。

```cmd
az cosmosdb database list \
-n <Azure Cosmos DB account name> \
-g `<resource group name>`
```

> このサンプルの範囲では、データベースを 1 つだけ使用します。
> 複数ある場合は、そのうちの 1 つだけ選択し、`id` フィールドの値をデータベース名として使用します。

CosmosDB データ ソースからすべてのデータを取得したので、以下の構造の、`Movies-LinkedService.json` という名前の新しい JSON ファイルを作成します。

```json
{
    "name": "<Name for the Movies Linked Service>",
    "properties": {
        "type": "CosmosDb",
        "typeProperties": {
            "connectionString": {
                "type": "SecureString",
                "value": "AccountEndpoint=<EndpointUrl>;AccountKey=<AccessKey>;Database=<Database>"
            }
        }
    }
}
```

以下のコマンドを使用して、CosmosDB リンク サービスを作成します。

```powershell
# Reassigning the linkedServiceName variable
$linkedServiceName = "<Name for the Movies Linked Service>"

Set-AzDataFactoryV2LinkedService `
    -DataFactoryName $dataFactoryName `
    -ResourceGroupName $resourceGroupName `
    -Name $linkedServiceName `
    -File "./Movies-LinkedService.json"
```

## <a name="03---configure-a-linked-service-for-the-data-lake-as-a-sink"></a>03 - Data Lake をシンクとするためのリンク サービスを構成する

[このリファレンス](https://docs.microsoft.com/en-us/azure/data-factory/connector-azure-data-lake-storage#linked-service-properties)を利用して、前に作成した Data Lake をパイプラインのシンクとして使用するためのリンク サービスを作成します。

まず、Data Factory が ADLS Gen2 で読み取りと書き込みを実行できるようにするために、ストレージ アカウントのキーが必要になります。 これを行うには、以下のコマンドを使用して ADLS Gen2 の `accountKey` を取得します。

```powershell
Get-AzStorageAccountAccountKey `
    -ResourceGroupName $resourceGroupName `
    -Name "<storage account name>"
```

次のような結果が表示されます。

```powershell
KeyName Value               Permissions
------- -----               -----------
key1    <key1 value>        Full
key2    <key2 value>        Full
```

JSON ファイルで `<key1 value>` または `<key2 value>` のいずれかを `<accountkey>` として使用します。

`accountKey` を用意できたので、以下の構造の、`ADLS-LinkedService.json` という名前の新しい JSON ファイルを作成します。

```json
{
    "name": "<Name for the Data Lake Linked Service>",
    "properties": {
        "type": "AzureBlobFS",
        "typeProperties": {
            "url": "https://<accountname>.dfs.core.windows.net",
            "accountkey": {
                "type": "SecureString",
                "value": "<accountkey>"
            }
        }
    }
}
```

以下のコマンドを使用して、Data Lake Gen2 リンク サービスを作成します。

```powershell
# Reassigning the linkedServiceName variable
$linkedServiceName = "<Name for the Data Lake Linked Service>"

Set-AzDataFactoryV2LinkedService `
    -DataFactoryName $dataFactoryName `
    -ResourceGroupName $resourceGroupName `
    -Name $linkedServiceName `
    -File "./ADLS-LinkedService.json"
```