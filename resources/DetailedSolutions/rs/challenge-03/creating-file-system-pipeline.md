---
ms.openlocfilehash: 51b01ebe45757aef564461eb2e23632eb2afcfb6
ms.sourcegitcommit: 3ff1164f4921cd2b423d97e7ff29cf3184026d57
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/22/2022
ms.locfileid: "140765975"
---
# <a name="creating-the-pipeline-to-copy-data-from-on-prem-sql-to-adls"></a>オンプレミスの SQL から ADLS にデータをコピーするパイプラインの作成

以下のリファレンスを使用します。

- [パイプライン JSON リファレンス](https://docs.microsoft.com/ja-jp/azure/data-factory/concepts-pipelines-activities#pipeline-json)
- [Azure Data Factory の ForEach アクティビティ](https://docs.microsoft.com/ja-jp/azure/data-factory/control-flow-for-each-activity)
- [コピー アクティビティのプロパティ](https://docs.microsoft.com/ja-jp/azure/data-factory/connector-file-system#copy-activity-properties)

オンプレミスのファイル システムから Azure Data Lake にデータをコピーする新しいパイプラインを作成します。

以下の構造の、`File-System-Pipeline.json` という名前の新しい JSON ファイルを作成します。


```json
{
    "name": "<pipeline name>",
    "properties": {
        "activities":[
            {
                "name": "ForEach_File",
                "type": "ForEach",
                "typeProperties": {
                    "isSequential": "true",
                    "items": {
                        "value": "@pipeline().parameters.files",
                        "type": "Expression"
                    },
                    "activities": [
                        {
                            "name": "CopyFromFileSystem",
                            "type": "Copy",
                            "inputs": [
                                {
                                    "referenceName": "<source dataset name>",
                                    "type": "DatasetReference",
                                    "parameters": {
                                        "fileName": "@item()"
                                    }
                                }
                            ],
                            "outputs": [
                                {
                                    "referenceName": "<sink dataset name>",
                                    "type": "DatasetReference",
                                    "parameters": {
                                        "fileName": "@item()",
                                        "filePath": "@pipeline().parameters.destinationFolder"
                                    }
                                }
                            ],
                            "typeProperties": {
                                "source": {
                                    "type": "FileSystemSource",
                                    "recursive": true
                                },
                                "sink": {
                                    "type": "AzureBlobFSSink"
                                }
                            }
                        }
                    ]
                }
            }
        ],
        "parameters": {
            "files": {
                "type": "Array",
                "defaultValue": [
                    "file1.ext",
                    "file2.ext",
                    "file3.ext",
                    "fileN.ext"
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
- `<sink dataset name>`: コピー先データセット (バイナリ) の名前
- `parameters:files`: シンクにコピーするファイルの一覧。
ファイル名は `fileN.ext` のようになります。 それを、コピーするファイルの名前とその拡張子に置き換え、必要に応じて新しい ites を追加します。
- `destinationFolder:defaultValue`: `<folder name>` を、FourthCoffee ファイルを格納するフォルダーの名前に置き換えます。

ファイルを適切に調整したら、以下の PowerShell コマンドを実行して、Data Factory にパイプラインを作成します。

```powershell
$dataFactoryName = "<data factory name>"
$resourceGroupName = "<resource group name>"
$pipelineName = "<pipeline name>"

Set-AzDataFactoryV2Pipeline `
    -DataFactoryName $dataFactoryName `
    -ResourceGroupName $resourceGroupName `
    -Name $pipelineName `
    -DefinitionFile ".\File-System-Pipeline.json"
```