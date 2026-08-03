# Java - Collections & Streams for Test Automation

## What This Covers

The collections you actually use in a framework, and Streams for processing test data and validation results.

Collections questions are near-guaranteed in a Java interview, and Streams increasingly so.

---

## Why It Matters (Interview Context)

Two levels:

1. **The standard question** — "difference between ArrayList and LinkedList", "HashMap vs HashTable". Table stakes.
2. **The applied question** — "you have a list of WebElements, get all their texts and check one contains X". This is where Streams show you write modern Java rather than 2010 Java.

The applied version comes up as a live coding warm-up quite often.

---

## Your 30-Second Interview Answer

> "ArrayList for ordered data with index access — which is most test data. HashMap for keyed lookups like config or a row keyed by ID. HashSet when I need uniqueness, typically deduplicating results. And ConcurrentHashMap when the framework is running parallel, because a plain HashMap will corrupt under concurrent writes. Streams are what I use for pulling values out of WebElement lists and filtering — it replaces a lot of loop boilerplate and reads far closer to intent."

---

## Your 2-Minute Deep Answer

> "**The collections that matter in a framework:**
>
> **`List` / `ArrayList`** — ordered, indexed, allows duplicates. Test data rows, WebElement collections from `findElements`.
>
> ```java
> List<WebElement> rows = driver.findElements(By.cssSelector("table tr"));
> ```
>
> **`ArrayList` vs `LinkedList`:** ArrayList is a backing array — O(1) random access, but inserting in the middle shifts elements. LinkedList is a doubly-linked list — O(1) insert/remove at a known position, but O(n) to reach an index. In practice ArrayList is almost always right for test code, because you're iterating and indexing, not inserting mid-list.
>
> **`Map` / `HashMap`** — key-value. Config, test data keyed by identifier, API response fields.
>
> ```java
> Map<String, String> config = new HashMap<>();
> ```
>
> **`HashMap` vs `Hashtable`:** HashMap allows one null key and null values and isn't synchronized; Hashtable is synchronized and allows neither. Hashtable is legacy — if you need thread safety, `ConcurrentHashMap` is the modern answer and performs far better because it locks segments rather than the whole map.
>
> **`ConcurrentHashMap`** matters specifically for parallel test execution. A plain HashMap written to by multiple threads can corrupt its internal structure — historically it could even cause an infinite loop during resize. If your framework keys anything by thread or test, use the concurrent version.
>
> **`Set` / `HashSet`** — unique elements, no order. Deduplicating, checking membership.
>
> `LinkedHashMap` and `LinkedHashSet` preserve insertion order, which matters when test data order is meaningful. `TreeMap` keeps keys sorted.
>
> ---
>
> **Streams — where they earn their place in test code:**
>
> ```java
> // Extract all row texts
> List<String> names = driver.findElements(By.cssSelector(".name"))
>     .stream()
>     .map(WebElement::getText)
>     .collect(Collectors.toList());
>
> // Is a value present?
> boolean found = names.stream().anyMatch(n -> n.equals("Sagar"));
>
> // Filter and count
> long errorCount = messages.stream()
>     .filter(m -> m.startsWith("ERROR"))
>     .count();
>
> // Find first match
> Optional<WebElement> row = rows.stream()
>     .filter(r -> r.getText().contains("Sagar"))
>     .findFirst();
>
> // Group test results by status
> Map<String, List<TestResult>> byStatus = results.stream()
>     .collect(Collectors.groupingBy(TestResult::getStatus));
>
> // Comma-joined string
> String joined = names.stream().collect(Collectors.joining(", "));
> ```
>
> The `map(WebElement::getText)` pattern is the one that comes up most — extracting text from a list of elements. It replaces a for-loop with an accumulator and reads much closer to what you mean.
>
> **Careful with Streams over WebElements though.** Each `getText()` is an HTTP round trip. A stream over two hundred elements is two hundred round trips — the syntax hides the cost. That's worth knowing, because it looks efficient and isn't.
>
> ---
>
> **Terminal vs intermediate:**
>
> `map`, `filter`, `sorted`, `distinct` are **intermediate** — lazy, nothing executes.
> `collect`, `forEach`, `count`, `anyMatch`, `findFirst` are **terminal** — they trigger the pipeline.
>
> A stream with only intermediate operations does nothing at all. That's a real bug and a fair interview question.
>
> ---
>
> **`Optional`** — worth knowing since `findFirst` returns one:
>
> ```java
> Optional<WebElement> row = rows.stream().filter(...).findFirst();
>
> row.ifPresent(WebElement::click);
> WebElement r = row.orElseThrow(() -> new AssertionError("Row not found"));
> ```
>
> The `orElseThrow` with a meaningful message is the pattern I'd use in tests — it turns a missing element into a clear assertion failure rather than a `NoSuchElementException` from the Optional itself.
>
> ---
>
> **The live-coding style question:**
>
> A common warm-up is something like *"given a list of strings, return the duplicates"*:
>
> ```java
> Set<String> seen = new HashSet<>();
> Set<String> duplicates = list.stream()
>     .filter(s -> !seen.add(s))       // add returns false if already present
>     .collect(Collectors.toSet());
> ```
>
> `Set.add` returning false on a duplicate is the trick worth remembering — it makes this a one-liner instead of a nested loop."

