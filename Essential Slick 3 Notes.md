## Essential Slick 3 Notes

### Chapter 1 Basics

#### Slick is not an ORM.

#### Queries, Actions(DBIOAction), Futures

### Chapter 2 Selecting Data

#### ```TableQuery``` is the equivalent of the SQL ```select * from table_name```

#### Filtering results: ```filter``` method

#### The ```map``` method
1. Use the ```map``` method on a ```Query``` to select specific columns for inclustion in the results.
   ```
   // select "content" from "message"
   messages.map(_.content)
   
   // select "id", "content" from "message"
   messages.map(m => (m.id, m.content))
   
   case class TextOnly(id: Long, content: String)
   //map sets of columns to Scala data structures using mapTo
   messages.map(m => (m.id, m.content).mapTo[TextOnly])
   ```
2. This all means that ```map``` is a powerful combinator for controlling the SELECT part of your query.

#### Column Expressions
1. Methods like ```filter``` and ```map``` require us to build expressions based on columns in our tables.The ```Rep``` type is used to represent expressions as well as individual columns. Slick provides a variety of extension methods on ```Rep``` for building expressions.
2. Equality(===) and Inequality(=!=) methods: ```=== =!= < > <= >=```
3. String Methods, Numeric Methods, Boolean Methods, Date and Time Methods, Option Methods

#### Controlling Queries: Sort, Take and Drop
1. Methods for ordering, skipping and limiting the results of a query

   | Scala Code         |      SQL Equivalent    |
   |--------------------|------------------------|
   | sortBy             |     ORDER BY           |
   | take               |     LIMIT              |
   | drop               |     OFFSET             |
   
#### Conditional Filtering
1. ```filterOpt``` and ```filterIf```
2. To filter according to a specified condition, if the condition is not provided, then do not filter at all, just select all.
   ``` 
   //nornal way to implement
   def query(name: Option[String]) = 
    messages.filter(msg => msg.sender === name)
   //if we feed the query above with None, you get no results, rather than all the results.
   
   //using filterOpt
   def query(name: Option[String]) =
    messages.filterOpt(name)( (row, value) => row.sender === value )
   //if provided None, then there's no condiotn no restriction on the SQL
   //short-hand version
   def query(name: Option[String]) =
    messages.filterOpt(name)( _.sender === _ )
   ```
3. ```filterIf``` turns a where condtion on or off.
   ``` 
   val hideOldMgs = true
   val query = messages.filterIf(hideOldMgs)(_.id > 100L)
   //condtion on for hideOldMgs being true
   ```
4. chain ```filterOpt``` and ```filterIf``` together
   ```
   val person = Some("Dave")
   val hideOldMsg = true
   val queryToRun = messages.filterOpt(person)(_.sender === _).
            filterIf(hideOlgMsg)(_.id > 100L)
   
   queryToRun.result.statements.mkString
   // res: String = select "sender", "content", "id" from "message" where ("sender" = 'Dave') and ("id" > 100)
   ```
#### Exercises and Good Examples
1. call the ```result.statements``` methods on a query will give you the SQL to be executed.
   ``` 
   val query = messages.filter(_.id === 1L)
   val sql = query.result.statements
   //sql: Iterable[String] = List(select "sender", "content", "id" from "message" where "id" = 1)
   println(sql.head)
   ```
2. First Result: call ```head``` on the ```result```; ```headOption``` for results that may be empty, for ```head``` will throw a run-time exception as we are trying to return the head of an empty collection.
3. The start of something
   + slick implies this for string columns.
   ``` 
   messages.filter(_.coneten startWith "Open")
   ```
4. solution to Liking
   ``` 
   messages.filter(_.content like "%do%")
   //case insensitive
   messages.filter(_.content.toLowerCase like "%do%")
   ```
5. To select a specific string column and append a common string to its ends.
   ```
   exec(messages.map(_.content + "!").result)
   //this is equivalent to 
   // select '(message Ref @421681221).content!' from "message"
   //which is not as expected
   ```
   
   ```
   //these will do
    messages.map(m => m.content ++ LiteralColumn("!"))
    
    messages.map(m => m.content ++ "!")
   ```

