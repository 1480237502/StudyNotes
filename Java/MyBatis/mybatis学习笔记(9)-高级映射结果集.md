
---
[toc]
---

# 一对一

## resultType实现

- sql语句


确定查询的主表：订单表

确定查询的关联表：用户表

关联查询使用内连接？还是外连接？

由于orders表中有一个外键（user_id），通过外键关联查询用户表只能查询出一条记录，可以使用内连接。

```sql
SELECT 
  orders.*,
  USER.username,
  USER.sex,
  USER.address 
FROM
  orders,
  USER 
WHERE orders.user_id = user.id
```

- 创建pojo

将上边sql查询的结果映射到pojo中，pojo中必须包括所有查询列名。

原始的Orders.java不能映射全部字段，需要新创建的pojo。

创建一个pojo继承包括查询字段较多的po类。

对应数据表的几个pojo类(Items,Orderdetail,Orders)就是把该类的属性名设为和数据表列字段名相同，并为这些属性添加getter和setter，在这里就不贴代码了，只贴出对应于关联查询的自定义pojo类`OrdersCustom`的代码

```java
package cn.edu.wtu.po;

/**
 * @Package cn.edu.wtu.po
 * @ClassName OrdersCustom
 * @Description 通过此类映射订单和用户查询的结果，让此类继承包括 字段较多的pojo类
 * @Date 19/11/10 11:17
 * @Author LIM
 * @Version V1.0
 */
public class OrdersCustom extends Orders{
    private String username;
    private String sex;
    private String address;

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    @Override
    public String toString() {
        return "OrdersCustom{" +
                "username='" + username + '\'' +
                ", sex='" + sex + '\'' +
                ", address='" + address + '\'' +
                '}';
    }
}

```



- OrderMapper.xml

```xml
 <!-- 查询订单关联查询用户信息 -->
<select id="findOrdersUser"  resultType="com.iot.mybatis.po.OrdersCustom">
  SELECT
      orders.*,
      user.username,
      user.sex,
      user.address
    FROM
      orders,
      user
    WHERE orders.user_id = user.id
</select>
```


- OrdeMapper.java

```java
//查询订单关联查询用户信息
public List<OrdersCustom> findOrdersUser()throws Exception;
}
```



## resultMap实现

使用resultMap将查询结果中的订单信息映射到Orders对象中，在orders类中添加User属性，将关联查询出来的用户信息映射到orders对象中的user属性中。

- 定义resultMap


```xml
   <!-- 订单查询关联用户的resultMap
    将整个查询的结果映射到cn.edu.wtu.po.Orders中
    -->
    <resultMap id="OrdersUsersResultMap" type="cn.edu.wtu.po.Orders">
        <!-- 配置映射的订单信息 -->
        <!-- id：指定查询列中的唯一标识，订单信息的中的唯 一标识，如果有多个列组成唯一标识，配置多个id
            column：订单信息的唯一标识列
            property：订单信息的唯一标识列所映射到Orders中哪个属性
          -->
        <id column="id" property="id"/>
        <result column="user_id" property="userId"/>
        <result column="createtime" property="createtime"/>
        <result column="number" property="number"/>
        <result column="note" property="note"/>
        <!-- 配置映射的关联的用户信息 -->
        <!-- association：用于映射关联查询单个对象的信息
        property：要将关联查询的用户信息映射到Orders中哪个属性
         -->
        <association property="user" javaType="cn.edu.wtu.po.User">
            <!-- id：关联查询用户的唯一标识
            column：指定唯 一标识用户信息的列
            javaType：映射到user的哪个属性
             -->
            <id column="user_id" property="id"/>
            <result column="username" property="username"/>
            <result column="sex" property="sex"/>
            <result column="address" property="address"/>
        </association>
    </resultMap>
```


- statement定义


```xml
<!-- 查询订单关联查询用户信息 -->
<select id="findOrdersUserResultMap" resultMap="OrdersUserResultMap">
    SELECT
    orders.*,
    user.username,
    user.sex,
    user.address
    FROM
    orders,
    user
    WHERE orders.user_id = user.id
</select>
```

- OrderMapper.java

```java
//查询订单关联查询用户使用resultMap
public List<Orders> findOrdersUserResultMap()throws Exception;
```

- 测试代码

```java
@Test
public void testFindOrdersUserResultMap() throws Exception {

	SqlSession sqlSession = sqlSessionFactory.openSession();
	// 创建代理对象
	OrdersMapperCustom ordersMapperCustom = sqlSession
			.getMapper(OrdersMapperCustom.class);

	// 调用maper的方法
	List<Orders> list = ordersMapperCustom.findOrdersUserResultMap();

	System.out.println(list);

	sqlSession.close();
}
```

## resultType和resultMap实现一对一查询小结

实现一对一查询：

- resultType：使用resultType实现较为简单，如果pojo中没有包括查询出来的列名，需要增加列名对应的属性，即可完成映射。如果没有查询结果的特殊要求建议使用resultType。
- resultMap：需要单独定义resultMap，实现有点麻烦，如果对查询结果有特殊的要求，使用resultMap可以完成将关联查询映射pojo的属性中。
- resultMap可以实现延迟加载，resultType无法实现延迟加载。

