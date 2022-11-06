## About Project
I am glad if you use mvn install in the first place because it's a maven. What's more, I build the project by spring-boot
framework, and I use mybatis to operate database. I am sure you guys are good at this. Other than that,test cases are provided
for APIs. I chose mysql and redis as a test data source,so you can run any case without extra config. Redisson is used as
a distributed lock to ensure thread-safety. I utilize canal to ensure the consistency of mysql and redis.

## Tips
you can find init sql script here. **src/main/resources/sql/user_20221029.sql**

## How to run this application
I guess the key point is that you need to replace your own mysql link,username and password. And I reaffirm that it's
the only thing you need to do or else you might fail to run this application. After that, Run Application you can see
the logs about the container is running. If you find that there is any error I didn't mention here,just contact me plz.

## About Document
you can use swagger if you run this application in your local env. Then you will see the documents for UserController.
I believe it is clear enough to get anything you need.

## Archeticture Feature
(1) Canal is used to sync data from mysql master database to redis.
    Canal Acted as a mysql slave and sends dump commands to mysql master database. canal receives binary logs, and
update the data in redis.
    How to run a canal by docker?
    A. docker pull canal:
       docker pull canal/canal-server:v1.1.5
       docker run --name canal -d canal/canal-server:v1.1.5
       docker cp canal:/home/admin/canal-server/conf/canal.properties /opt/canal
       docker cp canal:/home/admin/canal-server/conf/example/instance.properties /opt/canal
    B. modify instance.properties:
       canal.instance.mysql.slaveId=2
       canal.instance.master.address=192.168.168.157:3306
       canal.instance.dbUsername=canal
       canal.instance.dbPassword=canal
       canal.instance.connectionCharset = UTF-8
       # canal.instance.defaultDatabaseName =test
       canal.instance.filter.regex =.\*\\\\..\*
       canal.mq.topic=example
    C. restart canal container with properties file
       docker run --name canal \
       -p 11111:11111 \
       -v /opt/canal/instance.properties:/home/admin/canal-server/conf/example/instance.properties \
       -v /opt/canal/canal.properties:/home/admin/canal-server/conf/canal.properties \
       -d canal/canal-server:v1.1.5

(2) Redisson is used as a distributed lock when updating userInfo
       Considering UserApplication is deployed in a distributed way, reddisson is necessary to avoid conflicts
    between different threads. What's more, Redisson has a watch-dog mechanism to prolong the expire time
    automatically which ensures the integrity of the whole service computing in one thread.
