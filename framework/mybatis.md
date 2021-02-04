## 一、配置文件

### mybatis-config.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!-- 通过url引入资源配置信息 -->
    <!--<properties url="http://xxx"/>-->
    <!-- resource引入类路径下的配置信息 -->
    <properties resource="config.properties">
        <!-- 内部定义配置信息 -->
        <property name="" value=""/>
    </properties>
    <settings>
        <!-- 开启驼峰命名与下划线命名自动映射 -->
        <setting name="mapUnderscoreToCamelCase" value="true"/>
        <!-- 指定日志门面 -->
        <setting name="logImpl" value="SLF4J"/>
        <!-- 全局开启mapper文件中的缓存 -->
        <setting name="cacheEnabled" value="true"/>
        <!-- 设置超时时间，它决定数据库驱动等待数据库响应的秒数 -->
        <setting name="defaultStatementTimeout" value="3"/>
        <!-- 查询关联的实体类延迟加载, 用于级联属性 -->
        <setting name="lazyLoadingEnabled" value="true"/>
        <setting name="aggressiveLazyLoading" value="false"/>
        <!-- 修改null的默认映射值为null -->
        <setting name="jdbcTypeForNull" value="NULL"/>
    </settings>
    <typeAliases>
        <!-- 为单个类指定别名 -->
        <typeAlias type="com.example.entity.User" alias="User"/>
        <!-- 为整个包下的类起别名, 别名为类名。 如果想指定别名在对应的类加上@Alias注解 -->
        <package name="com.example.entity"/>
    </typeAliases>
    <!-- 用于配置数据库类型与java类型的映射 指定的类需要实现BaseTypeHandler -->
    <typeHandlers>
        <typeHandler handler="com.example.ExampleTypeHandler" javaType="" jdbcType=""/>
    </typeHandlers>
    <plugins>
        <plugin interceptor=""></plugin>
    </plugins>
    <environments default="dev">
        <environment id="dev">
            <transactionManager type="JDBC"/>
            <dataSource type="com.example.DruidConfig">
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <!-- 类路径 -->
        <mapper resource="mybatis/mapper/xxx.xml"/>
        <!-- url -->
        <mapper url="http://xxx"/>
        <!-- 指明接口, 需要把mapper文件与接口放在同一个目录下并且接口和mapper文件同名 -->
        <mapper class="com.example.UserMapper"/>
        <!-- 批量注册, 规则和上面一样 -->
        <package name="com.example.mapper"/>
    </mappers>
</configuration>
```

### xxx-mapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="cc.p8t.test.mybatis.UserMapper">
    <select id="selectById" resultType="cc.p8t.test.mybatis.User">
        select *
        from t_user
        where id = #{id}
    </select>

    <delete id="deleteById">
        delete
        from t_user
        where id = #{id}
    </delete>

    <!-- 获取自增的主键值 -->
    <insert id="insertUser" useGeneratedKeys="true" keyColumn="id" keyProperty="id">
        insert
        into t_user (username, password)
        values (#{user.username}, #{user.password})
    </insert>

    <update id="updateUser">
        update t_user
        set username = #{user.username, jdbcType=},
            password = #{user.password}
        where id = #{id}
    </update>
</mapper>
```

## 二、SQL参数

1、对于单个参数，取值时可以随便写

```java
User selectById(int id);

select * from t_user where id=#{qnwg}
```

2、多个参数会被封装为一个`Map`，且如果不使用`@Param`，则`key`为`[param1,param2...]`或参数名（以前是`[0,1...]`）

```java
boolean updateUser(int id, String uname, String pwd);

update t_user set uname=#{uname}, pwd=#{pwd} where id=#{id}
update t_user set uname=#{param2}, pwd=#{param3} where id=#{param1}
```

3、使用`@Param`指定参数的`key`值

```java
User selectById(@Param("uid") int id);

select * from t_user where id=#{uid}
```

4、通过`map`传参数

```java
Map<String, Object> map = new HashMap<>();
map.put("uname", "root");
map.put("pwd", "123456");
userMapper.insertUser(map);

insert into t_user(username, password) values(#{uname}, #{pwd})
```

5、集合参数

```java
User selectById(List<int> id);

#{list[0]}
```

## 三、${}和#{}区别

- `#{}`：预编译的方式传入参数，可以防止`sql`注入，但是只能用于原生`jdbc`支持占位符的地方

