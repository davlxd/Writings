# Spring Introduction
2017-03-17

This article is a recap of a presentation I gave to my teammates, a brief introduction to Spring framework and related concepts, and 3 common Spring projects (Spring MVC, Spring Data JAP, and Spring Boot).

### Spring and Java EE

![Spring and Java EE](images/spring-java-ee.png)

Let’s talk a bit about Java first. Java has multiple **edition**s: Java SE, Java EE and Java ME. Java SE and Java EE are closely related to Spring framework. Java SE (Java Platform, Standard Edition) contains JVM and all core libraries and APIs, while Java EE (Java Platform, Enterprise Edition) is a set of specifications and APIs which will be used to develop large-scale enterprise applications, such as `servlet specification` used to handle HTTP request and `JPA` describes the management of relational database. Java EE constitutes of JSRs (Java Specification Request) maintained by JCP (Java Community Process), just like RFC to IETF.

There are several Java EE implementations in the industry, all belong to IT giants, like IBM’s WebSphere Application Server (WAS), Oracle’s WebLogic, and JBoss which belongs to RedHat. They all comply to Java EE standards, and they all implement most of Java EE specifications like Servlet and JPA, and they mostly are called `Application Server`s since Java EE makes sure our applications can run upon. The open source alternatives of these application servers like Tomcat or Jetty are called `Web Container`, since they only implement Web related specifications such as Server spec and JSP specs. If we need dependency injection we bring in Spring, if we need to interact with relational databases, we bring in Hibernate.

And strictly speaking, Spring is not an implementation of Java EE, since even she does implement lower level Java EE specs like Servlet and JPA, but she uses its own stuff for higher level APIs like dependency injection and RESTful web service. Java EE specs for dependency injection is CDI while Spring uses Spring DI, and for REST it’s JAX-RS vs Spring MVC.

### Spring IoC / DI
At its core, the foundation concept of Spring Framework is DI (Dependency Injection) or a.k.a. IoC (Inversion of Control). These 2 abbreviations are really novice-unfriendly, and so require some explanations.

IoC is a theory, a technology, not only exist in Spring. Generally speaking, for some entity, the work/flow which this entity should initiate, turns out to this entity passively receive, this active-passive turnover is IoC. For example you want to write a small trivial program at work, at the beginning it’s a small program calling some 3rd party libraries, this is the active role your code is playing, as your trivial program grew bigger you import a framework to sort things out, your code becomes callbacks or interface implementations, now your code plays the passive role. With the help of IoC we can achieve decoupling, free our hands by relieving responsibilities.  

For Spring IoC, it’s the responsibility of object instantiation and dependency tree maintenance that being shifted from our hands to Spring.  For example, when we write Java application without Spring IoC, it could be like this:

```Java
public class B {
}

public class C {
}

public class A {
    private B b;
    private C c = new C();

    public A(B b) { this.b = b; }
    public void setB(B b) { this.b = b; }
}
A a = new A(new B());
```

It’s class A depends on Class B and Class C, we pass the objects of A and B through constructor and setter function. With the help of Spring IoC and some other Spring tricks, it could be like this:

```Java
@Component
public class B {
}

@Component
public class C {
}

@Component
public class A {
    private B b;
    @Autowired private C c;

    @Autowired
    public A(B b) { this.b = b; }
}
```

`@Component` annotations tell Spring the annotated classes need to be instantiated as beans, which are Java objects managed by Spring, while `@Autowired` annotations tell Spring to inject the type-matched beans to the annotated fields, and this is duly `dependency injection`.


### Web container
Web container a.k.a. Servlet container is different from Web Server. Nginx and Apache are web servers, which mainly handle HTTP requests. Web containers like Tomcat, Jetty or GlassFish, however, are like fish tanks where Java Servlets live in. Application Server I mention above can be considered as a superset of Web container, since it’s capable of more than containing servlets. Almost all Java Web applications we write in the end are a bunch of servlets or facaded by servlets, these servlets cannot run on their own, neither can they directly handle HTTP requests. They need to be managed by Web containers, and web containers dispatch HTTP requests to servlets based on mapping rules, like below:

