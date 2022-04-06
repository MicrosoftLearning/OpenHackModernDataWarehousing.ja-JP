---
ms.openlocfilehash: 62c5c7635ac90f4ffa7cec309b18a0175795a443
ms.sourcegitcommit: 3ff1164f4921cd2b423d97e7ff29cf3184026d57
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/22/2022
ms.locfileid: "140765896"
---
# <a name="21-loading-southridge-movies-and-actors"></a>2.1 Southridge の映画とアクターを読み込む

次のコードでは、Southridge の映画とアクターを読み込みます。

```python
sr_movies_filepath = mountPoint + "/raw/moviescatalog/movies.json"

sr_movies_raw = spark.read.option("multiLine", "true").options(header='true', inferschema='true').json(sr_movies_filepath)
sr_movies_pd = sr_movies_raw.toPandas()

sr_movies_pd = sr_movies_pd[['actors', 'availabilityDate', 'genre', 'id', 'rating', 'releaseYear', 'runtime', 'streamingAvailabilityDate', 'tier', 'title']]

movieactors = sr_movies_pd[['id', 'actors']]
movies = sr_movies_pd[['id', 'title', 'genre', 'availabilityDate', 'rating', 'releaseYear', 'runtime', 'streamingAvailabilityDate', 'tier']]

import numpy as np

actorslist = movieactors.actors.values.tolist()
actorcountbymovie = [len(r) for r in actorslist]
explodedmovieids = np.repeat(movieactors.id, actorcountbymovie)

movieactors = pd.DataFrame(np.column_stack((explodedmovieids, np.concatenate(actorslist))), columns=movieactors.columns)

sr_all_moviesactorsactors = pd.merge(movies, movieactors, on='id')

sr_all_moviesactorsactors = sr_all_moviesactorsactors.rename(index=str, columns={'id': 'MovieID', 'title': 'MovieTitle', 'genre': 'Genre', 'availabilityDate': 'AvailabilityDate', 'rating': 'Rating', 'releaseYear': 'ReleaseYear', 'runtime': 'RuntimeMin', 'streamingAvailabilityDate': 'StreamingAvailabilityDate', 'tier': 'Tier', 'actors': 'ActorName'})

sr_all_moviesactorsactors['ActorID'] = 'None'
sr_all_moviesactorsactors['MovieActorID'] = 'None'
sr_all_moviesactorsactors['ActorGender'] = 'None'
sr_all_moviesactorsactors['ReleaseDate'] = 'None'

sr_all_moviesactorsactors.ReleaseYear = sr_all_moviesactorsactors.ReleaseYear.astype(str)
sr_all_moviesactorsactors.Tier = sr_all_moviesactorsactors.Tier.astype(str)
sr_all_moviesactorsactors.RuntimeMin = sr_all_moviesactorsactors.RuntimeMin.astype(str)

sr_all_moviesactorsactors = sr_all_moviesactorsactors[['MovieID', 'MovieTitle', 'Genre', 'ReleaseDate', 'AvailabilityDate', 'StreamingAvailabilityDate', 'ReleaseYear', 'Tier', 'Rating', 'RuntimeMin', 'MovieActorID', 'ActorID', 'ActorName', 'ActorGender']]
```

## <a name="detailing-the-code-above"></a>上のコードの詳細な説明

このコードに関する詳細情報を次に示します。

### <a name="loading-the-data"></a>データを読み込む

```python
sr_movies_filepath = mountPoint + "/raw/movies/movies.json"

sr_movies_raw = spark.read.option("multiLine", "true").options(header='true', inferschema='true').json(sr_movies_filepath)
```

> Southridge の映画カタログは、以前に CosmosDB コレクションから読み込まれたため、JSON ファイルとしてデータ レイクに格納されています。 そのため、データを PySpark データフレームとして読み込むには、`spark.read.json` を使用します。

```python
sr_movies_pd = sr_movies_raw.toPandas()
```

> 変換しやすくするために、PySpark データフレームを Pandas データフレームに変換します。

```python
sr_movies_pd = sr_movies_pd[['actors', 'availabilityDate', 'genre', 'id', 'rating', 'releaseYear', 'runtime', 'streamingAvailabilityDate', 'tier', 'title']]
```

> JSON ファイルを読み込んだデータフレームには、JSON 固有の列もいくつか含まれているため、このスコープに関連する列のみを選択する必要があります。

```python
movieactors = sr_movies_pd[['id', 'actors']]
movies = sr_movies_pd[['id', 'title', 'genre', 'availabilityDate', 'rating', 'releaseYear', 'runtime', 'streamingAvailabilityDate', 'tier']]
```

