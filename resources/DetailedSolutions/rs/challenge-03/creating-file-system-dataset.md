---
ms.openlocfilehash: 1df8ec6e8f41086aeee50f22d37444fb2cd66bd0
ms.sourcegitcommit: 3ff1164f4921cd2b423d97e7ff29cf3184026d57
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/22/2022
ms.locfileid: "140765957"
---
# <a name="creating-a-dataset-for-file-system-sources"></a>ファイル システム ソース用のデータセットの作成

[このリファレンス](https://docs.microsoft.com/ja-jp/azure/data-factory/connector-file-system#dataset-properties)を使用して、ファイル システム ソースのデータセットを作成します。

新しい JSON ファイルを作成し、次の構造体で `File-System-Dataset.json` という名前を付けます。

```json
{
    "name": "<file system dataset name>",
    "properties": {
        "type": "FileShare",
        "linkedServiceName":{
            "referenceName": "<file system linked service name>",
            "type": "LinkedServiceReference"
        },
        "parameters": {
            "fileName": {
                "type": "String"
            }
        },
        "typeProperties": {
            "folderPath": "<folder/subfolder/ you will copy data from>",
            "fileName": "@dataset().fileName",
            "modifiedDatetimeStart": "2018-12-01T05:00:00Z",
            "modifiedDatetimeEnd": "2020-12-01T06:00:00Z",
            "format": {
                "type": "TextFormat",
                "columnDelimiter": ",",
                "rowDelimiter": "\n",
                "firstRowAsHeader": true
            }
        }
    }
}
```

> `properties:typeProperties:folderPath` は、[ファイル システム リンク サービス](creating-file-system-linked-service.md)で構成したルート パスを基準にしていることに注意してください。
> リンク サービスで指定したフォルダーからファイルをコピーする場合は、このプロパティの値として "." を使用してください。
>
> また、オンプレミスの仮想マシン上のファイルの `modifiedDate` プロパティが `properties:typeproperties:modifiedDatetimeStart` と `properties:typeproperties:modifiedDatetimeEnd` の間にあることを確認してください。

ファイルを作成したら、次の PowerShell コマンドを使用して Azure Data Factory にデータセットを作成します。

```powershell
$dataFactoryName = "<data factory name>"
$resourceGroupName = "<resource group name>"
$datasetName = "<dataset name>"

Set-AzDataFactoryV2Dataset `
    -DataFactoryName $dataFactoryName `
    -ResourceGroupName $resourceGroupName `
    -Name $datasetName `
    -DefinitionFile ".\File-System-Dataset.json"
```