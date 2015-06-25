---
layout: post
title:  "Scala Study Notes"
date:   2015-06-24
categories: Scala
excerpt: 
comments: true
---

## basics

### Partial application

You can partially apply a function with an underscore, which gives you another function. 
Scala uses the underscore to mean different things in different contexts, 
but you can usually think of it as an unnamed magical wildcard. 
In the context of { _ + 2 } it means an unnamed parameter. You can use it like so:

~~~ shell
scala> def adder(m: Int, n: Int) = m + n
adder: (m: Int,n: Int)Int
scala> val add2 = adder(2, _:Int)
add2: (Int) => Int = <function1>

scala> add2(3)
res50: Int = 5
~~~

### Curried functions

Sometimes it makes sense to let people apply some arguments to your function now and others later.

Here’s an example of a function that lets you build multipliers of two numbers together. 
At one call site, you’ll decide which is the multiplier and at a later call site, you’ll choose a multiplicand.

~~~ shell
scala> def multiply(m: Int)(n: Int): Int = m * n
multiply: (m: Int)(n: Int)Int
~~~

You can call it directly with both arguments.

~~~ shell
scala> multiply(2)(3)
res0: Int = 6
~~~

You can fill in the first parameter and partially apply the second.

~~~ shell
scala> val timesTwo = multiply(2) _
timesTwo: (Int) => Int = <function1>

scala> timesTwo(3)
res1: Int = 6
~~~

### Variable length arguments

There is a special syntax for methods that can take parameters of a repeated type. To apply String’s capitalize function to several strings, you might write:

~~~ scala
def capitalizeAll(args: String*) = {
  args.map { arg =>
    arg.capitalize
  }
}
~~~

~~~ shell
scala> capitalizeAll("rarity", "applejack")
res2: Seq[String] = ArrayBuffer(Rarity, Applejack)
~~~

**When do you want a Trait instead of an Abstract Class?**

If you want to define an interface-like type, you might find it difficult to choose between a trait or an abstract class. Either one lets you define a type with some behavior, asking extenders to define some other behavior. Some rules of thumb:

* Favor using traits. It’s handy that a class can extend several traits; a class can extend only one class.
* If you need a constructor parameter, use an abstract class. Abstract class constructors can take parameters; trait constructors can’t. For example, you can’t say trait t(i: Int) {}; the i parameter is illegal.

### Types

Earlier, you saw that we defined a function that took an Int which is a type of Number. Functions can also be generic and work on any type. When that occurs, you’ll see a type parameter introduced with the square bracket syntax. Here’s an example of a Cache of generic Keys and Values.

~~~ scala
trait Cache[K, V] {
  def get(key: K): V
  def put(key: K, value: V)
  def delete(key: K)
}
~~~

### Apply methods

apply methods give you a nice syntactic sugar for when a class or object has one main use.

~~~ shell
scala> class Foo {}
defined class Foo

scala> object FooMaker {
     |   def apply() = new Foo
     | }
defined module FooMaker

scala> val newFoo = FooMaker()
newFoo: Foo = Foo@5b83f762
~~~

### Functions are Objects

In Scala, we talk about object-functional programming often. What does that mean? What is a Function, really?

A Function is a set of traits. Specifically, a function that takes one argument is an instance of a Function1 trait. This trait defines the apply() syntactic sugar we learned earlier, allowing you to call an object like you would a function.

~~~ shell
scala> object addOne extends Function1[Int, Int] {
     |   def apply(m: Int): Int = m + 1
     | }
defined module addOne

scala> addOne(1)
res2: Int = 2
~~~

There is Function0 through 22. Why 22? It’s an arbitrary magic number. I’ve never needed a function with more than 22 arguments so it seems to work out.

### CASE CLASSES WITH PATTERN MATCHING

case classes are designed to be used with pattern matching. Let’s simplify our calculator classifier example from earlier.

~~~ scala
val hp20b = Calculator("hp", "20B")
val hp30b = Calculator("hp", "30B")

def calcType(calc: Calculator) = calc match {
  case Calculator("hp", "20B") => "financial"
  case Calculator("hp", "48G") => "scientific"
  case Calculator("hp", "30B") => "business"
  case Calculator(ourBrand, ourModel) => "Calculator: %s %s is of unknown type".format(ourBrand, ourModel)
}
~~~

