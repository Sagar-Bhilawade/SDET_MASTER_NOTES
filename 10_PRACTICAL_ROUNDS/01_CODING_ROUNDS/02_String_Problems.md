# String Problems — The Bank

## What it is

The most common category in SDET coding rounds. Strings show up because they're quick to state, easy to add edge cases to, and mirror real automation work (parsing logs, validating input, extracting data). Each problem below has the approach, both languages, complexity, and the edge cases to name. Run the [5-step method](01_Approach_And_Meta_Skill.md) on every one.

---

## Why it matters

Strings are where the "did you think about edge cases?" test lives — empty, whitespace, case, unicode, special characters. An interviewer asking a string problem is often *really* testing whether you enumerate inputs like a tester. Nail the edge cases out loud and you've answered the question behind the question.

---

## 1. Reverse a string

**Approach:** two-pointer swap, or built-in. Mention both.

**Java:**
```java
static String reverse(String s) {
    char[] c = s.toCharArray();
    int i = 0, j = c.length - 1;
    while (i < j) { char t = c[i]; c[i++] = c[j--]; t = c[j+1]; c[j+1] = t; }
    return new String(c);
}
// or simply: new StringBuilder(s).reverse().toString();
```
**TS:** `[...s].reverse().join("")`
*Edge: empty, single char, unicode (spread handles surrogate pairs; `split("")` doesn't).*

---

## 2. Reverse words in a sentence

**Java:** `String.join(" ", new StringBuilder... )` → simplest:
```java
static String reverseWords(String s) {
    String[] w = s.trim().split("\\s+");
    Collections.reverse(Arrays.asList(w));
    return String.join(" ", w);
}
```
**TS:** `s.trim().split(/\s+/).reverse().join(" ")`
*Edge: multiple spaces between words, leading/trailing whitespace, empty string. `\s+` collapses runs of spaces — call that out.*

---

## 3. Palindrome check

**Two-pointer (better than reversing — O(1) extra space):**
```java
static boolean isPalindrome(String s) {
    String c = s.toLowerCase().replaceAll("[^a-z0-9]", "");
    int i = 0, j = c.length() - 1;
    while (i < j) if (c.charAt(i++) != c.charAt(j--)) return false;
    return true;
}
```
**TS:**
```typescript
function isPalindrome(s: string): boolean {
  const c = s.toLowerCase().replace(/[^a-z0-9]/g, "");
  let i = 0, j = c.length - 1;
  while (i < j) if (c[i++] !== c[j--]) return false;
  return true;
}
```
*Edge: case ("Racecar"), punctuation ("A man, a plan..."), empty (usually true). Ask: "do I ignore case and non-alphanumerics?"*

---

## 4. First non-repeating character

**Two passes, O(n):** count into a map, then scan the string in order for the first count-1 char.
```java
static Character firstUnique(String s) {
    Map<Character,Integer> f = new LinkedHashMap<>();
    for (char c : s.toCharArray()) f.merge(c, 1, Integer::sum);
    for (char c : s.toCharArray()) if (f.get(c) == 1) return c;
    return null;
}
```
**TS:**
```typescript
function firstUnique(s: string): string | null {
  const f = new Map<string, number>();
  for (const c of s) f.set(c, (f.get(c) ?? 0) + 1);
  for (const c of s) if (f.get(c) === 1) return c;
  return null;
}
```
*Edge: none unique → null; empty → null. Scan the **string** (not the map) in the second pass to preserve order.*

---

## 5. Most frequent character

```java
static char mostFrequent(String s) {
    Map<Character,Integer> f = new HashMap<>();
    for (char c : s.toCharArray()) f.merge(c, 1, Integer::sum);
    return f.entrySet().stream()
        .max(Map.Entry.comparingByValue())
        .map(Map.Entry::getKey)
        .orElseThrow(() -> new IllegalArgumentException("empty"));
}
```
**TS:** `[...freq.entries()].reduce((a,b) => b[1] > a[1] ? b : a)[0]`
*Edge: ties (which to return? ask), spaces counted?, empty → error or null.*

---

## 6. Anagram check

**Count-compare (O(n)) beats sort-compare (O(n log n)) — say the trade-off.**
```java
static boolean isAnagram(String a, String b) {
    if (a.length() != b.length()) return false;
    int[] count = new int[26];
    for (int i = 0; i < a.length(); i++) { count[a.charAt(i)-'a']++; count[b.charAt(i)-'a']--; }
    for (int c : count) if (c != 0) return false;
    return true;
}
```
**TS:** sort both: `[...a].sort().join("") === [...b].sort().join("")` (simpler; mention count-map for large input).
*Edge: different lengths (early return), case, spaces ("listen"/"silent"). The `int[26]` trick only works for lowercase a-z — say so; use a map for general unicode.*

---

## 7. Count occurrences of each word / character

```java
Map<String,Long> counts = Arrays.stream(sentence.split("\\s+"))
    .collect(Collectors.groupingBy(w -> w, Collectors.counting()));
```
**TS:**
```typescript
const counts = new Map<string, number>();
for (const w of sentence.split(/\s+/)) counts.set(w, (counts.get(w) ?? 0) + 1);
```
*This is real automation work — counting log lines, statuses. Mention that connection.*

---

## 8. Remove duplicate characters (preserve order)

**Java:** `s.chars().distinct().mapToObj(c -> String.valueOf((char)c)).collect(Collectors.joining());`
**TS:** `[...new Set(s)].join("")`
*Edge: `Set` preserves insertion order in both JS and Java's `LinkedHashSet`; plain `HashSet` doesn't. Know the difference.*

---

## 9. Check if two strings are rotations

*"Is 'erbottlewat' a rotation of 'waterbottle'?"* — Trick: a rotation is always a substring of the doubled original.
```java
static boolean isRotation(String a, String b) {
    return a.length() == b.length() && (a + a).contains(b);
}
```
*Edge: lengths must match first (empty strings, different sizes). The `a+a` insight is the "aha" they're looking for.*

---

## 10. String compression (run-length)

*"aaabbc" → "a3b2c1"* (or return original if compressed isn't shorter).
```java
static String compress(String s) {
    if (s.isEmpty()) return s;
    StringBuilder sb = new StringBuilder();
    int count = 1;
    for (int i = 1; i <= s.length(); i++) {
        if (i < s.length() && s.charAt(i) == s.charAt(i-1)) count++;
        else { sb.append(s.charAt(i-1)).append(count); count = 1; }
    }
    return sb.length() < s.length() ? sb.toString() : s;
}
```
*Edge: empty, single char, no repeats (compressed is longer → return original). Use `StringBuilder`, not `+=` in a loop — O(n²) otherwise; mention it.*

---

## The string gotchas interviewers probe

| Gotcha | The catch |
|--------|-----------|
| `+=` in a loop (Java) | Creates a new String each time → O(n²). Use `StringBuilder`. |
| `==` vs `.equals()` (Java) | `==` compares references, `.equals()` compares value. Classic trap. |
| `split("")` vs spread (TS) | `split("")` breaks unicode surrogate pairs; `[...s]` is safe. |
| Immutability | Strings are immutable in both — every "modification" is a new string. |
| `substring` bounds | Off-by-one and `StringIndexOutOfBounds`; check indices. |

---

## Key takeaway

> **String problems are edge-case tests in disguise — empty, whitespace, case, unicode.** Nail those out loud and you've shown the tester instinct that's the whole point. Know the two-pointer palindrome, the count-map for frequency/anagram (with the O(n) vs O(n log n) trade-off), and never build a string with `+=` in a loop.
