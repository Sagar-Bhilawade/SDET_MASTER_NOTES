# Live Coding — Approach & the Meta-Skill

## What it is

How to *run* a live-coding round — the shared-screen, 20-30-minute problem. For an SDET this is rarely hard DSA; it's string/array/collection manipulation and logic you narrate cleanly. This note is the process that wins the round regardless of the specific problem. The actual problems live in the sibling files: [strings](02_String_Problems.md), [arrays/collections](03_Array_And_Collection_Problems.md), [counting/maps](04_Counting_And_Map_Problems.md), [logic/numbers](05_Logic_And_Number_Problems.md).

---

## Why it matters

You cannot bluff this round — the interviewer watches you think. And here's the thing most candidates miss: **they're not grading whether you reach the optimal solution. They're grading how you work.** Can you decompose a problem, handle edge cases, catch your own bugs, and communicate while coding? A candidate who talks through a slightly sub-optimal solution and tests it beats one who silently writes perfect code, because testing is fundamentally a communication job — and you're the *testing* hire.

---

## The 5-step method (run this every single time)

### 1. Restate the problem
> "So I need to take a string and return the first character that doesn't repeat — is that right? And by 'repeat' you mean appears more than once anywhere in the string?"

Catches a misunderstanding *before* you burn 10 minutes solving the wrong thing. Costs 15 seconds, saves the round.

### 2. Ask about edge cases up front
This is **the SDET signal** — it's literally your job to think about inputs. Ask before coding:
- Empty input? Null?
- Case sensitivity? ("Is 'A' the same as 'a'?")
- Duplicates? Whitespace? Special characters?
- Size — could it be huge? (hints at complexity constraints)
- What to return when there's no answer? (null, -1, throw, empty?)

A pure developer starts coding. You ask about inputs first. That contrast is visible and it's worth points.

### 3. State your approach out loud before coding
> "I'll use a HashMap to count each character in one pass, then scan the string again and return the first char with a count of one. That's O(n) time, O(n) space."

Lets the interviewer redirect you cheaply if you're heading the wrong way. If they nod, you're safe to code. If they say "can you do it without extra space?", you just saved yourself from coding the wrong solution.

### 4. Write it, narrating as you go
Talk while you type. "Now I'll loop through the characters... using `merge` to increment the count..." A wrong turn caught out loud is recoverable; silent floundering isn't. Silence also reads as "I don't know what I'm doing" even when you do.

### 5. Test your own solution — THIS is where you win
When the code compiles, don't say "done." Say:
> "Let me trace an example through it." — walk a real input line by line, out loud.
> "Now let me check the edge cases I asked about: empty string returns null, all-duplicates returns null, single char returns that char."

Then name the tests you'd write:
> "As tests I'd cover: happy path, empty, single character, all-same, and one with mixed case."

**No pure developer does this instinctively. You do, because it's your discipline.** This single habit is why an SDET out-scores a stronger coder in this round.

---

## Complexity — state it unprompted

Always say the time and space complexity without being asked:
> "This is O(n) time and O(n) space for the map."

If they ask "can you do better?", you usually can't beat O(n) time for these, but you can sometimes cut space (two-pointer, in-place). Have the trade-off ready: *"I can drop to O(1) space with a two-pointer, but the map version is more readable — which matters more here?"* Asking that question is itself senior.

Quick reference — the complexities you'll cite:
| Operation | Complexity |
|-----------|-----------|
| HashMap get/put | O(1) average |
| Sorting | O(n log n) |
| Single loop | O(n) |
| Nested loop | O(n²) |
| Binary search (sorted) | O(log n) |

---

## Practice discipline

- **Practice on a plain editor** (Notepad, an online pad) — the round has no autocomplete, no red squiggles. If you only ever code in a full IDE, you'll fumble syntax under pressure.
- **Drill both languages.** Java (legacy stack) and TypeScript/JS (modern stack). Mixing Java syntax into a TS answer under pressure reads as shallow — see the [Java idioms](06_Java_Idioms_And_Gotchas.md) and [TypeScript idioms](07_TypeScript_Idioms_And_Gotchas.md) files.
- **Time yourself.** Target: understand + approach in 5 min, code in 10, test in 5.
- **Say it out loud when you practice.** The narration is a skill; it doesn't appear for free on the day.

---

## Common interview Q&A

**Q: "Just solve it however you like."**
Still run the method. Restate, ask edge cases, state approach, narrate, test. The freedom is a trap — the structure is what's graded.

**Q: "You have a bug."**
Don't panic-edit. "Let me trace through where it diverges." Walk it calmly. Finding your own bug methodically scores *higher* than never having one — it's exactly the debugging skill they're hiring.

**Q: "How would you test this function?"**
The gift question for an SDET. Happy path, empty, null, boundaries, one malformed input. This is where you pull ahead — answer it thoroughly even if unasked.

---

## Weak areas to watch

- **Don't code in silence.** The narration *is* the assessment.
- **Don't skip the edge-case questions** — for an SDET that's the whole differentiator.
- **Don't over-engineer.** Clean and correct in 25 minutes beats a design-pattern showcase.
- **Don't forget to test your own code.** The single most common missed opportunity. You're the testing candidate — act like it.
- **Don't stay stuck silently.** If blocked, think out loud: "The brute force is O(n²)... can I trade space for time with a map?" They'll often nudge you.

---

## Key takeaway

> **They grade how you work, not whether you hit optimal — so restate, ask edge cases, state complexity, narrate, and above all test your own solution out loud.** That last habit is why an SDET beats a stronger pure coder in this round: enumerating inputs and verifying outputs is your literal job. Run the 5-step method every time, even when told "just solve it."