- `${}`：直接拼接`sql`的方式传入参数，可以用于任何地方

  ```mysql
  # 表名按年划分, 就可以动态按年查询信息, 这个地方就不能用#{}传参
  select * from t_${year}_user
  ```

## 四、#{}更多用法

- `javaType`：指定`java`类型

- `jdbcType`：指定数据库类型。对所有传入值为`null`的参数，`Mybatis`会它映射为`OTHER`类型，然而某些数据库不能正确识别（比如`Oracle`），因此需要重新指明类型

  ```xml
  <!-- 局部设置 -->
  #{email, jdbcType=NULL}
  <!-- 全局设置 -->
  <setting name="jdbcTypeForNull" value="NULL"/>
  ```

- `numericScale`：指定小数点保留几位

## 五、查询返回值

1、返回实体类，只需要指定`resultType`为实体类

```xml
<select id="selectById" resultType="cc.p8t.test.mybatis.User">
    select *
    from t_user
    where id = #{id}
</select>
```

2、返回实体类集合，只需要指定`resultType`为实体类

```xml
<select id="selectAll" resultType="cc.p8t.test.mybatis.User">
    select *
    from t_user
</select>
```

3、返回`Map`且数据只有一条，`key`为列名，`value`为值，只需要指定`resultType`为`map`

```xml
<select id="selectAll" resultType="map">
    select *
    from t_user
    where id = #{id}
</select>
```

3、返回`Map`且数据有多条，`key`为指定的属性，`value`为一个实体类。实现这个操作首先需要把`resultType`指定为实体类， 然后在对应的`mapper`方法上用`@MapKey()`指定`map`的`key`

```xml
@MapKey("id")
Map<String, User> selectAll();

<select id="selectAll" resultType="cc.p8t.test.mybatis.User">
    select *
    from t_user
    where id = #{id}
</select>
```

4、自定义结果集封装：`resultMap`

```xml
<resultMap id="user_mapping" type="cc.p8t.test.mybatis.User">
    <id property="id" column="userId"/>
    <result property="username" column="_username"/>
    <result property="password" column="_password"/>
    <association property="dog" javaType="cc.p8t.test.mybatis.Dog">
        <id property="id" column="dogId"/>
        <result property="dogname" column="_dogname"/>
    </association>
    <collection property="books" ofType="cc.p8t.test.mybatis.Book">
        <id property="id" column="bookId"/>
        <result property="bookname" column="_booklname"/>
    </collection>
</resultMap>

<select id="selectById" resultMap="user_mapping">
    select 
    	t_user._id  as userId,
        t_user._username,
        t_user._password,
        t_dog._id   as dogId,
        t_dog._dogname,
        t_book._id  as bookId,
        t_book._bookname
    from t_user
        left join t_dog on t_user._dogid = dogId
        left join t_book on t_book._userid = userId
    where userId = #{id}
</select>
```

## 六、动态SQL

> 动态`SQL`标签里面使用的是`OGNL`表达式

### if和where

```xml
<select id="selectByCondition" resultMap="cc.p8t.test.mybatis.User">
    select *
    from t_user
    where
    <if test="id != null">
        id = #{id}
    </if>
    <if test="username != null and username != ''">
        and username = #{username}
    </if>
    <if test="password != null and username != ''">
        and password = #{password}
    </if>
    <if test="gender != null and gender == 0 or gender == 1">
        and gender = #{gender}
    </if>
    <if test="email != null and email.trim() != ''">
        and email = #{email}
    </if>
</select>
```

问题：如果第一个`id`没有拼接则`where`后面紧跟`and`，会报错。而且如果一个条件都不满足也会报错

解决：

1、针对第一种情况可以简单的在`where`后面加上`1=1`

2、使用`<where>`标签，它会自动移除紧跟在它后面的`and`或`or`。如果想自定义移除元素使用`<trim>`标签

```xml
<select id="selectByCondition" resultMap="cc.p8t.test.mybatis.User">
    select *
    from t_user
    <where>
        <if test="id!=null">
            id = #{id}
        </if>
        <if test="username != null and username != ''">
            and username = #{username}
        </if>
        <if test="password != null and username != ''">
            and password = #{password}
        </if>
        <...>
    </where>
</select>
```

### trim

`where`是`trim`的子集。where只能解决前面多的元素

