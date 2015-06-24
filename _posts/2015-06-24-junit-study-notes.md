---
layout: post
title:  "JUnit Study Notes"
date:   2015-06-17 14:06:05
categories: JUnit
excerpt: JUnit
comments: true
---

* content
{:toc}

## Basics

// JUnit test framework provides following important features
1. Fixtures
2. Test suites
3. Test runners
4. JUnit classes

Fixtures
Fixtures is a fixed state of a set of objects used as a baseline for running tests. The purpose of a test fixture is to ensure that there is a well known and fixed environment in which tests are run so that results are repeatable. It includes
..* setUp() method which runs before every test invocation.
..* tearDown() method which runs after every test method.

Test suite means bundle a few unit test cases and run it together. In JUnit, both @RunWith and @Suite annotation are used to run the suite test. Here is an example which uses TestJunit1 & TestJunit2 test classes.

Test runner is used for executing the test cases. Here is an example which assumes TestJunit test class already exists.

JUnit classes are important classes which is used in writing and testing JUnits. Some of the important classes are
..* Assert which contain a set of assert methods.
..* TestCase which contain a test case defines the fixture to run multiple tests.
..* TestResult which contain methods to collect the results of executing a test case.

## Junit API
The most important package in classses. Some of the important class are:
1. Assert: a set of assert methods
2. TestCase: A test case defines the fixture to run multiple tests
3. TestResult: collects the results of executing a test case
4. TestSuite: a composite of tests

## execution procedure

..* in before class
..* in before
..* in test case 1
..* in after
..* in before
..* in test case 2
..* in after
..* in after class

..* @Before and @After will execute for every test cases

## Timeout
If a test case takes more time than specified number
of milliseconds then Junit will automatically mark it as failed.

## exception test
It provides an option of tracing the Exception handling of code. You can test the code
whether code throws desired exception or not. The expected parameter is used along with @Test annotation. Now let's see @Test(expected) in action.

@Test(expected = ArithmeticException.class)

## parameterized test

A new feature Parameterized tests. Parameterized tests allow developer to run the same test over and over again using different values. There are five steps,
that you need to follow to create Parameterized tests.
..* Annotate test class with @RunWith(Parameterized.class)
..* Create a public static method annotated with @Parameters that returns a Collection of Objects (as Array) as test data set.
..* Create a public constructor that takes in what is equivalent to one "row" of test data.
..* Create an instance variable for each "column" of test data.
..* Create your tests case(s) using the instance variables as the source of the test data.

```java
import java.util.Arrays;
import java.util.Collection;
import org.junit.Test;
import org.junit.Before;
import org.junit.runners.Parameterized;
import org.junit.runners.Parameterized.Parameters; import org.junit.runner.RunWith;
import static org.junit.Assert.assertEquals;
@RunWith(Parameterized.class)
public class PrimeNumberCheckerTest {
private Integer inputNumber;
private Boolean expectedResult;
private PrimeNumberChecker primeNumberChecker;
    @Before
    public void initialize() {
primeNumberChecker = new PrimeNumberChecker(); }
// Each parameter should be placed as an argument here // Every time runner triggers, it will pass the arguments
// from parameters we defined in primeNumbers() method
public PrimeNumberCheckerTest(Integer inputNumber,Boolean expectedResult) {
this.inputNumber = inputNumber; this.expectedResult = expectedResult;
}
@Parameterized.Parameters
public static Collection primeNumbers() {
return Arrays.asList(new Object[][] { { 2, true },
          { 6, false },
          { 19, true },
          { 22, false },
          { 23, true }
}); }
// This test will run 4 times since we have 5 parameters defined @Test
public void testPrimeNumberChecker() {
System.out.println("Parameterized Number is : " + inputNumber); assertEquals(expectedResult, primeNumberChecker.validate(inputNumber));
} }
```

### [good test practice rules](http://howtodoinjava.com/2012/11/05/unit-testing-best-practices-junit-reference-guide/)


Tips for writing great unit tests
..* nit testing is not about finding bugs
..* Test only one code unit at a time
..* Don’t make unnecessary assertions
..* Make each test independent to all the others
..* Mock out all external services and state
..* Don’t unit-test configuration settings
..* Name your unit tests clearly and consistently
..* Write tests for methods that have the fewest dependencies first, and work your way up
..* All methods, regardless of visibility, should have appropriate unit tests
..* Aim for each unit test method to perform exactly one assertion
..* Create unit tests that target exceptions
..* Use the most appropriate assertion methods.
..* Put assertion parameters in the proper order
..* Ensure that test code is separated from production code
..* Do not print anything out in unit tests
..* Do not use static members in a test class
..* Do not write your own catch blocks that exist only to fail a test
..* Do not rely on indirect testing
..* Integrate Testcases with build script
..* Do not skip unit tests
..* Capture results using the XML formatter
