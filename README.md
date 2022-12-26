# Configure MongoDB sharding (cluster)

To build mongodb sharding architecture, you have to have minimum:
* **1 shard** (replica set / single mongod instance)
* **1 config server** (replica set / single mongod instance) to store shard **metadata**
* **mongos server** to route queries to shards


&nbsp;
&nbsp;


### Configuring Config Server Replica set (CSRS)

Config server is mongod instance that store sharding metadata of cluster. Its configuration is mostly same as default mongod config. Main difference is:
```yaml
...

sharding:
  clusterRole: "configsvr"

...
```
Then you have to add this line to all CSRS members.


&nbsp;
&nbsp;


### Configuring Shard replica sets

To mark replica set as **shard** you have to add cluster role of **shardsvr** to sharding replica set members config file:

```yaml
...

sharding:
  clusterRole: "shardsvr"

...
```


&nbsp;
&nbsp;


### Configuring MongoS

Mongos is service that routes queries to specific shard or executes queries in every shard and merges them for client.

Mongos does not have to have storage nor replica set. But if we need more mongos, we can just create multiple clones.

Example mongos config file:
```yaml
net:
   bindIp: 127.0.0.1
   port: 29000

sharding:
  configDB: "myshard:config/localhost:29010,localhost:29011,localhost:29012"

systemLog:
   destination: file
   path: "/var/lib/mongodb/mongos/mongos.log"
   logAppend: true

processManagement:
   fork: true

setParameter:
   enableLocalhostAuthBypass: false
```

We have to show path to CSRS in mongos configuration, to fetch metadata


&nbsp;
&nbsp;


### Initializing sharding cluster

First you have to start all shard replica sets, and CSRS.

To start mongos process:
```bash
mongos -f mongos.conf
```

After mongos started, we connect to it:
```bash
mongo --host "localhost:29000"
```

And to add shard replica sets:
```bash
sh.addShard("shard-1-replica-set/localhost:28001")

sh.addShard("shard-2-replica-set/localhost:28011")
```
