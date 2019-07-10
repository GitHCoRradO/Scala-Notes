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
2. ```git config --global email```
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