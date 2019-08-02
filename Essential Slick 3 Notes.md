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
1. ```andThen``` or ```>>```
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
6. ```DBIO.fold```
7. ```zip```
8. ```andFinally``` and ```cleanUp```
9. ```asTry```

#### Logging Queries and Results
1. We can enable statement logging by turning up the logging to debug level. There are 5 kinds of Slick loggers. We can configure SLF4J in ```src/main/resources/logback.xml``` to control logging queries.

   | Logger                             |      Will log...                                |
   |------------------------------------|-------------------------------------------------|
   |slick.jdbc.JdbcBackend.statement    |SQL sent to the database                         |
   |slick.jdbc.JdbcBackend.parameter    |Parameters passed to a query.                    |
   |slick.jdbc.StatementInvoker.result  |The first few results of each query.             |
   |slick.session                       |Session events such as opening/closing connec􏰀ons.|
   |slick                               |Everything!                                      |
2. configurations added to ```logback.xml``` are like the following
   ``` 
   <logger name="slick.jdbc.JdbcBackend.statement" level="DEBUG"/>
   ```

#### Transactions
1. Tie sets of modifications together in a transaction so that they either all succeed or all fail; use the ```transactionally``` method.
   ``` 
   def updateContent(old: String) = 
        messages.filter(_.content === old).map(_.content)
   
   exec {
        (updateContent("Affirmative, Dave. I read you.").update("Wanna come in?")
            andThen
         updateContent("Open the pod bay doors, HAL.").update("Pretty please!")
            andThen
         updateContent("I'm sorry, Dave. I'm afraid I can't do that.").update("Opening now")).transactionally
         }
   ```
### Chapter 5 Data Modelling
#### Application Structure
1. Abstracting over Databases; the basic pattern:
   + isolate our database code into a trait(or a few traits)
   + declare the Slick profile as an abstract ```val``` and import from that
   + extend our database trait to make the profile concrete
2. Scaling to Larger Codebases
   ``` 
   trait Profile {
        val profile: JdbcProfile
        }
   trait DatabaseModule1 { self: Profile =>
        import profile.api._
        }
   trait DatabaseModule2 { self: Profile =>
        import profile.api._
        }
   class DatabaseLayer(val profile: JdbcProfile) extends Profile with DatabaseModule1
        with DatabaseModule2
        
   object Main extends App {
        val databaseLayer = new DatabaseLayer(slick.jdbc.H2Profile)
        }
   ```
   
   ``` 
   // TO work with a different database, we inject a different profile when we instantiate the
   // database code
   val anotherDatabaseLayer = new DatabaseLayer(slick.jdbc.PostgresProfile)
   ```
#### Representations for Rows
1. Projections, ```ProvenShapes```, ```mapTo```, and ```<>```
   + single column definition:
   ``` 
   final class MyTable(tag: Tag) extends Table[String](tag, "mytable") { 
         def column1 = column[String]("column1")
         def * = column1
   }
   ```
   + Tuples of database columns map tuples of their type parameters. For example, ```(Rep[String], Rep[Int])``` is mapped to ```(String, Int)```
   ``` 
   final class MyTable(tag: Tag) extends Table[(String, Int)](tag, " mytable") {
            def column1 = column[String]("column1") 
            def column2 = column[Int]("column2") 
            def * = (column1, column2)
   }
   ```
   +  projection operator ```<>```
   ``` 
   case class User(name: String, id: Long)
   
   final class UserTable(tag: Tag) extends Table[User](tag, "user") {
        def id = column[Long]("id", O.PrimaryKey, O.AutoInc)
        def name = column[String]("name")
        def * = (name, id) <> (User.tupled, User.unapply)
   }
   ```
2. The two arguments to ```<>``` are:
   + a func􏰀tion from ```A => B```, which converts from the exis􏰀ting shape’s unpacked row-level encoding ```(String, Long)``` to our preferred representa􏰀tion ```User```;
   + a func􏰀tion from ```B=>Option[A]```, which converts the other way.
3. we can supply these functions by hand if we want:
   ``` 
   def intoUser(pair: (String, Long)): User = User(pair._1, pair._2)
   
   def fromUser(user: User): Option[(String, Long)] = Some((user.name, user.id))
   
   /*    .....     and write the projection function like this      */
   def * = (name, id) <> (intoUser, fromUser)
   ```
