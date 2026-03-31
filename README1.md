## Основной функционал:
- В Java package company.vk.edu.distrib.compute.kuznetsovasvetlana6 реализован HTTP сервер с обработкой запросов:
  - GET /v0/status — проверка состояния сервиса (200 OK).
  - GET /v0/entity?id=... — получение данных (200 OK / 404 Not Found).
  - PUT /v0/entity?id=... — запись/обновление данных (201 Created).
  - DELETE /v0/entity?id=... — удаление данных (202 Accepted).

- Реализован интерфейс Dao (MyDaoMemory) для работы с данными в памяти с помощью ConcurrentHashMap.

## Бонусный функционал:
### 1. Persistent Dao
В соответствии с интерфейсом Dao реализован класс MyDaoFile, который обеспечивает хранение данных в файловой системе.

### 2. Load test
Проведено нагрузочное тестирование с помощью wrk2.
Тестирование проводилось в одно соединение для реализаций хранения данных в памяти и в файловой системе для PUT и GET запросов.
Были написаны файлы put.lua и get.lua, в которых формировались запросы со случайным id, причем для реалистичности было сделано так, чтобы id в GET-запросах отличался от созданных в PUT-запросах, чтобы убедиться в том, что ошибки обрабатываются корректно при обращении к несуществующему id

### Результаты:

**Для in-memory реализации:**

PUT: 
- Все запросы успешны
- Средняя задержка: 1.42ms

GET:
- 70% успешно, 30% промахов (поскольку GET-запрос не смог найти нужный ему id)
- Средняя задержка: 1.33ms

**Для реализации с файлами:**

PUT:
- Все запросы успешны
- Средняя задержка: 1.82ms

GET:
- 70% успешно, 30% промахов (поскольку GET-запрос не смог найти нужный ему id)
- Средняя задержка: 1.52ms


## Инструкция по сборке:
### 1. Для запуска сервера с хранением в файловой системе:
```
./gradlew run
```

### 2. Для запуска сервера с хранением в памяти:
В Server.java поменять MyKVServiceFactoryFile на MyKVServiceFactoryMemory.

Аналогично, ```./gradlew run```

### 3. Отправка запросов на сервер:
- HTTP GET /v0/status: ```curl -i -X GET "http://localhost:8080/v0/status"```
- HTTP GET /v0/entity?id=<ID>: ```curl -i -X GET "http://localhost:8080/v0/entity?id=1"```
- HTTP PUT /v0/entity?id=<ID>: ```curl -i -X PUT "http://localhost:8080/v0/entity?id=1" -d "my_data"```
- HTTP DELETE /v0/entity?id=<ID>:  ```curl -i -X DELETE "http://localhost:8080/v0/entity?id=1"```

### 4. Для запуска тестов:
1. docker pull haydenjeune/wrk2
2. Запустить в одном окне сервер
3. Во втором окне выполнить команду:
Для PUT:
```
docker run --rm -v "%cd%:/data" haydenjeune/wrk2 -t1 -c1 -R200 -d30s --latency -s /data/put.lua http://host.docker.internal:8080
```
Для GET:
```
docker run --rm -v "%cd%:/data" haydenjeune/wrk2 -t1 -c1 -R200 -d30s --latency -s /data/get.lua http://host.docker.internal:8080
```
