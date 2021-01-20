#####Pre-reqs#####
#You need to setup the ENV Var below:
export MYNAME=spark-jacace

#You also need to setup the file permissions below:
chmod +x start-worker.sh

#Then you build the docker image:
docker build -t $MYNAME/spark:latest .

#Next you create a software network:
docker network create spark_network

#Finally you spin-up your cluster with docker compose
docker-compose up

#To test a sample job you need a driver
docker run --rm -it --network spark-docker-compose_spark-network $MYNAME/spark:latest /bin/sh

#To then from teh container
/spark/bin/spark-submit --master spark://spark-master:7077 --class org.apache.spark.examples.SparkPi /spark/examples/jars/spark-examples_2.11-2.4.7.jar 1000


#####Running without docker-compose#####

#To start the MASTER (with a network)
docker run --rm -it --name spark-master --hostname spark-master  -p 7077:7077 -p 8080:8080 --network spark_network  $MYNAME/spark:latest /bin/sh

#Then inside the container
/spark/bin/spark-class org.apache.spark.deploy.master.Master --ip `hostname` --port 7077 --webui-port 8080 

#To start the WORKER (with a network)
docker run --rm -it --name spark-worker --hostname spark-worker  --network spark_network  $MYNAME/spark:latest /bin/sh

#Then inside the container
/spark/bin/spark-class org.apache.spark.deploy.worker.Worker  --webui-port 8080 spark://spark-master:7077
