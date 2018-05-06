---
layout: post
title:  "General coding: Dependency Injection In Depth"
date:   2017-05-05 20:00
categories: coding
excerpt: "This article is about dependency injection as a method to manage object graph in the app lifecycle"
---

# Dependency Injection In Depth

This post will describe dependency injection as a way to manage object graph in your app lifecycle including overviews of libraries and use cases.

-----------------

## Table of contents

  - [What is dependency injection?](#whatisit)
  - [Usage Introduction](#usage)
    - [Dynamic reflection object graph evaluation](#dynamic)
    - [Static compile time injection](#static)
  - [Dagger 2](#dagger2)
  - [Guice](#guice)

-----------------

## <a name="whatisit">What is Dependency Injection?</a>

Dependency Injection is a technique of creating objects without using constructor directly, but rather supplying
all the necessary dependencies and letting the DI (Dependency Injection) framework manage it for you.

<br />

<p class="embed-image">
    <img src="{{site.baseurl}}/assets/images/DI-graph-example.png" alt="Drawing"  alt="Sample Injector Representation in a service"/>
</p>

Example DI libraries are here:

- Java
  - [Dagger 2](https://github.com/google/dagger)
  - [Guice](https://github.com/google/guice)
- Golang
  - [Dig](https://github.com/uber-go/dig)
  - [Inject](https://github.com/facebookgo/inject)
- Javascript
  - [BottleJS](https://github.com/young-steveo/bottlejs)

## <a name="usage">Usage Introduction</a>

All approaches of dependency injection try to satisfy the same goal, which is ease of graph object management in different scopes, threads, etc. to avoid memory leaks, concurrency issues, and even some clarification on documentation, because annotations tend to be readable and clear already. Particularly, looking into Java world, we can divide DI into 2 classes `dynamic reflection` and `static compile-time`.

### <a name="dynamic">Dynamic reflection object graph evaluation</a>

Let's take a look at a simplest example of dynamic reflection based object graph population by means of Guice in Java.
Below are regular data access objects and services defined with dependencies. It's important to note that some of them have `@Inject` annotation on the constructor, while others are on the fields.

```java

public interface PlayerDao {}

public class PlayerDaoImpl implements PlayerDao {

  private final GameDao gameDao;

  @Inject
  public PlayerDaoImpl(GameDao gameDao) {
      this.gameDao = gameDao;
  }
}

public interface GameDao {}

public class GameDaoImpl implements GameDao {

    @Inject
    public GameDaoImpl() {

    }
}

public interface MatchService {}

public class MatchServiceImpl implements MatchService {

    @Inject
    private GameDao gameDao;

    @Inject
    private PlayerDao playerDao;

}

```

While the above is the regular code that you would have to write regardless of whether you use DI or not, the below is the code required by Guice in order to let the framework know how exactly you want them to be created, managed, etc.

```java

public class PlayerNetworkModule extends AbstractModule {

    @Override
    protected void configure() {
        bind(PlayerDao.class).to(PlayerDaoImpl.class).in(Scopes.SINGLETON);
        bind(GameDao.class).to(GameDaoImpl.class).in(Scopes.SINGLETON);
        bind(MatchService.class).to(MatchServiceImpl.class).in(Scopes.SINGLETON);
    }
}

public class EntryPoint {
    public static void main(String[] args) throws IOException {

        Injector injector = Guice.createInjector(
          new PlayerNetworkModule(),
          new GraphvizModule()
        );

        PrintWriter out = new PrintWriter(new File("target/graph.dot"), "UTF-8");

        MatchService matchService = injector.getInstance(MatchService.class);

        GraphvizGrapher grapher = injector.getInstance(GraphvizGrapher.class);
        grapher.setOut(out);
        grapher.setRankdir("TB");
        grapher.graph(injector);
    }
}

```

If you noticed, we also included some code to visualize the object graph Via Grapher. [You can learn more about Grapher from the original documentation](https://github.com/google/guice/wiki/Grapher).
In order to convert it to an image use the following commands:

```bash
# assuming on Mac
brew install graphviz
dot -T png graph.dot > graph.png
```

After all of the code above executed, you will eventually be able to see a nice representation of the dependencies graph. Keep in mind that these cool extensions are not necessarily available on all the frameworks, but you are absolutely free to write your own and make it public!

<p class="embed-image" style="text-align: center;">
    <img src="{{site.baseurl}}/assets/images/graphviz.png" alt="Drawing"  alt="Graphiviz Sample Output"/>
</p>


### <a name="static">Static compile time injection</a>


## Pros and cons

-----------------

## <a name="dagger2">Dagger 2</a>

TODO...
