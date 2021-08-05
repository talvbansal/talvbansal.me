---
title: Replacing Redis with DynamoDB and SQS for Local Docker Development
date: 2021-08-05 17:06:49

tags:
- Docker
- Redis
- DynamoDB
- SQS

autoThumbnailImage: yes
coverImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_1600/v1586202198/posts/bagan-baloons.jpg
coverSize: partial
coverMeta: in
metaAlignment: center
thumbnailImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_280/v1586202198/posts/bagan-baloons.jpg
thumbnailImagePosition: right
---

# Problem

Managed Redis instances are expensive. But Redis is amazing, I use it for handling cache, sessions and queues. 
When speaking to the AWS team about ways of cutting our bills down they recommended moving to DynamoDB and SQS, my initial question was:

> "How do I run the same stuff locally when developing so im not incurring costs?"

Their answer was:

> "Just run it against a real instance?" - ¯\_(ツ)_/¯

I'm really, really big on being able to run things locally. So I set out on working out how I could do the above.

<!-- more -->

### Docker Configuration

Amazon actually provides a docker image for us to create a local DynamoDB instance from.
No such luck for SQS, instead the image we're going to use is based on [elasticmq](https://github.com/softwaremill/elasticmq) which has an SQS compatible interface.

```yaml
# docker-compose.yml
volumes:
  ...
  dynamodb:
    driver: "local"


services:
    ...
    # DynamoDB and Sqs are containers used to simulate the production aws environment...
    dynamodb:
      # In order to persist the data and table we need to mount the
      # volume which can only be done as root on this container...
      user: root
      image: amazon/dynamodb-local:latest
      container_name: myproject-dynamodb
      ports:
        - "8000:8000"
      command: ["-jar", "DynamoDBLocal.jar", "-sharedDb", "-dbPath", "/home/dynamodblocal/data/"]
      volumes:
        - dynamodb:/home/dynamodblocal/data
      networks:
        - default
    sqs:
      image: graze/sqs-local
      container_name: myproject-sqs
      ports:
        - 9324:9324
      volumes:
        - ./elasticmq.conf:/elasticmq.conf
      networks:
        - default
```

Notice that an `elasticmq.conf` file is mounted in the sqs section. 
This file contains the queue configuration, you'll need to create this file in your project and make sure the paths all match:
```bash
nano ./elasticmq.conf
```

And put the following in, it creates 2 queues, one called `default` and another called `service-queue`.

```bash
include classpath("application.conf")

node-address {
   protocol = http
   host = "*"
   port = 9324
   context-path = ""
}

rest-sqs {
   enabled = true
   bind-port = 9324
   bind-hostname = "0.0.0.0"
   // Possible values: relaxed, strict
   sqs-limits = strict
}

queues {
   default {
     defaultVisibilityTimeout = 10 seconds
     delay = 5 seconds
     receiveMessageWait = 0 seconds
   }
   service-queue {
     defaultVisibilityTimeout = 10 seconds
     delay = 5 seconds
     receiveMessageWait = 0 seconds
   }
}
```

#### DynamoDb Configuration
When using the local instance we need to create a table within DynamoDB, first add / update the necessary environment files to `.env`:

```bash
CACHE_DRIVER=dynamodb
SESSION_DRIVER=dynamodb
...
DYNAMODB_CACHE_TABLE="myproject-cache"
DYNAMODB_ENDPOINT="http://dynamodb:8000"
```
Reload the `.env` file within your containers by restarting them `docker-compose down && docker-compose up -d`.

Then use the AWS CLI  to create the table, notice that `--table-name` and `DYNAMODB_CACHE_TABLE` are the same here:
```bash
aws dynamodb create-table --table-name myproject-cache --attribute-definitions AttributeName=key,AttributeType=S --key-schema AttributeName=key,KeyType=HASH --endpoint-url http://localhost:8000 --provisioned-throughput ReadCapacityUnits=2,WriteCapacityUnits=2
```

You can confirm that the table was made by running the following:

```bash
aws dynamodb list-tables --endpoint-url http://localhost:8000
```

And should see:

```javascript
{
    "TableNames": [
        "myproject-cache"
    ]
}
```

#### SQS Configuration
The following instructions will get everything running locally on a queue called `service-queue`.

Update the `.env` file:

```bash
QUEUE_CONNECTION=sqs
# SQS Config
SQS_PREFIX=http://sqs:9324/queue
SQS_QUEUE=service-queue
```

Restart the containers, `docker-compose up -d`.

### Closing

You should be able to dispatch jobs to SQS now and see cache / session items appearing in the local DynamoDB instance!
