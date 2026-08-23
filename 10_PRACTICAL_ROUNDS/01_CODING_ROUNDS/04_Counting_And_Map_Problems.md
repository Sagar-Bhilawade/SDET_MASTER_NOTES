# Counting & HashMap Problems — The Bank

## What it is

A focused drill on the single most useful pattern in SDET coding rounds: **use a map to count, then answer a question about the counts.** Frequency, first-unique, anagrams, "appears exactly once," most-common — they're all the same shape. Master this shape and a whole class of problems collapses into a template.

---

## Why it matters

If you internalize one pattern for these rounds, make it this one. The moment a problem says "how many times," "most/least frequent," "duplicate," "unique," or "group by" — your hand should already be reaching for a HashMap. Interviewers reuse this shape constantly because it separates people who *recognize patterns* from people who reinvent a solution each time.

---

## The template (memorize this shape)

```java
Map<K, Integer> freq = new HashMap<>();
for (K item : items) freq.merge(item, 1, Integer::sum);   // count
// then: iterate freq to answer the question
```
```typescript
const freq = new Map<K, number>();
for (const item of items) freq.set(item, (freq.get(item) ?? 0) + 1);   // count
// then: iterate to answer
```
`merge(key, 1, Integer::sum)` is the clean Java idiom — "put 1, or add 1 to what's there." Know it; it beats the verbose `getOrDefault` version.

---

## 1. Frequency of each element
The template *is* the answer. `freq` holds it.

## 2. Most / least frequent
```java
K most = freq.entrySet().stream()
    .max(Map.Entry.comparingByValue()).map(Map.Entry::getKey).orElseThrow();
```
Least = `.min(...)`. *Edge: ties — ask which to return.*

## 3. First non-repeating (order matters)
Use `LinkedHashMap` to keep insertion order, or re-scan the original sequence. Return first with count 1.

## 4. Elements appearing exactly k times
```java
freq.entrySet().stream().filter(e -> e.getValue() == k)
    .map(Map.Entry::getKey).collect(Collectors.toList());
```

## 5. Are two strings anagrams
Count one up, count the other down, check all zero (see [String Problems](02_String_Problems.md) #6). Same template.

## 6. Find all duplicates
Filter the map for `value > 1`.

## 7. Check if array has all unique elements
```java
boolean allUnique = Arrays.stream(a).distinct().count() == a.length;
// or: build a Set, compare sizes
```

## 8. Group anagrams together
*"['eat','tea','tan','ate'] → [['eat','tea','ate'],['tan']]"* — the key is the **sorted** word:
```java
Map<String, List<String>> groups = new HashMap<>();
for (String w : words) {
    char[] c = w.toCharArray(); Arrays.sort(c);
    groups.computeIfAbsent(new String(c), k -> new ArrayList<>()).add(w);
}
```
*`computeIfAbsent` is the clean "get-or-create-the-list" idiom. This is the senior version of the counting pattern — the map value is a list, and the key is a derived signature.*

## 9. Two sum / pair with given sum
Map of `value → index`, check for the complement (see [Array Problems](03_Array_And_Collection_Problems.md) #1). "Remember what you've seen" is the counting pattern's cousin.

## 10. Count words in a sentence / log lines by type
```java
Map<String,Long> counts = Arrays.stream(text.split("\\s+"))
    .collect(Collectors.groupingBy(w -> w, Collectors.counting()));
```
*Real automation work — bucketing test statuses (pass/fail/skip), log severities. Name that connection; it makes the answer concrete.*

---

## The key Java map idioms (know all four)

| Idiom | Use |
|-------|-----|
| `map.merge(k, 1, Integer::sum)` | Count / increment |
| `map.getOrDefault(k, 0)` | Read with a default |
| `map.computeIfAbsent(k, x -> new ArrayList<>())` | Get-or-create a collection value |
| `map.putIfAbsent(k, v)` | Set only if not present |

TS equivalents: `map.set(k, (map.get(k) ?? 0) + 1)` for counting; `(obj[k] ??= []).push(v)` for get-or-create.

---

## The map choices that matter

- **`HashMap`** — O(1), no order. Default.
- **`LinkedHashMap`** — preserves insertion order. For "first unique," "keep order."
- **`TreeMap`** — sorted by key, O(log n). For "counts in sorted order."
- **`ConcurrentHashMap`** — thread-safe, for parallel test state (your ThreadLocal world).

*Choosing `LinkedHashMap` when order matters — instead of `HashMap` then wondering why output is scrambled — is a small thing that reads as "this person knows the library."*

---

## Common interview Q&A

**Q: "Can you do this without a map?"**
Sometimes — sorting first, or `int[26]` for lowercase letters (O(1) space, but only for a known small alphabet). Have the trade-off: map is general and O(n); the array trick is faster but assumes the character set.

**Q: "What's the complexity?"**
Counting is O(n) time, O(k) space where k = distinct elements. Finding max after is another O(k). State both.

**Q: "The input is a stream too big for memory."**
Great follow-up — you can't hold all counts. Mention approximate structures (a Count-Min Sketch) or that you'd process in chunks. You don't need to implement it; recognizing the constraint is the signal.

---

## Weak areas to watch

- **Reach for the map on the right trigger words** — frequency, unique, duplicate, group, most/least. Don't reinvent per problem.
- **Pick the right map for order.** `HashMap` scrambles; `LinkedHashMap` preserves. Getting this wrong silently produces "why is my output in random order?"
- **Learn `merge` and `computeIfAbsent`.** The verbose `if (map.containsKey...) else...` works but reads junior.
- **Second pass on the sequence, not the map**, when order matters (first-unique).
- **State O(n) time, O(k) space** unprompted.

---

## Key takeaway

> **"Count with a map, then answer from the counts" is the single highest-value pattern in these rounds — internalize the trigger words and the template.** Frequency, first-unique, anagrams, group-by all reduce to it. Know the four Java map idioms (`merge`, `getOrDefault`, `computeIfAbsent`, `putIfAbsent`) and pick `LinkedHashMap` the moment order matters.