4. Heterogeneous Lists, for solving problems with more than 22 columns
   ``` 
   import slick.collection.heterogeneous.{ HList, HCons, HNil }
   import slick.collection.heterogeneous.syntax._
   import scala.language.postfixOps
   
   // defined type alias AttrHList
   type AttrHList =
            Long :: Long :: String :: Int :: String :: 
            Int :: String :: Int :: String :: Int :: String :: 
            Int :: String :: Int :: String :: Int :: String :: 
            Int :: String :: Int :: String :: Int :: String :: 
            Int :: String :: Int :: HNil
   
   final class AttrTable(tag: Tag) extends Table[AttrHList](tag, "attrs") {
         def id = column[Long]("id", O.PrimaryKey, O.AutoInc)
         def column1 = column[String]("column1")
         /* 25 columns go here */
         
         def * = id :: column1 :: /* 22 method names go here */ :: HNil
   }     
   ```
   
   ``` 
   // HList with case class
   case class Attrs(id: Long, column1: String, /* ... */)
   final class AttrTable(tag: Tag) extends Table[Attrs](tag, "attrs") {
        def id = column[Long]("id", O.PrimaryKey, O.AutoInc)
        def column1 = column[String]("column1")
        /* 25 columns go here */
        
        
        /* the pattern is like
         *
         * def * = (some hlist).mapTo[case class with the same fields]
         *
         */
        def * = (id :: column1 :: /* 22 method names go here */ :: HNil).mapTo[Attrs]
        }
   ```
#### Table and Column Representation
1. Nullable Columns: if you want a nullable column you model it in Scala as an Option[T]. To select null columns from tables:

   |Scala Code   | Operand Column types | Result type    |    SQL Equivalent      |
   |-------------|----------------------|----------------|------------------------|
   |col.isEmpty  | Option[A]            | Boolean        | col is null            |
   |col.isDefined| Option[A]            | Boolean        | col is not null        |

   ``` 
   val myUsers = exec(users.filter(_.email.isEmpty).result)
   // translated to SQL:
   // SELECT * FROM "user" WHERE "email" IS NULL
   ```
2. Optional Primary Keys: use this to identify unsaved records(records with None primary key); in real databases there are no records with null primary key; in this case, it is not possible to retrieve records from databases with null primary key; any record will be guaranteed with a primary key on insert into to database. To represent such kind of column, use the following:
   ``` 
   case class User(id: Option[Long], name: String)
   
   class UserTable(tag: Tag) extends Table[User](tag, "user") {
        def id = column[Long]("id", O.PrimaryKey, O.AutoInc)
        def name = column[String]("name")
        
        def * = (id.?, name).mapTo[User]
        //          ^  notice this ? method
        }
   ```
3. Another way to represent primary key; and to represent compound primary keys
   ``` 
   def id = column[Long]("id", O.AutoInc)
   def pk = primaryKey("pk_id", id)
   ```
   
   ``` 
   //compound primary keys
   class OccupantTable(tag: Tag) extends Table[Occupant](tag, "occupant") {
        def roomId = column[Long]("room")
        def userId = column[Long]("user")
        
        def pk = primaryKey("room_user_pk", (roomId, userId))
        
        def * = (roomId, userId).mapTo[Occupant]
   ```
4. Indices
   ``` 
   class IndexExample(tag: Tag) extends Table[(String,Int)](tag, "people") {
        def name = column[String]("name")
        def age = column[Int]("age")
        def * = (name, age)
        
        def nameIndex = index("name_idx", name, unique = true)
        def compoundIndex = index("c_idx", (name, age), unique = true) 
   }
   ```
5. Foreign Keys; the method of ```foreignKey``` takes four required parameters
   + a name
   + the column, or columns, that make up the foreign key
   + the ```TableQuery``` that the foreign key belongs to
   + a function on the supplied TableQuery[T] taking the supplied column(s) as parameters and returning an instance of T
   
   ``` 
   case class Message(senderId : Long, content : String, id : Long = 0L)
   
   class MessageTable(tag: Tag) extends Table[Message](tag, "message") {
        def id = column[Long]("id", O.PrimaryKey, O.AutoInc)
        def senderId = column[Long]("sender")
        def content = column[String]("content")
        
        def * = (senderId, content, id).mapTo[Message]
        
        def sender = foreignKey("sender_fk", senderId, users)(_.id)
        }
   ```
   + On Update and On Delete: there are a number of referential actions that could be triggered. The default is for nothing to happen, but you can change that:
   ```
   NoAction
   Cascade
   Restrict
   SetNull
   SetDefault
   ```
   ```
   def sender = foreignKey("sender_fk", senderId, users)(_.id, onDelete = ForeignKeyAction.Cascade)
   ```
6. Column Options are defined in ```ColumnOption```, they are accessed via ```O```:
   ```
   O.PrimaryKey
   O.SqlType
   O.Unique
   O.Default
   ```