```xml
<select id="selectByCondition" resultMap="cc.p8t.test.mybatis.User">
    select *
    from t_user
    <!--
        prefix: 给标签内部生成结果加个前缀
        suffix: 给标签内部生成结果加个后缀
        prefixOverrides: 去掉前面指定的字符串
        suffixOverrides: 去掉后面指定的字符串
	-->
    <trim prefix="where" suffix="" prefixOverrides="or |and " suffixOverrides="or |and ">
        <if test="id!=null">
            id = #{id}
        </if>
        <if test="username != null and username != ''">
            and username = #{username}
        </if>
        <if test="password != null and username != ''">
            and password = #{password}
        </if>
    </trim>
</select>
```

### choose（when, otherwise）

`if`可以同时查询多个条件，`choose`则选择其中一个

```xml
<select id="selectByCondition" resultMap="cc.p8t.test.mybatis.User">
    select *
    from t_user
    <where>
        <choose>
            <when test="id != null">
                id = #{id}
            </when>
            <when test="username != null and username != ''">
                username = #{username}
            </when>
            <otherwise>
                email = #{email}
            </otherwise>
        </choose>
    </where>
</select>
```

### set

`set`配合`if`可以用于动态更新列，传过来什么就更新什么，而且set会自动清除多余的`","`

同样也可以用`trim`实现`set`

```xml
<update id="updateUser">
    update t_user
    <set>
        <if test="user.username != null">
            username = #{user.username},
        </if>
        <if test="user.password != null">
            password = #{user.password}
        </if>
    </set>
    where id = #{user.id}
</update>
```

### foreach

`foreach`用于范围查询

```xml
List<User> selectByRange(@Param("ids")List<Integer> ids); // 集合元素为1 2 3

<select id="selectByRange" resultType="cc.p8t.test.mybatis.User">
    select *
    from t_user
    <where>
        id in
        <foreach collection="ids"   // 集合的名字, 也就是@Param指定的
                 index="i"		    // 索引, 如果传入的集合是map那么就key
                 item="uid"     // 给集合元素起个名字 
                 separator="," 	    // 分隔符
                 open="(" close=")" // 开始符和结束符
		// where内sql拼接的结果就是 id in (1,2,3)
		>
            #{uid}  
        </foreach>
    </where>
</select>
```

## 七、缓存

`Mybatis`默认的缓存机制实现就是一个`Map`。分为两级，一级默认开启，二级默认关闭。当它们都开启时，首先查询二级缓存，然后查询一级缓存，最后查询数据库。

![image-20210102112129044](https://gitee.com/p8t/picbed/raw/master/imgs/20210102112130.png)

### 一级缓存

作用范围：同一个`sqlSession`连接。

开启方式：默认开启，全局配置文件中配置`localCacheScope`为`STATEMENT`可以禁用。

缓存失效情况：

1、`sqlSession`不同

2、两次查询之间含有增删改操作（无论是否修改了要查询的数据）

3、手动清除一级缓存

### 二级缓存

作用范围：基于`namespace`，因此每一个`namespace`都有一个二级缓存

工作机制：一个`sqlSession`**关闭后**，其一级缓存的结果会放在当前`namespace`的二级缓存中

开启方式：首先在全局配置文件中启用二级缓存，然后在需要的`mapper`配置文件开启二级缓存，最后需要实体类实现序列化接口

```xml
<!-- 启用二级缓存 -->
<setting name="cacheEnabled" value="true"/>

<!-- 在mapper中配置二级缓存 -->
<cache eviction="" 		// 过期策略
       flushInterval="" // 缓存过期时间
       readOnly="" 		// 设置为true则返回缓存时直接返回对象的引用, 否则返回拷贝
       size="" 			// 缓存最大数量
       type=""			// 自定义缓存
/>
```

此外，每一个`select`标签也独立选择是否开启二级缓存；增删改查标签可以决定是否清空缓存，默认查询为`false`，其它三个为`true`

## SQL

对于`SELECT`语句需要指定`resultType`

对于`UPDATE、INSERT、DELETE`语句，不需要指定返回值，会自动适应。对于`int`返回影响行数，对于`boolean`返回影响行数是否为0

```xml
<select id="selectInfo" resultType="com.example.entity.xxx"></select>

<insert id="insertInfo"></insert>

<update id="updateInfo"></update>

<delete id="deleteInfo"></delete>
```

