---
layout: post
title: Dependency Injection
date:   2018-04-20 01:00:00 +1100
categories: OO Pattern development
---

## Creational Patterns
### Dependency Injection (DI)

#### Overview

DI is a form of _Inversion of control_. The basic form of DI is where a client delegates the responsibility of providing it's dependencies to external code (via injection). An injected dependency is generally referred to as a *service* in relation to the client.  The client needs to know only the service interface to make use of a dependency, and is not concered with any specific implementation or construction of dependencies.

Passing a service into a client, rather than the client having to find the appropriate service to use, is the fundamental requirement of DI. This means that the client must not make use of any static methods etc. to generate values, and the values are acquired from outside using the service(s) that are injected.

There are 3 common techniques for a client to accept injection of a depenendency;

1. Setter
2. Interface
3. Constructor

Setter and Constructor injection differ not in implementation, but mainly by when they can be called or used. Interface Injection differs in that it can enable the dependency to control it's own injection.

#### Problems this pattern solves

* Decouple objects from one another. This aims to prevent a client object from requiring code changes because of changes to any of it's dependencies
* Keeps a class independent of how objects that it requires are created
* Allows multiple dependency implementations to be used interchangably
* Simplifies client code and makes it easier to maintain

#### Problems this pattern may introduce

* Introduces additional configuration requirements, which may make simple requirements laborious
* Additional overhead for Developers understanding and maintaining application code may be introduced, where implementation details are separated from client code
* Additional configuration makes development effort greater, where injection is used instead of implementing services in a client directly
* Complex dependency links and dependency graphs can make a system harder to conceptualise/understand, which also can make configuring and management complex
* DI Frameworks may not easily be changed where they are relied upon for an application. Ironically enough, this may create a tightly coupled dependency on a DI framework (a problem DI strives to prevent)

#### Typical Uses

* A seperate dependency may be injected for testing purposes, such as a mock service. In Production a concrete implementation is injected.
* A different DBMS might be swapped out for an Application, eg. PostgreSQL to MS SQL Server. The client code remains the same, and unaffected.
* A single client class my be created for authorization logic, and use multiple providers depending on the context, eg. OATH or SAML

#### Frameworks that use this pattern

* Spring/Spring Boot (Java)
* Guice (Java)
* Managed Extensibility Framework (.NET)


#### Examples

#### No DI
```
// An example without dependency injection
public class Client {
    // Internal reference to the service used by this client
    private ExampleService service;

    // Constructor
    Client() {
        // Specify a specific implementation in the constructor instead of using dependency injection
        service = new ExampleService();
    }

    // Method within this client that uses the services
    public String greet() {
        return "Hello " + service.getName();
    }
}
```

#### Constructor Injection
```
// Constructor
Client(Service service) {
    // Save the reference to the passed-in service inside this client
    this.service = service;
}
```

#### Setter Injection
```
// Setter method
public void setService(Service service) {
    // Save the reference to the passed-in service inside this client
    this.service = service;
}
```

#### Interface Injection
```
// Service setter interface.
public interface ServiceSetter {
    public void setService(Service service);
}

// Client class
public class Client implements ServiceSetter {
    // Internal reference to the service used by this client.
    private Service service;

    // Set the service that this client is to use.
    @Override
    public void setService(Service service) {
        this.service = service;
    }
}
```
