## Neophyte's Guide to Scala Notes

### Part 1: Extractors
#### Several good examples:
   ``` 
   //#1
   case class User(firstName: String, lastName: String, score: Int)
   def advance(xs: List[User]) = xs match {
     case User(_, _, score1) :: User(_, _, score2) :: _ => score1 - score2
     case _ => 0
   }
   //#2
   trait User {
     def name: String
     def score: Int
   }
   class FreeUser(val name: String, val score: Int, val upgradeProbability: Double)
     extends User
   class PremiumUser(val name: String, val score: Int) extends User
   
   object FreeUser {
     def unapply(user: FreeUser): Option[(String, Int, Double)] =
       Some((user.name, user.score, user.upgradeProbability))
   }
   object PremiumUser {
     def unapply(user: PremiumUser): Option[(String, Int)] = Some((user.name, user.score))
   }
   
   val user: User = new FreeUser("Daniel", 3000, 0.7d)
   user match {
     case FreeUser(name, _, p) =>
       if (p > 0.75) name + ", what can we do for you today?" else "Hello " + name
     case PremiumUser(name, _) => "Welcome back, dear " + name
   }
   ```
### Part 2: Extracting Sequences
#### Several good examples:
   ``` 
   //#1 use a pattern that only matches a list of exactly 2 elements, or a list of exactly 3 elements
   val xs = 3 :: 6 :: 12 :: Nil
   xs match {
     case List(a, b) => a * b
     case List(a, b, c) => a + b + c
     case _ => 0
   }
   //#2
   object GivenNames {
     def unapplySeq(name: String): Option[Seq[String]] = {
       val names = name.trim.split(" ")
       if (names.forall(_.isEmpty)) None else Some(names)
     }
   }
   def greetWithFirstName(name: String) = name match {
     case GivenNames(firstName, _*) => "Good morning, " + firstName + "!"
     case _ => "Welcome! Please make sure to fill in your name!"
   }
   ```
### Part 3: Patterns Everywhere
#### Patterns and guard clause:
   ``` 
   //#1
   case class Player(name: String, score: Int)
   
   def printMessage(player: Player) = player match {
     case Player(_, score) if score > 100000 => println("Get a job, dude!")
     case Player(name, _) => println("Hey " + name + ", nice to see you again!")
   }
   ```
#### Patterns in value definitions
   ``` 
   //#1
   val Player(name, _) = currentPlayer()
   doSomethingWithTheName(name)
   //#2
   def gameResult(): (String, Int) = ("Daniel", 3500)
   val result = gameResult()
   println(result._1 + ": " + result._2)
   // It’s safe to destructure our tuple in the value definition, as we know we are dealing with a Tuple2
   val (name, score) = gameResult()
   println(name + ": " + score)
   ```

#### Patterns in for comprehensions
   ``` 
   //#1
   def gameResults(): Seq[(String, Int)] =
     ("Daniel", 3500) :: ("Melissa", 13000) :: ("John", 7000) :: Nil
   
   def hallOfFame = for {
     result <- gameResults()          //       (name, score) <- gameResults()
     (name, score) = result           //       if (score > 5000) 
     if (score > 5000)
   } yield name
   ```
