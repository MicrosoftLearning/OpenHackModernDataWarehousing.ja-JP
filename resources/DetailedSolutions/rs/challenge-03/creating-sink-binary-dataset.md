---
ms.openlocfilehash: 9cb89d0557368dec223f0c1a15c64c4e2cf3a3df
ms.sourcegitcommit: 3ff1164f4921cd2b423d97e7ff29cf3184026d57
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/22/2022
ms.locfileid: "140765888"
---
# <a name="creating-a-binary-dataset-for-sink"></a>シンク用のバイナリ データセットの作成

ソースからファイルを *そのまま* コピーすることが必要な場合があります。 たとえば、CSV ファイルがあるオンプレミスのマシンがあって、[Parquet](https://docs.microsoft.com/ja-jp/azure/data-factory/supported-file-formats-and-compression-codecs#parquet-format) ファイルを生成するために必要な要件がないとします。 これらの CSV ファイルを *そのまま* コピーして、さらに処理を行うことができます。

[このリファレンス](https://docs.microsoft.com/ja-jp/azure/data-factory/connector-azure-data-lake-storage#dataset-properties)を使用して、Azure Data Lake Storage Gen2 のデータ形式をシンクとして記述するためのデータセットを作成します。

次の構造を持つ `ADLS-Dataset-Binary.json` という名前の新しい JSON ファイルを作成します。

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
            "fileName": {
                "value": "@dataset().fileName",
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

必ず `<ADLS Linked Service name>` の正しい名前を使用するようにしてください。
また、データセットにも適切な名前 (`FourthCoffee_ADLS_Binary` など) を付けるようにしてください。

次に、次のコマンドを使用して Data Factory 上にデータセットを作成します。

```powershell
$dataFactoryName = "<data factory name>"
$resourceGroupName = "<resource group name>"
$datasetName = "<dataset name>"

Set-AzDataFactoryV2Dataset `
    -DataFactoryName $dataFactoryName `
    -ResourceGroupName $resourceGroupName `
    -Name $datasetName `
    -DefinitionFile ".\ADLS-Dataset-Binary.json"