Запустить контейнеры:

> docker compose up -d

Чтобы всё полноценно заработало, нужно выполнить ряд команд после запуска.

Подключитесь к серверу конфигурации и сделайте инициализацию:

docker exec -it configSrv mongosh --port 27017

> rs.initiate(
  {
    _id : "config_server",
       configsvr: true,
    members: [
      { _id : 0, host : "configSrv:27017" }
    ]
  }
);
> exit();

Инициализируйте шарды и настройте репликацию:

docker exec -it shard1_0 mongosh --port 27018

> rs.initiate({_id: "shard1", members: [
  {_id: 0, host: "shard1_0:27018"},
  {_id: 1, host: "shard1_1:27019"},
  {_id: 2, host: "shard1_2:27020"}
  ]});

> exit();

docker exec -it shard2_0 mongosh --port 27021

> rs.initiate({_id: "shard2", members: [
  {_id: 1, host: "shard2_0:27021"},
  {_id: 2, host: "shard2_1:27022"},
  {_id: 3, host: "shard2_2:27023"}
  ]});

> exit();

Инцициализируйте роутер и наполните его тестовыми данными:

docker exec -it mongos_router mongosh --port 27024

> sh.addShard( "shard1/shard1_0:27018");
> sh.addShard( "shard1/shard1_1:27019");
> sh.addShard( "shard1/shard1_2:27020");
> sh.addShard( "shard2/shard2_0:27021");
> sh.addShard( "shard2/shard2_1:27022");
> sh.addShard( "shard2/shard2_2:27023");

> sh.enableSharding("somedb");
> sh.shardCollection("somedb.helloDoc", { "name" : "hashed" } )

> use somedb

> for(var i = 0; i < 1000; i++) db.helloDoc.insert({age:i, name:"ly"+i})

> db.helloDoc.countDocuments() 
> exit(); 


Установите переменную окружения:

REDIS_URL=redis://redis_1:6379