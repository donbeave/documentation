---
date: 2018-11-07T00:11:02+01:00
title: Getting started
weight: 10
type: index
---

## Build with Gradle

First you set up a basic build script. You can use any build system you like, but the code you need to work with 
Gradle and Maven is included here. 

### Create a Gradle build file

Make sure `mavenCentral` is among your repositories. The example build script below uses `org.gretty` to provide
a webserver to show a working example. You need to add the `jcenter` repository as well to be able to pull in all
dependencies required by `gretty`.

```gradle
plugins {
    id 'war'
    id 'idea'
    id 'org.gretty' version '2.2.0'
}

repositories {
    mavenCentral()
    jcenter()
}

dependencies {
    compile 'com.graphql-java-kickstart:graphql-java-servlet:6.2.0'
}
```

## Build with Maven

First you set up a basic build script. You can use any build system you like, but the code you need to work with 
Gradle and Maven is included here. We're assuming you already have a basic understanding of the build system of your
choosing and how to use it. So instead of providing fully functioning build script we only provide the sections
needed to add `graphql-java-servlet` to your application.

### Update your Maven pom file

Add the `graphql-java-servlet` dependency to your `depencies` section:

```xml
<dependency>
  <groupId>com.graphql-java-kickstart</groupId>
  <artifactId>graphql-java-servlet</artifactId>
  <version>6.2.0</version>
</dependency>
```

## Create a Servlet class

Create a servlet class extending `SimpleGraphQLHttpServlet`. This is the basic "Hello world" example. It creates
the GraphQL Query `{ hello }` that can be executed and which will respond with "world". It creates the GraphQL
schema programmatically as shown in the [getting started example](https://www.graphql-java.com/documentation/v11/getting-started/) from `graphql-java`. 

```java
@WebServlet(name = "HelloServlet", urlPatterns = {"graphql"}, loadOnStartup = 1)
public class HelloServlet extends SimpleGraphQLHttpServlet {

  public HelloServlet() {
    super(invocationInputFactory(), queryInvoker(), objectMapper(), null, false);
  }

  private static GraphQLInvocationInputFactory invocationInputFactory() {
    return GraphQLInvocationInputFactory
        .newBuilder(new DefaultGraphQLSchemaProvider(createSchema()))
        .build();
  }

  private static GraphQLSchema createSchema() {
    String schema = "type Query{hello: String}";

    SchemaParser schemaParser = new SchemaParser();
    TypeDefinitionRegistry typeDefinitionRegistry = schemaParser.parse(schema);

    RuntimeWiring runtimeWiring = newRuntimeWiring()
        .type("Query", builder -> builder.dataFetcher("hello", new StaticDataFetcher("world")))
        .build();

    SchemaGenerator schemaGenerator = new SchemaGenerator();
    return schemaGenerator.makeExecutableSchema(typeDefinitionRegistry, runtimeWiring);
  }

  private static GraphQLQueryInvoker queryInvoker() {
    return GraphQLQueryInvoker.newBuilder()
        .withExecutionStrategyProvider(executionStrategyProvider())
        .build();
  }

  private static ExecutionStrategyProvider executionStrategyProvider() {
    return new DefaultExecutionStrategyProvider(new AsyncExecutionStrategy(), null,
        new SubscriptionExecutionStrategy());
  }

  private static GraphQLObjectMapper objectMapper() {
    return GraphQLObjectMapper.newBuilder().build();
  }
}
```

## Run the application

To run the application, execute the gradle `appRun` task that was added by `org.gretty`. If all goes well it will
start a jetty webserver and expose the Hello world Servlet on localhost. It will log the exact endpoint, like below:

```plain
17:45:53 INFO  Jetty 9.2.24.v20180105 started and listening on port 8080
17:45:53 INFO  graphql-java-servlet-hello-world runs at:
17:45:53 INFO    http://localhost:8080/graphql-java-servlet-hello-world

> Task :graphql-java-servlet-hello-world:appRun
Press any key to stop the server.
```

### Send a GraphQL Query

You can now send a GraphQL query to your local servlet, for example using [Insomnia](https://insomnia.rest/graphql/).
In this case the URL to post the GraphQL query to is http://localhost:8080/graphql-java-servlet-hello-world/graphql.

The following GraphQL query is what the example implementation supports:
```graphql
query {
    test
}
```
Our Hello Servlet will respond with:
```json
{
	"data": {
		"hello": "world"
	}
}
```