# Software Testing: Practice Exercises (with answers)

Based on Dr Chris Meudec's material: *Introduction to Software Testing*, *AI-Assisted Black Box Testing 1 & 2* and *Testing Theory and Practice 101*.

## What your lecturer cares about (read this first)

Going through the four documents, the same ideas come up again and again:

1. **A test without an expected outcome is not a test.** A test case is inputs **plus the expected behaviour taken from the specification**. "I ran it and it didn't crash" is worthless, even dangerous.
2. **The oracle problem.** Running tests is easy; deciding what the *correct* answer is, is the hard part. Never use the code as its own oracle (the "tautological test").
3. **The specification is where most bugs live** (~55% in Waterfall). Finding ambiguities, omissions and contradictions in a spec *is* testing. "An underdetermined spec is a finding, not an obstacle."
4. **Verification vs validation**: "building the product right" vs "building the right product".
5. **Static vs dynamic verification**: code reviews and static analysis vs running tests. Use both.
6. **Black box and white box are complementary.** There's no winner.
7. **Coverage is necessary but not sufficient.** 100% statement coverage "sounds good" but proves very little.
8. **Equivalence partitioning + boundary values**, done *systematically in a spreadsheet* with his exact column layout, including **invalid** partitions (robustness) and **output** partitions.
9. **AI is useful but must be reviewed.** LLMs partition what the spec mentions; they don't ask what it *doesn't* mention, and they silently resolve ambiguity. That's your job.
10. He loves **"Why?"** questions, real-world bug stories, and "what would you do about it?"

His exercises are usually: *derive partitions/test cases in a Google Sheet with a set layout*, *produce a specification defects list*, *critique what an LLM produced*, or *answer a short "why" question in your own words*. The exercises below follow those formats.

Try each one before opening the answer.

---

## Part A: Core concepts

### A1. Verification or validation?

For each activity, say whether it is mainly **verification** or **validation**, and why.

1. Running JUnit tests on a `calculateInterest()` method.
2. Showing a working prototype to the client at the end of a sprint.
3. Acceptance testing using tests supplied by the client.
4. A code review of a pull request.
5. Beta testing with 500 real users before release.
6. Running SonarQube on the codebase.
7. System testing the whole app through its API.
8. Checking that "a novice user can complete a booking in under 2 minutes".

<details><summary>Answer</summary>

| # | Activity | V&V | Why |
|---|---|---|---|
| 1 | JUnit on a method | Verification | Checks the code behaves as specified (unit testing). |
| 2 | Prototype demo to client | Validation | Asks the client "is this what you need?" |
| 3 | Acceptance testing | Validation | Client's own tests: does it satisfy *their* requirements? |
| 4 | Code review | Verification (static) | Looks for bugs without running the code. |
| 5 | Beta testing | Validation | Real users in a real setting; also increases confidence. |
| 6 | SonarQube | Verification (static) | Static analysis looking for code smells and potential bugs. |
| 7 | System testing | Verification | Complete system checked against the spec. |
| 8 | Novice < 2 min | **Debatable**, which is the point the tutorial raises | Verification if it's a written non-functional requirement you're checking; validation if you're finding out whether real users are actually satisfied. |

</details>

### A2. Static or dynamic?

Classify each as **static** or **dynamic** verification: code inspection, unit testing, a linter, fuzzing, a type checker, regression testing, PMD's Copy/Paste Detector, beta testing, a walkthrough, load testing.

<details><summary>Answer</summary>

- **Static** (code not run): code inspection, linter, type checker, PMD Copy/Paste Detector, walkthrough.
- **Dynamic** (code is run): unit testing, fuzzing, regression testing, beta testing, load testing.

Key point from the tutorial: most static analysis tools **can't find real bugs** because "they have no knowledge of what the code is supposed to do". They know what the code *does*, not the spec. They mainly find code smells and potential runtime errors.

</details>

### A3. Which bug rule?

The intro lecture lists five rules: a bug exists when the software…

1. doesn't do something the spec says it should;
2. does something the spec says it shouldn't;
3. does something the spec doesn't mention;
4. doesn't do something the spec doesn't mention but should;
5. is hard to understand, hard to use, slow (non-functional).

Using the **banking transfer spec** (max €10,000, whole euros, 10-digit account, 400 on invalid input, 200 on success), which rule does each scenario break?

- (a) A transfer of €12,000 returns 200.
- (b) A transfer of €500 to a valid account returns 400.
- (c) A successful transfer also sends the customer's balance to a marketing API.
- (d) The API happily accepts the same request twice in a row and moves the money twice.
- (e) Each transfer takes 40 seconds to respond.

<details><summary>Answer</summary>

