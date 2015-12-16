---
layout: post
title:  "Java Study Notes"
date:   2015-06-24
categories: Study-Notes
excerpt: Java study notes
comments: true
---

* content
{:toc}

## Java Generic Types

To specify the upper bound of a type wildcard, the extends keyword is used, which indicates that the type argument is a subtype of the bounding class. So List<? extends Number> means that the given list contains objects of some unknown type which extends the Number class. For example, the list could be List<Float> or List<Number>. Reading an element from the list will return a Number, while adding non-null elements is once again not allowed.

## import static

Where the normal import declaration imports classes from packages, allowing them to be used without package qualification, the static import declaration imports static members from classes, allowing them to be used without class qualification.

~~~ java
 double r = Math.cos(Math.PI * theta);
import static java.lang.Math.PI;
Once the static members have been imported, they may be used without qualification:
double r = cos(PI * theta);
~~~

**DTO** (Data Transfer Object) is a pattern and it is implementation (POJO/POCO) independent. DTO says, since each call to any remote interface is expensive, response to each call should bring as much data as possible. So, if multiple requests are required to bring data for a particular task, data to be brought can be combined in a DTO so that only one request can bring all the required data. 

**Java Architecture for XML Binding (JAXB)** is a Java standard that defines how Java objects are converted from and to XML. It uses a standard set of mappings.

**POJO** is just a plain, old Java Bean with the restrictions removed. Java Beans must meet the following requirements:

Default no-arg constructor
Follow the Bean convention of getFoo (or isFoo for booleans) and setFoo methods for a mutable attribute named foo; leave off the setFoo if foo is immutable.

Must implement java.io.Serializable

POJO does not mandate any of these. It's just what the name says: an object that compiles under JDK can be considered a Plain Old Java Object. No app server, no base classes, no interfaces required to use.

@HeaderParam annotation allows you to map a request HTTP header onto your method invocation.

@PathParam is a parameter annotation which allows you to map variable URI path fragments into your method call.

~~~ java
@Path("/library")
public class Library {

   @GET
   @Path("/book/{isbn}")
   public String getBook(@PathParm("isbn") String id) {
      // search my database and get a string representation and return it
   }
}
~~~


| Annotation | Meaning                                             |
+------------+-----------------------------------------------------+
| @Component | generic stereotype for any Spring-managed component |
| @Repository| stereotype for persistence layer                    |
| @Service   | stereotype for service layer                        |
| @Controller| stereotype for presentation layer (spring-mvc)      |


@Override Use it every time you override a method for two benefits. Do it so that you can take advantage of the compiler checking to make sure you actually are overriding a method when you think you are. This way, if you make a common mistake of misspelling a method name or not correctly matching the parameters, you will be warned that you method does not actually override as you think it does. Secondly, it makes your code easier to understand because it is more obvious when methods are overwritten. Additionally, in Java 1.6 you can use it to mark when a method implements an interface for the same benefits. 

@Transactional I think transactions belong on the Service layer. It's the one that knows about units of work and use cases. It's the right answer if you have several DAOs injected into a Service that need to work together in a single transaction.

@SuppressWarnings indicates that the named compiler warnings should be suppressed in the annotated element (and in all program elements contained in the annotated element). Note that the set of warnings suppressed in a given element is a superset of the warnings suppressed in all containing elements. For example, if you annotate a class to suppress one warning and annotate a method to suppress another, both warnings will be suppressed in the method.

Named parameters
This is the most common and user friendly way. It use colon followed by a parameter name (:example) to define a named parameter. See examples…

Example 1 – setParameter


The setParameter is smart enough to discover the parameter data type for you.

~~~ java
String hql = "from Stock s where s.stockCode = :stockCode";
List result = session.createQuery(hql)
.setParameter("stockCode", "7277")
.list();
~~~

Example 2 – setString


You can use setString to tell Hibernate this parameter date type is String.

~~~ java
String hql = "from Stock s where s.stockCode = :stockCode";
List result = session.createQuery(hql)
.setString("stockCode", "7277")
.list();
~~~

Example 3 – setProperties


This feature is great ! You can pass an object into the parameter binding. Hibernate will automatic check the object’s properties and match with the colon parameter.

~~~ java
Stock stock = new Stock();
stock.setStockCode("7277");
String hql = "from Stock s where s.stockCode = :stockCode";
List result = session.createQuery(hql)
.setProperties(stock)
.list();
~~~

@Temporal This annotation must be specified for persistent fields or properties of type java.util.Date andjava.util.Calendar. It may only be specified for fields or properties of these types.


Smoke testing is a set of basic cheap to run tests that precede actual testing. It aims to verify that the build is deployed successfully and that all test env. aspects are running and ready for the actual test process. It saves you bringing the full extent of your testing wrath down a faulty build and just realizing that you have been testing on a bad env. or erroneously deployed build possibly too late.

## immutable types like String
for thread save.

--- Effetive Java
// Enum singleton - the preferred approach

public enum Elvis {
INSTANCE;
public void leaveTheBuilding() { ... }
}

This approach is functionally equivalent to the public field approach, except that it
is more concise, provides the serialization machinery for free, and provides an
ironclad guarantee against multiple instantiation, even in the face of sophisticated
serialization or reflection attacks. While this approach has yet to be widely
adopted, a single-element enum type is the best way to implement a singleton.



