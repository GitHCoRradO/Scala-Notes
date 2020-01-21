### Ex2.1
   ```
   def fib(n: Int): Int = {
     @annotation.tailrec
     def go(cnt: Int, prev: Int, curr: Int): Int = cnt match {
       case i if i < 0 => System.err(s"negative")
       case 0 => prev
       case i => go(i - 1, curr, prev + curr)
     }
     go(n, 0, 1)
   }
   ```

### Ex2.2
   ``` 
   def isSorted[A](as: Array[A], ordered: (A,A) => Boolean): Boolean = {
     @annotation.tailrec
     def loop(index: Int): Boolean = index match {
       case i if i >= as.length - 1 => true
       case i if (!ordered(as(i), as(i + 1))) => false
       case i if (ordered(as(i), as(i + 1))) => loop(i + 1)
     }
     loop(0)
   }

   ```

### Ex2.3
   ``` 
   def curry[A,B,C](f: (A, B) => C): A => (B => C) = {
     a: A => b: B => f(a, b)
   }
   
   def curry[A, B, C, D, E](f: (A, B, C, D) => E): A => (B => (C => (D => E))) = {
     a => b => c => d => f(a, b, c, d)
   }
   ```
    
### Ex2.4
   ``` 
   def uncurry[A, B, C](f: A => B => C): (A, B) => C = {
     (a: A, b: B) => f(a)(b)
   }
   
   def uncurry[A, B, C, D, E](f: A => B => C => D => E): (A, B, C, D) => E = {
     (a, b, c, d) => f(a)(b)(c)(d)
   }
   ```