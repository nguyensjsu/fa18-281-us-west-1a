# Golang REST API

## How to start golang app locally

0. Open your terminal

1. set your gopath to golang project directory
```
## Mine is located at:

/Users/ijoe/dev/cmpe281/cmpe281-nerdijoe/golang-rest-api
```

**Note:** I use environment variable in my app to refer to my AWS EC2 MongoDb instance
  ```
  var mongodb_server = os.Getenv("AWS_MONGODB")
  ```
  How to set environtment variable locally
  ```
  $ export AWS_MONGODB=mongodb://<username>:<password>@<aws_public_ip>:27017
  ```

2. build you app
```
$ go build payments
```
3. Run the app from the executable
```
$ ./payments
```

4. Application will run on
```
localhost:3000
```

5. Open Postman, and do some queries

    [Postman API documentation](https://documenter.getpostman.com/view/2775428/RzfdrBT6)

Or use curl command on your terminal:

```
// Get all Payments
curl --request GET \
  --url http://localhost:3000/payments

// Get Payment by id
curl --request GET \
  --url http://localhost:3000/payments/1

// Create Payment
curl --request POST \
  --url http://localhost:8081/payments \
  --header 'Content-Type: application/json' \
  --data '{
	"userId":"55",
	"orderId":"35",
	"totalAmount":30.75
}'

// Edit Payment
curl --request POST \
  --url http://localhost:8081/payments \
  --header 'Content-Type: application/json' \
  --data '{
	"userId":"55",
	"orderId":"35",
	"totalAmount":30.75
}'

// Delete Payment
curl --request DELETE \
  --url http://localhost:8081/payments/3647712c-b439-4bb5-912c-73592955b7cc \
  --header 'Content-Type: application/json'
```


#### Reference
```
// To enable CORS, use this package
// Added Handler package
go get -v github.com/gorilla/handlers
```




## If database is pointing to your EC2, do this:

### Start Mongo Shard on EC2

1. SSH to Query Router Server

2. Login to mongo as admin
```
mongo mongos-query-router:27017 -u admin -p cmpe281 --authenticationDatabase admin
```

3. Enable **Mongos** Service
```
sudo systemctl enable mongos.service
```

4. Start **Mongos** 
```
sudo systemctl stop mongos
sudo systemctl start mongos
sudo systemctl status mongos
```



## Reference

##### Mongo Config Server
1. SSH to one of config server instance (mongos-config1)

2. Login to mongo as Admin
```
mongo mongos-config1:27019 -u admin -p cmpe281 --authenticationDatabase admin
```

#####


Deploy gumball to EC2

#### 1. Create Public EC2 Instance

Configuration:
1. AMI:             CentOS 7 (x86_64) - with Updates HVM
2. Instance Type:   t2.micro
3. VPC:             cmpe281
4. Network:         Public subnet (us-west-1c)
5. Auto Public IP:  Yes
6. SG Open Ports:   22, 80, 8080, 3000
7. Key Pair:        cmpe281-us-west-1


##### 2. Install Docker
	• https://docs.docker.com/install/linux/docker-ce/centos/#install-docker-ce
```
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install docker-ce
```

##### 3. Start Docker
```
sudo systemctl start docker
sudo systemctl is-active docker
```

#### 4. Test Docker Install (Run Hello Container)
```
sudo docker run hello-world
```

#### 5. Login to your docker hub account
```
sudo docker login
```

#### 6. Create Dockerfile
```
FROM golang:latest 
EXPOSE 3000
RUN mkdir /app 
ADD . /app/ 
WORKDIR /app 
ENV GOPATH /app
RUN cd /app ; go install payments
CMD ["/app/bin/payments"]

```


#### 6. Build docker image locally
```
docker build -t golang-payments .
docker images
```

#### 7. Push docker image to dockerhub
```
// Tag your image
docker tag c6ca4908b6ec nerdijoe/golang-payments:latest

// push to your dockerhub
docker push nerdijoe/golang-payments

```



#### 9. Create docker-compose.yml file

#### 10. deploy golang-payments on docker
```
// using docker-compose
docker-compose scale payments=1 
docker-compose up -d --scale payments=1 --no-recreate

// run docker normally
docker run --name payments -e AWS_MONGODB=mongodb://<username>:<password>@<mongo-ec2-instance-ip>:27017 -e MONGODB_DBNAME=cmpe281 -e MONGODB_COLLECTION=payments -p 8000:8000 -d nerdijoe/golang-payments
```


```
docker-compose stop payments
docker-compose rm payments
docker-compose rmi payments
```

See docker logs
```
docker logs -f payments
```


```
docker-ps:
	 docker ps --all --format "table {{.ID}}\t{{.Names}}\t{{.Image}}\t{{.Status}}\t"

docker-ps-ports:
	 docker ps --all --format "table {{.Names}}\t{{.Ports}}\t"
```




## Mongo sharding

1. SSH to Query Router Instance

2. Login to mongo as admin
```
mongo mongos-query-router:27017 -u admin -p cmpe281 --authenticationDatabase admin
```

3. Go to database cmpe281
```
use test
```

4. Enable Sharding
```
use cmpe281
db.payments.ensureIndex( { orderid : "hashed" } )
sh.shardCollection( "cmpe281.payments", { "orderid" : "hashed" } )
```

db.payments.getShardDistribution() 





# GKE

### Open cloud shell

### Set your project id and compute/zone
```
gcloud config set project august-now-224702
gcloud config set compute/zone us-west1-a
export PROJECT_ID="$(gcloud config get-value project -q)"
```

### clone your repo
```
git clone https://github.com/nguyensjsu/cmpe281-nerdijoe.git
```


### set your gopath
```
export GOPATH=$HOME/go
```

**go get commands**
```
rm -rf src/github.com
go get -v github.com/codegangsta/negroni
go get -v github.com/gorilla/mux
go get -v github.com/unrolled/render
go get -v github.com/satori/go.uuid
go get -v github.com/streadway/amqp
go get -v gopkg.in/mgo.v2
go get -v gopkg.in/mgo.v2/bson
go get -v github.com/gorilla/handlers

```


### build your image
```
docker build -t gcr.io/${PROJECT_ID}/payments:v1 .

gcloud auth configure-docker
```


### push 
```
docker push gcr.io/${PROJECT_ID}/payments:v1
```

### create cluster
```
gcloud container clusters create golang-cluster --num-nodes=3

gcloud compute instances list


kubectl run payments --image=gcr.io/${PROJECT_ID}/payments:v1 --port 8000 --env AWS_MONGODB=mongodb://<user>:<password>@ec2-54-215-217-211.us-west-1.compute.amazonaws.com:27017 --env MONGODB_DBNAME=cmpe281 --env MONGODB_COLLECTION=payments
```

```
kubectl expose deployment payments --type=LoadBalancer --port 80 --target-port 8000
kubectl get service

kubectl scale deployment payments --replicas=3
kubectl get deployment payments

```

### deploy new version
### build your image
```
docker build -t gcr.io/${PROJECT_ID}/payments:v2 .
docker push gcr.io/${PROJECT_ID}/payments:v2

kubectl set image deployment/payments payments=gcr.io/${PROJECT_ID}/payments:v2

```


### Delete kubernetes service and cluster
```
kubectl delete service payments
gcloud container clusters delete cluster
```