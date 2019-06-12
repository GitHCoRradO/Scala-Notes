### Scala Cookbook Notes

#### Ch01  String

##### String interpolators

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
   
##### Replacing Patterns in Strings  (Regex)

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
##### Add your own methods to the String class

#### Ch02 Numbers

##### Comparing floating-point numbers

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

##### Generating random numbers

1. [how to create a list of alpha or alphanumeric characters](http://alvinalexander.com/scala/create-list-alpha-alphanumeric-characters-in-scala)

2. [generating random strings](http://alvinalexander.com/scala/creating-random-strings-in-scala)

##### Formatting numbers and currency

1. [The Joda Money library is a Java library for handling currency](https://www.joda.org/joda-money/)

#### Ch03 Control Structures

##### Looping for and foreach

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
    
##### breaks; match case
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

#### Ch04 Classes and Properties

##### The effect of constructor parameter settings

    | Visibility        |     Accessor?      |  Mutator?     |
    |-------------------|--------------------|---------------|
    | var               |  yes               | yes           |
    |-------------------|--------------------|---------------|
    | val               |  yes               | no            |
    |-------------------|--------------------|---------------|
    | Default visibility|  no                | no            |
    |(no val or var)    |                    |               |
    |-------------------|--------------------|---------------|
    |Adding private     | no                 |  no           |
    |keyword to val,var |                    |               |
    |-------------------|--------------------|---------------|

##### Case class
1. Case class constructor parameters are val by default.
   So if you define a case class field without adding val or var,
   you can still access the field, just as if it were defined as a val.
   
##### companion object
1. A companion object is simply an object that’s defined in the same file as a class, where the object and class have the same name. If you declare a class named Foo in a file named Foo.scala, and then declare an object named Foo in that same file, the Foo object is the compan‐ ion object for the Foo class. A companion object has several purposes, and one purpose is that any method declared in a companion object will appear to be a static method on the object. 

##### Preventing getter and setter methods from being generated
1. To do this, define the field with *private* or *private[this]* access modifiers: with *private*, the field is only available to instances of the same class; with *private[this]*, the field can only be accessed by the instance of the class itself, not available by other instances of the same type.

##### Assigning a field to a block or function
1. Define a field to be *lazy* makes it not evaluated until accessed; it is a useful approach when the field might not be accessed in the normal processing of your algorithms, or if running the algorithm will take a long time, and you want to defer that to a later time.
##### Auxiliary classes; Calling a superclass constructor
1. the first line of an auxiliary constructor must be a call to another constructor of the current class, there is no way for auxiliary con‐ structors to call a superclass constructor.
2. the primary constructor of the Employee (sub) class can call any constructor in the Person (base) class, but the auxiliary constructors of the Employee class must call a previously defined constructor of its own class with the this method as its first line.
3. Therefore, there’s no direct way to control which superclass constructor is called from an auxiliary constructor in a subclass. In fact, because each auxiliary constructor must call a previously defined constructor in the same class, all auxiliary constructors will eventually call the same superclass constructor that’s called from the subclass’s primary constructor.
##### Defining case classes
1. defining case classes causes the following codes to be automatically generated:
   1. apply method, so you need not new keyword when creating new instances of the class.
   2. accessors and mutators for parameters.
   3. a good, default toString method.
   4. unapply method for use in match expressions
   5. copy method
##### Defining an equals method (Object equality)
1. in Scala, == is a method you use on each class to compare the equality of two instances, calling your equals method under the covers.
2. The Scaladoc for the equals method of the Any class states, “any implementation of this method should be an equivalence relation.” The documentation states that an equiva‐ lence relation should have these three properties:
   1. x == x true
   2. x == y true then y == x true
   3. x == y true and y == z true then x == z true
##### Inner class
1. inner classes are bound to its outer objects

#### Ch05 Methods
##### Controlling method scopes
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

##### Setting default values for method parameters
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
##### Defining a method that returns multiple items(tuples)
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

##### Forcing Callers to Leave Parentheses off Accessor Methods
1. Define your getter/accessor method without parentheses after the method name, This forces consumers of your class to call the accessor method without parentheses

##### Creating Methods That Take Variable-Argument Fields
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

##### declaring that a method can throw an exception
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
#### Ch06 Objects
 
##### Object casting
1. Use the asInstanceOf method to cast from one type to another.
    ``` 
    val cat = Cat("meow")
    val catAsAnimal = cat.asInstanceOf[Animal]
    ```
##### classOf method
1. classOf method is the scala equivalent of Java's .class; called on Class names
   ``` 
   val stringClass = classOf[String]
   stringClass.getMethods
   ```
##### Determine the class of an Object
1. call getClass on an object returns the class name of the object:
   ``` 
   scala> def printClass(c: Any) { println(c.getClass) } printClass: (c: Any)Unit
   scala> printClass(1) class java.lang.Integer
   scala> printClass("yo") class java.lang.String
   ```
##### Lauching an application with an object
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
##### Creating static members with companion objects
1. Here is how class and its companion object is defined:
   1. Define your class and object in the same file, giving them the same name.
   2. Define members that should appear to be "static" in the object
   3. Define nonstatic(instance) members in the class
2. a class and its companion object can access each other’s private members.

