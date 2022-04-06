---
ms.openlocfilehash: 6446674f2883e3b72263dcc5149f7d5bb6b21583
ms.sourcegitcommit: 3ff1164f4921cd2b423d97e7ff29cf3184026d57
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/22/2022
ms.locfileid: "140765943"
---
# <a name="challenge-01---define-a-data-lake"></a>課題 01 - データ レイクを定義する

## <a name="requirements"></a>必要条件

課題 1 から 3 の解決のために、次のツールをインストールする必要があります。

- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
- [Az PowerShell モジュール](https://docs.microsoft.com/en-us/powershell/azure/install-az-ps?view=azps-1.7.0)

## <a name="creating-adls-gen2"></a>ADLS Gen2 の作成

[このガイド](https://docs.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-quickstart-create-account#create-an-account-using-azure-cli)を使用して、Azure Data Lake Storage Gen2 でストレージ アカウントを作成します。

```bash
# Create variables to have consistency throughout the tutorial
RESOURCE_GROUP="<resource group name>"
LOCATION="eastus"

# Install the storage-preview extension
az extension add --name storage-preview

# Create the Storage Account with ADLS Gen2 enabled
az storage account create \
  --name "<storage account name>" \
  --resource-group $RESOURCE_GROUP \
  --location $LOCATION \
  --sku Standard_LRS \
  --kind StorageV2 \
  --hierarchical-namespace true
```

## <a name="learning-experiences-and-road-blocks"></a>学習エクスペリエンスと道路ブロック

- Azure Cloud Shell を作成して使用するか、または
- Azure CLI のインストール
- 人びとは読まないと、Azure CLI に `storage-preview` 拡張機能をインストールする必要があることを理解できない

## <a name="potential-feedbacks"></a>考えられるフィードバック

- [このチュートリアル](https://docs.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-quickstart-create-account#create-an-account-using-azure-cli)には、 *"クイックスタート: Azure Data Lake Storage Gen2 ストレージ アカウントを作成する"* というタイトルが付いています。 ストレージ アカウントの **機能** における ADLS Gen2 をより具体的に表すには、それを次のようなタイトルにするように提案できます。

    > クイックスタート: Data Lake Storage Gen2 を有効にした状態で Azure Storage アカウントを作成する