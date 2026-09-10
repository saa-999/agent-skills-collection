---
id: fix-overflow
name: fix-overflow
description: Fixes C/C++ memory-safety vulnerabilities (buffer overflows, integer overflows, use-after-free, and related classes) previously identified in a vulnerability report from a paired detection skill (e.g. check-overflow). Does not perform independent vulnerability detection — it acts strictly on findings already documented in an existing report. Use after a detection pass has produced a report and the vulnerabilities need to be patched, on any C/C++ codebase.
category: security
risk: middle
---

## How to Start

1. Locate the vulnerability report from the paired detection skill (commonly a markdown file such as `overflow-report.md`, or whatever path the user points you to).
2. Read the report carefully. For each finding, identify:
   - The exact file and line number(s).
   - The vulnerability class and severity.
   - The root cause described in the finding.
3. **Sort findings by severity before touching any code**: Critical → High → Medium → Low. Fix in that order. A lower-severity fix can change code that a higher-severity fix depends on — fixing out of order risks re-verifying against a moving target.
4. Open the actual source file at that location and inspect it: **has the programmer already fixed this specific vulnerability since the report was generated?**
   - If yes: skip it, note in your summary that it was already resolved, move to the next finding.
   - If no: fix it immediately, following the rules below.

## How to Write the Fix

* **Match existing style.** Write the fix using the coding style already present in the file (naming conventions, indentation, comment style, error-handling pattern). Do not invent your own style.
* **Prefer the smallest correct fix.** Default to a surgical, low-level correction — a bounds check, a corrected operand, a corrected comparison — rather than restructuring the function. Fix only the exact vulnerability identified; do not refactor, rename, or "improve" unrelated code, even if you notice other issues (log them under **Out-of-Scope Observations** instead).
* **Do not alter code logic unless the fix is impossible without it.** A bounds check, a corrected operand, or a guard condition is not a logic change. Restructuring control flow, changing a function's signature, or changing what a function returns *is* a logic change — it requires the `Logic-fix/` documentation described below.
* **Memory discipline.** A fix must not introduce unnecessary allocations, copies, or memory waste.
* **Record a before/after diff.** Before editing, capture the original lines being changed. After editing, present a clear before → after diff for every changed line in your summary, even a small one. The user must be able to review exactly what changed without re-reading the whole file.

## When to Stop and Ask Instead of Deciding Alone