- (a) Rule 2: it does something the spec says it shouldn't (accept > €10,000).
- (b) Rule 1: it doesn't do what the spec says (a valid transfer should succeed).
- (c) Rule 3: it does something the spec doesn't mention.
- (d) Rule 4: the spec never mentions duplicate requests, but any bank *should* protect against them. This is an **implicit requirement**, which the lecture says is part of your job to identify.
- (e) Rule 5: non-functional (performance), even though the spec gives no time limit.

</details>

### A4. Coincidental correctness

The lecture's example: `y = x²` wrongly coded as `y = 2*x`, tested with `x = 2`, passes.

1. Find **every** integer input for which `y = 2*x` gives the same answer as `y = x²`.
2. A method should return `x + y` but is coded as `x * y`. Which integer inputs would let this bug pass?
3. A method should return `abs(x)` but is coded as `return x;`. Which half of the input domain hides the bug?
4. What does this tell you about choosing test inputs?

<details><summary>Answer</summary>

1. `2x = x²` gives `x(x − 2) = 0`, so **x = 0 and x = 2**.
2. `x + y = x * y` holds for **(0, 0) and (2, 2)**. (Rearranged: `(x−1)(y−1) = 1`, so x−1 and y−1 are both 1 or both −1.)
3. **All x ≥ 0.** Any test with a non-negative input passes.
4. "Easy", "middle of the road" values like 0, 1 and 2 are often exactly the ones where wrong formulas agree with right ones. Choose inputs that are likely to **reveal** a bug, and test **more than one member** of a partition when the computation is non-trivial.

</details>

### A5. The cost of late bugs

Using the lecture's rule of thumb (×10 per phase: spec $1, design $10, code $100, released $1,000) and the Waterfall split (spec ~55%, design ~25%, code ~15%, other ~5%):

1. A project ships with 200 bugs, all found after release. Roughly how many came from the specification, and what did fixing *those* cost?
2. What would the same spec bugs have cost if found during specification?
3. The lecture asks: "how are these 'bugs' [in the spec] detected?" Give two ways.

<details><summary>Answer</summary>

1. 55% of 200 = **110 spec bugs**, × $1,000 = **$110,000**.
2. 110 × $1 = **$110**: a thousand times cheaper.
3. Any two of: reviewing and inspecting the spec; **writing test cases before coding** (TDD), which "offers an opportunity to query the specification"; producing a **specification defects list**; validation with stakeholders (prototypes, use cases written together, feedback at each iteration).

</details>

### A6. Project Mercury's FORTRAN bug

`DO I=1.10` was written instead of `DO I=1,10`, and the loop ran exactly once.

1. FORTRAN ignores spaces, so how did the compiler read `DO I=1.10`?
2. Name a **static** technique that could have caught this before the code was ever run.
3. Why was it found only through "an analysis of why the software did not seem to generate results that were sufficiently accurate"? What was missing from the testing?

<details><summary>Answer</summary>

1. As an assignment: a new variable `DOI` is set to `1.10`. The "loop body" then just runs once as straight-line code.
2. A **code review/inspection** (a trained reader would spot `.` vs `,`), or a **static analysis tool** flagging a variable (`DOI`) that is assigned but never used.
3. There was **no precise oracle** for the outputs: nobody had a clear expected value to compare against, so slightly wrong results looked plausible. Expected outcomes must be specified *before* the test is run.

</details>

### A7. Short "why" questions (exam style)

Answer each in 2–3 lines.

1. Why is exhaustive testing not practical, even for a method that takes an array of 10 `int`s?
2. Explain Dijkstra's quote in your own words.
3. Why should someone other than the author test the code?
4. "If I give you some code and tell you to test it, what should your answer be?"
5. Why are black-box tests typically applied at system level rather than unit level?
6. A test fails. List the possible causes.
7. When should you stop testing? Give the common criteria and the better one.
8. Put the debugging steps in order: localise, retest (regression), confirm, fix, reproduce, add tests.

<details><summary>Answer</summary>

