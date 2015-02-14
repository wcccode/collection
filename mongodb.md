#mongodb

mongodb Api  http://docs.mongodb.org/manual/

mongodb node-driver api http://mongodb.github.io/node-mongodb-native/1.4/api-generated/collection.html

mongodb基础知识  http://christiankvalheim.com/post/a_basic_introduction_to_mongodb/

连接池  https://blog.compose.io/node-js-mongodb-and-pool-pollution-problems/

http://www.infoq.com/articles/data-model-mongodb

http://www.toadworld.com/platforms/nosql/w/wiki/343.mongodb-and-e-commerce.aspx#Querying_Orders


/******************************/<br/>
#take advantage of indexes
$in、$all

#cann't take advantage of indexes
$nin、$ne、$size、$where、$regex、$mod<br/>
combination with another query term that does use an index



启动数据库<br/>
mongod --dbpath "D:\mongodb\data\db" --logpath "D:\mongodb\log\MongoDB.log"<br/>

创建集群<br/>
mongod --replSet myapp --dbpath "D:\mongodb\data\db" --port 4000<br/>
mongod --replSet myapp --dbpath "D:\mongodb\data\db2" --port 4001<br/>
mongod --replSet myapp --dbpath "D:\mongodb\data\arbiter" --port 4002<br/>

mongo --port 4000 连接主节点<br/>
rs.add("host:4001")添加副节点<br/>
rs.add("host:4002", {arbiterOnly: true})添加仲裁者<br/>

