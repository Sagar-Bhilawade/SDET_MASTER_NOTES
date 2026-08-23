# Java Idioms & Gotchas — Coding Round Reference

## What it is

The Java-specific things that come up in coding rounds and that interviewers use as quick depth probes: `==` vs `.equals()`, streams, autoboxing traps, the collection idioms. This is your legacy-stack language, so a Java-shop interviewer will expect fluency here. This file is the "don't get caught" reference.

---

## Why it matters

Coding rounds test the language *and* the algorithm. A correct algorithm with a rookie Java mistake — comparing strings with `==`, an NPE from unboxing a null `Integer` — undercuts the whole answer, because the interviewer now wonders what else you don't know. These gotchas are cheap to master and expensive to trip on.

---

## The gotchas that get people

### `==` vs `.equals()`
```java
String a = new String("hi"), b = new String("hi");
a == b        // false — different objects (reference comparison)
a.equals(b)   // true  — same value
```
`==` compares references; `.equals()` compares value. For objects, almost always use `.equals()`. The string-literal pool makes `"hi" == "hi"` true, which lulls people into the bug. **The single most common Java interview trap.**

### Autoboxing & the `Integer` cache
```java
Integer x = 127, y = 127;   x == y   // true  (cached -128..127)
Integer p = 128, q = 128;   p == q   // false (new objects!)
```
Java caches small `Integer` values, so `==` accidentally works for small numbers and breaks for large ones. Use `.equals()` for `Integer`. And unboxing a `null` Integer throws NPE:
```java
Map<String,Integer> m = new HashMap<>();
int v = m.get("missing");   // NPE — null unboxed to int
```
Use `getOrDefault("missing", 0)`.

### Integer division
`5 / 2 == 2` (truncates). For a real result: `(double) 5 / 2 == 2.5`. Bites in averages and percentages.

### Array vs List
`int[]` is fixed-size, no helpful methods. `List<Integer>` resizes and has the API. `Arrays.asList(arr)` on a primitive `int[]` gives a `List<int[]>` of size 1 (gotcha!) — box to `Integer[]` first, or use streams.

---

## The stream idioms worth knowing cold

```java
// filter + map + collect
List<String> names = people.stream()
    .filter(p -> p.getAge() > 18)
    .map(Person::getName)
    .collect(Collectors.toList());

// count occurrences (group + count)
Map<String,Long> counts = items.stream()
    .collect(Collectors.groupingBy(i -> i.getType(), Collectors.counting()));

// sum / average
int total = nums.stream().mapToInt(Integer::intValue).sum();
OptionalDouble avg = nums.stream().mapToInt(Integer::intValue).average();

// max by a field
Optional<Order> biggest = orders.stream().max(Comparator.comparing(Order::getAmount));

// join to a string
String csv = list.stream().collect(Collectors.joining(", "));

// distinct / sorted / limit
list.stream().distinct().sorted().limit(5).collect(Collectors.toList());
```
*Streams are declarative and read well — but they have overhead. For a hot loop over millions, a plain `for` is faster. Know the trade-off: "streams for readability, loops when profiling says so."*

---

## The map idioms (from counting problems)

```java
map.merge(key, 1, Integer::sum);                       // count
map.getOrDefault(key, 0);                               // safe read
map.computeIfAbsent(key, k -> new ArrayList<>()).add(v); // get-or-create list
map.putIfAbsent(key, value);                            // set once
```

## Collection cheat sheet

| Type | Ordered? | Dupes? | Use |
|------|:--------:|:------:|-----|
| `ArrayList` | insertion | yes | default list, index access |
| `HashSet` | no | no | dedup, fast contains |
| `LinkedHashSet` | insertion | no | dedup + keep order |
| `TreeSet` | sorted | no | sorted unique |
| `HashMap` | no | keys unique | default map |
| `LinkedHashMap` | insertion | keys unique | order-preserving map |
| `TreeMap` | sorted keys | keys unique | sorted map |
| `ConcurrentHashMap` | no | keys unique | **thread-safe (parallel tests)** |

---

## Java 8+ things they probe

- **Functional interfaces & lambdas:** `Runnable`, `Comparator`, `Predicate`, `Function`. **Why `ExpectedCondition` (Selenium) takes a lambda** — it's a functional interface (one abstract method), so a lambda satisfies it. This ties your Selenium work to Java 8. See [Java 8 Features](../../03_SELENIUM_JAVA/03_JAVA_FOR_SDET/03_Java_8_Features.md).
- **`Optional`:** avoid NPEs — `.map()`, `.orElse()`, `.orElseThrow()`. Don't just call `.get()` (defeats the purpose).
- **Method references:** `Person::getName` is shorthand for `p -> p.getName()`.
- **`var` (Java 10+):** local type inference — `var list = new ArrayList<String>();`.

---

## Common interview Q&A

**Q: "Why is `a == b` false for two equal strings?"**
Reference vs value comparison. `new String()` creates distinct objects; use `.equals()`.

**Q: "What's wrong with `int v = map.get(key)`?"**
NPE if the key is missing — null unboxed to int. Use `getOrDefault`.

**Q: "Stream or loop?"**
Stream for readability on transformations; loop for hot paths where overhead matters. Not dogmatic either way.

**Q: "HashMap vs ConcurrentHashMap?"**
HashMap isn't thread-safe — concurrent writes can corrupt it or infinite-loop (older JDKs). ConcurrentHashMap for shared state across threads — directly relevant to your parallel-test framework.

---

## Weak areas to watch

- **`.equals()` for objects, always** — `==` only for primitives and null checks.
- **`getOrDefault` / `Optional`** to dodge NPEs — never blind `.get()`.
- **Know the Integer cache** — it's a favourite "gotcha" question.
- **`StringBuilder` in loops**, never `+=` (O(n²)).
- **Streams have a cost** — have the readability-vs-performance answer ready.
- **`ConcurrentHashMap` links to your ThreadLocal story** — mention it, it shows the concepts connect.

---

## Key takeaway

> **A correct algorithm with `==` on strings or an NPE from unboxing undercuts the whole answer — these gotchas are cheap to master and expensive to trip.** Know `.equals()` vs `==`, the Integer cache, `getOrDefault`, the stream and map idioms, and connect `ConcurrentHashMap` and lambda-based `ExpectedCondition` back to your real Selenium framework.
