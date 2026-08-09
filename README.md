# cqrs-4-quarkus

The Quarkus flavour of [cqrs-4-java](https://github.com/fuinorg/cqrs-4-java) — everything a Quarkus
application needs to run the CQRS base classes, and nothing that is not Quarkus specific.

Split out of the cqrs-4-java repository so the two framework flavours can move at their own pace. **The
coordinates did not change:** every artifact keeps its `org.fuin.cqrs4j` groupId and its
`cqrs-4-java-quarkus-*` artifactId. A consumer sees one more BOM to import and nothing else.

## Using it

Import both BOMs — this one for what is built here, cqrs-4-java's for `core`, `esc`, `jsonb`, `jpa` and
`test-helper` — then declare the modules you need without versions:

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.fuin.cqrs4j</groupId>
            <artifactId>cqrs-4-java-bom</artifactId>
            <version>${cqrs4j.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
        <dependency>
            <groupId>org.fuin.cqrs4j</groupId>
            <artifactId>cqrs-4-java-quarkus-bom</artifactId>
            <version>${cqrs4j-quarkus.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

**Import `org.fuin:bom` before `quarkus-bom`** in your own project, as this repository's root pom does.
An imported BOM only wins against another imported BOM declared later in the same file, so the deliberate
overrides in `org.fuin:bom` — `protobuf-java`, which the KurrentDB client needs at 4.28.2 while
`quarkus-bom` pins 3.25.x — are otherwise silently ignored.

## Modules

Every module has a `README.md` of its own; this is the map.

| Module | |
|---|---|
| [`common`](common) | What the others share: the event store configuration and the response filter that clears thread locals between requests. |
| [`command`](command) | The write side: the REST resource that accepts commands, the dispatcher behind it and the bulkhead that bounds it. |
| [`query`](query) | The read side: the registry and manager that run projections, with their leases, positions and freshness. |
| [`process-manager`](process-manager) | The outbox that delivers commands over REST, and the sweeper that fires timeouts. |
| [`keycloak`](keycloak) | Multi-tenant Keycloak authentication: one tenant per realm, the tenant context, and the command execution context taken from the JWT. |

Not published: [`jacoco`](jacoco) aggregates the coverage report, and [`test`](test) is the sample
application the integration tests drive.

## Building

Requires JDK 25 and a container runtime for the integration tests.

```bash
./mvnw clean verify -s settings.xml
```

`-s settings.xml` adds the snapshot repository every `org.fuin` dependency resolves from; without it the
build fails on the first snapshot.
