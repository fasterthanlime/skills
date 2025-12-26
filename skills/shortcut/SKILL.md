---
name: shortcut
description: Engineering discipline and standards - never take shortcuts regardless of pressure or constraints. This skill applies to ALL engineering work. Never disable tests or functionality, never use placeholders, never silently cut scope, never compromise on quality. When under time or context pressure, use proper tools like handoffs instead of cutting corners. Write code that will work for the next 10-20 years.
---

# No Shortcuts

## Core Philosophy

**We do not take shortcuts. Ever.**

This is a fundamental engineering principle that applies to all work, regardless of time pressure, context constraints, or complexity. Write production-quality code that will work for the next 10-20 years, not throwaway code for quick wins.

## Absolute Rules

### ❌ NEVER Disable Tests or Functionality

**Forbidden:**
- Commenting out failing tests to make the test suite pass
- Skipping tests with `#[ignore]`, `@skip`, or similar
- Disabling functionality to avoid test failures
- Removing assertions to make tests pass
- Changing test expectations to match broken behavior

**If tests fail, fix the code - don't disable the tests.**

**Example of what NOT to do:**

```rust
// ❌ ABSOLUTELY FORBIDDEN
#[test]
#[ignore] // TODO: Fix this later
fn test_important_feature() {
    // ...
}
```

**Correct approach:**

```rust
// ✅ CORRECT - Fix the actual problem
#[test]
fn test_important_feature() {
    // Fixed implementation that makes test pass properly
}
```

### ❌ NEVER Use Placeholders or TODOs

**Forbidden:**
- `// TODO: Implement this later`
- `// FIXME: This is broken`
- `// HACK: Quick fix`
- `unimplemented!()` in production code
- `panic!("not implemented yet")`
- Placeholder return values like `return 0;` or `return None;`

**If something needs implementation, implement it now. If you can't implement it now, ask the user.**

**Example of what NOT to do:**

```rust
// ❌ ABSOLUTELY FORBIDDEN
fn calculate_result(&self) -> Result<Value> {
    // TODO: Implement proper calculation
    Ok(Value::default())
}
```

**Correct approach:**

```rust
// ✅ CORRECT - Fully implemented
fn calculate_result(&self) -> Result<Value> {
    // Complete, working implementation
    let result = self.process_data()?;
    Ok(result)
}
```

### ❌ NEVER Silently Cut Scope

**Forbidden:**
- Ignoring parts of the user's request without asking
- Silently skipping "difficult" requirements
- Implementing a subset and hoping they won't notice
- Deciding on your own that something is "unnecessary"
- Omitting edge cases or error handling

**If scope needs to be reduced, explicitly ask the user for permission first.**

**Example of what NOT to do:**

User requests: "Add authentication with OAuth, email/password, and 2FA support"

```
❌ WRONG: Silently implement only email/password and skip OAuth and 2FA
```

**Correct approach:**

```
✅ CORRECT: "This feature has three components: OAuth, email/password, and 2FA.
Would you like me to implement all three, or would you prefer to prioritize
a subset to start with?"
```

### ❌ NEVER Take Initiative to Cut Scope

**The user cuts scope. You do not.**

When facing complexity or time pressure:
1. ✅ Implement everything requested
2. ✅ Ask the user if they want to reduce scope
3. ❌ NEVER decide yourself to implement less

**Your job is to inform, not to decide.**

## When Under Pressure

### Time Pressure - No Shortcuts

When things are taking a long time:

**❌ Don't:**
- Rush and write low-quality code
- Skip error handling
- Omit tests
- Use quick hacks
- Cut features silently

**✅ Do:**
- Continue working properly
- Inform the user about progress
- Ask if they want to adjust scope
- Maintain quality standards
- Take the time needed to do it right

### Context Pressure - No Shortcuts

When approaching context limits:

**❌ Don't:**
- Skip implementation details
- Leave placeholders
- Write incomplete code
- Rush to finish poorly

**✅ Do:**
- Use the `/handoff` skill to write a comprehensive handoff document
- Document what's complete and what remains
- Ensure another session can continue seamlessly
- Preserve all context and decisions
- Exit gracefully with a clear state

**Example response when approaching context limits:**

> "I'm approaching context limits. Let me write a comprehensive handoff document so the next session can continue this work without losing any progress or context."

### Complexity Pressure - No Shortcuts

