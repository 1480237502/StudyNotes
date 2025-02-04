# Spring学习笔记（三）依赖注入（Dependency Injection）

---
[TOC]

---

## （一）、概述

1. 能注入的数据：
    - 基本类型和 String
    - 其他 bean 类型（在配置文件中或者注解中配置过的bean）
    - 复杂类型/集合类型
2. IOC的作用：减低程序间的耦合（即依赖关系）

    在当前类需要用到其他类的对象，由 Spring 为我们提供，而我们在配置文件中说明依赖关系的维护，这种方式就称为依赖注入。

## （二）、注入方式

### 1. 操作实例
- 接口如下：
    ~~~java
            public interface IAccountDao {
            
                void saveAccount();
            }

            public interface IAccountService {
            
                /**
                 * 模拟保存账户
                 */
                void saveAccount();
            }
    ~~~

- 实现类：
    ~~~java
            @Service("accountService")
            public class AccountServiceImpl implements IAccountService {
            
                @Autowired
                @Qualifier("accountDao2")
                private IAccountDao accountDao = null;
            
            
                public void  saveAccount() {
                    accountDao.saveAccount();
                }
            
            }

            @Repository("accountDao1")
            public class AccountDaoImpl implements IAccountDao {
            
                public void  saveAccount() {
                    System.out.println("对象创建了111");
                }
            
            }

            @Repository("accountDao2")
            public class IAccountDaoImpl2 implements IAccountDao{
            
                public void  saveAccount() {
                    System.out.println("对象创建了222");
                }
            }
    ~~~

### 2. 第一种：使用注解提供
1. 如何使用？

    第一步：在类或方法的前面加上注解关键字

    第二步：引入约束,注意此处约束多了xmlns:context...

    第三步：添加配置文件，告知 Spring 在创建容器时要扫描的包，配置所需的标签不是在 bean 约束中，而是一个名称为context 的名称孔家和约束中,完整配置如下：
    ~~~xml
        <?xml version="1.0" encoding="UTF-8"?>
        <beans xmlns="http://www.springframework.org/schema/beans"
                xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                xmlns:context="http://www.springframework.org/schema/context"
                xsi:schemaLocation="http://www.springframework.org/schema/beans
                http://www.springframework.org/schema/beans/spring-beans.xsd
                http://www.springframework.org/schema/context
                http://www.springframework.org/schema/context/spring-context.xsd">
        
            <context:component-scan base-package="cn.edu.wtu"></context:component-scan>
        </beans>
    ~~~

    **有哪些注解？**

    1. 用于创建对象的

        作用：等同于 xml 配置文件中编写一个 `<bean>` 标签

        - `@Component`
            - 形式：`@Component(value=" ")/@Component(" ")`
            - 作用：用于把当前类对象存入 Spring 容器中
            - 属性：

                value : 用于指定 `bean` 的` id`，当我们不写的时候，它的默认值是当前类名，且首字母改小写;当值只有一个的时候可以省略

        以下三个注解的作用与 `@Component` 完全一样，它们是 Spring 提供的更明确的划分，使三层对象更加清晰

        - `@Controlle`r  用于表现层
        - `@Service`       用于业务层
        - `@Repository` 用于持久层
    2. 用于注入数据的

        作用：等同于在` <bean> `标签中写一个 `<property>` 标签

        - `@Autowired`
            
            - 作用：自动按照类型注入，只要容器中有唯一的一个 bean 对象类型和要注入的变量类型匹配，  就可以注入成功如果IOC容器中没有任何 bean 的类型和要注入的变量类型匹配，则报错
            - 出现位置：可以是变量上，也可以是方法上，
            - 细节：在使用注解注入时，set 方法就不是必须的了
        - `@Qualifier`
            - 作用：在按照类型注入的基础上再按照名称注入，它在给类成员注入时不能单独使用，但是在给方法参数注入时可以。
        - 属性：
    
            value : 用于指定注入的 bean 的id
    
    - `@Resource`
    
        - 作用：直接按照 bean 的 id 注入，可以直接使用
    
            - 属性：
        name : 用于指定 bean 的 id
    
        - 等同于`@Autowired+@Qualifier`
    
    以上三个注入都只能注入其他 bean 类型的数据，而基本类型和 String 类型的数据无法使用上述注解实现。另外，集合类型的注入只能通过 xml 配置文件实现
    
        - `@Value`
            - 作用：用于注入基本类型和 String 类型的数据
        - 属性：
    
            value : 用于指定数据的值，它可以使用 Spring 中 Spel (即spring的el表达式)
    
            Spel 的写法：${表达式}
    
