---
ms.openlocfilehash: 3f5c5c595657f8fe6fa05505dcfb1cae917521c0
ms.sourcegitcommit: 3ff1164f4921cd2b423d97e7ff29cf3184026d57
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/22/2022
ms.locfileid: "140766031"
---
# <a name="important"></a>重要 #

BYOS 用のパブリック リポジトリにこの情報を配置しない

# <a name="openhack---modern-data-warehousing"></a>OpenHack - 最新のデータ ウェアハウジング #

## <a name="openhack-guides"></a>OpenHack ガイド ##  

リマインダー: ガイドはメインのプライベート リポジトリに含まれています。  これらは、GitHub Actions を使用してビルドされ、Opsgility システムによってデプロイ中に使用されるストレージ アカウントにデプロイされます。  デプロイがトリガーされると、ラボの定義とポリシーだけがこのリポジトリから提供されます。  

コンテンツの定義とガイド (上記) はデプロイ中に使用されますが、それらの変更はメインのリポジトリ (これ) からのみ行う必要があります。  リポジトリがビルドされると、GitHub アクションによって、最新バージョンのガイドと lab-content-definition がストレージ アカウントに発行されます。  

デプロイ成果物はすべて BYOS サブスクリプションに格納されるため、個人でもこれを行うことができます。  それらはストレージ アカウントに存在しますが、ストレージ アカウントが失われた場合に備え、マスター コピーが BYOS に存在します。  

これに対する唯一の例外は、現在、2 つの BACPAC ファイルです。

次のリンクを使用すると、重要な情報が見つかります。  

*  ガイドはこちらにあります: [MDW OpenHack リポジトリ](https://github.com/Microsoft-OpenHack/modern-data-warehousing/tree/main/portal)  

*  コンテンツ定義はこちらにあります: [MDW コンテンツ定義](https://github.com/Microsoft-OpenHack/modern-data-warehousing/blob/main/portal/en/lab-content-definition.json)  

*  成果物はこちら: [デプロイ成果物](https://github.com/microsoft/OpenHack/tree/main/byos/modern-data-warehousing)  

*  デプロイ成果物とスクリプトは、ストレージ アカウントに発行されます: OpsgilityProd - openhackguides - mdw-templates-tmp  

## <a name="artifacts"></a>Artifacts ##  

次のいずれかの変更が必要な場合は、BYOS リポジトリ内でのみ行う必要があります。その後、ストレージ アカウント [上記の Deployment Artifacts ストレージ アカウントの場所] にあるものを上書きして、ファイルへの変更を手動でコピーするか、ガイドについてはメインの Microsoft OpenHack リポジトリから、またはデプロイ成果物については BYOS リポジトリからビルドをトリガーする必要があります。これにより、これらの項目が適切なストレージ アカウントに発行されます
            
## <a name="arm-templates"></a>ARM テンプレート ##  

次のテンプレートはデプロイ中に使用され、BYOS と Opsgility ストレージにも格納されます。 

### <a name="microsoft-byos-repo"></a>Microsoft BYOS リポジトリ ###  

* [DeployCosmosDB](https://github.com/microsoft/OpenHack/blob/main/byos/modern-data-warehousing/deploy/ARM/DeployCosmosDB.json)  
* [DeployFileVM](https://github.com/microsoft/OpenHack/blob/main/byos/modern-data-warehousing/deploy/ARM/DeployFileVM.json)  
* [DeployMDWOpenHackLab](https://github.com/microsoft/OpenHack/blob/main/byos/modern-data-warehousing/deploy/ARM/DeployMDWOpenHackLab.json)  
* [DeployMDWOpenHackLab Parameters](https://github.com/microsoft/OpenHack/blob/main/byos/modern-data-warehousing/deploy/ARM/DeployMDWOpanHackLab.parameters.json)    
* [DeploySQLAzure](https://github.com/microsoft/OpenHack/blob/main/byos/modern-data-warehousing/deploy/ARM/DeploySQLAzure.json)
* [DeploySQLVM](https://github.com/microsoft/OpenHack/blob/main/byos/modern-data-warehousing/deploy/ARM/DeploySQLVM.json)  

## <a name="scripts"></a>スクリプト ## 

###  <a name="microsoft-byos-repo"></a>Microsoft BYOS リポジトリ ###  

* [Cosmos の作成と設定](https://github.com/microsoft/OpenHack/blob/main/byos/modern-data-warehousing/deploy/Scripts/CreateAndPopulateCosmos.ps1)
* [SQL VM のデプロイ](https://github.com/microsoft/OpenHack/blob/main/byos/modern-data-warehousing/deploy/Scripts/DeploySQLVM.ps1)
* [IEESC の無効化](https://github.com/microsoft/OpenHack/blob/main/byos/modern-data-warehousing/deploy/Scripts/DisableIEESC.ps1)
* [ファイル VM 拡張機能ドライバー](https://github.com/microsoft/OpenHack/blob/main/byos/modern-data-warehousing/deploy/Scripts/FileVMExtensionDriver.ps1)
* [CSV の取得](https://github.com/microsoft/OpenHack/blob/main/byos/modern-data-warehousing/deploy/Scripts/RetrieveCSV.ps1)
* [SQL VM 拡張機能ドライバー](https://github.com/microsoft/OpenHack/blob/main/byos/modern-data-warehousing/deploy/Scripts/SqlVMExtensionDriver.ps1)  

## <a name="csv-files"></a>CSV ファイル ##  

### <a name="microsoft-byos-repo"></a>Microsoft BYOS リポジトリ ###  

* [レンタル数](https://github.com/microsoft/OpenHack/blob/main/byos/modern-data-warehousing/deploy/CSVFiles/Rentals.zip)  
* [トランザクション](https://github.com/microsoft/OpenHack/blob/main/byos/modern-data-warehousing/deploy/CSVFiles/Transactions_2018.zip)  

## <a name="deployment-resource-files-sas-token-required"></a>デプロイ リソース ファイル [SAS トークンが必要] ##  

### <a name="opsgility-storage"></a>Opsgility ストレージ ###  

* [CloudSales BACPAC](https://openhackartifacts.blob.core.windows.net/mdw/CloudSales.bacpac)  
* [CloudStreaming BACPAC](https://openhackartifacts.blob.core.windows.net/mdw/CloudStreaming.bacpac)
* [movies_southridge JSON](https://openhackartifacts.blob.core.windows.net/mdw/movies_southridge.json)  
* [OnPremRentals バックアップ ファイル](https://openhackartifacts.blob.core.windows.net/mdw/onpremrentals.bak)  
* CSV ファイルの場合は、ストレージ アカウント内の `Rentals` コンテナー フォルダーと `onpremrentalscsv` コンテナー フォルダーを確認します。  これらは、BYOS リポジトリ内の ZIP ファイルにも含まれています。  