1. Each `int` has 2³² values, so 10 of them give (2³²)¹⁰ = **2³²⁰ ≈ 10⁹⁶ inputs**, and each one needs an expected outcome. Even "testing every path" doesn't guarantee correctness.
2. Passing tests only show that the inputs you tried worked. They say nothing about the inputs you didn't try. A failing test proves a bug exists; a passing suite never proves there are none. ("A passing test tells you nothing, but a failing test shows something is broken.")
3. Authors are psychologically less inclined to break their own code, and if they misunderstood the spec, their tests will share the same misunderstanding and won't recognise a wrong output. Independent testing finds more bugs, though it costs more.
4. **"Where's the specification?"** Without knowing the intended behaviour you can't write test *cases*, only run the code. Comments aren't a reliable spec.
5. Black-box tests come from the spec (use cases, user stories), which describes the behaviour of the **whole system**, not individual methods. Specs rarely describe each internal method.
6. A mistake in the automated test code; the oracle (expected result) is wrong; the system's behaviour is wrong; or a combination. It needs a thorough investigation, and you shouldn't assume it's the code.
7. In practice: when management says so, when the tester decides, or when the money or time runs out. Better: when the expected cost (time, money, effort) of finding the next bug, **weighted by seriousness**, is greater than its value, estimated from statistics on the bug-discovery rate over time (the whiteboard graph).
8. **Confirm → reproduce → localise → fix → add tests → retest (regression).**

</details>

---

## Part B: Oracles and tautological tests

### B1. Name the oracle

For each, name the **kind of oracle** (specification, reference implementation, human judgement, property) and say **how it could fail**.

1. Checking that a new tax calculator gives the same answers as last year's version.
2. A senior accountant looks at the generated payslips and says whether they're right.
3. Asserting that after any transfer, the total money across both accounts is unchanged.
4. Reading the expected grade off the module handbook's grading table.
5. Running the code, seeing it returns 200, and writing `assertEquals(200, …)`.

<details><summary>Answer</summary>

1. **Reference implementation.** Inherits every bug in last year's version, and this year's tax rules may have changed, so "different" isn't the same as "wrong".
2. **Human judgement.** Doesn't scale, isn't repeatable, and isn't available to the CI pipeline at 3am.
3. **Property.** Only partial: a transfer of the wrong amount still keeps the total unchanged.
4. **Specification.** Fails when the handbook is silent, ambiguous or out of date.
5. **Not an oracle at all.** It's a recording. It can never fail for a real reason, carries no information about the requirement, and turns any current bug into "expected" behaviour.

</details>

### B2. Spot the tautological test

```java
// Spec: amounts are whole euros only; invalid input returns 400.
@Test
void transferDecimal() {
    // I sent 100.00 and the API returned 200, so:
    assertEquals(200, api.transfer("1234567890", 100.00).status);
}
```

1. What's wrong with this test?
2. What happens later when someone fixes the parser to reject `100.00`?
3. Rewrite it properly. If you think the spec is ambiguous here, say what you'd do first.

<details><summary>Answer</summary>

1. The expected value came from **watching the implementation**, not from the spec. It will always pass, it raised coverage, and the spec may have been violated without anything firing.
2. The test goes red when the bug is fixed, so someone "fixes the test" instead. The defect is now **protected** by the suite.
3. `100.00` is **underdetermined**: is a fractional literal with a whole value "whole euros"? Follow the lecture's order:
   - **Resolve it** with the requirement owner and get the spec amended.
   - If you must test before then, **assert only what every permitted answer shares**, and assert the property that matters:

```java
@Test
void transferDecimalLiteral_rejectedOrUnchanged() {
    long before = bank.totalBalance("SENDER", "1234567890");
    int status = api.transfer("1234567890", "100.50").status;   // clearly not whole euros
    assertEquals(400, status);                                   // spec: invalid -> 400
    assertEquals(before, bank.totalBalance("SENDER", "1234567890")); // no money moved
}
```

- Use `100.50` for the unambiguous invalid case, and log `100.00` as a **query** in your defects list.

</details>

### B3. Write property oracles

Without knowing exact expected values, write **three properties** that must hold for:

1. the banking transfer API;
2. a `sort(int[] a)` method.

<details><summary>Answer</summary>

1. Banking:
   - The total balance across sender and recipient is unchanged by any request.
   - On **200**, the sender decreases by exactly `amount` and the recipient increases by exactly `amount`.
   - On **400**, **no** balance changes.
   - (Implicit requirement) the same request sent twice doesn't move money twice, or the second is refused. The spec is silent, so that's a query.
2. Sort:
   - The output is in non-decreasing order.
   - The output is a **permutation** of the input: same length, same elements with the same counts.
   - Sorting an already-sorted array returns it unchanged (idempotent).

- Note: "output is in order" alone is weak. `return new int[0]` satisfies it. Properties are partial by construction, so combine several.

</details>

---

## Part C: Specification defects list (his favourite exercise)

### C1. The car park

> "A car park charges €2 per hour or part hour. The maximum charge per day is €15. Customers displaying a valid disability badge park for free. The system is given the entry time and exit time and returns the charge. If the times are invalid, an error is returned."

Produce a **specification defects list**: for each **ambiguity, omission or contradiction**, explain the problem and **propose a resolution**. Use a sheet with columns: *ID, Type, Problem, Why it matters for testing, Proposed resolution*. Use AI if you like, but use your brain too.

