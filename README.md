# MachineStreamMonitor
 
## Solution for [backend](http://codingcase.zeiss.services/?type=backend)
-----

This document explains how to implement the solution for prototypical backend use .net microservice. Based on the circumstances of monitoring assets remotely in near real-time, we use microservice and MQ to reinforce the reliability and scaling solution. There have two architectures as planned. 

---

## Architecture

```Plan A```: 

We split the functions of ingesting and storing, exposing the data as two microservices. Ingest data service will force on receive data from a WebSocket connection. It will enqueue the data immediately once WebSocket has data. Another service named expose data service will consume and store the data from the queue. We plan to use Azure SQL, but it seems too expensive for this smart maintenance solution.

![PlanA](Picture/PlanA.png)

```Plan B```: 

We plan to use MongoDB and add a gateway to redirect the routes of these requests. So there will have five images in the docker container without Azure resources. 

![PlanB](Picture/PlanB.png)

## Run on CentOS (Linux)

### Prerequisites

1. rabbitmq server windows
1. mongodb for windows

#### Install rabbitmq on Docker
1. Search rabbitmq images on docker

``` docker command
# docker search rabbitmq
```
![rabbitmq1](Picture/rabbitmq1.png)

2. Install the latest rabbitmq images on docker

``` docker command
# docker pull rabbitmq
```
![rabbitmq2](Picture/rabbitmq2.png)

3. Run image of rabbitmq on docker

``` docker command
# docker run -d --name rabbit -p 15672:15672 -p 5673:5672 rabbitmq
```
![rabbitmq3](Picture/rabbitmq3.png)

#### Install MongoDB on Docker
1. Search mongo images on docker

``` docker command
# docker search mongo
```
![mongo1](Picture/mongo1.png)

2. Install the latest mongo images on docker

``` docker command
# docker pull mongo:latest
```
![mongo2](Picture/mongo2.png)

3. Run image of mongo on docker

``` docker command
# docker run -itd --name mongo -p 27017:27017 mongo --auth
```

4. Create a user for mongo on docker

``` docker command
# docker exec -it mongo mongo admin
 >  db.createUser({ user:'admin',pwd:'123456',roles:[ { role:'userAdminAnyDatabase', db: 'admin'},"readWriteAnyDatabase"]});
 >  db.auth('admin', '123456')
```

### Run IngestDataService on docker

1. Config the rabbitmq settings in the file of appsettings.json under the project, especially replace the IP with your host.

``` c#
  "MQSettings": {
    "RabbitMqRootUri": "rabbitmq://192.168.1.65",
    "RabbitMqQueueName": "machineStream",
    "RabbitMqUri": "rabbitmq://192.168.1.65/machineStream",
    "UserName": "guest",
    "Password": "guest",
    "NotificationServiceQueue": "notification.service"
  }
 ```

 2. Zip the project and upload it to the centOS via MobaXterm then unzip it

 ``` docker command
# unzip "/zip/MachineStream/IngestDataService.zip"
```
![IN2](Picture/IN1.png)

3. Change the directory to the project and Build this service image

 ``` docker command
# cd IngestDataService
# docker build -t ingestdataservice:dev .
```
![IN3](Picture/IN3.png)

4. Run this service image and check the status

 ``` docker command
# docker run --name ingestdataservice -d -p 8008:80 ingestdataservice:dev
# docker ps -a
```

5. Verify this service when typing the URI in the browser. Note, please use your IP address instead.

![IN5](Picture/IN5.png)

6. Send the JSON data to Ingest Data Service via Postman with a WebSocket connection to ws://192.168.1.65:8008/ws

``` json
{
  "topic": "events",
  "ref": null,
  "payload": {
    "machine_id": "59d9f4b4-018f-43d8-92d0-c51de7d987e5",
    "id": "41bb0908-15ba-4039-8c4f-8b7b99260eb2",
    "timestamp": "2017-04-16T19:42:26.542614Z",
    "status": "running"
  },
  "event": "new"
}
```
![IN6](Picture/IN6.png)

### Run ExposeDataService on docker

1. Config the rabbitmq settings in the file of appsettings.json under the project, especially replace the IP with your host.

``` c#
    "MQSettings": {
    "RabbitMqRootUri": "rabbitmq://192.168.1.65",
    "RabbitMqQueueName": "machineStream",
    "RabbitMqUri": "rabbitmq://192.168.1.65/machineStream",
    "UserName": "guest",
    "Password": "guest",
    "NotificationServiceQueue": "notification.service"
  },
  "MachineStreamDatabase": {
    "ConnectionString": "mongodb://admin:123456@192.168.1.65:27017",
    "DatabaseName": "MachineStream",
    "StreamCollectionName": "Stream"
  },
 ```

 2. Zip the project and upload it to the centOS via MobaXterm then unzip it

 ``` docker command
# unzip "/zip/MachineStream/IngestDataService.zip"
```

3. Change the directory to the project and Build this service image

 ``` docker command
# cd ExposeDataService
# docker build -t exposedataservice:dev .
```

4. Run this service image and check the status

 ``` docker command
# docker run --name exposedataservice -d -p 8080:80 exposedataservice:dev
# docker ps -a
```

5. Verify this service when typing the URI in the browser. Note, please use your IP address instead.
Here is the API expose and one request:
![EX5](Picture/EX5.png)
![EX6](Picture/EX6.png)

### Run MachineStreamGateway on docker

1. Config the rabbitmq settings in the file of Routes.json under the project, especially replace the IP with your host.

``` c#
    "Routes": [
    {
      "DownstreamPathTemplate": "/ws",
      "DownstreamScheme": "ws",
      "DownstreamHostAndPorts": [
        {
          "Host": "192.168.1.65",
          "Port": 8008
        }
      ],
      "UpstreamPathTemplate": "/ws",
      "UpstreamHttpMethod": [ "GET", "POST", "PUT", "DELETE", "OPTIONS" ]
    },
    {
      "DownstreamPathTemplate": "/api/MachineStream",
      "DownstreamScheme": "https",
      "DownstreamHostAndPorts": [
        {
          "Host": "192.168.1.65",
          "Port": 8080
        }
      ],
      "UpstreamPathTemplate": "/MachineStream",
      "UpstreamHttpMethod": [ "Get", "POST" ]
    },
    {
      "DownstreamPathTemplate": "/api/MachineStream/{id}",
      "DownstreamScheme": "https",
      "DownstreamHostAndPorts": [
        {
          "Host": "192.168.1.65",
          "Port": 8080
        }
      ],
      "UpstreamPathTemplate": "/MachineStream/{id}",
      "UpstreamHttpMethod": [ "Get", "POST", "PUT" ]
    }
  ]
 ```

 2. Zip the project and upload it to the centOS via MobaXterm then unzip it

 ``` docker command
# unzip "/zip/MachineStream/MachineStreamGateway.zip"
```

3. Change the directory to the project and Build this service image

 ``` docker command
# cd MachineStreamGateway
# docker build -t machinestreamgateway:dev .
```

4. Run this service image and check the status

 ``` docker command
# docker run --name machinestreamgateway -d -p 8888:80 machinestreamgateway:dev
# docker ps -a
```

5. Verify this service when typing the URI in the Postman and browser. Note, please use your IP address instead.

![GW1](Picture/GW1.png)
![GW2](Picture/GW2.png)
![GW3](Picture/GW3.png)