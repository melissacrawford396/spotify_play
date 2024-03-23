## Play with Spotify data

Use data someone scraped and uploaded to Kaggle: https://www.kaggle.com/datasets/maharshipandya/-spotify-tracks-dataset


#### Data
Columns
```
['track_id', 'artists', 'album_name', 'track_name', 'popularity',
       'duration_ms', 'explicit', 'danceability', 'energy', 'key', 'loudness',
       'mode', 'speechiness', 'acousticness', 'instrumentalness', 'liveness',
       'valence', 'tempo', 'time_signature', 'track_genre']
```

### Todo

#### SQL / MySQL / Docker
- Setup MySql locally in a docker image
- connect to the mysql
- upload the csv data into the db
- Query with SQL
  - partition over 
  - Using
  - multiple aggregates

#### PySpark / Scio / Probabilistic data 
- Run local aggregations
- Run
  - Bloom filter
  - Hyper Log Log
- Joins
  - Sparkey
  - large hash join
  - joins (generic how long to do?)


