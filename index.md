---
title: A Title
---

# Welcome
Home
```java
public static String hello(String name) {
  // todo
}
```

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>