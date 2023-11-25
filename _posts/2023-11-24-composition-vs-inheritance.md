---
layout: post
title: "Composition versus Inheritance in Software Engineering"
excerpt: "A comparison between both approaches, revealing their distinct roles in shaping software engineering practices."
tags: [composition, inheritance, software engineering, oop, functional programming, programming]
date: 2023-11-25T21:10:08+01:00
comments: true
image:
  feature: posts/composition-versus-inheritance/cover-composition-versus-inheritance.jpg
  credit: PngTree
  creditlink: https://pngtree.com/free-backgrounds
---

Programming paradigms such as functional programming (FP) and object-oriented programming (OOP) are used in conjunction with some important principles and concepts in order to create modular and maintainable code. Today we'll take a look at the concepts of composition and inheritance, demonstrating their use with some examples.

## In this article

- [What is Composition?](#what-is-composition)
  - [Pros of using Composition](#pros-of-using-composition)
  - [Cons of using Composition](#cons-of-using-composition)
- [What is Inheritance?](#what-is-inheritance)
  - [Pros of using Inheritance](#pros-of-using-inheritance)
  - [Cons of using Inheritance](#cons-of-using-inheritance)
- [Composition over inheritance](#composition-over-inheritance)
- [Conclusion](#conclusion)

## What is Composition?

When organizing your codebase using composition, you need to think of a "has-a" relationship.

In functional programming, functions are first-class citizens, and composition is a powerful tool for building complex behavior by combining simpler functions.

```typescript
const square = (x: number): number => x * x;
const double = (x: number): number => x * 2;

const squareAndDouble = (x: number): number => double(square(x));

const result: number = squareAndDouble(3);
console.log(result); // Result is 18
```

squareAndDouble is composed of the square and double functions. This allows us to create a pipeline of operations, making the code more modular and readable.

Not only limited to functional programming, composition can also be used in object-oriented projects.

```typescript
class Engine {
  public start(): void {
    console.log("Engine started");
  }
}

class Car {
  private engine: Engine;

  public constructor() {
    this.engine = new Engine();
  }

  public start(): void {
    this.engine.start();
  }
}

const myCar = new Car();
myCar.start(); // This will start the car's engine
```

![Car](/images/posts/composition-versus-inheritance/dodge_viper.jpg "Car")

In this example, a Car ```has an``` Engine. Through composition, a Car is composed of an Engine. This allows for a flexible and modular design where the Car class can leverage the functionality of the Engine class without directly inheriting from it.

### Pros of using Composition

- ```Flexibility```: Composition allows for a more flexible and dynamic relationship between classes. Objects can be composed of other objects at runtime, providing more flexibility in creating complex behaviors.
- ```Code Reusability```: By encapsulating objects within other objects, you can reuse components in different contexts. This promotes code reusability and modular design.
- ```Avoidance of tight coupling```: Changes to one class do not directly impact other classes, reducing the risk of unintended side effects. This makes the codebase more maintainable.
- ```No fragile base class problem```: Unlike inheritance, changes to the base class do not affect the functionality of the derived classes. This avoids breaking existing code in derived classes, which is an issue that happens with inheritance.

### Cons of using Composition

- ```Indirect access```: Accessing the functionality of a composed object may require additional methods in the containing class. This can lead to more verbose code when compared with inheritance.
- ```More boilerplate code```: In some cases, composition can lead to more boilerplate code, especially when dealing with multiple levels of nesting.

## What is Inheritance?

When organizing your codebase using inheritance, you need to think of a "is-a" relationship.

Inheritance is a core concept in object-oriented programming, allowing for the creation of specialized classes that inherit behavior and attributes from more general classes.

```typescript
class Animal {
  protected speak(): void {
    console.log("Animal speaks");
  }
}

class Dog extends Animal {
  public bark(): void {
    console.log("Dog barks");
  }
}

const myDog = new Dog();
myDog.speak(); // This will call the speak method inherited from Animal
myDog.bark();  // This will call the bark method specific to Dog
```

![Dog](/images/posts/composition-versus-inheritance/husky.jpg "Dog")

In this example, the Animal class has a speak method and a Dog class extends Animal, as a dog ```is an``` animal. Through inheritance, the Dog class inherits the speak method and adds its own bark method. This demonstrates a "is-a" relationship.

### Pros of using Inheritance

- ```Code Reusability```: Inheritance allows for the reuse of code from a base class in derived classes. This can lead to a more concise and DRY (Don't Repeat Yourself) codebase.
- ```Polymorphism```: Inheritance supports polymorphism, allowing objects of derived classes to be treated as objects of the base class. This promotes a more generalized and abstract design.
- ```Less boilerplate code```: In some scenarios, inheritance can result in less boilerplate code compared to composition, especially for simple hierarchies.

### Cons of using Inheritance

- ```Tight Coupling```: Changes in the base class can affect all derived classes, leading to tight coupling. This can make the code more fragile and less adaptable to changes.
- ```Inflexibility```: The "is-a" relationship enforced by inheritance may lead to a less flexible design. If the relationship between classes changes, it can be challenging to adapt the code.
- ```Limited to single inheritance```: Many languages support single inheritance only, which restricts the ability to inherit from multiple classes. This limitation can be addressed with interfaces or mixins, but it adds complexity.

## Composition over inheritance

Composition over inheritance is a design principle that suggests what the name says: favoring composition over inheritance. This approach promotes flexibility, code reuse, and maintainability.

For example, if you have a Logger class, it can be part of other classes via composition:

```typescript
// Base class representing a Logger
class Logger {
  log(message: string): void {
    console.log(`[Log]: ${message}`);
  }
}

// Class representing a User with logging capability using composition
class User {
  private logger: Logger;

  constructor(name: string) {
    this.logger = new Logger();
    this.name = name;
  }

  private name: string;

  greet(): void {
    this.logger.log(`Hello, my name is ${this.name}`);
  }
}

// Class representing an Admin with logging capability using composition
class Admin {
  private logger: Logger;

  constructor(name: string) {
    this.logger = new Logger();
    this.name = name;
  }

  private name: string;

  announce(): void {
    this.logger.log(`Admin announcement: ${this.name} is here!`);
  }
}

// Example usage
const regularUser = new User('John');
regularUser.greet(); // Outputs: [Log]: Hello, my name is John

const systemAdmin = new Admin('Admin1');
systemAdmin.announce(); // Outputs: [Log]: Admin announcement: Admin1 is here!
```

This approach provides:

- Flexibility, as you can easily change or extend the behavior of User and Admin without being constrained by the limitations of a fixed inheritance hierarchy.
- Code reusability, as the Logger class can be reused in other contexts as well, making it modular and reusable
- Maintainability: Changes to the Logger class do not affect the functionality of other classes, reducing side effects.

If we used instead inheritance for this:

```typescript
// Using inheritance
class UserWithLogger extends Logger {
  private name: string;

  constructor(name: string) {
    super();
    this.name = name;
  }

  greet(): void {
    this.log(`Hello, my name is ${this.name}`);
  }
}

class AdminWithLogger extends Logger {
  private name: string;

  constructor(name: string) {
    super();
    this.name = name;
  }

  announce(): void {
    this.log(`Admin announcement: ${this.name} is here!`);
  }
}
```

We'd achieve the same logging functionality, but now there's a tight coupling between the Logger and User or Admin classes. Changes to the Logger class might affect the behavior of User and Admin. Using composition over inheritance helps avoid this issue.

## Conclusion

Choosing between composition and inheritance often involves a trade-off between flexibility, code reusability, and maintainability. In practice, a combination of both approaches is frequently used to leverage the benefits of each while mitigating their weaknesses. This allows for a more adaptable and modular design, where composition and inheritance are used where they make the most sense in the context of the application's requirements.

I'd be more inclined to use inheritance when thinking about a "is-a" relationship in the codebase and to use composition when thinking about a "has-a" relationship, so it depends on the implementation that needs to be achieved. In a way, composition should always be there if you want to use Dependency Injection in your project, since it would be a prerequisite for it, as DI uses composition.
