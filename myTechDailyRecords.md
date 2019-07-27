## Miscellaneous Daily Records about tech, work, etc.

### 2019-7-2

#### tail
1. Print  the last 10 lines of each FILE to standard output.  With more than one FILE, precede each with a header giving the file name.
2. tail -f filename
   + output appended data as the file grows
3. -n, --lines=K
   + output the last K lines, instead of the last 10; or use -n +K to output lines starting with the Kth
4. -v, --verbose
   + always output headers giving file names

#### [revisit higher-order functions](https://danielwestheide.com/blog/2013/01/23/the-neophytes-guide-to-scala-part-10-staying-dry-with-higher-order-functions.html)
1. [what is the deference between method and function in scala](https://stackoverflow.com/questions/2529184/difference-between-method-and-function-in-scala)
2. [Scala functions vs methods](http://jim-mcbeath.blogspot.com/2009/05/scala-functions-vs-methods.html)
3. my brief understanding
   + a scala method is a part of a class; it has a name, a signature, optionally some annotations, and some bytecode.
   + a function in scala is a complete object. There are a series of traits in Scala to represent functions with various numbers of arguments: Function0, Function1, Function2, etc. As an instance of a class that implements one of these traits, a function object has methods. One of these methods is the apply method, which contains the code that implements the body of the function. Scala has special "apply" syntax: if you write a symbol name followed by an argument list in parentheses (or just a pair of parentheses for an empty argument list), Scala converts that into a call to the apply method for the named object. When we create a variable whose value is a function object and we then reference that variable followed by parentheses, that gets converted into a call to the apply method of the function object. 
### 2019-7-4
#### the difference between tinyint(1) and tinyint(2) in MySQL
1. table

   |type    | Storage(Bytes)|min value signed|min value unsigned|max value signed|max value unsigned|
   |--------|---------------|----------------|------------------|----------------|------------------|
   |tinyint | 1             |    -128        |     0            |   127          |        255       |
2. when used together with zerofill:
   ``` 
   CREATE TABLE `test` (                                  
            `id` int(11) NOT NULL AUTO_INCREMENT,                
            `state` tinyint(1) unsigned zerofill DEFAULT NULL,   
            `state2` tinyint(2) unsigned zerofill DEFAULT NULL,  
            `state3` tinyint(3) unsigned zerofill DEFAULT NULL,  
            `state4` tinyint(4) unsigned zerofill DEFAULT NULL,  
            PRIMARY KEY (`id`)                                   
          ) ENGINE=InnoDB AUTO_INCREMENT=6 DEFAULT CHARSET=utf8  
   
   insert into test (state,state2,state3,state4) values(4,4,4,4);
   select * from test; 
   
   //output
   state state2 state3 state4
   4       04    004   0004
   
   ```
### 2019-7-5
#### var initialization
   ``` 
   var s: Whatever = _
   ```
1. This will initialize ```s``` to the default value for ```Whatever``` (null for reference types, 0 for numbers, false for bools etc.)
#### [What is a DSL and where should I use it?](https://stackoverflow.com/questions/41724/what-is-a-dsl-and-where-should-i-use-it)

### 2019-7-8
#### configure a remote for a fork
1. List the current configured remote repository for your fork.
   + ```$ git remote -v```
2. Specify a new remote upstream repository that will be synced with the fork.
   + ```$ git remote add upstream https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY.git```
3. Verify the new upstream repository you've specified for your fork.
   + ```$ git remote -v```
#### set git global user.name and user.email
1. ```git config --global user.name```
2. ```git config --global user.email```
3. ```git config --global user.name "username"```
4. ```git config --global user.email "useremail"```

#### MySQL update a bunch of rows which meet a certain request
   + ```update `tablename` set `columnname` ='newvalue' where `anothercolumnname` ='someRequest'; ```
### 2019-7-9
#### rename a directory
   + ```$ mv thisDirectory/ Directorywithnewname/```
### 2019-7-10
#### ```$ jps |grep -v Jps``` command
   + ```jps``` is a terminal command; it is the Java Virtual Machine Process Status Tool
   + The jps tool lists the instrumented HotSpot Java Virtual Machines (JVMs) on the target system. 
   + SYNOPSIS: ```$ jps [options] [hostid]```
   + if hostid is omitted, jps lists localhost JVM processes status
   +```$ jps |grep -v Jps``` omits the Jps process itself which is also running on JVM.
#### ```sftp``` command
   + ```sftp username@remotehost``` default port 22
   + ```sftp -oPort=22 username@remotehost``` -oPort option for specifying a port
   + ```> get pathtoRemoteFile pathtoLocalDirectory``` for downloading a remote file
   + ```> get -r pathtoRemoteDirectory pathtoLocalDirectory``` for downloading a remote Directory
   + ```> put pathtoLocalFileOrDirectory pathtoRemoteDirectory``` for uploading a local file or directory
   + in a sftp session, use ```!command``` to run commands on local machine
   + ```> exit``` or ```> quit``` for exiting sftp program.
### 2019-7-16
#### To get a value out of a scala ```Option```: 
1. Directly extracting the value out
   + match expression
   ``` 
   val res = option match {
       case Some(i) => i
       case None => default
   }
   ```
   + ```getOrElse```
   ``` 
   val res = option.getOrElse(default)
   ```
2. Applying a function while extracting the value from the Option
   + match expression
   ``` 
   def f(...) = { ... }
   
   val res = option match {
       case Some(i) => f(i)
       case None => default
   }
   ```
   + use ```map``` followed by ```getOrElse```
   ``` 
   val res = option.map(f).getOrElse(default)
   ```
   + call ```fold``` on the ```Option```
   ```
   //applies the function f to the value in a Some, or returns the default value if option is really a None.
   val res = option.fold(default)(f)
   ```
#### [What is the relation between Iterable and Iterator?](https://stackoverflow.com/questions/11302270/what-is-the-relation-between-iterable-and-iterator)
### 2019-7-17
#### [Check what port MySQL is running on](https://serverfault.com/questions/116100/how-to-check-what-port-mysql-is-running-on)
1. In MySQL session:

   ``` mysql> SHOW GLOBAL VARIABLES LIKE 'PORT';```
2. The best way to actually know what application is listening to which interface and on what port is to use ```netstat```

   ``` 
   //you can do this as root, option -p does not work on Mac OS X
   $ netstat -tlnp
   //sample output
   Active Internet connections (only servers)
   Proto Recv-Q Send-Q Local Address               Foreign Address             State       PID/Program name
   tcp        0      0 127.0.0.1:9003              0.0.0.0:*                   LISTEN      16992/java
   tcp        0      0 127.0.0.1:9999              0.0.0.0:*                   LISTEN      17233/php-fpm
   tcp        0      0 0.0.0.0:80                  0.0.0.0:*                   LISTEN      5136/nginx
   tcp        0      0 0.0.0.0:22                  0.0.0.0:*                   LISTEN      -
   tcp        0      0 0.0.0.0:12345               0.0.0.0:*                   LISTEN      -
   tcp        0      0 127.0.0.1:15770             0.0.0.0:*                   LISTEN      -
   tcp        0      0 0.0.0.0:443                 0.0.0.0:*                   LISTEN      5136/nginx
   tcp        0      0 127.0.0.1:9000              0.0.0.0:*                   LISTEN      4184/java
   tcp        0      0 :::22                       :::*                        LISTEN      -
   tcp        0      0 :::12345                    :::*                        LISTEN      -
   ```
####[What is the utf8mb4_0900_ai_ci Collation?](https://www.monolune.com/what-is-the-utf8mb4_0900_ai_ci-collation/)
1. ```utf8mb4``` has become the default character set, with ```utf8mb4_0900_ai_ci``` as the default collation in MySQL 8.0.1 and later.
   + ```uft8mb4``` means that each character is stored as a maximum of 4 bytes in the UTF-8 encoding scheme.
   + ```0900``` refers to the Unicode Collation Algorithm version. (The Unicode Collation Algorithm is the method used to compare two Unicode strings that conforms to the requirements of the Unicode Standard).
   + ```ai``` refers accent insensitivity. That is, there is no difference between e, è, é, ê and ë when sorting.
   + ```ci``` refers to case insensitivity. This is, there is no difference between p and P when sorting.
2. If accent sensitivity and case sensitivity are required, you may use ```utf8mb4_0900_as_cs``` instead.
### 2019-7-18
#### sbt session, type in ```console``` command to enter console mode, then type in ```:paste``` to enter paste mode.

###2019-7-19
#### [How to zip or unzip files and folders using command line on Ubuntu server](https://www.wpoven.com/tutorial/how-to-zip-or-unzip-files-and-folders-using-command-line-on-ubuntu-server/)
1. First get ```zip``` and ```unzip``` installed
   + ```sudo apt-get install zip```
   + ```sudo apt-get install unzip```
2. ```-r``` option for zipping folder having more than one file or folder and do not use ```-r``` for single file
   + ```zip -r pathtoDirectory/zippedname.zip orignial_folder```
   + ```zip singlefile.zip original_file```

#### To check out the amount of free or used disk space in Ubuntu
1. Command line interface
   + ```$ df -h```
2. [refer to this ask ubuntu question](https://askubuntu.com/questions/73160/how-do-i-find-the-amount-of-free-space-on-my-hard-drive)
#### Check out system specifications in Ubuntu
1. ```$ lshw | less``` which means ls(list)hw(hardware)
2. [refer to this ask ubuntu question](https://askubuntu.com/questions/55609/how-do-i-check-system-specifications)
### 2019-7-22
#### [Execute Command Line Java program in background](https://stackoverflow.com/questions/3030605/execute-command-line-java-program-in-background)
1. You can add ```&``` at the end of the command line to run it in the background.
#### To check out CPU MEM usage in Ubuntu
1. ```$ sudo apt-get install htop```
2. run ```$ htop```
### 2019-7-23
#### [SQL LIMIT](http://www.sqltutorial.org/sql-limit/)
1. To retrieve a portion of rows returned by a query, use the ```LIMIT``` and ```OFFSET``` clauses.
   ``` 
   SELECT 
       column_list
   FROM
       table1
   ORDER BY column_list
   LIMIT row_count OFFSET offset;
   ```
   + The ```row_count``` determines the number of rows that will be returned.
   + The ```OFFSET```  clause skips the ```offset``` rows before beginning to return the rows. The ```OFFSET``` clause is optional so you can skip it. If you use both ```LIMIT``` and ```OFFSET``` clauses the ```OFFSET```  skips offset rows first before the ```LIMIT```  constrains the number of rows.
2. When you use the ```LIMIT``` clause, it is important to use an ```ORDER BY``` clause to make sure that the rows in the returned are in a specified order.

### 2019-7-25
#### Representing 1-element ```Seq``` and 1-element ```List```
   ``` 
   //1-element Seq
   element +: Nil
   //2-element Seq
   e1 +: e2 +: Nil
   //1-element List
   element :: Nil
   ```