> アクターは、各映画の配列として格納されているため、最初にこれらの 2 つの情報を分離する必要があります。これにより、アクターを *展開* し、その後再び結合することができます。 `movieactors` にはアクターの情報のみが含まれ、`movies` には映画のデータのみが含まれます。

### <a name="denormalizing-the-movies-and-actors-relationship"></a>映画とアクターの関係を非正規化する

```python
import numpy as np

actorslist = movieactors.actors.values.tolist()
actorcountbymovie = [len(r) for r in actorslist]
explodedactorids = np.repeat(movieactors.id, actorcountbymovie)
```

> コードのこの部分は、このスコープで重要です。 短期的には、各映画のアクター配列を別個の一時的な `movieactors` テーブルに展開して、後で映画のデータと結合できるようにします。
>
> しかし、まず第一に、`numpy` を使用してアクター データフレームを抽出するために、`import numpuy as np` を使用してこれをインポートする必要があります。

```python
actorslist = movieactors.actors.values.tolist()
```

> 最初に、`movieactors` データフレームからアクターの名前の未加工のリストを取得します。

```python
actorcountbymovie = [len(r) for r in actorslist]
```

> 次に、各映画に含まれるアクターの数を特定します。

```python
explodedmovieids = np.repeat(movieactors.id, actorcountbymovie)
```

> 最後に、映画 ID のリストを作成します。ここでは、映画に含まれるすべてのアクターについて各項目が繰り返されます。 たとえば、映画 *映画 1* に 2 人のアクター *アクター A*、*アクター B* があり、*映画 2* に 3 人のアクター *アクター A*、*アクター C*、および *アクター D* がある場合、目的のデータフレームは次のようになります。

|MovieID|ActorID|MovieName|ActorName|
|-------|-------|---------|---------|
|1|A|映画 1|アクター A|
|1|B|映画 1|アクター B|
|2|A|映画 2|アクター A|
|2|C|映画 2|アクター C|
|2|D|映画 2|アクター D|

```python
movieactors = pd.DataFrame(np.column_stack((explodedmovieids, np.concatenate(actorslist))), columns=movieactors.columns)
```

> このコードでは、すべての映画 ID (`explodedmovieids`) とそのアクター名 (`np.concatenate(actorslist)`) を使用して、新しい Pandas データフレームが作成されます。

### <a name="bringing-movies-and-actors-together"></a>映画とアクターをまとめる

```python
sr_all_moviesactors = pd.merge(movies, movieactors, on='id')
```

最後に、映画とアクターが映画 `id` によって結合されます

```python
sr_all_moviesactors = sr_all_moviesactors.rename(index=str, columns={'id': 'MovieID', 'title': 'MovieTitle', 'genre': 'Genre', 'availabilityDate': 'AvailabilityDate', 'rating': 'Rating', 'releaseYear': 'ReleaseYear', 'runtime': 'RuntimeMin', 'streamingAvailabilityDate': 'StreamingAvailabilityDate', 'tier': 'Tier', 'actors': 'ActorName'})
```

> 非正規化された映画およびアクターを取得した後、上記のコードでは、すべての映画およびアクターのデータフレームで使用される特定のパターンに一致するように列の名前が変更されます。

### <a name="conforming-the-data-types"></a>データ型を一致させる

```python
sr_all_moviesactors['ActorID'] = 'None'
sr_all_moviesactors['MovieActorID'] = 'None'
sr_all_moviesactors['ActorGender'] = 'None'
```

> 他のデータ フレームではこれらのフィールドに値が含まれている可能性があるため、他のデータ フレームから削除されないように、それらを `None` としてこのデータ フレームに追加します。

```python
sr_all_moviesactors.ReleaseYear = sr_all_moviesactors.ReleaseYear.astype(str)
sr_all_moviesactors.Tier = sr_all_moviesactors.Tier.astype(str)
sr_all_moviesactors.RuntimeMin = sr_all_moviesactors.RuntimeMin.astype(str)
```

> これらの既存の列は、他のデータ フレームではデータ型が異なる可能性があります。 これを処理するために、すべてのデータ フレームで、それらすべてに文字列を使用します。

```python
sr_all_moviesactors = sr_all_moviesactors[['MovieID', 'MovieTitle', 'Genre', 'AvailabilityDate', 'StreamingAvailabilityDate', 'ReleaseYear', 'Tier', 'Rating', 'RuntimeMin', 'MovieActorID', 'ActorID', 'ActorName', 'ActorGender']]
```

> すべてのデータが適切に変換されたら、関連する (かつ、すべてのデータ フレームにわたって一貫性のある) 列を選択し、表示しやすい順序で、それを `sr_all_moviesactors` に設定します。