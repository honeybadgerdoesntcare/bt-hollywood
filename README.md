# bt-hollywood
Movie Rating with Collaborative Filtering.   Featuring two ways of implementation -
`Python + PySpark'  &  'Scala + Spark`

## Python + PySpark implementation
### Environment
_Customized cloudera/quickstart image: upgrade python to 2.7.11_
The customized image can be downloaded by
```shell
docker pull zhou42/movierating:v1
```
### Files
_DataDownload.py_ download data from movielens website and store the data at the local file system
_MovieRating.py_ load the file in the HDFS file system and build the latent factor CF model
_recommenderSystem.py_ make recommendations to the new user
### Steps to run the program
- First step
```shell
spark-submit --driver-memory 4g DataDownload.py
```
The movielens data will be downloaded to the local file system (in a new folder "./datasets")

- Second step
```shell
hadoop fs -put datasets
hadoop fs -mkdir models
hadoop fs -mkdir checkpoint
```
Put the local folder "./datasets" into the HDFS; make a new folder in HDFS to store the final model trained; checkpoint is used to avoid stackover flow

- Third step
```shell
spark-submit --driver-memory 4g MovieRating.py
```
_Note the size of driver memory can be further increased depending on the OS memory size_
Train the model; choose the parameters according the results on validation set;
result is as below
![result](./trainingresult.png "result")



## Scala + Spark implementation
_MovieRater.scala_ trains mode, evaluates and selects best model based on user ratings.

### Prerequisitives:

  * Cloudera Quickstart 5.7 docker
  * scala 2.10
  * maven
  * wget

### Run
- First step
```shell
wget http://files.grouplens.org/datasets/movielens/ml-latest-small.zip
unzip ml-latest-small.zip
wget http://files.grouplens.org/datasets/movielens/ml-latest.zip
unzip ml-latest.zip
```
The movielens data will be downloaded to the current dir and unzipped into separate folders

- Second step
```shell
hadoop fs -mkdir -p /tmp/mydata/movielens/small/
hadoop fs -copyFromLocal  ml-latest-small/ratings.csv /tmp/mydata/movielens/small/
hadoop fs -mkdir -p /tmp/mydata/movielens/big/
hadoop fs -copyFromLocal  ml-latest/ratings.csv /tmp/mydata/movielens/big/
```
Make new folders in HDFS and copy ratings.csv over

- Third step
  - run on small dataset :
```shell
spark-submit --master yarn-client --class io.bittiger.movierating.hollywood.MovieRater hollywood-1.0-SNAPSHOT-jar-with-dependencies.jar  /tmp/mydata/movielens/small/ratings.csv
```
  - run on large dataset :
```shell
spark-submit --master yarn-client --class io.bittiger.movierating.hollywood.MovieRater hollywood-1.0-SNAPSHOT-jar-with-dependencies.jar  /tmp/mydata/movielens/big/ratings.csv
```

### Result example
```
Got 22884377 ratings from 247753 users on 33670 movies.
Training: 13729614, validation: 4580902, test: 4573861
RMSE (validation) = 0.8258453994610284 for the model trained with rank = 8, lambda = 0.1, and numIter = 10.
RMSE (validation) = 0.8214951299457715 for the model trained with rank = 8, lambda = 0.1, and numIter = 20.
RMSE (validation) = 3.681021156174323 for the model trained with rank = 8, lambda = 10.0, and numIter = 10.
RMSE (validation) = 3.681021156174323 for the model trained with rank = 8, lambda = 10.0, and numIter = 20.
RMSE (validation) = 0.8205171634436613 for the model trained with rank = 12, lambda = 0.1, and numIter = 10.
RMSE (validation) = 0.818235041472173 for the model trained with rank = 12, lambda = 0.1, and numIter = 20.
RMSE (validation) = 3.681021156174323 for the model trained with rank = 12, lambda = 10.0, and numIter = 10.
RMSE (validation) = 3.681021156174323 for the model trained with rank = 12, lambda = 10.0, and numIter = 20.
The best model was trained with rank = 12 and lambda = 0.1, and numIter = 20, and its RMSE on the test set is 0.8185896948127657.
The best model improves the baseline by 77.77%.
```

### Reference  ###
* [Collaborative Filtering - spark.mllib](http://spark.apache.org/docs/latest/mllib-collaborative-filtering.html)
* [Source code of Databricks ML training on ALS](https://github.com/databricks/spark-training/blob/master/machine-learning/scala/solution/MovieLensALS.scala)
* [Spark summit 2014 Hands-on Exercises](https://databricks-training.s3.amazonaws.com/index.html)



