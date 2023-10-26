---
title: "Common design patterns in Golang - Part 1"
seoTitle: "Golang Design Patterns: Part 1"
seoDescription: "Explore Golang design patterns like Singleton, Factory, and Builder for flexible, reusable objects. Learn Go implementation and benefits."
datePublished: Thu Jul 20 2023 09:37:57 GMT+0000 (Coordinated Universal Time)
cuid: clkaylsdi000d09jsa8s9hy7l
slug: common-design-patterns-in-golang-part-1
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/jVZ_BKzDOJg/upload/73c552bb6d1339ab9960d5d1c4710a9a.jpeg
tags: design-patterns, go, golang, concurrency, creational-patterns

---

Design patterns are general reusable solutions to common software design problems that provide a blueprint or a best-practice approach for structuring code and solving specific problems.

There are 3 common categories of design patterns:

* **Creational Patterns**: These patterns deal with object creation mechanisms, providing ways to create objects in a flexible and decoupled manner. Examples include Singleton, Factory, Builder, and Prototype patterns.
    
* **Structural Patterns**: These patterns focus on the structure of classes and objects, defining how they are composed to form larger structures. Examples include Adapter, Decorator, Facade, and Composite patterns.
    
* **Behavioral Patterns**: These patterns focus on the interaction and communication between objects, defining how they collaborate and operate together. Examples include Observer, Strategy, Template, and Command patterns.
    

And a not-so-common category of design pattern (But apply a lot in very niche problems):

* **Concurrency Patterns**: These patterns are specifically designed to address concurrent and parallel programming challenges, leveraging language-specific concurrency features, such as threads or processes. Examples include Mutex, Semaphore, and Barrier patterns
    

In the first part of this subject. We’ll introduce the basics of design patterns with the definition and implementation of Interfaces in Golang and the first category of design patterns: The Creational Patterns

## Golang Interface

Before we start diving into the big ocean of design patterns we have to go through the base of design patterns: the interface.

### **Then, what is an interface in Go?**

Interfaces play a vital role in modern programming languages by facilitating loose coupling and enabling flexible code reuse. Go, a statically typed language developed by Google, incorporates interfaces as a core feature to promote clean and efficient code design.

In Go, an interface is a collection of method signatures. It defines a contract that specifies the methods an implementing type must implement. Unlike traditional object-oriented languages, Go interfaces are implicit; a type automatically satisfies an interface if it implements all the methods defined by the interface.

Interfaces in Go promote polymorphism and code decoupling. By programming to interfaces rather than concrete types, developers can write flexible and reusable code. Interfaces facilitate loose coupling between components, enabling independent development and testing.

To illustrate the practical application of interfaces in Go, we present an example of a file system interface. Consider a scenario where we need to perform operations on different types of file systems, such as local disk, remote storage, or cloud-based file systems. By defining an interface for file system operations, we can write generic code that works seamlessly with any file system implementation.

```go
// File represent the file data
type File struct {
    // File representation
}

type FileSystem interface {
    OpenFile(name string) (File, error)
}

// LocalFileSystem implement interface FileSystem
type LocalFileSystem struct {
}

func (fs LocalFileSystem) OpenFile(name string) (File, error) {
    // Open file implementation for local file system
}

// S3FileSystem implement interface FileSystem
type S3FileSystem struct {
}

func (fs S3FileSystem) OpenFile(name string) (File, error) {
    // Open file implementation for S3 file system
}

func ProcessFilesystem(fs File) {
	data := fs.ReadAll()
	fmt.Println(data)
}

func main() {
    localFS := LocalFileSystem{}
    ProcessFilesystem(localFS)

    s3FS := S3FileSystem{}
    ProcessFilesystem(s3FS)

    // Use other file system implementations
}
```

In the example above, we define a `FileSystem` interface representing common file system operations. We then provide two implementations: `LocalFileSystem` and `S3FileSystem`, each with its specific details. The `ProcessFilesystem` function takes any type satisfying the `FileSystem` interface and performs operations on it. This design enables us to easily switch between different file system implementations without modifying the client code.

### **But then, what is the role of interfaces in design patterns?**

Interfaces in Go facilitate abstraction by defining a contract that specifies the methods a type must implement. This allows for loose coupling between components, enabling independent development and testing. Developers can write flexible and reusable code by programming to interfaces rather than concrete types.

Interfaces provide a means to achieve code modularity by separating abstract behavior from implementation details. Design patterns often rely on interfaces to define common functionality and enable interchangeable components. This promotes code reusability and simplifies maintenance.

## The Creational Patterns

