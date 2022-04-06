---
ms.openlocfilehash: e99184e6d7f456d523ed385dc31e3182487fa498
ms.sourcegitcommit: 3ff1164f4921cd2b423d97e7ff29cf3184026d57
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/22/2022
ms.locfileid: "140765909"
---
# <a name="23-loading-fourthcoffee-movies-and-actors"></a>2.3 FourthCoffee の映画と俳優の読み込み

次のコードでは、FourthCoffee の映画と俳優を読み込みます。

```python
fc_movies_filepath = "/dbfs/" + mountPoint + "/raw/fourthcoffee/Movies.csv"
fc_actors_filepath = "/dbfs/" + mountPoint + "/raw/fourthcoffee/Actors.csv"
fc_movieactors_filepath = "/dbfs/" + mountPoint + "/raw/fourthcoffee/MovieActors.csv"

fc_movies_pd = pd.read_csv(fc_movies_filepath)
fc_actors_pd = pd.read_csv(fc_actors_filepath)
fc_movieactors_pd = pd.read_csv(fc_movieactors_filepath)

fc_all_moviesactors = pd.merge(fc_movieactors_pd, fc_movies_pd, on='MovieID')
fc_all_moviesactors = pd.merge(fc_all_moviesactors, fc_actors_pd, on='ActorID')

fc_all_moviesactors = fc_all_moviesactors.rename(index=str, columns={'Category': 'Genre', 'RunTimeMin': 'RuntimeMin', 'Gender': 'ActorGender'})

fc_all_moviesactors['AvailabilityDate'] = 'None'
fc_all_moviesactors['StreamingAvailabilityDate'] = 'None'
fc_all_moviesactors['ReleaseYear'] = 'None'
fc_all_moviesactors['Tier'] = 'None'

fc_all_moviesactors.RuntimeMin = fc_all_moviesactors.RuntimeMin.astype(str)

fc_all_moviesactors = fc_all_moviesactors[['MovieID', 'MovieTitle', 'Genre', 'ReleaseDate', 'AvailabilityDate', 'StreamingAvailabilityDate', 'ReleaseYear', 'Tier', 'Rating', 'RuntimeMin', 'MovieActorID', 'ActorID', 'ActorName', 'ActorGender']]

display(fc_all_moviesactors)
```

## <a name="detailing-the-code-above"></a>前のコードの詳細な説明

このコードに関する詳細情報を次に示します。

### <a name="loading-the-data"></a>データを読み込む

```python
fc_movies_filepath = mountPoint + "/raw/fourthcoffee/Movies.json"
fc_actors_filepath = mountPoint + "/raw/fourthcoffee/Actors.json"
fc_movieactors_filepath = mountPoint + "/raw/fourthcoffee/MovieActors.json"

fc_movies_pd = pd.read_csv(fc_movies_filepath)
fc_actors_pd = pd.read_csv(fc_actors_filepath)
fc_movieactors_pd = pd.read_csv(fc_movieactors_filepath)
```

> FourthCoffee のデータは、CSV ファイルとしてデータ レイクに格納されます。そのため、データを Pandas データフレームとして読み込むには、`pandas.read_csv` を使用します。
>
> それにより、PySpark データフレームを Pandas データフレームに変換する必要はなくなるため、プロセスが簡単になります。

### <a name="denormalizing-the-movies-and-actors-relationship"></a>映画と俳優の関係の非正規化

```python
fc_all_movies = pd.merge(fc_movieactors_pd, fc_movies_pd, on='MovieID')
fc_all_movies = pd.merge(fc_all_movies, fc_actors_pd, on='ActorID')
```

> すべてのデータが読み込まれたら、`MoviesActors` マップ テーブルに基づいて映画と俳優をマージします。

```python
fc_all_moviesactors = fc_all_moviesactors.rename(index=str, columns={'Category': 'Genre', 'RunTimeMin': 'RuntimeMin', 'Gender': 'ActorGender'})
```

> 列の名前付けを標準化するために、これらの列の名前を、以前に読み込まれた Southridge データセットに一致するように変更します。

### <a name="conforming-data-types-and-columns"></a>データ型と列を一致させる

```python
fc_all_moviesactors['AvailabilityDate'] = 'None'
fc_all_moviesactors['StreamingAvailabilityDate'] = 'None'
fc_all_moviesactors['ReleaseYear'] = 'None'
fc_all_moviesactors['Tier'] = 'None'
```

> 他のデータ フレームではこれらのフィールドに値が含まれている可能性があるため、他のデータ フレームから削除されないように、それらを `None` としてこのデータ フレームに追加します。

```python
fc_all_moviesactors.RuntimeMin = fc_all_moviesactors.RuntimeMin.astype(str)
```

> この既存の列は、他のデータ フレームとの間でデータ型が異なる可能性があります。 それを処理するために、すべてのデータ フレームで、この列には文字列を使用します。

```python
fc_all_moviesactors = fc_all_moviesactors[['MovieID', 'MovieTitle', 'Genre', 'ReleaseDate', 'AvailabilityDate', 'StreamingAvailabilityDate', 'ReleaseYear', 'Tier', 'Rating', 'RuntimeMin', 'MovieActorID', 'ActorID', 'ActorName', 'ActorGender']]'ActorGender']]
```

> すべてのデータが適切に変換されたら、関連する (かつ、すべてのデータ フレームにわたって一貫性のある) 列を選択し、それを表示しやすい順序で `fc_all_moviesactors` に設定します。