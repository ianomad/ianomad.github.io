---
layout: post
title:  "Dependency injection in code"
date:   2017-05-05 20:00
categories: coding
excerpt: "This article is about dependency injection as a method to manage object graph in the app lifecycle"
---

# Dependency injection in code

This post will describe dependency injection, its common features and types and some examples in different programming languages.

-----------------

## Table of contents

  - [What is dependency injection?](#whatisit)
  - [Usage Introduction](#usage)
    - [Dynamic reflection object graph evaluation](#dynamic)
      - [GraphvizModule](#graphviz)
  - [Common DI features](#commondifeatures)
    - [Scopes](#scopes)
    - [Automatic Injection, Type Binding, To Instance, To Provider](#feature2)
    - [Annotation Binding](#annotationbinding)
  - [Static compile time injection](#static)
  - [DI in Golang](#diingo)
  - [Best Practices](#bestpractices)

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

All approaches of dependency injection try to satisfy the same goal, which is ease of graph object management in different scopes, threads, etc. to avoid memory leaks, concurrency issues, and even some clarification on documentation, because annotations tend to be readable and clear already. Particularly, looking into Java world, we can divide DI into 2 classes `dynamic reflection` and `static compile-time` (the latter is relatively new and mostly is used in mobile apps).

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
        bind(PlayerDao.class).to(PlayerDaoImpl.class).in(Scopes.SINGLETON); // this is a regular linked binding
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

### <a name="graphviz">GraphvizModule</a>

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

-----------------

## <a name="commondifeatures">Common DI Features</a>

## <a name="scopes">Scopes</a>

This is the most obvious and common feature to manage different dependencies. Below are some examples of scopes:

### Singleton

This scope is the most widely used scope to avoid memory waste, concurrency issues, etc. Below is a sample snippet to implement such a scope.

```java

// The difference between methods vary based on the stage configured via Guice
// see more here: https://github.com/google/guice/wiki/Scopes

// example 1
bind(MatchService.class).in(Singleton.class);

// example 2
bind(MatchService.class).in(Scopes.SINGLETON);

// example 3
@Singleton
class MatchService {
  //...
}

```

### ServletScopes.REQUEST

This method works well for services like web. The idea is injection of certain dependencies within the context of the request (like HTTP request).

```java

bind(UserTokenInfo.class)
      .toProvider(UserTokenInfoProvider.class)
      .in(ServletScopes.REQUEST);

```

You have to make sure that your request is intercepted and has proper configuration to make this work. Guice has a <a href='https://github.com/google/guice/wiki/Servlets' target='_blank'>Servlet interceptor</a> that makes it work only for some frameworks like <a href='https://spring.io/' target='_blank'>Spring</a>.

### Custom Scopes

Obviously, you can have your own scopes and apply them accordingly. This is a more sophisticated feature and not used that often. Normally, simple annotation based approach always works well to separate out same types.

## <a name="feature2">Automatic Injection, Type Binding, To Instance, To Provider</a>

The above title is super obvious, the question is more about which one to use for what. There are multiple ways to let the injector know how to populate your objects.

**Automatic injection** - this method will normally use the default empty constructor to create your object and possibly inject all the fields annotated as well.

**To Instance binding** - the method of injecting an instance directly providing instance is also possible. This is pretty convenient, but sort of deviates from the implicit object creation approach.

```java

MatchService matchService = new MatchServiceImpl();

bind(MatchService.class).toInstance(matchService);

```

**To provider** - finally, this way is very convenient, if you are trying to manually manage the instances provided on every injection. One of the best examples is dynamic configuration or per user request context injection.

```java

// @Provides annotation example
public class PlayerNetworkModule extends AbstractModule {

    @Override
    protected void configure() {
        //...
    }

    @Provides
    public MatchService providesMatchService() {
      new MatchServiceImpl();
    }
}

// Provider implementation example
public class PlayerNetworkModule extends AbstractModule {

    @Override
    protected void configure() {
        bind(MatchService.class).toProvider(MatchProvider.class);
    }
}

public class MatchProvider<MatchService> {
  MatchService get() {
    return new MatchServiceImpl();
  }
}

```

## <a name="annotationbinding">Annotation Binding</a>

Using annotation binding to differentiate the binding scope for different objects is very useful. See below simple named or custom annotation bindings.

#### Named binding

This method is very convenient to differentiate the objects, but it requires hardcoding of the names as strings which is easy to misspell, etc. See below example usage.

```java

public class PlayerNetworkModule extends AbstractModule {

    @Override
    protected void configure() {

      bind(MatchService.class)
        .annotatedWith(Names.named("NFS"))
        .to(NFSMatchServiceImpl.class);

      bind(MatchService.class)
        .annotatedWith(Names.named("PVZ"))
        .to(PVZMatchServiceImpl.class);

    }

    class MatchMakingProxy {

      @Inject
      @Named("NFS")
      MatchService nfsMatches;

      @Inject
      @Named("PVZ")
      MatchService pvzMatches;

    }
}

```


#### Custom annotation binding

This method is a little better IMHO than the above, because you avoid mistyping and just use the declared annotations.

```java

@BindingAnnotation @Target({ FIELD, PARAMETER, METHOD }) @Retention(RUNTIME)
public @interface NFS {}

@BindingAnnotation @Target({ FIELD, PARAMETER, METHOD }) @Retention(RUNTIME)
public @interface PVZ {}

public class PlayerNetworkModule extends AbstractModule {

    @Override
    protected void configure() {

      bind(MatchService.class)
        .annotatedWith(NFS.class)
        .to(NFSMatchServiceImpl.class);

      bind(MatchService.class)
        .annotatedWith(PVZ.class)
        .to(PVZMatchServiceImpl.class);

    }

    class MatchMakingProxy {

      @Inject
      @NFS
      MatchService nfsMatches;

      @Inject
      @PVZ
      MatchService pvzMatches;

    }
}

```

### <a name="static">New Static Compile Time Dependency Injection</a>

Dependency Injection has been known for a long time and has been used and implemented in many languages. Now that there is a higher demand on DI to populate the object graph super quickly, there is a new implementation of it via static compile time. For example, there is a frequent cold start time in mobile apps (upon the app screen lifecycle going background/foreground with all the crazy memory optimizations) and it's crucial to have it instant which is not that great via traditional dynamic reflection based injection. In short, the idea of static compile time injection is code generation of all the object dependencies, so that we don't have to perform heavy reflection operations every time app starts and possibly error out all the mismatches during the code compilation. See below example of how it's setup with Dagger 2.

First of all, we start off with the basic setup of the same dependency entities.

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

Now we have to setup the same module as in the previous dynamic approach, but more of the provider approach (below is just an example, but Dagger 2 has a lot more flexible ways of binding and scoping objects)


```java

@Module
public class PlayerNetworkModule {

    @Singleton
    @Provides
    public GameDao provideGameDao() {
        return new GameDaoImpl();
    }

    @Singleton
    @Provides
    public PlayerDao providePlayerDao(GameDao gameDao) {
        return new PlayerDaoImpl(gameDao);
    }

    @Provides
    @Singleton
    public MatchService provideMatchService(MatchServiceImpl matchService) {
        return matchService;
    }

}

```

Finally, let's setup a component that will install the corresponding module and connect all the necessary bindings.

```java

@Singleton
@Component(modules = PlayerNetworkModule.class)
public interface PlayerNetworkComponent {

    void inject(EntryPoint entryPoint);

    MatchService matchService();

}

```

As you can see above, we specify what modules should be included, and more importantly how we access the dependencies (it can be either an injection into the classes with its own injections or a simple getter like `PlayerNetworkComponent.matchService()`). We define only the interface, so that the library will help us generate all the code with the actual classes that will create everything via constructors directly, etc.

```java

public class EntryPoint {

    @Inject
    MatchService matchService;

    public EntryPoint() {
        PlayerNetworkComponent playerNetworkComponent =
                DaggerPlayerNetworkComponent.builder()
                        .playerNetworkModule(new PlayerNetworkModule())
                        .build();

        playerNetworkComponent.inject(this);

        System.out.println(playerNetworkComponent.matchService());
        System.out.println(matchService);
    }

    public static void main(String[] args) throws IOException {
        new EntryPoint();

        // output is the populated singleton service
        // --> com.example.di.MatchServiceImpl@2626b418
        // ->> com.example.di.MatchServiceImpl@2626b418
    }

}

```

The compiler generated lots of code with factories with corresponding classes with `@Inject` annotated constructors. We endup using only the class `DaggerPlayerNetworkComponent` which is explicitly prefixed with `Dagger` word. See below some of the code snippets of the generated code.

```java

public final class MatchServiceImpl_Factory implements Factory<MatchServiceImpl> {
    private final Provider<GameDao> gameDaoProvider;
    private final Provider<PlayerDao> playerDaoProvider;

    public MatchServiceImpl_Factory(Provider<GameDao> gameDaoProvider, Provider<PlayerDao> playerDaoProvider) {
        this.gameDaoProvider = gameDaoProvider;
        this.playerDaoProvider = playerDaoProvider;
    }

    public MatchServiceImpl get() {
        return provideInstance(this.gameDaoProvider, this.playerDaoProvider);
    }

    public static MatchServiceImpl provideInstance(Provider<GameDao> gameDaoProvider, Provider<PlayerDao> playerDaoProvider) {
        return new MatchServiceImpl((GameDao)gameDaoProvider.get(), (PlayerDao)playerDaoProvider.get());
    }

    public static MatchServiceImpl_Factory create(Provider<GameDao> gameDaoProvider, Provider<PlayerDao> playerDaoProvider) {
        return new MatchServiceImpl_Factory(gameDaoProvider, playerDaoProvider);
    }

    public static MatchServiceImpl newMatchServiceImpl(GameDao gameDao, PlayerDao playerDao) {
        return new MatchServiceImpl(gameDao, playerDao);
    }
}


@Generated(
  value = "dagger.internal.codegen.ComponentProcessor",
  comments = "https://google.github.io/dagger"
)
public final class DaggerPlayerNetworkComponent implements PlayerNetworkComponent {
  private Provider<GameDao> provideGameDaoProvider;

  private Provider<PlayerDao> providePlayerDaoProvider;

  private MatchServiceImpl_Factory matchServiceImplProvider;

  private Provider<MatchService> provideMatchServiceProvider;

  private DaggerPlayerNetworkComponent(Builder builder) {
    initialize(builder);
  }

  public static Builder builder() {
    return new Builder();
  }

  public static PlayerNetworkComponent create() {
    return new Builder().build();
  }

  @SuppressWarnings("unchecked")
  private void initialize(final Builder builder) {
    this.provideGameDaoProvider =
        DoubleCheck.provider(
            PlayerNetworkModule_ProvideGameDaoFactory.create(builder.playerNetworkModule));
    this.providePlayerDaoProvider =
        DoubleCheck.provider(
            PlayerNetworkModule_ProvidePlayerDaoFactory.create(
                builder.playerNetworkModule, provideGameDaoProvider));
    this.matchServiceImplProvider =
        MatchServiceImpl_Factory.create(provideGameDaoProvider, providePlayerDaoProvider);
    this.provideMatchServiceProvider =
        DoubleCheck.provider(
            PlayerNetworkModule_ProvideMatchServiceFactory.create(
                builder.playerNetworkModule, matchServiceImplProvider));
  }

  @Override
  public void inject(EntryPoint entryPoint) {
    injectEntryPoint(entryPoint);
  }

  @Override
  public MatchService matchService() {
    return provideMatchServiceProvider.get();
  }

  private EntryPoint injectEntryPoint(EntryPoint instance) {
    EntryPoint_MembersInjector.injectMatchService(instance, provideMatchServiceProvider.get());
    return instance;
  }

  public static final class Builder {
    private PlayerNetworkModule playerNetworkModule;

    private Builder() {}

    public PlayerNetworkComponent build() {
      if (playerNetworkModule == null) {
        this.playerNetworkModule = new PlayerNetworkModule();
      }
      return new DaggerPlayerNetworkComponent(this);
    }

    public Builder playerNetworkModule(PlayerNetworkModule playerNetworkModule) {
      this.playerNetworkModule = Preconditions.checkNotNull(playerNetworkModule);
      return this;
    }
  }
}


```



## <a name="diingo">DI in Golang</a>

Let's see a super basic example of how DI would look like in Golang with Facebook Inject library.

```golang

package main

import (
    "fmt"
    "net/http"
    "os"

    "github.com/facebookgo/inject"
)

type PlayerNetworkGraph struct {
    // The tags below indicate to the inject library that these fields are
    // eligible for injection. They do not specify any options, and will
    // result in a singleton instance created for each of the APIs.

    MatchService   *MatchService   `inject:""`
    GameDao *GameDao `inject:""`
}

func main() {
    var g inject.Graph

    var a PlayerNetworkGraph
    err := g.Provide(
        &inject.Object{Value: &a}
    )

    if err != nil {
        fmt.Fprintln(os.Stderr, err)
        os.Exit(1)
    }
}

```

## <a name="bestpractices">Best practices link</a>

Guice documentation has a good list of best practices for DI. <a href="https://github.com/google/guice/wiki" target="_blank">Check it out</a>.