<details><summary>Answer (not exhaustive, so try to find more)</summary>

| ID | Type | Problem | Why it matters | Proposed resolution |
|---|---|---|---|---|
| D1 | Ambiguity | "Per day": calendar day (midnight to midnight) or per 24-hour period? | A stay from 23:00 to 01:00 costs €4 either way, but 20:00 to 10:00 next day differs. | Define "day" as each 24h period from entry. |
| D2 | Omission | Multi-day stays: is the €15 cap applied per day, or once per stay? | No expected output for a 3-day stay. | €15 per started 24h period. |
| D3 | Ambiguity | Entry = exit (0 minutes): €0 or €2 ("part hour")? | Boundary value with two defensible answers. | Define a grace period, e.g. < 10 min is free. |
| D4 | Ambiguity | Exactly 60 min: 1 hour or "1 hour + part hour"? | Classic off-by-one boundary. | ≤ 60 min = 1 hour. |
| D5 | Omission | Cap reached at 7.5h (8 × €2 = €16 > €15): is it €15 from the 8th hour? | Output boundary. | Charge = min(2 × ceil(hours), 15). |
| D6 | Omission | What makes times "invalid"? Exit before entry? Future times? Wrong format? | Can't build invalid partitions. | List the invalid cases explicitly. |
| D7 | Omission | Which error? Code or message? | Oracle for the invalid tests is missing. | Return error code E1, with a message per case. |
| D8 | Omission | Daylight saving changes and time zones. | 1 hour appears or disappears in October/March. | Use UTC timestamps. |
| D9 | Ambiguity | "Valid disability badge": who checks it, and how is it given to the system? It isn't one of the inputs! | Contradiction: the spec says the system is given two times, but the charge depends on a third input. | Add `badgeId` as an input, validated against the registry. |
| D10 | Omission | Lost ticket, unknown entry time. | Common real case. | Flat €15 (or €30) fee. |
| D11 | Omission | Currency format and rounding. | Oracle precision. | Return whole euros. |
| D12 | Implicit req. | Non-functional: response time at the barrier, availability. | Long queues at the exit if slow. | Charge returned in < 1 s. |

</details>

### C2. Revisit the banking spec

Re-read the banking transfer spec from *AI-Assisted Black Box Testing 1*. List at least **10** defects. The lecture's discussion points give you a head start (10,000 exactly, 0, own account, nonexistent account, insufficient funds, injection, "what's missing?").

<details><summary>Answer</summary>

1. **10,000 exactly**: "maximum" suggests inclusive, but that's not stated.
2. **0 and negative amounts**: valid? Probably not, but not stated.
3. **Whole euros**: is `100.00` whole? What about `"100"` as a string, or `1e2`?
4. **Account number format**: are leading zeros allowed (`0123456789`)? As a number that's only 9 digits. Is it a string or a number?
5. **"Valid" account**: well-formed, or existing? Active? Closed? Frozen?
6. **Transfer to own account**: silence "is not permission, and it is not prohibition either".
7. **Insufficient funds**: "funds available" implies a balance check but **no error code is given** (400? 402? 422?).
8. **The sender account isn't an input**: where does it come from? Authentication is never mentioned (401/403?).
9. **Limit per transaction only**: two transfers of €10,000 in a row are allowed? Is there a daily limit?
10. **Duplicate or retried requests** (idempotency): double debit?
11. **200 "has succeeded"**: is the money moved, or just accepted for later processing?
12. **One error code for everything**: the client can't tell *why* it failed. Usability and support issue.
13. **Security**: injection or escape characters in fields, rate limiting.
14. **Atomicity**: the money must leave one account *and* arrive in the other, or neither (implicit requirement).
15. **Currency**: euro only? Cross-currency accounts?

</details>

---

## Part D: Equivalence partitioning and boundary values

Use his layout:

- **Tab Partitions:** ID, Input or output, Partition, Valid or invalid
- **Tab Test Cases:** ID, inputs…, Expected output, Partition IDs covered
- **Tab Queries:** Query, Affects tests, Assumption made

### D1. Exercise A, disk quota (his exercise, fully worked)

> Year 1 → 20Mb always; Year 2 → 30MB; Year 3 computing → 60MB; Year 3 not computing → 40MB; Year 4 computing → 100MB; Year 4 physics → 80MB; Year 4 neither → 60MB.

Derive **robust** test cases.

<details><summary>Answer: Partitions</summary>

