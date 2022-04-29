---
ms.openlocfilehash: 50106c82620f6ba4b6fcada7e608047981f83620
ms.sourcegitcommit: 3ff1164f4921cd2b423d97e7ff29cf3184026d57
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/22/2022
ms.locfileid: "140765947"
---
# <a name="modern-data-warehousing-openhack-lab-deployment"></a>最新のデータ ウェアハウス OpenHack Lab のデプロイ

## <a name="overview"></a>概要

最新のデータ ウェアハウス OpenHack Lab 環境のデプロイには、次が含まれます。

### <a name="southridge-video-resources"></a>Southridge Video リソース

- 1 つの論理 SQL 内の 2 つの Azure SQL データベース
- ムービー カタログのコレクションを 1 つ持つ Cosmos DB アカウント

### <a name="fourth-coffee-resources"></a>Fourth Coffee リソース

- CSV データのディレクトリを持つ VM

### <a name="vanarsdel-ltd-resources"></a>VanArsdel, Ltd. リソース

- SQL DB を持つ VM

## <a name="quickstart"></a>クイックスタート

1. PowerShell セッションの `$sqlpwd` 変数と `$vmpwd` 変数を **セキュリティで保護された文字列** として割り当てます。 どちらにも強力なパスワードを使用してください。 仮想マシンのパスワード要件については[このリンク](https://docs.microsoft.com/ja-jp/azure/virtual-machines/windows/faq#what-are-the-password-requirements-when-creating-a-vm)に、SQL Server については[このリンク](https://docs.microsoft.com/ja-jp/sql/relational-databases/security/password-policy?view=sql-server-2017#password-complexity)に移動してください。

    ```powershell
    $sqlpwd = "0p3nH4ckD4t4!" | ConvertTo-SecureString -AsPlainText -Force
    $vmpwd = "0p3nH4ckD4t4!" | ConvertTo-SecureString -AsPlainText -Force
    ```

1. PowerShell セッションで `$containerSAS` 変数を **セキュリティで保護された文字列** としても割り当てます。 これは、データベースのバックアップを保持するコンテナーを対象とする rwl SAS です。 SAS は、これらの手順の外で (多くの場合、lab_backup_sas.txt というファイルとして) 提供されます。

    ```powershell
    $containerSAS = "TheContainerSASYouHave" | ConvertTo-SecureString -AsPlainText -Force
    ```

1. [Az モジュール](https://docs.microsoft.com/ja-jp/powershell/azure/new-azureps-module-az?view=azps-1.8.0)がインストールされていることを確認します。

1. OpenHack チームのサブスクリプションと関連する資格情報のリストを [Opsgility クラスルーム](https://aka.ms/ohadmin)からエクスポートします。 これを行うには、次の手順を実行します。  
    - クラスルームのラボ ビューにアクセスする  
    - すべてのチームが `In Use` である必要があります。そうではない場合は、それらを `Start` します。
    - すべてが `In Use` の場合は、`List credentials` をクリックします
    - 資格情報のポップアップを下スクロールして、`Export to CSV`

1. PowerShell セッションで `$credentialFile` 変数を割り当てます。 これは、前の手順で取得した資格情報ファイルへのパスです。

    ```powershell
    $credentialFile = "C:\Users\alias\oh-credentials.csv"
    ```

1. 次を実行して環境をデプロイします (このプロセスには 10 〜 15 分かかる場合があります)。

    ```powershell
    .\ExecuteARMDeploymentsForTeamSubscriptions.ps1 `
        -CredentialFile $credentialFile `
        -DeploymentTemplateFile ".\DeployMDWOpenHackLab.json" `
        -DeploymentParameterFile ".\DeployMDWOpenHackLab.parameters.json" `
        -ResourceGroupName mdw-oh-lab `
        -Location eastus `
        -SqlAdminLoginPassword $sqlpwd `
        -VMAdminPassword $vmpwd `
        -BackupStorageContainerSAS $containerSAS

1. A background job will now be running to deploy the MDW OpenHack lab resources into each team subscription.
To check the status of these jobs, use PowerShell job cmdlets:

    ```powershell
    Get-Job

    Receive-Job -Id 1
    ```