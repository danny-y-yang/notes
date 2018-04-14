# notes
Notes for a lifetime of engineering

# Introduction
Learning a programming language is analogous to learning a real language: you must understand the:
1. Grammar (how the language is structured)
  - Is the programming language algorithmic, functional, or object oriented? 
2. Vocabulary (how to name things you want to talk about):
  - What data structures, operations, and facilities are provided by standard libraries? 
3. Usage (how to communicate common ideas naturally and effectively):
  - Customary and effective ways to structure your code

## Vocabulary
#### Imperative
- Uses a sequence of statements to determine how to reach a certain goal. Each statement changes the state of the program as they are executed in turn.

```
int total = 0;
int n1 = 5;
int n2 = 10;
int n3 = 15;
total = n1 + n2 + n3;
```
#### Pure functions (Declarative);
- self-contained, and stateless. The return value is only determined by its input values, without any observable side effects, such as write to stdout, ask for user input, do I/O. e.g Math.cos(x) will always return the same result. Idempotent. It’s a coffee grinder: beans go in, powder comes out, end of story.

- Benefits: easy readability & maintainability. No reliance on external state. Easy to test since each pure function is an isolated piece of logic.

```
// Impure functions
func f(): 
  int a;
  // do something to a
  // do something again to a
  // invoke network I/O and apply to a
  return a;
```

```
// Using pure functions, each function has a specific use case, and can handle any input.
func f1(int x) {}
func f2(int x) {}
func f3(int x) {}

func f():
  int a;
  f1(a);
  f2(a);
  f3(a);
  return a;
```

# Chapter I: Creation and Destruction of Objects
## Item I: Static Factory Methods
- Another way for a class to provide a client an instance of itself, the other being a public constructor
- This is simply a static method that returns an instance of the class
- This is *not* the same as the Factory Method design pattern

### Advantages
1. They have names, unlike constructors. 
```
BigInteger(a, b, c) --> ?? Could be a prime integer, or it might not. Needs more description! 

BigInteger.probablePrime(a, b, c) --> returns a prime integer most likely. Descriptive!
BigInteger.defaultNumber(a, b, c)
BigInteger.evenNumber()
..
```

2. They are not required to create a new object each time they are invoked. 

```
public class DonutShopCreator {

  Map<String, DonutShop> shops = new Hashmap<>();
  
  public static DonutShop newDonutShop(String typeOfDonut) {
    // if typeOfDonut is a key in shops, return that shop
    // else make a new one
  }
}
```
Classes that use static factory methods are said to be **instance-controlled**. This allows a class to be a guaranteed **singleton** or **noninstantiable**. One can also force a class to have no two objects that are equal to each other.
```
a.equals(b) IFF a == b
```

3. They can return an object of any subtype of their return type. 
4. They can reduce verbosity. 

```
// Bad
Map<ExtremelyLongString, AnotherExtremelyLongString> = new HashMap<ExtremelyLongString, AnotherExtremelyLongString>();

// Good
Map<ExtremelyLongString, AnotherExtremelyLongString> = HashMap.newInstance();
```

### Disadvantages
1. Classes with public static factory methods, and __private constructors__ cannot be subclassed. 
```
class Person {
  public static Person createPersonWithName(String name) {
    return new Person(...);
  }
  // other factory methods
   
  // private constructor
  private Person(...);
}
```
Now we can't do:
```
class Programmer extends Person {}... 
```
since Person does not have public constructors.

2. They are not readily distingusable from other static methods. So try to follow common coding conventions.
```
DonutShop.valueOf             // returns new instance that has the value of its parameters
DonutShop.of                  // returns new instance that has the value of its parameters (more concise)
DonutShop.getInstance         // returns an existing instance. Singleton instances would have no parameters
DonutShop.newInstance         // returns a new instance using parameters
Mall.getType(DonutShop)       // returns an existing instance, but usually this is a method in a different class
DonutShop.newType(DonutShop)  // returns a new instance, but usually this is a method in a different class
```
