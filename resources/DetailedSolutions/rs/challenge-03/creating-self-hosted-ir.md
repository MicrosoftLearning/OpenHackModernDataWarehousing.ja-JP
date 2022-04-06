---
ms.openlocfilehash: 8dadb406c33adb38bd188f281c908add181853e5
ms.sourcegitcommit: 3ff1164f4921cd2b423d97e7ff29cf3184026d57
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/22/2022
ms.locfileid: "140765890"
---
# <a name="creating-the-self-hosted-integration-runtime"></a>セルフホステッド統合ランタイムの作成

## <a name="01---creating-the-self-hosted-integration-runtime-reference-on-data-factory"></a>01 - Data Factory でのセルフホステッド統合ランタイム参照の作成

次の PowerShell コマンドを使用して、Data Factory に IR 参照を作成します。

```powershell
$dataFactoryName = "<data factory name>"
$resourceGroupName = "<resource group name>"
$selfHostedIntegrationRuntimeName = "<pipeline name>"
$selfHostedIntegrationRuntimeDescription = "<self-hosted IR description>"

Set-AzDataFactoryV2IntegrationRuntime `
    -ResourceGroupName $resourceGroupName `
    -DataFactoryName $dataFactoryName `
    -Name $selfHostedIntegrationRuntimeName `
    -Type SelfHosted `
    -Description $selfHostedIntegrationRuntimeDescription
```

## <a name="02---retrieve-the-self-hosted-ir-authentication-key"></a>02 - セルフホステッド IR 認証キーを取得する

このキーは、セルフホステッド IR エージェントを仮想マシンにインストールして登録するために必要です。 これを行うには、PowerShell を使用して次のコマンドを実行します。

```powershell
Get-AzDataFactoryV2IntegrationRuntimeKey `
    -ResourceGroupName $resourceGroupName `
    -DataFactoryName $dataFactoryName `
    -Name $selfHostedIntegrationRuntimeName
```

次のような出力が表示されます。

```powershell
AuthKey1    AuthKey2
--------    --------
<authkey1>  <authkey2>
```

これらのキーのいずれかをメモし、後で仮想マシンで使用します。

## <a name="03---download-install-and-register-the-self-hosted-ir-on-the-virtual-machine"></a>03 - セルフホステッド IR を仮想マシンにダウンロードし、インストールして登録する

IR を作成し、認証キーを取得したら、仮想マシンにインストーラーをダウンロードしてインストールを開始できます。 [このリンク](https://www.microsoft.com/en-us/download/details.aspx?id=39717)に従って、ダウンロード ページに移動します。

適切にインストールすると、次のようなウィンドウが自動的に開きます。

![セルフホステッド統合ランタイム構成ウィンドウ](self-hosted-ir/self-hosted-ir-config-screen.jpg)

強調表示されている場所に認証キーを貼り付け、 *[登録]* をクリックします。
登録パラメーターが検証されます。 成功メッセージが表示されたら、 *[Finish] (完了)* をクリックします。

登録が完了するまで待ち、セルフホステッド IR の *コントロール ペイン* で *[構成マネージャーの起動]* をクリックします。 ステータス バーに *クラウド サービスに接続しました (Data Factory V2)* と表示された場合は、セルフホステッド IR が正常にインストールされ、構成されています。