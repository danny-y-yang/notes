# notes
Notes for a lifetime of engineering

[I'm an inline-style link](https://github.com/danny-y-yang/notes/edit/master/README.md#<design patterns>)


# Introduction
Learning a programming language is analogous to learning a real language: you must understand the:
1. Grammar (how the language is structured)
  - Is the programming language algorithmic, functional, or object oriented? 
2. Vocabulary (how to name things you want to talk about):
  - What data structures, operations, and facilities are provided by standard libraries? 
3. Usage (how to communicate common ideas naturally and effectively):
  - Customary and effective ways to structure your code
  
## Vocabulary
### Imperative
- Uses a sequence of statements to determine how to reach a certain goal. Each statement changes the state of the program as they are executed in turn.

```
int total = 0;
int n1 = 5;
int n2 = 10;
int n3 = 15;
total = n1 + n2 + n3;
```
### Pure functions (Declarative);
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

### Creational Patterns
- Abstract the instantiation process

# Effective Java Practices
## Chapter I: Creation and Destruction of Objects
### Item I: Static Factory Methods
- Another way for a class to provide a client an instance of itself, the other being a public constructor
- This is simply a static method that returns an instance of the class
- This is *not* the same as the Factory Method design pattern

#### Advantages
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

#### Disadvantages
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

# Design Patterns
## Abstract Factory
### Overview
Provide an interface for creating families of related or dependent objects without specifying their concrete classes.

```
private abstract class ProductFactory() {
  public ProductA createProductA();
  public ProductB createProductB();
}

public class ChineseProductFactory extends ProductFactory() {...}
public class AmericanProductFactory extends ProductFactory() {...}

private abstract class ProductA {...}
private abstract class ProductB {...}

public class ChineseProductA extends ProductA {...}
public class ChineseProductB extends ProductB {...}
public class AmericanProductA extends ProductA {...}
public class AmericanProductB extends ProductB {...}
```

From the perspective of the client: 
```
ProductFactory factory = new ChineseProductFactory();

// Uses interface declared by abstract factory and product
ProductA productA = factory.createProductA();            
productA.ship();
productA.handle();
factory.close();
```

![alt text](https://github.com/danny-y-yang/notes/blob/master/abstract_factory_design.png "Abstract Factory Design")

- __AbstractFactory__ (ProductFactory): declares interface for operations that create abstract product objects
- __ConcreteFactory__ (ChineseProductFactory): implements operations to create concrete product objects
- __AbstractProduct__ (ProductA): declares interface for a type of product object
- __ConcreteProduct__ (ChineseProductA): implements AbstractProduct interface, defines the product being created by corresponding concrete factory
- __Client__: uses only interfaces declared by AbstractFactory and AbstractProduct

### Advantages/Disadvantages
This allows the client to only use interfaces declared by abstract classes, which means the underlying concrete classes are decoupled from the actualy client code, allowing more flexibility and robustness. Since the concrete factory is only instantiated once in the code, changing the factory type will not affect the behavior of the rest of the code since the client uses methods provided by an interface.

This also enforces __consistency__. By instantiating only ONE type of factory at a time, all the remaining behavior will correspond to that family of items (only Chinese methods of handling products). 

__Problem__: If a new product (ProductC) is to be created by the AbstractFactory, then we must add functionality to the abstract class/interface, and ultimately add this feature to all of its subclasses. 

### Techniques for Implementation
#### Singleton Factories
Only one instance of a ConcreteFactory is needed per product family (Only ONE Chinese factory and ONE american factory is needed)

#### Creating Products
If there are multiple product families, then a new concrete factory must be created for each family, even if the product families differ only slightly. To solve this, create __prototypes__. The concrete factory is initialized with a prototype of each product in the family, and is cloned and changed if a new product needs to be created. (ChineseFactory now produces Taiwan products, so use a Chinese product prototype to create this instead of creating a whole new TaiwanFactory).
```
// Bad
class TaiwanProductFactory extends ProductFactory {
  ... literally the same as Chinese except for a few features
}

// Good
class ChineseProductFactory {
  ProductA prototypeA = new ChineseProductA();
  
  ProductA CreateTaiwanProductA () {
    return prototypeA.setLabel("MadeInTaiwan");
  }
}
```

#### Defining Extensible Factories
If a productC is to be created by the factory, then we must implement this operation in the AbstractFactory as well as the subclasses implementing it. One solution would be to create a parameterized Make function, such as enforcing the client to request what kind of product they want (e.g Make("C"), Make("B"), etc..) The downside to this is that there is only one class that Make() will return, making the return type difficult to differentiate. This is the tradeoff between flexibility/extensibility and safety.

```
Product p = Make("A") // May or may not return ProductA 
```
We can downcast this:
```
Product p = (ProductA) Make("A") // Inherently unsafe
```

#### Add Inversion of Control
Give the factory as a parameter to the class/method using it
```
public ProductA handleProductA(ProductFactory factory) {
  ProductA p = factory.createProductA();
  p.handle();
  p.send();
}
```
This decouples the hard dependency between the method and a specific type of factory, allowing for more flexibility and easier testing.