After going through the base of the design patterns in Go. We can jump right into the first category of design patterns: The Creational Patterns

### What are Creational Patterns?

Creational design patterns address the challenges associated with creating objects in a flexible, reusable, and extensible manner. Go, a statically typed language known for its simplicity and concurrency support, provides a range of idiomatic features that facilitate the implementation of creational design patterns.

If you are still confused, don’t worry, let’s dive into some notable patterns in this category and its implementations to have a clearer view.

### 1\. **Singleton Pattern**

The Singleton pattern ensures that only one instance of a class is created throughout the application's lifecycle, providing global access to that instance.

Example implementation:

```go
//single.go
package main

import (
    "fmt"
    "sync"
)

var lock = &sync.Mutex{}

type single struct {
}

var singleInstance *single

func getInstance() *single {
    if singleInstance == nil {
        lock.Lock()
        defer lock.Unlock()
        if singleInstance == nil {
            fmt.Println("Creating single instance now.")
            singleInstance = &single{}
        } else {
            fmt.Println("Single instance already created.")
        }
    } else {
        fmt.Println("Single instance already created.")
    }

    return singleInstance
}
```

```go
//main.go
package main

import (
    "fmt"
)

func main() {

    for i := 0; i < 30; i++ {
        go getInstance()
    }

    // Scanln is similar to Scan, but stops scanning at a newline and
    // after the final item there must be a newline or EOF.
    fmt.Scanln()
}
```

The example code presented showcases the Singleton pattern implemented in Go through a struct named "single" and a corresponding "getInstance()" function. The purpose of this pattern is to control the instantiation of the struct, ensuring that only one instance is created and accessed throughout the execution of the program.

The struct definition is intentionally minimalistic, lacking any specific attributes or methods. The focus here is on the Singleton pattern itself, rather than the specific functionality of the struct. However, in practical scenarios, the struct would typically encapsulate relevant data and behavior.

To guarantee the creation of a single instance, the code employs a combination of lazy initialization and synchronization. The "singleInstance" variable, initialized as nil, holds the reference to the sole instance of the "single" struct. The "lock" variable, a Mutex from the sync package, is used to ensure thread-safe access during the instance creation process.

The "getInstance()" function acts as the entry point for acquiring the Singleton instance. Initially, it checks if the "singleInstance" variable is nil. If so, it acquires the lock to prevent simultaneous instantiation by multiple threads. This approach ensures that only one goroutine proceeds with the creation of the instance, while others wait for the lock to be released.

Inside the critical section protected by the lock, a double-checking mechanism is employed to prevent redundant object creation. While holding the lock, the function verifies once more if the instance has been instantiated to avoid unnecessary creation attempts. If the instance is still nil, it creates a new instance of the "single" struct. Finally, the lock is released to allow other goroutines to access the critical section.

The output:

```go
Creating single instance now.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
...
```

### 2\. **Factory Pattern**

In the Factory Method pattern, the goal is to delegate the responsibility of object creation to subclasses or concrete implementations, while still adhering to a common interface or base class. This allows for the flexible creation of objects without tightly coupling the client code to specific implementations.

Implementation example:

```go
package main

import (
	"fmt"
)

type PaymentMethod interface {
	Pay(amount float64)
}

// CreditCard pay via credit card
type CreditCard struct{}
func (c *CreditCard) Pay(amount float64) {
	fmt.Printf("Paying %.2f using credit card.\\n", amount)
}

// DebitCard pay via debit card
type DebitCard struct{}
func (d *DebitCard) Pay(amount float64) {
	fmt.Printf("Paying %.2f using debit card.\\n", amount)
}

// DebitCard pay via cash
type Cash struct{}
func (c *Cash) Pay(amount float64) {
	fmt.Printf("Paying %.2f using cash.\\n", amount)
}

type PaymentMethodFactory struct{}

func (pmf *PaymentMethodFactory) CreatePaymentMethod(paymentType string) (PaymentMethod, error) {
	switch paymentType {
	case "credit":
		return &CreditCard{}, nil
	case "debit":
		return &DebitCard{}, nil
	case "cash":
		return &Cash{}, nil
	default:
		return nil, fmt.Errorf("Invalid payment method type: %s", paymentType)
	}
}

func main() {
	paymentMethodFactory := &PaymentMethodFactory{}

	paymentMethod, err := paymentMethodFactory.CreatePaymentMethod("credit")
	if err != nil {
		fmt.Println(err)
		return
	}
	paymentMethod.Pay(100.0)

	paymentMethod, err = paymentMethodFactory.CreatePaymentMethod("debit")
	if err != nil {
		fmt.Println(err)
		return
	}
	paymentMethod.Pay(50.0)

	paymentMethod, err = paymentMethodFactory.CreatePaymentMethod("cash")
	if err != nil {
		fmt.Println(err)
		return
	}
	paymentMethod.Pay(20.0)
}
```