### Option

Option is a container that may or may not hold something.

The basic interface for Option looks like:

~~~ scala
trait Option[T] {
  def isDefined: Boolean
  def get: T
  def getOrElse(t: T): T
}
~~~

Option itself is generic and has two subclasses: Some[T] or None

Let’s look at an example of how Option is used:

Map.get uses Option for its return type. Option tells you that the method might not return what you’re asking for.

We would suggest that you use either getOrElse or pattern matching to work with this result.

getOrElse lets you easily define a default value.

~~~ scala
val result = res1.getOrElse(0) * 2
~~~

### foreach

foreach is like map but returns nothing. foreach is intended for side-effects only.

~~~ shell
scala> numbers.foreach((i: Int) => i * 2)
returns nothing.
~~~

You can try to store the return in a value but it’ll be of type Unit (i.e. void)

~~~ shell
scala> val doubled = numbers.foreach((i: Int) => i * 2)
doubled: Unit = ()
~~~

### partition
partition splits a list based on where it falls with respect to a predicate function.

~~~ shell
scala> val numbers = List(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
scala> numbers.partition(_ % 2 == 0)
res0: (List[Int], List[Int]) = (List(2, 4, 6, 8, 10),List(1, 3, 5, 7, 9))
~~~

### foldLeft

~~~ shell
scala> numbers.foldLeft(0)((m: Int, n: Int) => m + n)
res0: Int = 55
~~~

0 is the starting value (Remember that numbers is a List[Int]), and m
acts as an accumulator.

Seen visually:

~~~ shell
scala> numbers.foldLeft(0) { (m: Int, n: Int) => println("m: " + m + " n: " + n); m + n }
m: 0 n: 1
m: 1 n: 2
m: 3 n: 3
m: 6 n: 4
m: 10 n: 5
m: 15 n: 6
m: 21 n: 7
m: 28 n: 8
m: 36 n: 9
m: 45 n: 10
res0: Int = 55
~~~

### flatMap

flatMap is a frequently used combinator that combines mapping and flattening. flatMap takes a function that works on the nested lists and then concatenates the results back together.

~~~ shell
scala> val nestedNumbers = List(List(1, 2), List(3, 4))
nestedNumbers: List[List[Int]] = List(List(1, 2), List(3, 4))

scala> nestedNumbers.flatMap(x => x.map(_ * 2))
res0: List[Int] = List(2, 4, 6, 8)
~~~

Think of it as short-hand for mapping and then flattening:

~~~ shell
scala> nestedNumbers.map((x: List[Int]) => x.map(_ * 2)).flatten
res1: List[Int] = List(2, 4, 6, 8)
~~~

Now filter out every entry whose phone extension is lower than 200.

~~~ shell
scala> extensions.filter((namePhone: (String, Int)) => namePhone._2 < 200)
res0: scala.collection.immutable.Map[String,Int] = Map((steve,100), (bob,101))
~~~

Because it gives you a tuple, you have to pull out the keys and values with their positional accessors. Yuck!
Lucky us, we can actually use a pattern match to extract the key and value nicely.

~~~ shell
scala> extensions.filter({case (name, extension) => extension < 200})
res0: scala.collection.immutable.Map[String,Int] = Map((steve,100), (bob,101))
~~~

### compose
compose makes a new function that composes other functions f(g(x))

~~~ shell
scala> val fComposeG = f _ compose g _
fComposeG: (String) => java.lang.String = <function>

scala> fComposeG("yay")
res0: java.lang.String = f(g(yay))
~~~

### andThen
andThen is like compose, but calls the first function and then the second, g(f(x))

~~~ shell
scala> val fAndThenG = f _ andThen g _
fAndThenG: (String) => java.lang.String = <function>

scala> fAndThenG("yay")
res1: java.lang.String = g(f(yay))
~~~

### Understanding PartialFunction

A function works for every argument of the defined type. In other words, a function defined as (Int) => String takes any Int and returns a String.

A Partial Function is only defined for certain values of the defined type. A Partial Function (Int) => String might not accept every Int.

isDefinedAt is a method on PartialFunction that can be used to determine if the PartialFunction will accept a given argument.

Note PartialFunction is unrelated to a partially applied function that we talked about earlier.

See Also Effective Scala has opinions about PartialFunction.

~~~ shell
scala> val one: PartialFunction[Int, String] = { case 1 => "one" }
one: PartialFunction[Int,String] = <function1>

scala> one.isDefinedAt(1)
res0: Boolean = true

scala> one.isDefinedAt(2)
res1: Boolean = false
~~~

### Mutable

All of the above classes we’ve discussed were immutable. Let’s discuss the commonly used mutable collections.

**HashMap** defines getOrElseUpdate, += HashMap API

~~~ shell
scala> val numbers = collection.mutable.Map(1 -> 2)
numbers: scala.collection.mutable.Map[Int,Int] = Map((1,2))

scala> numbers.get(1)
res0: Option[Int] = Some(2)

scala> numbers.getOrElseUpdate(2, 3)
res54: Int = 3

scala> numbers
res55: scala.collection.mutable.Map[Int,Int] = Map((2,3), (1,2))

scala> numbers += (4 -> 1)
res56: numbers.type = Map((2,3), (4,1), (1,2))
~~~

### Learning Functional Programming without Growing a Neckbeard

Tell computer What to do => what things are by evaluation of expressions

function is a relation between values where each of its input values gives back exactly one output value.

Functions are 1. deterministic, 2. encapsulation 3. commutative (lazy ) 4. data is immutable, you need to copy an object to change once it is created.

foreach, map, filter, fold, flatten, compose,

partial application, higher order function, function composition, for-yield.

testing more like plain english: must have, must startwith.

none using option and isdefined option. for-yield over option,

### [Martin Odersky: Scala with Style -- VIDEO](https://www.youtube.com/watch?v=kkTFx3-duc8)

guidelines:
1. keep it simple
2. don't pack too much in one exp
3. prefer functional
4. don't diabolize local state
5. careful with mutable objects
6. don't stop improving too early

three ways to write:
1. combinators
2. recursive function + pattern matching 
3. loop

DON'T:
1. don't use procedure syntax
2. nesting method is preferred, not very deep
3. pattern matching (functional convenient. for close system, single point) vs dynamic dispatch (core mechanism for extensible systems)

[scala simple parts -- VIDEO](https://www.youtube.com/watch?v=ecekSCX3B4Q)

![alt text](https://cloud.githubusercontent.com/assets/5607138/8345003/d5efe3c6-1a9d-11e5-8896-1fb0b466965b.png "Summary")

1. compose: use if-else, case
2. match exp match{case number(n), case plus(l,r)}
3. group nest the function def in def
4. recurse: tail-recurse is efficient @tailrec
5. abstract
6. aggregate: tranform immutable data
7. mutate

modelling parameterized types:

~~~ scala
class Set[T]{...} => class set {type $T}
class Set[+T]{...}
Set[String] => Set {type $T = String}
List[Number] => List{type $T <: Number}
~~~

---

1. Null- Its a Trait.
2. null- Its an instance of Null- Similar to Java null.
3. Nil- Represents an emptry List of anything of zero length. Its not that it refers to nothing but it refers to List which has no contents.
4. Nothing is a Trait. Its a subtype of everything. But not superclass of anything. There are no instances of Nothing.
5. None- Used to represent a sensible return value. Just to avoid null pointer exception. Option has exactly 2 subclasses- Some and None. None signifies no result from the method.
6. Unit- Type of method that doesn’t return a value of any sort.


[monad VIDEO](https://www.youtube.com/watch?v=Mw_Jnn_Y5iA&feature=relmfu)
![alt text](https://cloud.githubusercontent.com/assets/5607138/8345068/c504f9b0-1a9e-11e5-9fee-4150c922aaeb.png)

[monad VIDEO 2](https://www.youtube.com/watch?v=ZhuHCtR3xq8)
![alt text](https://cloud.githubusercontent.com/assets/5607138/8345069/c71094f8-1a9e-11e5-96a9-38f4f7b6d095.png)
![alt text](https://cloud.githubusercontent.com/assets/5607138/8345073/d406b1b0-1a9e-11e5-9e0d-a35ebdeb8daf.png)

(ga) = Mb combines function (b -> Mc which is f), and rethrn Mc. 

### good resource:

[LINK](http://twitter.github.io/effectivescala/)
[LINK](http://kukuruku.co/hub/scala/java-8-vs-scala-the-difference-in-approaches-and-mutual-innovations)