---

## Collections Reference

| Type | Ordered? | Duplicates? | Use for |
|------|:---:|:---:|---------|
| `ArrayList` | ✅ | ✅ | Test data, WebElement lists |
| `LinkedList` | ✅ | ✅ | Rarely — frequent mid-list inserts |
| `HashMap` | ❌ | Keys unique | Config, keyed data |
| `LinkedHashMap` | ✅ | Keys unique | Order-sensitive data |
| `TreeMap` | Sorted | Keys unique | Sorted keys |
| `HashSet` | ❌ | ❌ | Deduplication, membership |
| `ConcurrentHashMap` | ❌ | Keys unique | **Parallel execution** |

---

## Your Real Project Example (NielsenIQ)

**How to frame it:**

> "The most common use was pulling values out of grids — `findElements` gives you a list of WebElements, and `stream().map(WebElement::getText)` turns that into the list of strings you actually want to assert on. That replaced a lot of loop-and-accumulate boilerplate and reads much closer to intent.
>
> `ConcurrentHashMap` mattered because we ran parallel. Anything the framework keyed by thread or test — driver references, report objects, retry counts — has to be a concurrent map. A plain HashMap under concurrent writes can corrupt its internal structure, and the resulting failures look like anything except a collections problem.
>
> The thing I'd flag about streams over WebElements is that the syntax hides cost. Every `getText()` is an HTTP round trip to the driver, so a stream over two hundred rows is two hundred round trips. It looks efficient and isn't. On large grids we'd sometimes pull the data via JavaScript in a single call instead, or validate at the database level rather than scraping the UI — which was usually the better answer anyway."

---

## Common Interview Questions & Answers

### Q1: "ArrayList vs LinkedList?"

**A:** "ArrayList is array-backed — O(1) random access, but mid-list insertion shifts elements. LinkedList is doubly-linked — cheap insert and remove at a known position, but O(n) to reach an index. For test code ArrayList is almost always right, because you're iterating and indexing rather than inserting into the middle."

### Q2: "HashMap vs Hashtable vs ConcurrentHashMap?"

**A:** "HashMap isn't synchronized and permits a null key and null values. Hashtable is synchronized and permits neither — it's legacy. ConcurrentHashMap is the modern thread-safe option and performs much better because it locks segments rather than the whole map. For a parallel test framework, anything shared across threads needs ConcurrentHashMap."

### Q3: "Get all row texts from a table and check one contains 'Sagar'."

**A:**
```java
boolean found = driver.findElements(By.cssSelector("table td"))
    .stream()
    .map(WebElement::getText)
    .anyMatch(t -> t.contains("Sagar"));
```
"Though on a large table I'd note each `getText()` is a round trip to the driver, so this is more expensive than it looks."

### Q4: "What's the difference between intermediate and terminal stream operations?"

**A:** "Intermediate operations — map, filter, sorted — are lazy and return a stream; nothing executes. Terminal operations — collect, count, anyMatch, forEach — trigger the pipeline. A stream with only intermediate operations does literally nothing, which is a real bug people hit."

### Q5: "Find duplicates in a list."

**A:**
```java
Set<String> seen = new HashSet<>();
Set<String> dupes = list.stream()
    .filter(s -> !seen.add(s))
    .collect(Collectors.toSet());
```
"`Set.add` returns false when the element is already present, which makes the filter a one-liner."

---

## Weak Areas to Watch

❌ **Don't use plain HashMap in a parallel framework**
→ Concurrent writes can corrupt it.

❌ **Don't forget the terminal operation**
→ Intermediate-only streams do nothing.

❌ **Don't ignore the cost of streams over WebElements**
→ Every accessor is an HTTP round trip.

❌ **Don't cite Hashtable as the thread-safe option**
→ It's legacy. ConcurrentHashMap.

---

## Key Takeaway for Interview

Collections questions are guaranteed; Streams are where you show current Java.

The pattern to have ready:

> **`findElements(...).stream().map(WebElement::getText).anyMatch(...)`** — the standard grid-validation idiom.

And the detail that shows you understand the cost:

> **"Every `getText()` in that stream is an HTTP round trip to the driver. Over two hundred rows that's two hundred round trips — the syntax hides it. On large grids I'd validate at the database instead of scraping the UI."**

---

## Checklist

✅ ArrayList default; know when LinkedList differs
✅ ConcurrentHashMap wherever the framework is parallel
✅ Comfortable with map / filter / collect / anyMatch
✅ Know intermediate vs terminal
✅ `Optional.orElseThrow` with a clear message
✅ Duplicate-finding one-liner ready
✅ Aware of round-trip cost in element streams

---

**Remember:** Elegant stream syntax over WebElements can still be two hundred network calls. 🎯