In this example, we have defined a `PaymentMethod` interface that declares the `Pay()` method. Three concrete implementations of this interface are provided: `CreditCard`, `DebitCard`, and `Cash`.

The `PaymentMethodFactory` struct represents the factory responsible for creating instances of payment methods based on the requested type. It has a `CreatePaymentMethod()` method that takes a `paymentType` string as a parameter and returns an instance of the corresponding payment method.

In the `main()` function, we create an instance of the `PaymentMethodFactory` and use it to create different payment methods based on the desired type. We then invoke the `Pay()` method on each payment method with a specific amount to simulate a payment.

By using the Factory Pattern, the client code (in this case, the `main()` function) doesn't need to worry about the specifics of creating the payment method objects. Instead, it relies on the factory to provide the appropriate implementation based on the requested type.

This design allows for easy extensibility, as new payment methods can be added by implementing the `PaymentMethod` interface and modifying the `CreatePaymentMethod()` method in the factory accordingly.

The Factory Pattern enables loose coupling between the client code and the concrete payment method implementations, promoting flexibility and maintainability in the codebase.

### 3\. Builder Pattern

In the Builder pattern, the goal is to separate the construction of complex objects from their representation. It allows the same construction process to create different representations of an object. This pattern is particularly useful when you need to create objects step by step or when you want to isolate the construction logic from the client code. Following is the example of this implementation:

```go
// server/server.go
package server

struct Server {
	host string
  port int
  timeout time.Duration
  maxConn int
}

func (s *Server) Start() {
	//TODO: implement server start
}
```

```go
// server/server.go
package server

struct Server {
	host string
  port int
  timeout time.Duration
  maxConn int
}

func (s *Server) Start() {
	//TODO: implement server start
}
```

```go
// main.go
package main

import (
  "log"
  "time"
  "github.com/server"
)

func main() {
  builder := server.NewServerBuilder()
	srv := builder.
					SetHost("localhost").
					SetPort("8080").
					SetTimeout(time.Minute).
					SetMaxConn(120).
					Build()

	srv.Start()	
}
```

In the `server` package, there is a `Server` struct defined with fields representing the host, port, timeout, and maximum connections. The `Server` struct also has a method `Start()` which is responsible for starting the server.

The `builder.go` file contains a `ServerBuilder` struct. This builder is used to create and configure a `Server` object. It follows a fluent API design, allowing method chaining for setting different properties of the server.

In the `main` package, the `main()` function demonstrates the usage of the builder pattern to create a server instance. It first imports the necessary packages (`log`, `time`, and [`github.com/server`](http://github.com/server)).

Inside `main()`, a `ServerBuilder` is created using `server.NewServerBuilder()`. Then, a server instance (`srv`) is constructed by chaining method calls on the builder to set the host, port, timeout, and maximum connection properties. Finally, `srv.Start()` is called to start the server.

## Conclusion

Creational design patterns provide effective solutions for object creation and initialization. This article explored three widely used creational design patterns: Singleton, Factory Method, and Builder, in the context of the Go programming language. By analyzing implementation examples, we highlighted the practical usage and benefits of these patterns. Understanding and applying these creational design patterns in Go can contribute to the development of well-designed, modular, and maintainable software systems. Further research can explore additional creational design patterns and their implementation in Go.

This is the first part of multipart articles about design patterns in Go. Next time we move to the next category of design patterns in Golang: **Structural Patterns.** Stay tuned and see you next time.

### ***Contributing***

*At Dwarves, we encourage our people to read, write, share what we learn with others, and* [***contributing to the Brainery***](https://brain.d.foundation/CONTRIBUTING) *is an important part of our learning culture. For visitors, you are welcome to read them, contribute to them, and suggest additions. We maintain a monthly pool of $1500 to reward contributors who support our journey of lifelong growth in knowledge and network.*

### *Love what we are doing?*

* *Check out our* [***products***](https://superbits.co/)
    
* *Hire us to* [***build your software***](https://d.foundation/)
    
* *Join us,* [***we are also hiring***](https://github.com/dwarvesf/WeAreHiring)
    
* *Visit our* [***Discord Learning Site***](https://discord.gg/dzNBpNTVEZ)
    
* *Visit our* [*GitHub*](https://github.com/dwarvesf)