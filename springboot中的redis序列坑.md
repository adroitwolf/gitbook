---
title: springboot中的redis序列坑
date: 2019-11-29 11:02:58
categories:
- springboot
tags:
- springboot
- redis
---


# springboot中的redis序列坑

当作到博客的点击排行功能的时候，心不甘情不愿的使用起了redis，但是redis存进来的key在我的redisDesktopManage中是这样的

![](https://s2.ax1x.com/2019/11/29/QktA6s.png)

我写进去的是一个字符串,这里则是一串Hex.当然，我并不是这样不行，只是这样太不直观了。

经过排查得知，如果你的redis相关jar包导入的是这个话，就会产生上面的状况

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

```

这是因为上面的jar默认导入的jdk的序列化机制.

想要解决上面的问题有两个解决办法：
一个就是在我们对redis操作前，现将我们的对象用json序列化，但是这样我们的代码会变得特别臃肿，不符合代码整洁之道。
第二个就是写一个配置文件，将redis默认序列化机制换成我们的Jackson序列化


所以我们需要自定义一个序列化工具，比如说阿里的Fastjson什么的，但是之前开始做的时候好多博客都是下面的这种写法
> 观前提示，下面的代码是错误的！


```java


import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;

/**
 * Created with IntelliJ IDEA.
 * User: WHOAMI
 * Time: 2019 2019/11/29 10:46
 * Description:redis序列化配置
 */
@Configuration
public class RedisConfig {

    @Bean
    @ConditionalOnMissingBean(name = "redisTemplate")
    public RedisTemplate<String,Object> redisTemplate(RedisConnectionFactory redisConnectionFactory){
        Jackson2JsonRedisSerializer<Object> jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<Object>(Object.class);
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);
        RedisTemplate<String, Object> template = new RedisTemplate<String, Object>();
        template.setConnectionFactory(redisConnectionFactory);
        template.setKeySerializer(jackson2JsonRedisSerializer);
        template.setValueSerializer(jackson2JsonRedisSerializer);
        template.setHashKeySerializer(jackson2JsonRedisSerializer);
        template.setHashValueSerializer(jackson2JsonRedisSerializer);
        template.afterPropertiesSet();
        return template;

    }
}


```


这样我们的数据就会变成这样的啦:


![](https://s2.ax1x.com/2019/11/29/QkULpF.png)

![](https://s2.ax1x.com/2019/12/03/QMUUAS.png)



上面的代码乍一看没什么问题，But好巧不巧，我的业务是直接将一个对象存进去了，结果反序列的时候直接报错！！！

![](https://s2.ax1x.com/2019/12/03/QMdjOI.png)

查看一下报错就会看到我们的Jackson2的反序列化工具出错了！！


一开始我并没有以为是序列化不对，我以为是我的entity没有序列化导致的，所以我在我的类里面实现了序列化。还是不行，后来经过多方查阅得知，Jackson2的锅。

## 正确做法

既然Jackson2不行，那可以利用阿里的fastjson的序列器。具体代码如下:

现在我们需要两个序列器一个是StringSerializer和FastJsonSerializer

StringSerializer是用来序列K的，而后面那个是序列V的。


```java

public class StringRedisSerializer  implements RedisSerializer<Object> {

    private final Charset charset;

    private final String target = "\"";

    private final String replacement = "";

    public StringRedisSerializer() {
        this(Charset.forName("UTF8"));
    }

    public StringRedisSerializer(Charset charset) {
        Assert.notNull(charset, "Charset must not be null!");
        this.charset = charset;
    }


    @Override
    public byte[] serialize(Object o) throws SerializationException {
        String string = JSON.toJSONString(o);
        if (string == null) {
            return null;
        }
        string = string.replace(target, replacement);
        return string.getBytes(charset);
    }

    @Override
    public String  deserialize(byte[] bytes) throws SerializationException {
        return (bytes == null ? null : new String(bytes, charset));
    }
}

```


```java

public class FastJsonRedisSerializer<T> implements RedisSerializer<T> {

    private static final Charset DEFAULT_CHARSET = Charset.forName("UTF-8");

    private Class<T> type;


    public FastJsonRedisSerializer(Class<T> type) {
        super();
        this.type = type;
    }

    @Override
    public byte[] serialize(T t)  {
        if(null == t){

            return new byte[0];
        }
        try{

            return JSON.toJSONString(t, SerializerFeature.WriteClassName).getBytes(DEFAULT_CHARSET);
        }catch (SerializationException ex){
            throw new SerializationException("Something wrong..." + ex.getMessage(),ex);
        }

    }

    @Override
    public T deserialize(byte[] bytes) throws SerializationException {
        if (bytes == null || bytes.length == 0) {
            return null;
        }
        try {

            String str = new String(bytes, DEFAULT_CHARSET);
            return (T) JSON.parseObject(str, type);
        } catch (Exception ex) {
            throw new SerializationException("Could not deserialize: " + ex.getMessage(), ex);
        }

    }
}

```



```java

@Configuration
public class RedisConfig {

    @Bean
    @ConditionalOnMissingBean(name = "redisTemplate")
    public RedisTemplate<String,Object> redisTemplate(RedisConnectionFactory redisConnectionFactory){

        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory);

        FastJsonRedisSerializer<Object> fastJsonRedisSerializer = new FastJsonRedisSerializer(Object.class);
        ParserConfig.getGlobalInstance().addAccept("run.app.");

        template.setValueSerializer(fastJsonRedisSerializer);
        template.setHashValueSerializer(fastJsonRedisSerializer);


        template.setKeySerializer(new StringRedisSerializer());
        template.setHashKeySerializer(new StringRedisSerializer());

        template.afterPropertiesSet();
        return template;

    }
}


```


这样取出来的数据就正常啦

![](https://s2.ax1x.com/2019/12/03/QM6qsg.png)
