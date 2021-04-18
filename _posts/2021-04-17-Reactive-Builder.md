# Reactive Builder
This document shows a solution to a problem I encountered. The solution is written as a test in JAVA.
## Problem
I have to compile data from multiple reactive services.
## Solution
I use the builder pattern mixed with flatMap's:

```java
package at.io_0.examples;

import lombok.Builder;
import org.apache.commons.lang3.StringUtils;
import org.junit.jupiter.api.Test;
import reactor.core.publisher.Mono;
import java.time.Duration;
import java.util.List;
import java.util.function.Function;
import static at.io_0.examples.Tests.Container.*;
import static org.assertj.core.api.Assertions.assertThat;

class Tests {
  @Test
  void testReactiveBuilder() {
    Mono<Container> result = Mono.just(Container.builder())
      .flatMap(task1())
      .flatMap(task2())
      .flatMap(task3("Sally"))
      .map(ContainerBuilder::build);

    assertThat(
      result
        .map(container -> StringUtils.join(List.of(container.a, container.b, container.c), " "))
        .block()
    ).isEqualTo("Hello there Sally");
  }

  @Builder
  static class Container {
    private final String a;
    private final String b;
    private final String c;
  }

  private Function<ContainerBuilder, Mono<ContainerBuilder>> task1() {
    return container -> serviceA.map(response -> container.a(response));
  }

  private Function<ContainerBuilder, Mono<ContainerBuilder>> task2() {
    return container -> serviceB.map(container::b); // same as task1, just shorter
  }

  private Function<ContainerBuilder, Mono<ContainerBuilder>> task3(String name) {
    return container -> serviceC(name).map(container::c);
  }

  // Simulate services
  Mono<String> serviceA = Mono.just("Hello").delayElement(Duration.ofMillis(2));
  Mono<String> serviceB = Mono.just("there").delayElement(Duration.ofMillis(5));
  Mono<String> serviceC(String name) { return Mono.just(name).delayElement(Duration.ofMillis(3)); }
}
```
