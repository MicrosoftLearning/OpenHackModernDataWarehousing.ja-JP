---
ms.openlocfilehash: b9514d06c7093e6cd7b81cd13e3dfea7526028aa
ms.sourcegitcommit: 3ff1164f4921cd2b423d97e7ff29cf3184026d57
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/22/2022
ms.locfileid: "140765938"
---
# <a name="creating-the-file-system-linked-service-to-use-the-self-hosted-integration-runtime"></a>セルフホステッド統合ランタイムを使用するためのファイル システム リンク サービスの作成

[このドキュメント](https://docs.microsoft.com/ja-jp/azure/data-factory/connector-file-system#linked-service-properties)に基づいて作成した以前のリンク サービスとして、次の構造を持つ `FileSystem-LinkedService.json` という名前のファイルを作成します。

```json
{
    "name": "<filesystem linked service name>",
    "properties": {
        "type": "FileServer",
        "typeProperties": {
            "host": "<host>",
            "userid": "<domain>\\<user>",
            "password": {
                "type": "SecureString",
                "value": "<password>"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

ここの `<host>` は仮想マシン ホスト名 **ではない** ことに注意してください。 このリンク サービス経由で処理および共有しようとしているルート フォルダー (`C:\\Rentals` など) を指定する必要があります。

必ず `<domain>\\<user>`、`<password>`、`<name of Integration Runtime>` も適切な値に置き換えてください。

> また、これは JSON ファイルであるため、二重円記号にも注意してください。
>
> セキュリティ侵害を防ぐために、C: ドライブ全体をリンク サービスとして公開することは避けてください。 代わりに、オンプレミスのマシン上にあるドライブ サブフォルダーごとに異なるリンク サービスを作成します。

ファイルを正しく作成したら、次の PowerShell コマンドを実行して、Data Factory 上にリンク サービスを作成します。

```powershell
# Creating the values to avoid typing errors
$resourceGroupName = "<resource group name>"
$dataFactoryName = "<data factory name>"
$linkedServiceName = "<filesystem linked service name>"

Set-AzDataFactoryV2LinkedService `
    -DataFactoryName $dataFactoryName `
    -ResourceGroupName $resourceGroupName `
    -Name $linkedServiceName `
    -File "./FileSystem-LinkedService.json"
```