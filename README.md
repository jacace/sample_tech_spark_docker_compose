export MYNAME=spark-jacace

chmod +x start-worker.sh
docker build -t $MYNAME/spark:latest .


#####MASTER start
docker run --rm -it --name spark-master --hostname spark-master  -p 7077:7077 -p 8080:8080 $MYNAME/spark:latest /bin/sh
#inside the container
/spark/bin/spark-class org.apache.spark.deploy.master.Master --ip `hostname` --port 7077 --webui-port 8080

docker network create spark_network

#now run with a network
docker run --rm -it --name spark-master --hostname spark-master  -p 7077:7077 -p 8080:8080 --network spark_network  $MYNAME/spark:latest /bin/sh 


#####WORKER start
docker run --rm -it --name spark-worker --hostname spark-worker  --network spark_network  $MYNAME/spark:latest /bin/sh
#inside teh container
/spark/bin/spark-class org.apache.spark.deploy.worker.Worker  --webui-port 8080 spark://spark-master:7077


##with docker compose
docker-compose up
