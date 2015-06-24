---
layout: post
title:  "Hibernate Study Notes"
date:   2015-06-24
categories: Hibernate
excerpt: study notes
comments: true
---

* content
{:toc}


## when you want sql query, don't miss add entity

~~~ java
(List<WingsApiStatementlog>) factory.getCurrentSession().createSQLQuery(query)
     .addEntity(WingsApiStatementlog.class).setMaxResults(daoPageSize).list();
~~~

## remove hibernate set of objects

~~~ java
HibernateTemplate template = new HibernateTemplate(factory);
  template.deleteAll(list);
~~~

## special serialization for object in hibernate: implement jsonserializer

use 

~~~ java
@JsonSerialize(using = StatementLogSerializer.class) in getting method of Entity

{
     label: Subject.getId()
     value: Subject.getName()
}
~~~

## Internal class

~~~java 
public class SubjectAutocompleteSerializer extends JsonSerializer<Subject> {

    @Override
    public void serialize(Subject value, JsonGenerator jgen, SerializerProvider provider)
            throws IOException, JsonProcessingException {
        jgen.writeStartObject();
        jgen.writeStringField("label", value.getId().toString());
        jgen.writeStringField("value", value.getName());
        jgen.writeEndObject();
    }

}
~~~

[Learning Source](http://rambalajis.blogspot.com/2013/01/hibernate-introduction.html)


## Basics

Hibernate provides us the following advantages:

1. Direct mapping between the table and the java object.

2. Through hibernate, a database table can be represented as a Java object and provides a flexibility of designing the database table through java object.

3. It reduces the developer's responsibility of writing SQL queries(but not to the extent) and database implementations.

4. Provides a layer for database implementation and other persisting activities.

The only reason for the above advantages , Hibernate is a framework which has got it own API for

1. Connecting to database and will take the responsibility of mapping database table to java objects.

2. Handling Transactions and optimization techniques

3. performing CRUD operations on database tables.

### Hibernate Approach:

* Top-Down approach:

In this approach, we will create the database structure /table and then create the hbm files and hibernate-cfg.xml, finally generate the java objects. This is the suggested approach by hibernate.

* Bottom-up approach:

If you are good at Ant and XDoclet, first create the domain (java) objects (need bit knowledge in Domain Modelling)  annotated with hibernate tags (xdoclet will be useful to generate the hibernate annotations) and then generate the hibernate-cfg.xml , hbm files ,schema ddl files using xdoclet task in ant.

* How Hibernate works:

Hibernate is purely driven by the xml config file (hibernate-cfg.xml) which will have the connection details ,the mapping java object names, connection pooling details and transaction details.

During startup, the application will create a SessionFactory by providing the cfg.xml file. Hibernate API will create a session(is one per application) from the SessionFactory. The session object will wraps a JDBC connection and will act as a interface between the java objects and database tables and this session will be closed if the Hibernate uses runtime reflection to load all the data objects and will create proxy classes.

For retrieving records from the database, you have to issue a load command through hibernate API.Hibernate framework will generate a hql and then it will converted into database native query. The conversion will be taken care by Hibernate API.

Hibernate can fit with two different architecture.

1. Two tier architecture (Web -> JSP/Servlet -> Persistent Object -> Hibernate ->XML mappings -> Database

2. Three tier architecture

* Hibernate Advantage:

Hibernate is good tool / framework for providing an object relational mapping and it is more flexible for the developers.

* Disadvantage:

Hibernate doesn't have a good connection pooling mechanism or transaction handling mechanism.
That's the good reason to go for other tools like apache connection management like DBCP and JTA for transactions.

The Object-Relational Mapping (ORM) is the solution to handle the difference between OO and RDBMS

Hibernate maps Java classes to database tables and from Java data types to SQL data types and relieve the developer from 95% of common data persistence related programming tasks.

## hibernate architechture

1. Configuration Object is the first Hibernate object you create in any Hibernate application and usually created only once during application initialization. It represents a configuration or properties file required by the Hibernate. The Configuration object provides two keys components:

* Database Connection: This is handled through one or more configuration files supported by Hibernate.
These files are hibernate.properties and hibernate.cfg.xml.

* Class Mapping Setup: This component creates the connection between the Java classes and database tables.

2. SessionFactory Object

￼Configuration object is used to create a SessionFactory object which inturn configures Hibernate for the application using the supplied configuration file and allows for a Session object to be instantiated. The SessionFactory is a thread safe object and used by all the threads of an application.
￼
￼The SessionFactory is heavyweight object so usually it is created during application start up and kept for later use. You would need one SessionFactory object per database using a separate configuration file. So if you are using multiple databases then you would have to create multiple SessionFactory objects.

3. A Session Object is used to get a physical connection with a database. The Session object is lightweight and designed to be instantiated each time an interaction is needed with the database. Persistent objects are saved and retrieved through a Session object. The session objects should not be kept open for a long time because they are not usually thread safe and they should be created and destroyed them as needed.

4. A Transaction Object represents a unit of work with the database and most of the RDBMS supports transaction functionality. Transactions in Hibernate are handled by an underlying transaction manager and transaction (from JDBC or JTA).

This is an optional object and Hibernate applications may choose not to use this interface, instead managing transactions in their own application code.

5. Query Object use SQL for Hibernate Qeury Language (HQL) string to retrieve data from DB and create objects.

6. Critical object: create and execute object oriented criteria queries to retrieve objects.

Must know in advance where to find the mapping information that defines how your Java
classes relate to the database tables. Hibernate also requires a set of configuration settings related to database and other related parameters. All such information is usually supplied as a standard Java properties file called hibernate.properties, or as an XML file named hibernate.cfg.xml.

## Hibernate Configuration

~~~ xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE hibernate-configuration SYSTEM
"http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">

<hibernate-configuration>
   <session-factory>
   <property name="hibernate.dialect">  // language
      org.hibernate.dialect.MySQLDialect
   </property>
   <property name="hibernate.connection.driver_class"> //drive
      com.mysql.jdbc.Driver
</property>
   <!-- Assume test is the database name --> // dbname
   <property name="hibernate.connection.url">
      jdbc:mysql://localhost/test
   </property>
   <property name="hibernate.connection.username"> // user name
root
</property>
<property name="hibernate.connection.password"> // pwd
root123
</property>
<!-- List of XML mapping files -->
<mapping resource="Employee.hbm.xml"/>
</session-factory>
</hibernate-configuration>
~~~

##  Seesions

The Session is to offer create, read and delete operations for instances of mapped entity classes. Instances may exist in one of the following three states at a given point in time:

1. transient: A new instance of a a persistent class which is not associated with a Session and has no representation in the database and no identifier value is considered transient by Hibernate.

2. persistent: You can make a transient instance persistent by associating it with a Session. A persistent instance has a representation in the database, an identifier value and is associated with a Session.

3. detached: Once we close the Hibernate Session, the persistent instance will become a detached instance.

A Session instance is serializable if its persistent classes are serializable. A typical transaction should use the following idiom:
￼
~~~ java
Session session = factory.openSession();
Transaction tx = null;
try {
   tx = session.beginTransaction();
   // do some work
   ...
tx.commit();
}
catch (Exception e) {
   if (tx!=null) tx.rollback();
   e.printStackTrace();
}finally {
   session.close();
}
~~~

If the Session throws an exception, the transaction must be rolled back and the session must be discarded.

## Persistant class

The entire concept of Hibernate is to take the values from Java class attributes and persist them to a database table. A mapping document helps Hibernate in determining how to pull the values from the classes and
map them with table and associated fields.

1. All Java classes that will be persisted need a default constructor.

2. All classes should contain an ID in order to allow easy identification of your objects within Hibernate and
the database. This property maps to the primary key column of a database table.

3. All attributes that will be persisted should be declared private and have getXXX and setXXXmethods defined in the JavaBean style.

4. A central feature of Hibernate, proxies, depends upon the persistent class being either non-final, or the implementation of an interface that declares all public methods.

5. All classes that do not extend or implement some specialized classes and interfaces required by the EJB framework.

## mapping

Object/relational mappings are usually defined in an XML document. This mapping file instructs Hibernate how to map the defined class or classes to the database tables.

~~~ xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE hibernate-mapping PUBLIC
"-//Hibernate/Hibernate Mapping DTD//EN" "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
<hibernate-mapping>
<class name="Employee" table="EMPLOYEE">
  <meta attribute="class-description">
    This class contains the employee detail.
  </meta>
  <id name="id" type="int" column="id">
         <generator class="native"/>
  </id>
  <property name="firstName" column="first_name" type="string"/>
  <property name="lastName" column="last_name" type="string"/>
  <property name="salary" column="salary" type="int"/>
</class>
</hibernate-mapping>
~~~

You should save the mapping document in a file with the format <classname>.hbm.xml. We saved our mapping document in the file Employee.hbm.xml.

##  R/O Mapping

~~~ xml
<hibernate-mapping>
<class name="Employee" table="EMPLOYEE">
<meta attribute="class-description">
This class contains the employee detail.
</meta>
<id name="id" type="int" column="id">
         <generator class="native"/>
</id>
<set name="certificates" cascade="all">
<key column="employee_id"/>
<one-to-many class="Certificate"/>
</set>
<property name="firstName" column="first_name" type="string"/>
<property name="lastName" column="last_name" type="string"/>
<property name="salary" column="salary" type="int"/>
</class>

<class name="Certificate" table="CERTIFICATE">
<meta attribute="class-description">
This class contains the certificate records.
</meta>
<id name="id" type="int" column="id">
<generator class="native"/>
</id>
<property name="name" column="certificate_name" type="string"/>
</class>

</hibernate-mapping>
~~~

// sortedset

~~~ xml
<set name="certificates" cascade="all" sort="MyClass">
<key column="employee_id"/>
<one-to-many class="Certificate"/>
</set>
~~~

// list

~~~ xml
<list name="certificates" cascade="all">
<key column="employee_id"/>
<list-index column="idx"/>
<one-to-many class="Certificate"/>
</list>
~~~

// bag

~~~ xml
<bag name="certificates" cascade="all">
<key column="employee_id"/>
<one-to-many class="Certificate"/>
</bag>
~~~

//map

~~~ xml
<map name="certificates" cascade="all">
<key column="employee_id"/>
<index column="certificate_type" type="string"/>
<one-to-many class="Certificate"/>
</map>
~~~

## Annotation

Hibernate annotations is the newest way to define mappings without a use of
XML file. You can use annotations in addition to or as a replacement of XML mapping metadata.
Hibernate Annotations is the powerful way to provide the metadata for the Object and Relational Table mapping. All the metadata is clubbed into the POJO java file along with the code this helps the user to understand the table structure and POJO simultaneously during the development.

~~~ java
import javax.persistence.*;
@Entity
@Table(name = "EMPLOYEE")
public class Employee {
   @Id @GeneratedValue
   @Column(name = "id")
   private int id;
   @Column(name = "first_name")
   private String firstName;
   @Column(name = "last_name")
private String lastName;
   @Column(name = "salary")
   private int salary;
   public Employee() {}
   public int getId() {
return id; }
   public void setId( int id ) {
      this.id = id;
   }
   public String getFirstName() {
      return firstName;
   }
   public void setFirstName( String first_name ) {
      this.firstName = first_name;
   }
   public String getLastName() {
      return lastName;
   }
   public void setLastName( String last_name ) {
      this.lastName = last_name;
   }
   public int getSalary() {
      return salary;
   }
   public void setSalary( int salary ) {
      this.salary = salary;
} }
~~~

* Hibernate detects that the @Id annotation is on a field and assumes that it should access properties on an object
directly through fields at runtime. If you placed the @Id annotation on the getId() method, you would enable access to properties through getter and setter methods by default. Hence, all other annotations are also placed on either fields or getter methods, following the selected strategy. Following section will explain the annotations used in the above class.

* @Entity Annotation
The EJB 3 standard annotations are contained in the javax.persistence package, so we import this package as the first step. Second we used the @Entity annotation to the Employee class which marks this class as an entity bean, so it must have a no-argument constructor that is visible with at least protected scope.

* ￼@Table Annotation
The @Table annotation allows you to specify the details of the table that will be used to persist the entity in the database. The @Table annotation provides four attributes, allowing you to override the name of the table, its catalogue, and its schema, and enforce unique constraints on columns in the table. For now we are using just table name which is EMPLOYEE.

* @Id and @GeneratedValue Annotations
Each entity bean will have a primary key, which you annotate on the class with the @Id annotation. The primary key can be a single field or a combination of multiple fields depending on your table structure.
By default, the @Id annotation will automatically determine the most appropriate primary key generation strategy to be used but you can override this by applying the@GeneratedValueannotation which takes two parameters strategy and generator let us use only default the default key generation strategy. Letting Hibernate determine which generator type to use makes your code portable between different databases.

* @Column Annotation
The @Column annotation is used to specify the details of the column to which a field or property will be mapped. You can use column annotation with the following most commonly used attributes:
* name attribute permits the name of the column to be explicitly specified.
* length attribute permits the size of the column used to map a value particularly for a String value.
* nullable attribute permits the column to be marked NOT NULL when the schema is generated.
* unique attribute permits the column to be marked as containing only unique values.

~~~ java
// how to use
factory = new AnnotationConfiguration().
configure().
//addPackage("com.xyz")
//add package if used.
addAnnotatedClass(Employee.class). buildSessionFactory();
~~~

## Criteria queries

The Hibernate Session interface provides createCriteria() method which can be used to create aCriteria object that returns instances of the persistence object's class when your application executes a criteria query.
simplest example of a criteria query is one which will simply return every object that corresponds to the Employee class.
￼
~~~ java
Criteria cr = session.createCriteria(Employee.class);
List results = cr.list();

//You can use add() method available for Criteria object to add restriction for a criteria query.
Criteria cr = session.createCriteria(Employee.class);
// To get records having salary more than 2000
cr.add(Restrictions.gt("salary", 2000));
// To get records having salary less than 2000
cr.add(Restrictions.lt("salary", 2000));
// To get records having fistName starting with zara
cr.add(Restrictions.like("firstName", "zara%"));
// Case sensitive form of the above restriction.
cr.add(Restrictions.ilike("firstName", "zara%"));
// To get records having salary in between 1000 and 2000
cr.add(Restrictions.between("salary", 1000, 2000));
// To check if the given property is null
cr.add(Restrictions.isNull("salary"));
// To check if the given property is not null
cr.add(Restrictions.isNotNull("salary"));
// To check if the given property is empty
cr.add(Restrictions.isEmpty("salary"));
// To check if the given property is not empty
cr.add(Restrictions.isNotEmpty("salary"));

// use AND and OR
Criteria cr = session.createCriteria(Employee.class);
Criterion salary = Restrictions.gt("salary", 2000);
Criterion name = Restrictions.ilike("firstNname","zara%");
// To get records matching with OR condistions
LogicalExpression orExp = Restrictions.or(salary, name);
cr.add( orExp );
// To get records matching with AND condistions
LogicalExpression andExp = Restrictions.and(salary, name);
cr.add( andExp );
List results = cr.list();

// Pagination using Criteria
There are two methods of the Criteria interface for pagination. We can construct a paging component in our web or Swing application. Following is the example which you can extend to fetch 10 rows at a time:
Criteria cr = session.createCriteria(Employee.class);
cr.setFirstResult(1);
cr.setMaxResults(10);
List results = cr.list();

// sort the order
Criteria cr = session.createCriteria(Employee.class);
// To get records having salary more than 2000
cr.add(Restrictions.gt("salary", 2000));
// To sort records in descening order
crit.addOrder(Order.desc("salary"));
// To sort records in ascending order
crit.addOrder(Order.asc("salary"));
List results = cr.list();

// projections and aggregations

Criteria cr = session.createCriteria(Employee.class);
// To get total row count.
cr.setProjection(Projections.rowCount());
// To get average of a property.
cr.setProjection(Projections.avg("salary"));
// To get distinct count of a property.
cr.setProjection(Projections.countDistinct("firstName"));
// To get maximum of a property.
cr.setProjection(Projections.max("salary"));
// To get minimum of a property.
cr.setProjection(Projections.min("salary"));
// To get sum of a property.
cr.setProjection(Projections.sum("salary"));
~~~

1. first-level cache is the Session cache and is a mandatory cache through which all requests must pass. The Session object keeps an object under its own power before committing it to the database.

￼If you issue multiple updates to an object, Hibernate tries to delay doing the update as long as possible to reduce the number of update SQL statements issued. If you close the session, all the objects being cached are lost and either persisted or updated in the database.

2. Second level cache is an optional cache and first-level cache will always be consulted before any attempt is made to locate an object in the second-level cache. The second-level cache can be configured on a per-class and per- collection basis and mainly responsible for caching objects across sessions.
Any third-party cache can be used with Hibernate. An org.hibernate.cache.CacheProvider interface is provided, which must be implemented to provide Hibernate with a handle to the cache implementation.

3. Query-level cache
Hibernate also implements a cache for query resultsets that integrates closely with the second-level cache.
This is an optional feature and requires two additional physical cache regions that hold the cached query results and the timestamps when a table was last updated. This is only useful for queries that are run frequently with the same parameters

To use the query cache, you must first activate it using the hibernate.cache.use_query_cache="true"property in the configuration file. By setting this property to true, you make Hibernate create the necessary caches in memory to hold the query and identifier sets.

~~~ java
query.setCacheable(ture);

// region cache
query.setCacheRegion("employee");
~~~

// Concurrency strategies
Hibernate uses first-level cache by default and you have nothing to do to use first-level cache. Let's go straight to the optional second-level cache. Not all classes benefit from caching, so it's important to be able to disable the second-level cache

The Hibernate second-level cache is set up in two steps. First, you have to decide which concurrency strategy to use. After that, you configure cache expiration and physical cache attributes using the cache provider.
A concurrency strategy is a mediator which responsible for storing items of data in the cache and retrieving them from the cache. If you are going to enable a second-level cache, you will have to decide, for each persistent class and collection, which cache concurrency strategy to use.
* Transactional: Use this strategy for read-mostly data where it is critical to prevent stale data in concurrent transactions,in the rare case of an update.
* Read-write: Again use this strategy for read-mostly data where it is critical to prevent stale data in concurrent transactions,in the rare case of an update.
* Nonstrict-read-write: This strategy makes no guarantee of consistency between the cache and the database. Use this strategy if data hardly ever changes and a small likelihood of stale data is not of critical concern.
* Read-only: A concurrency strategy suitable for data which never changes. Use it for reference data only.

~~~ xml
<hibernate-mapping>
   <class name="Employee" table="EMPLOYEE">
   <meta attribute="class-description">
         This class contains the employee detail.
      </meta>
      <cache usage="read-write"/>
      <id name="id" type="int" column="id">
         <generator class="native"/>
      </id>
      <property name="firstName" column="first_name" type="string"/>
      <property name="lastName" column="last_name" type="string"/>
      <property name="salary" column="salary" type="int"/>
   </class>
</hibernate-mapping>
~~~

// Cashe provider: EHCache, OSCache, SwarmCashe, JBoss Cashe

Now, you need to specify the properties of the cache regions. EHCache has its own configuration file,ehcache.xml, which should be in the CLASSPATH of the application.

## batch processing

~~~ java
Session session = SessionFactory.openSession();
Transaction tx = session.beginTransaction();
for ( int i=0; i<100000; i++ ) {
    Employee employee = new Employee(.....);
    session.save(employee);
        if( i % 50 == 0 ) { // Same as the JDBC batch size
        //flush a batch of inserts and release memory:
        session.flush();
        session.clear();
} }
tx.commit();
session.close();
~~~

// edit batch size in cfg file

~~~ xml
<property name="hibernate.jdbc.batch_size">
50
</property>
~~~

//  Interceptors
An object passes through different stages in its life cycle and Interceptor Interface provides methods which can be called at different stages to perform some required tasks. These methods are callbacks from the session to the application, allowing the application to inspect and/or manipulate properties of a persistent object before it is saved, updated, deleted or loaded.

To build an interceptor you can either implement Interceptor class directly or extend EmptyInterceptorclass

~~~ java
public class MyInterceptor extends EmptyInterceptor { private int updates;
private int creates;
private int loads;
// This method is called when Employee object gets updated.
public boolean onFlushDirty(Object entity, Serializable id,
                     Object[] currentState,
                     Object[] previousState,
                     String[] propertyNames,
                     Type[] types) {
if ( entity instanceof Employee ) { System.out.println("Update Operation"); return true;
}
       return false;

 }  }
~~~

