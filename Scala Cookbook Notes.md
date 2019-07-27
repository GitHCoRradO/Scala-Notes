## Scala Cookbook Notes

### Ch01  String

#### String interpolators

1. print(s".....")
2. print(f"......")  for printf style: Table 1-1 and [format specifiers and examples](http://alvinalexander.com/programming/printf-format-cheat-sheet)
   ```
   scala> println(f"$name is $age years old, and weighs $weight%.2f pounds.") Fred is 33 years old, and weighs 200.00 pounds.
   ```
3. raw
   ```
   scala> raw"foo\nbar" 
   res1: String = foo\nbar
   ```
   
#### Replacing Patterns in Strings  (Regex)

1. replaceAll
      ```scala> val address = "123 Main Street".replaceAll("[0-9]", "x")
      address: java.lang.String = xxx Main Street
      ```
2. regex.replaceAllIn
      ```
    scala> val regex = "[0-9]".r
    regex: scala.util.matching.Regex = [0-9]
    scala> val newAddress = regex.replaceAllIn("123 Main Street", "x") 
    newAddress: String = xxx Main Street
      ```
3. regex.replaceFirstIn
     ```scala> val regex = "H".r
     regex: scala.util.matching.Regex = H
     scala> val result = regex.replaceFirstIn("Hello world", "J") result: String = Jello world
     ```
#### Add your own methods to the String class

### Ch02 Numbers

#### Comparing floating-point numbers

1.
     ```
     def ~=(x: Double, y: Double, precision: Double) = {
     if ((x - y).abs < precision) true else false }
      
     scala> val a = 0.3 a: Double = 0.3
     scala> val b = 0.1 + 0.2
     b: Double = 0.30000000000000004
     scala> ~=(a, b, 0.0001) res0: Boolean = true
     scala> ~=(b, a, 0.0001) res1: Boolean = true
     ```

#### Generating random numbers

1. [how to create a list of alpha or alphanumeric characters](http://alvinalexander.com/scala/create-list-alpha-alphanumeric-characters-in-scala)

2. [generating random strings](http://alvinalexander.com/scala/creating-random-strings-in-scala)

#### Formatting numbers and currency

1. [The Joda Money library is a Java library for handling currency](https://www.joda.org/joda-money/)

### Ch03 Control Structures

#### Looping for and foreach

1. zipWithIndex
   ```
   scala> for ((e, count) <- a.zipWithIndex) { println(s"$count is $e")
      }
          0 is apple
          1 is banana
          2 is orange
   
   ```
2. for loop can have as many generators and guards as needed
   ```
    for {
    i <- 1 to 10
    if i > 3
    if i < 6
    if i % 2 == 0
    } println(i)
   ```
3. for comprehension
    Except for rare occasions, the collection type returned by a for comprehension is the same type that you begin with.
    ```
    var fruits = scala.collection.mutable.ArrayBuffer[String]() fruits += "apple"
    fruits += "banana"
    fruits += "orange"
    scala> val out = for (e <- fruits) yield e.toUpperCase
    out: scala.collection.mutable.ArrayBuffer[java.lang.String] =
    ArrayBuffer(APPLE, BANANA, ORANGE)
    
    scala> val fruits = "apple" :: "banana" :: "orange" :: Nil fruits: List[java.lang.String] = List(apple, banana, orange)
    scala> val out = for (e <- fruits) yield e.toUpperCase out: List[java.lang.String] = List(APPLE, BANANA, ORANGE)
    ```
    
#### breaks; match case
4. labeled breaks

    ```
    object LabeledBreakDemo extends App {
          import scala.util.control._
          val Inner = new Breaks
          val Outer = new Breaks
          Outer.breakable {
          for (i <- 1 to 5) { Inner.breakable {
          for (j <- 'a' to 'e') {
          if (i == 1 && j == 'c') Inner.break else println(s"i: $i, j: $j")
          if (i == 2 && j == 'b') Outer.break }
          } }
          } }
          
    ```
5. advantages of tail recursion over recursion;
   use the @tailrec annotation properly
   [discussion](https://stackoverflow.com/questions/1187433/scala-factorial-on-large-numbers-sometimes-crashes-and-sometimes-doesnt)
   [discussion](http://blog.richdougherty.com/2009/04/tail-calls-tailrec-and-trampolines.html)
   
6. Using pattern matching in Match expressions
   Patterns:
        Constant patterns
        Variable patterns
        Constructor patterns
        Sequence patterns
        Tuple patterns
        Type patterns
   At times you may want to add a variable to a pattern. You can do this with the following general syntax:
   
   ```variableName @ patter```
   
7. _case classes_ can be used in match expressions

#### Adding if Expressions (Guards) to Case Statements
1. add simple matches to your case statements on the left side of the expression. 
   ``` 
   i match {
   case a if 0 to 9 contains a => println("0-9 range: " + a) 
   case b if 10 to 19 contains b => println("10-19 range: " + b) 
   case c if 20 to 29 contains c => println("20-29 range: " + c) 
   case _ => println("Hmmm...")
   }
   ```
   ``` 
   def speak(p: Person) = p match {
   case Person(name) if name == "Fred" => println("Yubba dubba doo") 
   case Person(name) if name == "Bam Bam" => println("Bam bam!") 
   case _ => println("Watch the Flintstones!")
   }
   ```
2. all of these examples could be written by putting the if tests on the right side of the expressions; However, for many situations, your code will be simpler and easier to read by joining the if guard directly with the case statement.
   ``` 
   case Person(name) =>
        if (name == "Fred") println("Yubba dubba doo") 
        else if (name == "Bam Bam") println("Bam bam!")
   ```

### Ch04 Classes and Properties

#### The effect of constructor parameter settings

    | Visibility         |     Accessor?      |  Mutator?     |
    |--------------------|--------------------|---------------|
    | var                |  yes               | yes           |
    |--------------------|--------------------|---------------|
    | val                |  yes               | no            |
    |--------------------|--------------------|---------------|
    | Default visibility |  no                | no            |
    |(no val or var)     |                    |               |
    |--------------------|--------------------|---------------|
    | Adding private     | no                 |  no           |
    | keyword to val,var |                    |               |
    |--------------------|--------------------|---------------|

#### Case class
1. Case class constructor parameters are val by default.
   So if you define a case class field without adding val or var,
   you can still access the field, just as if it were defined as a val.
   
#### companion object
1. A companion object is simply an object that’s defined in the same file as a class, where the object and class have the same name. If you declare a class named Foo in a file named Foo.scala, and then declare an object named Foo in that same file, the Foo object is the compan‐ ion object for the Foo class. A companion object has several purposes, and one purpose is that any method declared in a companion object will appear to be a static method on the object. 

#### Preventing getter and setter methods from being generated
1. To do this, define the field with *private* or *private[this]* access modifiers: with *private*, the field is only available to instances of the same class; with *private[this]*, the field can only be accessed by the instance of the class itself, not available by other instances of the same type.

#### Assigning a field to a block or function
1. Define a field to be *lazy* makes it not evaluated until accessed; it is a useful approach when the field might not be accessed in the normal processing of your algorithms, or if running the algorithm will take a long time, and you want to defer that to a later time.
#### Auxiliary classes; Calling a superclass constructor
1. the first line of an auxiliary constructor must be a call to another constructor of the current class, there is no way for auxiliary con‐ structors to call a superclass constructor.
2. the primary constructor of the Employee (sub) class can call any constructor in the Person (base) class, but the auxiliary constructors of the Employee class must call a previously defined constructor of its own class with the this method as its first line.
3. Therefore, there’s no direct way to control which superclass constructor is called from an auxiliary constructor in a subclass. In fact, because each auxiliary constructor must call a previously defined constructor in the same class, all auxiliary constructors will eventually call the same superclass constructor that’s called from the subclass’s primary constructor.
#### Defining case classes
1. defining case classes causes the following codes to be automatically generated:
   1. apply method, so you need not new keyword when creating new instances of the class.
   2. accessors and mutators for parameters.
   3. a good, default toString method.
   4. unapply method for use in match expressions
   5. copy method
#### Defining an equals method (Object equality)
1. in Scala, == is a method you use on each class to compare the equality of two instances, calling your equals method under the covers.
2. The Scaladoc for the equals method of the Any class states, “any implementation of this method should be an equivalence relation.” The documentation states that an equiva‐ lence relation should have these three properties:
   1. x == x true
   2. x == y true then y == x true
   3. x == y true and y == z true then x == z true
#### Inner class
1. inner classes are bound to its outer objects

### Ch05 Methods
#### Controlling method scopes
1. Descriptions of Scala’s access control modifiers

   | Access modifier            | Description                               |
   |----------------------------|-------------------------------------------|
   |private[this]               | available only to the current instance    |
   |                            | of the class it's declared in             |
   |----------------------------|-------------------------------------------|
   |private                     |available to the current instance and other|
   |                            |instances of the class it’s declared in    |
   |----------------------------|-------------------------------------------|
   |protected                   |available only to instances of the current |
   |                            |class and subclasses of the current class  |
   |----------------------------|-------------------------------------------|
   |private[coolapp]            |available to all classes beneath coolapp   |
   |                            |package                                    |
   |----------------------------|-------------------------------------------|
   |(no modifier)               | the method is public                      |
   |----------------------------|-------------------------------------------|

#### Setting default values for method parameters
1. If your method provides a mix of some fields that offer default values and others that don’t, list the fields that have default values last. 
   ```  
   class Connection {
   // intentional error
   def makeConnection(timeout: Int = 5000, protocol: String) {
           println("timeout = %d, protocol = %s".format(timeout, protocol))
           // more code here
   } }
   
   class Connection {
   // corrected implementation
   def makeConnection(timeout: Int, protocol: String = "http") {
           println("timeout = %d, protocol = %s".format(timeout, protocol))
           // more code here
   } }
   
   ```
#### Defining a method that returns multiple items(tuples)
1. Tuples can contain up to 22 variables and are imple‐ mented as Tuple1 through Tuple22 classes. 
   ``` 
   def getStockInfo = {
   // other code here ...
   ("NFLX", 100.00, 101.00) // this is a Tuple3
   }
   scala> val (symbol, currentPrice, bidPrice) = getStockInfo symbol: java.lang.String = NFLX
   currentPrice: Double = 100.0
   bidPrice: Double = 101.0
   
   scala> val result = getStockInfo
   x: (java.lang.String, Double, Double) = (NFLX,100.0)
   scala> result._1
   res0: java.lang.String = NFLX
   scala> result._2 res1: Double = 100.0
   ```
2. tuple values can be accessed by position as result._1, result._2 and so on.

#### Forcing Callers to Leave Parentheses off Accessor Methods
1. Define your getter/accessor method without parentheses after the method name, This forces consumers of your class to call the accessor method without parentheses

#### Creating Methods That Take Variable-Argument Fields
1. Define a varargs field in your method declaration by adding a * character after the field type:
   ``` 
   def printAll(strings: String*) {
         strings.foreach(println)
       }
   
   // these all work
       printAll()
       printAll("foo")
       printAll("foo", "bar")
       printAll("foo", "bar", "baz")
   ```
2. you can use Scala’s _* operator to adapt a sequence (Array, List, Seq, Vector, etc.) so it can be used as an argument for a varargs field:
   ``` 
   // a sequence of strings
   val fruits = List("apple", "banana", "cherry") // pass the sequence to the varargs field
   printAll(fruits: _*)
   ```
3. When declaring that a method has a field that can contain a variable number of argu‐ ments, the varargs field must be the last field in the method signature. As an implication of that rule, a method can have only one varargs field.

#### declaring that a method can throw an exception
1. Use the @throws annotation to declare the exception(s) that can be thrown:
   ``` 
   @throws(classOf[IOException])
   @throws(classOf[LineUnavailableException])
   @throws(classOf[UnsupportedAudioFileException])
   def playSoundFileWithJavaAudio {
         // exception throwing code here ...
   }
   ```
2. Scala doesn’t require that methods declare that exceptions can be thrown, and it also doesn’t require calling methods to catch them. Even so, if one fails to test for them, they'll blow up one's code like they do in Java.
### Ch06 Objects
 
#### Object casting
1. Use the asInstanceOf method to cast from one type to another.
    ``` 
    val cat = Cat("meow")
    val catAsAnimal = cat.asInstanceOf[Animal]
    ```
#### classOf method
1. classOf method is the scala equivalent of Java's .class; called on Class names
   ``` 
   val stringClass = classOf[String]
   stringClass.getMethods
   ```
#### Determine the class of an Object
1. call getClass on an object returns the class name of the object:
   ``` 
   scala> def printClass(c: Any) { println(c.getClass) } printClass: (c: Any)Unit
   scala> printClass(1) class java.lang.Integer
   scala> printClass("yo") class java.lang.String
   ```
#### Lauching an application with an object
1. define an object(object Hello) that extends the App trait, save code to a file Hello.scala(preferred)
   ``` 
   object Hello extends App {
         println("Hello, world")
       }
   ```
2. Or manually implement a main method with the correct signature in an object, in a manner similar to Java
   ``` 
   object Hello2 {
   def main(args: Array[String]) {
           println("Hello, world")
         }
   }
   ```
   
#### Creating static members with companion objects
1. Here is how class and its companion object is defined:
   1. Define your class and object in the same file, giving them the same name.
   2. Define members that should appear to be "static" in the object
   3. Define nonstatic(instance) members in the class
2. a class and its companion object can access each other’s private members.

#### Putting common code in package objects
1. package objects are a great place to put methods and functions that are common to the package, as well as constants, enumerations, and implicit conversions.
2. package objects are named _package.scala_ and they have several conventions:
   ``` 
   package com.alvinalexander.myapp
   //note this blank line, it is needed
   package object model {
   
   ```
   1. this file package.scala in the com/alvinalexander/myapp/model source code directory

#### Creating object instances without using the new keyword
1. two ways:
   1. Create a companion object for your class, and define an apply method in the com‐ panion object with the desired constructor signature.
   2. Define your class as a case class.
   
### Ch07 Packaging and Imports
#### Renaming members on Import
1. You can create a new name for a class when you import it, and can then refer to it by the new name, or alias.
2. This can be very helpful when trying to avoid namespace collisions and confusion. Class names like Listener, Message, Handler, Client, Server, and many more are all very common, and it can be helpful to give them an alias when you import them.
3. Not only can you rename classes on import, but you can even rename class members.
   ``` 
   //example 1
   import java.util.{Date => JDate, HashMap => JHashMap}
   
   //example 2
   import java.util.{HashMap => JavaHashMap}
   import scala.collection.mutable.{Map => ScalaMutableMap}
        //or
   import java.util.{HashMap => JavaHashMap}
   import scala.collection.mutable.Map
   
   //example 3
   scala> import System.out.{println => p} 
   import System.out.{println=>p}
   
   scala> p("hello") hello
   ```
#### Hiding a class during the import process
1. The following portion of the code is what “hides” the List, Map, Set classes, the second _ character inside the curly braces is the same as stating that you want to import everything else in the package. The _ import wildcard must be in the last position.This is because you may want to hide multiple members during the import process, and to do, so you need to list them first.
   ``` 
   import java.util.{List => _, Map => _, Set => _, _}
   ```
#### Using import statements anywhere
1. Import statements are read in the order of the file, so where you place them in a file also limits their scope.
   ```
   // this doesn't work because the import is after the attempted reference
   class ImportTests { 
         def printRandom {
             val r = new Random // fails
      }
   }
   import scala.util.Random
   ```
2. In the following example, members can be accessed as follows:
  + Code in the orderentry package can access members of foo, but can’t access mem‐
    bers of bar or baz.
  + Code in customers and customers.database can’t access members of foo.
  + Code in customers can access members of bar.
  + Code in customers.database can access members in bar and baz.
  ``` 
  package orderentry { import foo._
  // more code here ...
    }
  package customers { import bar._
  // more code here ...
  package database { import baz._
  // more code here ...
    } 
  }
  ```
### Ch08 Traits
#### A trait can require that any type that wishes to extend it must extend multiple other types.
1. The following WarpCore definition requires that any type that wishes to mix it in must extend WarpCoreEjector and FireExtinguisher, in addition to extending Starship:
   ``` 
   trait WarpCore {
   this: Starship with WarpCoreEjector with FireExtinguisher =>
   }
   
   class Starship
   trait WarpCoreEjector
   trait FireExtinguisher
   // this works
   class Enterprise extends Starship
     with WarpCore
     with WarpCoreEjector
     with FireExtinguisher
   ```
   
#### Ensuring a trait can only be added to a type that has a specific method
1. In the following example, the WarpCore trait requires that any classes that attempt to mix it in must have an ejectWarpCore method; it further states that the ejectWarpCore method must accept a String argument and return a Boolean value.:
   ``` 
   trait WarpCore {
   this: { def ejectWarpCore(password: String): Boolean } =>
   }
   
   class Starship {
         // code here ...
   }
   class Enterprise extends Starship with WarpCore { def ejectWarpCore(password: String): Boolean = {
   if (password == "password") { println("ejecting core") true
   } else { false
   } }
   }
   
   //A trait can also require that a class have multiple methods. To require more than one method, just add the additional method signatures inside the block:
   trait WarpCore { 
        this: {
                def ejectWarpCore(password: String): Boolean
                def startWarpCore: Unit
        } => 
   }
       class Starship
   class Enterprise extends Starship with WarpCore { def ejectWarpCore(password: String): Boolean = {
   if (password == "password") { println("core ejected"); true } else false }
   def startWarpCore { println("core started") } }
   ```
   
#### Adding a trait to an object instance
#### Extending a Java interface like a trait
1. In your Scala application, use the extends and with keywords to implement your Java interfaces, just as though they were Scala traits.The difference is that Java interfaces don’t implement behavior, so if you’re defining a class that extends a Java interface, you’ll need to implement the methods, or declare the class abstract.

### Ch09 Functional Programming
#### Using function literals
1. anonymous function aka function literal

#### Using functions as variables
1. Summary notes:
   1. Think of the => symbol as a transformer. It transforms the input data on its left side to some new output data, using the algorithm on its right side.
   2. Use def to define a method, val, to create a function.
   3. When assigning a function to a variable, a function literal is the code on the right
      side of the expression.
   4. A function value is an object, and extends the FunctionN traits in the main scala package, such as Function0 for a function that takes no parameters.
2. Several examples:
   ``` 
   val f: (Int) => Boolean = i => { i % 2 == 0 }
   val f: Int => Boolean = i => { i % 2 == 0 }
   val f: Int => Boolean = i => i % 2 == 0
   val f: Int => Boolean = _ % 2 == 0
   
   // implicit approach
   val add = (x: Int, y: Int) => { x + y } 
   val add = (x: Int, y: Int) => x + y
   
   // explicit approach
   val add: (Int, Int) => Int = (x,y) => { x + y } 
   val add: (Int, Int) => Int = (x,y) => x + y
   
   ```
#### Defining a method that accepts a simple function parameter
1. Define your method, including the signature for the function you want to take as a method parameter.
2. Define one or more functions that match this signature.
3. Sometime later, pass the function(s) as a parameter to your method.
   ``` 
   scala> def executeFunction(callback:() => Unit) { callback() } 
   executeFunction: (callback: () => Unit)Unit
   scala> val sayHello = () => { println("Hello") }
   sayHello: () => Unit = <function0>
   scala> executeFunction(sayHello)
   Hello
   ```
#### More complex functions
1. The general syntax for describing a function as a method parameter is this:

   ``` parameterName: (parameterType(s)) => returnType```
2. In all of the previous examples where you created functions with the val keyword, you could have created methods (with a def keyword), and the examples would still work. 
   ``` 
   def exec(callback: (Any, Any) => Unit, x : Any, y : Any): Unit = {
     callback(x, y)
   }
   
   val printTwoThings = (a: Any, b: Any) => {
     println(a)
     println(b)
   }
   def printTwoThings1(a: Any, b: Any) {
     println(a)
     println(b)
   }
   case class Person(name: String)
   
   exec(printTwoThings1, "Hello", Person("Dave"))
   ```
#### Using closures
#### Using partially applied functions
1. You can use partially applied functions to make programming easier by binding some arguments—typically some form of local arguments—and leaving the others to be filled in.
   ``` 
   def wrap(prefix: String, html: String, suffix: String) = {
     prefix + html + suffix
   }
   //partially applied wrap
   val wrapWithDiv = wrap("<div>", _: String, "</div>")
   
   scala> wrapWithDiv("<p>Hello, world</p>")
   res0: String = <div><p>Hello, world</p></div>
   
   //use the orignial wrap
   scala> wrap("<pre>", "val x = 1", "</pre>")
   res1: String = <pre>val x = 1</pre>
   ```
#### Creating a function that returns a function
1. Define a function that returns an algorithm (an anonymous function), assign that to a new function, and then call that new function.
   ``` 
   def greeting(language: String) = (name: String) => {
        language match {
            case "english" => "Hello, " + name
            case "spanish" => "Buenos dias, " + name
            }
        }
   
   ```
2. On the left side of the = symbol you have a normal method declaration:def and method name and parameters list;
   On the right side of the = is a function literal (also known as an anonymous function)

#### Creating partial functions
1. A partial function is a function that does not provide an answer for every possible input value it can be given. It provides an answer only for a subset of possible data, and defines the data it can handle. In Scala, a partial function can also be queried to determine if it can handle a particular value.

### Ch10 Collections
#### predicate
1. A predicate is simply a method, function, or anonymous function that takes one or more parameters and returns a Boolean value. 
#### Choosing a collection class
1. strict and lazt collections: In a strict collection, memory for the elements is allocated immediately, and all of its elements are immediately evaluated when a transformer method is invoked. In a lazy collection, memory for the elements is not allocated immediately, and transformer methods do not construct new elements until they are demanded.
#### Choosing a collection method to solve a problem
1. Methods organized by category
   1. filtering methods: Methods that can be used to filter a collection
   2. transformer methods: Transformer methods take at least one input collection to create a new output col‐ lection, typically using an algorithm you provide.
   3. grouping methods: These methods let you take an existing collection and create multiple groups from that one collection.
   4. Informational and mathematical methods: These methods provide information about a collection
   5. other methods
#### Understanding the performance of collections
1. When choosing a collection for an application where performance is extremely impor‐ tant, you want to choose the right collection for the algorithm.
#### Declaring a type when creating a collection
1. Scala does not automatically assign the type you want when you create a collection of mixed types.
   ``` 
   //code example
   scala> val x = List(1, 2.0, 33D, 400L)
   x: List[Double] = List(1.0, 2.0, 33.0, 400.0)
         // ^ note the Double here
         
   scala> val x = List[Number](1, 2.0, 33D, 400L)
   x: List[java.lang.Number] = List(1, 2.0, 33.0, 400)
   
   scala> val x = List[AnyVal](1, 2.0, 33D, 400L) 
   x: List[AnyVal] = List(1, 2.0, 33.0, 400)
   ```
#### Understanding mutable variables with immutable collections
1. Though it looks like you’re mutating an immutable collection, what’s really happening is that the sisters variable points to a new collection each time you use the :+ method. The sisters variable is mutable—like a non-final field in Java—so it’s actually being reassigned to a new collection during each step. 
   ``` 
   scala> var sisters = Vector("Melinda")
   sisters: collection.immutable.Vector[String] = Vector(Melinda)
   scala> sisters = sisters :+ "Melissa"
   sisters: collection.immutable.Vector[String] = Vector(Melinda, Melissa)
   scala> sisters = sisters :+ "Marisa"
   sisters: collection.immutable.Vector[String] = Vector(Melinda, Melissa, Marisa)
   scala> sisters.foreach(println)
   Melinda
   Melissa
   Marisa
   ```
2. The elements in a mutable collection can be changed; the elements in an immutable collection cannot be changed.
#### Make Vector your "Go To" immutable sequence
1. if you create an instance of an IndexedSeq, scala returns a Vector; As a result, I’ve seen some developers create an IndexedSeq in their code, rather than a Vector, to be more generic and to allow for potential future changes.
#### Make ArrayBuffer your "Go To" mutable sequence
#### Looping over a collection with foreach
1. The foreach method takes a function as an argument. The function you define should take an element as an input parameter, and should not return anything. The input parameter type should match the type stored in the collection. As foreach executes, it passes one element at a time from the collection to your function until it reaches the last element in the collection.
#### Looping over a collection with a for loop
1. for; for/ yield; for with guards
#### Using zipWithIndex or zip to create loop counters
1. Use the zipWithIndex or zip methods to create a counter automatically. 
   ``` 
   val days = Array("Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday")
   
   days.zipWithIndex.foreach {
        case(day, count) => println(s"$count is $day")
   }
   for ((day, count) <- days.zipWithIndex) { 
        println(s"$count is $day")
   }
   /*  output:
        0 is Sunday
        1 is Monday
        2 is Tuesday
        3 is Wednesday
        4 is Thursday   
        .....  */
   ```
2. When using zipWithIndex, the counter always starts at 0. You can also use the zip method with a Stream to create a counter. This gives you a way to control the starting value:
   ``` 
   for ((day,count) <- days.zip(Stream from 1)) {
       println(s"day $ count is $day")
       }
   ```
3. When zipWithIndex is used on a sequence, it returns a sequence of Tuple2 elements
   ``` 
   val list = List("a", "b", "c")
   val zwi = list.zipWithIndex
   // zwi: List[(String, Int)] = List((a,0), (b,1), (c,2))
   ```
#### Transforming One Collection to Another with for/ yield
1. In general, the collection type that’s returned by a for comprehension will be the same type that you begin with. If you begin with an ArrayBuffer, you’ll end up with an ArrayBuffer.However, this isn’t always the case.
#### Transforming one collection to another with map
1. For simple cases, using map is the same as using a basic for/yield loop
2. But once you add a guard, a for/yield loop is no longer directly equivalent to just a map method call. If you attempt to use an if statement in the algorithm you pass to a map method, you’ll get a very different result:
   ``` 
   scala> val fruits = List("apple", "banana", "lime", "orange", "raspberry")
   scala> val newFruits = fruits.map( f =>
            if (f.length < 6) f.toUpperCase)
   newFruits: List[Any] = List(APPLE, (), LIME, (), ())
   
   //here we could filter the result after calling map to clean up the result:
   scala> newFruits.filter(_ != ()) 
   res0: List[Any] = List(APPLE, LIME)
   
   //in this situation, it helps to think of an if statement as being a filter, so the correct solution is to first filter the collection, and then call map:
   scala> val fruits = List("apple", "banana", "lime", "orange", "raspberry") 
   fruits: List[String] = List(apple, banana, lime, orange, raspberry)
   scala> fruits.filter(_.length < 6).map(_.toUpperCase) 
   res1: List[String] = List(APPLE, LIME)
   ```
#### Flattening a list of lists with flatten
1. Because an Option can be thought of as a container that holds zero or one elements, flatten has a very useful effect on a sequence of Some and None elements. It pulls the values out of the Some elements to create the new list, and drops the None elements.
   ```
   val x = Vector(Some(1), None, Some(3), None)
   x.flatten
   //res1: Vector[Int] = Vector(1, 3)
   
   ```
#### Combining map and flatten with flatMap
1. Use flatMap in situations where you run map followed by flatten. 
   + You’re using map (or a for/yield expression) to create a new collection from an existing collection.
   + The resulting collection is a list of lists.
   + You call flatten immediately after map (or a for/yield expression).
   + When you’re in this situation, you can use flatMap instead.
#### Using filter to filter a collection
1. filter returns all elements from a sequence that return true when your function/predicate is called. There’s also a filterNot method that returns all elements from a list for which your function returns false.
   + filter walks through all of the elements in the collection; some of the other methods stop before reaching the end of the collection.
   + filter lets you supply a predicate (a function that returns true or false) to filter the elements.
#### Extracting a sequence of elements from a collection
1. There are quite a few collection methods you can use to extract a __*contiguous* list__ of elements from a sequence, including drop, dropWhile, head, headOption, init, last, lastOption, slice, tail, take, takeWhile.
#### Splitting sequences into subsets
1. groupBy, partition, span, splitAt; sliding, unzip
#### Walking through a collection with the reduce and fold methods
1. [Methods may have multiple parameter lists. Its use cases are 3 of the following:
   SINGLE FUNCTIONAL PARAMETER, IMPLICIT PARAMETERS, PARTIAL APPLICATION](https://docs.scala-lang.org/tour/multiple-parameter-lists.html)
2. reduceLeft, foldLeft, reduceRight, foldRight, scanLeft, scanRight
   + ```reduceLeft``` walks through the sequence from left to right, it starts by applying the function to the first two elements, then apply the result to third element, and so on.
   ``` 
   val a = Array(12, 6, 15)
   val b = a.reduceLeft(_ + _)      // Or         a.reduceLeft((x, y) => x + y)
   //b = 33
   ```
   + ```reduceRight``` walks through from right to left. In many cases, ```reduceLeft``` and ```reduceRight``` make no difference; we can use ```reduce``` in this case. 
   ``` 
   //in some cases, reduceLeft and reduceRight are totally different
   val a = Array(1, 2 , 4)
   val b = a.reduceLeft(_ / _)
   //b = 1 / 8
   val c = a.reduceRight(_ / _)
   //c = 2
   ```
   + ```foldLeft``` works just like ```reduceLeft``` except that ```foldLeft``` provides a starting value
   ```
   val a = Array(1, 2, 3)
   val b = a.foldLeft(10)(_ + _)
   //b = 16
   ```
#### Extracting unique elements from a sequence
1. The distinct method returns a new collection with the duplicate values removed. Remember to assign the result to a new variable. This is required for both immutable and mutable collections.
2. If you happen to need a Set, converting the collection to a Set is another way to remove the duplicate elements. 
#### Merging sequential collections
1. Join two sequences into one sequence, either keeping all of the original elements, finding the elements that are common to both collections, or finding the difference between the two sequences:
   + ++=
   + ++
   + union, diff, intersect
#### Merging two sequential collections into pairs with zip
1. Use the zip method to join two sequences into one, the following code block creates an Array of Tuple2 elements, which is a merger of the two original se‐ quences.Once you have a sequence of tuples like couples, you can convert it to a Map, which may be more convenient.
   ```
   val women = List("Wilma", "Betty")
   val men = List("Fred", "Barney")
   val couples = women zip men
   couples: List[(String, String)] = List((Wilma,Fred), (Betty,Barney))
   
   val couplesMap = couples.toMap
   ```
2. If one collection contains more items than the other collection, the items at the end of the longer collection will be dropped.
#### Creating a lazy view on a collection
1. Except for the Stream class, whenever you create an instance of a Scala collection class, you’re creating a strict version of the collection. This means that if you create a collection that contains one million elements, memory is allocated for all of those elements im‐ mediately.
2. In Scala you can optionally create a view on a collection. A view makes the result non‐ strict, or lazy. This changes the resulting collection, so __when it’s used with a transformer method__, the elements will only be calculated as they are accessed, and not “eagerly,” as they normally would be.
3. Use cases
   + Performance: Regarding performance, assume that you get into a situation where you may (or may not) have to operate on a collection of a billion elements. You certainly want to avoid running an algorithm on a billion elements if you don’t have to, so using a view makes sense here.
   + To treat a collection like a database view: Changing the elements in the array updates the view, and changing the elements referenced by the view changes the elements in the array. When you need to modify a subset of elements in a collection, creating a view on the original collection and modifying the elements in the view can be a powerful way to achieve this goal.
4. As a final note, don’t confuse using a view with saving memory when creating a collection. The benefit of using a view in regards to performance comes with how the view works with transformer methods.
#### Populating a collection with a range
1. Call the range method on sequences that support it, or create a Range and convert it to the desired sequence. 
2. In the first approach, the range method is available on the companion object of supported types like Array, List, Vector, ArrayBuffer, and others.
3. The REPL shows the collections that can be created directly from a Range: toArray, toList, toString, toBuffer, toIndexedSeq, toIterable, toIterator, toMap, toSeq, toSet, toStream, toTraversable. Using this approach is useful for some collections, like Set, which don’t offer a range method:
   ``` 
   //intentional error, value range is not a member of object Set
   val set = Set.range(0, 5)
   ```
#### Creating and using enumerations
1. Extend the scala.Enumeration class to create your enumeration:
   ```
   package com.acme.app {
        object Margin extends Enumeration {
                type Margin = Value
                val TOP, BOTTOM, LEFT, RIGHT = Value
                }
        }
   ```
#### Tuples, for when you just need a bag of things
1. A tuple gives you a way to store a group of heterogeneous items in a container:
   ``` 
   case class Person(name: String)
   val t = (3, "Three", new Person("Al"))
   
   scala> t._1
   res1: Int = 3
   scala> t._2
   res2: java.lang.String = Three
   scala> t._3
   res3: Person = Person(Al)
   ```
2. You can also convert a tuple to a collection
   ```
   val t = ("Al", "Alabama")
   t.productIterator.toArray
   //res0: Array[Any] = Array(AL, Alabama)
   ```
#### Sorting a collection
1. The sorted method can sort collections with type Double, Float, Int, and any other type that has an implicit scala.math.Ordering
2. The sortWith method lets you provide your own sorting function.
3. Mix the Ordered trait into the class, and implement a _compare_ method; then you can use the class with the sorted method.
4. An added benefit of mixing the Ordered trait into your class is that it also lets you compare object instances directly in your code; This works because the Ordered trait implements the <=, <, >, and >= methods, and calls your compare method to make those comparisons.
   ```
   if (al > ty) println("Al") else println("Tyler")
        ^^^^^
   ```
#### Converting a collection to a String with mkString
1. Use the mkString method to print a collection as a String.
   ```
   val a = Array("apple", "banana", "cherry")
   
   scala> a.mkString
   res1: String = applebananacherry
   
   scala> a.mkString(" ")
   res2: String = apple banana cherry
   
   scala> a.mkString("[", ", ", "]")
   res4: String = [apple, banana, cherry]
   ```
2. You can also use the toString method on a collection, but it returns the name of the collection with the elements in the collection listed inside parentheses
   ``` 
   val v = Vector("apple", "banana", "cherry")
   v.toString
   //res0: String = Vector(apple, banana, cherry)
   ```
### Ch11 List, Array, Map, Set(and More)
#### list
1. List immutable
2. ListBuffer mutable
3. Stream, a lazy version of a list
4. Array, its elements can be changed, its size cannot be changed
5. ArrayBuffer, completely mutable array
6. Multidimensional Arrays: 
   ``` 
   //approach 1, ofDim is unique to the Array class, there is no such method
   //on a List Vector ArrayBuffer,etc.
   Array.ofDim(x)(y)(z)
   //approach 2: array of arrays; allows you to create ragged array; you can
   //use this technique to create list of list, vector of vectors,etc.
   val a = Array(Array("a", "b", "c"), Array("d", "e"))
   ```
7. Map: 
   + auto imported Map is the immutable Map;
   + to use the mutable Map, use ```collection.mutable.Map("AL" -> "Alabama")```
   + SortedMap; LinkedHashMap; ListMap
   + accessing maps: 
   
   ``` 
   //approach 1
   val states = Map("AL" -> "Alabama").withDefaultValue("Not found")
   states("foo") // Not found
   //approach 2
   val s = states.getOrElse("FOO", "No such state")  //No such state
   //approach 3
   val az = states.get("AZ") // Some(Arizona)
   val za = states.get("FOO") // None
   ```
   
   + To get the keys, use ```keySet``` to get the keys as a Set, ```keys``` to get an Iterable, or ```keysIterator``` to get the keys as an iterator; To get the values from a map, use the ```values``` method to get the values as an Iterable, or ```valuesIterator``` to get them as an Iterator
   + To test for the existence of a key in a map, use the contains method; To test whether a value exists in a map, use the valuesIterator method to search for the value using exists and contains
   + filtering maps, sorting maps by key or value; finding the largest key or value
8. Set
   + Set, mutable Set, SortedSet(when used with your own classes, you have to extend the ordered trait, and implement a compare method)
9. Queue: FIFO, immutable, mutable
10. Stack: LIFO, immutable, mutable
11. Range
### Ch13 Actors and concurrency
#### Check out [Akka.io documentations](https://akka.io/docs/)

### Ch18 The Simple Build Tool
#### Creating a project directory structure for SBT
1. Use a shell script
#### Compiling, Running, and Packaging a scala project with SBT
1. Use ```sbt``` in command line to enter SBT interactive mode; this improves the speed of the compile run package,etc. processes.
2. You can run multiple commands at one time, such as: ```> clean compile```
3. Descriptions of most common SBT commands:

   | Command                 |        Description                                                           |
   |-------------------------|------------------------------------------------------------------------------|
   | clean                   |Removes all generated files from the target directory.                        |
   | compile                 |Compiles source code files that are in src/main/scala, src/main/java, and the |
   |                         |root directory of the project.                                                |
   | ~ compile               |Automatically recompiles source code files while you’re running  SBT in       |
   |                         | interactive mode (i.e., while you’re at the SBT command prompt).             |
   | help <command>          | help list common commands. when given a command, provides description        |
   |package                  |Creates a JAR file (or WAR file for web projects) containing  the files in    |
   |                         |src/main/scala, src/main/java, and resources in src/main/resources.           |
   |publish                  |Publishes your project to a remote repository.                                |
   |publish-local            |Publishes your project to a local Ivy repository.                             |
   |reload                   |                                                                              |
   |run                      |compile and run                                                               |
   |test                     |compile and run all tests                                                     |
   |update                   |updates external dependencies                                                 |
4. ```last <last command>``` prints logging information for the last command that was executed. This can help you understand what’s happening, including understanding why some‐ thing is being recompiled over and over when using incremental compilation.
   + Typing help last in the SBT interpreter shows a few additional details, including a note about the last-grep(deprecated, replaced with lastGrep) command, which can be useful when you need to filter a large amount of output.
#### Managing dependencies with SBT
1. unmanaged dependencies(JAR files) are added to the lib folder in the root directory of SBT project.
2. conventions:
   ``` libraryDependencies += groupID % artifactID % revision
       
       libraryDependencies += groupID % artifactID % revision % configuration
       
       libraryDependencies += groupID %% artifactID % revision
       
       libraryDependencies ++= Seq(
            groupID1 % artifactID1 % revision1,
            groupID2 % artifactID2 % revision2,
            groupID3 % artifactID3 % revision3,
            "org.scalatest" % "scalatest_2.10" % "1.9.1" % "test"
       )
   ```
3. The %% method adds your project’s Scala version to the end of the artifact name. The practice of adding the Scala version  to the artifactID is used because modules may be compiled for different Scala versions.
4. ```libraryDependencies += "org.scalatest" % "scalatest_2.10" % "1.9.1" % "test"``` this means that the dependency you’re defining “will be added to the classpath only for the Test configuration, and won’t be added in the Compile configuration. This is useful for adding dependencies like ScalaTest, specs2, Mockito, etc., that will be used when you want to test your application, but not when you want to compile and run the application.
#### Controlling which version of a managed dependency is used
1. The ```revision``` field in the libraryDependencies setting isn’t limited to specifying a single, fixed version. According to the Apache Ivy documentation, you can specify terms such as latest.integration, latest.milestone, and other terms.
2. The Ivy dependency documentation states that the following tags can be used:
   + ```latest.[any status]``` such as ```latest.milestone```
   + end the revision with a ```+``` character. This selects the latest subrevision of the dependency module. For instance, if the dependency module exists in revisions 1.0.3, 1.0.7, and 1.1.2, specifying 1.0.+ as your dependency will result in 1.0.7 being selected.
   + [version ranges](http://ant.apache.org/ivy/history/2.2.0/ivyfile/dependency.html#revision)
#### Creating a project with subprojects
1. see [SBT Multi-Project documentation](https://www.scala-sbt.org/release/docs/Multi-Project.html) for more instructions.
#### Generating Project API Documentation
1. SBT commands:
   + ```doc``` create scaladoc API documentation from scala source code files in src/main/scala
   + ```test:doc``` create API documentation for src/test/scala
#### Specifying a Main Class to Run
1. when you have multiple main classes, you have to specify which main class for sbt to run
2. one way: add a line to build.sbt file
   ``` 
   //set the main class for 'sbt run'
   mainClass in (Compile, run) := Some("com.Foo")
   
   //set the main class for packaging the main jar
   mainClass in (Compile, packageBin) := Some("com.Foo")
   ```
3. another way: when running application with SBT
   ``` 
   $ sbt "run-main com.Foo"
   //or sbt interactive mode
   $ sbt
   
   > run-main com.Foo
   ```
#### Using GitHub Projects as Project Dependencies
1. example:
   ``` 
   //put the following contents in a file named project/Build.scala in your SBT project
   import sbt._
   
   object MyBuild extends Build {
        lazy val root = Project("root", file("."))
                                .dependsOn(soundPlayerProject)
                                .dependsOn(appleScriptUtils)
        lazy val soundPlayerProject =
        RootProject(uri("git://github.com/alvinj/SoundFilePlayer.git"))
        lazy val appleScriptUtils = 
        RootProject(uri("git://github.com/alvinj/AppleScriptUtils.git"))
   }
   
   ```
2. then you can use the github library. this works fine for compiling and running your project, yet you cannot package all of this code into a JAR file by just using the ```sbt package``` command. SBT  does not include the code from GitHub. you may use a plug-in name sbt-assenbly for this purpose.
#### Telling SBT How to find a repository(Working with resolvers)
1. Use the ```resolvers``` key in the build.sbt file
   ``` 
   // format:
   // resovlers += "repository name" at "location"
   resolvers += "Typesafe" at "http://repo.typesafe.com/typesafe/releases/"
   
   //to add multiple resolvers:
   resolvers ++= Seq(
         "Typesafe" at "http://repo.typesafe.com/typesafe/releases/",
         "Java.net Maven2 Repository" at "http://download.java.net/maven/2/"
   )
   ```
#### Resolving Problems by Getting an SBT stack trace
1. When an SBT command silently fails (typically with a “Nonzero exit code” message), but you can’t tell why, run your command from within the SBT shell, then use the ```last run``` command after the command that failed.
#### Setting the SBT Log Level
1. set the SBT logging level in build.sbt: ```logLevel := Level.Debug```
2. working interactively from SBT command line:
   ```> set logLevel := Level.Debug```
3. other logging levels :
   + ```Level.Info```
   + ```Level.Warning```
   + ```Level.Error```
#### Deploying a Single, Executable JAR File
1. ```sbt package``` command creates a JAR file including class files it compiles from your source code as well as with the resources in the projcet(from src/main/resources); it does not include project dependencies and libraries from the Scala distribution needed to execute JAR file with the ```java``` command.
2. Use an SBT plug-in such as sbt-assembly to build a single, complete JAR file.

### Ch19 Types
#### Variance
1. Type variance is a generic type concept, and defines the rules by which parameterized types can be passed into methods.

   | Symbols         |        Name            | Description                        |
   |-----------------|------------------------|------------------------------------|
   | Array[T]        |     Invarirant         |Used when elements in the container |
   |                 |                        |are mutable                         |
   |                 |                        |eg. can only pass Array[String] to a|
   |                 |                        |method expecting Array[String]      |
   |-----------------|------------------------|------------------------------------|
   | Seq[+A]         |    Covariant           |Used when elements in that container|
   |                 |                        |are immutable                       |
   |                 |                        |eg. can only pass Seq[String] to a  |
   |                 |                        |method expected Seq[Any]            |
   |-----------------|------------------------|------------------------------------|
   | Foo[-A]         |   Contravariant        |Contravariance is essentially the   |
   |                 |                        |opposite of covariance, and is      |
   |Function1[-A, +B]|                        |rarely used.                        |
   |-----------------|------------------------|------------------------------------|
2. Bounds: they let you place restrictions on type parameters.
   + A <: B   Upper bound    A must be a subtype of B
   + A >: B   Lower bound    A must be a supertype of B
   + A <: Upper >: Lower   Lower and upper bounds used together  The type A has both an upper and lower bound
#### Creating classes that use generic types
1. Scala standard is that simple types' naming conventions are A, the next with B,and so on.
   ``` 
   //create a linked-list with generic type A
   class LinkedList[A] { /* ....*/ }
   // a hierachy of classes
   class GrandParent { }
   class Parent { }
   class Child { }
   
   val family = new LinkedList[GrandParent]
   
   val grandpa = new GrandParent
   val papa = new Parent
   val me = new Child
   //the following are all ok
   family.add(grandpa)
   family.add(papa)
   family.add(me)
   ```
2. The last line won’t compile because (a) printPersonTypes wants a LinkedList[GrandParent], (b) LinkedList elements are mutable, and (c) children is a LinkedList[Child]. This creates a conflict the compiler can’t resolve.
   ``` 
   def printPersonTypes(persons: LinkedList[GrandParent]) {
        persons.printAll()
        }
   val children = new LinkedList[Child]
   children.add(me)
   
   //won't compile
   printPersonTypes(children)
   ```
#### Using Duck Typing
1. Scala's version of "Duck Typing" is known as using a structural type.

#### Make mutable collections invariant
1. When creating a collection of elements that can be changed (mutated), its generic type parameter should be declared as [A], making it invariant.
   ``` 
   //code example
   class Array[A] ...
   class ArrayBuffer[A] ...
   
   trait Animal {
        def speak
        }
   class Dog(var name: String) extends Animal {
        def speak { println("woof")
        override def toString = name
        }
   class SuperDog(name: String) extends Dog(name) {
        def useSuperPower { println("Using my superpower!") }
        }
        
   val fido = new Dog("Fido")
   val wonderDog = new SuperDog("Wonder Dog")
   val shaggy = new SuperDog("Shaggy")
   val dogs = ArrayBuffer[Dog]()
   dogs += fido
   dogs += wonderdog
   
   import collection.mutable.ArrayBuffer
   def makeDogsSpeak(dogs: ArrayBuffer[Dog]) {
        dogs.foreach(_.speak)
        }
   val superDogs = ArrayBuffer[SuperDog]()
   superDogs += shaggy
   superDogs += wonderdog
   makeDogsSpeak(superDogs)  //ERROR: won't compile
   ```
2. The last line won't compile because of the conflict built up in this situation
   + Elements in an ArrayBuffer can be mutated.
   + makeDogsSpeak is defined to accept a parameter of type ArrayBuffer[Dog].
   + You’re attempting to pass in superDogs, whose type is ArrayBuffer[SuperDog].
   + If the compiler allowed this, makeDogsSpeak could replace SuperDog elements in superDogs with plain old Dog elements. This can’t be allowed.
   
#### Make immutable collections covariant
1. You can define a collection of immutable elements as invariant, but your collection will be much more flexible if you declare that your type parameter is covariant. To make a type parameter covariant, declare it with the + symbol, like [+A].
2. Had we defined the ```makeDogsSpeak``` as ```def makeDogsSpeak(dogs: Seq[Dog])```, that ```makeDogsSpeak(superDogs)``` would compile; because scala defined Seq as ```trait Seq[+A]```: Seq is immutable and defined with a covariant parameter type.
3. If we change definition of ```Seq[+A]``` to ```Seq[A]```, the last line still won't compile, because type A is invariant.

#### Create a collection whose elements are all of some base type
1. Define the class or method by specifying the type parameter with an upper bound.
   ```
   trait Basetrait
   class SubClass1 extends Basetrait
   class SubClass2 extends Basetrait
   trait Othertrait1
   trait Othertrait2
   
   class Crew[A <: Basetrait] extends ArrayBuffer[A]
   
   val class1s = new Crew[SubClass1]()
   val class2s = new Crew[SubClass2]()
   
   class Crew[A <: Basetrait with Othertrait1] extends ArrayBuffer[A]
   val class11 = new SubClass1 with Othertrait1
   val class1ss = new Crew[SubClass1 with Othertrait1]()
   class1ss += class11
   ```
#### Selectively Adding New Behavior to a Closed Model
1. Here is the problem: You have a closed model, and want to add new behavior to certain types within that model, while potentially excluding that behavior from being added to other types.
2. Here is one solution: you can implement your solution as a type class.
### Ch20 Idioms
#### Create methods with no side effects(Pure functions)
1. Pure function; referential transparency
2. several statements about pure functions:
   + A pure function is given one or more input parameters.
   + Its result is based solely off of those parameters and its algorithm. The algorithm will not be based on any hidden state in the class or object it’s contained in.
   + It won't mutate the parameters it’s given.
   + It won't mutate the state of its class or object.
   + It doesn't perform any I/O operations, such as reading from disk, writing to disk, prompting for input, or reading input.
#### Prefer immutable objects
1. Prefer immutable collections. For instance, use immutable sequences like List and Vector before reaching for the mutable ArrayBuffer.
2. Prefer immutable variables. That is, prefer val to var.
#### Think "Expression-Oriented Programming"
1. Difference between statements and expressions:
   + Statements do not return results and are executed solely for their side effects, while expressions always return a result and often do not have side effects at all.
2. EOP
   + An expression-oriented programming language is a programming language where every (or nearly every) construction is an expression, and thus yields a value.
#### Use Match Expressions and pattern matching
1. match expression are used in many situations:
   + In try/catch expressions
   + As the body of a function or method
   + With the Option/Some/None coding pattern
   + In the receive method of actors
#### Eliminate null values from your code
1. "Ban ```null``` from any of your code. Period."
2. the following demonstrates how to not use ```null``` values in different situations:
   + When a var field in a class or method doesn’t have an initial default value, initialize it with Option instead of null.
   + When a method doesn’t produce the intended result, you may be tempted to return null. Use an Option or Try instead.
   + If you’re working with a Java library that returns null, convert it to an Option, or something else.
3. if you want the error information instead of a Some or None, use the Try/ Success/ Failure approach instead:
   ``` 
   import scala.util.{Try, Success, Failure}
   
   object Test extends App {
        def readTextFile(filename: String): Try[List[String]] = {
            Try(io.Source.fromFile(filename).getLines.toList)
        }
   val filename = "/etc/passwd" 
   readTextFile(filename) match {
        case Success(lines) => lines.foreach(println)
        case Failure(f) => println(f)
      }
   }
   ```
#### Using the Option/Some/None pattern
1. Getting the value from an Option:
   + use ```getOrElse```
   + use ```foreach```
   + use a match expression
2. Use Try, Success, and Failure: 
   + Scala 2.10 introduced scala.util.Try as an approach that’s similar to Option, but re‐ turns failure information rather than a None.
   + The result of a computation wrapped in a Try will be one of its subclasses: Success or Failure. If the computation succeeds, a Success instance is returned; if an exception was thrown, a Failure will be returned, and the Failure will hold information about what failed.
3. Prior to Scala 2.10,  an approach similar to Try was available with the Either, Left, and Right classes. With these classes, Either is analogous to Try, Right is similar to Success, and Left is similar to Failure.