Do not silently choose between competing valid fix strategies when the choice has real design trade-offs (e.g. reject the input vs. clamp it, change a function's return type vs. keep the old signature and accept a limitation, lock-based vs. lock-free synchronization). If more than one reasonable fix exists and they differ in behavior, performance, or API surface — stop, summarize the options with their trade-offs, and ask the user to choose. Apply the fix only after they decide. This does not apply to fixes with only one correct implementation (e.g. a clearly wrong comparison operator, a missing pointer dereference).

## Logic-Change Documentation (required when logic must change)

If a fix cannot be made without changing existing logic:

1. Create a folder named `Logic-fix/` inside the current skill's working directory.
2. Inside it, create a `.md` file (named after the finding) explaining what changed, why the surgical fix alone was insufficient, and every call site affected.
3. **Search the entire project, not just the locations named in the report**, for every other call site of the changed function/macro. A signature or return-type change breaks any caller silently if missed. Use a project-wide text/regex search for the exact symbol name before considering the fix complete.
4. In the same file, list the pros and cons of the change honestly (e.g. "adds one branch per call", "changes the function's return type, which could break external callers not covered by this codebase").

## After Fixing: Mandatory Re-Verification

A fix is not complete until all of the following are done:

1. **Compile check.** Attempt to compile the modified file (or the project, if a build system is available). A logic fix that is syntactically broken is not a fix. Record the compiler command used and its output (success, or the exact error) in your summary — do not just assume it compiles.
2. **Re-run the detection skill** on the same file(s) to confirm the specific finding no longer reproduces and that the fix did not introduce a new vulnerability (a new overflow path, a new ambiguous sentinel value, a missed call site, a new off-by-one).
3. **Update the original vulnerability report** with one of three states, not just Open/Resolved:
   - **Resolved** — the vulnerability is fully closed, verified by re-check and successful compile.
   - **Partially Resolved** — the immediate exploit path is closed but a known limitation remains (e.g. an ambiguous sentinel value, a precondition still required from the caller). Document the remaining gap explicitly so it isn't silently forgotten.
   - **Blocked** — could not be fixed without a decision from the user (see previous section) or without breaking compilation; explain why.
   Include the date and a one-line pointer to what changed. Never leave a resolved finding still marked Open, and never create a second disconnected report — the original report is the single source of truth.
4. **Update doc comments** (`@pre`, `@warning`, `@note`, Doxygen or otherwise) on the fixed function/macro so they match the new behavior. A comment that still tells the caller to guarantee something the function now checks itself is a documentation bug — fix it in the same pass.

## Out-of-Scope Observations

If you notice a separate issue while fixing a finding, do not fix it silently. List it under "Out-of-scope observations" in your summary so the user can decide whether it needs its own detection pass.

---

## Background Knowledge: Low-Level C You Must Reason From

This section is not optional reading — apply it every time you evaluate or write a fix, on any codebase. Memory-safety bugs in C/C++ almost always trace back to one of these mechanisms. Understanding *why* is what lets you pick the correct fix instead of a superficial one.

### The C memory and type model

* **No bounds checking is ever automatic.** Arrays, pointers, and `memcpy`/`memmove`/`strcpy`-family functions never know the "real" size of what they're pointing at. Every safety guarantee in C code exists only because a programmer wrote an explicit check — there is no language-level safety net. When you fix a vulnerability, you are *adding* the check that should have existed; you are never "restoring" one the language would otherwise provide.
* **Integer types have fixed, finite ranges.** `size_t` on a 64-bit target is unsigned, 8 bytes, range `0` to `2^64 - 1` (`SIZE_MAX`). Signed types (`int`, `long`, `ssize_t`) use one bit for sign; their overflow (unlike unsigned wraparound) is **undefined behavior** in C, not just "wraps silently" — the compiler is legally allowed to optimize assuming it never happens, which can hide the bug entirely at higher optimization levels.
* **Unsigned overflow wraps, it does not trap.** `SIZE_MAX + 1 == 0`. This wrap is exactly the mechanism behind most integer-overflow findings: an addition or multiplication that should have failed loudly instead silently produces a small, plausible-looking number, which then flows into an allocation size.
* **Implicit conversions are a common hiding place for bugs.** When a signed value is compared or combined with an unsigned one, C converts the signed value to unsigned first (the "usual arithmetic conversions"). A negative `int` becomes a huge `size_t`. Whenever a signed length or count reaches a function that takes `size_t`, check what happens if that value is negative.
* **Narrowing casts silently drop bits.** Assigning a `u64`/`size_t` into a `u32` (or passing a 64-bit value to a function expecting a 32-bit parameter) truncates the high 32 bits with no warning at the value level (only a compiler warning, if enabled). A bounds check performed on the original wide value can be completely bypassed if a narrowing cast happens afterward on the path to the actual write.
* **`sizeof` includes padding, not just declared members.** A struct's `sizeof` can be larger than the sum of its members due to alignment requirements (explicit `aligned(N)` attributes or natural alignment). Never assume `sizeof(struct) == sum of field sizes` — compute or verify it. A guard written against one alignment constant is not automatically correct against the struct's true `sizeof`.
* **Pointer arithmetic is scaled by element size, not bytes.** `ptr + n` on a `T*` advances by `n * sizeof(T)` bytes, not `n` bytes. An off-by-one in element count becomes `sizeof(T)` bytes of overflow, not 1 byte — often far more damaging than it looks from the source alone.

### The vulnerability classes, and their root-cause fix pattern

For each class below: understand *why* it happens before applying the fix — a fix that patches the symptom without understanding the mechanism tends to leave a variant of the same bug nearby.

* **Integer overflow → undersized allocation (CWE-190 → CWE-122).**
  Root cause: an allocation size computed via `+` or `*` on attacker-influenced or unchecked values, without checking the operation stayed within the type's range.
  Fix pattern: check *before* the operation, not after. Prefer compiler builtins (`__builtin_add_overflow`, `__builtin_mul_overflow`) which check at the hardware level, or an explicit pre-check (`if (a > SIZE_MAX - b)` for addition, `if (a > SIZE_MAX / b)` for multiplication, guarding against `b == 0`).

* **Off-by-one (CWE-193).**
  Root cause: `<=` where `<` was needed (or vice versa) in a loop bound, or an inclusive/exclusive boundary mismatch between an index and a length.
  Fix pattern: identify whether the array/buffer has `N` valid elements indexed `0..N-1`. The loop condition must be strictly `<` against `N`, or `<=` against `N-1` — never `<=` against `N` directly.

* **Use-after-free / double-free (CWE-416 / CWE-415).**
  Root cause: a pointer is used, or `free()`'d, after the memory it points to has already been released — often across a branch or error path that frees but doesn't clear the pointer.
  Fix pattern: set the pointer to `NULL` immediately after `free()`. Trace every `goto`/`return` after a free to ensure no other path can free the same pointer again.

* **TOCTOU (CWE-367).**
  Root cause: a validity/bounds check is performed, then the value or resource can change before it's actually used (common in concurrent or interrupt-driven code).
  Fix pattern: re-validate at the point of use, not just at the point of check, or hold whatever lock/atomic guarantee prevents the value from changing between check and use.

* **Uninitialized read (CWE-457).**
  Root cause: a stack or heap buffer is read before any write has occurred on that path.
  Fix pattern: zero-initialize at declaration/allocation, or ensure every code path that can reach the read has already written the value (not just the "normal" path).

### Advanced Vulnerability Patterns (Hard to Spot)

These are the classes that slip past a surface read because the code "looks fine" at a glance. Each entry explains *why it hides* before the fix — recognizing the disguise is the actual skill here.

* **Unsigned integer underflow (subtraction wraparound).**
  Why it hides: subtraction reads as obviously safe ("we're just computing a remaining size"), so it rarely gets the scrutiny given to `+`/`*`. But `size_t a - b` when `b > a` does not go negative — it wraps to a number near `SIZE_MAX`, which then sails past every subsequent bounds check that assumes "smaller must mean safer."
  Where to look: any `remaining = total - used`, `len = end - start`, or `capacity = max - offset` where the two operands aren't provably ordered at that point in the code.
  Fix pattern: check `if (b > a)` (would-underflow) *before* subtracting. Never trust a subtraction result to be "naturally small" just because it's typed as `size_t`.

* **Struct padding as an information leak.**
  Why it hides: it isn't a crash or corruption — the code runs perfectly. Compilers insert unnamed padding bytes between struct members to satisfy alignment; those bytes are never explicitly written, so they hold whatever garbage was previously in that memory. If the whole struct is later copied to the network, to disk, or to user-space via `memcpy`/`send`, the padding bytes leak along with it (CWE-200).
  Where to look: any struct with mixed-size members (e.g. `uint8_t` next to `uint64_t`) that gets passed whole to `memcpy`, a network send function, or a copy-to-user routine.
  Fix pattern: `memset(&s, 0, sizeof(s))` immediately after declaration, before any fields are set — or serialize field-by-field instead of copying the raw struct.

* **Returning or storing a pointer to a local (stack) variable.**
  Why it hides: the function compiles and often "works" in testing, because the stack memory hasn't been overwritten yet by the time it's used — the bug is timing-dependent, not deterministic.
  Where to look: any `return &local_var;`, or any global/heap struct field assigned the address of a variable declared inside the current function, without that variable being `static` or heap-allocated.
  Fix pattern: the value must live in heap memory (owned and freed explicitly) or in memory the caller provided, never in a callee's local stack frame that unwinds on return.

* **Double-free through pointer aliasing.**
  Why it hides: the two `free()` calls are often in different functions, or separated by many lines — it's two *different-looking* pointer variables that happen to hold the same address, so the simple "same variable freed twice" pattern doesn't visually match.
  Where to look: any place a pointer value is copied into a second variable, stored in two structs, or passed to two subsystems that each assume they own it and may each release it on cleanup/error paths.
  Fix pattern: establish single, unambiguous ownership for every allocation — exactly one code path is responsible for freeing it. If two components need access, one owns it and the other only borrows a reference.

* **Confused deputy across a trust boundary.**
  Why it hides: each individual function validates its input correctly *for the unit it expects* — the bug is that two layers disagree about what the number means (bytes vs. elements, one struct's alignment vs. a different struct's true size), not that either layer skipped validation.
  Where to look: any length/count value that crosses from one subsystem to another (e.g. a network layer's declared payload length reused directly as a buffer element count, or a bound validated against one constant being reused for an unrelated size).
  Fix pattern: re-validate the value against the *actual* constraint of the layer using it — never assume a check performed upstream, for a different purpose or unit, still applies downstream.

* **Race conditions from missing synchronization (beyond simple TOCTOU).**
  Why it hides: the code is logically correct under a single-threaded reading — the bug only exists in the interleaving between threads/interrupts/DMA, which won't show up by reading the function in isolation.
  Where to look: any shared state (a counter, a free-list head, a length field) touched from more than one execution context without a lock, atomic operation, or memory barrier.
  Fix pattern: identify the actual concurrent contexts that can touch the state, and use the project's existing synchronization primitive consistently with how the rest of the codebase protects similar state — do not invent a new synchronization pattern for one fix if the codebase already has a convention.

### Logical Habits to Apply on Every Fix

* **Never trust a value's type to guarantee its value.** `size_t` being unsigned tells you it can't be negative — it tells you nothing about whether it's a *plausible* size. A `size_t` of `0xFFFFFFFFFFFFFFF0` is just as "valid" a `size_t` as `64`.
* **Ask what the value's origin is, every time.** A length originating from a `sizeof()` or a compile-time constant needs far less scrutiny than one originating from a network packet, a user-controlled file, or a return value from an untrusted subsystem. Prioritize your suspicion accordingly.
* **A fix that only handles the exact reported input isn't a root-cause fix.** If the report shows one attack value, verify your fix by reasoning about the *general* condition (any value where the operation could exceed the type's range), not just by checking that the one example value no longer triggers it.
* **When two checks look redundant, verify they're actually checking the same thing before removing either.** A check can *look* sufficient (it guards against overflow) while silently guarding the wrong quantity. Redundant-looking checks sometimes each cover a different, non-overlapping case — read what each one actually compares before assuming one is dead code.

### Applying This to the Fix, Not Just the Diagnosis

When you write the actual fix line, ask yourself explicitly:
1. What is the exact type and width of every operand in the vulnerable expression?
2. Is there a narrowing cast anywhere between the check and the actual write/allocation?
3. Does my fix check *before* the dangerous operation, or does it try to detect the damage *after* it already happened (the second is almost always wrong for overflow)?
4. Does the fixed value's "safe" sentinel (e.g. returning `0` on failure) collide with any legitimate value the function could return? If so, flag it as a **Partially Resolved** limitation rather than silently shipping an ambiguous fix.