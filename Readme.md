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


### MySQL / Docker
https://www.datacamp.com/tutorial/set-up-and-configure-mysql-in-docker


mysql_password


```

docker pull mysql:latest

docker run --name test-mysql -e MYSQL_ROOT_PASSWORD=strong_password -d mysql

docker run -d --name test-mysql -e MYSQL_ROOT_PASSWORD=strong_password -p 3307:3306 mysql
```

- `-p 3307:3306` Container port 3306 (default MySQL)--> map to local port 3307
```bash
# test port mapping
docker port test-mysql
#  > 3306/tcp -> 0.0.0.0:3307
```

Change configurations locally, so I can change those, but keep them locally
```bash
mkdir conf/
touch conf/my.cnf

# Run docker mounting the subdir conf/ to connect to where the confs
# are stored in mysql
docker run \
   --name test-mysql \
   -v /Users/melissacrawford/Documents/github/spotify-play/conf:/etc/mysql/conf.d \
   -e MYSQL_ROOT_PASSWORD=pass \
   -d mysql

# Bash run the image
docker exec -it test-mysql bash

# Look for my mounted files (in the volume)
ls /etc/mysql/conf.d
```

#### Configuration options
- Performance: innodb_buffer_pool_size, query_cache_size, thread_pool_size
- Security: bind-address, mysql_bind_host, validate_password_policy
- Resource utilization: max_connections, innodb_file_per_table, innodb_io_capacity
- Other common: character_set_server, collation_server, log_bin

```bash
docker run -it --rm mysql --verbose --help
docker run -it --rm mysql:<version> --verbose --help
```

#### Preserve Data stored in MySQL Docker Container
- Create a volume and mount it where the data is stored in our container
  - After we mount the volume, all container data will be linked to it
```bash
docker volume create test-mysql-dta
```

- Restart the container linked to the volume 
  - All volumnes should be mounted in /var/lib/mysql
```bash
docker stop test-mysql; docker rm test-mysql
docker run \
   --name test-mysql \
   -v test-mysql-data:/var/lib/mysql \
   -e MYSQL_ROOT_PASSWORD=strong_password \
   -d mysql
```

#### Clean up and final combo
``` 
docker stop test-mysql; docker rm test-mysql
docker volume rm test-mysql-data
```

- Run
  - password
  - map the port from 3307 (local) --> to mysql port (3306)
  - Mount config volume from local --> conf.d/ map the folders
  - Create a new volumne called final-mysql-data and mount it
```
docker run \
   --name final-mysql \
   -e MYSQL_ROOT_PASSWORD=strong_password \
   -p 3307:3306 \
   -v /etc/docker/test-mysql:/etc/mysql/conf.d \
   -v final-mysql-data:/var/lib/mysql \
   -d mysql
   
# Kill all
docker stop final-mysql; docker rm final-mysql \
docker volume rm final-mysql-data 
```


#### Try to connect and play with it?
```
 docker run \
   --name play-mysql \
   -e MYSQL_ROOT_PASSWORD=strong_password \
   -v /Users/melissacrawford/Documents/github/spotify-play/conf:/etc/mysql/conf.d \
   -v play-mysql-data:/var/lib/mysql \
   -d mysql
   
docker exec -it play-mysql bash

# Once in the bash mysql
mysql -u root -p
```

```bash
# Delete things
docker stop play-mysql ; docker rm play-mysql
```

#### Once in the MySQL CLI
https://dev.mysql.com/doc/mysql-getting-started/en/#mysql-getting-started-basic-ops

```sql
SHOW DATABASES;
CREATE DATABASE pets;
```

- Create a dataset within a database
```sql
USE pets

CREATE TABLE cats
(
  id              INT unsigned NOT NULL AUTO_INCREMENT, # Unique ID for the record
  name            VARCHAR(150) NOT NULL,                # Name of the cat
  owner           VARCHAR(150) NOT NULL,                # Owner of the cat
  birth           DATE NOT NULL,                        # Birthday of the cat
  PRIMARY KEY     (id)                                  # Make the id the primary key
);

SHOW TABLES;

DESCRIBE cats;  

INSERT INTO cats ( name, owner, birth) VALUES
  ( 'Sandy', 'Lennon', '2015-01-03' ),
  ( 'Cookie', 'Casey', '2013-11-13' ),
  ( 'Charlie', 'River', '2016-05-21' );

SELECT * FROM cats;

SELECT name FROM cats WHERE owner = 'Casey';

DELETE FROM cats WHERE name='Cookie';

ALTER TABLE cats ADD gender CHAR(1) AFTER name;

DESCRIBE cats;

SHOW CREATE TABLE cats\G

ALTER TABLE cats DROP gender;

```

#### Try to mount cvs file

```

 docker run \
   --name local-mysql \
   -e MYSQL_ROOT_PASSWORD=pass \
   -v /Users/melissacrawford/Documents/github/spotify-play/conf:/etc/mysql/conf.d \
   -v /Users/melissacrawford/Documents/github/spotify-play/data:/var/lib/mysql-files/data\
   -v /Users/melissacrawford/Documents/github/spotify-play/local:/var/lib/mysql \
   -d mysql
   
docker exec -it local-mysql bash

# Once in the bash mysql
mysql -u root -p


docker stop local-mysql; docker rm local-mysql
docker volume rm local-mysql-data
```

https://blog.skyvia.com/how-to-import-csv-file-into-mysql-table-in-4-different-ways/
```
SHOW DATABASES;
CREATE DATABASE spotify;

USE spotify;
SHOW TABLES;


CREATE TABLE fruit (
  fruit VARCHAR(100),
  size INTEGER,
  cost INTEGER,
  description VARCHAR(100),
  PRIMARY KEY (fruit)
);

SHOW TABLES;

DESCRIBE fruit;
 
 
LOAD DATA INFILE '/var/lib/mysql-files/data/test.csv'
INTO TABLE fruit
FIELDS TERMINATED BY ',' 
IGNORE 1 ROWS;
```

