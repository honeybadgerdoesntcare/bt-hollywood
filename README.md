# bt-hollywood
Movie Rating with Collaborative Filtering

## Environment
_Customized cloudera/quickstart image: upgrade python to 2.7.11_
The customized image can be downloaded by 
```shell
docker pull zhou42/movierating:v1
```
## Files
_DataDownload.py_ download data from movielens website and store the data at the local file system
_MovieRating.py_ load the file in the HDFS file system and build the latent factor CF model
_recommenderSystem.py_ make recommendations to the new user
## Steps to run the program
1. First step
```shell
spark-submit --driver-memory 4g DataDownload.py 
```
The movielens data will be downloaded to the local file system (in a new folder "./datasets")
2. Second step
```shell
hadoop fs -put datasets
hadoop fs -mkdir models
hadoop fs -mkdir checkpoint
```
Put the local folder "./datasets" into the HDFS; make a new folder in HDFS to store the final model trained; checkpoint is used to avoid stackover flow
3. Third step
```shell
spark-submit --driver-memory 4g MovieRating.py
```
_Note the size of driver memory can be further increased depending on the OS memory size_
Train the model; choose the parameters according the results on validation set; 
result is as below
![result](./trainingresult.png "result")
