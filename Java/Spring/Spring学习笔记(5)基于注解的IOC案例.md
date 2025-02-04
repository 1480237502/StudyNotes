# Spring学习笔记（五）基于注解的IOC案例
---
[TOC]

---

# 1.项目目录结构

![](../../images/基于注解的IOC案例的项目结构.jpg)

# 2.创建账户实体类

- Account

```java
@Component
public class Account {
    @Value("1")
    private Integer id;
    @Value("Tom")
    private String name;
    @Value("34567")
    private Float money;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Float getMoney() {
        return money;
    }

    public void setMoney(Float money) {
        this.money = money;
    }

    @Override
    public String toString() {
        return "Account{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", money=" + money +
                '}';
    }
}
```

# 3.创建接口

- IAccountDao

```java
public interface IAccountDao {

    /**
     * 保存
     */
    void saveAccount();
}
```
- IAccountService
  
```java
public interface IAccountService {
    /**
     * 保存
     */
    void saveAccount();
}
```

# 4.创建实现类

- AccountDaoImpl

```java
@Repository("accountDao")
public class AccountDaoImpl implements IAccountDao {
    @Autowired
    private Account account;
    /**
     * 保存
     *
     */
    @Override
    public void saveAccount() {
        System.out.println("基于注解的spring");
        System.out.println("保存的账户为:"+"Account{" +
                "id=" + account.getId() +
                ", name='" + account.getName() + '\'' +
                ", money=" + account.getMoney() +
                '}');
    }
}
```
- AccountServiceImpl

```java
@Service("accountService")
public class AccountServiceImpl implements IAccountService {
    @Autowired
    @Qualifier("accountDao")
    private IAccountDao accountDao=null;

    /**
     * 保存
     */
    @Override
    public void saveAccount() {
        accountDao.saveAccount();
    }
}
```

# 5.创建测试类

- AccountServiceTest

```java
public class AccountServiceTest {
    ApplicationContext applicationContext;
    @Before
    public void setUp(){
        // 获取配置文件
        applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
    }

    @Test
    public void testAccountService(){
        //获取bean
        IAccountService accountService = applicationContext.getBean("accountService", AccountServiceImpl.class);

        accountService.saveAccount();
    }
}
```
# 6.配置 applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
    <context:component-scan base-package="cn.edu.wtu"></context:component-scan>
</beans>
```
