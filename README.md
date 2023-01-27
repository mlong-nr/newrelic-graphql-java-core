[![New Relic Community Project header](https://github.com/newrelic/open-source-office/raw/master/examples/categories/images/Community_Project.png)](https://github.com/newrelic/open-source-office/blob/master/examples/categories/index.md#new-relic-community-project)

# newrelic-graphql-java-core

This library provides helpers for integrating [graphql-java](https://github.com/graphql-java/graphql-java) with your JVM application. These simplify schema configuration and input mapping, and are in active use for GraphQL services at New Relic.

## Getting Started

### Maven
```
    <dependency>
      <groupId>com.newrelic.graphql</groupId>
      <artifactId>core</artifactId>
      <version>0.1.1</version>
    </dependency>
```

### Gradle
```
compile("com.newrelic.graphql:core:0.1.1")
```

## Usage

Version specific Javadocs can be found [here](https://newrelic.github.io/newrelic-graphql-java-core/docs/v0.1.1/).

### [`SimpleGraphQLBuilder`](https://github.com/newrelic/newrelic-graphql-java-core/blob/master/src/main/java/com/newrelic/graphql/schema/SimpleGraphQLBuilder.java)

`SimpleGraphQLBuilder` provides cleaner wireup of an executable schema. It expects a `Reader` with your GraphQL schema contents. Further configuration is done with a fluent interface.

```java
    Reader schemaReader = //... Read your GraphQL SDL from somewhere
    GraphQL graphql = new SimpleGraphQLBuilder(schemaReader)
        .fetcher("Query", "myField", new QueryMyFieldFetcher())
        .scalar("MyScalar",
                GraphQLScalarType
                    .newScalar()
                    .name("MyScalar")
                    .coercing(new MyScalarCoercing())
                    .build())
        .exceptionHandler(new MyDataFetcherExceptionHandler())
        .build()
```

Defauls from the builder:

* The `fetcher` method associates a `DataFetcher` implementation with a GraphQL type and field definition
* Unregistered scalars are defaulted to a String coercion for simplicity of starting up.
* The default type resolver expects result class names to align with GraphQL types. You only need to override this if your scheme is different.
* Defaults exception handler to `SimpleDataFetcherExceptionHandler` from the `graphql-java` library

If functionality isn't available on the the fluent interface, a `configure` callback provides access the underlying `graphql-java` objects.

```java
    GraphQL graphql = new SimpleGraphQLBuilder(schemaReader)
        .configure((typeRegistry, runtimeWiringBuilder) -> {
            // Use the builder to do your work
        })
        .build()
```

### [`GraphQLInputMapper`](https://github.com/newrelic/newrelic-graphql-java-core/blob/master/src/main/java/com/newrelic/graphql/mapper/GraphQLInputMapper.java)

`GraphQLInputMapper` assists in handling incoming input types. It relies on Jackson, and configures to work between the `graphql-java` types and your custom classes.

```java
    GraphQLInputMapper mapper = new GraphQLInputMapper("com.newrelic.my.model");

    public T get(DataFetchingEnvironment environment) throws Exception {
        InputObject obj = mapper.convert(
                                environment.getArgument("inputObject"),
                                environment.getFieldDefinition().getArgument("inputObject").getType());
        //...
    }
```

### [Custom Scalars](https://github.com/newrelic/newrelic-graphql-java-core/tree/master/src/main/java/com/newrelic/graphql/schema/scalars)

At New Relic we've found lots of uses for custom scalars, especially around time. These predefined scalars are available and registered by default for use in your application. To use these, simply include the related scalar declaration as below in your GraphQL schema file, then use the related Java class in your queries or mutations.

* `scalar EpochMilliseconds` - maps to wrapper around Java `Instant`
* `scalar EpochSeconds` - maps to wrapper around Java `Instant`
* `scalar Milliseconds` - maps to wrapper around Java `Duration`
* `scalar Seconds` - maps to wrapper around Java `Duration`
* `scalar Minutes` - maps to wrapper around Java `Duration`
* `scalar DateTime` - maps to `ZonedDateTime` from ISO 8601 compatible strings

```java
    public T get(DataFetchingEnvironment environment) throws Exception {
        DateTime theDate = environment.getArgument("theDate");
    }
```

To opt out of auto-registration of these predefined scalars, use the `usePredefinedScalars` method on `SimpleGraphQLBuilder`.

## For Developers

### Requirements

* Java 8 or greater

### Building

The project uses Gradle 6 and GitHub Actions for building.

To compile, run the tests and build the jars:

`$ ./gradlew build`

### Code style
This project uses the [google-java-format](https://github.com/google/google-java-format) code style, and it is
easily applied via an included [gradle plugin](https://github.com/sherter/google-java-format-gradle-plugin):

`$ ./gradlew googleJavaFormat verifyGoogleJavaFormat`

Please be sure to run the formatter before committing any changes. There is a `pre-commit-hook.sh` which can
be applied automatically before commits by moving it into `.git/hooks/pre-commit`.

### Licensing
newrelic-graphql-java-core is licensed under the Apache 2.0 License.

newrelic-graphql-java-core also uses source code from third party libraries.
Full details on which libraries are used and the terms under which they are licensed can be found in the
third party notices document.

### Contributing
Full details are available in our [CONTRIBUTING.md](CONTRIBUTING.md) file.
We'd love to get your contributions! Keep in mind when you submit your pull request, you'll need to sign the CLA via the click-through using CLA-Assistant. You only have to sign the CLA one time per project.
To execute our corporate CLA, which is required if your contribution is on behalf of a company, or if you have any questions, please drop us an email at opensource@newrelic.com.

### Publishing

Mainly of interest to maintainers on the project. We publish this library to
Maven Central. This requires a few steps:

* GitHub actions on merge to master will publish to a staging repo on Sonatype
  * The credentials for this are a user access token stored in GitHub action secrets if they need to be updated
  * The [Sonatype UI](https://oss.sonatype.org/) shows a staging repo's specific URL if you want to test it directly
* Following [Sonatype instructions for releasing from Staging](https://central.sonatype.org/publish/release/):
  * Locate and Close the auto-created staging repo
  * Once closed, you can Release the repo which will push it to Maven Central
* Wait a bit... it sometimes takes a while for these changes to propagate

If you're part of New Relic and need access to Sonatype, find someone internally who releases public Java libraries for direction.
