---
ms.openlocfilehash: fa20fdaff219893e3d07f591c0335be55a6150c2
ms.sourcegitcommit: 3ff1164f4921cd2b423d97e7ff29cf3184026d57
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/22/2022
ms.locfileid: "140765914"
---
# <a name="creating-a-dataset-for-an-on-premises-sql-server-database"></a>オンプレミスの SQL Server データベースのデータセットを作成する

[このリファレンス](https://docs.microsoft.com/en-us/azure/data-factory/connector-sql-server#dataset-properties)を利用して、以下の構造の `SQL-Server-Dataset.json` という名前の新しい JSON ファイルを作成します。

```json
{
    "name": "<dataset name>",
    "properties": {
        "linkedServiceName": {
            "referenceName": "<On prem SQL Server Linked Service Name>",
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

`<On prem SQL Server Linked Service Name>` の正しい名前を使用していることをご確認ください。

以下のコマンドを使用して、Data Factory にデータセットを作成します。

```powershell
$dataFactoryName = "<data factory name>"
$resourceGroupName = "<resource group name>"
$datasetName = "<dataset name>"

Set-AzDataFactoryV2Dataset `
    -DataFactoryName $dataFactoryName `
    -ResourceGroupName $resourceGroupName `
    -Name $datasetName `
    -DefinitionFile ".\SQL-Server-Dataset.json"
```