| ID | Input/Output | Partition | Valid? |
|---|---|---|---|
| P1 | Input: year | 1 | Valid |
| P2 | Input: year | 2 | Valid |
| P3 | Input: year | 3 | Valid |
| P4 | Input: year | 4 | Valid |
| P5 | Input: year | < 1 (e.g. 0, −1) | Invalid |
| P6 | Input: year | > 4 (e.g. 5) | Invalid |
| P7 | Input: year | Not an integer (2.5, "two", empty) | Invalid |
| P8 | Input: dept | Computing | Valid |
| P9 | Input: dept | Physics | Valid |
| P10 | Input: dept | Any other real department (e.g. History) | Valid |
| P11 | Input: dept | Not a department (empty, null, "x#1") | Invalid |
| P12 | Output | 20 MB | Valid |
| P13 | Output | 30 MB | Valid |
| P14 | Output | 40 MB | Valid |
| P15 | Output | 60 MB | Valid |
| P16 | Output | 80 MB | Valid |
| P17 | Output | 100 MB | Valid |
| P18 | Output | Error / request refused | Invalid (spec silent: query) |
| P19 | Output | Any other quota (0, 50, negative…) | Invalid (speculative) |

</details>

<details><summary>Answer: Test cases</summary>

| ID | Year | Dept | Expected | Partitions |
|---|---|---|---|---|
| T1 | 1 | Computing | 20 MB | P1, P8, P12 |
| T2 | 1 | Physics | 20 MB | P1, P9, P12 ("always") |
| T3 | 2 | History | 30 MB | P2, P10, P13 |
| T4 | 3 | Computing | 60 MB | P3, P8, P15 |
| T5 | 3 | **Physics** | **40 MB** | P3, P9, P14 (physics is only special in Year 4: a likely bug) |
| T6 | 3 | History | 40 MB | P3, P10, P14 |
| T7 | 4 | Computing | 100 MB | P4, P8, P17 |
| T8 | 4 | Physics | 80 MB | P4, P9, P16 |
| T9 | 4 | History | 60 MB | P4, P10, P15 (60 reached by a *different* path than T4) |
| T10 | 0 | Computing | Error | P5, P8, P18 |
| T11 | 5 | Computing | Error | P6, P8, P18 |
| T12 | "two" | Computing | Error | P7, P8, P18 |
| T13 | 4 | "" (empty) | Error | P4, P11, P18 |
| T14 | 1 | "" (empty) | **Query**: 20 MB or Error? | P1, P11 |

P19 has no test: you can't force an invalid output. It's a check that no test ever returns it.

</details>

<details><summary>Answer: Queries</summary>

| Query | Affects | Assumption made |
|---|---|---|
| "20**Mb**" vs "30**MB**": megabits or megabytes? (20 Mb = 2.5 MB!) | T1, T2 | All values are MB. |
| What happens on invalid input? | T10–T14 | An error is returned, nothing allocated. |
| Is dept needed for Years 1–2 ("always")? | T14 | Dept ignored, so T14 → 20 MB. |
| Year 5 / postgrad / repeat students? | T11 | Invalid. |
| Joint honours (Computing *and* Physics)? | T7, T8 | Primary dept only. |
| Case sensitivity: "computing" vs "Computing"? | T4, T7 | Case-insensitive. |
| Does the quota change mid-year if a student changes dept? | n/a | No. |

</details>

### D2. Exercise B, `generate_grading` ("the nineteen")

> Exam mark out of 75, coursework mark out of 25, integers. Total = exam + c/w. ≥70 'A'; ≥50 and <70 'B'; ≥30 and <50 'C'; <30 'D'. A mark outside its range gives 'FM'.

1. Identify all equivalence partitions (input and output, valid and invalid). Your lecturer's version has **19**.
2. Add boundary values.
3. Now find the **partition most LLMs miss** (hint: an invalid input that produces a *valid-looking* total).

<details><summary>Answer: the 19 partitions</summary>

| ID | Input/Output | Partition | Valid? |
|---|---|---|---|
| 1 | Input: exam | 0 ≤ e ≤ 75 | Valid |
| 2 | Input: exam | e < 0 | Invalid |
| 3 | Input: exam | e > 75 | Invalid |
| 4 | Input: c/w | 0 ≤ c ≤ 25 | Valid |
| 5 | Input: c/w | c < 0 | Invalid |
| 6 | Input: c/w | c > 25 | Invalid |
| 7 | Input: exam | Real number (e.g. 48.7) | Invalid |
| 8 | Input: exam | Alphabetic (e.g. 'q') | Invalid |
| 9 | Input: c/w | Real number | Invalid |
| 10 | Input: c/w | Alphabetic | Invalid |
| 11 | Output | 70 ≤ t ≤ 100 → 'A' | Valid |
| 12 | Output | 50 ≤ t < 70 → 'B' | Valid |
| 13 | Output | 30 ≤ t < 50 → 'C' | Valid |
| 14 | Output | 0 ≤ t < 30 → 'D' | Valid |
| 15 | Output | t > 100 → 'FM' | Invalid |
| 16 | Output | t < 0 → 'FM' | Invalid |
| 17 | Output | 'E' | Invalid (speculative) |
| 18 | Output | 'A+' | Invalid (speculative) |
| 19 | Output | null | Invalid (speculative) |

