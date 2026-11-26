# OOP & Low-Level Design (LLD): Complete Interview Guide

> **Who this is for:** Sanidhya, for the backend SDE intern / new-grad **OOP + LLD round** (Amazon, Google, Zomato/Eternal, Flipkart, and others).
> **Language:** Java is the main language, because you use Java/Spring Boot at Amazon. C++ is shown where it differs in a way interviewers ask about (virtual, vtable, destructors, diamond problem, copy constructor, rule of 3/5, smart pointers, operator overloading).
> **Format of each topic:** Simple explanation → Key points → Code → Interview tip / common trap.

---

## Table of Contents

1. [How the OOP/LLD round works](#1-how-the-ooplld-round-works)
2. [OOP Basics: class, object, constructors, this, static, final, access modifiers](#2-oop-basics)
3. [The 4 Pillars of OOP](#3-the-4-pillars-of-oop)
4. [Abstraction vs Encapsulation (classic confusion)](#4-abstraction-vs-encapsulation)
5. [Inheritance in depth: types, diamond problem, C++ virtual inheritance](#5-inheritance-in-depth)
6. [Polymorphism in depth: overloading vs overriding, dynamic dispatch, vtable](#6-polymorphism-in-depth)
7. [Abstract class vs Interface](#7-abstract-class-vs-interface)
8. [Relationships: Association, Aggregation, Composition, Inheritance (UML)](#8-relationships-between-classes)
9. [equals() and hashCode()](#9-equals-and-hashcode)
10. [Immutability](#10-immutability)
11. [Shallow vs Deep copy (plus C++ copy constructor, rule of 3/5, smart pointers)](#11-shallow-vs-deep-copy)
12. [C++ operator overloading](#12-c-operator-overloading)
13. [Exception handling](#13-exception-handling)
14. [Generics basics](#14-generics-basics)
15. [Java memory basics: stack, heap, String pool, GC](#15-java-memory-basics)
16. [Comparable vs Comparator](#16-comparable-vs-comparator)
17. [Java Collections quick map](#17-java-collections-quick-map)
18. [Multithreading basics for OOP/LLD](#18-multithreading-basics-for-oopld)
19. [SOLID principles (bad code → good code)](#19-solid-principles)
20. [DRY, KISS, YAGNI](#20-dry-kiss-yagni)
21. [Design Patterns](#21-design-patterns)
22. [UML class diagram basics (text notation)](#22-uml-class-diagram-basics)
23. [LLD: the 7-step approach](#23-lld-the-7-step-approach)
24. [LLD worked designs](#24-lld-worked-designs)
    - 24.1 Parking Lot · 24.2 LRU Cache · 24.3 Rate Limiter · 24.4 Elevator · 24.5 BookMyShow · 24.6 Splitwise · 24.7 Notification Service · 24.8 Logger · 24.9 Vending Machine
25. [Most Asked OOP & LLD Interview Questions (Top 40)](#25-most-asked-oop--lld-interview-questions-top-40)
26. [Last-minute revision](#26-last-minute-revision)

---

## 1. How the OOP/LLD round works

**Simple explanation:** The interviewer gives you a problem like "Design a parking lot" or "Design Splitwise". You have 45–60 minutes. They want to see **clean classes**, **good names**, **correct relationships**, **sensible patterns**, and **working core code**. They do NOT want a distributed system (that is HLD).

**What they score:**
- Did you **ask questions** before coding? (requirements)
- Are your classes **small and focused**? (SRP)
- Can you **add a new type** without editing old code? (Open/Closed, interfaces)
- Did you handle **concurrency** where it matters? (two people booking the same seat)
- Is the code **readable and compilable-looking**?

**Interview tip:** Talk while you design. Say "I will make `PricingStrategy` an interface so we can add weekend pricing later." That one sentence shows SOLID + Strategy + extensibility.

**Common trap:** Jumping into code in minute 1. Spend the first 5 minutes on requirements and entities.

---

## 2. OOP Basics

### 2.1 Class and Object

**Simple explanation:**
- A **class** is a **blueprint** (the architect's drawing of a house).
- An **object** is a **real house** built from that drawing. You can build many houses from one drawing. Each house has its own paint colour (state), but all share the same design (behaviour).

**Key points:**
- Class = fields (data/state) + methods (behaviour).
- Object = an instance of a class, created with `new`, lives on the **heap** in Java.
- A reference variable (`Car c`) lives on the stack (if local) and points to the object.

```java
class Car {                       // blueprint
    String brand;                 // state
    int speed;

    void accelerate(int delta) {  // behaviour
        speed += delta;
    }
}

Car a = new Car();   // object 1 (house 1)
Car b = new Car();   // object 2 (house 2)
a.brand = "Tata"; b.brand = "Mahindra";  // each has its own state
```

**Interview tip:** If asked "Is a class a memory allocation?" → No. A class is a definition. Memory for fields is allocated when you create an object. (Class metadata itself is loaded once into the JVM's Metaspace.)

---

### 2.2 Constructors

**Simple explanation:** A constructor is the "setup step" that runs when the object is born. Same name as the class, no return type.

**Types:**
| Type | What it does |
|---|---|
| **Default** | No arguments. Java gives you one automatically **only if you write no constructor at all**. |
| **Parameterized** | Takes arguments to set initial state. |
| **Copy** | Builds a new object from an existing one. Java has no built-in copy constructor — you write it yourself. C++ generates one automatically. |

**Constructor chaining:**
- `this(...)` calls **another constructor of the same class**.
- `super(...)` calls the **parent class constructor**.
- Either must be the **first statement**. You cannot use both in the same constructor.
- If you don't write `super(...)`, Java inserts `super()` (no-arg) automatically. If the parent has no no-arg constructor → **compile error**.

```java
class Account {
    private final String id;
    private double balance;

    Account() {                       // default
        this("UNKNOWN", 0.0);         // chaining to parameterized
    }

    Account(String id, double balance) {  // parameterized
        this.id = id;
        this.balance = balance;
    }

    Account(Account other) {          // copy constructor (hand-written)
        this(other.id, other.balance);
    }
}

class SavingsAccount extends Account {
    private final double rate;
    SavingsAccount(String id, double bal, double rate) {
        super(id, bal);               // parent constructor first
        this.rate = rate;
    }
}
```

**Order of construction:** Parent constructor runs first, then child. (Think: foundation before walls.) In C++, destructors run in the **reverse** order: child first, then parent.

**C++ difference:**
```cpp
class Account {
    std::string id; double balance;
public:
    Account() : Account("UNKNOWN", 0.0) {}                 // delegating ctor (C++11)
    Account(std::string id, double b) : id(std::move(id)), balance(b) {} // initializer list
    Account(const Account& other) = default;               // copy ctor (compiler gives one anyway)
    ~Account() {}                                          // destructor
};
```

**Common traps:**
- "Can a constructor be `private`?" → Yes. Used in **Singleton** and factory methods.
- "Can a constructor be `final`/`static`/`abstract`?" → No.
- "Can a constructor return a value?" → No. If you add a return type, it becomes a normal method.
- "Are constructors inherited?" → No. But a child must call one of them via `super`.

---

### 2.3 The `this` keyword

**Simple explanation:** `this` means "the current object" — "me".

**Uses:**
1. Remove name clash: `this.id = id;`
2. Call another constructor: `this(...)`
3. Pass the current object: `listener.register(this);`
4. Return the current object for chaining (Builder): `return this;`

**Trap:** `this` cannot be used inside a `static` method, because a static method belongs to the class, not to any object.

---

### 2.4 The `static` keyword

**Simple explanation:** `static` means "belongs to the **class**, not to each object". Like the **society's common water tank** — one tank shared by all houses, not one per house.

**Key points:**
- **Static variable:** one copy shared by all objects (e.g., a counter of objects created).
- **Static method:** called on the class (`Math.max`). Cannot touch instance fields directly. Cannot be overridden (only **hidden**).
- **Static block:** runs once when the class is loaded.
- **Static nested class:** does not need an outer object (used in Builder pattern).

```java
class User {
    private static int count = 0;      // shared
    private final int id;

    static { System.out.println("User class loaded"); }  // runs once

    User() { id = ++count; }

    static int getCount() { return count; }  // no 'this' here
}
```

**Common trap:** "Can we override a static method?" → No. If child defines the same static method, it is **method hiding**. Which one runs depends on the **reference type**, not the object type.

```java
class A { static void hi() { System.out.println("A"); } }
class B extends A { static void hi() { System.out.println("B"); } }
A obj = new B();
obj.hi();   // prints "A"  (hiding, decided at compile time)
```

---

### 2.5 The `final` keyword

| Applied to | Meaning |
|---|---|
| Variable | Value cannot be reassigned (a constant). For objects: the **reference** can't change, but the object's insides still can. |
| Method | Cannot be **overridden**. |
| Class | Cannot be **extended** (e.g., `String`). |

```java
final List<String> names = new ArrayList<>();
names.add("Sandy");        // OK: object content changes
// names = new ArrayList<>(); // ERROR: reference can't change
```

**C++ equivalents:** `const` for values, `final` keyword for classes/virtual methods (C++11).

**Trap:** `final` vs `finally` vs `finalize`:
- `final` = keyword (constant / no override / no extend).
- `finally` = block that always runs after try/catch.
- `finalize()` = old GC hook, **deprecated** since Java 9. Don't use it.

---

### 2.6 Access modifiers

**Java:**
| Modifier | Same class | Same package | Subclass (other package) | Everywhere |
|---|---|---|---|---|
| `private` | ✅ | ❌ | ❌ | ❌ |
| *(default / package-private)* | ✅ | ✅ | ❌ | ❌ |
| `protected` | ✅ | ✅ | ✅ | ❌ |
| `public` | ✅ | ✅ | ✅ | ✅ |

**C++:** `private`, `protected`, `public`. Class members are `private` by default; `struct` members are `public` by default. C++ also has `friend` (a function/class that can access private members). Java has no `friend`.

**C++ inheritance access:** `class B : public A` (normal "is-a"), `: protected A`, `: private A` (implementation-only, rare). Java always behaves like `public` inheritance.

**Rule of thumb:** Make fields `private`. Expose only what's needed. Start strict, loosen only when needed.

**Trap:** When overriding, you **cannot reduce visibility** (a `public` method can't become `protected` in the child). You can increase it.

---

## 3. The 4 Pillars of OOP

Memory hook: **"A PIE"** → **A**bstraction, **P**olymorphism, **I**nheritance, **E**ncapsulation.

### 3.1 Encapsulation

**Simple explanation:** Put data and the methods that work on that data together in one class, and **hide the data** behind methods. Like a **medicine capsule** — you swallow the capsule, you don't handle the powder directly. Or like an **ATM**: you can't open the cash box, you can only use the buttons.

**Key points:**
- Fields `private`, access via methods.
- Methods **validate** changes, so the object can never be in a bad state.
- You can change the inside later without breaking callers.

```java
class BankAccount {
    private double balance;            // hidden

    public void deposit(double amt) {
        if (amt <= 0) throw new IllegalArgumentException("Amount must be positive");
        balance += amt;
    }

    public void withdraw(double amt) {
        if (amt > balance) throw new IllegalStateException("Insufficient funds");
        balance -= amt;
    }

    public double getBalance() { return balance; }   // read-only access, no setter
}
```

**Real-world example:** A Spring `@Service` keeps its repository `private` and exposes business methods like `placeOrder()`. Controllers can't touch the DB directly.

**Trap:** "Getters and setters for every field = encapsulation?" → **No.** A public setter for every field is barely better than public fields. Real encapsulation means exposing **behaviour** (`withdraw`) and protecting **invariants** (balance never negative).

---

### 3.2 Abstraction

**Simple explanation:** Show **what** something does, hide **how** it does it. You drive a car using the steering, brake and accelerator. You don't need to know how fuel injection works.

**Key points:**
- Achieved with **interfaces** and **abstract classes**.
- Callers depend on the "what" (contract), so the "how" can change.

```java
interface PaymentGateway {             // WHAT
    PaymentResult charge(String userId, long amountPaise);
}

class RazorpayGateway implements PaymentGateway {   // HOW (hidden from callers)
    public PaymentResult charge(String userId, long amountPaise) {
        // HTTP call, signatures, retries...
        return new PaymentResult(true, "rzp_123");
    }
}

record PaymentResult(boolean success, String txnId) {}
```

**Real-world example:** Your Adapter wrapping STT/TTS vendors behind one interface — the rest of the code only knows `transcribe(audio)`, not which vendor is behind it.

---

### 3.3 Inheritance

**Simple explanation:** A child class **reuses** and **extends** a parent class. "A `Dog` **is an** `Animal`." The child gets the parent's fields and methods for free and can add or change behaviour.

**Key points:**
- Java: `extends` (class), `implements` (interface).
- Java supports **single** class inheritance, but a class can implement **many** interfaces.
- Use it only for a true **is-a** relationship. Otherwise use composition.

```java
abstract class Vehicle {
    protected final String plate;
    Vehicle(String plate) { this.plate = plate; }
    abstract int wheels();
    String describe() { return plate + " with " + wheels() + " wheels"; }
}

class Bike extends Vehicle {
    Bike(String plate) { super(plate); }
    int wheels() { return 2; }
}

class Car extends Vehicle {
    Car(String plate) { super(plate); }
    int wheels() { return 4; }
}
```

**Real-world example:** In Spring, your custom exception `OrderNotFoundException extends RuntimeException`.

**Trap:** Inheritance gives **tight coupling** — a change in the parent can break all children ("fragile base class problem"). Prefer composition unless it's clearly is-a.

---

### 3.4 Polymorphism

**Simple explanation:** "Poly" = many, "morph" = forms. **One interface, many forms.** The same call does different things depending on the object. Example: you press "play" on a remote — the TV plays video, the speaker plays music. Same button, different behaviour.

**Two kinds:**
| Compile-time (static) | Runtime (dynamic) |
|---|---|
| Method **overloading** (and operator overloading in C++) | Method **overriding** |
| Decided by the **compiler** from argument types | Decided at **runtime** from the actual object type |
| Faster (no lookup) | Uses dynamic dispatch / vtable |

```java
// Compile-time: overloading
class Calc {
    int add(int a, int b) { return a + b; }
    double add(double a, double b) { return a + b; }
    int add(int a, int b, int c) { return a + b + c; }
}

// Runtime: overriding
interface Notifier { void send(String msg); }
class EmailNotifier implements Notifier { public void send(String m) { System.out.println("Email: " + m); } }
class SmsNotifier   implements Notifier { public void send(String m) { System.out.println("SMS: " + m); } }

List<Notifier> all = List.of(new EmailNotifier(), new SmsNotifier());
for (Notifier n : all) n.send("Order shipped");   // same call, different behaviour
```

**Real-world example:** Spring injects a `List<Notifier>` of all beans; your code loops and calls `send()` without knowing which channel each is.

**Trap:** Fields are **not** polymorphic in Java. `Parent p = new Child(); p.x` uses `Parent`'s field `x`. Only (non-static, non-private, non-final) **methods** are overridden.

---

## 4. Abstraction vs Encapsulation

This is the **#1 confusion question**. Learn this table.

| | Abstraction | Encapsulation |
|---|---|---|
| Idea | Hide **complexity** (the "how") | Hide **data** (the internal state) |
| Question it answers | "What does it do?" | "Who can touch the data?" |
| Level | **Design** level | **Implementation** level |
| Tools | Interfaces, abstract classes | `private` fields + public methods |
| Analogy | Car dashboard: you see "speed", not the engine | Engine is locked under the bonnet; only the mechanic opens it |
| Example | `PaymentGateway.charge()` | `private double balance` + `withdraw()` with checks |

**One-liner for interview:** "Abstraction is about **exposing only the essential behaviour**; encapsulation is about **protecting the data** that implements that behaviour. Abstraction is the outside view, encapsulation is the inside protection. They work together."

---

## 5. Inheritance in depth

### 5.1 Types of inheritance

```
Single:        A → B
Multilevel:    A → B → C
Hierarchical:  A → B, A → C
Multiple:      A, B → C          (Java: only via interfaces;  C++: yes, with classes)
Hybrid:        mix of the above  (can create the Diamond)
```

| Type | Java (classes) | Java (interfaces) | C++ |
|---|---|---|---|
| Single | ✅ | ✅ | ✅ |
| Multilevel | ✅ | ✅ | ✅ |
| Hierarchical | ✅ | ✅ | ✅ |
| Multiple | ❌ | ✅ | ✅ |
| Hybrid | ❌ (with classes) | ✅ | ✅ |

### 5.2 The Diamond Problem

**Simple explanation:** `B` and `C` both inherit from `A`. `D` inherits from both `B` and `C`. Now `D` has **two copies** of `A`, and if both `B` and `C` override `A.show()`, which one does `D` use? Confusion.

```
      A
     / \
    B   C
     \ /
      D
```

**Java's answer:**
1. **No multiple inheritance of classes**, so no state duplication.
2. With interfaces + `default` methods, if two interfaces give the same default method, the class **must override** it and can pick one with `X.super.method()`.

```java
interface Flyer  { default String move() { return "fly"; } }
interface Swimmer{ default String move() { return "swim"; } }

class Duck implements Flyer, Swimmer {
    @Override
    public String move() {                  // compile error if you don't override
        return Flyer.super.move() + " & " + Swimmer.super.move();
    }
}
```

Rules when there's a conflict: **(1) class wins** over interface default; **(2) more specific interface wins** (child interface over parent); **(3) otherwise you must override.**

**C++'s answer: virtual inheritance**

Without `virtual`: `D` has **two** `A` sub-objects → ambiguous.
```cpp
#include <iostream>
struct A { int x = 0; void show() { std::cout << "A\n"; } };
struct B : A {};
struct C : A {};
struct D : B, C {};

int main() {
    D d;
    // d.x = 5;        // ERROR: ambiguous (B::A::x or C::A::x?)
    d.B::x = 5;        // works, but there are two copies of x
}
```

With `virtual`: only **one shared** `A`.
```cpp
#include <iostream>
struct A { int x = 0; virtual void show() { std::cout << "A\n"; } virtual ~A() = default; };
struct B : virtual A {};
struct C : virtual A {};
struct D : B, C {};

int main() {
    D d;
    d.x = 5;          // OK: single shared A
    d.show();         // OK
}
```
Note: with virtual inheritance, the **most-derived class (`D`) is responsible for constructing `A`**. If `B` and `C` both override `show()` differently, `D` must also override it, or the call is ambiguous.

**Interview tip:** Say "Java avoids the diamond problem for state by banning multiple class inheritance; for default methods it forces the class to resolve the conflict. C++ allows it and solves duplication with virtual inheritance."

---

## 6. Polymorphism in depth

### 6.1 Overloading vs Overriding: rules

| Rule | Overloading | Overriding |
|---|---|---|
| Where | Same class (or child) | Child class |
| Method name | Same | Same |
| Parameters | **Must differ** (number/type/order) | **Must be same** |
| Return type | Can differ (but return type alone is NOT enough) | Same or **covariant** (a subtype) |
| Access | Any | Same or **wider** (not narrower) |
| Checked exceptions | Any | Same, narrower, or none — **not broader/new** |
| `static` methods | Can be overloaded | Cannot be overridden (hidden) |
| `private` / `final` | Can be overloaded | Cannot be overridden |
| Binding | Compile time | Runtime |

```java
class Animal {
    protected Animal create() throws java.io.IOException { return new Animal(); }
}
class Dog extends Animal {
    @Override
    public Dog create() { return new Dog(); }   // wider access, covariant return, no exception: all legal
}
```

**Overloading trap (very common):**
```java
void print(Object o) { System.out.println("Object"); }
void print(String s) { System.out.println("String"); }

Object x = "hello";
print(x);       // prints "Object" — overloading picks by REFERENCE type at compile time
print(null);    // prints "String" — most specific type wins
```

**Always use `@Override`.** If you mistype the signature, the compiler catches it.

### 6.2 Dynamic dispatch

**Simple explanation:** When you call `animal.sound()`, Java looks at the **actual object** in memory (not the variable type) and runs that object's version. This decision happens at runtime.

```java
Animal a = new Dog();   // reference type: Animal, object type: Dog
a.sound();              // runs Dog.sound()
```

- **Compile time:** the compiler checks that `Animal` has a `sound()` method (otherwise error).
- **Run time:** the JVM runs `Dog`'s `sound()`.

### 6.3 vtable explained simply

**Simple explanation:** Think of each class having a **menu card** (vtable) that says "for `sound`, go to this function". Every object carries a hidden pointer (**vptr**) to its class's menu card. When you call a virtual method, the program:
1. Follows the object's vptr to its class's vtable.
2. Looks up the slot for that method.
3. Jumps to the function in that slot.

```
Dog object                 Dog's vtable
+---------+               +---------------------+
| vptr ---+-------------> | slot0: Dog::sound   |
| name    |               | slot1: Animal::eat  |  ← not overridden, so parent's
+---------+               | slot2: Dog::~Dog    |
                          +---------------------+
```

- **One vtable per class** (not per object). **One vptr per object.**
- Cost: one extra pointer per object + one indirect jump per call. Very small.
- In **Java**, all non-static, non-private, non-final instance methods are virtual by default. The JVM uses a similar method table.
- In **C++**, methods are **not virtual by default**. You must write `virtual`.

### 6.4 C++: virtual functions, pure virtual, virtual destructor

```cpp
#include <iostream>
#include <memory>

class Animal {
public:
    virtual void sound() const { std::cout << "...\n"; }  // virtual: dynamic dispatch
    void eat() const { std::cout << "eating\n"; }         // non-virtual: static binding
    virtual ~Animal() { std::cout << "~Animal\n"; }        // VIRTUAL destructor: must have!
};

class Dog : public Animal {
public:
    void sound() const override { std::cout << "Woof\n"; } // 'override' = compiler check
    ~Dog() override { std::cout << "~Dog\n"; }
};

int main() {
    Animal* a = new Dog();
    a->sound();   // Woof   (virtual → looks at real object)
    delete a;     // ~Dog then ~Animal  (because destructor is virtual)
}
```

**Why virtual destructor?** If `~Animal()` is NOT virtual, `delete a` only calls `~Animal()`. `Dog`'s destructor never runs → **memory/resource leak** (undefined behaviour). **Rule: if a class has any virtual function, give it a virtual destructor.**

**Pure virtual function / abstract class in C++:**
```cpp
class Shape {
public:
    virtual double area() const = 0;   // pure virtual → Shape is abstract
    virtual ~Shape() = default;
};
class Circle : public Shape {
    double r;
public:
    explicit Circle(double r) : r(r) {}
    double area() const override { return 3.14159 * r * r; }
};
// Shape s;  // ERROR: cannot create object of abstract class
```

**Object slicing (C++ trap):**
```cpp
Dog d;
Animal a = d;   // SLICED: only the Animal part is copied; a.sound() prints "..."
Animal& r = d;  // OK: reference keeps the Dog → r.sound() prints "Woof"
```
Polymorphism in C++ works only through **pointers or references**, not through objects by value.

**Trap questions:**
- "Can a constructor be virtual?" → **No** (C++). The vptr isn't set up yet while constructing. Use a virtual `clone()` (Prototype) instead.
- "Calling a virtual function inside a constructor?" → In C++, it calls the **current class's** version, not the child's (the child part doesn't exist yet). In Java, it **does** call the child's override, which can see uninitialized child fields — a known bug source. Avoid calling overridable methods in constructors in both languages.

---

## 7. Abstract class vs Interface

**Simple explanation:**
- **Abstract class** = a **half-built house**. Some rooms are done (concrete methods, fields), some are left for you (abstract methods). "Is-a" with shared code.
- **Interface** = a **contract / job description**. "Anyone who can do these things can take this job." "Can-do" capability.

| | Abstract class | Interface |
|---|---|---|
| Keyword | `abstract class`, `extends` | `interface`, `implements` |
| Instance fields (state) | ✅ Yes | ❌ No (only `public static final` constants) |
| Constructors | ✅ Yes (called via `super`) | ❌ No |
| Methods | Abstract + concrete | Abstract + `default` + `static` (Java 8) + `private` (Java 9) |
| Access of methods | Any | `public` (except `private` helpers) |
| Multiple | A class extends **one** | A class implements **many** |
| Relationship | "is-a" (Dog is an Animal) | "can-do" (Dog can be `Comparable`, `Serializable`) |
| Best for | Sharing code + state among close relatives | Defining a capability across unrelated classes |

**Java 8+ interface features:**
```java
interface Shape {
    double area();                                   // abstract

    default String describe() {                      // default: has a body, can be overridden
        return getClass().getSimpleName() + " area=" + area();
    }

    static Shape unitSquare() { return () -> 1.0; }  // static: call as Shape.unitSquare()

    private void helper() {}                         // Java 9: private helper for defaults
}
```
Why default methods were added: to **add new methods to old interfaces** (like `List.forEach`, `Collection.stream()`) **without breaking** every existing implementation.

**Abstract class with a template:**
```java
abstract class Report {
    private final String title;                     // state
    protected Report(String title) { this.title = title; }

    public final String generate() {                // fixed algorithm
        return header() + body() + "\n-- end --";
    }
    private String header() { return "== " + title + " ==\n"; }
    protected abstract String body();               // child fills this
}
```

**When to use which:**
- Use an **interface** by default — for contracts (`PaymentGateway`, `Notifier`, `Repository`). It allows multiple implementation and easy mocking in tests.
- Use an **abstract class** when several closely related classes share **state and code**, and you want to force a common structure (Template Method).
- Common real design: **interface + abstract base class** (e.g., `List` interface + `AbstractList` base). Get both flexibility and reuse.

**C++:** No `interface` keyword. An "interface" is a class with only pure virtual functions and a virtual destructor.

**Trap:** "Can an abstract class have a constructor?" → **Yes**. It runs when a child is created. "Can it have zero abstract methods?" → **Yes**, it just can't be instantiated. "Can an interface have a constructor?" → **No**.

---

## 8. Relationships between classes

**Simple explanation (strength from weak to strong):**

| Relationship | Meaning | Lifetime | Analogy | UML (text) |
|---|---|---|---|---|
| **Dependency** | A uses B briefly (method parameter / local) | No ownership | You use a taxi for one ride | `A ..> B` |
| **Association** | A knows B (has a reference) | Independent | Teacher ↔ Student | `A --> B` |
| **Aggregation** | A **has** B, but B can live without A ("weak has-a") | Independent | Team has Players; team disbands, players remain | `A o-- B` (hollow diamond at A) |
| **Composition** | A **owns** B; B dies with A ("strong has-a") | Same lifetime | House has Rooms; demolish house → rooms gone | `A *-- B` (filled diamond at A) |
| **Inheritance** (Generalization) | B **is-a** A | — | Dog is an Animal | `B --\|> A` (hollow triangle at A) |
| **Realization** | B implements interface A | — | Duck implements Flyer | `B ..\|> A` |

```java
// Association: Doctor knows Patients, both live independently
class Doctor { private List<Patient> patients = new ArrayList<>(); }

// Aggregation: Department gets existing Professors from outside
class Department {
    private final List<Professor> profs;
    Department(List<Professor> profs) { this.profs = profs; }   // passed in, not created
}

// Composition: Order creates and owns its OrderItems
class Order {
    private final List<OrderItem> items = new ArrayList<>();
    void addItem(String sku, int qty) { items.add(new OrderItem(sku, qty)); }  // created inside
}
record OrderItem(String sku, int qty) {}

// Inheritance
class Professor extends Person {}
class Person {}
class Patient {}
```

**Composition vs Inheritance ("favour composition"):**

Bad (inheritance for reuse):
```java
class MyStack<T> extends ArrayList<T> {     // Now users can call add(0, x), remove(5)... breaks stack rules!
    void push(T x) { add(x); }
    T pop() { return remove(size() - 1); }
}
```
Good (composition):
```java
class MyStack<T> {
    private final Deque<T> data = new ArrayDeque<>();   // has-a
    void push(T x) { data.push(x); }
    T pop() { return data.pop(); }
    boolean isEmpty() { return data.isEmpty(); }
}
```

**Why composition is preferred:** looser coupling, you expose only what you want, can swap the part at runtime (Strategy), easier to unit test with mocks.

**Interview tip:** In an LLD, say the relationship out loud: "ParkingLot **composes** Floors (floors don't exist without the lot), Floor **composes** Spots, a Ticket is **associated** with a Vehicle."

---

## 9. equals() and hashCode()

**Simple explanation:** `==` checks "is this the **same object** in memory?" `equals()` checks "do these two objects **mean the same thing**?" `hashCode()` gives a number used to pick a **bucket** in HashMap/HashSet.

**The contract:**
1. If `a.equals(b)` is true → `a.hashCode() == b.hashCode()` **must** be true.
2. If hash codes are equal → objects **may or may not** be equal (collision is allowed).
3. `equals` must be: **reflexive** (a=a), **symmetric** (a=b ⇒ b=a), **transitive**, **consistent**, and `a.equals(null)` is false.

**Correct implementation:**
```java
import java.util.Objects;

final class Point {
    private final int x, y;
    Point(int x, int y) { this.x = x; this.y = y; }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Point)) return false;
        Point p = (Point) o;
        return x == p.x && y == p.y;
    }

    @Override
    public int hashCode() { return Objects.hash(x, y); }
}
// Java 16+: record Point(int x, int y) {}  → equals/hashCode/toString generated for you
```

**The HashMap bug if you break the contract (override `equals` only):**
```java
class BadPoint {
    int x, y;
    BadPoint(int x, int y) { this.x = x; this.y = y; }
    @Override public boolean equals(Object o) {
        return o instanceof BadPoint b && x == b.x && y == b.y;
    }
    // hashCode NOT overridden → uses identity hash (different for each object)
}

Map<BadPoint, String> m = new HashMap<>();
m.put(new BadPoint(1, 2), "A");
m.get(new BadPoint(1, 2));        // null!  Different hash → looks in a different bucket
Set<BadPoint> s = new HashSet<>();
s.add(new BadPoint(1, 2)); s.add(new BadPoint(1, 2));
s.size();                          // 2!  "Duplicates" in a Set
```

**How HashMap uses them:** `hashCode()` → choose bucket → inside the bucket, compare with `equals()`. Both must agree.

**Second bug — mutable keys:**
```text
Point-like key with mutable fields → put into map → change a field → hashCode changes →
map.get(key) returns null even with the SAME object. The entry is "lost".
```
Fix: use **immutable** objects as keys (String, Integer, records).

**Trap:** "Using `getClass() != o.getClass()` vs `instanceof`?" → `getClass` is strict (subclasses never equal), `instanceof` allows subclasses but can break symmetry if a subclass adds fields. For final classes / records, both are fine.

---

## 10. Immutability

**Simple explanation:** An immutable object **never changes after it is created**. Like a **printed certificate** — to change it, you print a new one. `String`, `Integer`, `LocalDate`, records are immutable.

**Why it matters:**
- **Thread-safe** for free (nobody can change it, so no locks needed).
- **Safe HashMap keys** (hash never changes).
- Easy to reason about and cache.

**Recipe to write an immutable class (5 rules):**
1. Make the class `final` (no subclass can add mutable behaviour).
2. Make all fields `private final`.
3. No setters.
4. **Defensive copy** mutable inputs in the constructor.
5. **Defensive copy** (or return unmodifiable views) for mutable fields in getters.

```java
import java.util.*;

public final class Order {
    private final String id;
    private final List<String> items;
    private final Date createdAt;               // Date is mutable (old API)

    public Order(String id, List<String> items, Date createdAt) {
        this.id = id;
        this.items = List.copyOf(items);        // defensive copy + unmodifiable
        this.createdAt = new Date(createdAt.getTime());   // defensive copy
    }

    public String getId() { return id; }
    public List<String> getItems() { return items; }      // already unmodifiable
    public Date getCreatedAt() { return new Date(createdAt.getTime()); }  // copy out

    public Order withItem(String item) {        // "change" = return a NEW object
        List<String> copy = new ArrayList<>(items);
        copy.add(item);
        return new Order(id, copy, createdAt);
    }
}
```

Modern Java: `public record Money(long paise, String currency) {}` — fields are final, no setters. But records do **not** deep-copy lists automatically; do it in a compact constructor.

**Why is String immutable? (very common)**
1. **String pool** — many references share one literal; if one could change it, all would change.
2. **Security** — file paths, URLs, DB usernames can't be changed after validation.
3. **Thread safety.**
4. **Hash caching** — `String` caches its hashCode, making it a fast HashMap key.

**Trap:** `final` ≠ immutable. A `final List` can still have elements added.

---

## 11. Shallow vs Deep copy

**Simple explanation:**
- **Shallow copy:** copy the outer object, but **share** the inner objects. Like photocopying a **list of addresses** — both copies point to the same houses.
- **Deep copy:** copy the outer object **and** all inner objects. Like building new houses too.

```java
class Address { String city; Address(String c) { city = c; } }

class Person {
    String name; Address address;
    Person(String n, Address a) { name = n; address = a; }

    Person shallowCopy() { return new Person(name, address); }                 // shares Address
    Person deepCopy()    { return new Person(name, new Address(address.city)); } // new Address
}

Person p1 = new Person("Sandy", new Address("Delhi"));
Person s = p1.shallowCopy();
s.address.city = "Pune";
System.out.println(p1.address.city);   // "Pune" — p1 changed too! (shallow)

Person d = p1.deepCopy();
d.address.city = "Mumbai";
System.out.println(p1.address.city);   // still "Pune" — independent (deep)
```

**Java `clone()`:** `Object.clone()` is **shallow** by default, needs `Cloneable`, and is considered badly designed. **Prefer copy constructors or static factory `copyOf`.**

### 11.1 C++ copy constructor, assignment, Rule of Three / Five

**Simple explanation:** In C++, when a class owns a raw resource (e.g., `new`ed memory), the default copy just copies the **pointer** (shallow). Two objects then point at the same memory, and both `delete` it → **double free** crash.

**Rule of Three:** If you write any one of **destructor, copy constructor, copy assignment**, you probably need all three.
**Rule of Five (C++11):** Add **move constructor** and **move assignment**.
**Rule of Zero:** Best of all — use `std::string`, `std::vector`, smart pointers, so you need none of them.

```cpp
#include <cstring>
#include <utility>

class Buffer {
    char* data; size_t size;
public:
    explicit Buffer(const char* s) : size(std::strlen(s)) {
        data = new char[size + 1];
        std::memcpy(data, s, size + 1);
    }
    ~Buffer() { delete[] data; }                                    // 1. destructor

    Buffer(const Buffer& o) : size(o.size) {                        // 2. copy ctor (DEEP)
        data = new char[size + 1];
        std::memcpy(data, o.data, size + 1);
    }
    Buffer& operator=(const Buffer& o) {                            // 3. copy assignment
        if (this != &o) {                                           // self-assignment check
            Buffer tmp(o);                                          // copy-and-swap idiom
            std::swap(data, tmp.data); std::swap(size, tmp.size);
        }
        return *this;
    }
    Buffer(Buffer&& o) noexcept : data(o.data), size(o.size) {      // 4. move ctor (steal)
        o.data = nullptr; o.size = 0;
    }
    Buffer& operator=(Buffer&& o) noexcept {                        // 5. move assignment
        if (this != &o) { delete[] data; data = o.data; size = o.size; o.data = nullptr; o.size = 0; }
        return *this;
    }
};
```

- **Copy constructor** runs: `Buffer b2 = b1;`, passing by value, returning by value (often elided).
- **Copy assignment** runs: `b2 = b1;` (both already exist).
- **Move** = "steal the guts" of a temporary instead of copying — fast.
- Why copy constructor takes `const Buffer&` (reference)? If it took by value, calling it would need a copy → which calls the copy constructor → **infinite recursion**.

### 11.2 Smart pointers (C++)

**Simple explanation:** A smart pointer is a wrapper that **deletes the object automatically** when it's no longer needed. This is **RAII** (Resource Acquisition Is Initialization): tie the resource's life to an object's scope.

| Smart pointer | Ownership | Copyable? | Use when |
|---|---|---|---|
| `std::unique_ptr<T>` | **One** owner | No (move only) | Default choice |
| `std::shared_ptr<T>` | **Shared**, reference counted | Yes | Many owners, last one deletes |
| `std::weak_ptr<T>` | Non-owning observer of a `shared_ptr` | Yes | Break **cycles** (parent↔child), caches |

```cpp
#include <memory>
#include <iostream>

struct Node {
    int val;
    std::shared_ptr<Node> next;
    std::weak_ptr<Node> prev;        // weak to avoid a reference cycle leak
    explicit Node(int v) : val(v) {}
    ~Node() { std::cout << "free " << val << "\n"; }
};

int main() {
    auto u = std::make_unique<Node>(1);        // freed at end of scope
    // auto u2 = u;                            // ERROR: can't copy unique_ptr
    auto u2 = std::move(u);                    // OK: ownership moved

    auto a = std::make_shared<Node>(2);
    auto b = std::make_shared<Node>(3);
    a->next = b; b->prev = a;                  // no cycle thanks to weak_ptr
    std::cout << a.use_count() << "\n";        // 1
}   // all freed automatically
```

**Trap:** Two `shared_ptr`s pointing at each other → reference count never hits 0 → **memory leak**. Fix with `weak_ptr`.

---

## 12. C++ operator overloading

**Simple explanation:** Give operators like `+`, `==`, `<<` a meaning for your own class. It's **compile-time polymorphism**. Java does **not** support user operator overloading (only `+` for String is built in).

```cpp
#include <iostream>

class Complex {
    double re, im;
public:
    Complex(double r = 0, double i = 0) : re(r), im(i) {}

    Complex operator+(const Complex& o) const { return {re + o.re, im + o.im}; }  // member
    bool operator==(const Complex& o) const { return re == o.re && im == o.im; }
    Complex& operator+=(const Complex& o) { re += o.re; im += o.im; return *this; }

    friend std::ostream& operator<<(std::ostream& os, const Complex& c) {        // non-member
        return os << c.re << "+" << c.im << "i";
    }
};

int main() {
    Complex a(1, 2), b(3, 4);
    std::cout << (a + b) << "\n";   // 4+6i
}
```

**Key points:**
- `<<` and `>>` with streams must be **non-member** (left side is `ostream`), usually `friend`.
- Cannot overload: `::`, `.`, `.*`, `?:`, `sizeof`.
- Cannot create new operators or change precedence.
- For `std::sort`/`std::set`/`std::map` on custom types, overload `operator<` (or pass a comparator — the C++ version of Java's Comparator).

---

## 13. Exception handling

**Simple explanation:** An exception is a signal that "something went wrong, I can't continue normally". You **throw** it, and someone up the call stack **catches** it.

**Java hierarchy:**
```
Throwable
├── Error                     (JVM problems: OutOfMemoryError, StackOverflowError — don't catch)
└── Exception
    ├── IOException, SQLException, ...   → CHECKED (compiler forces handle/declare)
    └── RuntimeException                  → UNCHECKED
        ├── NullPointerException
        ├── IllegalArgumentException
        ├── IllegalStateException
        └── IndexOutOfBoundsException
```

| Checked | Unchecked |
|---|---|
| Compiler forces `try/catch` or `throws` | Not forced |
| Recoverable, outside your control (file missing, network) | Programming bugs or bad input (null, bad index) |
| `IOException` | `NullPointerException` |

**try / catch / finally / try-with-resources:**
```java
// try-with-resources: any AutoCloseable is closed automatically, even on exception
try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
    return br.readLine();
} catch (FileNotFoundException e) {            // more specific first
    log.warn("missing file", e);
    return null;
} catch (IOException e) {
    throw new UncheckedIOException(e);          // wrap and rethrow, keep cause
} finally {
    metrics.increment("read.attempt");          // always runs
}
```

**Custom exceptions:**
```java
public class InsufficientBalanceException extends RuntimeException {
    private final long shortfall;
    public InsufficientBalanceException(long shortfall) {
        super("Short by " + shortfall + " paise");
        this.shortfall = shortfall;
    }
    public long getShortfall() { return shortfall; }
}
```
In Spring Boot, map it to an HTTP status with `@ControllerAdvice` + `@ExceptionHandler` (e.g., 409 / 422).

**Best practices:**
- Catch **specific** exceptions, not `Exception`.
- Never **swallow** (`catch (Exception e) {}`) — at least log.
- Keep the **cause** when wrapping: `new MyException("msg", e)`.
- Use unchecked exceptions for business errors in modern Spring code (simpler signatures, `@Transactional` rolls back on unchecked by default).

**Traps:**
- "Does `finally` run if there's a `return` in try?" → **Yes** (except `System.exit()` or JVM crash). A `return` inside `finally` **overrides** the try's return — never do that.
- `throw` (throws an object) vs `throws` (declares in signature).
- Catch order: child before parent, otherwise compile error (unreachable).
- **C++:** no checked exceptions, no `finally` → use **RAII** (destructors clean up). Mark non-throwing functions `noexcept`. Destructors should never throw.

---

## 14. Generics basics

**Simple explanation:** Generics let you write a class or method **once** for **any type**, with type safety. Like a **labelled box**: `Box<Apple>` only holds apples; the compiler stops you putting oranges in.

```java
class Box<T> {
    private T value;
    void set(T v) { value = v; }
    T get() { return value; }
}

Box<String> b = new Box<>();
b.set("hi");
String s = b.get();        // no cast needed
// b.set(5);               // compile error — type safety

// Generic method with a bound
static <T extends Comparable<T>> T max(List<T> list) {
    T best = list.get(0);
    for (T x : list) if (x.compareTo(best) > 0) best = x;
    return best;
}
```

**Wildcards — PECS: "Producer Extends, Consumer Super":**
```java
double sum(List<? extends Number> nums) { ... }   // reads Numbers (producer) → extends
void addInts(List<? super Integer> out) { out.add(1); }  // writes Integers (consumer) → super
```

**Type erasure:** Java generics exist only at compile time. At runtime, `List<String>` and `List<Integer>` are both just `List`. So you **can't** do `new T()`, `T[]` creation, or `instanceof List<String>`. Generics also can't use primitives (`List<int>` ✗ → `List<Integer>` ✓).

**C++ templates:** Similar idea, but the compiler generates a **separate copy of code per type** (no erasure), works with primitives, and is resolved fully at compile time.

---

## 15. Java memory basics

**Simple explanation:**
- **Stack** = your **desk**. Small, fast, per-thread. Holds method calls (frames), local primitives, and **references**. Cleared automatically when the method returns.
- **Heap** = the **warehouse**. Big, shared by all threads. Holds **all objects** and arrays. Cleaned by the **Garbage Collector**.

```java
void foo() {
    int n = 5;                       // stack
    User u = new User("Sandy");      // 'u' (reference) on stack, User object on heap
}                                     // frame popped; User object now unreachable → GC eligible
```

| Stack | Heap |
|---|---|
| Per thread | Shared |
| Locals, references, call frames | Objects, arrays, instance fields |
| Auto freed on return | Freed by GC |
| `StackOverflowError` (deep recursion) | `OutOfMemoryError` |

**String pool:**
```java
String a = "hello";               // pool
String b = "hello";               // same pooled object
String c = new String("hello");   // new heap object
a == b;          // true  (same object)
a == c;          // false (different objects)
a.equals(c);     // true  (same content)
c.intern() == a; // true  (intern returns the pooled one)
```
**Rule: compare strings with `equals()`, never `==`.**

**Garbage Collection (brief):**
- GC frees objects that are **unreachable** from "GC roots" (stack references, static fields, active threads).
- **Generational:** most objects die young. New objects go to the **Young Gen** (Eden); survivors are promoted to **Old Gen**. Minor GC is frequent and cheap; full GC is slower.
- Default collector in modern Java is **G1**; low-latency options are **ZGC/Shenandoah**.
- You **can't force** GC (`System.gc()` is only a hint).
- Memory leak in Java = objects still **referenced** but not needed (e.g., a static `Map` cache that keeps growing, listeners never removed).

**Class metadata** lives in **Metaspace** (replaced PermGen in Java 8).

**C++ difference:** no GC. You manage memory with `new/delete` or (better) RAII and smart pointers. Objects can live on the stack too (`Dog d;`).

---

## 16. Comparable vs Comparator

**Simple explanation:**
- **Comparable** = the object's **natural/default order**, built inside the class. "How do I compare myself to another?" — `compareTo`.
- **Comparator** = an **external** rule you pass in. Many possible orders. "Sort these by price, or by rating, or by name."

| | Comparable | Comparator |
|---|---|---|
| Package | `java.lang` | `java.util` |
| Method | `compareTo(T o)` | `compare(T a, T b)` |
| Where | Inside the class | Separate class / lambda |
| How many orders | One (natural) | Many |
| Used by | `Collections.sort(list)`, `TreeMap` keys by default | `list.sort(cmp)`, `new TreeMap<>(cmp)`, `PriorityQueue<>(cmp)` |

```java
class Product implements Comparable<Product> {
    String name; double price; double rating;
    Product(String n, double p, double r) { name = n; price = p; rating = r; }

    @Override
    public int compareTo(Product o) { return name.compareTo(o.name); }  // natural: by name
}

List<Product> ps = new ArrayList<>(List.of(new Product("Pen", 10, 4.5), new Product("Bag", 500, 4.8)));
Collections.sort(ps);                                           // by name (Comparable)
ps.sort(Comparator.comparingDouble(p -> p.price));              // by price
ps.sort(Comparator.comparingDouble((Product p) -> p.rating).reversed()
                  .thenComparing(p -> p.name));                 // rating desc, then name
```

**Trap:** Don't write `return this.price - o.price;` for ints that can overflow (or doubles that get truncated). Use `Integer.compare(a, b)` / `Double.compare(a, b)`.
**Trap:** `compareTo` should be **consistent with equals** — `TreeSet` uses `compareTo`, so two items comparing as 0 are treated as duplicates.

**C++ analogue:** `operator<` (natural) vs a comparator functor/lambda passed to `std::sort` or `std::priority_queue`.

---

## 17. Java Collections quick map

```
Collection
├── List   (ordered, duplicates OK)       ArrayList, LinkedList
├── Set    (no duplicates)                HashSet, LinkedHashSet, TreeSet
└── Queue/Deque                           ArrayDeque, PriorityQueue, LinkedList
Map (key → value, not a Collection)       HashMap, LinkedHashMap, TreeMap, ConcurrentHashMap
```

| Class | Backed by | get / contains | add / put | remove | Ordered? | Notes |
|---|---|---|---|---|---|---|
| `ArrayList` | Dynamic array | get(i) **O(1)**, contains O(n) | O(1) amortized at end | O(n) | Insertion | Best default list |
| `LinkedList` | Doubly linked list | get(i) O(n) | O(1) at ends | O(1) at ends / via iterator | Insertion | Rarely better than ArrayList/ArrayDeque |
| `HashSet` | HashMap | **O(1)** avg | O(1) avg | O(1) avg | No | Needs good equals/hashCode |
| `LinkedHashSet` | Hash + linked list | O(1) | O(1) | O(1) | Insertion | |
| `TreeSet` | Red-black tree | **O(log n)** | O(log n) | O(log n) | Sorted | floor/ceiling/first/last |
| `HashMap` | Array of buckets (list → tree when bucket ≥ 8, Java 8) | **O(1)** avg, O(log n) worst | O(1) avg | O(1) avg | No | One null key allowed |
| `LinkedHashMap` | Hash + linked list | O(1) | O(1) | O(1) | Insertion or **access** order | Easy LRU via `removeEldestEntry` |
| `TreeMap` | Red-black tree | O(log n) | O(log n) | O(log n) | Sorted keys | floorKey/ceilingKey |
| `PriorityQueue` | Binary heap | peek O(1) | offer **O(log n)** | poll O(log n) | Heap order | Min-heap by default |
| `ArrayDeque` | Circular array | O(1) ends | O(1) ends | O(1) ends | — | Best stack & queue |
| `ConcurrentHashMap` | Buckets + CAS + per-bin locks | O(1) | O(1) | O(1) | No | Thread-safe, no null keys/values |

**HashMap internals (1 minute answer):** Array of buckets (default 16, load factor 0.75). `index = hash(key) & (n-1)`. Collisions go into a linked list; if one bucket gets ≥ 8 entries (and table ≥ 64), it becomes a red-black tree. When size > capacity × 0.75 → **resize** (double) and rehash.

**Traps:**
- `ConcurrentModificationException`: modifying a list while looping with for-each. Use `iterator.remove()` or `removeIf`.
- `Arrays.asList()` is fixed-size; `List.of()` is fully immutable.
- `Hashtable`/`Vector` are old synchronized classes — use `ConcurrentHashMap` / `ArrayList` instead.

---

## 18. Multithreading basics for OOP/LLD

**Simple explanation:** Many threads = many cooks in one kitchen. If two cooks grab the same knife at once, trouble. **Shared mutable state** needs protection.

**Tools you must know:**

| Tool | What it gives |
|---|---|
| `synchronized` (method/block) | **Mutual exclusion** (one thread at a time) + **visibility** |
| `volatile` | **Visibility** (all threads see latest value) + no reordering. **Not atomic** for `count++` |
| `AtomicInteger`, `AtomicLong` | Lock-free atomic ops (`incrementAndGet`, `compareAndSet`) |
| `ReentrantLock` | Like synchronized + `tryLock(timeout)`, fairness |
| `ConcurrentHashMap` | Thread-safe map; atomic `computeIfAbsent`, `merge`, `putIfAbsent` |
| `ExecutorService` | Thread pool — don't create raw threads in production |

```java
class Counter {
    private int count = 0;
    public synchronized void inc() { count++; }   // count++ is read-modify-write: NOT atomic without lock
    public synchronized int get() { return count; }
}

class FastCounter {
    private final AtomicInteger count = new AtomicInteger();
    public void inc() { count.incrementAndGet(); }
}
```

**Thread-safe Singleton with double-checked locking (DCL):**
```java
public final class ConfigManager {
    private static volatile ConfigManager instance;   // volatile is REQUIRED
    private ConfigManager() {}

    public static ConfigManager getInstance() {
        if (instance == null) {                       // 1st check: no lock (fast path)
            synchronized (ConfigManager.class) {
                if (instance == null) {               // 2nd check: inside lock
                    instance = new ConfigManager();
                }
            }
        }
        return instance;
    }
}
```
**Why `volatile`?** `instance = new ConfigManager()` is 3 steps: allocate memory → run constructor → assign reference. The CPU/JIT may **reorder** to allocate → assign → construct. Another thread could see a **non-null but half-built** object. `volatile` forbids this reordering.

**ConcurrentHashMap — atomic per-key operations:**
```java
ConcurrentHashMap<String, AtomicInteger> hits = new ConcurrentHashMap<>();
hits.computeIfAbsent(userId, k -> new AtomicInteger()).incrementAndGet();  // safe
hits.merge(userId2, 1, Integer::sum);   // (on a Map<String,Integer>) atomic add
```
**Trap:** `if (!map.containsKey(k)) map.put(k, v);` is **check-then-act** — not atomic even on ConcurrentHashMap. Use `putIfAbsent` / `computeIfAbsent`.

**Deadlock:** Thread 1 holds lock A, waits for B; Thread 2 holds B, waits for A. **Fix:** always take locks in a **fixed global order** (e.g., by ID), or use `tryLock` with timeout.

**Race condition:** result depends on thread timing (e.g., two users booking the last seat). Fix with locks, atomic ops, or DB-level constraints.

---

## 19. SOLID principles

Memory hook: **S**ingle responsibility, **O**pen/closed, **L**iskov substitution, **I**nterface segregation, **D**ependency inversion.

### 19.1 S — Single Responsibility Principle

**Simple explanation:** A class should have **one reason to change**. A chef cooks; he doesn't also do the accounts.

❌ **Bad:**
```java
class InvoiceService {
    double calculateTotal(Invoice inv) { /* math */ return 0; }
    void saveToDb(Invoice inv)          { /* SQL */ }
    void sendEmail(Invoice inv)         { /* SMTP */ }
}
// 3 reasons to change: tax rules, DB schema, email provider
```
✅ **Good:**
```java
class InvoiceCalculator { double total(Invoice inv) { return 0; } }
class InvoiceRepository { void save(Invoice inv) { } }
class InvoiceMailer     { void send(Invoice inv) { } }

class InvoiceService {                       // coordinates, doesn't do everything
    private final InvoiceCalculator calc; private final InvoiceRepository repo; private final InvoiceMailer mailer;
    InvoiceService(InvoiceCalculator c, InvoiceRepository r, InvoiceMailer m) { calc = c; repo = r; mailer = m; }
    void process(Invoice inv) { calc.total(inv); repo.save(inv); mailer.send(inv); }
}
```
**Your example:** `PaymentService` doesn't send emails; `NotificationService` does.

### 19.2 O — Open/Closed Principle

**Simple explanation:** **Open for extension, closed for modification.** Add new features by **adding new classes**, not by editing working code. Like adding a new plug to a power strip, not rewiring the house.

❌ **Bad:**
```java
class DiscountCalculator {
    double discount(String type, double amt) {
        if (type.equals("FESTIVE")) return amt * 0.2;
        else if (type.equals("NEW_USER")) return amt * 0.1;
        // every new discount → edit this method → risk breaking old ones
        return 0;
    }
}
```
✅ **Good:**
```java
interface DiscountPolicy { double discount(double amt); }
class FestiveDiscount implements DiscountPolicy { public double discount(double a) { return a * 0.2; } }
class NewUserDiscount implements DiscountPolicy { public double discount(double a) { return a * 0.1; } }
// New one? Just add a class. Calculator code never changes.
class DiscountCalculator {
    double apply(DiscountPolicy p, double amt) { return amt - p.discount(amt); }
}
```
**Your example:** add a new `PaymentMethod` implementation without editing checkout code.

### 19.3 L — Liskov Substitution Principle

**Simple explanation:** A child must be usable **anywhere the parent is expected**, without surprises. If it looks like a duck but needs batteries, the abstraction is wrong.

❌ **Bad (classic Rectangle/Square trap):**
```java
class Rectangle {
    protected int w, h;
    void setWidth(int w) { this.w = w; }
    void setHeight(int h) { this.h = h; }
    int area() { return w * h; }
}
class Square extends Rectangle {
    @Override void setWidth(int w) { this.w = w; this.h = w; }   // surprise!
    @Override void setHeight(int h) { this.w = h; this.h = h; }
}
void test(Rectangle r) { r.setWidth(5); r.setHeight(4); assert r.area() == 20; }  // fails for Square (16)
```
Another bad one: `class Penguin extends Bird` where `Bird.fly()` throws `UnsupportedOperationException` for Penguin.

✅ **Good:**
```java
interface Shape { int area(); }
final class Rect implements Shape { private final int w, h; Rect(int w, int h) { this.w = w; this.h = h; } public int area() { return w * h; } }
final class Sq   implements Shape { private final int s; Sq(int s) { this.s = s; } public int area() { return s * s; } }

interface Bird {}
interface FlyingBird extends Bird { void fly(); }
class Sparrow implements FlyingBird { public void fly() {} }
class Penguin implements Bird {}        // no fly() promise to break
```
**LSP checklist:** child must not (1) throw new exceptions, (2) demand stronger inputs, (3) give weaker outputs, (4) break the parent's invariants.

### 19.4 I — Interface Segregation Principle

**Simple explanation:** Don't force a class to implement methods it doesn't use. Many **small, focused** interfaces are better than one **fat** one. A TV remote shouldn't have washing-machine buttons.

❌ **Bad:**
```java
interface Worker { void work(); void eat(); void sleep(); }
class Robot implements Worker {
    public void work() {}
    public void eat()   { throw new UnsupportedOperationException(); }  // forced
    public void sleep() { throw new UnsupportedOperationException(); }
}
```
✅ **Good:**
```java
interface Workable { void work(); }
interface Eatable  { void eat(); }
class Human implements Workable, Eatable { public void work() {} public void eat() {} }
class Robot implements Workable { public void work() {} }
```

### 19.5 D — Dependency Inversion Principle

**Simple explanation:** High-level code (business logic) should depend on **interfaces**, not on concrete low-level classes. Both depend on the abstraction. Like a lamp with a **standard plug** — it works with any socket, not soldered to one wire.

❌ **Bad:**
```java
class OrderService {
    private final StripeClient stripe = new StripeClient();   // hard-wired, can't test or swap
    void checkout(Order o) { stripe.pay(o.total()); }
}
```
✅ **Good:**
```java
interface PaymentGateway { void pay(long amount); }
class StripeGateway implements PaymentGateway { public void pay(long a) { } }
class SolanaPayGateway implements PaymentGateway { public void pay(long a) { } }

@Service
class OrderService {
    private final PaymentGateway gateway;
    OrderService(PaymentGateway gateway) { this.gateway = gateway; }   // injected (Spring DI)
    void checkout(Order o) { gateway.pay(o.total()); }
}
```
In tests, pass a `FakePaymentGateway`. In prod, Spring injects Stripe or Solana Pay.

**DIP vs DI vs IoC:**
- **DIP** = the principle (depend on abstractions).
- **DI (Dependency Injection)** = the technique (pass dependencies in, usually via constructor).
- **IoC (Inversion of Control)** = the framework creates and wires objects for you (Spring container).

**Interview tip:** Prefer **constructor injection** in Spring — fields can be `final`, dependencies are explicit, easy to test.

---

## 20. DRY, KISS, YAGNI

| Principle | Means | Example | Trap |
|---|---|---|---|
| **DRY** — Don't Repeat Yourself | Each piece of knowledge lives in **one** place | One `validateEmail()` used everywhere instead of 5 copied regexes | Don't merge code that only *looks* similar but changes for different reasons (wrong abstraction is worse than duplication) |
| **KISS** — Keep It Simple, Stupid | Simplest design that works | A `HashMap` instead of a custom tree, if it's enough | Over-engineering with 6 patterns for a 50-line problem |
| **YAGNI** — You Aren't Gonna Need It | Don't build features "just in case" | Don't add multi-currency until it's a requirement | In LLD interviews, **mention** extension points, but implement only what's asked |

Also good to name-drop: **Law of Demeter** ("talk only to your friends" — avoid `a.getB().getC().doX()`), **Composition over Inheritance**, **Program to an interface, not an implementation**, **High cohesion, low coupling**.

---

## 21. Design Patterns

**Simple explanation:** Design patterns are **proven recipes** for common design problems. Three families (Gang of Four):
- **Creational** — how objects are **created** (Singleton, Factory, Abstract Factory, Builder, Prototype).
- **Structural** — how objects are **composed** (Adapter, Decorator, Facade, Proxy, Composite).
- **Behavioral** — how objects **talk** and share work (Strategy, Observer, Command, State, Template Method, Chain of Responsibility).

### 21.0 Patterns map to your real work

| Pattern | One line | Where you have seen it (your work) |
|---|---|---|
| **Singleton** | One shared instance | DB connection pool / **Prisma client** (one client per process, reused, so you don't open a new pool per request) |
| **Factory** | Create objects without exposing the concrete class | Creating **handlers per event type** (event type → handler) |
| **Strategy** | Swap algorithms at runtime | Different **retry/backoff policies**; **payment methods** |
| **Observer / Pub-Sub** | Publisher notifies many subscribers, without knowing them | **SNS → SQS** fan-out, **Kafka consumers** (consumer groups subscribe to a topic), **WebSocket hub** (broadcast to connected clients) |
| **Builder** | Step-by-step construction of a complex object | **Solana `new Transaction().add(ix1).add(ix2)`** — chain instructions, then sign/send |
| **Adapter** | Make an incompatible interface fit the one you expect | **Wrapping STT/TTS vendors** behind one `SpeechToText` / `TextToSpeech` interface |
| **Decorator** | Add behaviour by wrapping, same interface | **Middleware** chains in Express (`app.use(auth)`) and Spring filters/interceptors (auth, logging). (Strictly, a middleware chain is also Chain of Responsibility — say both.) |
| **Circuit Breaker** | Stop calling a failing dependency for a while | Resilience in **LocalArena (ADR-0008)** |
| **Outbox** | Write DB change + event in one transaction, publish later | Reliable event publishing in **LocalArena** |

**Interview tip:** When asked "which patterns have you used?", pick 2–3 from this table and explain the **problem** each solved, not just the name.

---

### 21.1 Singleton (Creational)

**Simple explanation:** Only **one object** of a class exists, with a global access point. Like a country's **one Prime Minister** at a time.

**When to use:** shared resources — config, connection pool, logger, cache. **When not:** it hides dependencies and makes testing harder; in Spring, beans are singleton-scoped by default, so you rarely hand-write one.

**Key points:** private constructor, static instance, static accessor.

```java
// 1) Eager — simple, thread-safe (JVM class loading is thread-safe), created even if unused
public final class EagerSingleton {
    private static final EagerSingleton INSTANCE = new EagerSingleton();
    private EagerSingleton() {}
    public static EagerSingleton getInstance() { return INSTANCE; }
}

// 2) Lazy, NOT thread-safe — two threads can both see null and create two
public final class LazyUnsafe {
    private static LazyUnsafe instance;
    private LazyUnsafe() {}
    public static LazyUnsafe getInstance() {
        if (instance == null) instance = new LazyUnsafe();
        return instance;
    }
}

// 3) Double-checked locking with volatile (see Section 18)

// 4) Bill Pugh / holder idiom — lazy + thread-safe + no locking. Great default.
public final class HolderSingleton {
    private HolderSingleton() {}
    private static class Holder { static final HolderSingleton INSTANCE = new HolderSingleton(); }
    public static HolderSingleton getInstance() { return Holder.INSTANCE; }  // Holder loaded on first call
}

// 5) Enum singleton — Joshua Bloch's recommendation.
//    Thread-safe, safe against reflection AND serialization attacks.
public enum DbPool {
    INSTANCE;
    private final javax.sql.DataSource ds = createDataSource();
    public java.sql.Connection get() throws java.sql.SQLException { return ds.getConnection(); }
    private static javax.sql.DataSource createDataSource() { /* HikariCP etc. */ return null; }
}
// usage: DbPool.INSTANCE.get();
```

**How Singleton can be broken:**
- **Reflection** (`setAccessible(true)` on the private constructor) → guard: throw in constructor if instance exists, or use enum.
- **Serialization** (deserializing creates a new object) → add `readResolve() { return INSTANCE; }` or use enum.
- **Cloning** → don't implement `Cloneable` / throw in `clone()`.
- **Multiple class loaders** → separate instances per loader.

**Your example:** Prisma client / DB pool — creating one per request would exhaust DB connections, so one shared instance is reused.

---

### 21.2 Factory Method / Simple Factory (Creational)

**Simple explanation:** A method decides **which class to create** based on input. The caller asks for "a notifier for EMAIL" and doesn't need `new EmailNotifier()`. Like ordering at a counter — you say "pizza", the kitchen decides how to make it.

**When to use:** many subtypes, chosen at runtime; you want creation logic in one place.

```java
interface EventHandler { void handle(String payload); }
class OrderCreatedHandler implements EventHandler { public void handle(String p) { System.out.println("order " + p); } }
class PaymentFailedHandler implements EventHandler { public void handle(String p) { System.out.println("fail " + p); } }

class EventHandlerFactory {
    private final Map<String, Supplier<EventHandler>> registry = Map.of(
        "ORDER_CREATED",  OrderCreatedHandler::new,
        "PAYMENT_FAILED", PaymentFailedHandler::new
    );
    EventHandler create(String type) {
        Supplier<EventHandler> s = registry.get(type);
        if (s == null) throw new IllegalArgumentException("Unknown event: " + type);
        return s.get();
    }
}
```
Using a **map registry** instead of a `switch` keeps it closer to Open/Closed. (GoF "Factory Method" proper = an abstract `createX()` method that subclasses override; the idea is the same: defer which class to instantiate.)

**Your example:** creating handlers per event type when consuming messages from a queue.

---

### 21.3 Abstract Factory (Creational)

**Simple explanation:** A **factory of factories** — creates **families of related objects** that must match. Like choosing a furniture **set**: "Modern" gives a modern chair + modern table; "Victorian" gives both Victorian. You never mix.

**When to use:** you need several related objects that must be consistent (UI theme widgets, cloud provider clients: AWS storage + AWS queue vs GCP storage + GCP queue).

```java
interface Storage { void put(String k, byte[] v); }
interface Queue   { void send(String msg); }

interface CloudFactory { Storage storage(); Queue queue(); }

class AwsFactory implements CloudFactory {
    public Storage storage() { return (k, v) -> System.out.println("S3 put " + k); }
    public Queue queue()     { return m -> System.out.println("SQS send " + m); }
}
class GcpFactory implements CloudFactory {
    public Storage storage() { return (k, v) -> System.out.println("GCS put " + k); }
    public Queue queue()     { return m -> System.out.println("PubSub send " + m); }
}

class App {
    private final Storage storage; private final Queue queue;
    App(CloudFactory f) { storage = f.storage(); queue = f.queue(); }   // always a matching pair
}
```
**Factory vs Abstract Factory:** Factory creates **one product**; Abstract Factory creates a **family** of products.

---

### 21.4 Builder (Creational)

**Simple explanation:** Build a complex object **step by step**, then call `build()`. Avoids **telescoping constructors** (`new Pizza(true, false, true, null, 3, ...)`) — which arg is which? Like ordering at Subway: bread → filling → sauce → done.

**When to use:** many optional parameters, want an immutable result, want validation before creation.

```java
public final class HttpRequest {
    private final String url, method; private final Map<String, String> headers; private final String body; private final int timeoutMs;

    private HttpRequest(Builder b) {
        url = b.url; method = b.method; headers = Map.copyOf(b.headers); body = b.body; timeoutMs = b.timeoutMs;
    }
    public static Builder builder(String url) { return new Builder(url); }

    public static final class Builder {
        private final String url;                 // required
        private String method = "GET";            // optional with defaults
        private final Map<String, String> headers = new HashMap<>();
        private String body; private int timeoutMs = 5000;

        private Builder(String url) { this.url = Objects.requireNonNull(url); }
        public Builder method(String m) { this.method = m; return this; }   // return this → chaining
        public Builder header(String k, String v) { headers.put(k, v); return this; }
        public Builder body(String b) { this.body = b; return this; }
        public Builder timeoutMs(int t) { this.timeoutMs = t; return this; }
        public HttpRequest build() {
            if (body != null && method.equals("GET")) throw new IllegalStateException("GET with body");
            return new HttpRequest(this);
        }
    }
}

HttpRequest req = HttpRequest.builder("https://api.x.com/pay")
        .method("POST").header("Idempotency-Key", "abc").body("{...}").timeoutMs(2000).build();
```
In Spring/Java projects, **Lombok `@Builder`** generates this. Also `StringBuilder`, `Stream.builder()`, AWS SDK v2 request builders (`PutItemRequest.builder()...build()`).

**Your example:** Solana `new Transaction().add(instruction1).add(instruction2)` then sign and send — you assemble a complex transaction step by step.

---

### 21.5 Prototype (Creational)

**Simple explanation:** Create new objects by **cloning an existing one** (the prototype) instead of building from scratch. Like **photocopying a filled form** and changing only the name.

**When to use:** object creation is expensive (loaded from DB/config) and copies differ only a little; or you need a copy without knowing the concrete class.

```java
interface Prototype<T> { T copy(); }

class GameUnit implements Prototype<GameUnit> {
    private final String type; private int hp; private final List<String> skills;
    GameUnit(String type, int hp, List<String> skills) { this.type = type; this.hp = hp; this.skills = skills; }
    public GameUnit copy() { return new GameUnit(type, hp, new ArrayList<>(skills)); }  // deep copy of list
}

Map<String, GameUnit> registry = Map.of("archer", new GameUnit("archer", 100, List.of("shoot")));
GameUnit a1 = registry.get("archer").copy();
```
In C++, this is the **virtual `clone()`** idiom (since constructors can't be virtual):
```cpp
struct Shape { virtual std::unique_ptr<Shape> clone() const = 0; virtual ~Shape() = default; };
struct Circle : Shape { double r = 1; std::unique_ptr<Shape> clone() const override { return std::make_unique<Circle>(*this); } };
```
**Trap:** Decide shallow vs deep copy explicitly.

---

### 21.6 Adapter (Structural)

**Simple explanation:** Convert one interface into another that the client expects. Like a **travel plug adapter** — Indian plug into a UK socket.

**When to use:** integrating a third-party/legacy library whose API doesn't match your interface.

```java
// Your interface — the rest of your app only knows this
interface SpeechToText { String transcribe(byte[] audio); }

// Vendor SDKs with different shapes
class VendorASdk { VendorAResult recognize(java.io.InputStream in, String lang) { return new VendorAResult("hi"); } }
record VendorAResult(String text) {}
class VendorBClient { String[] speechToWords(byte[] pcm, int sampleRate) { return new String[]{"hi"}; } }

class VendorAAdapter implements SpeechToText {
    private final VendorASdk sdk;
    VendorAAdapter(VendorASdk sdk) { this.sdk = sdk; }
    public String transcribe(byte[] audio) {
        return sdk.recognize(new java.io.ByteArrayInputStream(audio), "en-IN").text();
    }
}
class VendorBAdapter implements SpeechToText {
    private final VendorBClient client;
    VendorBAdapter(VendorBClient c) { client = c; }
    public String transcribe(byte[] audio) { return String.join(" ", client.speechToWords(audio, 16000)); }
}
```
**Your example:** wrapping STT/TTS vendors behind one interface — switching vendor = new adapter, zero changes in the calling code.

**Adapter vs Facade vs Decorator vs Proxy** (all wrap something):
| Pattern | Interface after wrapping | Purpose |
|---|---|---|
| Adapter | **Different** (converted) | Make it fit |
| Facade | **New, simpler** over many classes | Make it easy |
| Decorator | **Same** | Add behaviour |
| Proxy | **Same** | Control access |

---

### 21.7 Decorator (Structural)

**Simple explanation:** Add behaviour to an object by **wrapping** it in another object with the **same interface**. Wraps can stack. Like a **coffee**: plain coffee → + milk → + sugar → + caramel. Each wrapper adds cost and description.

**When to use:** add optional features (logging, caching, metrics, auth) without subclass explosion (`LoggingCachingRetryingService`...).

```java
interface DataSource { String read(String key); }

class DbDataSource implements DataSource {
    public String read(String key) { return "value-from-db"; }
}

abstract class DataSourceDecorator implements DataSource {
    protected final DataSource inner;
    DataSourceDecorator(DataSource inner) { this.inner = inner; }
}

class LoggingDecorator extends DataSourceDecorator {
    LoggingDecorator(DataSource d) { super(d); }
    public String read(String k) {
        long t = System.nanoTime();
        String v = inner.read(k);
        System.out.println("read " + k + " took " + (System.nanoTime() - t) / 1000 + "us");
        return v;
    }
}

class CachingDecorator extends DataSourceDecorator {
    private final Map<String, String> cache = new ConcurrentHashMap<>();
    CachingDecorator(DataSource d) { super(d); }
    public String read(String k) { return cache.computeIfAbsent(k, inner::read); }
}

DataSource ds = new LoggingDecorator(new CachingDecorator(new DbDataSource()));  // stack them
```
Java I/O is the textbook example: `new BufferedReader(new InputStreamReader(new FileInputStream("f")))`.

**Your example:** Express middleware (`app.use(authMiddleware)`) and Spring filters/interceptors wrap the request handling with auth/logging without changing the handler.

---

### 21.8 Facade (Structural)

**Simple explanation:** One **simple front door** to a complex subsystem. Like a **travel agent**: you say "book my Goa trip"; they handle flights, hotel, cab.

**When to use:** callers shouldn't need to know 5 services and their order.

```java
class InventoryService { void reserve(String sku) {} }
class PaymentService   { String charge(String user, long amt) { return "txn1"; } }
class ShippingService  { void schedule(String orderId) {} }
class NotificationSvc  { void notify(String user, String msg) {} }

class CheckoutFacade {
    private final InventoryService inv; private final PaymentService pay;
    private final ShippingService ship; private final NotificationSvc notif;
    CheckoutFacade(InventoryService i, PaymentService p, ShippingService s, NotificationSvc n) { inv = i; pay = p; ship = s; notif = n; }

    public String placeOrder(String user, String sku, long amt) {   // one simple call
        inv.reserve(sku);
        String txn = pay.charge(user, amt);
        String orderId = "ord-" + txn;
        ship.schedule(orderId);
        notif.notify(user, "Order placed: " + orderId);
        return orderId;
    }
}
```
A Spring `@Service` that orchestrates several repositories/clients is often a facade. An **API Gateway** is a facade at system level.

---

### 21.9 Proxy (Structural)

**Simple explanation:** A **stand-in** object with the same interface that **controls access** to the real one. Like a **security guard / receptionist** — you talk to them first; they decide if/when you meet the boss.

**Types:** **Virtual proxy** (lazy-load heavy object), **Protection proxy** (auth checks), **Remote proxy** (gRPC/RMI stub), **Caching proxy**, **Logging proxy**.

```java
interface Image { void display(); }

class RealImage implements Image {
    private final String file;
    RealImage(String file) { this.file = file; System.out.println("Loading " + file); }  // expensive
    public void display() { System.out.println("Showing " + file); }
}

class LazyImageProxy implements Image {
    private final String file; private RealImage real;
    LazyImageProxy(String file) { this.file = file; }
    public void display() {
        if (real == null) real = new RealImage(file);   // load only when actually needed
        real.display();
    }
}
```
**Real-world:** Spring `@Transactional`, `@Cacheable`, `@Async` work through **dynamic proxies** (JDK proxy or CGLIB). That's why calling a `@Transactional` method from **inside the same class** (self-invocation) does NOT start a transaction — the call bypasses the proxy. Great interview point.

**Proxy vs Decorator:** same structure. Decorator **adds features** and is usually composed by the client; Proxy **controls access** and usually manages the real object's lifecycle.

---

### 21.10 Composite (Structural)

**Simple explanation:** Treat a **single object** and a **group of objects** the same way, in a tree. Like a **folder**: a folder contains files and other folders, and "get size" works on both.

**When to use:** tree structures — file systems, org charts, UI components, menu with sub-menus, nested categories.

```java
interface FileNode { long size(); String name(); }

record FileLeaf(String name, long size) implements FileNode {}

class Folder implements FileNode {
    private final String name; private final List<FileNode> children = new ArrayList<>();
    Folder(String name) { this.name = name; }
    Folder add(FileNode n) { children.add(n); return this; }
    public String name() { return name; }
    public long size() { return children.stream().mapToLong(FileNode::size).sum(); }  // recursive
}

FileNode root = new Folder("root")
        .add(new FileLeaf("a.txt", 100))
        .add(new Folder("img").add(new FileLeaf("p.png", 2000)));
root.size();   // 2100
```

---

### 21.11 Strategy (Behavioral)

**Simple explanation:** Define a family of algorithms, put each in its own class, and **swap them at runtime**. Like Google Maps: **car / bike / walk** — same "get route" button, different algorithm.

**When to use:** a big `if/else` or `switch` that picks behaviour; you want to add new behaviour without editing old code (OCP).

```java
interface BackoffStrategy { long delayMs(int attempt); }

class FixedBackoff implements BackoffStrategy {
    private final long ms; FixedBackoff(long ms) { this.ms = ms; }
    public long delayMs(int attempt) { return ms; }
}
class ExponentialBackoff implements BackoffStrategy {
    private final long baseMs, maxMs;
    ExponentialBackoff(long baseMs, long maxMs) { this.baseMs = baseMs; this.maxMs = maxMs; }
    public long delayMs(int attempt) {
        long exp = Math.min(maxMs, baseMs * (1L << Math.min(attempt, 20)));
        return java.util.concurrent.ThreadLocalRandom.current().nextLong(exp + 1);   // full jitter
    }
}

class Retrier {
    private final BackoffStrategy backoff;                 // composed, injectable
    Retrier(BackoffStrategy b) { backoff = b; }
    // uses backoff.delayMs(i) between attempts (see 21.18)
}
```
With Java lambdas, simple strategies can be lambdas: `BackoffStrategy fixed = a -> 100;`. `Comparator` is itself a Strategy.

**Your example:** different retry/backoff policies; payment methods (UPI/card/Solana Pay) behind one `PaymentStrategy`.

**Strategy vs State:** same shape. In Strategy, the **client chooses** the algorithm. In State, the **object switches its own state** based on events.

---

### 21.12 Observer (Behavioral)

**Simple explanation:** One object (**subject/publisher**) keeps a list of **observers/subscribers** and **notifies all** when something happens. They don't know each other's concrete types. Like **YouTube subscribe + bell** — new video → all subscribers notified.

**When to use:** one event, many independent reactions (order placed → email, SMS, analytics, inventory).

```java
interface OrderListener { void onOrderPlaced(String orderId); }

class OrderEventPublisher {
    private final List<OrderListener> listeners = new CopyOnWriteArrayList<>();  // safe iteration while (un)subscribing
    void subscribe(OrderListener l)   { listeners.add(l); }
    void unsubscribe(OrderListener l) { listeners.remove(l); }
    void publish(String orderId) {
        for (OrderListener l : listeners) {
            try { l.onOrderPlaced(orderId); }
            catch (RuntimeException e) { /* log; one bad listener must not stop others */ }
        }
    }
}

OrderEventPublisher pub = new OrderEventPublisher();
pub.subscribe(id -> System.out.println("Email for " + id));
pub.subscribe(id -> System.out.println("Update analytics " + id));
pub.publish("ord-42");
```
**Observer vs Pub/Sub:** In classic Observer, the subject calls observers **directly, in-process, synchronously**. In **Pub/Sub**, a **broker** sits in between (Kafka, SNS/SQS), so publisher and subscribers are fully decoupled, async, and can be on different machines.

**Your examples:**
- **SNS → SQS:** SNS topic is the subject; each SQS queue is a subscriber; one publish fans out to all.
- **Kafka consumers:** producers write to a topic; each consumer group gets every message.
- **WebSocket hub:** hub keeps a set of connected clients; on an event it broadcasts to all.

**Trap:** **Memory leak** — forgetting to unsubscribe keeps observers alive. Also, a slow observer blocks others in the synchronous version → use async/broker.

---

### 21.13 Command (Behavioral)

**Simple explanation:** Wrap a **request as an object** (with `execute()` and optionally `undo()`). You can queue it, log it, retry it, or undo it. Like a **restaurant order slip** — the waiter writes it, the chef executes it later.

**When to use:** undo/redo, job queues, macro recording, transactional operations, task schedulers.

```java
interface Command { void execute(); void undo(); }

class TextEditor { final StringBuilder text = new StringBuilder(); }

class AppendCommand implements Command {
    private final TextEditor ed; private final String s;
    AppendCommand(TextEditor ed, String s) { this.ed = ed; this.s = s; }
    public void execute() { ed.text.append(s); }
    public void undo()    { ed.text.setLength(ed.text.length() - s.length()); }
}

class CommandManager {
    private final Deque<Command> undo = new ArrayDeque<>(), redo = new ArrayDeque<>();
    void run(Command c) { c.execute(); undo.push(c); redo.clear(); }
    void undo() { if (!undo.isEmpty()) { Command c = undo.pop(); c.undo(); redo.push(c); } }
    void redo() { if (!redo.isEmpty()) { Command c = redo.pop(); c.execute(); undo.push(c); } }
}
```
`Runnable` submitted to an `ExecutorService` is a Command. A message in a job queue is a serialized Command.

---

### 21.14 State (Behavioral)

**Simple explanation:** An object **changes its behaviour when its internal state changes**. Each state is its own class. Like a **traffic light** or a **phone**: pressing the power button does different things when it's off, locked, or unlocked.

**When to use:** objects with clear states and transitions (vending machine, order lifecycle CREATED → PAID → SHIPPED → DELIVERED, elevator, document workflow). Replaces big `switch(state)` blocks scattered across methods.

```java
interface OrderState {
    default OrderState pay()     { throw new IllegalStateException("Cannot pay in " + name()); }
    default OrderState ship()    { throw new IllegalStateException("Cannot ship in " + name()); }
    default OrderState cancel()  { throw new IllegalStateException("Cannot cancel in " + name()); }
    String name();
}
class Created implements OrderState {
    public OrderState pay()    { return new Paid(); }
    public OrderState cancel() { return new Cancelled(); }
    public String name() { return "CREATED"; }
}
class Paid implements OrderState {
    public OrderState ship()   { return new Shipped(); }
    public OrderState cancel() { return new Cancelled(); }   // + refund
    public String name() { return "PAID"; }
}
class Shipped implements OrderState { public String name() { return "SHIPPED"; } }
class Cancelled implements OrderState { public String name() { return "CANCELLED"; } }

class Order {
    private OrderState state = new Created();
    void pay()    { state = state.pay(); }
    void ship()   { state = state.ship(); }
    void cancel() { state = state.cancel(); }
}
```
For simple cases, an `enum` with a transition table is fine. See the **Vending Machine** design (24.9) for a fuller example.

---

### 21.15 Template Method (Behavioral)

**Simple explanation:** A parent class defines the **skeleton of an algorithm** (fixed steps in fixed order); subclasses **fill in some steps**. Like a **recipe template**: "boil water → add ___ → steep ___ minutes → serve". Tea and coffee fill blanks differently.

**When to use:** several classes share the same flow but differ in a few steps (data import from CSV vs JSON, different payment flows with the same validation/logging).

```java
abstract class DataImporter {
    public final void importData(String path) {          // final: skeleton can't be changed
        String raw = read(path);
        List<String[]> rows = parse(raw);                 // varies
        validate(rows);
        save(rows);
        afterImport();                                    // hook (optional)
    }
    private String read(String path) { return "..."; }
    protected abstract List<String[]> parse(String raw);
    private void validate(List<String[]> rows) { if (rows.isEmpty()) throw new IllegalStateException("empty"); }
    private void save(List<String[]> rows) { }
    protected void afterImport() {}                       // default no-op hook
}

class CsvImporter extends DataImporter {
    protected List<String[]> parse(String raw) {
        List<String[]> out = new ArrayList<>();
        for (String line : raw.split("\n")) out.add(line.split(","));
        return out;
    }
}
```
Real-world: Spring's `JdbcTemplate`, `RestTemplate`, JUnit's `@BeforeEach → test → @AfterEach` lifecycle, `AbstractList`.

**Template Method vs Strategy:** Template uses **inheritance** (vary steps of one algorithm). Strategy uses **composition** (swap the whole algorithm). Strategy is more flexible.

---

### 21.16 Chain of Responsibility (Behavioral)

**Simple explanation:** Pass a request along a **chain of handlers**. Each one either handles it, or passes it to the next (or does something and passes it on). Like **customer support escalation**: bot → L1 agent → L2 engineer → manager.

**When to use:** request filtering/validation pipelines, middleware, approval workflows (amount < 10k → manager, < 1L → director...), log level handling.

```java
abstract class Handler {
    private Handler next;
    Handler linkWith(Handler next) { this.next = next; return next; }
    boolean handle(Request r) {                   // returns false = rejected
        return next == null || next.handle(r);
    }
}
record Request(String user, String token, String ip) {}

class AuthHandler extends Handler {
    boolean handle(Request r) {
        if (r.token() == null) { System.out.println("401"); return false; }
        return super.handle(r);
    }
}
class RateLimitHandler extends Handler {
    private final RateLimiter limiter; RateLimitHandler(RateLimiter l) { limiter = l; }
    boolean handle(Request r) {
        if (!limiter.allow(r.user())) { System.out.println("429"); return false; }
        return super.handle(r);
    }
}
class LoggingHandler extends Handler {
    boolean handle(Request r) { System.out.println("req from " + r.ip()); return super.handle(r); }
}

Handler chain = new LoggingHandler();
chain.linkWith(new AuthHandler()).linkWith(new RateLimitHandler(user -> true));
chain.handle(new Request("u1", "tok", "1.2.3.4"));
```
Real-world: **Servlet filter chain / Spring Security filter chain**, **Express middleware** (`next()`), Java logging handlers. See the **Logger** design (24.8).

---

### 21.17 Circuit Breaker (Backend / resilience)

**Simple explanation:** Like the **electrical MCB** in your house. If a downstream service keeps failing, **stop calling it** for a while ("trip"), fail fast, and try again later. This prevents your threads piling up waiting on a dead service, and gives it time to recover.

**States:**
```
CLOSED  --(failures >= threshold)-->  OPEN
OPEN    --(after cool-down timeout)-->  HALF_OPEN
HALF_OPEN --(trial call succeeds)-->  CLOSED
HALF_OPEN --(trial call fails)----->  OPEN
```

```java
import java.util.function.Supplier;

public class CircuitBreaker {
    enum State { CLOSED, OPEN, HALF_OPEN }

    private final int failureThreshold; private final long openTimeoutMs;
    private State state = State.CLOSED;
    private int failures = 0;
    private long openedAt = 0;

    public CircuitBreaker(int failureThreshold, long openTimeoutMs) {
        this.failureThreshold = failureThreshold; this.openTimeoutMs = openTimeoutMs;
    }

    public synchronized <T> T call(Supplier<T> action, Supplier<T> fallback) {
        if (state == State.OPEN) {
            if (System.currentTimeMillis() - openedAt >= openTimeoutMs) state = State.HALF_OPEN;
            else return fallback.get();                    // fail fast
        }
        try {
            T result = action.get();
            failures = 0; state = State.CLOSED;            // success resets
            return result;
        } catch (RuntimeException e) {
            failures++;
            if (state == State.HALF_OPEN || failures >= failureThreshold) {
                state = State.OPEN; openedAt = System.currentTimeMillis();
            }
            return fallback.get();
        }
    }
}
```
(Simplified: holding the lock during the call serialises calls. Production libraries like **Resilience4j** use sliding windows of failure rates, lock-free state, and allow a limited number of half-open trial calls.)

**Your example:** LocalArena's resilience design (**ADR-0008**) uses a circuit breaker so a failing dependency doesn't take the whole service down.

---

### 21.18 Retry with exponential backoff + jitter (Backend)

**Simple explanation:** If a call fails with a **temporary** error (timeout, 503, throttling), wait and try again — wait **longer each time** (100ms, 200ms, 400ms...), add **randomness (jitter)** so thousands of clients don't retry at the same instant (**thundering herd**).

**Rules:**
- Retry only **transient** errors (5xx, timeouts, 429). Never retry 400/401/404.
- Only retry **idempotent** operations — or send an **idempotency key** (e.g., payments).
- Cap max attempts and max delay.
- Combine with a **circuit breaker** so you don't hammer a dead service.

```java
public final class Retry {
    public static <T> T withBackoff(java.util.concurrent.Callable<T> op, int maxAttempts,
                                    BackoffStrategy backoff,
                                    java.util.function.Predicate<Exception> isRetryable) throws Exception {
        for (int attempt = 0; ; attempt++) {
            try {
                return op.call();
            } catch (Exception e) {
                if (attempt + 1 >= maxAttempts || !isRetryable.test(e)) throw e;
                Thread.sleep(backoff.delayMs(attempt));
            }
        }
    }
}
// Retry.withBackoff(() -> client.get(url), 5, new ExponentialBackoff(100, 5000), e -> e instanceof java.io.IOException);
```
Note how `BackoffStrategy` is a **Strategy** (21.11) — this is your "different retry/backoff policies" example.

---

### 21.19 Transactional Outbox (Backend)

**Simple explanation:** Problem — **dual write**: you save an order in the DB **and** publish "OrderCreated" to Kafka/SNS. If the DB commit succeeds but the publish fails (or vice versa), systems go out of sync.

**Solution:** In the **same DB transaction**, write the business row **and** an `outbox` row. A separate **relay/poller** (or CDC like Debezium) reads unsent outbox rows, publishes them, and marks them sent. Like writing a letter and putting it in your **outbox tray** in the same moment; the postman collects it later — it can't get lost.

```java
@Service
class OrderService {
    private final OrderRepository orders; private final OutboxRepository outbox;
    OrderService(OrderRepository o, OutboxRepository ob) { orders = o; outbox = ob; }

    @Transactional                                     // ONE transaction: both or neither
    public void placeOrder(Order o) {
        orders.save(o);
        outbox.save(new OutboxEvent(UUID.randomUUID(), "OrderCreated", o.getId(), toJson(o)));
    }
    private String toJson(Order o) { return "{}"; }
}

@Component
class OutboxRelay {
    private final OutboxRepository outbox; private final EventPublisher publisher;
    OutboxRelay(OutboxRepository ob, EventPublisher p) { outbox = ob; publisher = p; }

    @Scheduled(fixedDelay = 1000)
    public void relay() {
        for (OutboxEvent e : outbox.findUnsent(100)) {
            publisher.publish(e.type(), e.payload());    // may publish twice on crash → at-least-once
            outbox.markSent(e.id());
        }
    }
}
record OutboxEvent(UUID id, String type, String aggregateId, String payload) {}
```
**Key point:** delivery is **at-least-once**, so consumers must be **idempotent** (dedupe by event ID).

**Your example:** LocalArena uses the outbox pattern for reliable event publishing.

---

### 21.20 Repository (Backend)

**Simple explanation:** A class that acts like an **in-memory collection of domain objects** and hides the database. Business code says `repo.findById(id)`, not SQL. Like a **librarian** — you ask for a book by name; you don't care which shelf.

```java
interface UserRepository {
    Optional<User> findById(String id);
    List<User> findByCity(String city);
    void save(User u);
}

class InMemoryUserRepository implements UserRepository {      // for tests / LLD interviews
    private final Map<String, User> store = new ConcurrentHashMap<>();
    public Optional<User> findById(String id) { return Optional.ofNullable(store.get(id)); }
    public List<User> findByCity(String c) { return store.values().stream().filter(u -> u.city().equals(c)).toList(); }
    public void save(User u) { store.put(u.id(), u); }
}
record User(String id, String city) {}

// Spring Data JPA generates the implementation:
// interface UserJpaRepository extends JpaRepository<UserEntity, String> { List<UserEntity> findByCity(String city); }
```
**Interview tip:** In LLD rounds, use **in-memory repositories** behind interfaces. Say: "In production this would be a JPA/DynamoDB repository; the service doesn't change." That's DIP in action.

---

### 21.21 Dependency Injection (Backend)

**Simple explanation:** An object **receives** what it needs from outside instead of creating it with `new`. Like a **restaurant chef who gets ingredients delivered** instead of growing vegetables himself.

**Types:** **Constructor injection** (best — immutable, required deps explicit), setter injection (optional deps), field injection (`@Autowired` on fields — avoid, hard to test).

```java
// Without DI: hard-wired, untestable
class ReportService { private final EmailClient client = new SmtpEmailClient(); }

// With DI (constructor)
class ReportService2 {
    private final EmailClient client;
    ReportService2(EmailClient client) { this.client = client; }
}
// Test: new ReportService2(new FakeEmailClient());
// Spring: @Service + single constructor → Spring injects the EmailClient bean automatically.
interface EmailClient {}
class SmtpEmailClient implements EmailClient {}
class FakeEmailClient implements EmailClient {}
```
**Spring bean scopes:** `singleton` (default, one per container), `prototype` (new each time), `request`, `session`. Two beans of the same type → use `@Qualifier` or `@Primary`.

---

## 22. UML class diagram basics

**Simple explanation:** UML class diagrams are boxes (classes) and arrows (relationships). In interviews, a **text version** on the whiteboard/doc is enough.

**A class box:**
```
+-------------------------------+
|          ParkingLot           |   ← name (italics / <<abstract>> if abstract)
+-------------------------------+
| - id: String                  |   ← fields:  visibility name: Type
| - floors: List<Floor>         |
+-------------------------------+
| + park(v: Vehicle): Ticket    |   ← methods: visibility name(params): ReturnType
| + unpark(t: Ticket): Money    |
| # helper(): void              |
+-------------------------------+
```
**Visibility symbols:** `+` public, `-` private, `#` protected, `~` package. **Static** = underlined (write `{static}` in text). Interfaces: `<<interface>>`.

**Arrows (text notation used in this guide):**
```
A --|> B     A extends B            (inheritance: solid line, hollow triangle)
A ..|> B     A implements B         (realization: dashed line, hollow triangle)
A *-- B      A composes B           (composition: filled diamond at owner A; B dies with A)
A o-- B      A aggregates B         (aggregation: hollow diamond at A; B lives independently)
A --> B      A is associated with B (has a reference)
A ..> B      A depends on B         (uses temporarily, e.g. method parameter)
```
**Multiplicity:** `1`, `0..1`, `*` (many), `1..*`. Example: `ParkingLot "1" *-- "1..*" Floor`.

**Interview tip:** Draw only the important 6–10 classes with key fields/methods. Don't waste 10 minutes on a perfect diagram.

---

## 23. LLD: the 7-step approach

Use this **every time** for "Design X". Say the steps out loud so the interviewer sees structure.

| Step | What you do | Time (45-min round) |
|---|---|---|
| **1. Clarify requirements** | Functional (what must it do?) + non-functional (concurrency? scale? in-memory OK?). Write a numbered list. Confirm what's **out of scope**. | 3–5 min |
| **2. Identify entities & behaviours** | **Nouns → classes**, **verbs → methods**. Spot enums (types, statuses). | 3 min |
| **3. Define relationships & class diagram** | is-a vs has-a, composition vs aggregation, interfaces. Text UML. | 5–7 min |
| **4. Define key APIs / flows** | Public methods of the main service, and the 2–3 main flows as method-call sequences. | 3–5 min |
| **5. Apply patterns** | Only where they remove `if/else` or coupling: Strategy (pricing, assignment), Observer (notify), State (lifecycle), Factory (create by type), Singleton (one manager). | while coding |
| **6. Handle concurrency** | What is **shared and mutable**? What must be **atomic**? (last spot, last seat, balance). Choose: `synchronized`/locks, `ConcurrentHashMap`, atomic, DB constraint/optimistic lock. | 3–5 min |
| **7. Extensibility & edge cases** | "How do I add a new vehicle type / payment method / channel?" → new class, no edits (OCP). Mention failures, validation, tests. | 2–3 min |

Then **write the code** for the core classes and 1–2 main flows (about 20 minutes).

**Checklist while coding:**
- Fields `private final` where possible; use enums for fixed types.
- Interfaces for things that vary.
- Services hold logic; entities hold data + simple invariants.
- In-memory repositories (`ConcurrentHashMap`) behind interfaces.
- Throw meaningful exceptions (`SpotNotAvailableException`).
- Name things clearly. No `Manager1`, `doStuff`.

**Practice list:** Parking lot · LRU cache · Rate limiter · Elevator · BookMyShow · Splitwise · Notification service · Logger · Vending machine · Snake & ladder · Tic-tac-toe · Payment/checkout system (← talk about SolanaPay as a `PaymentGateway` implementation).

---

## 24. LLD worked designs

> Code is compact and focused on the core. In the interview, write the main classes fully and stub the rest.

### 24.1 Parking Lot

**1. Requirements**
- Multiple floors; each floor has spots of types: **SMALL** (bike), **MEDIUM** (car), **LARGE** (truck).
- Vehicle enters → system assigns a free compatible spot → issues a **ticket**.
- Vehicle exits → compute **fee** by duration and vehicle type → free the spot.
- Multiple entry/exit gates → **concurrent** parking must never give the same spot to two vehicles.
- Extensible: new vehicle types, new pricing (hourly, weekend), new spot-assignment rules (nearest, EV-first).

**2. Entities:** `ParkingLot`, `Floor`, `ParkingSpot`, `Vehicle` (+ types), `Ticket`, `SpotType`, `VehicleType`, `PricingStrategy`, `SpotAssignmentStrategy`, `ParkingService`.

**3. Class diagram (text)**
```
ParkingLot "1" *-- "1..*" Floor
Floor      "1" *-- "1..*" ParkingSpot
ParkingSpot --> Vehicle (0..1, currently parked)
Ticket --> Vehicle, Ticket --> ParkingSpot
ParkingService --> ParkingLot, PricingStrategy, SpotAssignmentStrategy
HourlyPricing ..|> PricingStrategy
FirstFreeSpotStrategy ..|> SpotAssignmentStrategy
enum VehicleType { BIKE, CAR, TRUCK }    enum SpotType { SMALL, MEDIUM, LARGE }
```

**4. Code**
```java
import java.time.*;
import java.util.*;
import java.util.concurrent.*;
import java.util.concurrent.atomic.*;

enum VehicleType { BIKE, CAR, TRUCK }
enum SpotType { SMALL, MEDIUM, LARGE }

record Vehicle(String plate, VehicleType type) {}

class ParkingSpot {
    private final String id; private final SpotType type;
    private final AtomicReference<Vehicle> parked = new AtomicReference<>();   // null = free

    ParkingSpot(String id, SpotType type) { this.id = id; this.type = type; }

    boolean tryOccupy(Vehicle v) { return parked.compareAndSet(null, v); }   // atomic: only one wins
    void release() { parked.set(null); }
    boolean isFree() { return parked.get() == null; }
    String id() { return id; }
    SpotType type() { return type; }
}

class Floor {
    private final int number; private final List<ParkingSpot> spots;
    Floor(int number, List<ParkingSpot> spots) { this.number = number; this.spots = List.copyOf(spots); }
    List<ParkingSpot> spots() { return spots; }
    int number() { return number; }
}

record Ticket(String id, Vehicle vehicle, ParkingSpot spot, Instant entryTime) {}

interface SpotAssignmentStrategy {
    Optional<ParkingSpot> assign(List<Floor> floors, Vehicle v);
}

class FirstFreeSpotStrategy implements SpotAssignmentStrategy {
    private static final Map<VehicleType, SpotType> FIT =
        Map.of(VehicleType.BIKE, SpotType.SMALL, VehicleType.CAR, SpotType.MEDIUM, VehicleType.TRUCK, SpotType.LARGE);

    public Optional<ParkingSpot> assign(List<Floor> floors, Vehicle v) {
        SpotType need = FIT.get(v.type());
        for (Floor f : floors)
            for (ParkingSpot s : f.spots())
                if (s.type() == need && s.isFree() && s.tryOccupy(v))   // CAS: retry next if someone beat us
                    return Optional.of(s);
        return Optional.empty();
    }
}

interface PricingStrategy { long feePaise(Ticket t, Instant exit); }

class HourlyPricing implements PricingStrategy {
    private static final Map<VehicleType, Long> RATE =
        Map.of(VehicleType.BIKE, 2000L, VehicleType.CAR, 5000L, VehicleType.TRUCK, 10000L);
    public long feePaise(Ticket t, Instant exit) {
        long minutes = Duration.between(t.entryTime(), exit).toMinutes();
        long hours = Math.max(1, (minutes + 59) / 60);                 // round up, min 1 hour
        return hours * RATE.get(t.vehicle().type());
    }
}

class ParkingFullException extends RuntimeException { ParkingFullException(String m) { super(m); } }

class ParkingService {
    private final List<Floor> floors;
    private final SpotAssignmentStrategy assigner; private final PricingStrategy pricing;
    private final Map<String, Ticket> activeTickets = new ConcurrentHashMap<>();
    private final AtomicLong seq = new AtomicLong();

    ParkingService(List<Floor> floors, SpotAssignmentStrategy a, PricingStrategy p) {
        this.floors = floors; this.assigner = a; this.pricing = p;
    }

    public Ticket park(Vehicle v) {
        ParkingSpot spot = assigner.assign(floors, v)
            .orElseThrow(() -> new ParkingFullException("No spot for " + v.type()));
        Ticket t = new Ticket("T" + seq.incrementAndGet(), v, spot, Instant.now());
        activeTickets.put(t.id(), t);
        return t;
    }

    public long unpark(String ticketId) {
        Ticket t = activeTickets.remove(ticketId);                    // remove is atomic → no double exit
        if (t == null) throw new IllegalArgumentException("Invalid/used ticket");
        long fee = pricing.feePaise(t, Instant.now());
        t.spot().release();
        return fee;
    }
}
```

**5. Concurrency notes**
- Two gates can try the same spot. `AtomicReference.compareAndSet(null, v)` guarantees **only one** succeeds; the loser moves to the next spot. No global lock → high throughput.
- `activeTickets.remove()` is atomic, so the same ticket can't be used to exit twice.
- With a DB: `UPDATE spot SET vehicle=? WHERE id=? AND vehicle IS NULL` and check "1 row updated" (same idea as CAS).

**6. Extension notes**
- New vehicle (EV) → add enum value + spot type + mapping; or an `EvSpot` with a charger.
- New pricing (weekend/flat) → new `PricingStrategy`. Nearest-to-gate → new `SpotAssignmentStrategy`.
- Display board showing free counts → **Observer** on spot occupy/release.
- Payment → `PaymentGateway` interface (cash/UPI/card).

**Interview trap:** Don't make `Vehicle` abstract with `Car`/`Bike` subclasses that add nothing — an enum is simpler (KISS). Use subclasses only if behaviour differs.

---

### 24.2 LRU Cache (generic, O(1))

**1. Requirements:** `get(key)` and `put(key, value)` in **O(1)**. Fixed capacity. When full, evict the **Least Recently Used** item. `get` counts as a use. Thread-safe version as follow-up.

**2. Idea:** **HashMap** (key → node, O(1) lookup) + **Doubly Linked List** (order of use, O(1) move/remove). Most recent at the head, least recent at the tail. Dummy head/tail nodes remove null checks.

```
HashMap: key → Node
DLL:  head <-> [most recent] <-> ... <-> [least recent] <-> tail
get:  find via map, move node to front
put:  if exists → update + move to front; else add to front; if size > cap → remove tail.prev
```

**3. Code**
```java
import java.util.*;

public class LRUCache<K, V> {
    private final class Node {
        K key; V val; Node prev, next;
        Node(K k, V v) { key = k; val = v; }
    }

    private final int capacity;
    private final Map<K, Node> map = new HashMap<>();
    private final Node head = new Node(null, null), tail = new Node(null, null);   // sentinels

    public LRUCache(int capacity) {
        if (capacity <= 0) throw new IllegalArgumentException("capacity must be > 0");
        this.capacity = capacity;
        head.next = tail; tail.prev = head;
    }

    public synchronized Optional<V> get(K key) {
        Node n = map.get(key);
        if (n == null) return Optional.empty();
        moveToFront(n);
        return Optional.of(n.val);
    }

    public synchronized void put(K key, V val) {
        Node n = map.get(key);
        if (n != null) { n.val = val; moveToFront(n); return; }
        if (map.size() == capacity) {
            Node lru = tail.prev;
            unlink(lru);
            map.remove(lru.key);              // why we store key in node: to remove from map
        }
        Node fresh = new Node(key, val);
        addFront(fresh);
        map.put(key, fresh);
    }

    private void moveToFront(Node n) { unlink(n); addFront(n); }
    private void unlink(Node n) { n.prev.next = n.next; n.next.prev = n.prev; }
    private void addFront(Node n) { n.next = head.next; n.prev = head; head.next.prev = n; head.next = n; }
}
```

**Shortcut (mention it, but interviewers usually want the manual one):**
```java
class LruMap<K, V> extends LinkedHashMap<K, V> {
    private final int cap;
    LruMap(int cap) { super(16, 0.75f, true); this.cap = cap; }   // true = access order
    @Override protected boolean removeEldestEntry(Map.Entry<K, V> e) { return size() > cap; }
}
```

**C++ version (common in DSA rounds):** `std::list<pair<K,V>>` + `unordered_map<K, list<pair<K,V>>::iterator>`, use `list.splice(list.begin(), list, it)` to move to front in O(1).

**4. Concurrency & extension notes**
- `synchronized` on both methods is simple and correct (note: even `get` **mutates** the list, so it needs the lock — a `ReadWriteLock` read-lock is NOT enough for `get`).
- High concurrency → **segment/shard** the cache (N small LRUs by `hash(key) % N`), or use **Caffeine** in real code.
- Extensions: TTL per entry (store `expiresAt`, check on get), eviction listener (Observer), pluggable `EvictionPolicy` (LRU/LFU) as **Strategy**.

**Trap:** Forgetting to store the **key inside the node** — then you can't remove the evicted entry from the map.

---

### 24.3 Rate Limiter (Token Bucket + Sliding Window)

**1. Requirements:** Limit each client (user/IP/API key) to N requests per time window. `allow(clientId)` returns true/false. Thread-safe. Pluggable algorithm. Rejected requests get **HTTP 429** with a `Retry-After` header.

**2. Algorithms (know the trade-offs):**
| Algorithm | Idea | Pros | Cons |
|---|---|---|---|
| **Token bucket** | Bucket holds up to `capacity` tokens, refills at `r`/sec; each request takes 1 | Allows short **bursts**, smooth average, O(1) memory | Two params to tune |
| Leaky bucket | Requests flow out at a fixed rate (queue) | Very smooth output | No bursts; queued latency |
| Fixed window counter | Count per minute bucket | Simplest | **Boundary burst**: 2× limit around the minute edge |
| **Sliding window log** | Store timestamps of requests in last W sec | Exact | Memory O(limit) per client |
| Sliding window counter | Weighted mix of current + previous window counts | Good approximation, O(1) memory | Approximate |

**3. Class diagram**
```
<<interface>> RateLimiter { allow(clientId): boolean }
TokenBucketLimiter   ..|> RateLimiter
SlidingWindowLogLimiter ..|> RateLimiter
RateLimitFilter --> RateLimiter        (Chain of Responsibility / middleware)
```

**4. Code**
```java
import java.util.*;
import java.util.concurrent.*;

interface RateLimiter { boolean allow(String clientId); }

// ---- Token bucket ----
class TokenBucketLimiter implements RateLimiter {
    private final int capacity; private final double refillPerSec;
    private final ConcurrentHashMap<String, Bucket> buckets = new ConcurrentHashMap<>();

    TokenBucketLimiter(int capacity, double refillPerSec) {
        this.capacity = capacity; this.refillPerSec = refillPerSec;
    }

    public boolean allow(String id) {
        Bucket b = buckets.computeIfAbsent(id, k -> new Bucket(capacity));   // atomic create
        synchronized (b) {                                                   // per-client lock, not global
            long now = System.nanoTime();
            b.tokens = Math.min(capacity, b.tokens + (now - b.last) / 1e9 * refillPerSec);  // lazy refill
            b.last = now;
            if (b.tokens >= 1) { b.tokens -= 1; return true; }
            return false;
        }
    }

    private static class Bucket {
        double tokens; long last = System.nanoTime();
        Bucket(int c) { tokens = c; }                                        // start full
    }
}

// ---- Sliding window log ----
class SlidingWindowLogLimiter implements RateLimiter {
    private final int limit; private final long windowNanos;
    private final ConcurrentHashMap<String, Deque<Long>> logs = new ConcurrentHashMap<>();

    SlidingWindowLogLimiter(int limit, long windowMillis) {
        this.limit = limit; this.windowNanos = windowMillis * 1_000_000L;
    }

    public boolean allow(String id) {
        Deque<Long> q = logs.computeIfAbsent(id, k -> new ArrayDeque<>());
        synchronized (q) {
            long now = System.nanoTime();
            while (!q.isEmpty() && now - q.peekFirst() >= windowNanos) q.pollFirst();  // drop old
            if (q.size() < limit) { q.addLast(now); return true; }
            return false;
        }
    }
}
```

**5. Talking points**
- **Lazy refill:** no background thread; compute tokens from elapsed time on each call. `System.nanoTime()` is monotonic (doesn't jump with clock changes).
- **Per-client lock** (`synchronized (b)`) instead of one global lock → clients don't block each other.
- **Memory:** idle clients' buckets pile up → evict with TTL (e.g., Caffeine cache with `expireAfterAccess`).
- **Distributed version:** many app servers must share counts → store bucket in **Redis**, and do read-refill-decrement in a **Lua script** so it's **atomic** on the Redis server.
- Return **HTTP 429 Too Many Requests** with `Retry-After`. Put the limiter in a filter/middleware (Chain of Responsibility, 21.16).
- Different limits per plan (free vs paid) → a `Map<Plan, RateLimiter>` or a `LimitConfigProvider` (Strategy).

---

### 24.4 Elevator System (brief)

**1. Requirements:** N elevators, M floors. Users press **hall buttons** (up/down on a floor) and **cab buttons** (destination inside). A controller assigns hall requests to the best elevator. Each elevator serves requests in its current direction before reversing (SCAN/LOOK). Extensible scheduling.

**2. Entities:** `ElevatorController` (Singleton per building), `Elevator`, `Direction` (UP/DOWN/IDLE), `Request`, `DispatchStrategy`.

**3. Class diagram**
```
ElevatorController "1" *-- "1..*" Elevator
ElevatorController --> DispatchStrategy
NearestElevatorStrategy ..|> DispatchStrategy
Elevator: currentFloor, direction, upStops: TreeSet, downStops: TreeSet
```

**4. Key code**
```java
import java.util.*;

enum Direction { UP, DOWN, IDLE }
record Request(int floor, Direction dir) {}

class Elevator {
    private final int id; private int floor = 0; private Direction dir = Direction.IDLE;
    private final TreeSet<Integer> up = new TreeSet<>(), down = new TreeSet<>();

    Elevator(int id) { this.id = id; }

    synchronized void addStop(int target) {
        if (target > floor) up.add(target); else if (target < floor) down.add(target);
        if (dir == Direction.IDLE) dir = target > floor ? Direction.UP : target < floor ? Direction.DOWN : Direction.IDLE;
    }

    synchronized void step() {                         // called every tick by a scheduler thread
        if (dir == Direction.UP) {
            Integer next = up.ceiling(floor);
            if (next == null) { dir = down.isEmpty() ? Direction.IDLE : Direction.DOWN; return; }
            floor++; if (floor == next) { up.remove(next); openDoors(); }
        } else if (dir == Direction.DOWN) {
            Integer next = down.floor(floor);
            if (next == null) { dir = up.isEmpty() ? Direction.IDLE : Direction.UP; return; }
            floor--; if (floor == next) { down.remove(next); openDoors(); }
        }
    }
    private void openDoors() { System.out.println("Elevator " + id + " at floor " + floor); }
    synchronized int floor() { return floor; }
    synchronized Direction direction() { return dir; }
}

interface DispatchStrategy { Elevator pick(List<Elevator> elevators, Request r); }

class NearestElevatorStrategy implements DispatchStrategy {
    public Elevator pick(List<Elevator> es, Request r) {
        // prefer idle or same-direction elevators that will pass this floor; fall back to nearest
        return es.stream().min(Comparator.comparingInt(e -> cost(e, r))).orElseThrow();
    }
    private int cost(Elevator e, Request r) {
        int d = Math.abs(e.floor() - r.floor());
        boolean onTheWay = (e.direction() == Direction.UP && r.floor() >= e.floor() && r.dir() == Direction.UP)
                        || (e.direction() == Direction.DOWN && r.floor() <= e.floor() && r.dir() == Direction.DOWN);
        return e.direction() == Direction.IDLE || onTheWay ? d : d + 100;   // penalty if going away
    }
}

class ElevatorController {
    private final List<Elevator> elevators; private final DispatchStrategy strategy;
    ElevatorController(List<Elevator> es, DispatchStrategy s) { elevators = es; strategy = s; }
    void hallCall(int floor, Direction dir) { strategy.pick(elevators, new Request(floor, dir)).addStop(floor); }
    void cabCall(Elevator e, int floor) { e.addStop(floor); }
}
```

**5. Notes:** **State pattern** fits elevator states (Idle, MovingUp, MovingDown, DoorsOpen, Maintenance). `TreeSet.ceiling/floor` gives the next stop in O(log n). Mention: emergency stop, weight limit, VIP/fire mode as new states or strategies. Concurrency: buttons arrive from many threads → `synchronized` per elevator.

---

### 24.5 BookMyShow-style Seat Booking (focus: concurrent seat locking)

**1. Requirements**
- Browse movies → theatres → shows. A show has a seat map.
- User selects seats → seats are **temporarily locked** (e.g., 5–10 minutes) while paying.
- Payment success → seats **BOOKED**. Payment fails / timeout → seats released.
- **Core rule: one seat can never be booked by two users** (no double booking), even with thousands of users clicking at the same moment.
- Booking multiple seats is **all-or-nothing**.

**2. Entities:** `Movie`, `Theatre`, `Screen`, `Seat` (physical), `Show` (movie + screen + time), `ShowSeat` (a seat **for a specific show**, with status), `Booking`, `SeatLockService`, `BookingService`, `PaymentGateway`.

**Key insight:** Status lives on **ShowSeat**, not Seat. Seat A1 is free for the 6 PM show but booked for 9 PM.

**3. Class diagram**
```
Theatre "1" *-- "*" Screen
Screen  "1" *-- "*" Seat
Show --> Movie, Show --> Screen
Show "1" *-- "*" ShowSeat          ShowSeat --> Seat
ShowSeat: status {AVAILABLE, LOCKED, BOOKED}, lockedBy, lockExpiresAt
Booking --> Show, Booking --> User, Booking "1" --> "1..*" ShowSeat
BookingService --> SeatLockService, PaymentGateway
```

**4. Code**
```java
import java.time.*;
import java.util.*;
import java.util.concurrent.*;

enum SeatStatus { AVAILABLE, LOCKED, BOOKED }
enum BookingStatus { PENDING, CONFIRMED, CANCELLED }

class ShowSeat {
    final String seatId;
    SeatStatus status = SeatStatus.AVAILABLE;
    String lockedBy; Instant lockExpiresAt;
    ShowSeat(String seatId) { this.seatId = seatId; }

    boolean isFreeFor(String user, Instant now) {
        return status == SeatStatus.AVAILABLE
            || (status == SeatStatus.LOCKED && (now.isAfter(lockExpiresAt) || user.equals(lockedBy)));  // expired lock = free
    }
}

class Show {
    final String id;
    final Map<String, ShowSeat> seats = new HashMap<>();   // guarded by 'this'
    Show(String id, List<String> seatIds) { this.id = id; seatIds.forEach(s -> seats.put(s, new ShowSeat(s))); }
}

class Booking {
    final String id, userId, showId; final List<String> seatIds; BookingStatus status = BookingStatus.PENDING;
    Booking(String id, String u, String s, List<String> seats) { this.id = id; userId = u; showId = s; seatIds = List.copyOf(seats); }
}

class SeatUnavailableException extends RuntimeException { SeatUnavailableException(String m) { super(m); } }

class SeatLockService {
    private final Duration lockTtl;
    SeatLockService(Duration ttl) { this.lockTtl = ttl; }

    /** All-or-nothing lock of several seats. Lock scope = one show, so different shows never block each other. */
    void lockSeats(Show show, List<String> seatIds, String userId) {
        synchronized (show) {
            Instant now = Instant.now();
            for (String id : seatIds) {                                    // 1) check ALL first
                ShowSeat s = show.seats.get(id);
                if (s == null || !s.isFreeFor(userId, now)) throw new SeatUnavailableException(id);
            }
            for (String id : seatIds) {                                    // 2) then lock ALL
                ShowSeat s = show.seats.get(id);
                s.status = SeatStatus.LOCKED; s.lockedBy = userId; s.lockExpiresAt = now.plus(lockTtl);
            }
        }
    }

    void confirm(Show show, List<String> seatIds, String userId) {
        synchronized (show) {
            Instant now = Instant.now();
            for (String id : seatIds) {
                ShowSeat s = show.seats.get(id);
                if (s.status != SeatStatus.LOCKED || !userId.equals(s.lockedBy) || now.isAfter(s.lockExpiresAt))
                    throw new SeatUnavailableException("Lock lost for " + id);    // paid too late
            }
            for (String id : seatIds) { ShowSeat s = show.seats.get(id); s.status = SeatStatus.BOOKED; s.lockedBy = null; }
        }
    }

    void release(Show show, List<String> seatIds, String userId) {
        synchronized (show) {
            for (String id : seatIds) {
                ShowSeat s = show.seats.get(id);
                if (s.status == SeatStatus.LOCKED && userId.equals(s.lockedBy)) { s.status = SeatStatus.AVAILABLE; s.lockedBy = null; }
            }
        }
    }
}

interface PaymentGateway { boolean charge(String userId, long amountPaise); }

class BookingService {
    private final Map<String, Show> shows = new ConcurrentHashMap<>();
    private final SeatLockService locks; private final PaymentGateway payments;
    BookingService(SeatLockService l, PaymentGateway p) { locks = l; payments = p; }

    void addShow(Show s) { shows.put(s.id, s); }

    Booking book(String userId, String showId, List<String> seatIds, long pricePaise) {
        Show show = Objects.requireNonNull(shows.get(showId), "no show");
        locks.lockSeats(show, seatIds, userId);                         // fast, in-memory critical section
        Booking b = new Booking(UUID.randomUUID().toString(), userId, showId, seatIds);
        boolean paid;
        try {
            paid = payments.charge(userId, pricePaise * seatIds.size());   // slow call OUTSIDE the lock
        } catch (RuntimeException e) {
            paid = false;
        }
        if (paid) { locks.confirm(show, seatIds, userId); b.status = BookingStatus.CONFIRMED; }
        else { locks.release(show, seatIds, userId); b.status = BookingStatus.CANCELLED; }
        return b;
    }
}
```
(Edge case to mention: if payment succeeds but `confirm` fails because the lock expired, trigger a **refund**.)

**5. Concurrency deep-dive (this is what they grade)**
- **Check-then-act must be atomic.** "Is seat free? → mark locked" in two steps without a lock = two users both see "free" → double booking.
- **Lock granularity:** one global lock = safe but slow. **Per-show lock** (used above) = different shows run in parallel. Per-seat locks = more parallel, but for multi-seat booking you must lock seats in a **sorted order** (by seat ID) to avoid **deadlock**.
- **Never hold the lock during payment.** Lock → mark LOCKED with expiry → release the Java lock → pay → confirm. The **seat status** is the long-lived "lock"; the Java lock is held only for microseconds.
- **Expiry:** expired locks are treated as free on the next check (lazy). Optionally a scheduled job cleans them up.
- **Multiple servers / DB-level (real systems):**
  - **Optimistic locking:** `UPDATE show_seat SET status='LOCKED', locked_by=?, version=version+1 WHERE show_id=? AND seat_id IN (...) AND status='AVAILABLE'` → if **rows updated ≠ seats requested**, roll back. Or JPA `@Version`.
  - **Pessimistic:** `SELECT ... FOR UPDATE` inside a transaction (sort seat IDs to avoid deadlocks).
  - **Unique constraint** on `(show_id, seat_id)` in the bookings table as the final safety net.
  - **Redis** `SET seat:{show}:{seat} user NX PX 600000` for a distributed temporary lock with TTL.

**Tie to your work — LocalArena "never overfill a match":** This is the same problem as your LocalArena concurrent-join test: many players join at the same moment and the match must **never go above capacity**. The check ("seats left > 0?") and the update ("add player") must be **one atomic step** — a lock, an atomic conditional update, or a DB constraint. The way to prove it is the same kind of test you wrote: fire many concurrent requests at once and assert the final count never exceeds capacity.

```java
// Concurrency test idea (JUnit): 100 users race for the same seat; exactly 1 must win.
@Test
void onlyOneUserGetsTheSeat() throws Exception {
    Show show = new Show("s1", List.of("A1"));
    SeatLockService locks = new SeatLockService(Duration.ofMinutes(5));
    int n = 100;
    ExecutorService pool = Executors.newFixedThreadPool(32);
    CountDownLatch start = new CountDownLatch(1);
    AtomicInteger winners = new AtomicInteger();
    List<Future<?>> fs = new ArrayList<>();
    for (int i = 0; i < n; i++) {
        String user = "u" + i;
        fs.add(pool.submit(() -> {
            start.await();                                   // all threads start together
            try { locks.lockSeats(show, List.of("A1"), user); winners.incrementAndGet(); }
            catch (SeatUnavailableException ignored) {}
            return null;
        }));
    }
    start.countDown();
    for (Future<?> f : fs) f.get();
    pool.shutdown();
    assertEquals(1, winners.get());                          // never "overfilled"
}
```

**6. Extensions:** pricing by seat category (Strategy), offers/coupons (Decorator or Strategy), notifications on booking (Observer), waitlist, cancellation + refund.

---

### 24.6 Splitwise (expense splitting + simplify debts)

**1. Requirements**
- Users and groups. Add an expense: **who paid**, **amount**, **participants**, **split type**: EQUAL, EXACT, PERCENT.
- Show balances: "A owes B ₹300".
- **Simplify debts:** minimise the number of payments needed to settle up.
- Money in **paise (long)** — never `double` for money (0.1 + 0.2 ≠ 0.3).

**2. Entities:** `User`, `Group`, `Expense`, `Split` (user + amount), `SplitStrategy` (Equal/Exact/Percent), `BalanceSheet`, `ExpenseService`, `DebtSimplifier`, `Transaction` (from, to, amount).

**3. Class diagram**
```
Group "1" o-- "*" User
Group "1" *-- "*" Expense
Expense --> User (paidBy), Expense "1" *-- "1..*" Split
<<interface>> SplitStrategy { computeSplits(total, users, params): List<Split> }
EqualSplit ..|> SplitStrategy, ExactSplit ..|> SplitStrategy, PercentSplit ..|> SplitStrategy
ExpenseService --> BalanceSheet, SplitStrategy (via factory by SplitType)
DebtSimplifier ..> BalanceSheet
```

**4. Code**
```java
import java.util.*;
import java.util.concurrent.*;

record User(String id, String name) {}
record Split(String userId, long amountPaise) {}
record Transaction(String from, String to, long amountPaise) {}
enum SplitType { EQUAL, EXACT, PERCENT }

interface SplitStrategy {
    List<Split> split(long total, List<String> userIds, List<Long> params);   // params: exact amounts or percents
}

class EqualSplit implements SplitStrategy {
    public List<Split> split(long total, List<String> users, List<Long> ignored) {
        long base = total / users.size(), rem = total % users.size();
        List<Split> out = new ArrayList<>();
        for (int i = 0; i < users.size(); i++)
            out.add(new Split(users.get(i), base + (i < rem ? 1 : 0)));    // spread leftover paise
        return out;
    }
}

class ExactSplit implements SplitStrategy {
    public List<Split> split(long total, List<String> users, List<Long> amounts) {
        if (amounts.stream().mapToLong(Long::longValue).sum() != total)
            throw new IllegalArgumentException("Exact amounts must add up to total");
        List<Split> out = new ArrayList<>();
        for (int i = 0; i < users.size(); i++) out.add(new Split(users.get(i), amounts.get(i)));
        return out;
    }
}

class PercentSplit implements SplitStrategy {
    public List<Split> split(long total, List<String> users, List<Long> pcts) {
        if (pcts.stream().mapToLong(Long::longValue).sum() != 100)
            throw new IllegalArgumentException("Percents must add up to 100");
        List<Split> out = new ArrayList<>(); long assigned = 0;
        for (int i = 0; i < users.size(); i++) {
            long amt = (i == users.size() - 1) ? total - assigned : total * pcts.get(i) / 100;  // last gets rounding
            assigned += amt; out.add(new Split(users.get(i), amt));
        }
        return out;
    }
}

class SplitStrategyFactory {
    private static final Map<SplitType, SplitStrategy> M =
        Map.of(SplitType.EQUAL, new EqualSplit(), SplitType.EXACT, new ExactSplit(), SplitType.PERCENT, new PercentSplit());
    static SplitStrategy of(SplitType t) { return M.get(t); }
}

record Expense(String id, String paidBy, long total, List<Split> splits, String note) {}

/** Net balance per user: positive = others owe them; negative = they owe. */
class BalanceSheet {
    private final ConcurrentHashMap<String, Long> net = new ConcurrentHashMap<>();
    void apply(Expense e) {
        net.merge(e.paidBy(), e.total(), Long::sum);                       // payer gets credit
        for (Split s : e.splits()) net.merge(s.userId(), -s.amountPaise(), Long::sum);  // each owes their share
    }
    Map<String, Long> snapshot() { return Map.copyOf(net); }
}

class ExpenseService {
    private final BalanceSheet sheet = new BalanceSheet();
    private final List<Expense> history = new CopyOnWriteArrayList<>();

    Expense addExpense(String paidBy, long total, List<String> users, SplitType type, List<Long> params, String note) {
        List<Split> splits = SplitStrategyFactory.of(type).split(total, users, params);
        Expense e = new Expense(UUID.randomUUID().toString(), paidBy, total, splits, note);
        synchronized (sheet) { sheet.apply(e); history.add(e); }          // one expense applied atomically
        return e;
    }
    Map<String, Long> balances() { return sheet.snapshot(); }
}

/** Greedy: repeatedly settle the biggest creditor with the biggest debtor. */
class DebtSimplifier {
    static List<Transaction> simplify(Map<String, Long> net) {
        PriorityQueue<Map.Entry<String, Long>> creditors = new PriorityQueue<>((a, b) -> Long.compare(b.getValue(), a.getValue()));
        PriorityQueue<Map.Entry<String, Long>> debtors   = new PriorityQueue<>((a, b) -> Long.compare(b.getValue(), a.getValue()));
        net.forEach((u, v) -> {
            if (v > 0) creditors.add(new AbstractMap.SimpleEntry<>(u, v));
            else if (v < 0) debtors.add(new AbstractMap.SimpleEntry<>(u, -v));   // store as positive debt
        });
        List<Transaction> result = new ArrayList<>();
        while (!creditors.isEmpty() && !debtors.isEmpty()) {
            var c = creditors.poll(); var d = debtors.poll();
            long amt = Math.min(c.getValue(), d.getValue());
            result.add(new Transaction(d.getKey(), c.getKey(), amt));
            if (c.getValue() > amt) creditors.add(new AbstractMap.SimpleEntry<>(c.getKey(), c.getValue() - amt));
            if (d.getValue() > amt) debtors.add(new AbstractMap.SimpleEntry<>(d.getKey(), d.getValue() - amt));
        }
        return result;
    }
}
```
**Example:** A paid ₹300 for A, B, C (equal). B paid ₹150 for B, C.
- Net: A = +300 − 100 = **+200**; B = +150 − 100 − 75 = **−25**; C = −100 − 75 = **−175**.
- Simplified: **C pays A ₹175**, **B pays A ₹25** → 2 transactions.

**5. Notes**
- Sum of all net balances is always **0** — good invariant to assert.
- Greedy gives **at most n−1** transactions. The true minimum is NP-hard (subset-sum style), so greedy is the accepted interview answer — say this.
- New split type (by shares, e.g., 2:1:1) → new `SplitStrategy` (OCP).
- Concurrency: adding expenses updates many balances → apply each expense atomically (shown with a lock; in a DB, one transaction).
- Extensions: multi-currency, settle-up records, activity feed + notifications (Observer).

---

### 24.7 Notification Service (Strategy + Observer, multiple channels, retries)

**1. Requirements**
- Send notifications over **multiple channels**: Email, SMS, Push (later WhatsApp).
- Business events (OrderPlaced, PaymentFailed) trigger notifications to interested users based on their **preferences**.
- **Retries** with backoff on transient failures; after max attempts → **dead-letter** for later inspection.
- **Async**: the caller (e.g., order service) must not wait for SMS delivery.
- **Idempotent**: the same event must not send duplicate notifications.

**2. Entities:** `Notification` (id, userId, channel, message), `NotificationChannel` (**Strategy** per channel), `ChannelFactory/Registry`, `NotificationService`, `EventBus` (**Observer**), `UserPreferenceRepository`, `RetryPolicy`, `DeadLetterStore`, `TemplateEngine` (optional).

**3. Class diagram**
```
<<interface>> NotificationChannel { type(): ChannelType; send(n: Notification) }
EmailChannel ..|> NotificationChannel, SmsChannel ..|> NotificationChannel, PushChannel ..|> NotificationChannel
<<interface>> EventListener { onEvent(e: DomainEvent) }
EventBus --> "*" EventListener                       (Observer)
NotificationService ..|> EventListener
NotificationService --> Map<ChannelType, NotificationChannel>, UserPreferenceRepository, RetryPolicy, DeadLetterStore
```

**4. Code**
```java
import java.util.*;
import java.util.concurrent.*;

enum ChannelType { EMAIL, SMS, PUSH }
record Notification(String id, String userId, ChannelType channel, String message) {}
record DomainEvent(String id, String type, String userId, Map<String, String> data) {}

class TransientDeliveryException extends RuntimeException { TransientDeliveryException(String m) { super(m); } }

// ---- Strategy: one class per channel ----
interface NotificationChannel { ChannelType type(); void send(Notification n); }

class EmailChannel implements NotificationChannel {
    public ChannelType type() { return ChannelType.EMAIL; }
    public void send(Notification n) { System.out.println("EMAIL to " + n.userId() + ": " + n.message()); }  // SES/SMTP adapter
}
class SmsChannel implements NotificationChannel {
    public ChannelType type() { return ChannelType.SMS; }
    public void send(Notification n) { System.out.println("SMS to " + n.userId() + ": " + n.message()); }    // vendor adapter
}
class PushChannel implements NotificationChannel {
    public ChannelType type() { return ChannelType.PUSH; }
    public void send(Notification n) { System.out.println("PUSH to " + n.userId() + ": " + n.message()); }   // FCM/APNs adapter
}

// ---- Observer: event bus ----
interface EventListener { void onEvent(DomainEvent e); }

class EventBus {
    private final Map<String, List<EventListener>> listeners = new ConcurrentHashMap<>();
    void subscribe(String eventType, EventListener l) {
        listeners.computeIfAbsent(eventType, k -> new CopyOnWriteArrayList<>()).add(l);
    }
    void publish(DomainEvent e) {
        listeners.getOrDefault(e.type(), List.of()).forEach(l -> l.onEvent(e));
    }
}

interface UserPreferenceRepository { Set<ChannelType> channelsFor(String userId, String eventType); }
interface DeadLetterStore { void save(Notification n, Exception cause); }

// ---- Service ----
class NotificationService implements EventListener {
    private final Map<ChannelType, NotificationChannel> channels = new EnumMap<>(ChannelType.class);
    private final UserPreferenceRepository prefs; private final DeadLetterStore dlq;
    private final ScheduledExecutorService pool = Executors.newScheduledThreadPool(8);
    private final Set<String> sentIds = ConcurrentHashMap.newKeySet();           // idempotency (use Redis/DB in prod)
    private final int maxAttempts = 4; private final long baseDelayMs = 200;

    NotificationService(List<NotificationChannel> chs, UserPreferenceRepository prefs, DeadLetterStore dlq) {
        chs.forEach(c -> channels.put(c.type(), c));                            // registry: add channel = add bean
        this.prefs = prefs; this.dlq = dlq;
    }

    @Override
    public void onEvent(DomainEvent e) {
        for (ChannelType ch : prefs.channelsFor(e.userId(), e.type())) {
            String id = e.id() + ":" + ch;                                     // deterministic → dedupe
            Notification n = new Notification(id, e.userId(), ch, render(e));
            pool.execute(() -> attempt(n, 1));                                  // async: caller never waits
        }
    }

    private void attempt(Notification n, int attemptNo) {
        if (sentIds.contains(n.id())) return;                                   // already delivered
        try {
            channels.get(n.channel()).send(n);
            sentIds.add(n.id());
        } catch (TransientDeliveryException ex) {
            if (attemptNo >= maxAttempts) { dlq.save(n, ex); return; }
            long delay = ThreadLocalRandom.current().nextLong(baseDelayMs * (1L << attemptNo));  // exp backoff + jitter
            pool.schedule(() -> attempt(n, attemptNo + 1), delay, TimeUnit.MILLISECONDS);   // non-blocking retry
        } catch (RuntimeException ex) {
            dlq.save(n, ex);                                                    // permanent error: don't retry
        }
    }

    private String render(DomainEvent e) {
        return switch (e.type()) {
            case "ORDER_PLACED"   -> "Your order " + e.data().get("orderId") + " is placed";
            case "PAYMENT_FAILED" -> "Payment failed for " + e.data().get("orderId");
            default -> e.type();
        };
    }
}

// wiring
// EventBus bus = new EventBus();
// NotificationService ns = new NotificationService(List.of(new EmailChannel(), new SmsChannel(), new PushChannel()), prefsRepo, dlqStore);
// bus.subscribe("ORDER_PLACED", ns);
// bus.publish(new DomainEvent("evt-1", "ORDER_PLACED", "u1", Map.of("orderId", "o-9")));
```

**5. Notes**
- **Strategy:** each channel is pluggable. New channel (WhatsApp) = new class + register it. No `if (channel == SMS)` anywhere.
- **Observer:** the order service only publishes an event; it doesn't know notifications exist (loose coupling).
- **Retry:** only on **transient** errors; exponential backoff with jitter; scheduled retry so threads aren't blocked sleeping; **DLQ** after max attempts.
- **Idempotency:** deterministic notification ID (`eventId:channel`) + "already sent" store. Note the in-memory check-then-send can still double-send on a crash between send and mark — real systems accept **at-least-once** and dedupe with a provider idempotency key where possible.
- **At scale (connect to your work):** replace the in-process `EventBus` with **SNS → SQS** (fan-out per channel queue) or **Kafka** topics; workers per channel; SQS **redrive policy** gives the DLQ for free; rate-limit per vendor (24.3); the event should be written with the **Outbox** pattern (21.19) so it's never lost.
- Other extensions: templates per locale, user quiet hours, priority (OTP > marketing), per-channel circuit breaker (21.17).

---

### 24.8 Logger (Singleton + Chain of Responsibility)

**1. Requirements**
- Levels: DEBUG < INFO < WARN < ERROR.
- A global logger (`Logger.getInstance()`).
- Messages flow through a **chain of handlers by level**; each handler handles its level and passes on (classic CoR design).
- Multiple outputs ("appenders/sinks"): console, file. Configurable minimum level.
- Thread-safe (many threads logging).

**2. Entities:** `LogLevel` (enum), `LogMessage`, `Logger` (Singleton), `LogHandler` (abstract, chain), `DebugHandler`/`InfoHandler`/`WarnHandler`/`ErrorHandler`, `LogSink` (Console/File — Strategy/Observer).

**3. Class diagram**
```
Logger (singleton) --> LogHandler (head of chain)
LogHandler --> LogHandler (next)                   ← chain
DebugHandler, InfoHandler, WarnHandler, ErrorHandler --|> LogHandler
LogHandler --> "*" LogSink
ConsoleSink ..|> LogSink, FileSink ..|> LogSink
```

**4. Code**
```java
import java.io.*;
import java.time.Instant;
import java.util.*;
import java.util.concurrent.*;

enum LogLevel { DEBUG, INFO, WARN, ERROR }       // ordinal gives the order

record LogMessage(LogLevel level, String text, String thread, Instant at) {
    String format() { return at + " [" + level + "] [" + thread + "] " + text; }
}

interface LogSink { void write(LogMessage m); }

class ConsoleSink implements LogSink {
    public synchronized void write(LogMessage m) { System.out.println(m.format()); }
}
class FileSink implements LogSink {
    private final PrintWriter out;
    FileSink(String path) throws IOException { out = new PrintWriter(new FileWriter(path, true), true); }
    public synchronized void write(LogMessage m) { out.println(m.format()); }   // one line at a time, no interleaving
}

// ---- Chain of Responsibility ----
abstract class LogHandler {
    private final LogLevel level; private LogHandler next;
    protected final List<LogSink> sinks;
    LogHandler(LogLevel level, List<LogSink> sinks) { this.level = level; this.sinks = sinks; }

    LogHandler setNext(LogHandler next) { this.next = next; return next; }

    void handle(LogMessage m) {
        if (m.level() == level) write(m);        // my level → I handle it
        else if (next != null) next.handle(m);   // otherwise pass along
    }
    protected void write(LogMessage m) { sinks.forEach(s -> s.write(m)); }
}
class DebugHandler extends LogHandler { DebugHandler(List<LogSink> s) { super(LogLevel.DEBUG, s); } }
class InfoHandler  extends LogHandler { InfoHandler(List<LogSink> s)  { super(LogLevel.INFO, s); } }
class WarnHandler  extends LogHandler { WarnHandler(List<LogSink> s)  { super(LogLevel.WARN, s); } }
class ErrorHandler extends LogHandler {
    ErrorHandler(List<LogSink> s) { super(LogLevel.ERROR, s); }
    @Override protected void write(LogMessage m) { super.write(m); /* e.g. also page on-call / push metric */ }
}

// ---- Singleton (holder idiom) ----
public final class Logger {
    private volatile LogLevel minLevel = LogLevel.INFO;
    private final LogHandler chain;

    private Logger() {
        List<LogSink> console = List.of(new ConsoleSink());
        LogHandler head = new DebugHandler(console);
        head.setNext(new InfoHandler(console)).setNext(new WarnHandler(console)).setNext(new ErrorHandler(console));
        this.chain = head;
    }
    private static class Holder { static final Logger INSTANCE = new Logger(); }
    public static Logger getInstance() { return Holder.INSTANCE; }

    public void setMinLevel(LogLevel l) { minLevel = l; }

    public void log(LogLevel level, String msg) {
        if (level.ordinal() < minLevel.ordinal()) return;            // filter early, cheap
        chain.handle(new LogMessage(level, msg, Thread.currentThread().getName(), Instant.now()));
    }
    public void debug(String m) { log(LogLevel.DEBUG, m); }
    public void info(String m)  { log(LogLevel.INFO, m); }
    public void warn(String m)  { log(LogLevel.WARN, m); }
    public void error(String m) { log(LogLevel.ERROR, m); }
}

// Logger.getInstance().info("Server started");
```

**5. Notes**
- **Singleton** via holder idiom → lazy and thread-safe without locks. `volatile minLevel` so a level change is visible to all threads.
- **CoR** lets you add a new level/handler or reorder without touching `Logger`. A variant: each handler handles "this level **and above**" and always passes on (e.g., ERROR also goes to an alert sink).
- **Thread safety:** sinks are `synchronized` so lines don't interleave.
- **Performance:** real loggers (Log4j2, Logback) use **async appenders** — a `BlockingQueue` + background writer thread, so request threads don't block on disk I/O. Mention it.
- Extensions: JSON format (Strategy for `Formatter`), log rotation, per-class loggers, MDC/trace IDs for request correlation.

---

### 24.9 Vending Machine (State pattern)

**1. Requirements**
- Machine holds products in slots with price and quantity.
- Flow: **select product → insert money → dispense product + change**. Can **cancel** and get a refund.
- Behaviour depends on state: you can't dispense before paying; you can't insert money when sold out.
- Admin can restock.

**2. States:** `IDLE` (waiting for selection) → `HAS_SELECTION` (waiting for money) → `DISPENSING` → back to `IDLE`. Any state with an empty inventory → `SOLD_OUT`.

**3. Class diagram**
```
VendingMachine --> VendingState (current), Inventory
<<interface>> VendingState { select, insertMoney, dispense, cancel }
IdleState, HasSelectionState, DispensingState, SoldOutState ..|> VendingState
Inventory "1" *-- "*" Slot      Slot: code, Product, qty
```

**State transition table:**
| State | select(code) | insertMoney(x) | cancel() |
|---|---|---|---|
| Idle | → HasSelection (if in stock) | reject | nothing |
| HasSelection | change selection | add balance; if ≥ price → Dispensing → dispense → Idle | refund → Idle |
| Dispensing | reject (busy) | reject | reject |
| SoldOut | reject | reject (refund) | refund |

**4. Code**
```java
import java.util.*;

record Product(String name, int pricePaise) {}

class Slot { final Product product; int qty; Slot(Product p, int q) { product = p; qty = q; } }

class Inventory {
    private final Map<String, Slot> slots = new HashMap<>();
    void add(String code, Product p, int qty) { slots.merge(code, new Slot(p, qty), (a, b) -> { a.qty += b.qty; return a; }); }
    Slot get(String code) { return slots.get(code); }
    boolean isEmpty() { return slots.values().stream().allMatch(s -> s.qty == 0); }
}

interface VendingState {
    default void select(VendingMachine m, String code) { throw new IllegalStateException("Can't select now"); }
    default void insertMoney(VendingMachine m, int paise) { throw new IllegalStateException("Can't insert money now"); }
    default void cancel(VendingMachine m) {}
}

class IdleState implements VendingState {
    public void select(VendingMachine m, String code) {
        Slot s = m.inventory().get(code);
        if (s == null || s.qty == 0) throw new IllegalArgumentException("Unavailable: " + code);
        m.setSelected(code);
        m.setState(new HasSelectionState());
    }
}

class HasSelectionState implements VendingState {
    public void select(VendingMachine m, String code) { new IdleState().select(m, code); }  // change mind
    public void insertMoney(VendingMachine m, int paise) {
        m.addBalance(paise);
        if (m.balance() >= m.selectedProduct().pricePaise()) {
            m.setState(new DispensingState());
            m.dispense();
        }
    }
    public void cancel(VendingMachine m) { m.refund(); m.reset(); }
}

class DispensingState implements VendingState {}          // all actions rejected while dispensing (defaults)

class SoldOutState implements VendingState {
    public void insertMoney(VendingMachine m, int paise) { System.out.println("Sold out, returning " + paise); }
}

public class VendingMachine {
    private final Inventory inventory = new Inventory();
    private VendingState state = new SoldOutState();
    private String selected; private int balance;

    // public API — synchronized: one customer at a time on a physical machine
    public synchronized void select(String code) { state.select(this, code); }
    public synchronized void insertMoney(int paise) { state.insertMoney(this, paise); }
    public synchronized void cancel() { state.cancel(this); }
    public synchronized void restock(String code, Product p, int qty) {
        inventory.add(code, p, qty);
        if (state instanceof SoldOutState) state = new IdleState();
    }

    // used by states
    void dispense() {
        Slot s = inventory.get(selected);
        s.qty--;
        int change = balance - s.product.pricePaise();
        System.out.println("Dispensed " + s.product.name() + (change > 0 ? ", change " + change : ""));
        balance = 0; selected = null;
        state = inventory.isEmpty() ? new SoldOutState() : new IdleState();
    }
    void refund() { if (balance > 0) System.out.println("Refund " + balance); }
    void reset() { balance = 0; selected = null; state = inventory.isEmpty() ? new SoldOutState() : new IdleState(); }
    void setState(VendingState s) { state = s; }
    void setSelected(String c) { selected = c; }
    void addBalance(int p) { balance += p; }
    int balance() { return balance; }
    Product selectedProduct() { return inventory.get(selected).product; }
    Inventory inventory() { return inventory; }
}

// VendingMachine vm = new VendingMachine();
// vm.restock("A1", new Product("Chips", 2000), 5);
// vm.select("A1"); vm.insertMoney(1000); vm.insertMoney(2000);   // Dispensed Chips, change 1000
```

**5. Notes**
- **Why State pattern:** without it, every method has `switch (state) { ... }` — adding a state means editing every method. With it, each state class owns its rules; illegal actions fail by default.
- States are stateless here, so they could be **singletons** (`static final IdleState IDLE = new IdleState()`) to avoid allocations.
- Change-making with limited coins = a **coin inventory** + greedy (or DP if coin set is non-canonical). Mention as an extension.
- Extensions: card/UPI payment (`PaymentStrategy`), admin/maintenance state, low-stock alerts (Observer).

---

## 25. Most Asked OOP & LLD Interview Questions (Top 40)

⭐ = top 15 most frequently asked. Say the answer in your own words; keep it 30–60 seconds.

**⭐ 1. What are the 4 pillars of OOP?**
Encapsulation (bundle data + methods, hide data), Abstraction (show what, hide how), Inheritance (reuse via is-a), Polymorphism (one interface, many forms). Give one example each: `private balance` + `withdraw()`; `PaymentGateway` interface; `Car extends Vehicle`; `List<Notifier>` calling `send()`.

**⭐ 2. Difference between abstraction and encapsulation?**
Abstraction hides **complexity** at the design level — interfaces expose only what an object does. Encapsulation hides **data** at the implementation level — private fields with controlled access. Car dashboard = abstraction; locked bonnet = encapsulation.

**⭐ 3. Abstract class vs interface? When to use which?**
Abstract class: can have state, constructors, concrete methods; single inheritance; for close "is-a" relatives sharing code. Interface: contract, no instance state, multiple implementation, `default`/`static` methods since Java 8; for "can-do" capabilities. Default to interfaces; use an abstract class to share state/code (often both: `List` + `AbstractList`).

**⭐ 4. Overloading vs overriding?**
Overloading: same name, different parameters, same class, resolved at **compile time**. Overriding: child redefines a parent method with the same signature, resolved at **runtime** via dynamic dispatch. Overriding can't reduce visibility or throw broader checked exceptions; return type can be covariant.

**⭐ 5. What is runtime polymorphism / dynamic dispatch? How does a vtable work?**
The method that runs depends on the actual object, not the reference type (`Animal a = new Dog(); a.sound()` → Dog's). In C++, each class with virtual functions has a vtable (array of function pointers); each object has a hidden vptr to it. A virtual call follows vptr → slot → function. Java does the same with method tables; all non-static, non-final, non-private methods are virtual.

**⭐ 6. What is the diamond problem? How do Java and C++ handle it?**
D inherits from B and C, which both inherit from A → two copies of A and ambiguous methods. Java bans multiple class inheritance; for conflicting interface default methods, the class must override and can call `X.super.m()`. C++ allows it and uses **virtual inheritance** (`class B : virtual public A`) so only one A exists.

**⭐ 7. Explain SOLID with examples.**
S: one reason to change (calculator ≠ mailer). O: add a new `DiscountPolicy` class instead of editing an if-else. L: subclasses must be substitutable (Square/Rectangle trap). I: small interfaces (`Workable`, `Eatable`). D: depend on `PaymentGateway` interface; Spring injects Stripe/Solana Pay.

**⭐ 8. `equals()` and `hashCode()` contract? What breaks if you override only `equals`?**
Equal objects must have equal hash codes. If only `equals` is overridden, two "equal" objects get different identity hashes → land in different HashMap buckets → `map.get()` returns null and `HashSet` stores duplicates. Always override both (or use a `record`).

**⭐ 9. How do you make a class immutable? Why is String immutable?**
`final` class, `private final` fields, no setters, defensive copies in constructor and getters, return new objects for "changes". String is immutable for the string pool (sharing), security (paths/URLs can't change after checks), thread safety, and cached hashCode.

**⭐ 10. Implement a thread-safe Singleton. Why `volatile` in double-checked locking?**
Options: eager static field, holder idiom, enum (safest — handles reflection and serialization), or DCL. In DCL, `volatile` prevents reordering of "allocate → construct → assign" so no thread sees a half-constructed object.

**⭐ 11. Composition vs inheritance — which do you prefer and why?**
Prefer composition (has-a): looser coupling, can change behaviour at runtime, only exposes what you choose, easier to test. Inheritance is fine for a true, stable is-a. Example: `Stack extends ArrayList` leaks `add(index)`; a Stack that **has** a Deque is correct.

**⭐ 12. Explain the Strategy / Observer / Factory pattern with a real example.**
Strategy: swap algorithms (backoff policies, payment methods). Observer: publisher notifies subscribers (SNS → SQS fan-out, Kafka consumer groups, WebSocket hub broadcast). Factory: create the right handler for an event type from a registry. Always say what problem it solved.

**⭐ 13. Design a Parking Lot (or any LLD). What's your approach?**
Clarify requirements → entities → relationships/class diagram → APIs → patterns (Strategy for pricing/spot assignment) → concurrency (atomic spot assignment with CAS or DB conditional update) → extensibility (new vehicle type = new enum/class). Then code core classes.

**⭐ 14. Design an LRU cache in O(1).**
HashMap (key → node) + doubly linked list (recency order) with sentinel head/tail. `get`: lookup + move to front. `put`: update/move or insert at front; if over capacity, remove tail's previous and delete its key from the map (store key in node). Thread safety: synchronize both (get mutates order) or shard.

**⭐ 15. How do you prevent double booking of a seat (BookMyShow)?**
Make check-and-lock atomic: per-show lock in memory, or DB conditional update (`... WHERE status='AVAILABLE'`, check rows updated), optimistic `@Version`, or `SELECT FOR UPDATE`, plus a unique constraint as a safety net. Temporary lock with expiry during payment; never hold a DB/Java lock across the payment call. Same idea as my LocalArena concurrent-join test: a match must never overfill.

**16. Types of constructors? What is constructor chaining?**
Default, parameterized, copy (hand-written in Java; auto-generated in C++). Chaining: `this(...)` calls another constructor of the same class, `super(...)` calls the parent's; must be the first statement. Parent constructors always run before the child's.

**17. What does `static` mean? Can static methods be overridden?**
Belongs to the class, shared across all objects (counters, utility methods, constants). Static methods can't be overridden — redefining in a child is **method hiding**, resolved by reference type at compile time. Static methods can't use `this`.

**18. `final` vs `finally` vs `finalize`?**
`final`: constant variable / non-overridable method / non-extendable class. `finally`: block that always runs after try/catch (cleanup). `finalize()`: deprecated GC hook — use try-with-resources/`Cleaner` instead.

**19. Checked vs unchecked exceptions? When to create custom exceptions?**
Checked (`IOException`) must be caught or declared — recoverable external failures. Unchecked (`RuntimeException`) — programming errors or invalid input. Create custom exceptions for meaningful business errors (`InsufficientBalanceException`), usually unchecked, and map them to HTTP codes via `@ControllerAdvice`.

**20. What is try-with-resources?**
`try (Resource r = ...) { }` automatically calls `close()` on any `AutoCloseable` at the end, even on exceptions, in reverse order of opening. Suppressed exceptions from `close()` are kept via `getSuppressed()`. Replaces messy `finally` blocks.

**21. Shallow copy vs deep copy?**
Shallow copies the outer object but shares inner objects (changes leak between copies). Deep copies everything recursively. In Java prefer copy constructors over `clone()` (which is shallow and awkward). In C++, a class owning raw memory needs a deep-copy copy constructor (Rule of Three).

**22. What is the Rule of Three / Five / Zero in C++?**
If you define a destructor, copy constructor, or copy assignment, you likely need all three (Three). C++11 adds move constructor and move assignment (Five). Best: use RAII members (`std::vector`, `std::string`, smart pointers) so you need none (Zero).

**23. Why should a base class destructor be virtual in C++?**
Deleting a derived object through a base pointer calls only the base destructor if it isn't virtual → derived resources leak (undefined behaviour). Rule: any class with virtual functions gets a virtual destructor.

**24. `unique_ptr` vs `shared_ptr` vs `weak_ptr`?**
`unique_ptr`: single owner, move-only, zero overhead — default choice. `shared_ptr`: shared ownership via reference count; last owner deletes. `weak_ptr`: non-owning reference to a `shared_ptr` object, used to break cycles and for caches.

**25. Can a constructor be private? Virtual? Can we call a virtual function in a constructor?**
Private: yes (Singleton, factories). Virtual: no in C++ (vptr not ready) — use virtual `clone()`. Virtual call in constructor: C++ calls the current class's version; Java calls the child override, which may see uninitialized fields — avoid in both.

**26. Association vs aggregation vs composition?**
Association: objects know each other, independent (Doctor–Patient). Aggregation: weak has-a, part can exist alone (Team–Player), hollow diamond. Composition: strong has-a, part dies with whole (House–Room, Order–OrderItem), filled diamond.

**27. Comparable vs Comparator?**
Comparable: natural order inside the class (`compareTo`), one ordering. Comparator: external, many orderings (`Comparator.comparing(...).thenComparing(...)`), passed to `sort`, `TreeMap`, `PriorityQueue`. Use `Integer.compare`, not subtraction (overflow).

**28. How does HashMap work internally?**
Array of buckets; index = hash(key) & (n−1). Collisions chain in a linked list, converted to a red-black tree when a bucket has ≥ 8 entries (Java 8). Resizes (doubles) when size > capacity × 0.75. `get`/`put` O(1) average, O(log n) worst with treeified buckets.

**29. HashMap vs ConcurrentHashMap vs Hashtable / synchronizedMap?**
HashMap: not thread-safe, allows one null key. Hashtable/synchronizedMap: one lock for the whole map — slow. ConcurrentHashMap: fine-grained (per-bin) locking + CAS, high concurrency, atomic `computeIfAbsent`/`merge`, no nulls, iterators don't throw ConcurrentModificationException.

**30. Stack vs heap memory in Java? What is the String pool?**
Stack: per-thread, method frames, locals and references, auto-freed. Heap: shared, all objects, garbage-collected. String pool: a special area on the heap where literals are interned; `"a" == "a"` is true but `new String("a")` creates a new object — always compare with `equals`.

**31. How does garbage collection work (briefly)?**
GC frees objects unreachable from GC roots (stack refs, static fields, threads). Generational: new objects in Young Gen (frequent cheap minor GCs), survivors promoted to Old Gen. Modern default is G1; ZGC for low latency. Leaks in Java come from references kept alive (growing static caches, unremoved listeners).

**32. `synchronized` vs `volatile` vs Atomic classes?**
`synchronized`: mutual exclusion + visibility. `volatile`: visibility and ordering only — not atomic for `count++`. `AtomicInteger`: lock-free atomic operations with CAS. Use atomic for single counters, locks for multi-variable invariants.

**33. What is a deadlock and how do you prevent it?**
Two+ threads each hold a lock and wait for the other's. Prevent by acquiring locks in a fixed global order (e.g., sorted seat IDs), using `tryLock` with timeout, keeping lock scope small, and avoiding calling external code while holding a lock.

**34. Builder vs Factory? When would you use Builder?**
Factory decides **which** class to create (one call). Builder constructs **one complex object step by step** — many optional params, readable chained calls, validation in `build()`, immutable result. Example: Solana `new Transaction().add(ix)...`, AWS SDK request builders, Lombok `@Builder`.

**35. Adapter vs Decorator vs Proxy vs Facade?**
All wrap. Adapter **changes** the interface to fit (STT/TTS vendor wrappers). Decorator keeps the interface and **adds behaviour** (logging/caching wrappers, middleware). Proxy keeps the interface and **controls access** (lazy load, auth, Spring `@Transactional`). Facade gives a **new simple interface** over many classes (CheckoutFacade).

**36. When would you use the State pattern? Strategy vs State?**
When behaviour changes with internal state and transitions are well-defined (vending machine, order lifecycle, elevator). Strategy: client picks an algorithm, strategies don't know each other. State: the object switches states itself, and states know their transitions.

**37. What is Dependency Injection? DIP vs DI vs IoC?**
DI: an object receives its dependencies (constructor injection preferred) instead of creating them. DIP: principle — depend on abstractions. IoC: the framework (Spring container) creates and wires objects. Benefits: loose coupling, easy mocking in tests, swappable implementations.

**38. Explain Circuit Breaker and Retry with backoff. How do they work together?**
Retry handles short transient failures with exponential backoff + jitter, only for idempotent calls. Circuit breaker stops calling a dependency that keeps failing (CLOSED → OPEN → HALF_OPEN), failing fast with a fallback. Together: retries inside, breaker outside so retries don't hammer a dead service. Used in LocalArena (ADR-0008).

**39. What is the Outbox pattern and why is it needed?**
Solves the dual-write problem (DB commit succeeds but event publish fails). Write the business row and an outbox event row in the **same transaction**; a relay publishes unsent outbox rows and marks them sent. Delivery is at-least-once, so consumers must be idempotent. Used in LocalArena for reliable events.

**40. Design a rate limiter. Token bucket vs sliding window?**
`RateLimiter.allow(clientId)` interface with pluggable algorithms. Token bucket: capacity + refill rate, lazy refill from elapsed time, allows bursts, O(1) memory. Sliding window log: exact, stores timestamps. Per-client locks + `ConcurrentHashMap`; distributed version in Redis with a Lua script for atomicity; respond 429 with `Retry-After`.

---

## 26. Last-minute revision

**OOP core**
- Class = blueprint, object = house. Constructor sets up; parent runs first. `this(...)`/`super(...)` first line.
- `static` = per class (hiding, not overriding). `final` = no reassign / no override / no extend.
- Access: private < default (package) < protected < public. Can't narrow visibility when overriding.
- **A PIE:** Abstraction (what, not how), Polymorphism (one interface, many forms), Inheritance (is-a reuse), Encapsulation (hide data + guard invariants).
- Abstraction = hide **complexity** (design). Encapsulation = hide **data** (implementation).
- Overloading = compile time, params differ. Overriding = runtime, same signature, covariant return, no narrower access, no broader checked exceptions.
- vtable: one per class (function pointers); vptr: one per object. C++ needs `virtual`; Java is virtual by default.
- C++: virtual destructor in polymorphic bases; slicing when copying by value; polymorphism only via pointer/reference.
- Diamond: Java bans multiple class inheritance, forces override for default-method conflicts (`X.super.m()`); C++ uses `virtual` inheritance.
- Abstract class = half-built house (state + code, single). Interface = contract (multiple, `default`/`static`/`private` methods).
- Association → Aggregation (hollow ◇, independent) → Composition (filled ◆, dies with owner). **Prefer composition.**

**Java specifics**
- `equals` ⇒ same `hashCode`. Override only `equals` → HashMap get() null, Set duplicates. Mutable keys get "lost".
- Immutable: final class, private final fields, no setters, defensive copies in and out.
- Shallow shares inner objects; deep copies them. Prefer copy constructors over `clone()`.
- C++ Rule of 3/5/0; copy ctor takes `const T&`; `unique_ptr` default, `shared_ptr` shared, `weak_ptr` breaks cycles.
- Checked (must handle) vs unchecked (RuntimeException). try-with-resources for `AutoCloseable`. Never swallow; keep cause.
- Generics: type safety; erased at runtime; PECS (`? extends` read, `? super` write).
- Stack (per-thread frames, refs) vs heap (objects, GC). String pool → compare with `equals`.
- Comparable = natural order inside; Comparator = external, many orders. Use `Integer.compare`.
- ArrayList O(1) get; HashMap O(1) avg (treeify at 8, resize at 0.75); TreeMap O(log n) sorted; PriorityQueue O(log n) offer/poll; ArrayDeque for stack/queue.
- `synchronized` = lock + visibility; `volatile` = visibility only; Atomic = CAS. Check-then-act is a race — use `computeIfAbsent`/`putIfAbsent`/conditional updates.
- DCL singleton needs `volatile`. Best: enum or holder idiom.

**SOLID + principles**
- S one reason to change · O add classes, don't edit · L child substitutable (no Square/Rectangle, no Penguin.fly) · I small interfaces · D depend on interfaces, inject implementations.
- DRY, KISS, YAGNI. Law of Demeter. Program to an interface.

**Patterns in one line (+ your example)**
| Pattern | One line | Yours |
|---|---|---|
| Singleton | one instance (enum/holder/DCL) | DB pool / Prisma client |
| Factory | pick class by input | handler per event type |
| Abstract Factory | family of matching objects | AWS vs GCP clients |
| Builder | step-by-step, immutable result | Solana `Transaction().add(...)` |
| Prototype | clone an existing object | virtual `clone()` in C++ |
| Adapter | convert interface | STT/TTS vendor wrappers |
| Decorator | wrap, same interface, add behaviour | Express/Spring middleware |
| Facade | simple front for a subsystem | CheckoutFacade |
| Proxy | wrap, same interface, control access | Spring `@Transactional` proxies |
| Composite | tree; leaf and group treated the same | folders/files |
| Strategy | swap algorithm | backoff policies, payment methods |
| Observer | notify subscribers | SNS→SQS, Kafka consumers, WebSocket hub |
| Command | request as object (undo/queue) | job queue tasks |
| State | behaviour per state, object switches | vending machine, order lifecycle |
| Template Method | fixed skeleton, subclass fills steps | JdbcTemplate, importers |
| Chain of Responsibility | pass along handlers | middleware/filters, logger levels |
| Circuit Breaker | stop calling failing dependency | LocalArena (ADR-0008) |
| Retry + backoff + jitter | retry transient, idempotent calls | backoff Strategy |
| Outbox | DB row + event in one transaction | LocalArena |
| Repository | collection-like data access | Spring Data JPA |
| DI | receive dependencies via constructor | Spring |

**LLD 7 steps:** Requirements → Entities → Relationships/diagram → APIs/flows → Patterns → **Concurrency** → Extensibility. Then code.

**LLD designs — one-line core of each**
- **Parking Lot:** Lot ◆ Floors ◆ Spots; `PricingStrategy` + `SpotAssignmentStrategy`; spot `compareAndSet(null, v)` so only one gate wins.
- **LRU:** HashMap + DLL with sentinels; key stored in node; synchronize get & put.
- **Rate Limiter:** token bucket with lazy refill + per-client lock; sliding window log with deque; Redis + Lua for distributed; 429 + `Retry-After`.
- **Elevator:** controller + `DispatchStrategy`; per-elevator up/down `TreeSet`s (LOOK); State for elevator modes.
- **BookMyShow:** status on `ShowSeat`; atomic all-or-nothing lock with expiry; never lock during payment; DB conditional update / `@Version` / unique constraint; = LocalArena "never overfill a match".
- **Splitwise:** `SplitStrategy` (Equal/Exact/Percent), money in paise, net balances sum to 0, greedy max-creditor vs max-debtor simplification (≤ n−1 txns).
- **Notification:** channels as Strategy, events via Observer (→ SNS/SQS/Kafka at scale), async + exponential backoff + DLQ, idempotent IDs, Outbox for the event.
- **Logger:** holder-idiom Singleton, CoR handlers per level, synchronized sinks, async appender in real life.
- **Vending Machine:** State pattern (Idle → HasSelection → Dispensing → Idle / SoldOut); illegal actions throw by default.

**Common traps to avoid in the round**
- Coding before clarifying requirements.
- God classes (`ParkingLotManager` that does everything).
- `double` for money.
- Check-then-act without atomicity.
- Holding a lock during a slow external call (payment, network).
- Inheritance where composition fits; subclasses that add nothing (use an enum).
- Forgetting `@Override`, `equals`/`hashCode` together, and `volatile` in DCL.