#### Custom Column Mappings
1. Value classes: a good approach to model primary keys is using value classes.
   ``` 
   case class MessagePK(value: Long) extends AnyVal
   
   implicit val messagePKColumnType =
        MappedColumnType.base[MessagePK, Long](_.value, MessagePK(_))
   ```
   
   ``` 
   //Slick provides a short-hand called MappedTo
   case class MessagePK(value: Long) extends AnyVal with MappedTo[Long]
   ```
   + When we use ```MappedTo``` we don't need to define a separate ```ColumnType```. ```MappedTo``` works with any class that: has a method called ```value``` that returns the underlying database value; and has a single-parameter constructor to create the Scala value from the database value.
2. Modelling sum types
   ```
   sealed trait Flag
   
   case object Important extends Flag
   case object Offensive extends Flag
   case object Spam extends Flag
   
   case class Message(
        senderId: UserPK,
        content: String,
        flag: Option[Flag] = None,
        id : MessagePK = MessagePK(0L))
        
   implicit val flagType =
        MappedColumnType.base[Flag, Char](
            flag => flag match {
                case Important => '!'
                case Offensive => 'X'
                case Spam      => '$'
            },
            code => code match {
                case '!' => Important
                case 'X' => Offensive
                case '$' => Spam
            })
   ```
   ``` 
   class MessageTable(tag: Tag) extends Table[Message](tag, "flagmessage") {
    def id = column[MessagePK]("id", O.PrimaryKey, O.AutoInc)
    def senderId = column[UserPK]("sender")
    def content = column[String]("content")
    def flag = column[Option[Flag]]("flag")
    
    def * = (senderId, content, flag, id).mapTo[Message]
    
    def sender = foreignKey("sender_fk", senderId, users)(_.id, onDelete= ForeignKeyAction.Cascade)
   }
   ```
   + when querying messages with a particular flag, we need to give the compiler a little help with the types:
   ``` 
   exec(messages.filter(_.flag === (Important: Flag)).result
   ```
   + the type annotation here is annoying, we can work around it in two ways.
   ``` 
   // First, we can define a “smart constructor” method for each flag that returns it pre-cast as a Flag:
   object Flags {
        val important : Flag = Important 
        val offensive : Flag = Offensive 
        val spam : Flag = Spam
        val action = messages.filter(_.flag === Flags.important).result 
   }
   
   // Second, we can define some custom syntax to build our filter expressions:
   implicit class MessageQueryOps(message: MessageTable) { 
        def isImportant = message.flag === (Important : Flag) 
        def isOffensive = message.flag === (Offensive : Flag) 
        def isSpam = message.flag === (Spam : Flag)
   }
   
   messages.filter(_.isImportant).result
   ```
### Chapter 6 Joins and Aggregates
#### Monadic Joins
   ``` 
   val q = for {
        msg <- messages
        //foreign key sender
        usr <- msg.sender
        } yield (usr.name, msg.content)
   // OR
   val q = messages flatMap { msg =>
                msg.sender.map { usr =>
                    (usr.name, msg.content)
                    }
                }
   ```
#### Applicative Joins
1. An applicative join is where we explicitly write the join in code. In SQL this is via the JOIN and ON keywords, which are mirrored in Slick with the following methods:
   + ```join``` -- an inner join
   + ```joinLeft``` -- a left outer join
   + ```joinRight``` -- a right outer join
   + ```joinFull``` -- a full outer join
#### Inner Join
1. An inner join selects data from mul􏰀ple tables, where the rows in each table match up in some way. Typically, the matching up is done by comparing primary keys. If there are rows that don’t match up, they won’t appear in the join results.
   ```
   val usersAndRooms =
        messages.
        join(users).on(_.senderId === id).
        join(rooms).on{ case ((msg, user), room) => msg.roomId === room.id }
        
   val usersAndRooms =
        messages.
        join(users).on(_.senderId === _.id). 
        join(rooms).on(_._1.roomId === _.id)
   ```
   + when turned into an action the action will contain nested tuples; ```map``` to get what you want; ```filter``` to select specific records.
   ``` 
   val action: DBIO[Seq[(Message, User), Room] = useraAndRooms.result
   
   //map like this
   val usersAndRooms =
        messages.
        join(users).on(_.senderId === _.id).
        join(rooms).on { case ((msg,user), room) => msg.roomId === room.id }. 
        map { case ((msg, user), room) => (msg.content, user.name, room.title) }
   ```
   ``` 
   val usersAndRooms =
        messages.
        join(users).on(_.senderId === _.id).
        join(rooms).on { case ((msg,user), room) => msg.roomId === room.id }
   
   val airLockMsgs =
        usersAndRooms.
        filter { case (_, room) => room.title === "Air Lock" }   
   ```
