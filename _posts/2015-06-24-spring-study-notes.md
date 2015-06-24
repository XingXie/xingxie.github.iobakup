---
layout: post
title:  "Spring Study Notes"
date:   2015-06-24
categories: Spring
excerpt: Spring Notes
comments: true
---

* content
{:toc}

## [lazy-init for singleton and prototype](http://www.intertech.com/Blog/singleton-beans-lazy-and-not-so-lazy/)

They are all about bean
eager init is only for singleton

if you want to apply to prototype

1. a singleton wrap with prototype instances

2. use multiple singleton bean

For destroy method, only context.close call then destroy method is called.

Use the properties in bean

method 1: 

~~~ xml
<context:property-placeholder location="classpath:helloworld2.properties" />
~~~

method 2:

~~~ xml
<bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
 <property name="location">
  <value>helloworld2.properties </value>
  </property>
   </bean>
~~~

You can use multiple input in first method, using ,but save the classpath keyword.

include multiple properties: use locations instead of location: get and set methods must include for adding @Value("${name}") to primitives 
(http://springcert.sourceforge.net/study-notes-3.html)

Spring is a container (Beans.xml). It contains beans (pojo classes) which can be initiated (getbean) by others.  

~~~ xml
<bean id="beanTeamplate" abstract="true"> 
<bean id="helloIndia" class="com.tutorialspoint.HelloIndia" parent="beanTeamplate">
~~~

To inheritate the properties from the teamplate.

## Basics

the Beans.xml file is under the src directory
~~~ xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-
3.0.xsd">
<bean id="helloWorld" class="com.tutorialspoint.HelloWorld">
<property name="message" value="Hello World!"/>
</bean>
</beans>
~~~ 

~~~ java
// you can get the object via the following code
ApplicationContext context = new ClassPathXmlApplicationContext("Beans.xml");
HelloWorld obj = (HelloWorld) context.getBean("helloWorld");
obj.getMessage();

// or you can use the object as a file
ApplicationContext context = new FileSystemXmlApplicationContext
             ("C:/Users/ZARA/workspace/HelloSpring/src/Beans.xml");
~~~

Scope

1. singleton: ￼￼This scopes the bean definition to a single instance per Spring IoC container (default).
￼
2. prototype:￼￼ This scopes a single bean definition to have any number of object instances.
￼
3. request, session: ￼This scopes a bean definition to an HTTP request. Only valid in the context of a web-aware Spring ApplicationContext.
￼
4. global-session: ￼This scopes a bean definition to a global HTTP session. Only valid in the context of a web-aware Spring ApplicationContext.        

An ApplicationContext automatically detects any beans that are defined

With implementation of theBeanPostProcessor interface and registers these

Beans as post-processors, to be then called appropriately by the container upon bean creation.

~~~ java
package com.tutorialspoint;
import org.springframework.beans.factory.config.BeanPostProcessor;
import org.springframework.beans.BeansException;
public class InitHelloWorld implements BeanPostProcessor {
   public Object postProcessBeforeInitialization(Object bean,
                 String beanName) throws BeansException {
      System.out.println("BeforeInitialization : " + beanName);
      return bean; // you can return any other object as well }

   public Object postProcessAfterInitialization(Object bean,
                 String beanName) throws BeansException {
      System.out.println("AfterInitialization : " + beanName);
      return bean; // you can return any other object as well
}
}

public class MainApp {
 public static void main(String[] args) {
  AbstractApplicationContext context = new ClassPathXmlApplicationContext("Beans.xml");
  HelloWorld obj = (HelloWorld) context.getBean("helloWorld");
  obj.setMessage("i am obj");
  obj.getMessage();
  context.registerShutdownHook();

 }

}
~~~ 

context.registerShutdownHook for triggering the destroy method. 

In Beans.xml configuration, must have
~~~ xml
<bean class="com.tutorialspoint.InitHelloWorld" /> </beans>
~~~

A child bean definition inherits configuration data from a parent definition. The child
definition can override some values, or add others, as needed. Spring
Bean definition inheritance has nothing to do with Java class inheritance but inheritance
concept is same. You can define a parent bean definition as a template and other child
beans can inherit required configuration from the parent bean.

You can create a Bean definition template which can be used by other child bean definitions
without putting much effort. While defining a Bean Definition Template, you should not
specify class attribute and should specify abstract attribute with a value of true as shown below:

~~~ xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-
3.0.xsd">
<bean id="beanTeamplate" abstract="true">
<property name="message1" value="Hello World!"/>
<property name="message2" value="Hello Second World!"/>
<property name="message3" value="Namaste India!"/>
</bean>
<bean id="helloIndia" class="com.tutorialspoint.HelloIndia" parent="beanTeamplate">
<property name="message1" value="Hello India!"/>
<property name="message3" value="Namaste India!"/>
   </bean>
</beans>
~~~

## Constructor-based Dependency Injection￼
~~~ xml
<bean id="textEditor" class="com.tutorialspoint.TextEditor">
<constructor-arg ref="spellChecker"/>
</bean>
   <!-- Definition for spellChecker bean -->
<bean id="spellChecker" class="com.tutorialspoint.SpellChecker">
</bean>
~~~

## DI for TextEditor with SpellChecker
~~~ java
￼public class TextEditor {
   private SpellChecker spellChecker;
   // in this case, you will search the beans.xml to identify the correct one
   public TextEditor(SpellChecker spellChecker) {
      System.out.println("Inside TextEditor constructor." );
      this.spellChecker = spellChecker;
   }
   public void spellCheck() {
    spellChecker.checkSpelling();
    }
~~~

~~~ xml
// Beans.xml  
<!-- Definition for textEditor bean -->
<bean id="textEditor" class="com.tutorialspoint.TextEditor"> <constructor-arg ref="spellChecker"/>
</bean>
   <!-- Definition for spellChecker bean -->
<bean id="spellChecker" class="com.tutorialspoint.SpellChecker"> </bean>
~~~

~~~ java
// how to use it. we don't need to initialize the spellchecker in java code
public class MainApp {
   public static void main(String[] args) {
      ApplicationContext context =
             new ClassPathXmlApplicationContext("Beans.xml");
      TextEditor te = (TextEditor) context.getBean("textEditor");
      te.spellCheck();
}

// more than one constructor ￼
package x.y;
public class Foo {
   public Foo(Bar bar, Baz baz) {
// ...
} }
~~~

~~~ xml￼
<beans>
 <bean id="foo" class="x.y.Foo">
 <constructor-arg ref="bar"/>
 <constructor-arg ref="baz"/> </bean>
 <bean id="bar" class="x.y.Bar"/>
 <bean id="baz" class="x.y.Baz"/>
</beans>￼
~~~

~~~ java 
// arguments are primitive types
package x.y;
public class Foo {
   public Foo(int year, String name) {

} }
~~~

~~~ xml
<beans>
<bean id="exampleBean" class="examples.ExampleBean">
<constructor-arg type="int" value="2001"/>
<constructor-arg type="java.lang.String" value="Zara"/>
</bean>
</beans>
~~~

￼Finally and the best way to pass constructor argument
~~~ xml
<beans>
<bean id="exampleBean" class="examples.ExampleBean">
<constructor-arg index="0" value="2001"/> <constructor-arg index="1" value="Zara"/>
</bean>
</beans>
~~~


setter-based injection, different from constructor-based: <property + ref> vs <constructor-based + ref>, object should user ref attr, value uses value.
~~~ xml
<?xml version="1.0" encoding="UTF-8"?>
 <beans xmlns="http://www.springframework.org/schema/beans"
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.springframework.org/schema/beans   http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">
   <!-- Definition for textEditor bean -->
<bean id="textEditor" class="com.tutorialspoint.TextEditor">
<property name="spellChecker" ref="spellChecker"/>
 </bean>
   <!-- Definition for spellChecker bean -->
 <bean id="spellChecker" class="com.tutorialspoint.SpellChecker"> </bean>
</beans>
~~~

XML p-namespace, spouse-ref: refer to another bean. Notice that value always has ""
~~~ xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">
<bean id="john-classic" class="com.example.Person" p:name="John Doe"
p:spouse-ref="jane"/>
</bean>
<bean name="jane" class="com.example.Person" p:name="John Doe"/>
   </bean>
</beans>
~~~

## Injecting inner beans

~~~ xml
// a <bean/> element inside the <property/> or <constructor-arg/> elements is called inner bean and it is shown below.

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">
   <!-- Definition for textEditor bean using inner bean -->
   <bean id="textEditor" class="com.tutorialspoint.TextEditor">
      <property name="spellChecker">
       <bean id="spellChecker" class="com.tutorialspoint.SpellChecker"/>
      </property>
   </bean>
</beans>
~~~

## Injecting collection / references/ null/empty value
~~~ xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">
   <!-- Definition for javaCollection -->
<bean id="javaCollection" class="com.tutorialspoint.JavaCollection">
      <!-- results in a setAddressList(java.util.List) call -->
 <property name="addressList">
    <list>
     <value>INDIA</value> <value>Pakistan</value> <value>USA</value> <value>USA</value>
       </list>
 </property>
  <!-- Passing bean reference  for java.util.Set -->
 <property name="addressSet">
   <set>
   <ref bean="address1"/> <ref bean="address2"/> <value>Pakistan</value>
   </set>
 </property>

<bean id="..." class="exampleBean">
 <property name="email" value=""/>
</bean>
~~~

￼￼￼￼￼￼￼￼The preceding example is equivalent to the Java code: exampleBean.setEmail(""). If you need to pass an NULL value then you can pass it as follows:
~~~ xml
￼￼￼￼￼￼<bean id="..." class="exampleBean">
 <property name="email"><null/></property>
</bean>
~~~
￼
## Autowiring
~~~ xml
<!-- by name -->
<!-- origin: Definition for textEditor bean -->
   <bean id="textEditor" class="com.tutorialspoint.TextEditor">
      <property name="spellChecker" ref="spellChecker" />
      <property name="name" value="Generic Text Editor" />
  </bean>
   <!-- Definition for spellChecker bean -->
   <bean id="spellChecker" class="com.tutorialspoint.SpellChecker">
   </bean>
</beans>

   <!-- byName: Definition for textEditor bean -->
   <bean id="textEditor" class="com.tutorialspoint.TextEditor"
      autowire="byName">
      <property name="name" value="Generic Text Editor" />
</bean>
~~~ xml

Spring Autowiring 'byType'. Spring container looks at the beans on which autowireattribute is set to byType in the XML configuration file. It then tries to match and wire a property if its typematches with exactly one of the beans name in configuration file.  

~~~ java
// by constructor: This mode is very similar to byType, but it applies to constructor arguments.
public TextEditor( SpellChecker spellChecker, String name ) {
      this.spellChecker = spellChecker;
      this.name = name;
   }
~~~

~~~ xml
// beans.xml
<!-- Definition for textEditor bean -->
<bean id="textEditor" class="com.tutorialspoint.TextEditor" autowire="constructor">
<constructor-arg value="Generic Text Editor"/>
</bean>
   <!-- Definition for spellChecker bean -->
<bean id="SpellChecker" class="com.tutorialspoint.SpellChecker">
</bean>
~~~

## Anotation configuration
~~~ java
// @required : beans.xml must specify the name/value

import org.springframework.beans.factory.annotation.Required;
 public class Student {
    private Integer age;
    private String name;

  @Required
    public void setAge(Integer age) {
       this.age = age;
    }
 
    public Integer getAge() {
return age; }

  @Required
    public void setName(String name) {
       this.name = name;
    }
 
    public String getName() {
   return name;
  }}
~~~
// @Autowired  
You can use @Autowired annotation on properties to get rid of the setter methods. When you will pass values of autowired properties using <property> Spring will automatically assign those properties with the passed values or references.
￼The @Required annotation applies to bean property setter methods and it indicates that the affected bean property must be populated in XML configuration file at configuration time otherwise the container throws a BeanInitializationException exception.
~~~ java
// no setter method!
import org.springframework.beans.factory.annotation.Autowired;
public class TextEditor {
  @Autowired
   private SpellChecker spellChecker;
 
  public TextEditor() {
   System.out.println("Inside TextEditor constructor." );
   }
   public SpellChecker getSpellChecker( ){
      return spellChecker;
   }
   public void spellCheck(){
      spellChecker.checkSpelling();
} }
~~~

By default, the @Autowired annotation implies the dependency is required similar to @Required annotation, however, you can turn off the default behavior by using (required=false) option with @Autowired.
The following example will work even if you do not pass any value for age property but still it will
demand for name property.

￼There may be a situation when you create more than one bean of the same type and want to wire only one of them with a property, in such case you can use @Qualifier annotation along with @Autowired to remove the confusion by specifying which exact bean will be wired. Below is an example to show the use of @Qualifier annotation.￼

~~~ java
public class Profile {
 @Autowired
 @Qualifier("student1")
 private Student student;
}
~~~

~~~ xml
<!-- Definition for profile bean -->
<bean id="profile" class="com.tutorialspoint.Profile"> </bean>
   <!-- Definition for student1 bean -->
<bean id="student1" class="com.tutorialspoint.Student">
<property name="name" value="Zara" />
<property name="age" value="11"/> </bean>
   <!-- Definition for student2 bean -->
<bean id="student2" class="com.tutorialspoint.Student"> <property name="name" value="Nuha" />
<property name="age" value="2"/>
   </bean>
   ~~~
Output will be using the student1 value for name and age


Annotating a class with the @Configuration indicates that the class can be used by the Spring IoC container as a source of bean definitions. The @Bean annotation tells Spring that a method annotated with @Bean will return an object that should be registered as a bean in the Spring application context. The simplest possible @Configuration class would be:

~~~ java
import org.springframework.context.annotation.*;
@Configuration
public class HelloWorldConfig {
@Bean
   public HelloWorld helloWorld(){
      return new HelloWorld();
} }
// used in main method; must specify the container￼￼￼￼￼￼￼￼￼￼￼￼￼￼￼￼￼￼￼￼￼￼￼￼￼￼
public static void main(String[] args) {
   ApplicationContext ctx =
   new AnnotationConfigApplicationContext(HelloWorldConfig.class);
   HelloWorld helloWorld = ctx.getBean(HelloWorld.class);
   helloWorld.setMessage("Hello World!");
   helloWorld.getMessage();
}￼￼

// You can load (via register) various configuration classes as follows:
 public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx =
      new AnnotationConfigApplicationContext();
   ctx.register(AppConfig.class, OtherConfig.class);
     ctx.register(AdditionalConfig.class);
     ctx.refresh(); // very important
 
     MyService myService = ctx.getBean(MyService.class);
     myService.doStuff();
}
//The @Import annotation allows for loading @Bean definitions from another configuration class.
@Configuration
@Import(ConfigA.class)
public class ConfigB {
@Bean
   public B a() {
      return new A();
}
}

public static void main(String[] args) {
   ApplicationContext ctx =
   new AnnotationConfigApplicationContext(ConfigB.class);
   // now both beans A and B will be available...
   A a = ctx.getBean(A.class);
   B b = ctx.getBean(B.class);
}

// ￼The @Bean annotation supports specifying arbitrary initialization and destruction callback methods, much like Spring XML's init-method and destroy-method attributes on the bean element:

public class Foo {
   public void init() {
      // initialization logic
   }
   public void cleanup() {
      // destruction logic
} }

@Configuration
public class AppConfig {
   @Bean(initMethod = "init", destroyMethod = "cleanup" )
public Foo foo() {
      return new Foo();
} }

// you can use @scope to change default singleton
~~~

## Event handling
~~~ java
// CStartEventHandler.java
 package com.tutorialspoint;
 import org.springframework.context.ApplicationListener;
 import org.springframework.context.event.ContextStartedEvent;
 public class CStartEventHandler
    implements ApplicationListener<ContextStartedEvent>{
    public void onApplicationEvent(ContextStartedEvent event) {
       System.out.println("ContextStartedEvent Received");
} }

//MainApp.java
package com.tutorialspoint;
import org.springframework.context.ConfigurableApplicationContext;
import
org.springframework.context.support.ClassPathXmlApplicationContext;
public class MainApp {
   public static void main(String[] args) {
      ConfigurableApplicationContext context =
      new ClassPathXmlApplicationContext("Beans.xml");
     // Let us raise a start event.
      context.start();
      HelloWorld obj = (HelloWorld) context.getBean("helloWorld");
      obj.getMessage();
   }
}
~~~


beans.xml piece
~~~ xml
<bean id="cStartEventHandler"
 class="com.tutorialspoint.CStartEventHandler"/>
 ~~~

## JDBC Spring

Spring JDBC Framework takes care of all the low-level details starting from opening the connection, prepare and execute the SQL statement, process exceptions, handle transactions and finally close the connection.

The JdbcTemplate class executes SQL queries, update statements and stored procedure calls, performs iteration over ResultSets and extraction of returned parameter values. It also catches JDBC exceptions and translates them to the generic, more informative, exception hierarchy defined in the org.springframework.dao package.
~~~ xml
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSour ce">
<property name="driverClassName" value="com.mysql.jdbc.Driver"/>
<property name="url" value="jdbc:mysql://localhost:3306/TEST"/> <property name="username" value="root"/>
<property name="password" value="password"/>
</bean>
~~~
DAO stands for data access object which is commonly used for database interaction. DAOs exist to provide a means to read and write data to the database and they should expose this functionality through an interface by which the rest of the application will access them.
DAO support in Spring works with data access technologies like JDBC, Hibernate, JPA or JDO in a consistent way.
You can use the execute(..) method from jdbcTemplate to execute any SQL statements or DDL statements.
Now we need to supply a DataSource to the JdbcTemplate so it can configure itself to get database access. You can   configure the DataSource in the XML file with a piece of code as shown below:
~~~ xml
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSour ce">
 <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
 <property name="url" value="jdbc:mysql://localhost:3306/TEST"/> <property name="username" value="root"/>
 <property name="password" value="password"/>
</bean>
~~~

//Inserting a row into the table:
~~~ java
 String SQL = "insert into Student (name, age) values (?, ?)";
 jdbcTemplateObject.update( SQL, new Object[]{"Zara", 11} );
~~~

// Beans.xml
~~~ xml
   <!-- Initialization for data source -->
   <bean id="dataSource"
class="org.springframework.jdbc.datasource.DriverManagerDataSource">
<property name="driverClassName" value="com.mysql.jdbc.Driver"/>
      <property name="url" value="jdbc:mysql://localhost:3306/TEST"/>
      <property name="username" value="root"/>
      <property name="password" value="password"/>
</bean>
   <!-- Definition for studentJDBCTemplate bean -->
   <bean id="studentJDBCTemplate"
      class="com.tutorialspoint.StudentJDBCTemplate">
      <property name="dataSource"  ref="dataSource" />
   </bean>
~~~

Stored procedure calls
The SimpleJdbcCall class can be used to call a stored procedure with IN and OUT parameters.

~~~ mysql
// MYSQL procedure
DELIMITER $$
DROP PROCEDURE IF EXISTS `TEST`.`getRecord` $$
CREATE PROCEDURE `TEST`.`getRecord` (
IN in_id INTEGER,
OUT out_name VARCHAR(20),
OUT out_age  INTEGER)
BEGIN
   SELECT name, age
   INTO out_name, out_age
   FROM Student where id = in_id;
END $$
DELIMITER ;
~~~

The TransactionDefinition is the core interface of the transaction support in Spring and it is defined as below:
~~~ java
public interface TransactionDefinition {
   int getPropagationBehavior();
   int getIsolationLevel();
   String getName();
   int getTimeout();
   boolean isReadOnly();
}

// The TransactionStatus interface provides a simple way for transactional code to control transaction execution and query transaction status.
 public interface TransactionStatus extends SavepointManager {
    boolean isNewTransaction();
    boolean hasSavepoint();
    void setRollbackOnly();
    boolean isRollbackOnly();
    boolean isCompleted();
 }
~~~

Programmatic transaction management approach allows you to manage the transaction with the help of programming in your source code. That gives you extreme flexibility, but it is difficult to maintain.

Declarative transaction management approach allows you to manage the transaction with the help of configuration instead of hard coding in your source code. This means that you can separate transaction management from the business code. You only use annotations or XML based configuration to manage the transactions. The bean configuration will specify the methods to be transactional. Here are the steps associated with declarative transaction:
* We use <tx:advice /> tag, which creates a transaction-handling advice and same time we define a pointcut that matches all methods we wish to make transactional and reference the transactional advice.
* If a method name has been included in the transactional configuration then created advice will begin the transaction before calling the method.
* Target method will be executed in a try / catch block.
* If the method finishes normally, the AOP advice commits the transaction successfully
otherwise it performs a rollback.

1. DispatcherServlet consults the HandlerMapping to call the appropriate Controller.￼
2. The Controller takes the request and calls the appropriate service methods based on used GET or POST method. The service method will set model data based on defined business logic and returns view name to the DispatcherServlet.
3. The DispatcherServlet will take help from ViewResolver to pickup the defined view for the request.
4. Once view is finalized, The DispatcherServlet passes the model data to the view which is finally rendered on the browser.

DispatcherServlet delegates the request to the controllers to execute the functionality specific to it. The@Controller annotation indicates that a particular class serves the role of a controller. The@RequestMapping annotation is used to map a URL to either an entire class or a particular handler method.