**Test cases (one per partition):**

| ID | Exam | C/W | Total | Expected | Covers |
|---|---|---|---|---|---|
| T1 | 44 | 15 | 59 | B | 1, 4, 12 |
| T2 | −10 | 15 | | FM | 2 |
| T3 | 93 | 15 | | FM | 3 |
| T4 | 40 | −15 | | FM | 5 |
| T5 | 40 | 47 | | FM | 6 |
| T6 | 48.7 | 15 | | FM | 7 |
| T7 | 'q' | 15 | | FM | 8 |
| T8 | 40 | 12.76 | | FM | 9 |
| T9 | 40 | 'g' | | FM | 10 |
| T10 | 60 | 20 | 80 | A | 11 |
| T11 | 30 | 10 | 40 | C | 13 |
| T12 | 10 | 5 | 15 | D | 14 |
| T13 | 80 | 30 | 110 | FM | 15 |
| T14 | −10 | −10 | −20 | FM | 16 |

Partitions 17–19 can't be targeted by an input: they are outputs that must **never** appear in any test.

**Boundary values:**
- exam −1, 0, 75, 76
- c/w −1, 0, 25, 26
- totals 29/30, 49/50, 69/70, and 100 (75 + 25)

**The one LLMs usually miss:** exam = **80**, c/w = **0** → total **80**, which is in the 'A' range, but exam is out of range, so the answer must be **'FM'**. A lazy implementation that only range-checks the *total* returns 'A'. Likewise exam = 70, c/w = −5 → total 65 'B', but it should be 'FM'.

</details>

### D3. Boundary values for the banking spec

List the boundary test inputs (and expected status) for **amount** and **account number**.

<details><summary>Answer</summary>

| Input | Value | Expected | Note |
|---|---|---|---|
| amount | −1 | 400 | |
| amount | 0 | 400? | **Query**: is 0 a transfer? |
| amount | 1 | 200 | Smallest valid |
| amount | 9,999 | 200 | |
| amount | 10,000 | 200? | **Query**: inclusive? |
| amount | 10,001 | 400 | |
| amount | 100.50 | 400 | Not whole euros |
| account | 9 digits | 400 | |
| account | 10 digits, existing | 200 | |
| account | 11 digits | 400 | |
| account | 10 chars with a letter | 400 | |
| account | 0000000001 (leading zeros) | ? | **Query** |
| account | 10 digits, doesn't exist | ? | **Query**: "valid" means it exists? |

</details>

### D4. The `multiply` example (from the tutorial)

> Spec: *"given two integers x and y it returns x multiplied by y. Both inputs must be less than 1000 or an illegalArgumentsException should be raised."*

```java
public int multiply(int x, int y) {
    if (x > 999) throw new IllegalArgumentException("X should be less than 1000");
    return x / y;
}
```

```java
@Test void multiplyBasic()    { assertEquals(5,  tester.multiply(1, 5)); }
@Test void multiplyNegative() { assertEquals(-1, x.multiply(-2, 5)); }
@Test void multiplyBy0()      { assertEquals(0,  x.multiply(0, 0)); }
```

1. Predict the outcome of each of the three tests, and say whether the **code** or the **test** is at fault.
2. The tutorial later says the method should "return an error if the **first argument** is greater than 999". Compare that with the spec above. What's the problem?
3. List the black-box test cases (with expected results) you'd add, including boundaries.
4. What happens with `multiply(-2147483648, 2)`? Is it a bug in the code or in the spec?

<details><summary>Answer</summary>

1. Test outcomes:
   - `multiplyBasic`: actual `1/5 = 0` ≠ 5, so it **fails**. The **code** is wrong (divides).
   - `multiplyNegative`: the **test is wrong too**. −2 × 5 = **−10**, not −1. Actual `−2/5 = 0`, so it fails, but the oracle is also broken. (This is the "a failed test may indicate the oracle is wrong" point.)
   - `multiplyBy0`: `0/0` throws **ArithmeticException**, so it errors. The code is wrong.