# 一对多
## 需求
查询订单及订单明细的信息。(根据数据库模型分析的结果来查询)
使用resultMap
![](https://krislin.oss-cn-beijing.aliyuncs.com/pictures/study-notes-images/%E4%B8%80%E5%AF%B9%E5%A4%9A%E6%95%B0%E6%8D%AE%E5%BA%93%E6%9F%A5%E8%AF%A2%E5%9B%BE.jpg)

## 要求
对orders映射不能出现重复记录。

## 解决思路
1. 在orders.java类中添加List<>, orderDetails属性。
2. 最终会将订单信息映射到orders中，订单所对应的订单明细映射到orders中的orderDetails属性中。
3. 映射成的orders记录数为两条（orders信息不重复）
4. 每个orders中的orderDetails属性存储了该订单所对应的订单明细

## resultMap
```xml
<!-- 订单及订单明细的resultMap
    使用extends继承，不用在中配置订单信息和用户信息的映射
-->
<resultMap id="OrdersAndOrderDetailResultMap" type="cn.edu.wtu.po.Orders" extends="OrdersUsersResultMap">
        <!-- 订单信息 -->
        <!-- 用户信息 -->
        <!-- 使用extends继承，不用在中配置订单信息和用户信息的映射 -->
        <!-- 订单明细信息
        一个订单关联查询出了多条明细，要使用collection进行映射
        collection：对关联查询到多条记录映射到集合对象中
        property：将关联查询到多条记录映射到cn.edu.wtu.po.Orders哪个属性
        ofType：指定映射到list集合属性中pojo的类型
         -->
        <collection property="orderDetails" ofType="cn.edu.wtu.po.OrderDetail">
            <!-- id：订单明细唯 一标识
            property:要将订单明细的唯 一标识 映射到cn.edu.wtu.po.OrderDetail的哪个属性
              -->
            <id column="orderDetail_id" property="id"/>
            <result column="items_id" property="itemsId"/>
            <result column="order_id" property="ordersId"/>
            <result column="items_num" property="itemsNum"/>
        </collection>
</resultMap>
```
## OrderMapper.xml
```xml
<!-- 查询订单关联查询用户及订单明细，使用resultMap -->
    <select id="findOrdersAndOrderDetailResultMap" resultMap="OrdersAndOrderDetailResultMap">
        SELECT
          orders.*,
          user.username,
          user.sex,
          user.address,
          orderdetail.id orderdetail_id,
          orderdetail.items_id,
          orderdetail.items_num,
          orderdetail.orders_id
        FROM
          orders,
          user,
          orderdetail
        WHERE orders.user_id = user.id AND orderdetail.orders_id=orders.id;
    </select>
```
## OrderMapper.java
```java
/**
     * 查询订单关联查询用户及订单明细，使用resultMap
     * @return Orders的列表
     * @throws Exception
     */
    public List<Orders> findOrdersAndOrderDetailResultMap() throws Exception;
```

## 测试
```java
@Test
    public void testFindOrdersAndOrderDetailResultMap() throws Exception{
        // 创建会话
        SqlSession sqlSession = sqlSessionFactory.openSession();
        // 创建代理对象
        OrderMapper orderMapper = sqlSession.getMapper(OrderMapper.class);

        List<Orders> lists = orderMapper.findOrdersAndOrderDetailResultMap();

        for (Orders list:lists){
            System.out.println(list.toString());
        }
    }
```
## 小结

mybatis使用resultMap的collection对关联查询的多条记录映射到一个list集合属性中。

使用resultType实现：将订单明细映射到orders中的orderdetails中，需要自己处理，使用双重循环遍历，去掉重复记录，将订单明细放在orderdetails中。


另外，下面这篇文章对一对多的resultMap机制解释的很清楚：

> [MyBatis：一对多表关系详解(从案例中解析)](http://blog.csdn.net/xzm_rainbow/article/details/15336933)

# 多对多

## 需求
查询用户及用户购买商品信息。

查询主表是：用户表

关联表：由于用户和商品没有直接关联，通过订单和订单明细进行关联，所以关联表：orders、orderdetail、items

## sql
```sql
SELECT 
  orders.*,
  user.username,
  user.sex,
  user.address,
  orderdetail.id orderdetail_id,
  orderdetail.items_id,
  orderdetail.items_num,
  orderdetail.orders_id,
  items.name items_name,
  items.detail items_detail,
  items.price items_price
FROM
  orders,
  user,
  orderdetail,
  items
WHERE orders.user_id = user.id AND orderdetail.orders_id=orders.id AND orderdetail.items_id = items.id
```

## 映射思路

1. 将用户信息映射到user中。

2. 在user类中添加订单列表属性`List<Orders> orderslist`，将用户创建的订单映射到orderslist

3. 在Orders中添加订单明细列表属性`List<OrderDetail>orderdetials`，将订单的明细映射到orderdetials

4. 在OrderDetail中添加`Items`属性，将订单明细所对应的商品映射到Items

## resultMap
```xml
<!-- 查询用户及购买的商品 -->
    <resultMap id="UserAndItemsResultMap" type="cn.edu.wtu.po.User">
        <!-- 用户信息 -->
        <id column="user_id" property="id"/>
        <result column="username" property="username"/>
        <result column="sex" property="sex"/>
        <result column="address" property="address"/>
        <!-- 订单信息
        一个用户对应多个订单，使用collection映射
         -->
        <collection property="ordersList" ofType="cn.edu.wtu.po.Orders">
            <id column="id" property="id"/>
            <result column="user_id" property="userId"/>
            <result column="number" property="number"/>
            <result column="createtime" property="createtime"/>
            <result column="note" property="note"/>
            <!-- 订单明细
             一个订单包括 多个明细
             -->
            <collection property="orderDetails" ofType="cn.edu.wtu.po.OrderDetail">
                <id column="orderDetail_id" property="id"/>
                <result column="items_id" property="itemsId"/>
                <result column="items_num" property="itemsNum"/>
                <result column="orders_id" property="ordersId"/>
                <!-- 商品信息
                 一个订单明细对应一个商品
                 -->
                <association property="items" javaType="cn.edu.wtu.po.Items">
                    <id column="items_id" property="id"/>
                    <result column="items_name" property="name"/>
                    <result column="items_detail" property="detail"/>
                    <result column="items_price" property="price"/>
                </association>
            </collection>
        </collection>
    </resultMap>
```

## OrderMapper.xml
```xml
<!-- 查询用户及购买的商品信息，使用resultMap -->
    <select id="findUserAndItemsResultMap" resultMap="UserAndItemsResultMap">
        SELECT
        orders.*,
        USER.username,
        USER.sex,
        USER.address,
        orderdetail.id orderdetail_id,
        orderdetail.items_id,
        orderdetail.items_num,
        orderdetail.orders_id,
        items.name items_name,
        items.detail items_detail,
        items.price items_price
        FROM
        orders,
        USER,
        orderdetail,
        items
        WHERE orders.user_id = user.id AND orderdetail.orders_id=orders.id AND orderdetail.items_id = items.id
    </select>
```

## OrderMapper.java
```java
/**
     * 查询用户购买商品信息
     * @return User的列表
     * @throws Exception
     */
    public List<User>  findUserAndItemsResultMap()throws Exception;
```

## 测试
```java
@Test
    public void testFindUserAndItemsResultMap() throws Exception{
        // 创建会话
        SqlSession sqlSession = sqlSessionFactory.openSession();
        // 创建代理对象
        OrderMapper orderMapper = sqlSession.getMapper(OrderMapper.class);

        List<User> lists = orderMapper.findUserAndItemsResultMap();

        for (User list:lists){
            System.out.println(list.toString());
        }
    }
```

## 多对多查询总结

将查询用户购买的商品信息明细清单，（用户名、用户地址、购买商品名称、购买商品时间、购买商品数量）

针对上边的需求就使用resultType将查询到的记录映射到一个扩展的pojo中，很简单实现明细清单的功能。

一对多是多对多的特例，如下需求：

查询用户购买的商品信息，用户和商品的关系是多对多关系。

- 需求1：

查询字段：用户账号、用户名称、用户性别、商品名称、商品价格(最常见)

企业开发中常见明细列表，用户购买商品明细列表，

使用resultType将上边查询列映射到pojo输出。

- 需求2：

查询字段：用户账号、用户名称、购买商品数量、商品明细（鼠标移上显示明细）

使用resultMap将用户购买的商品明细列表映射到user对象中。

总结：

使用resultMap是针对那些对查询结果映射有特殊要求的功能，比如特殊要求映射成list中包括多个list。



## 总结

### resultType
   - 作用：将查询结果按照sql列名pojo属性名一致性映射到pojo中。
   - 场合：常见一些明细记录的展示，比如用户购买商品明细，将关联查询信息全部展示在页面时，此时可直接使用resultType将每一条记录映射到pojo中，在前端页面遍历list（list中是pojo）即可。

### resultMap
	
使用association和collection完成一对一和一对多高级映射（对结果有特殊的映射要求）。

#### association(一对一)

- 作用：将关联查询信息映射到一个pojo对象中。
- 场合：为了方便查询关联信息可以使用association将关联订单信息映射为用户对象的pojo属性中，比如：查询订单及关联用户信息。

使用resultType无法将查询结果映射到pojo对象的pojo属性中，根据对结果集查询遍历的需要选择使用resultType还是resultMap。
	
#### collection(一对多)

- 作用：将关联查询信息映射到一个list集合中。
- 场合：为了方便查询遍历关联信息可以使用collection将关联信息映射到list集合中，比如：查询用户权限范围模块及模块下的菜单，可使用collection将模块映射到模块list中，将菜单列表映射到模块对象的菜单list属性中，这样的作的目的也是方便对查询结果集进行遍历查询。如果使用resultType无法将查询结果映射到list集合中。