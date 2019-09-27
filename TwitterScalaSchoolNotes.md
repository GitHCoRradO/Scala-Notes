## Twitter Scala School Notes

### Basics

#### Partial application
1. You can partially apply a function with an underscore, which gives you another function. 
   ``` 
   def expo(m: Int, n: Int) = scala.math.pow(m, n)
   val expo2 = expo(2, _: Int)
   
   val a = expo2(3)     // a = 8.0
   ```
#### Curried functions
1. Sometimes it makes sense to let people apply some arguments to your function now and others later.
   ``` 
   def expo(m: Int)(n: Int) = scala.math.pow(m, n)
   //direct use
   val a = expo(2)(3)   //a = 8.0
   //You can fill in the first parameter and partially apply the second.
   val expo2 = expo(2) _            //expo2: Int => Double = <function>
   
   val b = expo2(4)         //b = 16.0
   ```
2. You can take any function of multiple arguments and curry it.
   ``` 
   //expo takes 2 arguments
   def expo(m: Int, n: Int) = scala.math.pow(m, n)
   val curriedExpo = (expo _).curried   //curriedExpo: Int => (Int => Double) = scala.Function2<function>
   
   val expo2 = curriedExpo(2)
   val c = expo2(5)     //c = 32.0 
   ```
### Collections

#### ```flatten``` collapses one level of nested structure
   ``` 
   List(List(1, 2), List(3, 4)).flatten
   // List[Int] = List(1, 2, 3, 4)
   ```
#### ```flatMap``` is a frequently used combinator that combines mapping and flattening. flatMap takes a function that works on the nested lists and then concatenates the results back together.
   ``` 
   val nestedNumbers = List(List(1, 2), List(3, 4))
   nestedNumbers.flatMap(x => x.map(_ * 2))  // List[Int] = List(2, 4, 6, 8)

   // Think of it as short-hand for mapping and then flattening
   nestedNumbers.map((x: List[Int]) => x.map(_ * 2)).flatten
   ```

### Pattern matching & functional composition

#### Function Composition
1. First make two aptly-named functions
   ``` 
   def f(s: String) = "f(" + s + ")"
   def g(s: String) = "g(" + s + ")"
   ```
2. ```compose``` makes a new function that composes other functions ```f(g(x))```
   ``` 
   val fComposeG = f _ compose g _
   fComposeG("Gay")     //f(g(Gay))
   ```
3. ```andThen``` is like ```compose```, but calls the first function and then the second, ```g(f(x))```
   ``` 
   val fAndThenG = f _ andThen g _
   fAndThenG("Gay")     //g(f(Gay))
   ```
#### Understanding PartialFunction
1. A Partial Function is only defined for certain values of the defined type. A Partial Function (Int) => String might not accept every Int. ```isDefinedAt``` is a method on PartialFunction that can be used to determine if the PartialFunction will accept a given argument. Note ```PartialFunction``` is unrelated to a partially applied function that we talked about earlier.
   ``` 
   val one: PartialFunction[Int, String] = { case 1 => "one" }
   one.isDefinedAt(1)   //true
   one.isDefinedAt(2)   //false
   
   one(1)               //one(1):String = "one"
   ```
2. PartialFunctions can be composed with something new, called ```orElse```, that reflects whether the PartialFunction is defined over the supplied argument.
   ``` 
   val two: PartialFunction[Int, String] = { case 2 => "two" }
   val three: PartialFunction[Int, String] = { case 3 => "three" }
   val wildcard: PartialFunction[Int, String] = { case _ => "something else" }
   val partial = one orElse two orElse three orElse wildcard
   
   partial(5)   //"something else"
   partial(3)   //"three"
   partial(0)   //"something else"
   ```
3. The mystery of case; what is a case statement?
   ``` 
   /*
   Why does this work?
   
   filter takes a function. In this case a predicate function of (PhoneExt) => Boolean.
   
   A PartialFunction is a subtype of Function so filter can also take a PartialFunction!
   */
   case class PhoneExt(name: String, ext: Int)
   val extensions = List(PhoneExt("steve", 100), PhoneExt("robey", 200))
   
   extensions.filter({ case PhoneExt(name, extension) => extension < 200 })
   ```
### Type & polymorphism basics