2. **Contradiction:** the spec says *both* inputs must be < 1000, but the later text (and the code) only checks the *first*. Raise it as a query. Also, the spec names `illegalArgumentsException` but Java's class is `IllegalArgumentException`.
3. Tests to add:

   | Test | x | y | Expected |
   |---|---|---|---|
   | Valid max | 999 | 999 | 998001 |
   | x boundary | 1000 | 1 | IllegalArgumentException |
   | y boundary | 1 | 1000 | IllegalArgumentException (per "both") |
   | Zero | 0 | 7 | 0 |
   | Negatives | −3 | −4 | 12 |
   | Mixed sign | −2 | 5 | −10 |
   | Exception with extremes | Integer.MAX_VALUE | 0 | IllegalArgumentException |

4. −2147483648 < 1000, so it's "valid", but the product **overflows** an `int` (giving 0 in Java). The **spec** has no lower bound and no rule for overflow: an **underdetermined spec**. Add it to the defects list.

</details>

---

## Part E: White box and coverage

### E1. 100% branch coverage of `whatever`

```java
public int whatever(int x, int y) {
    int a; int b;
    if (x + y > 42) { a = x; b = 0; } else { a = y; b = 5; }
    if (a > 10) { b = b * 2; } else { b = b - 2; }
    return a + b + x + y;
}
```

1. Give the **minimum** number of tests for 100% branch coverage, with the value each returns.
2. Add boundary tests for both decisions.
3. The tutorial says this code "does not do anything useful so the expected result is difficult to describe". Why does that make your expected values in (1) suspicious?

<details><summary>Answer</summary>

1. **Two tests** cover all four branches:
   - (x = 40, y = 10): 50 > 42 → a = 40, b = 0; 40 > 10 → b = 0. Returns 40 + 0 + 40 + 10 = **90**. (True, True)
   - (x = 1, y = 2): 3 ≤ 42 → a = 2, b = 5; 2 ≤ 10 → b = 3. Returns 2 + 3 + 1 + 2 = **8**. (False, False)
2. Boundaries:
   - x + y = 42 vs 43: (21, 21) → else branch, a = 21, b = 5 → b = 10, returns 21 + 10 + 21 + 21 = **73**; (22, 21) → a = 22, b = 0, returns **65**.
   - a = 10 vs 11 (via the else branch, a = y): (0, 10) → b = 3, returns **23**; (0, 11) → b = 10, returns **32**.
   - Also extremes: (Integer.MAX_VALUE, 1) → `x + y` overflows to negative!
3. With no spec, the "expected" values were worked out **from the code itself**. That's exactly the tautological oracle: the tests can only confirm the code does what the code does. White-box techniques choose the **inputs**; the expected outputs must still come from a spec.

</details>

### E2. Why 100% isn't enough

```
a;
if (b) { c; }
d;
```

1. Minimum tests for 100% **statement** coverage? For 100% **branch** coverage?
2. For `if (A or (B and C))`, show a test set with 100% branch coverage where **B is never true**.
3. Give a test set achieving 100% **condition** coverage (each of A, B, C both true and false). What does short-circuit evaluation do to this?

<details><summary>Answer</summary>

1. **1** test (b true) for statement coverage; **2** (b true, b false) for branch coverage. Statement coverage never tests b = false.
2. {A = T} → true branch; {A = F, B = F} → false branch. Both branches covered, and B is never true. C is never even evaluated.
3. Without short-circuit: {(T, T, F), (F, F, T)}. Each condition takes both values, and the decision is T then F. **With short-circuit** (Java `||`/`&&`), when A is true, B and C are *never evaluated*, so you need more tests, e.g. {(F, T, T), (F, T, F), (F, F, –), (T, –, –)}.

- The point (tutorial): coverage numbers "sound good, are reassuring, easy to collect", but they reflect **effort more than quality**.

</details>

### E3. The hidden division

`x = 1 / (y - 42);`

1. Why can a million tests give 100% coverage of this line without finding the bug?
2. Which technique *would* find it: black box, white box, or static?

<details><summary>Answer</summary>

1. Coverage only needs the line **executed**, with any y. Only y = 42 triggers division by zero, and random or "middle" inputs are very unlikely to hit it.
2. **White-box boundary analysis** (looking at the expression, test y = 41, 42, 43) or a **static analysis** tool that flags a possible division by zero. Black box only finds it if the spec happens to make 42 a boundary.

</details>

### E4. Code smells

Which smells would a static analysis tool flag here?

```java
int total = 0;
total = computeBase();
if (total < 0 && total > 100) { applyDiscount(); }
String label = "Total: " + total;
total = computeBase();
return label;
```

<details><summary>Answer</summary>

- **Double assignment without use**: `total = 0` is immediately overwritten.
- **Unreachable (dead) code**: `total < 0 && total > 100` can never be true, so `applyDiscount()` never runs. (Probably meant `||`: the classic "and instead of or" that the tutorial says code inspections catch.)
- **Assigned but never used**: the final `total = computeBase();` has no effect.
- **Duplicated code**: `computeBase()` is called twice.