#### Left Join
1. Now we are selecting all the records from a table, and matching records from another table if they exist. If we find no matching record on the left, we will end up with NULL values in our results.
   ``` 
   val left = messages.joinLeft(rooms).on(_.roomId === _.id)
   // Not all messages are in a room, so in that case the roomId column will be NULL. Slick will lift those possibly null values into something more comfortable: Option.
   ```
   
   ``` 
   val left = messages.joinLeft(rooms).on(_.roomId === _.id).
   map { case (msg, room) => (msg.content, room.map(_.title)) }
   // because the room element is optional, we naturally extract the title element using Option.map: room.map(_.title)
   ```   
#### Right Join
1. a left join selects all the records from the left hand side of the join, with possibly NULL values from the right. Right joins reverse the situation, selecting all records from the right side of the join, with possibly NULL values from the left.
   ``` 
   val right = for {
        (msg, room) <- messages joinRight (rooms) on (_.roomId === _.id)
        } yield (room.title, msg.map(_.content))
   ```
#### Full Outer Join
1. Full outer joins mean either side can be NULL.
   ``` 
   val outer = for {
    (room, msg) <- rooms joinFull messages on (_.id === _.roomId)
    } yield (room.map(_.title), msg.map(_.content))
    
    // Query[
    //   (Rep[Option[String]], Rep[Option[String]]), (Option[String], Option[String]),
    //   Seq]
   ```
#### Cross Joins
1. In the examples above, whenever we've used ```join``` we've also used an ```on``` to constrain the join. If we omit the ```on``` condition for any join, joinLeft, or joinRight, we end up with a cross join. Cross joins include every row from the le􏰂 table with every row from the right table. If we have 10 rows in the first table and 5 in the second, the cross join produces 50 rows.
   ``` 
   val cross = messages joinLeft users
   ```
#### Zip Joins
1. normal ```zip```
   ``` 
   val msgs = messages.sortBy(_.id.asc).map(_.content)
   
   val conversations = msgs zip msgs.drop(1) 
   ```
2. ```zipWith```
   ``` 
   // zipWith, lets us provide a mapping function along with the join.
   def combiner(c1: Rep[String], c2: Rep[String]) = (c1.toUpperCase, c2.toLowerCase)
   
   val query = msgs.zipWith(msgs.drop(1), combiner)
   ```
3. ```zipWithIndex```
   ``` 
   val query = messages.map(_.content).zipWithIndex
   
   val action: DBIO[Seq[(String, Long)]] = query.result
   ```

#### Aggregation
1. Aggregate functions are all about computing a single value from some set of rows.
#### Functions
   |            Method            |        SQL                             |
   |------------------------------|----------------------------------------|
   |       length                 |      COUNT(1)                          |
   |min                           |     MIN(column)                        |
   |max                           |     MAX(column)                        |
   |sum                           |SUM(column)                             |
   |avg                           |AVG(column) - mean of the column values |
#### Grouping
1. Aggregate functions are often used with column grouping. For exmaple, how many messages has each user sent? That's a grouping(by user) of an aggregate(count).
2. ```groupBy``` to group rows by some expression. A ```groupBy``` must be followed by a ```map```
   ``` 
   val msgPerUser: DBIO[Seq[(Long, Int)]] = 
        messages.groupBy(_.senderId).
        map { case (senderId, msgs) => senderId -> msgs.length }.result
   ```
3. Groups and Joins
   ``` 
   val msgsPerUser =
    messages.join(users).on(_.senderId === _.id).
    groupBy { case (msg, user) => user.name }.
    map     { case (name, group) => name -> group.length }.result
    
    exec(msgsPerUser).foreach(println)
    // (HAL, 2)
    // (Dave, 2)
   ```
4. More Complicated Grouping
   ``` 
   val stats =
        messages.join(users).on(_.senderId === _.id).
        groupBy { case (msg, user) => user.name }.
        map     {
           case (name, group) =>
                (name, group.length, group.map{ case (msg, user) => msg.id}.min)
           }
   ```
5. Grouping by Multiple columns
   ``` 
   val msgsPerRoomPerUser = 
        rooms.
        join(messages).on(_.id === _.roomId).
        join(users).on { case ((room, msg), user) => user.id === msg.senderId }.
        groupBy { case ((room, msg), user) => (room.title, user.name) }.
        map { case((room,user), group) => (room, user, group.length) }.
        sortBy { case (room, user, group) => room }
   ```