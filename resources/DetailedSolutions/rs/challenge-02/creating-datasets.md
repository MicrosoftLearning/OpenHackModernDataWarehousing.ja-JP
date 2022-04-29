---
ms.openlocfilehash: c92ae1db0dd0204be596af62e44851cb9769c886
ms.sourcegitcommit: 3ff1164f4921cd2b423d97e7ff29cf3184026d57
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/22/2022
ms.locfileid: "140765939"
---
# <a name="creating-the-datasets-on-data-factory"></a>Data Factory 上でのデータセットの作成

データセットを作成するときは、コピー元またはコピー先のデータを表すデータセットを定義します。 どのデータセットも、前の手順で作成したリンク サービスに関連付けられています。

## <a name="01---create-the-dataset-for-the-cloudsales"></a>01 - CloudSales のデータセットを作成する

> データセットの名前には、文字、数字、'_' のみを使用できます。

[このリファレンス](https://docs.microsoft.com/ja-jp/azure/data-factory/quickstart-create-data-factory-powershell#create-a-dataset)を使用して、次の構造を持つ `CloudSales-Dataset.json` という名前の新しい JSON ファイルを作成します。

```json
{
    "name": "<dataset name>",
    "properties": {
        "linkedServiceName": {
            "referenceName": "<CloudSales Linked Service Name>",
            "type": "LinkedServiceReference"
        },
        "parameters": {
            "tableName": {
                "type": "String"
            }
        },
        "type": "AzureSqlTable",
        "typeProperties": {
            "tableName": {
                "value": "@dataset().tableName",
                "type": "Expression"
            }
        }
    },
    "type": "Microsoft.DataFactory/factories/datasets"
}
```

必ず `<CloudSales Linked Service Name>` の正しい名前を使用するようにしてください。
また、データセットにも適切な名前 (`CloudSales_SQL` など) を付けるようにしてください。

次に、次のコマンドを使用して Data Factory 上にデータセットを作成します。

```powershell
$dataFactoryName = "<data factory name>"
$resourceGroupName = "<resource group name>"
$datasetName = "<dataset name>"

Set-AzDataFactoryV2Dataset `
    -DataFactoryName $dataFactoryName `
    -ResourceGroupName $resourceGroupName `
    -Name $datasetName `
    -DefinitionFile ".\CloudSales-Dataset.json"
```

## <a name="02---create-the-dataset-for-the-cloudstreaming"></a>02 - CloudStreaming のデータセットを作成する

前の手順を繰り返して、CloudStreaming データベースのデータセットを作成します。
確認する必要がある、前と異なる点は次のとおりです。

- データセットの JSON 定義ファイルの `linkedService:ReferenceName`
- JSON 定義ファイル上のデータセットの `name`
- データセットを作成するための PowerShell スクリプトの `<dataset name>` (これは、JSON 定義ファイルで使用したものと同じ名前にする必要がある)

## <a name="03---create-the-dataset-for-the-movies-catalog-on-cosmosdb"></a>03 - CosmosDB 上の Movies カタログのデータセットを作成する

[このリファレンス](https://docs.microsoft.com/ja-jp/azure/data-factory/connector-azure-cosmos-db#dataset-properties)を使用して、Cosmos DB 上の Movies データベースのデータ形式を定義するためのデータセットを作成します。

次の構造を持つ `Movies-Dataset.json` という名前の新しい JSON ファイルを作成します。

```json
{
    "name": "<dataset name>",
    "properties": {
        "type": "DocumentDbCollection",
        "linkedServiceName":{
            "referenceName": "<Movies Linked Service Name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "collectionName": "<collection name>"
        }
    }
}
```

必ず `<Movies Linked Service Name>` の正しい名前を使用するようにしてください。
また、`<collection name>` を、このプロセスを使用して取り込む CosmosDB データベース コレクションの名前に置き換えることも忘れないでください。 このサンプルでは、`movies` を使用します。

次に、次のコマンドを使用して Data Factory 上にデータセットを作成します。

```powershell
$dataFactoryName = "<data factory name>"
$resourceGroupName = "<resource group name>"
$datasetName = "<dataset name>"

Set-AzDataFactoryV2Dataset `
    -DataFactoryName $dataFactoryName `
    -ResourceGroupName $resourceGroupName `
    -Name $datasetName `
    -DefinitionFile ".\Movies-Dataset.json"
```

## <a name="04---create-the-sink-dataset-for-azure-sql-database-tables"></a>04 - Azure SQL Database テーブルのシンク データセットを作成する

[このリファレンス](https://docs.microsoft.com/ja-jp/azure/data-factory/connector-azure-data-lake-storage#dataset-properties)を使用して、Azure Data Lake Storage Gen2 のデータ形式を Parquet ファイルのシンクとして記述するためのデータセットを作成します。 このデータセットは、Azure SQL Database テーブルをコピーするために使用されます。

> Azure SQL からデータをコピーする Southridge の場合は、既にデータを Parquet としてコピーできるようになっているため、後で変換のために読み込むことが簡単になります。

次の構造を持つ `ADLS-Dataset-Parquet.json` という名前の新しい JSON ファイルを作成します。

```json
{
    "name": "<dataset name>",
    "properties": {
        "linkedServiceName": {
            "referenceName": "<ADLS Linked Service name>",
            "type": "LinkedServiceReference"
        },
        "parameters": {
            "tableName": {
                "type": "String"
            },
            "filePath": {
                "type": "string"
            }
        },
        "type": "AzureBlobFSFile",
        "typeProperties": {
            "format": {
                "type": "ParquetFormat"
            },
            "fileName": {
                "value": "@{concat(dataset().tableName, '.parquet')}",
                "type": "Expression"
            },
            "folderPath": {
                "value": "@dataset().filePath",
                "type": "Expression"
            }
        }
    },
    "type": "Microsoft.DataFactory/factories/datasets"
}
```

必ず、正しいリンク サービス名を使用して `<ADLS Linked Service name>` を置き換えるようにしてください。

- `@dataset().filePath` は、パイプラインによってファイルが保存されるフォルダー構造を定義するために使用されます。
- `@dataset().tableName` は、Parquet ファイルを自身で保存できるように、テーブル名を受け取ります。

次に、次のコマンドを使用して Data Factory 上にデータセットを作成します。

```powershell
$dataFactoryName = "<data factory name>"
$resourceGroupName = "<resource group name>"
$datasetName = "<dataset name>"

Set-AzDataFactoryV2Dataset `
    -DataFactoryName $dataFactoryName `
    -ResourceGroupName $resourceGroupName `
    -Name $datasetName `
    -DefinitionFile ".\ADLS-Dataset-Parquet.json"
```

## <a name="05---create-the-sink-dataset-for-the-movies-catalog"></a>05 - Movies カタログのシンク データセットを作成する

[このリファレンス](https://docs.microsoft.com/ja-jp/azure/data-factory/connector-azure-data-lake-storage#dataset-properties)を使用して、Azure Data Lake Storage Gen2 のデータ形式を JSON ファイルのシンクとして記述するためのデータセットを作成します。

次の構造を持つ `ADLS-Dataset-JSON.json` という名前の新しい JSON ファイルを作成します。

```json
{
    "name": "<dataset name>",
    "properties": {
        "linkedServiceName": {
            "referenceName": "<ADLS Linked Service name>",
            "type": "LinkedServiceReference"
        },
        "parameters": {
            "fileName": {
                "type": "String"
            },
            "filePath": {
                "type": "string"
            }
        },
        "type": "AzureBlobFSFile",
        "typeProperties": {
            "format": {
                "type": "JsonFormat",
                "filePattern": "arrayOfObjects"
            },
            "fileName": {
                "value": "@{concat(dataset().fileName, '.json')}",
                "type": "Expression"
            },
            "folderPath": {
                "value": "@dataset().filePath",
                "type": "Expression"
            }
        }
    },
    "type": "Microsoft.DataFactory/factories/datasets"
}
```

必ず `<CloudSales Linked Service Name>` の正しい名前を使用するようにしてください。
また、データセットにも適切な名前 (`ADLS_CloudSales_JSON` など) を付けるようにしてください。

次に、次のコマンドを使用して Data Factory 上にデータセットを作成します。

```powershell
$dataFactoryName = "<data factory name>"
$resourceGroupName = "<resource group name>"
$datasetName = "<dataset name>"

Set-AzDataFactoryV2Dataset `
    -DataFactoryName $dataFactoryName `
    -ResourceGroupName $resourceGroupName `
    -Name $datasetName `
    -DefinitionFile ".\ADLS-Dataset-JSON.json"
```