3. 用于改变范围的
    
作用：等同于在 `<bean>` 标签中使用 `scope `属性
    
    - `@Scope`
        - 作用：用于指定 bean 的作用范围
    - 属性：
    
        value : 指定范围的取值，同 xml 中值，常用为 singleton ,  prototype
    
4. 和生命周期相关（了解）
    
作用：等同于在`<bean>`标签中使用 `init-method `和 `destroy-method`
    
- `@PreDestory`
    
    作用：用于指定销毁方法
    
- `@Postcontrust`
    
        作用：用于指定初始化方法

- 测试类：
    ~~~java
        public static void main(String[] args) {
                //1.获取核心容器对象
                ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
                //2.根据id获取Bean对象
                IAccountService as  = (IAccountService)ac.getBean("accountService");
        
                as.saveAccount();
            }
        }
    ~~~

### 3. 第二种：使用 set 方法提供（更常用的方式）

使用的标签：property

出现的位置：bean 标签的内部

标签的属性：

name : 用于指定注入时所使用的 set 方法

value : 用于提供基本类型和 String 类型的数据

ref : 用于指定其他的bean类型数据，它指的就是在 Spring 容器中出现过的bean对象

优势：创建对象时没有明确的限制，可以直接使用默认构造函数

弊端：如果有某个成员必须有值，是有可能 set 方法没有执行

1. 基本类型和 String 的注入方式
    - 业务层

        ```java
            public class AccountServiceImpl implements IAccountService {
                // 如果时经常变化的数据不适用于依赖注入，此处仅为演示
                private String name;
                private Integer age;
                private Date birthday;

                public void setName(String name) {
                    this.name = name;
                }

                public void setAge(Integer age) {
                    this.age = age;
                }

                public void setBirthday(Date birthday) {
                    this.birthday = birthday;
                }

                public void  saveAccount() {
                    System.out.println("service中的saveaccount()执行了" + name + "," + age + "," +birthday);
                }

            }
        ```

    - 配置bean.xml
        ~~~xml
            <bean id = "accountService" class = "cn.edu.wtu.service.impl.AccountServiceImpl">
                    <property name="name" value ="taylor"></property>
                    <property name="age" value="21"></property>
                    <property name="birthday" ref="now"></property>
                </bean>
            
                <bean id = "now" class = "java.util.Date"></bean>
        ~~~

        测试类同上

2. 复杂集合类型的注入方式
    - 用于给 List 结构集合注入的标签
        - list
        - array
        - set
    - 用于给map结构集合注入的标签
        - map
        - properties

    结构相同，标签可以互换，因此开发中只要记住两组标签即可

    编写实例：
~~~java
        public class AccountServiceImpl implements IAccountService {
        
            private String[] myStrs;
            private List<String> myList;
            private Set<String> mySet;
            private Map<String, String> myMap;
            private Properties myProps;
        
            public void setMyStrs(String[] myStrs) {
                this.myStrs = myStrs;
            }
        
            public void setMyList(List<String> myList) {
                this.myList = myList;
            }
        
            public void setMySet(Set<String> mySet) {
                this.mySet = mySet;
            }
        
            public void setMyMap(Map<String, String> myMap) {
                this.myMap = myMap;
            }
        
            public void setMyProps(Properties myProps) {
                this.myProps = myProps;
            }
        
            public void  saveAccount() {
                System.out.println(Arrays.toString(myStrs));
                System.out.println(myList);
                System.out.println(myMap);
                System.out.println(mySet);
                System.out.println(myProps);
            }
        
        }
