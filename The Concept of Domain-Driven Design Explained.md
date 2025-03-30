# The Concept of Domain-Driven Design Explained

Domain-driven design

Using [microservices](https://aws.amazon.com/microservices/) means **creating applications from loosely coupling services**. The application consists of several small services, each representing a separate business goal. [They can be developed and easily maintained individually](https://microtica.com/the-concept-of-domain-driven-design-explained/?utm_source=medium&utm_medium=medium&utm_campaign=medium_ddd), after what they are joint in a complex application.

[Microservices](https://microtica.com/everything-about-microservices/?utm_source=medium&utm_medium=medium&utm_campaign=medium_ddd) is an architecture design model with a **specific bounded context, configuration, and dependencies.** These result from the architectural principles of the domain-driven design and DevOps. **Domain-driven design is the idea of solving problems of the organization through code.**

The business goal is important to the business users, with a **clear interface and functions**. This way, the microservice can run independently from other microservices. Moreover, the team can also work on it independently, which is, in fact, the point of the microservice architecture.

Many developers claim microservices have made them more efficient. This is due to the ability to work in small teams. This allows them to develop different small parts that will later be merged as a large app.

They spend less time coordinating with other developers and more time on developing the actual code. Eventually, this creates more value for the end-user.

# The Complexity Challenge

Complexity is a relative term. What’s complex for one person is simple for another. However, **complexity is the problem that domain-driven design should solve**. In this context, complexity means interconnectedness, many different data sources, different business goals, etc.

The domain-driven approach is here to solve the complexity of software development. On the other hand, you can use emergent design when the challenge is simple. However, when your application is complex, the complexity will only grow, and so will your problems.

**Domain-driven design bases on the business domain.** Modern business environments are very complex and wrong moves can lead to fatal outcomes. Domain-driven design solves complex domain models, connecting to the core business concepts.

Eric Evans, introduced the concept in 2004, in his book *Domain-Driven Design: Tackling Complexity in the Heart of Software*. According to the book, it focuses on three principles:

- The primary focus of the project is the core **domain** and **domain logic.**
- Complex designs are based on **models of the domain.**
- Collaboration between technical and **domain experts** is crucial to creating an application model that will solve particular domain problems.

# Important terms in Domain-Driven Design

In DDD, it’s important to pay attention to the following terms:

## Domain logic

Domain logic is **the purpose of your modeling**. Most commonly, it’s referred to as the **business logic.** This is where your business rules define the way data gets created, stored, and modified.

## Domain model

Domain model includes the **ideas, knowledge, data, metrics, and goals** that revolve around that problem you’re trying to solve. It contains all the rules and patterns that will help you deal with complex business logic. Moreover, they will be useful to meet the requirements of your business.

## Subdomain

A domain consists of several subdomains that refer to **different parts of the business logic**. For example, an online retail store could have a product catalog, inventory, and delivery as its subdomains.

## Design patterns

Design patterns are all about **reusing code**. No matter the complexity of the problem you encounter, someone who’s been doing object-oriented programming has probably already created a pattern that will help you solve it. Breaking down your problem into its initial elements will lead you to its solution. Everything you learn through patterns, you can later use for any object-oriented language you start to program in.

## Bounded context

Bounded context is a **central pattern** in domain-driven design that contains the complexity of the application. It handles large models and teams. This is where you implement the code, after you’ve defined the **domain** and the **subdomains.**

Bounded contexts actually represent **boundaries in which a certain subdomain is defined and applicable.** Here, the specific subdomain makes sense, while others don’t. One entity can have different names in different contexts. When a subdomain within the bounded context changes, the entire system doesn’t have to change too. That’s why developers use adapters between contexts.

## The Ubiquitous Language

The Ubiquitous Language is a methodology that refers to **the same language domain experts and developers use when they talk about the domain they are working on.** This is necessary because projects can face serious issues with a disrupted language. This happens because domain experts use their own jargon. At the same time, tech professionals use their own terms to talk about the domain.

There’s a gap between the terminology used in daily discussions and the terms used in the code. That’s why it’s necessary to define a set of terms that everyone uses. All the terms in the ubiquitous language are structured around the domain model.

## Entities

Entities are a combination of **data and behavior**, like a user or a product. They have identity, but represent data points with behavior.

## Value objects and aggregates

Value objects **have attributes**, but can’t exist on their own. For example, the shipping address can be a value object. Large and complicated systems have countless entities and value objects. That’s why the domain model needs some kind of structure. This will put them into logical groups that will be easier to manage. These groups are called **aggregates.** They represent a collection of objects that are connected to each other, with the goal to treat them as units. Moreover, they also have an **aggregate root.** This is the only entity that any object outside of the aggregate can reference to.

## Domain service

The domain service is **an additional layer that also contains domain logic**. It’s part of the domain model, just like entities and value objects. At the same time, the **application service** is another layer that doesn’t contain business logic. However, it’s here to coordinate the activity of the application, placed above the domain model.

## Repository

The repository pattern is a **collection of business entities** that simplifies the data infrastructure. It releases the domain model from infrastructure concerns. The **layering** concept enforces the separation of concerns.

# Example of Domain-Driven Design

![img](https://miro.medium.com/v2/resize:fit:1094/0*28z_AXhf8mXoUihE.png)

Domain-driven design example

If we take an e-commerce app, for example, the business domain would be to process an order. When a customer wants to place an order, they first need to go through the products. Then, they choose their desired ones, confirm the order, choose shipping type, and pay. The app then processes the data the client provides.

So, a user app would consist of the following layers:

## User Interface

This is where the customer can find **all the information needed to place an order**. In an e-commerce case, this is where the products are. This layer presents the information to the client and interprets their actions.

## Application layer

This layer doesn’t contain business logic. It’s the part that **leads the user from one to another UI screen**. It also interacts with application layers of other systems. It can perform simple validation but it contains no domain-related logic or data access. Its purpose is to organize and delegate domain objects to do their job. Moreover, it’s the only layer accessible to other bounded contexts.

## Domain layer

This is where the **concepts of the business domain** are. This layer has all the information about the business case and the business rules. Here’s where the **entities** are. As we mentioned earlier, entities are a combination of data and behavior, like a user or a product.

They have a unique identity guaranteed via a unique key and remains even when their attributes change. For example, in an e-commerce store, every order has a unique identifier. It has to go through several actions like confirming and shipping to be considered as an entity.

On the other hand, **value objects** don’t have unique identifiers. They represent attributes that various entities can share. For example, this could be the same last name of different customers.

This part also contains services with **defined operational behavior** that don’t have to be a part of any domain. However, they are still part of the business domain. The services are named according to the ubiquitous language. They shouldn’t deprive entities and value objects of their clear accountability and actions. Customers should be able to use any given service instance. The history of that instance during the lifetime of the application shouldn’t be a problem.

Most importantly, the domain layer is in the center of the business application. This means that it should be separated from the rest of the layers. It shouldn’t depend on the other layers or their frameworks.

## Infrastructure layer

This layer **supports communication between other layers** and can contain supporting libraries for the UI layer.

# Advantages of Domain-Driven Design

- **Simpler communication.** Thanks to the Ubiquitous Language, communication between developers and teams becomes much easier. As the ubiquitous language is likely to contain simpler terms developers refer to, there’s no need for complicated technical terms.
- **More flexibility.** As DDD is object-oriented, everything about the domain is based on and object is modular and caged. Thanks to this, the entire system can be modified and improved regularly.
- **The domain is more important than UI/UX.** As the domain is the central concept, developers will build applications suited for the particular domain. This won’t be another interface-focused application. Although you shouldn’t leave out UX, using the DDD approach means that the product targets exactly the users that are directly connected to the domain.

# Downsides of Domain-Driven Design

- **Deep domain knowledge is needed.** Even for the most technologically advanced teams working on development, there has to be at least one domain specialist on the team who understands the precise characteristics of the subject area that’s the center of the application. Sometimes there’s a need for several team members who thoroughly know the domain to incorporate in the development team.
- **Contains repetitive practices.** Although many would say this is an advantage, the domain-driven design contains many repetitive practices. DDD encourages the use of continuous integration to build strong applications that can adapt themselves when necessary. Many organizations may have difficulties with these methods. More particularly, if their previous experience is generally tied to less-flexible models of growth, like the waterfall model.
- **It might not work best for highly-technical projects.** Domain-driven design is perfect for applications that have complex business logic. However, it might not be the best solution for applications with minor domain complexity but high technical complexity. Applications with great technical complexity can be very challenging for business-oriented domain experts. This could cause many limitations that might not be solvable for all team members.

# Conclusion

Domain-driven design is a software engineering approach to solving a specific domain model. The solution **circles around the business model** by connecting execution to the key business principles.

Common terminology between the domain experts and the development team includes **domain logic, subdomains, bounded contexts, context maps, domain models, and ubiquitous language as a way of collaborating and improving the application model and solving any** domain-related challenges.

With this article, we wanted to define the core concepts around domain-driven design. Moreover, we wanted to explain them, adding the advantages and downsides of the approach. This way, we hope to help you decide whether this is the right approach for your business and your application.

Microservices offer some serious advantages over traditional architectures, providing scalability, accessibility, and flexibility. Moreover, this approach keeps developers focused as each microservice is a loosely coupled service with a single idea of accountability.

![img](https://miro.medium.com/v2/resize:fit:1094/0*dqe_CJi0GgED9LXj.png)

# **Want to get started with domain-driven design?**

[**Make software development efficient today.**](https://microtica.com/?utm_source=medium&utm_medium=medium&utm_campaign=medium_ddd)

Domain Driven Design

Softw