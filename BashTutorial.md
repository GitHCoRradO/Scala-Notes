## Bash Tutorial Notes ([reference](https://wangdoc.com/bash/intro.html))

#### 一般shell默认是bash，如果不是可以通过键入```bash```以启动Bash

#### ```echo```
1. ```echo``` 输出多行文本时，把多行文本放在引号内部
2. 默认情况下```echo```输出的文本末尾带换行符，```-n```参数 可以取消输出换行符, 使得下一个命令提示符紧跟文本末尾(zsh中好像此参数会在文本末尾输出%然后输出换行符？)
   ```
   $echo -n hello world
   helloworld$echo a;echo b
   a
   b
   $echo -n a;echo b
   ab
   $
   ```
3. 默认情况下```echo```会原样输出引号内的特殊字符；参数```-e```会解释引号（双引号和单引号）里面的特殊字符（比如换行符\n）
   ``` 
   $ echo "hello\nworld"
   hello\nworld
   $ echo -e "hello\nworld"
   hello
   world
   ```
#### Bash命令的格式
1. Bash 单个命令一般都是一行，用户按下回车键，就开始执行。有些命令比较长，写成多行会有利于阅读和编辑，这时可以在每一行的结尾加上反斜杠 ```\ ```，Bash 就会将下一行跟当前行放在一起解释。
   ``` 
   $ echo hello world, Bash
   hello world, Bash
   $ echo hello world,\
    Bash
   hello world, Bash
   ```
2. Bash 用空格或tab来区分不同的参数，如果不同的参数之间有多个空格，Bash会忽略多余的空格
   ``` 
   $ echo this is a         test
   this is a test
   ```
3. ```;```是命令结束符，使得一行可以放置多个命令，上一个命令执行结束后，再执行第二个命令。使用分号时，第二个命令总是接着第一个命令执行，不管第一个命令执行成功或失败。
   ``` 
   $ cear
   bash: cear: command not found
   $ cear; echo ls
   ls
   ```
4. 命令组合符 ```&&```，只有前面的命令成功才会执行后面的命令
   ``` 
   $ cear && echo ls
   bash: cear: command not found
   $ echo ls && echo clear
   ls
   clear
   ```
5. 命令组合符 ```||```，只有前面的命令失败才会执行后面的命令
   ``` 
   $ cear || echo ls
   ls
   $ echo lss|| echo clear
   lss
   ```
   