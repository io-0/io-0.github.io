# Reactive Bookworm
This document shows a solution to a problem I encountered. The solution is written as a test in JAVA.
## Problem
I have to compile a small amount of data from a reactive pageinated source.
## Solution
I flatMap in a recursive manner:

```java
package at.io_0.examples;

import lombok.*;
import org.apache.commons.lang3.StringUtils;
import org.junit.jupiter.api.Test;
import reactor.core.publisher.Mono;
import java.time.Duration;
import java.util.*;
import java.util.function.*;
import java.util.stream.Collectors;
import static org.assertj.core.api.Assertions.assertThat;

class Tests {
  @Test
  void testReactiveBookworm() {
    Mono<List<Word>> result = readAllPages("A tiny strange Book");

    assertThat(
      result
        .map(words -> StringUtils.join(words, " "))
        .block()
    ).isEqualTo("This is a very short story.");
  }

  Mono<List<Word>> readAllPages(String bookTitle) {
    return getAllWords(bookTitle)
      .apply(Mono.just(new AggregateWordsOnPagesContext()))
      .map(ctx -> ctx.aggregatedWords);
  }

  private UnaryOperator<Mono<AggregateWordsOnPagesContext>> getAllWords(String bookTitle) {
    return mono -> mono.flatMap(ctx -> {
      if (lastPageWasEmpty(ctx)) {
        return Mono.just(ctx);
      }

      return getAllWords(bookTitle)
        .apply(serviceCallToGetPage(bookTitle, ctx.currentPage).map(addPageToContext(ctx)));
    });
  }

  private boolean lastPageWasEmpty(AggregateWordsOnPagesContext ctx) {
    return Objects.equals(ctx.wordsOnLastPage, 0);
  }

  private Function<String, AggregateWordsOnPagesContext> addPageToContext(AggregateWordsOnPagesContext ctx) {
    return page -> {
      List<Word> words = Arrays.stream(StringUtils.split(page))
        .map(Word::new)
        .collect(Collectors.toList());

      return new AggregateWordsOnPagesContext(union(ctx.aggregatedWords, words), ctx.currentPage + 1, words.size());
    };
  }

  @NoArgsConstructor @AllArgsConstructor
  static class AggregateWordsOnPagesContext {
    List<Word> aggregatedWords = new ArrayList<>();
    int currentPage = 1;
    Integer wordsOnLastPage;
  }

  @AllArgsConstructor
  static class Word {
    private final String word;

    @Override
    public String toString() {
      return this.word;
    }
  }

  private static <T> List<T> union(List<T> a, List<T> b) {
    List<T> union = new ArrayList<>(a);
    union.addAll(b);
    return union;
  }

  // Simulate services
  Mono<String> serviceCallToGetPage(String bookTitle, int page) {
    if (page == 1) return Mono.just("This is").delayElement(Duration.ofMillis(3));
    if (page == 2) return Mono.just("a very").delayElement(Duration.ofMillis(5));
    if (page == 3) return Mono.just("short story.").delayElement(Duration.ofMillis(2));
    return Mono.just("").delayElement(Duration.ofMillis(1));
  }
}
```
