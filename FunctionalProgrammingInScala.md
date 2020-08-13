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
### Ex.2.5
   ``` 
   def compose[A, B, C](f: B => C, g: A => B): A => C = {
     a : A => f(g(a))
   }
   ```
### Notes for Ch3.1
1. Adding ```sealed``` in front means that all implementations of the ```trait``` must be declared in this file.

### Ex3.2 through Ex3.6
   ``` 
   sealed trait List[+A]
   
   case object Nil extends List[Nothing]
   
   case class Cons[+A](head: A, tail: List[A]) extends List[A]
   
   object List {
   
     def length[A](l: List[A]): Int = l match {
       case Nil => 0
       case Cons(_, xs) => 1 + length(xs)
     }
   
     def sum(ints: List[Int]): Int = ints match {
       case Nil => 0
       case Cons(x, xs) => x + sum(xs)
     }
   
     def product(ds: List[Double]): Double = ds match {
       case Nil => 1.0
       case Cons(0.0, _) => 0.0
       case Cons(x, xs)  => x * product(xs)
     }
   
     // what are different choices you could make in your implementation if the List is Nil?
     def tail[A](ls: List[A]): List[A] = ls match {
       case Nil => Nil             // or  throw List is Nil runtime Error?
       case Cons(_, xs) => xs
     }
   
     def setHead[A](ls: List[A], value: A): List[A] = ls match {
       case Nil => throw new RuntimeException("Setting head of Nil List")
       case Cons(_, xs) => Cons(value, xs)
     }
   
     def drop[A](l: List[A], n: Int): List[A] =
         if (n <= 0) l
         else l match {
           case Nil => Nil
           case Cons(_,t) => drop(t, n-1)
         }
   
     def dropWhile[A](l: List[A], f: A => Boolean): List[A] = l match {
       case Nil => Nil
       case Cons(x, xs) if f(x) => dropWhile(xs, f)
       case _ => l
     }
   
     def init[A](l: List[A]): List[A] = l match {
       case Nil => Nil
       case Cons(x, xs) =>
         if (length(xs) > 1) Cons(x, init(xs)) else Cons(x, Nil)
     }
   
     def apply[A](as: A*): List[A] =
       if (as.isEmpty) Nil else Cons(as.head, apply(as.tail: _*))
   }

   ``` 