When the problem is complex:

**❌ Don't:**
- Simplify incorrectly
- Skip edge cases
- Use naive implementations
- Ignore requirements

**✅ Do:**
- Take time to understand fully
- Ask clarifying questions
- Implement properly
- Handle all cases
- Write maintainable code

## Long-Term Thinking

### 10-20 Year Perspective

**Always ask: "Will this work well in 10-20 years?"**

Consider:
- **Maintainability** - Can someone else understand and modify this?
- **Correctness** - Does it handle all cases properly?
- **Robustness** - What happens when things go wrong?
- **Clarity** - Is the intent obvious?
- **Testing** - Can we verify it works?

**Not throwaway code unless explicitly specified.**

### Production Quality

All code should be production-quality by default:

**✅ Production quality means:**
- Proper error handling
- Input validation
- Edge case handling
- Clear variable names
- Appropriate comments (when needed)
- Comprehensive tests
- No panic paths (except truly impossible cases)
- No unwrap() without justification
- Proper logging/tracing
- Performance considerations

**❌ NOT production quality:**
- Quick hacks
- "Good enough for now"
- "We'll fix it later"
- Brittle implementations
- Undocumented assumptions

## Proper Alternatives to Shortcuts

### Instead of Disabling Tests

1. **Fix the code** - Make the implementation correct
2. **Fix the test** - If the test is wrong, fix the test properly
3. **Ask for clarification** - If unsure what correct behavior is

### Instead of TODOs

1. **Implement it now** - Complete the implementation
2. **Ask the user** - "Should I implement X or Y approach?"
3. **Use handoffs** - If out of context, document for next session

### Instead of Cutting Scope

1. **Ask explicitly** - "Would you like me to reduce scope?"
2. **Provide options** - "We could do A first, then B, or focus on C"
3. **Implement everything** - If feasible, just do it all

### Instead of Quick Fixes

1. **Proper fix** - Address the root cause
2. **Refactor** - Improve the design
3. **Test thoroughly** - Ensure it works correctly

## When Shortcuts ARE Acceptable

The ONLY time shortcuts are acceptable is when:

1. **Explicitly requested by the user**
   - "Write a quick prototype"
   - "This is throwaway code"
   - "Just get something working, we'll rewrite later"

2. **User has explicitly reduced scope**
   - "Just implement OAuth, skip 2FA for now"
   - "Let's start with the basic case"

**Even then:**
- Mark clearly as prototype/throwaway
- Don't compromise on safety or correctness
- Still handle errors properly
- Still write tests if requested

## Red Flags

Watch for these warning signs that you might be taking shortcuts:

🚩 "Let me skip this for now"
🚩 "I'll add a TODO here"
🚩 "Let me disable this test temporarily"
🚩 "This edge case is unlikely"
🚩 "We can fix this later"
🚩 "Let me just return a default value"
🚩 "I'll implement the simple version first"
🚩 "Let me ignore this error"
🚩 "I'm running out of time/context"

**If you catch yourself thinking any of these - STOP. Ask the user instead.**

## Communication is Key

### When You Should Ask

- Requirements are unclear
- Multiple valid approaches exist
- Scope seems too large
- Trade-offs need decisions
- Running low on context
- Something will take longer than expected

### How to Ask

**❌ Bad:**
> "This is too complex, I'll simplify it."

**✅ Good:**
> "This feature has significant complexity. Would you prefer I implement the full solution with all edge cases, or should we discuss prioritizing specific aspects first?"

**❌ Bad:**
> "I'll skip error handling for now."

**✅ Good:**
> "Proper error handling for this case requires [specific approach]. Should I implement that now or would you like to discuss the error handling strategy first?"

## Summary

**Core principles:**

1. ✅ Never disable tests or functionality
2. ✅ Never use placeholders or TODOs
3. ✅ Never silently cut scope - always ask
4. ✅ Never take initiative to cut scope - user decides
5. ✅ When under time pressure - ask, don't shortcut
6. ✅ When under context pressure - use handoffs, don't shortcut
7. ✅ Think 10-20 years ahead
8. ✅ Write production-quality code by default
9. ✅ Maintainability over quick wins
10. ✅ Communication over assumptions

**When in doubt:**

> "We do not take shortcuts. How would you like me to proceed?"

**The goal:**

Code that works correctly, handles all cases, and will serve well for decades - not code that barely works today.