~~~

配置如下：

~~~xml
        <bean id = "accountService" class = "com.itheima.service.impl.AccountServiceImpl">
                <!--以下三个标签是等价的，set未列出-->
                <property name="myList">
                    <list>
                        <value>aaa</value>
                        <value>bbb</value>
                    </list>
                </property>
        
                <property name="myStrs">
                    <array>
                        <value>aaa</value>
                        <value>bbbb</value>
                    </array>
                </property>
        
                <property name="mySet">
                    <array>
                        <value>aaa</value>
                        <value>bbbb</value>
                    </array>
                </property>
        
                <!--以下两种方式等价-->
                <property name="myMap">
                    <map>
                        <!--以下两种配置方式都可以-->
                        <entry key="testA" value="aaa"></entry>
                        <entry key="testA">
                            <value>bbb</value>
                        </entry>
                    </map>
                </property>
        
                <property name="myProps">
                    <props>
                        <prop key="testB">bbb</prop>
                    </props>
                </property>
            </bean>
~~~

### 4. 第三种：使用构造函数提供

使用的标签：constructor-arg

标签所在位置：bean 标签的内部

标签中的属性：

- type : 用于指定要注入的数据类型，该类型也是构造函数中某个或某些参数的类型
- index : 用于指定要注入的数据给构造函数中指定索引位置的参数赋值，索引的位置时从0开始
- name(常用) : 用于指定给构造函数中指定名称的参数赋值
- value : 用于提供基本类型和String类型的数据
- ref : 用于指定其他的bean类型数据。它指的就是在spring的IOC核心容器出现过的bean对象

特点：在获取 bean 对象时，注入数据是必须的操作，否则无法操作成功

弊端：改变了 bean 对象的实例化方式，使我们在用不到这些数据的情况下也必须提供带参构造函数，因此开发中较少使用此方法，除非避无可避

例：
~~~java
    public class AccountServiceImpl implements IAccountService {
        // 如果时经常变化的数据不适用于依赖注入，此处仅为演示
        private String name;
        private Integer age;
        private Date birthday;
    
        public AccountServiceImpl(String name, Integer age, Date birthday){
            this.name = name;
            this.age = age;
            this.birthday = birthday;
        }
    
        public void  saveAccount() {
            System.out.println("service中的saveaccount()执行了");
        }
    
    }
~~~

测试类：
~~~java
    public static void main(String[] args) {
            //1.获取核心容器对象
            ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
            //2.根据id获取Bean对象
            IAccountService as  = (IAccountService)ac.getBean("accountService");
            as.saveAccount();
        }
~~~

配置如下：
~~~xml
<bean id = "accountService" class="cn.edu.wtu.service.impl.AccountServiceImpl">
    <constructor-arg name = "name" value="taylor"><constructor-arg>
    <constructor-arg name = "age" value = "23"></constructor-arg>
    <constructor-arg name = "birthday" ref = "now"></constructor-arg>
</bean>
    
<bean id = "now" class = "java.util.Date"></bean>
~~~

如上，AccountDaoImpl1 和 AccountDaoImpl2 实现接口 IAccountDao ，两个类中分别实现了不同的 saveAccount() 方法，AccountServiceImpl 实现接口 IAccountService ，其中调用了 IAccountDao 接口。AccountServiceImpl 通过注解关键字 Autowired 去 Spring 容器中寻找 accountDao ， 再根据 Qualifier 配置的 value 找到两个 dao 的实现类中与之相匹配的 Repository 的值。
