
# MongoDB Sharded Cluster Setup

This guide outlines the steps to set up a MongoDB sharded cluster with three replica sets, each containing three members.

## Step 1: Create Directories

```bash
mkdir replica11 replica12 replica13
mkdir replica21 replica22 replica23
mkdir replica31 replica32 replica33
mkdir cfg0 cfg1 cfg2
```

---

## Step 2: Change Permissions (Optional)

```bash
chmod -R 777 replica11 
chmod -R 777 replica12 
chmod -R 777 replica13 
chmod -R 777 replica21 
chmod -R 777 replica22 
chmod -R 777 replica23 
chmod -R 777 replica31 
chmod -R 777 replica32 
chmod -R 777 replica33 
chmod -R 777 cfg0 
chmod -R 777 cfg1 
chmod -R 777 cfg2
```

---

## Step 3: Start the Replica Set Servers

```bash
mongod --dbpath replica11 --port 27011 --logpath replica11/log --logappend --fork --bind_ip_all --replSet replicaset1 --shardsvr
mongod --dbpath replica12 --port 27012 --logpath replica12/log --logappend --fork --bind_ip_all --replSet replicaset1 --shardsvr
mongod --dbpath replica13 --port 27013 --logpath replica13/log --logappend --fork --bind_ip_all --replSet replicaset1 --shardsvr

mongod --dbpath replica21 --port 27021 --logpath replica21/log --logappend --fork --bind_ip_all --replSet replicaset2 --shardsvr
mongod --dbpath replica22 --port 27022 --logpath replica22/log --logappend --fork --bind_ip_all --replSet replicaset2 --shardsvr
mongod --dbpath replica23 --port 27023 --logpath replica23/log --logappend --fork --bind_ip_all --replSet replicaset2 --shardsvr

mongod --dbpath replica31 --port 27031 --logpath replica31/log --logappend --fork --bind_ip_all --replSet replicaset3 --shardsvr
mongod --dbpath replica32 --port 27032 --logpath replica32/log --logappend --fork --bind_ip_all --replSet replicaset3 --shardsvr
mongod --dbpath replica33 --port 27033 --logpath replica33/log --logappend --fork --bind_ip_all --replSet replicaset3 --shardsvr
```

---

## Step 4: Verify Running Servers

```bash
ps -ef | grep mongo
```

---

## Step 5: Configure Replica Set 1

```bash
mongo --port 27011
```

Run the following commands in the MongoDB shell:

```javascript
rs.initiate()
rs.add("ip-172-21-35-50.ap-south-1.compute.internal:27012")
rs.add("ip-172-21-35-50.ap-south-1.compute.internal:27013")
rs.conf()
exit
```

---

## Step 6: Configure Replica Set 2

```bash
mongo --port 27021
```

Run the following commands in the MongoDB shell:

```javascript
rs.initiate()
rs.add("ip-172-21-35-50.ap-south-1.compute.internal:27022")
rs.add("ip-172-21-35-50.ap-south-1.compute.internal:27023")
rs.conf()
exit
```

---

## Step 7: Configure Replica Set 3

```bash
mongo --port 27031
```

Run the following commands in the MongoDB shell:

```javascript
rs.initiate()
rs.add("ip-172-21-35-50.ap-south-1.compute.internal:27032")
rs.add("ip-172-21-35-50.ap-south-1.compute.internal:27033")
rs.conf()
exit
```

---

## Step 8: Start the Config Servers

```bash
mongod --configsvr --dbpath cfg0 --port 26050 --fork --logpath log.cfg0 --replSet cfg
mongod --configsvr --dbpath cfg1 --port 26051 --fork --logpath log.cfg1 --replSet cfg
mongod --configsvr --dbpath cfg2 --port 26052 --fork --logpath log.cfg2 --replSet cfg
```

---

## Step 9: Configure Config Servers

```bash
mongo --port 26050
```

Run the following commands in the MongoDB shell:

```javascript
rs.initiate()
rs.add("localhost:26051")
rs.add("localhost:26052")
rs.status()
exit
```

---

## Step 10: Check Running Servers

```bash
ps -ef | grep mongod | grep a0
```

---

## Step 11: Start Mongos Instances

```bash
mongos --configdb "cfg/localhost:26050,localhost:26051,localhost:26052" --fork --logpath log.mongos1 --port 26061
mongos --configdb "cfg/localhost:26050,localhost:26051,localhost:26052" --fork --logpath log.mongos2 --port 26062
```

---

## Step 12: Add Shards to Mongos

```bash
mongo --port 26061
```

Run the following commands in the MongoDB shell:

```javascript
sh.addShard("replicaset1/localhost:27011")
sh.addShard("replicaset2/localhost:27021")
sh.addShard("replicaset3/localhost:27031")
sh.status()
exit
```

---

## Step 13: Enable Sharding for a Database

```bash
mongo --port 26061
```

Run the following commands in the MongoDB shell:

```javascript
sh.enableSharding("dbname")
sh.status()
exit
```

---

## Step 14: Shard a Collection

```bash
mongo --port 26061
```

Run the following commands in the MongoDB shell:

```javascript
sh.shardCollection("dbname.collectionname", { _id: 1 })
sh.status()
exit
```

---

## Step 15: Verify Data Distribution

Run the following commands in the MongoDB shell:

```javascript
use dbname
db.sales.insert({"name": "vivek"})
db.getLastErrorObj()

use config
db.shards.find()
exit
```
