---
ms.openlocfilehash: d1cccfaf74aaed2bf6ade4aa4a8a8475b8210568
ms.sourcegitcommit: 3ff1164f4921cd2b423d97e7ff29cf3184026d57
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/22/2022
ms.locfileid: "140765960"
---
# <a name="table-definitions"></a>テーブル定義

- [DimActors](#DimActors)
- [DimCategories](#DimCategories)
- [DimCustomers](#DimCustomers)
- [DimDate](#DimDate)
- [DimLocations](#DimLocations)
- [DimMovieActors](#DimMovieActors)
- [DimMovies](#DimMovies)
- [DimRatings](#DimRatings)
- [DimTime](#DimTime)
- [FactRentals](#FactRentals)
- [FactSales](#FactSales)
- [FactStreaming](#FactStreaming)

## <a name="dimactors"></a>DimActors

列名 | データ型 | Null 値 | ルール
---|---|---|---
ActorSK | 32 ビット整数 | いいえ | シーケンシャル整数キー
ActorID | GUID | いいえ | ソース システムからの一意識別子
ActorName | String | いいえ | アクターの名前 (最大 81 文字)
ActorGender | String | いいえ | アクターの性別の 1 文字表記

### <a name="data-generationpopulation-rules"></a>データ生成/作成規則

- データは、すべてのソース システムからのアクター リストを組み合わせたものである必要があります
- ウェアハウスに読み込まれる前に、重複を削除し、考慮する必要があります
- データベース プラットフォームで参照整合性とキーがサポートされている場合は、次の規則を使用できます
    - 主キー - `ActorSK`
    - 一意なインデックス - `ActorID`

[ページのトップへ](#Table-Definitions)

## <a name="dimcategories"></a>DimCategories

列名 | データ型 | Null 値 | ルール
---|---|---|---
MovieCategorySK | 8 ビット整数 | いいえ | シーケンシャル整数キー
MovieCategoryDescription | String | いいえ | 映画のカテゴリの説明

### <a name="data-generationpopulation-rules"></a>データ生成/作成規則

- データは、すべてのソース システムからの映画のカテゴリを組み合わせたものである必要があります
- ウェアハウスに読み込まれる前に、重複を削除し、考慮する必要があります
- データベース プラットフォームで参照整合性とキーがサポートされている場合は、次の規則を使用できます
    - 主キー - `MovieCategorySK`
    - 一意なインデックス - `MovieCategoryDescription`

[ページのトップへ](#Table-Definitions)

## <a name="dimcustomers"></a>DimCustomers

列名 | データ型 | Null 値 | ルール
---|---|---|---
CustomerSK | 32 ビット整数 | いいえ | シーケンシャル整数キー
CustomerID | GUID | いいえ | ソース システムからの一意識別子
LastName | String | いいえ | 顧客の姓 (最大 50 文字)
FirstName | String | いいえ | 顧客の名 (最大 50 文字)
AddressLine1 | String | いいえ | 顧客の住所 - 1 行目 (最大 50 文字)
AddressLine2 | String | はい | 顧客の住所 - 2 行目 (最大 50 文字)
City | String | いいえ | 顧客の市区町村 (最大 30 文字)
State | String | いいえ | 顧客の都道府県コード (2 文字)
郵便番号 | String | いいえ | 顧客の郵便番号 (5 桁)
PhoneNumber | String | いいえ | 顧客の電話番号 (10 桁)
RecordStartDate | Date | いいえ | このバージョンの顧客レコードがアクティブになった日付
RecordEndDate | Date | はい | このバージョンの顧客レコードが非アクティブになった日付
ActiveFlag | ブール型 | いいえ | 特定の顧客のアクティブなレコードに対して true に設定されたフラグ

### <a name="data-generationpopulation-rules"></a>データ生成/作成規則

- これはタイプ 2 ディメンションです。レコードを変更すると、前のレコードが非アクティブになります
- データは、すべてのソース システムからの顧客レコードを組み合わせたものである必要があります
- ウェアハウスに読み込まれる前に、重複を削除し、考慮する必要があります
- データベース プラットフォームで参照整合性とキーがサポートされている場合は、次の規則を使用できます
    - 主キー - `CustomerSK`
    - 一意なインデックス - `CustomerID`

[ページのトップへ](#Table-Definitions)

## <a name="dimdate"></a>DimDate

列名 | データ型 | Null 値 | ルール
---|---|---|---
DateSK | 32 ビット整数 | いいえ | シーケンシャル整数キー
DateValue | Date | いいえ | 日付値 (時刻なし)
DateYear | 16 ビット整数 | いいえ | DateValue の年の部分
DateMonth | 8 ビット整数 | いいえ | DateValue の月の部分
DateDay | 8 ビット整数 | いいえ | DateValue の日の部分
DateDayOfWeek | 8 ビット整数 | いいえ | DateValue の曜日 (日曜日 = 1)
DateDayOfYear | 16 ビット整数 | いいえ | DateValue の年の通算日
DateWeekOfYear | 8 ビット整数 | いいえ | DateValue の年の通算週

### <a name="data-generationpopulation-rules"></a>データ生成/作成規則

- 最初の日付レコードは、`2017-01-01` のものである必要があります
- 初期 (一括) データ読み込みでは、読み込む日付ごとに 1 つの `DimDate` レコードを含めます
- 毎日の (増分) データ読み込みでは、読み込む日の `DimDate` レコードを追加します
- データベース プラットフォームで参照整合性とキーがサポートされている場合は、次の規則を使用できます
    - 主キー - `DateSK`
    - 一意なインデックス - `DateValue`

[ページのトップへ](#Table-Definitions)

## <a name="dimlocations"></a>DimLocations

列名 | データ型 | Null 値 | ルール
---|---|---|---
LocationSK | 16 ビット整数 | いいえ | シーケンシャル整数キー
LocationName | String | いいえ | 場所の名前 (最大 50 文字)
ストリーム | ブール型 | いいえ | Location でストリーミングがサポートされている場合は true
レンタル数 | ブール型 | いいえ | Location でレンタルがサポートされている場合は true
Sales | ブール型 | いいえ | Location で販売がサポートされている場合は true

### <a name="data-generationpopulation-rules"></a>データ生成/作成規則

- ソース データを提供する場所ごとに 1 つのレコードを作成します
- このテーブルには、最初にデータ ソースの場所ごとに 1 つのレコードを読み込む必要があります
    - 場所が追加されたら、その新しい場所のレコードをこのテーブルに追加します
- データベース プラットフォームで参照整合性とキーがサポートされている場合は、次の規則を使用できます
    - 主キー - `LocationSK`
    - 一意なインデックス - `LocationName`

[ページのトップへ](#Table-Definitions)

## <a name="dimmovieactors"></a>DimMovieActors

列名 | データ型 | Null 値 | ルール
---|---|---|---
MovieID | GUID | いいえ | 映画の ID
ActorID | GUID | いいえ | アクターの ID

### <a name="data-generationpopulation-rules"></a>データ生成/作成規則

- これは、映画とアクターの多対多マッピング テーブルです
- データは、すべてのソース システムからの映画の年齢区分を組み合わせたものである必要があります
- ウェアハウスに読み込まれる前に、重複を削除し、考慮する必要があります
- データベース プラットフォームで参照整合性とキーがサポートされている場合は、次の規則を使用できます
    - 主キー - `MovieID`、`ActorID`

[ページのトップへ](#Table-Definitions)

## <a name="dimmovies"></a>DimMovies

列名 | データ型 | Null 値 | ルール
---|---|---|---
MovieSK | 32 ビット整数 | いいえ | シーケンシャル整数キー
MovieID | GUID | いいえ | ソース システムからの一意識別子
MovieTitle | String | いいえ | 映画のタイトル (最大 255 文字)
MovieCategorySK | 8 ビット整数 | いいえ | (DimCategories テーブルの) 映画のカテゴリを指すキー
MovieRatingSK | 8 ビット整数 | いいえ | (DimRatings テーブルの) 映画の年齢区分を指すキー
MovieRunTimeMin | 16 ビット整数 | いいえ | 映画の実行時間 (分単位)

### <a name="data-generationpopulation-rules"></a>データ生成/作成規則

- データは、すべてのソース システムからの映画リストを組み合わせたものである必要があります
- ウェアハウスに読み込まれる前に、重複を削除し、考慮する必要があります
- データベース プラットフォームで参照整合性とキーがサポートされている場合は、次の規則を使用できます
    - 主キー - `MovieSK`
    - 一意なインデックス - `MovieID`
    - 外部キー - `MovieCategorySK` (`DimCategories` テーブルへの外部キー)
    - 外部キー - `MovieRatingSK` (`DimRatings` テーブルへの外部キー)

[ページのトップへ](#Table-Definitions)

## <a name="dimratings"></a>DimRatings

列名 | データ型 | Null 値 | ルール
---|---|---|---
MovieRatingSK | 8 ビット整数 | いいえ | シーケンシャル整数キー
MovieRatingDescription | String | いいえ | 映画の年齢区分の説明 (最大 5 文字)

### <a name="data-generationpopulation-rules"></a>データ生成/作成規則

- データは、すべてのソース システムからの映画の年齢区分を組み合わせたものである必要があります
- ウェアハウスに読み込まれる前に、重複を削除し、考慮する必要があります
- データベース プラットフォームで参照整合性とキーがサポートされている場合は、次の規則を使用できます
    - 主キー - `MovieRatingSK`
    - 一意なインデックス - `MovieRatingDescription`

[ページのトップへ](#Table-Definitions)

## <a name="dimtime"></a>DimTime

列名 | データ型 | Null 値 | ルール
---|---|---|---
TimeSK | 32 ビット整数 | いいえ | シーケンシャル整数キー
TImeValue | Time | いいえ | 時刻値 (日付なし、1 秒の粒度)
TimeHour | 8 ビット整数 | いいえ | TimeValue の時間の部分
TimeMinute | 8 ビット整数 | いいえ | TimeValue の分の部分
TimeSecond | 8 ビット整数 | いいえ | TimeValue の秒の部分
TimeMinuteOfDay | 16 ビット整数 | いいえ | 日の通算分 (午前 0 時 = 0)
TimeSecondOfDay | 32 ビット整数 | いいえ | 日の通算秒 (午前 0 時 = 0)

### <a name="data-generationpopulation-rules"></a>データ生成/作成規則

- 1 日の 1 秒あたり 1 レコードを作成します (合計 86,400 レコード)
- これは 1 回限りの読み込みテーブルであり、データを追加したり変更したりすることはできません
- データベース プラットフォームで参照整合性とキーがサポートされている場合は、次の規則を使用できます
    - 主キー - `TimeSK`
    - 一意なインデックス - `TimeValue`

[ページのトップへ](#Table-Definitions)

## <a name="factrentals"></a>FactRentals

列名 | データ型 | Null 値 | ルール
---|---|---|---
RentalSK | 32 ビット整数 | いいえ | シーケンシャル整数キー
TransactionID | GUID | いいえ | ソース システムからのトランザクション ID
CustomerSK | 32 ビット整数 | いいえ | 顧客に対応する、Customers ディメンションの代理キー
LocationSK | 16 ビット整数 | いいえ | ソース データの場所に対応する、Locations ディメンションの代理キー
MovieSK | 32 ビット整数 | いいえ | 映画に対応する、Movies ディメンションの代理キー
RentalDateSK | 32 ビット整数 | いいえ | 映画がレンタルされた日付に対応する、Date ディメンションの代理キー
ReturnDateSK | 32 ビット整数 | はい | 映画が返却された日付に対応する、Date ディメンションの代理キー
RentalDuration | 8 ビット整数 | はい | 映画がレンタルされてから返却されるまでの日数
RentalCost | Currency | いいえ | 基本レンタル料金
LateFee | Currency | はい | レンタルの延滞料金
TotalCost | Currency | はい | RentalCost + LateFee
RewindFlag | Boolean | はい | 映画は巻き戻されているか

### <a name="data-generationpopulation-rules"></a>データ生成/作成規則

- これは、物理的な映画レンタルを表すファクト テーブルです
- データは、映画レンタルを扱うすべてのソース システムからの顧客レコードを組み合わせたものである必要があります
- データベース プラットフォームで参照整合性とキーがサポートされている場合は、次の規則を使用できます
    - 主キー - `RentalSK`
    - 一意なインデックス - `TransactionID`
    - 外部キー - `RentalDateSK`、参照 `DimDate` (`DateSK`)
    - 外部キー - `ReturnDateSK`、参照 `DimDate` (`DateSK`)
    - 外部キー - `CustomerSK`、参照 `DimCustomers` (`CustomerSK`)
    - 外部キー - `LocationSK`、参照 `DimLocations` (`LocationSK`) -外部キー - `MovieSK`、参照 `DimMovies` (`MovieSK`)

[ページのトップへ](#Table-Definitions)

## <a name="factsales"></a>FactSales

列名 | データ型 | Null 値 | ルール
---|---|---|---
SalesSK | 32 ビット整数 | いいえ | シーケンシャル整数キー
OrderID | GUID | いいえ | ソース システムからの注文 ID
LineNumber | 8 ビット整数 | いいえ | ソース システムからの注文明細行番号
OrderDateSK | 32 ビット整数 | いいえ | 注文が行われた日付に対応する、Date ディメンションの代理キー
ShipDateSK | 32 ビット整数 | はい | 注文が発送された日付に対応する、Date ディメンションの代理キー
CustomerSK | 32 ビット整数 | いいえ | 顧客に対応する、Customers ディメンションの代理キー
LocationSK | 16 ビット整数 | いいえ | ソース データの場所に対応する、Locations ディメンションの代理キー
MovieSK | 32 ビット整数 | いいえ | 映画に対応する、Movies ディメンションの代理キー
DaysToShip | 8 ビット整数 | はい | 受注から発送までの日数
Quantity | 8 ビット整数 | いいえ | この品目の購入数量
UnitCost | Currency | いいえ | この品目の 1 個あたりの購入価格
ExtendedCost | Currency | いいえ | Quantity * UnitCost

### <a name="data-generationpopulation-rules"></a>データ生成/作成規則

- これは、映画の物理的な販売を表すファクト テーブルです
- データは、映画の販売を扱うすべてのソース システムからの顧客レコードを組み合わせたものである必要があります
- データベース プラットフォームで参照整合性とキーがサポートされている場合は、次の規則を使用できます
    - 主キー - `SalesSK`
    - 一意なインデックス - `OrderID` & `LineNumber`
    - 外部キー - `OrderDateSK`、参照 `DimDate` (`DateSK`)
    - 外部キー - `ShipDateSK`、参照 `DimDate` (`DateSK`)
    - 外部キー - `CustomerSK`、参照 `DimCustomers` (`CustomerSK`)
    - 外部キー - `LocationSK`、参照 `DimLocations` (`LocationSK`) -外部キー - `MovieSK`、参照 `DimMovies` (`MovieSK`)

[ページのトップへ](#Table-Definitions)

## <a name="factstreaming"></a>FactStreaming

列名 | データ型 | Null 値 | ルール
---|---|---|---
StreamingSK | 32 ビット整数 | いいえ | シーケンシャル整数キー
TransactionID | GUID | いいえ | ソース システムからのトランザクション ID
CustomerSK | 32 ビット整数 | いいえ | 顧客に対応する、Customers ディメンションの代理キー
MovieSK | 32 ビット整数 | いいえ | 映画に対応する、Movies ディメンションの代理キー
StreamStartDateSK | 32 ビット整数 | いいえ | ストリーミングが開始された日付に対応する、Date ディメンションの代理キー
StreamStartTimeSK | 32 ビット整数 | いいえ | ストリーミングが開始された時刻に対応する、Time ディメンションの代理キー
StreamEndDateSK | 32 ビット整数 | はい | ストリーミングが終了した日付に対応する、Date ディメンションの代理キー
StreamEndTimeSK | 32 ビット整数 | はい | ストリーミングが終了した時刻に対応する、Time ディメンションの代理キー
StreamDurationSec | 32 ビット整数 | はい | ストリーミング セッションの継続時間 (秒単位)
StreamDurationMin | Decimal | はい | ストリーミング セッションの継続時間 (分 + 小数の分単位) (小数点以下 4 桁)

### <a name="data-generationpopulation-rules"></a>データ生成/作成規則

- これは、映画のストリーミング セッションを表すファクト テーブルです
- データは、映画のストリーミングを扱うすべてのソース システムからの顧客レコードを組み合わせたものである必要があります
- データベース プラットフォームで参照整合性とキーがサポートされている場合は、次の規則を使用できます
    - 主キー - `StreamingSK`
    - 一意なインデックス - `TransactionID`
    - 外部キー - `StreamStartDateSK`、参照 `DimDate` (`DateSK`)
    - 外部キー - `StreamStartTimeSK`、参照 `DimTime` (`TimeSK`)
    - 外部キー - `StreamEndDateSK`、参照 `DimDate` (`DateSK`)
    - 外部キー - `StreamEndTimeSK`、参照 `DimTime` (`TimeSK`)
    - 外部キー - `CustomerSK`、参照 `DimCustomers` (`CustomerSK`) -外部キー - `MovieSK`、参照 `DimMovies` (`MovieSK`)

[ページのトップへ](#Table-Definitions)