### Chapter 3 Creating and Modifying Data
#### Inserting single rows
1. Use ```+=``` method to insert a single row into a table. unlike the select queries we've seen, this creates a DBIOAction immediately without an intermediate Query.
   ``` 
   val action = messages += Message("", "")
   //action: slick.sql.FixedSqlAction[Int,slick.dbio.NoStream,slick.dbio.Effect. Write]
   
   exec(action)
   //res1: Int = 1
   //result of the action is the number of rows inserted.
   ```
#### Retrieving Primary Keys on Insertion
1. When the database allocates primary keys for us it’s oft􏰁en the case that we want get the key back a􏰁fter an insert. Slick supports this via the ```returning``` method
   ``` 
   val insert: DBIO[Long] =
        messages returning messages.map(_.id) += Message("Dave", "Point taken.")
   ```
#### Retrieving Rows on Insertion
1. Some databases(including Oracle and PostgreSQL) allow us to retrieve the complete inserted record, not just the primary key:
   ``` 
   exec(messages returning messages += Message("Dave", "So... what do we do now?"))
   ```
#### Inserting specific columns
1. If our databse contains a lot of columns with default values, it is sometimes useful to specify a subset of columns in our insert queries. We can do this by mapping over a query before calling insert:
   ``` 
   exec(messages.map(m => (m.sender, m.content)) += ("HAL", "Fck this."))
   ```
#### Inserting multiple rows
   ``` 
   val testMessages = Seq( ......... )
   
   exec(message ++= testMessages)
   
   exec(messagesReturningRow ++= testMessages)
   ```
#### Deleting Rows
1. We can only call ```delete``` on a ```TableQuery```
   ``` 
   val removeHal: DBIO[Int] =
        messages.filter(_.sender === "HAL").delete
   exec(removeHal)
   //res: Int = 9        the return value is the number of rows affected.
   ```
   
   ``` 
   //error
   messages.map(_.content).delete
   ```
#### Updating a Single Field and Multiple Fields
   ``` 
   val updateQuery =
        messages.filter(_.sender === "HAL).map(_.sender)
        
   exec(updateQuery.update("HAL 9000"))
   ```
   
   ``` 
   val query = messages.filter(_.id === 1016L).
                map(message => (message.sender, message.content))
   val action: DBIO[Int] =
                query.update(("HAL 9000", "Sure, Dave. Come right inside me."))
   exec(action)
   ```
1. we can even use mapTo to use case classes as the parameter to update.

### Chapter 4 Combining Actions
#### Combinators in Detail
1. ```andThen``` or ```>>>```
   ``` 
   //to run one action after another using andThen; the combined actions are both run, but only the result of the second is returned.
   
   val reset: DBIO[Int] =
        messages.delete andThen messages.size.result
   exec(reset)
   ```
2. ```seq``` method on the ```DBIO``` object; haveing a bunch of actions you want to run, you can use DBIO.seq to combine them; it just like combing the actions with ```andThen``` , but even the last value is discarded.
   ``` 
   val reset: DBIO[Unit] =
        DBIO.seq(messages.delete, messages.size.result)
   ```
3. Mapping over an action is a way to set up a transformation of a value from the database. The transformation will run on the result of the action when it is returned by the database.
   ```
   import scala.concurrent.ExecutionContext.Implicits.global
   val text: DBIO[Option[String]] =
        messages.map(_.content).result.headOption
   val backwards: DBIO[Option[String]] =
        text.map(optionalContent => optionalContent.map(_.reverse))
   exec(backwards)
   ```
   + we have made three uses of ```map``` in the example above
   + an ```Option``` map to apply ```reverse``` to Option[String] result
   + a ```map``` on a query to select just the content column
   + ```map``` on our action so that the result will be transform when the action is run
4. ```flatMap``` gives us the power to sequence actions and decide what we want to do at each step; we give ```flatMap``` a function that depends on the value from an action, and evaluates to another action.
   ```
   val delete: DBIO[Int] = messages.delete
   def insert(count: Int) = messages += Message("NOBODY", s"I removed ${count} messages")
   
   import scala.concurrent.ExecutionContext.Implicits.global
   val resetMessagesAction: DBIO[Int] =
        delete.flatMap { count => insert(count) }
   ```
   
   + beyond sequencing, ```flatMap``` also gives us control over which actions are run. ```flatMap``` can give us arbitrary control over how actions can be combined.
   ``` 
   val resetMessagesAction: DBIO[Int] =
        delete.flatMap {
            case 0 => DBIO.successful(0)
            case n => insert(n)
   ```
5. ```DBIO.sequence```
   