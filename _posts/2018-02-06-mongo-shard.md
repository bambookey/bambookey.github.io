---
layout: post
title: Mongo Shard集群搭建
category: tech
---

```
[webedit@localredis2 mongo]$ mongodb-3.2.5/bin/mongod  --shardsvr --port 20000 --dbpath ~/mongo/data/shard/s --fork --
s0/ s1/
[webedit@localredis2 mongo]$ mongodb-3.2.5/bin/mongod  --shardsvr --port 20000 --dbpath ~/mongo/data/shard/s0 --fork --
BadValue: --fork has to be used with --logpath or --syslog
try 'mongodb-3.2.5/bin/mongod --help' for more information
[webedit@localredis2 mongo]$ ~/mongo/mongodb-3.2.5/bin/mongod --shardsvr --port 20000 --dbpath ~/mongo/data/shard/s0 --fork --logpath ~/mongo/data/shard/log/s0.log --directoryperdb  
about to fork child process, waiting until server is ready for connections.
forked process: 13959
child process started successfully, parent exiting
[webedit@localredis2 mongo]$
[webedit@localredis2 mongo]$
[webedit@localredis2 mongo]$
[webedit@localredis2 mongo]$ ~/mongo/mongodb-3.2.5/bin/mongod --shardsvr --port 20001 --dbpath ~/mongo/data/shard/s1 --fork --logpath ~/mongo/data/shard/log/s1.log --directoryperdb  
about to fork child process, waiting until server is ready for connections.
forked process: 13998
child process started successfully, parent exiting
[webedit@localredis2 mongo]$
[webedit@localredis2 mongo]$
[webedit@localredis2 mongo]$ mkdir -p ~/mongo/data/shard/config
[webedit@localredis2 mongo]$ ~/mongo/mongodb-3.2.5/bin/mongod --configsvr --port 30000 --dbpath ~/mongo/data/shard/config --fork --logpath ~/mongo/data/shard/log/config.log --directoryperdb  
about to fork child process, waiting until server is ready for connections.
forked process: 14254
child process started successfully, parent exiting
[webedit@localredis2 mongo]$
[webedit@localredis2 mongo]$
[webedit@localredis2 mongo]$ ~/mongo/mongodb-3.2.5/bin/mongos --port 40000 --configdb localhost:30000 --fork --logpath ~/mongo/data/shard/log/route.log --chunkSize 1
2018-02-06T19:11:20.256+0800 W SHARDING [main] Running a sharded cluster with fewer than 3 config servers should only be done for testing purposes and is not recommended for production.
about to fork child process, waiting until server is ready for connections.
forked process: 14359
child process started successfully, parent exiting
[webedit@localredis2 mongo]$ ps -ef | grep mongod
webedit  13959     1  0 19:07 ?        00:00:01 /home/webedit/mongo/mongodb-3.2.5/bin/mongod --shardsvr --port 20000 --dbpath /home/webedit/mongo/data/shard/s0 --fork --logpath /home/webedit/mongo/data/shard/log/s0.log --directoryperdb
webedit  13998     1  0 19:07 ?        00:00:01 /home/webedit/mongo/mongodb-3.2.5/bin/mongod --shardsvr --port 20001 --dbpath /home/webedit/mongo/data/shard/s1 --fork --logpath /home/webedit/mongo/data/shard/log/s1.log --directoryperdb
webedit  14254     1  0 19:09 ?        00:00:00 /home/webedit/mongo/mongodb-3.2.5/bin/mongod --configsvr --port 30000 --dbpath /home/webedit/mongo/data/shard/config --fork --logpath /home/webedit/mongo/data/shard/log/config.log --directoryperdb
webedit  14359     1  0 19:11 ?        00:00:00 /home/webedit/mongo/mongodb-3.2.5/bin/mongos --port 40000 --configdb localhost:30000 --fork --logpath /home/webedit/mongo/data/shard/log/route.log --chunkSize 1
webedit  14400 12709  0 19:11 pts/3    00:00:00 grep mongod
[webedit@localredis2 mongo]$ ~/mongo/
data/          mongodb-3.2.5/
[webedit@localredis2 mongo]$ ~/mongo/mongodb-3.2.5/bin/mongo admin --port 40000
MongoDB shell version: 3.2.5
connecting to: 127.0.0.1:40000/admin
Server has startup warnings:
2018-02-06T19:11:20.269+0800 I CONTROL  [main]
2018-02-06T19:11:20.269+0800 I CONTROL  [main] ** WARNING: Insecure configuration, access control is not enabled and no --bind_ip has been specified.
2018-02-06T19:11:20.269+0800 I CONTROL  [main] **          Read and write access to data and configuration is unrestricted,
2018-02-06T19:11:20.269+0800 I CONTROL  [main] **          and the server listens on all available network interfaces.
2018-02-06T19:11:20.269+0800 I CONTROL  [main]
mongos> db.runCommand({addshard:"localhost:20000"});
{ "shardAdded" : "shard0000", "ok" : 1 }
mongos> db.runCommand({addshard:"localhost:20001"});
{ "shardAdded" : "shard0001", "ok" : 1 }
mongos> use admin
switched to db admin
mongos> db.runCommand({enablesharding:"test"});
{ "ok" : 1 }
mongos> db.runCommand({shardcollection:"test.users",key:{_id:1}});
{ "collectionsharded" : "test.users", "ok" : 1 }
mongos> db.runCommand({listshards:1});
{
	"shards" : [
		{
			"_id" : "shard0000",
			"host" : "localhost:20000"
		},
		{
			"_id" : "shard0001",
			"host" : "localhost:20001"
		}
	],
	"ok" : 1
}
mongos> db.printShardingStatus();
--- Sharding Status ---
  sharding version: {
	"_id" : 1,
	"minCompatibleVersion" : 5,
	"currentVersion" : 6,
	"clusterId" : ObjectId("5a798d5838dae7a3ac6043ab")
}
  shards:
	{  "_id" : "shard0000",  "host" : "localhost:20000" }
	{  "_id" : "shard0001",  "host" : "localhost:20001" }
  active mongoses:
	"3.2.5" : 1
  balancer:
	Currently enabled:  yes
	Currently running:  no
	Failed balancer rounds in last 5 attempts:  0
	Migration Results for the last 24 hours:
		No recent migrations
  databases:
	{  "_id" : "test",  "primary" : "shard0000",  "partitioned" : true }
		test.users
			shard key: { "_id" : 1 }
			unique: false
			balancing: true
			chunks:
				shard0000	1
			{ "_id" : { "$minKey" : 1 } } -->> { "_id" : { "$maxKey" : 1 } } on : shard0000 Timestamp(1, 0)

mongos>
mongos> db.runCommand({isdbgrid:1});
{ "isdbgrid" : 1, "hostname" : "localredis2.lib.hz.infra", "ok" : 1 }
mongos> show dbs;
config  0.001GB
test    0.000GB
```
