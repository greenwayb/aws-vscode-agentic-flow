# Fix Engine — Requirements

## Summary

This project provides australian stock market information, it is partitioned by relevant subcategories, under the market-data directory, each of which will contain its own identfyable requirements.

## The product


## High-level technical guidance

Just enough direction to keep things on track — specific choices are left to the Coding Agent.

- Use the latest spring boot application framework 
- Use Spring Modulith
- Use HTMX as the web stack with modern designs and controls
- For FIX use quickfixj
- use Maven as the build system
- Use **SQLite** as a database for storing fix messages and tracking message resumption
- End-to-end tests drive the real app in a real browser. The choice of tooling is the Coding Agent's.
- End-to-end testing should be lightweight
- Ensure any Rest APIs built use openapi and swagger ui
- Keep the implementation simple and conventional. Library, data and structure choices are the
  Coding Agent's call, as long as the requirements and success criteria are met.
- Ensure that deprecated APIs or methods are not used within the code base
- There should be no use of node, clean up any residual items from a previous design

## Project Structure
Structure the project a spring modulart monolith, but so i can have one well understood codebase and branching strategy but with the ability to build "mini" jars (grouping of specific modules)

Structure the project into two layers:
 - Domain Modules (Libraries): Plain Maven/Gradle submodules containing your business logic, interfaces, and Spring Modulith configurations. These produce standard, non-executable .jar files.
 - Runtime Apps (Executable JARs): Thin deployment modules that only contain a @SpringBootApplication entry point, application properties, and Maven/Gradle dependencies pointing to the specific domain modules they need.

An example strucutre is illustrated below:
```
my-platform/
├── pom.xml (or settings.gradle)
│
├── modules/                         <-- Pure business libraries (non-runnable)
│   ├── identity/                    <-- Spring Modulith domain module
│   │   ├── pom.xml
│   │   └── src/main/java/...
│   ├── billing/                     <-- Spring Modulith domain module
│   │   ├── pom.xml
│   │   └── src/main/java/...
│   ├── orders/                      <-- Spring Modulith domain module
│   │   └── ...
│   └── shared-kernel/
│
└── apps/                            <-- Deployment runtimes (produce executable JARs)
    ├── monolith-app/                <-- Runs EVERYTHING together
    │   ├── pom.xml                  <-- Depends on identity, billing, orders
    │   └── src/.../MonolithApp.java
    │
    ├── commerce-service/            <-- Mini JAR: Billing + Orders
    │   ├── pom.xml                  <-- Depends only on billing, orders
    │   └── src/.../CommerceApp.java
    │
    └── identity-service/            <-- Mini JAR: Identity alone
        ├── pom.xml                  <-- Depends only on identity
        └── src/.../IdentityApp.java
```

For the ongoing project i want to structure such that we have the following concepts, existing work needs to be placed into the bookings/asx space, this is where all the fix engine requirements currently work, it should handle its own set or requirements and testing within that submodule (market-data/bookins/asx) 

```
market-data
+--- bookings
    +-- asx
      +-- exec-client
      +-- exec-server
      REQUIREMENTS.md
    +-- tmx
       REQUIREMENTS.md
+--- depth
   +-- asx
       REQUIREMENTS.md
   +-- tmx
       REQUIREMENTS.md
+----orderentry
   +-- asx
       REQUIREMENTS.md
   +-- tmx
       REQUIREMENTS.md
   +-- bmr
       REQUIREMENTS.md
```



The domain subprojects do not usse the Spring Boot repackage plugin, They are built as standard Java library jars:
```
<!-- modules/orders/pom.xml -->
<artifactId>orders-module</artifactId>

<dependencies>
    <dependency>
        <groupId>org.springframework.modulith</groupId>
        <artifactId>spring-modulith-starter-core</artifactId>
    </dependency>
</dependencies>
<!-- Do NOT add spring-boot-maven-plugin here -->
```

To conifure a "mini jar" runtime bring relevant modules together and add the spring boot maven plugin, example:

```
<!-- apps/commerce-service/pom.xml -->
<artifactId>commerce-service</artifactId>

<dependencies>
    <!-- Pick and choose the modules for this mini-jar -->
    <dependency>
        <groupId>com.example</groupId>
        <artifactId>orders-module</artifactId>
    </dependency>
    <dependency>
        <groupId>com.example</groupId>
        <artifactId>billing-module</artifactId>
    </dependency>
</dependencies>

<build>
    <plugins>
        <!-- This plugin makes it a standalone executable fat JAR -->
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

The Java entry point only scans the packages it depends on:

```
package com.example.commerce;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication(scanBasePackages = {
    "com.example.orders",
    "com.example.billing"
})
public class CommerceApplication {
    public static void main(String[] args) {
        SpringApplication.run(CommerceApplication.class, args);
    }
}
```

For cross-module communication use spring moduliths Event Externalisation.  

For local development where inter service communication is required, leverage spring-kafka-test and the EmbeddedKafkaBroker.


## Not in scope

<!-- Deliberately left out to keep this buildable in one pass. Do not build these: -->

- GraalVM native-image builds — deferred. Standard executable JVM jars are the deliverable;
  revisit only if startup time becomes a real issue.

## Look and feel

Applies to the whole app:

- Make it **bold and impressive** — it should look designed on purpose. This is a product to show
  off, and first impressions matter.
- Palette: **`#ecad0a` (amber), `#209dd7` (blue) and `#753991` (purple)**, over grays. Both themes
  draw from the same palette; dark mode is a first-class theme, equally considered.
- Avoid the tells of generated design: overuse of gradients, purple-dominated backgrounds, and
  thin accent borders down one side of cards or panels.
- Beyond these rules, layout and visual style are the Coding Agent's call.

## Phases and success criteria

Build in these phases, in order. **Do not start a phase until every success criterion of the
previous phase is demonstrably met** — each criterion must be something you can show working, not
just assert.

Every phase closes the same loop: its unit tests pass, its end-to-end tests pass against the real
app in a real browser, and the new features have been used in the running app with screenshots
taken and inspected. Unit tests accompany every phase on both frontend and backend, building
toward the coverage target verified in Phase 6.

### Phases
Phases are defined in each of the market data subitem REQUIREMENTS.md files 
