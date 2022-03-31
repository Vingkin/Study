### 项目目录树

* controller
  * RedisController.java
* dao
  * impl
    * RedisDaoImpl.java
  * RedisDao.java
* service
  * impl
    * RedisServiceImpl.java
  * RedisService.java
* Application.java

### 需要的坐标

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

### 持久层代码

RedisDao

```java
public interface RedisDao {

    Set<String> getKeys();

    DataType getTypeByKey(String key);

    Object getStringValueByKey(String key);

    Map<?, ?>  getHashValueByKey(String key);

    List<?> getListValueByKey(String key);

    Set<?> getSetValueByKey(String key);

    Set<?> getZSetValueByKey(String key);

}
```

RedisDaoImpl

```java
@Repository
public class RedisDaoImpl implements RedisDao {

    @Autowired
    private RedisTemplate<String, ?> redisTemplate;


    @Override
    public Set<String> getKeys() {
        return redisTemplate.keys("*");
    }

    @Override
    public DataType getTypeByKey(String key) {
        return redisTemplate.type(key);
    }

    @Override
    public Object getStringValueByKey(String key) {
        return redisTemplate.boundValueOps(key).get();
    }

    @Override
    public Map<?, ?> getHashValueByKey(String key) {
        return redisTemplate.boundHashOps(key).entries();
    }

    @Override
    public List<?> getListValueByKey(String key) {
        return redisTemplate.boundListOps(key).range(0, -1);
    }

    @Override
    public Set<?> getSetValueByKey(String key) {
        return redisTemplate.boundSetOps(key).members();
    }

    @Override
    public Set<?> getZSetValueByKey(String key) {
        return redisTemplate.boundZSetOps(key).range(0, -1);
    }
}
```

### 业务层代码

RedisService

```java
public interface RedisService {

    StringBuilder getRedis();

}
```

RedisServiceImpl

```java
@Service
public class RedisServiceImpl implements RedisService {

    @Autowired
    private RedisDao redisDao;

    @Override
    public StringBuilder getRedis() {
        Set<String> keys = redisDao.getKeys();
        long id = 1;
        StringBuilder str = new StringBuilder();

        str.append("<div style='display: flex; justify-content: center'><table border='1'><tr><th>id</th><th>key</th><th>type</th><th>value</th></tr>");
        for (String key : keys) {
            DataType type = redisDao.getTypeByKey(key);
            Object value = null;
            if (type == DataType.STRING) {
                value = redisDao.getStringValueByKey(key);
            } else if (type == DataType.HASH) {
                value = redisDao.getHashValueByKey(key);
            } else if (type == DataType.LIST) {
                value = redisDao.getListValueByKey(key);
            } else if (type == DataType.SET) {
                value = redisDao.getSetValueByKey(key);
            } else if (type == DataType.ZSET) {
                value = redisDao.getZSetValueByKey(key);
            }
            str.append("<tr><td>");
            str.append(id);
            str.append("</td><td>");
            str.append(key);
            str.append("</td><td>");
            str.append(type);
            str.append("</td><td>");
            str.append(value);
            str.append("</td></tr>");
            id++;
        }
        str.append("</table></div>");
        return str;
    }
}
```

### 控制器编写

```java
@RestController
public class RedisController {

    @Autowired
    private RedisService redisService;

    @RequestMapping("/")
    public String index() {
        return "<a href='/redis'>Redis</a><br/>'";
    }

    @RequestMapping("/redis")
    public StringBuilder RedisMonitor() {
        return redisService.getRedis();
    }

}
```

配置文件application.yml

```yaml
spring:
  redis:
    host: localhost
    port: 6379
    password: password

server:
  port: 8080
```

