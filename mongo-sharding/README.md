1. создаем контейнеры: 2 шарда, конфигурационный и роутер - в compose.yaml
2. добавляем к ним сервис на python. сервис добавляем в общую сеть app-network.
3. для сервиса указываем точкой подключения роутер-сервис. указываем порт, потому что по-умолчанию драйвер питона ломится на дефолтный порт 27017.
4. запускаем сборку. 
> docker-compose -f compose.yaml up --build --remove-orphans --force-recreate
5. дальше (из примера) инициализируем конфигурацию
> docker exec -it configSrv mongosh --port 27017
``` 
rs.initiate(
    {
        _id : "config_server",
        configsvr: true,
        members: [
            { _id : 0, host : "configSrv:27017" }
        ]
    }
);
exit();
```
6. настраиваем шарды
> docker exec -it shard1 mongosh --port 27018
``` 
rs.initiate(
    {
      _id : "shard1",
      members: [
        { _id : 0, host : "shard1:27018" },
      ]
    }
);
exit();
```

> docker exec -it shard2 mongosh --port 27019
``` 
rs.initiate(
    {
      _id : "shard2",
      members: [
        { _id : 1, host : "shard2:27019" }
      ]
    }
  );
exit();
```
7. инцициализируем роутер и наполняем его данными.
> docker exec -it mongos_router mongosh --port 27020
``` 
sh.addShard( "shard1/shard1:27018");
sh.addShard( "shard2/shard2:27019");

sh.enableSharding("somedb");
sh.shardCollection("somedb.helloDoc", { "name" : "hashed" } )

use somedb
for(var i = 0; i < 1000; i++) db.helloDoc.insert({age:i, name:"ly"+i})

db.helloDoc.countDocuments() 
exit();
```
8. запускаем приложение на порту 8080