1. Explanations for the following code block
   ``` 
   val lists = List(1, 2, 3) :: List.empty :: List(5, 3) :: Nil
   
   for {
     list @ head :: _ <- lists
   } yield list.size
   ```
   + lists = List(List(1, 2, 3), List(), List(5, 3))
   + pattern ```head :: _``` matches List with at least 1 elements, i.e. not empty; head is matched with head of the List
   + as for the [Pattern Matching `@` Symbol](https://stackoverflow.com/questions/20748858/pattern-matching-symbol): it makes list refer to the object ```head :: _``` itself
### Part 4: Pattern Matching Anonymous Functions
#### pattern matching anonymous functions
1. A pattern matching anonymous function is an anonymous function that is defined as a block consisting of a sequence of cases, surrounded as usual by curly braces, but without a ```match``` keyword before the block:
   ``` 
   //nornal anonymous functions:
   val wordFrequencies = ("habitual", 6) :: ("and", 56) :: ("consuetudinary", 2) ::
     ("additionally", 27) :: ("homely", 5) :: ("society", 13) :: Nil
   def wordsWithoutOutliers(wordFrequencies: Seq[(String, Int)]): Seq[String] =
     wordFrequencies.filter(wf => wf._2 > 3 && wf._2 < 25).map(_._1)
   wordsWithoutOutliers(wordFrequencies) // List("habitual", "homely", "society")
   
   //pattern matching anonymous funcstions:
   def wordsWithoutOutliers(wordFrequencies: Seq[(String, Int)]): Seq[String] =
     wordFrequencies.filter { case (_, f) => f > 3 && f < 25 } map { case (w, _) => w }
   ```
#### Partial functions
1. In short, partial function is a unary function that is known to be defined only for certain input values and that allows clients to check whether it is defined for a specific input value.
   ```
   //simple form
   val pf: PartialFunction[(String, Int), String] = {
        case (word, freq) if freq > 3 && freq < 25 => word
        }
   //explicitly extending the PartialFunction trait
   val pf = new PartialFunction[(String, Int), String] {
        def apply(wordFrequency: (String, Int)) = wordFrequency match {
            case (word, freq) if freq > 3 && freq < 25) => word
            }
        def isDefinedAt(wordFrequency: (String, Int)) = wordFrequency match {
            case (word, freq) if freq >3 && freq < 25 => true
            case _ => false
            }
        }
   ```
### Part 5: The Option Type
#### Option type
1. Option types can be equally viewed as a Collection just like other collections List, Map, Set, Seq,etc. In this regard, Option is like a container of no element or exactly one element.
#### Pattern matching
   ``` 
   val user = User(2, "Johanna", "Doe", 30, None)
   val gender = user.gender match {
     case Some(gender) => gender
     case None => "not specified"
   }
   println("Gender: " + gender)
   ```
#### As with other collections, Option can be used with Map,flatMap, for comprehension
   ``` 
   for {
     User(_, _, _, _, Some(gender)) <- UserRepository.findAll
   } yield gender
   ```
### Part 6 Error handling with Try
#### The semantics of Try
1. Where Option[A] is a container for a value of type A that may be present or not, Try[A] represents a computation that may result in a value of type A, if it is successful, or in some Throwable if something has gone wrong. Instances of such a container type for possible errors can easily be passed around between concurrently executing parts of your application.
2. There are two different types of Try: If an instance of Try[A] represents a successful computation, it is an instance of Success[A], simply wrapping a value of type A. If, on the other hand, it represents a computation in which an error has occurred, it is an instance of Failure[A], wrapping a Throwable, i.e. an exception or other kind of error.
3. Working with Try values:
   ``` 
   // If the given url is syntactically correct, this will be a Success[URL]. If the URL constructor throws a MalformedURLException, however, it will be a Failure[URL]
   import scala.util.Try
   import java.net.URL
   def parseURL(url: String): Try[URL] = Try(new URL(url))
   
   //you can check if a Try is a success by calling isSuccess on it and then
   // conditionally retrieve the wrapped value by calling get on it
   //it is also possible to use getOrElse to pass in a default value to be
   //returned if the Try is a Failure
   val url = parseURL(Console.readLine("URL: ")) getOrElse new URL("http://duckduckgo.com")
   
   ```
#### Chaining operations
1. Like Option, Try type supports all the higher-order methods from other types of collections.
   + mapping and flat mapping
   ``` 
   import java.io.InputStream
   //return type Try[Try[Try[InputStream]]]
   def inputStreamForURL(url: String): Try[Try[Try[InputStream]]] = parseURL(url).map { u =>
     Try(u.openConnection()).map(conn => Try(conn.getInputStream))
   }
   //return type Try[InputStream]
   def inputStreamForURL(url: String): Try[InputStream] = parseURL(url).flatMap { u =>
     Try(u.openConnection()).flatMap(conn => Try(conn.getInputStream))
   }
   ```
   + filter and foreach
   ``` 
   def parseHttpURL(url: String) = parseURL(url).filter(_.getProtocol == "http")
   parseHttpURL("http://apache.openmirror.de") // results in a Success[URL]
   parseHttpURL("ftp://mirror.netcologne.de/apache.org") // results in a Failure[URL]
   //The function passed to foreach is executed only if the Try is a Success, which allows you to execute a side-effect. The function passed to foreach is executed exactly once in that case, being passed the value wrapped by the Success
   parseHttpURL("http://danielwestheide.com").foreach(println)
   ```
   + pattern matching
   ``` 
   import scala.util.Success
   import scala.util.Failure
   getURLContent("http://danielwestheide.com/foobar") match {
     case Success(lines) => lines.foreach(println)
     case Failure(ex) => println(s"Problem rendering URL content: ${ex.getMessage}")
   }
   ```
2. Recovering from a failure
   ``` 
   import java.net.MalformedURLException
   import java.io.FileNotFoundException
   val content = getURLContent("garbage") recover {
     case e: FileNotFoundException => Iterator("Requested page does not exist")
     case e: MalformedURLException => Iterator("Please make sure to enter a valid URL")
     case _ => Iterator("An unexpected error has occurred. We are so sorry!")
   }
   ```
3. Other methods supported by Try type: orElse, transform, recoverWith,etc.(worth looking at)
### Part 7: The Either Type
#### The semantics
1. Like Option and Try, Either is a container type: An Either[A, B] instance can contain either an instance of A, or an instance of B.
2. Either has exactly two sub types, Left and Right. If an Either[A, B] object contains an instance of A, then the Either is a Left. Otherwise it contains an instance of B and is a Right.
3. error handling is a popular use case for it, and by convention, when using it that way, the Left represents the error case, whereas the Right contains the success value.
   ``` 
   //createing an Either
   import scala.io.Source
   import java.net.URL
   def getContent(url: URL): Either[String, Source] =
     if (url.getHost.contains("google"))
       Left("Requested URL is blocked for the good of the people!")
     else
       Right(Source.fromURL(url))
   //you can ask an instance of Either if it isLeft or isRight; you can
   //also do pattern matching on it
   getContent(new URL("http://google.com")) match {
     case Left(msg) => println(msg)
     case Right(source) => source.getLines.foreach(println)
   }
   
   ```
#### Projections
1. call left or right on an Either value, you get a LeftProjection or RightProjection; then you can call map, flatmap on the projection.
2. If you want to transform an Either value regardless of whether it is a Left or a Right, you can do so by means of the fold method that is defined on Either, expecting two transform functions with the same result type, the first one being called if the Either is a Left, the second one if it’s a Right.
3. **Be careful when dealing with for comprehensions**
4. call toOption on one of Either's Projection to get an Option
#### When to use Either
1. Error handling; processing collections

### Part 8: Welcome to the Future
#### Semantics of Future
1. ```Future``` is a write-once container: after a future has been completed, it is effectively immutable. The ```Future``` type only provides an interface for reading the value to be computed. The task of writing the computed value is achieved via a ```Promise```.
#### Composing futures
1. ```Future``` is a container type: you can map, flatMap, filter and use for comprehensions on ```Future```.
2. Sometimes, you may want to be able to work in this nice functional way for the timeline in which things go wrong. By calling the failed method on an instance of ```Future[T]```, you get a failure projection of it, which is a Future[Throwable]. Now you can map that ```Future[Throwable]```, for example, and your mapping function will only be executed if the original ```Future[T]``` has completed with a failure.
### Part 9: Promises and Futures in Practice
#### Future-based programming in practice
1. Non-blocking IO: web application talks to databases, act as a client calling other web services, try to make use of Java's non-blocking IO capabilities,either via Java's NIO API directly or through a library like Netty.
2. Blocking IO & Long-running computations: dedicated thread pool, seperate ```ExecutionContext```.
### Part 10: Staying DRY with higher-order functions
#### Higher-order functions
1. A higher-order function, as opposed to first-order function, can have one of threee forms:
   + One or more of its parameters is a function, and it returns some value.
   + It returns a function, but none of its parameters is a function.
   + Both of the above: One or more of its parameters is a function, and it returns a function.
### Part 11: Currying and Partially Applied Functions
#### Difference between ```PartialFunction``` and parially applied functions:
   + Partially applied function: when applying the function, you do not pass in arguments for all of the parameters defined by the functions,but only for some of them, leaving the remaining one blank. What you get back is a new function whose parameter list only contains those parameters from the original function that were left blank.
   + ```PartialFunction``` type, aka partially defined function, means that its domain is partially defined for input parameters.
### Part 12: Type Classes
### Part 13: Path-dependent types
### Part 14: The Actor approach to concurrency
