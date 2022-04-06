---
ms.openlocfilehash: 1ecf0eebe8a77e4b52126e498a973a6a4ce7c2fb
ms.sourcegitcommit: 3ff1164f4921cd2b423d97e7ff29cf3184026d57
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/22/2022
ms.locfileid: "140765973"
---
# <a name="22-loading-vanarsdel-movies-and-actors"></a>2.2 VanArsdel の映画とアクターを読み込む

次のコードでは、VanArsdel の映画とアクターを読み込みます。

```python
va_movies_filepath = mountPoint + "/raw/vanarsdel/Movies.json"
va_actors_filepath = mountPoint + "/raw/vanarsdel/Actors.json"
va_movieactors_filepath = mountPoint + "/raw/vanarsdel/MovieActors.json"

va_movies_raw = spark.read.option("multiLine", "true").options(header='true', inferschema='true').json(va_movies_filepath)
va_actors_raw = spark.read.option("multiLine", "true").options(header='true', inferschema='true').json(va_actors_filepath)
va_movieactors_raw = spark.read.option("multiLine", "true").options(header='true', inferschema='true').json(va_movieactors_filepath)

va_movies_pd = va_movies_raw.toPandas()
va_actors_pd = va_actors_raw.toPandas()
va_movieactors_pd = va_movieactors_raw.toPandas()

va_all_movies = pd.merge(va_movieactors_pd, va_movies_pd, on='MovieID')
va_all_movies = pd.merge(va_all_movies, va_actors_pd, on='ActorID')

va_all_movies = va_all_movies.rename(index=str, columns={'Category': 'Genre', 'RunTimeMin': 'RuntimeMin', 'Gender': 'ActorGender'})

va_all_movies['AvailabilityDate'] = 'None'
va_all_movies['StreamingAvailabilityDate'] = 'None'
va_all_movies['ReleaseYear'] = 'None'
va_all_movies['Tier'] = 'None'

va_all_movies.RuntimeMin = va_all_movies.RuntimeMin.astype(str)

va_all_movies = va_all_movies[['MovieID', 'MovieTitle', 'Genre', 'ReleaseDate', 'AvailabilityDate', 'StreamingAvailabilityDate', 'ReleaseYear', 'Tier', 'Rating', 'RuntimeMin', 'MovieActorID', 'ActorID', 'ActorName', 'ActorGender']]

va_all_movies.dtypes
```

## <a name="detailing-the-code-above"></a>上記のコードの詳細

このコードに関する詳細情報を次に示します。

### <a name="loading-the-data"></a>データを読み込む

```python
va_movies_filepath = mountPoint + "/raw/vanarsdel/Movies.json"
va_actors_filepath = mountPoint + "/raw/vanarsdel/Actors.json"
va_movieactors_filepath = mountPoint + "/raw/vanarsdel/MovieActors.json"

va_movies_raw = spark.read.option("multiLine", "true").options(header='true', inferschema='true').json(va_movies_filepath)
va_actors_raw = spark.read.option("multiLine", "true").options(header='true', inferschema='true').json(va_actors_filepath)
va_movieactors_raw = spark.read.option("multiLine", "true").options(header='true', inferschema='true').json(va_movieactors_filepath)
```

> VanArsdel のデータは、JSON ファイルとしてデータ レイクに保存されています。そのため、データを Pyspark データフレームとして読み込むには、`spark.read.json` を使用します。

```python
va_movies_pd = va_movies_raw.toPandas()
va_actors_pd = va_actors_raw.toPandas()
va_movieactors_pd = va_movieactors_raw.toPandas()
```

> 変換しやすくするために、PySpark データフレームを Pandas データフレームに変換します。

### <a name="denormalizing-the-movies-and-actors-relationship"></a>映画とアクターの関係を非正規化する

```python
va_all_movies = pd.merge(va_movieactors_pd, va_movies_pd, on='MovieID')
va_all_movies = pd.merge(va_all_movies, va_actors_pd, on='ActorID')
```

> すべてのデータが読み込まれたら、`MoviesActors` マップ テーブルに基づいて映画とアクターをマージします。

```python
va_all_moviesactors = va_all_moviesactors.rename(index=str, columns={'Category': 'Genre', 'RunTimeMin': 'RuntimeMin', 'Gender': 'ActorGender'})
```

> 列の名前付けを標準化するために、以前に読み込まれた Southridge データセットに合わせて、これらの列の名前を変更します。

### <a name="conforming-data-types-and-columns"></a>データ型と列を一致させる

```python
va_all_moviesactors['AvailabilityDate'] = 'None'
va_all_moviesactors['StreamingAvailabilityDate'] = 'None'
va_all_moviesactors['ReleaseYear'] = 'None'
va_all_moviesactors['Tier'] = 'None'
```

> 他のデータ フレームではこれらのフィールドに値が含まれている可能性があるため、他のデータ フレームから削除されないように、それらを `None` としてこのデータ フレームに追加します。

```python
va_all_moviesactors.RuntimeMin = va_all_moviesactors.RuntimeMin.astype(str)
```

> 他のデータ フレームでは、この既存の列のデータ型が異なる可能性があります。 それを処理するために、すべてのデータ フレームで、この列には文字列を使用します。

```python
va_all_moviesactors = va_all_moviesactors[['MovieID', 'MovieTitle', 'Genre', 'ReleaseDate', 'AvailabilityDate', 'StreamingAvailabilityDate', 'ReleaseYear', 'Tier', 'Rating', 'RuntimeMin', 'MovieActorID', 'ActorID', 'ActorName', 'ActorGender']]'ActorGender']]
```

> すべてのデータが適切に変換されたら、関連する (かつ、すべてのデータ フレーム間で一貫性のある) 列を選択し、表示しやすい順序で、それを `va_all_moviesactors` に設定します。