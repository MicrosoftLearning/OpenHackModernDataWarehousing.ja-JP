---
ms.openlocfilehash: 26b81fc0c3e9f686f8c6dd9d99021ea25fc02cf8
ms.sourcegitcommit: 3ff1164f4921cd2b423d97e7ff29cf3184026d57
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/22/2022
ms.locfileid: "140765912"
---
# <a name="creating-the-on-premises-sql-server-linked-service-to-use-the-self-hosted-integration-runtime"></a>セルフホステッド統合ランタイムを使用するためのオンプレミスの SQL Server のリンク サービスの作成

[このドキュメント](https://docs.microsoft.com/ja-jp/azure/data-factory/connector-sql-server#linked-service-properties)に基づいて作成した以前のリンク サービスとして、次の構造を持つ `SQL-Server-LinkedService.json` という名前のファイルを作成します。

```json
{
    "name": "<filesystem linked service name>",
    "properties": {
        "type": "SqlServer",
        "typeProperties": {
            "connectionString": {
                "type": "SecureString",
                "value": "Data Source=<servername>\\<instance name if using named instance>;Initial Catalog=<databasename>;Integrated Security=False;User ID=<username>;Password=<password>;"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

> これは JSON ファイルであるため、`connectionString` 値の二重円記号に注意してください。
>
> また、セルフホステッド統合ランタイム エージェントはそのマシン自体で実行されるため、`<servername>` は `localhost` として設定できることにも注意してください。

ファイルを正しく作成したら、次の PowerShell コマンドを実行して、Data Factory 上にリンク サービスを作成します。

```powershell
$resourceGroupName = "<resource group name>"
$dataFactoryName = "<data factory name>"
$linkedServiceName = "<sql server linked service name>"

Set-AzDataFactoryV2LinkedService `
    -DataFactoryName $dataFactoryName `
    -ResourceGroupName $resourceGroupName `
    -Name $linkedServiceName `
    -File "./SQL-Server-LinkedService.json"
```