---
ms.openlocfilehash: 187ab7f7362e0fb16af55867f1cf59766790dc04
ms.sourcegitcommit: 3ff1164f4921cd2b423d97e7ff29cf3184026d57
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/22/2022
ms.locfileid: "140765891"
---
# <a name="creating-the-pipeline-to-copy-data-from-on-prem-sql-to-adls"></a>オンプレミスの SQL から ADLS にデータをコピーするパイプラインの作成

以下のリファレンスを使用します。

- [パイプライン JSON リファレンス](https://docs.microsoft.com/en-us/azure/data-factory/concepts-pipelines-activities#pipeline-json)
- [Azure Data Factory の ForEach アクティビティ](https://docs.microsoft.com/en-us/azure/data-factory/control-flow-for-each-activity)
- [SQL Server の Copy アクティビティのプロパティ](https://docs.microsoft.com/en-us/azure/data-factory/connector-sql-server#copy-activity-properties)

オンプレミスの SQL Server データベースから Azure Data Lake にデータをコピーする新しいパイプラインを作成します。

以下の構造の、`SQL-Server-Pipeline.json` という名前の新しい JSON ファイルを作成します。

```json
{
    "name": "<pipeline name>",
    "properties": {
        "activities": [
            {
                "name": "ForEach_Table",
                "type": "ForEach",
                "typeProperties": {
                    "items": {
                        "value": "@pipeline().parameters.items",
                        "type": "Expression"
                    },
                    "activities": [
                        {
                            "name": "Copy_Rows",
                            "type": "Copy",
                            "policy": {
                                "timeout": "7.00:00:00",
                                "retry": 0,
                                "retryIntervalInSeconds": 30,
                                "secureOutput": false,
                                "secureInput": false
                            },
                            "userProperties": [
                                {
                                    "name": "Source",
                                    "value": "@{item().source.tableName}"
                                },
                                {
                                    "name": "Destination",
                                    "value": "@{pipeline().parameters.destinationFolder}/@{item().destination.fileName}"
                                }
                            ],
                            "typeProperties": {
                                "source": {
                                    "type": "SqlSource"
                                },
                                "sink": {
                                    "type": "AzureBlobFSSink"
                                },
                                "enableStaging": false
                            },
                            "inputs": [
                                {
                                    "referenceName": "<source dataset name>",
                                    "type": "DatasetReference",
                                    "parameters": {
                                        "tableName": "@item().source.tableName"
                                    }
                                }
                            ],
                            "outputs": [
                                {
                                    "referenceName": "<sink dataset name>",
                                    "type": "DatasetReference",
                                    "parameters": {
                                        "fileName": "@item().destination.fileName",
                                        "filePath": "@pipeline().parameters.destinationFolder"
                                    }
                                }
                            ]
                        }
                    ]
                }
            }
        ],
        "parameters": {
            "items": {
                "type": "Array",
                "defaultValue": [
                    {
                        "source": {
                            "tableName": "[dbo].[Table1]"
                        },
                        "destination": {
                            "fileName": "Table1"
                        }
                    },
                    {
                        "source": {
                            "tableName": "[dbo].[Table2]"
                        },
                        "destination": {
                            "fileName": "Table2"
                        }
                    },
                    {
                        "source": {
                            "tableName": "[dbo].[TableN]"
                        },
                        "destination": {
                            "fileName": "TableN"
                        }
                    }
                ]
            },
            "destinationFolder": {
                "type": "String",
                "defaultValue": "<folder name>"
            }
        }
    },
    "type": "Microsoft.DataFactory/factories/pipelines"
}
```

次の値を置き換える点に注意してください。

- `<pipeline name>`: パイプラインの名前
- `<source dataset name>`: ソース データセットの名前
- `<sink dataset name>`: コピー先データセット (JSON) の名前
- `parameters:items`: シンクにコピーするテーブルの一覧。
テーブル名は `[dbo].[Table1]` のようになります。 それを、コピーして必要に応じて新しい項目を追加するテーブルの名前に置き換えます。
- `destinationFolder:defaultValue`: `<folder name>` を、VanArsdel ファイルを格納するフォルダーの名前に置き換えます。

> また、`parameters:items` については、このパイプラインでは非常に重要な命令である `ForEach` が利用されます。 **ForEach** は、特定の項目ごとに 1 つ以上のアクティビティ (この場合は `Copy_Rows`) を繰り返すループ命令です。
>
> `ForEach` アクティビティの詳細については、[このリンク](https://docs.microsoft.com/en-us/azure/data-factory/control-flow-for-each-activity)をご覧ください。

ファイルを適切に調整したら、以下の PowerShell コマンドを実行して、Data Factory にパイプラインを作成します。

```powershell
$dataFactoryName = "<data factory name>"
$resourceGroupName = "<resource group name>"
$pipelineName = "<pipeline name>"

Set-AzDataFactoryV2Pipeline `
    -DataFactoryName $dataFactoryName `
    -ResourceGroupName $resourceGroupName `
    -Name $pipelineName `
    -DefinitionFile ".\SQL-Server-Pipeline.json"
```