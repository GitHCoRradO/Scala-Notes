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