</details>

---

## Part F: AI-assisted testing (critique the LLM)

### F1. Review this AI output

An LLM was given the banking spec and produced:

| # | Amount | Account | Expected |
|---|---|---|---|
| 1 | 500 | 1234567890 | 200 |
| 2 | 15000 | 1234567890 | 400 |
| 3 | 500 | 12345 | 400 |
| 4 | −100 | 1234567890 | 400 |
| 5 | 9999 | 1234567890 | 200 |

Review it like the lecture does. What's good, what's missing, and where has it silently resolved an ambiguity?

<details><summary>Answer</summary>

- **Good:** valid case, over the limit, short account number, negative amount, near the limit.
- **Missing boundaries:** **10,000** exactly, 10,001, 0, 1; 9- and 11-digit accounts.
- **Missing invalid partitions:** decimal amounts (100.50), non-numeric amount, letters in the account number, empty or missing fields.
- **Missing spec-level cases:** own account; well-formed but **nonexistent** account; **insufficient funds**; authentication; duplicate requests; injection or escape characters.
- **Silent resolution:** by testing 9999 instead of 10,000 it **avoided** the inclusive/exclusive question rather than reporting it. Test 4 assumes "negative → 400", which the spec never says.
- **Weak oracle:** only the status code is checked. Nothing checks that **money moved** on 200 or **didn't** on 400.
- **No justifications or partition IDs**, so you can't tell why each test exists.
- The takeaway: "AI is useful for generating test cases if it has access to a solid specification. But its work must always be reviewed."

</details>

### F2. What the spec doesn't mention

For `generate_grading`, list outputs the component might produce that the spec **never mentions**, and the **implementation fault** that would cause each.

<details><summary>Answer</summary>

| Unmentioned output | Likely fault |
|---|---|
| 'A' for exam 80, c/w 0 | Only the total is range-checked, not each mark. |
| 'B' for total 70 | `>` used instead of `>=` (off-by-one). |
| Lowercase 'a' | Wrong constant or formatting. |
| 'E' or blank | Missing `else`, or a fall-through in a switch. |
| null | A path that never assigns the grade. |
| Exception or crash on non-integer input | No input validation (parsing error not caught). |
| 'FM' for exam 75, c/w 25 (total 100) | Upper bound coded as `< 100` instead of `<= 100`. |
| Wrong grade on huge inputs | Integer overflow in `exam + cw`. |
| 'FM' **and** a grade (two outputs) | Validation doesn't return early. |

- Lecture takeaway: LLMs partition what the specification **mentions**; asking what it might do that the spec **never mentions** "is the human tester's job".

</details>

---

## Part G: Quick-fire revision (cover the answers)

1. Define a **test case**.
2. Define a **test oracle**.
3. What is **regression testing**, and why must it be automated?
4. What's the difference between a **test** and a **test case**?
5. Name the three **testing phases** for verification and the two for validation.
6. Why do non-functional requirements tend to be harder to test? Give three examples.
7. What's the relationship between coverage and remaining bugs?
8. Approximately how many bugs does "even good" code have per 100 lines?
9. Roughly what share of commercial development effort goes into making the software work as expected?
10. An underdetermined spec allows 400, 402 or 422 for insufficient funds. What should your test assert?

<details><summary>Answers</summary>

1. Test inputs **plus the predicted outcome according to the specification**: inputs prepared, outcomes predicted, test documented, executed, results observed and evaluated.
2. Any mechanism (a document, a person, a second program, a property) for deciding whether an observed output is correct.
3. Re-running previous tests after a change, to check nothing has "regressed". Fixing bugs often introduces new ones, and re-running 1000s of tests by hand isn't feasible (e.g. JUnit in a DevOps pipeline).
4. A test only runs the code; a test case also states the **expected behaviour**. Without it, you can only detect crashes.
5. Verification: **unit** and **system** testing (plus integration between them). Validation: **acceptance** (alpha) and **beta** testing.
6. They are constraints, not features, and are often vague. Examples: worst-case execution time (real-time systems), memory usage (embedded), usability.
7. **No simple relationship**. Even 100% path coverage doesn't mean bugs are gone (Inozemtseva & Holmes, 2014). Coverage is necessary, not sufficient.
8. **1–3 bugs per 100 lines.**
9. **At least 50%**: making it work for real users takes at least 100% of the initial effort again.
10. Assert what every permitted answer shares: **a 4xx status, and that no money moved**. Then raise the ambiguity to get the spec fixed.

</details>
