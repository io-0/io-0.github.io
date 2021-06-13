# Reactive Bookworm
This document shows a solution to a problem I encountered. The solution is written as a test in JAVA.
## Problem
I have to compile a small amount of data from a reactive pageinated source.
## Solution
I dot it in a recursive manner:

```java
package at.io_0.examples;

import lombok.AllArgsConstructor;
import org.apache.commons.lang3.StringUtils;
import org.junit.jupiter.api.Test;
import reactor.core.publisher.Mono;

import java.time.Duration;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.Objects;
import java.util.function.Function;
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
    return addAllWordsOfCurrentPageToContext(new AggregateWordsOnPagesContext(bookTitle))
      .expand(this::addAllWordsOfCurrentPageToContext) // mono recursion => flux
      .last()
      .map(ctx -> ctx.aggregatedWords);
  }

  private Mono<AggregateWordsOnPagesContext> addAllWordsOfCurrentPageToContext(AggregateWordsOnPagesContext ctx) {
    if (lastPageWasEmpty(ctx)) {
      return Mono.empty(); // recursion end
    }
    return serviceCallToGetPage(ctx)
      .map(addPageToContext(ctx));
  }

  private boolean lastPageWasEmpty(AggregateWordsOnPagesContext ctx) {
    return Objects.equals(ctx.wordsOnLastPage, 0);
  }

  private Function<String, AggregateWordsOnPagesContext> addPageToContext(AggregateWordsOnPagesContext ctx) {
    return page -> {
      List<Word> words = Arrays.stream(StringUtils.split(page))
        .map(Word::new)
        .collect(Collectors.toList());
      return new AggregateWordsOnPagesContext(ctx.bookTitle, union(ctx.aggregatedWords, words), ctx.currentPage + 1, words.size());
    };
  }

  @AllArgsConstructor
  static class AggregateWordsOnPagesContext {
    String bookTitle;
    List<Word> aggregatedWords = new ArrayList<>();
    int currentPage = 1;
    Integer wordsOnLastPage;

    public AggregateWordsOnPagesContext(String bookTitle) {
      this.bookTitle = bookTitle;
    }
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
  Mono<String> serviceCallToGetPage(AggregateWordsOnPagesContext ctx) {
    if (!ctx.bookTitle.equals("A tiny strange Book")) throw new IllegalStateException("wrong book");
    if (ctx.currentPage == 1) return Mono.just("This is").delayElement(Duration.ofMillis(3));
    if (ctx.currentPage == 2) return Mono.just("a very").delayElement(Duration.ofMillis(5));
    if (ctx.currentPage == 3) return Mono.just("short story.").delayElement(Duration.ofMillis(2));
    return Mono.just("").delayElement(Duration.ofMillis(1));
  }
}
```
