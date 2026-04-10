---
title: "Java"
date: 2026-01-14
tags:
  - engineering
  - java
---

[[Generics]]

```Java

@Test
public void testReference() {
    Map<String, String> map = new HashMap<>();
    Foo foo = new Foo(map);

    map.put("foo","bar");
assertEquals("bar",foo.getVal("foo"));
}

public static class Foo {
    private final Map<String, String> map;

    public Foo(Map<String, String> map) {
        this.map = map;
    }

    public String getVal(String k) {
        return map.get(k);
    }
}
```

  

[[Reflection]]