---
ms.openlocfilehash: ad21585456e30ecfdc75eb0aa09580ecf1f80407
ms.sourcegitcommit: 3ff1164f4921cd2b423d97e7ff29cf3184026d57
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/22/2022
ms.locfileid: "140765911"
---
# <a name="defining-useful-functions"></a>便利な関数を定義する

この課題のすべてのノートブックにおける最初のセルでは、この課題の残りの部分のために重要な関数を宣言する必要があります。

個々のノートブックは、それ自体の実行前に別のものが実行されたかどうかを識別できないため、Data Lake ファイル システムが Databricks クラスターにマウントされており、インジェスト中にそこに格納されたファイルを読み込み、そのデータを変換することができることを確かめる必要があります。

また、変換の結果を Parquet として Data Lake に保存するのに役立つ関数も宣言します。

以下のコードで、ノートブックの実行における残りの部分のために準備を整えます。

```python
import pandas as pd

def mountFileSystem(containerName, storageAccountName):
  configs = {"fs.azure.account.auth.type": "OAuth",
       "fs.azure.account.oauth.provider.type": "org.apache.hadoop.fs.azurebfs.oauth2.ClientCredsTokenProvider",
       "fs.azure.account.oauth2.client.id": "<Service Principal Application ID>",
       "fs.azure.account.oauth2.client.secret": "<Service Principal Application Secret (or password)>",
       "fs.azure.account.oauth2.client.endpoint": "https://login.microsoftonline.com/<Service Principal Tenant ID>/oauth2/token",
       "fs.azure.createRemoteFileSystemDuringInitialization": "true"}
  
  mountPoint = "/mnt/adls/" + containerName
  
  try:
    dbutils.fs.mount(
      source = "abfss://" + containerName + "@" + storageAccountName + ".dfs.core.windows.net",
      mount_point = mountPoint,
      extra_configs = configs
    )
    print(mountPoint + " mounted successfully")
  except:
    print("The mount " + mountPoint + " already exists.")

  return mountPoint

def saveAsParquet(dataFrame, filePath):
  df = sqlContext.createDataFrame(dataFrame)

  df.write.parquet(filePath, mode='overwrite')

  print(filePath + " saved successfully")

mountPoint = mountFileSystem("southridge", "<storage account name>")
print(mountPoint)
```

## <a name="detailing-the-code-above"></a>上記のコードの詳細

上記のコードは、分かりやすく説明できるように、区分する意味のある部分に分割されています。

### <a name="defining-the-mountfilesystem-function"></a>mountFileSystem 関数を定義する

```python
import pandas as pd
```

> Pandas を使用して、取り込んだデータを読み込んで変換します。
> この行では、Panda モジュールをインポートして、ノートブック全体で使用できるようにします。

```python
def mountFileSystem(containerName, storageAccountName):
  configs = {"fs.azure.account.auth.type": "OAuth",
       "fs.azure.account.oauth.provider.type": "org.apache.hadoop.fs.azurebfs.oauth2.ClientCredsTokenProvider",
       "fs.azure.account.oauth2.client.id": "<Service Principal Application ID>",
       "fs.azure.account.oauth2.client.secret": "<Service Principal Application Secret (or password)>",
       "fs.azure.account.oauth2.client.endpoint": "https://login.microsoftonline.com/<Service Principal Tenant ID>/oauth2/token",
       "fs.azure.createRemoteFileSystemDuringInitialization": "true"}
```

> この一連のコマンドは、次の 2 つのことを実行します。
>
> - 以下の 2 つのパラメーターを指定して、`mountFileSystem` 関数を宣言する。
>     - `containerName`: ADLS Gen2 ストレージ上のファイル システムの名前
>     - `storageAccountName`: ADLS Gen2 ストレージ アカウントの名前
> - クラスターで実行中のノートブックを介して ADLS Gen2 ストレージにアクセスするためのパラメーターを定義する `configs` 変数を定義する。 この構成の詳細については、[このリンク](https://docs.microsoft.com/ja-jp/azure/storage/blobs/data-lake-storage-use-databricks-spark#create-a-file-system-and-mount-it)をご覧ください。
>     - 次のプレースホルダーに注目してください。
>         - `<Service Principal Application ID>`
>         - `<Service Principal Application Secret (or password)>`
>         - `<Service Principal Tenant ID>`
>
> 前に作成したサービス プリンシパルの情報を使用して置き換えます。

```python
mountPoint = "/mnt/adls/" + containerName
  
  try:
    dbutils.fs.mount(
      source = "abfss://" + containerName + "@" + storageAccountName + ".dfs.core.windows.net",
      mount_point = mountPoint,
      extra_configs = configs
    )
    print(mountPoint + " mounted successfully")
  except:
    print("The mount " + mountPoint + " already exists.")
```

> - `mountPoint` 変数が設定されます。これは、
>     - ファイル システムのマウント (`dbutils.fs.mount`) 中に使用され、
>     - この関数が呼び出されるたびに返されるようにするためです。こうして、ファイル システムをマウント後に参照しやすくします。
> - このコードが初回に実行されない場合に、例外処理が役に立ちます。 ファイル システムを再度マウントする代わりに、マウントが既に存在することを通知するメッセージを書き込みます。

```python
return mountPoint
```

> ファイル システムが既にマウントされているかどうかにかかわらず、この関数はファイル システムのパスを返します。そのため、コンシューマー コードで、使用するファイルを簡単に見つけることができます。

### <a name="defining-the-saveasparquet-function"></a>saveAsParquet 関数を定義する

```python
def saveAsParquet(dataFrame, filePath):
  df = sqlContext.createDataFrame(dataFrame)
  
  df.write.parquet(filePath, mode='overwrite')
  
  print(filePath + " saved successfully")
```

> 上記のコードでは、以下のパラメーターを受け取る `saveAsParquet` 関数を宣言しています。
>
> - `dataFrame`: Parquet として保存される Pandas DataFrame
> - `filePath`: 保存されるファイルのパス

```python
df = sqlContext.createDataFrame(dataFrame)
```

> Pandas DataFrame によってそれ自体を Parquet ファイルとして書き込むことはできず、PySpark DataFrame によって行います。 そのため、以下のコマンドを実行して、Pandas DataFrame から PySpark Dataframe に変換します。

```python
df.write.parquet(filePath, mode='overwrite')
  
print(filePath + " saved successfully")
```

> 上記の 2 行のコードでは、PySpark データ フレームを、指定されたパスの Parquet ファイルに書き込んでから、成功メッセージを書き込みます。

### <a name="calling-the-mountfilesystem-function"></a>mountFileSystem 関数を呼び出す

ノートブックでいずれかの処理を行う前に、ファイル システムをマウントする必要があります。 そのため、最初のノートブック セルでは、`mountFileSystem` 関数が、それ自体と `saveAsParquet` 関数の宣言の直後に呼び出されます。

```python
...
mountPoint = mountFileSystem("southridge", "<storage account name>")
print(mountPoint)
```

この例で、`southridge` は、Data Factory を介してすべてのデータの取り込みが行われたファイル システム Data Lake Storage Gen2 コンテナーです。