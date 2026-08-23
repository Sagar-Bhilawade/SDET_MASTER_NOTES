# Array & Collection Problems — The Bank

## What it is

The second big category: arrays and lists — searching, deduping, finding pairs, rearranging. These test whether you know when a HashMap turns an O(n²) brute force into O(n), and whether you handle the boundaries (empty, one element, all-same). Run the [5-step method](01_Approach_And_Meta_Skill.md).

---

## Why it matters

Array problems are where the "can you improve your first solution?" conversation happens. Almost always the brute force is a nested loop, and the improvement is a HashMap or a sort. Showing the brute force first, then improving it *out loud*, demonstrates range — you see the naive answer AND the better one. That progression is often worth more than jumping straight to optimal.

---

## 1. Two Sum (the classic)

*"Return indices of two numbers that add to target."*

**Brute force (state it, then improve):** nested loop, O(n²).
**Optimal — one pass with a map, O(n):**
```java
static int[] twoSum(int[] nums, int target) {
    Map<Integer,Integer> seen = new HashMap<>();       // value -> index
    for (int i = 0; i < nums.length; i++) {
        int need = target - nums[i];
        if (seen.containsKey(need)) return new int[]{seen.get(need), i};
        seen.put(nums[i], i);
    }
    return new int[]{-1, -1};
}
```
**TS:**
```typescript
function twoSum(nums: number[], target: number): [number, number] {
  const seen = new Map<number, number>();
  for (let i = 0; i < nums.length; i++) {
    const need = target - nums[i];
    if (seen.has(need)) return [seen.get(need)!, i];
    seen.set(nums[i], i);
  }
  return [-1, -1];
}
```
*The insight: "instead of searching for the pair, I remember what I've seen and check if the complement already exists." Edge: no pair, duplicates, negatives.*

---

## 2. Find duplicates in an array

**Set-based, O(n):**
```java
static Set<Integer> duplicates(int[] a) {
    Set<Integer> seen = new HashSet<>(), dupes = new HashSet<>();
    for (int x : a) if (!seen.add(x)) dupes.add(x);   // add() returns false if already present
    return dupes;
}
```
**TS:**
```typescript
function duplicates(a: number[]): number[] {
  const seen = new Set<number>(), dupes = new Set<number>();
  for (const x of a) seen.has(x) ? dupes.add(x) : seen.add(x);
  return [...dupes];
}
```
*The `set.add()` returning false trick (Java) is clean — mention it. Edge: no dupes, all dupes.*

---

## 3. Remove duplicates, preserve order

**Java:** `new ArrayList<>(new LinkedHashSet<>(list))`
**TS:** `[...new Set(arr)]`
*`LinkedHashSet`/`Set` preserve insertion order; `HashSet` scrambles it. This distinction is frequently the whole question.*

---

## 4. Find the missing number (1..n)

**Sum formula, O(n) time O(1) space:**
```java
static int missing(int[] a, int n) {
    int expected = n * (n + 1) / 2;
    int actual = Arrays.stream(a).sum();
    return expected - actual;
}
```
*Elegant — no extra space. Alternative: XOR all indices and values. Edge: watch integer overflow for large n (use long for the sum).*

---

## 5. Max / min / second-largest

```java
static int secondLargest(int[] a) {
    int max = Integer.MIN_VALUE, second = Integer.MIN_VALUE;
    for (int x : a) {
        if (x > max) { second = max; max = x; }
        else if (x > second && x != max) second = x;
    }
    return second;
}
```
*One pass, no sorting. Edge: array of size < 2, all-equal elements (`x != max` guard). Sorting works but is O(n log n) — mention the one-pass is better.*

---

## 6. Move zeros to the end (in-place)

```java
static void moveZeros(int[] a) {
    int pos = 0;
    for (int x : a) if (x != 0) a[pos++] = x;
    while (pos < a.length) a[pos++] = 0;
}
```
*Two-pointer, in-place, O(1) space, preserves order of non-zeros. A common "do it in-place" follow-up.*

---

## 7. Merge two sorted arrays

**Two-pointer, O(n+m):**
```java
static int[] merge(int[] a, int[] b) {
    int[] r = new int[a.length + b.length];
    int i = 0, j = 0, k = 0;
    while (i < a.length && j < b.length) r[k++] = a[i] <= b[j] ? a[i++] : b[j++];
    while (i < a.length) r[k++] = a[i++];
    while (j < b.length) r[k++] = b[j++];
    return r;
}
```
*Edge: one empty, different lengths, duplicates across arrays.*

---

## 8. Group / bucket by a property (real SDET work)

*"Group orders by status."* — Java Streams shine here:
```java
Map<String, List<Order>> byStatus =
    orders.stream().collect(Collectors.groupingBy(Order::getStatus));
```
**TS:**
```typescript
const byStatus = orders.reduce((acc, o) => {
  (acc[o.status] ??= []).push(o);
  return acc;
}, {} as Record<string, Order[]>);
```
*This is exactly what you do grouping test results/log lines — say so. Know `groupingBy` cold.*

---

## 9. Intersection / union of two lists

**Intersection:** `a.stream().filter(bSet::contains).collect(toList())`
**Union:** `Stream.concat(a.stream(), b.stream()).distinct().collect(toList())`
**TS intersection:** `a.filter(x => new Set(b).has(x))` (build the Set once outside for O(n+m)).
*Common gotcha: rebuilding the Set inside the filter makes it O(n·m) — build it once.*

---

## 10. Rotate an array by k

```java
static void rotate(int[] a, int k) {
    k %= a.length;                 // handle k > length
    reverse(a, 0, a.length - 1);
    reverse(a, 0, k - 1);
    reverse(a, k, a.length - 1);
}
```
*The triple-reverse trick, O(1) space. Edge: `k > length` (mod it), `k = 0`, empty. The naive approach uses an extra array — mention both.*

---

## The collection choices they probe

| Need | Java | TS/JS | Why |
|------|------|-------|-----|
| Fast lookup by key | `HashMap` | `Map`/object | O(1) average |
| Unique values | `HashSet` | `Set` | dedup |
| Unique + insertion order | `LinkedHashSet` | `Set` (ordered by default) | preserve order |
| Sorted keys | `TreeMap` | — (sort manually) | O(log n) ordered |
| Thread-safe (parallel tests!) | `ConcurrentHashMap` | — | your ThreadLocal angle |
| Ordered list | `ArrayList` | `Array` | index access |

*The parallel-testing angle: `ConcurrentHashMap` ties straight to your ThreadLocal driver work — see [Collections & Streams](../../03_SELENIUM_JAVA/03_JAVA_FOR_SDET/02_Collections_And_Streams.md).*

---

## Key takeaway

> **The pattern is almost always "nested loop O(n²) → HashMap or sort → O(n)".** State the brute force, then improve it out loud — the progression shows range. Know which collection buys you what (Set for dedup, Map for lookup, LinkedHashSet for order), and reach for `groupingBy` on any "group by" question since it's your real automation work.