![Servlet](images/servlet-container.png)

The whole process is Java EE spec, which Web containers, application servers, and Java Web applications should all follow, so we can deploy our applications to any Web containers or application servers with minor adaptation.



### Spring MVC
Spring framework consists of many modules, spring-core and spring-beans are at at core, spring-jdbc and spring-orm are responsible for database interaction etc. It’s too common to use Spring to build backend RESTful APIs, so it’s necessary to give a quick view of Spring Web MVC module.

As I stated before you cannot get away with servlets for HTTP in Java, Spring MVC is tangled with servlet as well. Spring MVC is facaded by `DispatchServlet` which accepts all HTTP requests and then dispatch to other HTTP endpoints in your application. The following is an HTTP endpoint demo with Spring MVC:

```Java
@RestController
@RequestMapping("/stores/{storeId}/items")
public class ItemController {
    @RequestMapping(path = "/{itemId}", method = RequestMethod.GET, produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
    public Item retrieveItem(@PathVariable String storeId, @PathVariable String itemId) {
        ...
        return item;
    }

    @RequestMapping(method = RequestMethod.POST)
    public ResponseEntity createItem(@RequestBody Item item) {
        ...
        return new ResponseEntity<>(HttpStatus.OK);
    }
}
```


- `@RestController` is annotated on Java Class, which means annotated Class is a RESTful API endpoint, hence the Controller in MVC architecture. All Controllers and RestControllers are Spring-managed beans as well.
- `@RequestMapping` defines URL routing mapping rules, the one annotated on Class can specify absolute path while the one annotated on method specify the relative path. And methods annotated with `@RequestMapping` are responsible for handle HTTP requests and serve responses.
- Paths specified by `@RequestMapping` comply with URL Template<sup>[[1](https://tools.ietf.org/html/rfc6570)]</sup> and `@PathVariable` can extract variable values in it.
- By annotating `@RequestBody` on method parameters, Spring will convert HTTP request body to Java object and inject to annotated parameters. And Spring also will convert method return value to HTTP response in JSON format, since class is annotated with `@RestController`
- `ResponseEntity` is a generic class containing HTTP response body, and HTTP headers and status code which you can tweak. In this demo the method responds empty body with status code specified to 200. 

With an extremely simplified Class Item, the terminal should look like this if we call the RESTful API with cURL

```Java
public class Item {
    private String name;
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
}
```

```
$ curl -v http://localhost:8080/stores/16/items/1
...
> 
< HTTP/1.1 200 OK
< Server: Apache-Coyote/1.1
< Content-Type: application/json;charset=UTF-8
...

{"name":"hehe"}%
```


```
curl -v -H "Content-Type: application/json" -i -X POST -d '{"name":"haha"}' http://localhost:8080/stores/16/items
...
> Content-Type: application/json
> Content-Length: 15
> 
< HTTP/1.1 200 OK
< Server: Apache-Coyote/1.1
< Content-Length: 0
...

```



### Spring Data JPA
Spring has always been considered a framework, but in recent years it’s more like a portfolio, contains dozens of [projects](https://spring.io/projects), all based on original Spring framework but provide various abundant features, Spring Data is one of them. Spring Data provides consistent, elegant, and easy-to-use interfaces to access persistent facilities, and Spring Data JPA is one of the modules specifically deal with relational databases. As we can see soon, Spring Data JPA provides a set of very tidy interfaces for us to implement routine CRUDs.

Spring Data JPA exposes a bunch of Java interfaces to users, behind the scenes it uses Dynamic Proxy AOP to inject actual work in runtime. The interface at the top is `Repository<T, ID extends Serializable>`, then it’s `CrudRepository<T, ID extends Serializable>` providing basic CRUD operations, and `JpaRepository<T, ID extends Serializable>` with JPA related actions etc. They all generic interfaces whose first type parameter is entity class corresponding to a database table, and the 2nd parameter is the type of ID in the entity class. To use Spring Data JPA all we need is create an interface extends one of these interfaces and then declare methods we will be calling in it.

The following example demonstrates how Spring Data JPA simplifies the process of exposing CRUD interfaces to a simple data entity.

Given the table structure and Entity class:

```sql
CREATE TABLE item (
  id          VARCHAR(36)  NOT NULL,
  name        VARCHAR(128) NOT NULL,
  description VARCHAR(128),
  CONSTRAINT itemPk PRIMARY KEY (id)
);
```

```Java
@Entity
public class Item {
    @Id
    @GeneratedValue(generator = "uuid")
    @GenericGenerator(name = "uuid", strategy = "uuid2")
    private String id;

    public Item() {
    }

    public Item(String name, String description) {
        this.name = name;
        this.description = description;
    }
    private String name;
    private String description;
}
```


Firstly we declare an interface extends `CrudRepository`, and designate type parameter explicitly 

```Java
public interface ItemRepository extends CrudRepository<Item, String> {
}
```

Since `CrudRepository` already has common CRUD methods like `save(S entity)`, `findOne(ID id)`, `exists(ID id)` and `delete(ID id)`, so now we only need to fill in some query methods specific to our need, as long as follow naming conventions[1]. For example, if we’d like to find all items with the same name, we can declare `findByName` [here](http://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories.query-methods.query-creation)

```Java
public interface ItemRepository extends CrudRepository<Item, String> {
    List<Item> findByName(String name);
}
```


Inject `ItemRepository` to wherever we need, then we are able to call these CURD methods directly.

```Java
public class SomeClient {
  @Autowired private ItemRepository itemRepository;

  public void doSomething() {
    List<Item> items = itemRepository.findByName("teapot");
  }
  public void doSomethingElse() {
    Item newItem = new Item("smug", "smug without mug");
    itemRepository.save(newItem);
  }
}
```


User-defined query method mimics Rails [Dynamic Finders](http://guides.rubyonrails.org/active_record_querying.html#dynamic-finders) and it supports more sophisticated combinations like and, or, pagination, limit, asc/desc etc, you can refer to official documentation here[2].

How does Spring Data JPA do all the magic tricks? Well, it’s not that magic for Spring core. During Spring application bootup, Spring creates a proxy bean for `ItemRepository` interface at first with the help of Spring AOP, then in that proxy Spring parses all user-defined query method names and store ASTs on the bean. When these query methods get called in runtime, Spring AOP intercepts the method call, fetch the AST, assemble into a Query object combined with query parameters, then hand over to JPA. I give a more thorough analysis in [another post](/2017-03-22-spring-data-jpa-internals-en).


### Spring Boot

Spring Boot is another Spring project, a Convention over Configuration follower and a relatively successful one. It has done a lot of work based on Spring which makes it much easier to build a Spring application.

In old days we need to list all depending artifacts in pom.xml or gradle.build, which is really annoying and cumbersome, Spring Boot thoughtfully gives us `Spring Boot Starters`, a one-stop-shop for all the Spring and related technology. It organizes related dependencies into groups from a Spring user’s perspective, we only need to include the group dependency descriptor artifactId in our pom.xml or build.gradle. For example when we develop a Spring Web application, generally we need to import Spring MVC, spring-web module, and Jackson etc, with Spring Boot we just import `spring-boot-starter-web` once for all. The same goes to `spring-boot-starter-test` which includes JUnit, Hamcrest and Mockito; and `spring-boot-starter-data-jpa` which includes everything related to relational database interaction.

Like I said before Spring Web applications need to put into Servlet containers in order to run, however, `spring-boot-starter-web` starter includes an embedded Tomcat, which can be started through `SpringApplication.run()` within our code in main function. In this way, our Spring Web applications are directly runnable via commandline, or IDE, or `java -jar` after packaged into a independent jar file.

Spring Boot also has fine-tuned a lot of details to keep to Convention over Configuration. Hibernate has `ImprovedNamingStrategy` class automatically map camelCase field names in Entity class to snake_case field names in DB tables, however it doesn’t support foreign keys. Spring Boot provides `SpringNamingStrategy` which inherits `ImprovedNamingStrategy` and adds foreign keys support.


