> ⚠️ **This is an example adaptation, not the canonical implementation.**  
> **The CORE protocol is the source of truth.**

---

# Behavior Example — Ambiguity Handling Before Code Generation

This document shows a **realistic, simplified example** of how an AI coding assistant behaves when operating under the **Zero-Hallucination Protocol**.

The goal is not to demonstrate syntax or tooling, but **decision-making under uncertainty**.

---

## Context

A developer asks the AI to modify existing logic to link related entities based on specific criteria.

The request is **partially specified** and could be implemented in multiple valid ways, each with different trade-offs.

Under the Zero-Hallucination Protocol, the AI is **not allowed to assume intent**.

---

## AI Response (Before Any Code Is Generated)

### Pre-Flight Check Initiated

Before generating code, the AI performs a validation step to confirm understanding and detect ambiguity.

---

### What the AI Understands

Based on the request, the AI explicitly states its current understanding:

- A collection of related entities already exists in the current context
- Only a subset of those entities should be selected, based on specific criteria
- For each selected entity, an association should be created with a target entity
- The change is expected to be applied within an existing function

At this stage, **no assumptions are made** about data structures, ownership, or performance constraints.

---

### Identified Ambiguity

The AI detects that the request can be fulfilled in **multiple valid ways**, each with different implications.

Specifically, it is unclear **how the existing data should be reused or retrieved**.

---

### Clarification Required — Explicit Options

Instead of guessing, the AI presents **clear, numbered options** and explains the trade-offs:

1. **Reuse existing data already in scope**  
   - No additional data retrieval  
   - Requires passing existing context into the target function  

2. **Fetch the data again inside the target function**  
   - Keeps function signature unchanged  
   - Introduces an additional data access  

3. **Change the function interface to receive the data explicitly**  
   - Most explicit and testable  
   - Requires updating all callers  

The AI asks the developer to choose **one option explicitly** before proceeding.

---

### Stop Condition Applied

At this point:

- ❌ No code is generated  
- ❌ No defaults are assumed  
- ❌ No “best guess” is applied  

The AI **halts execution** and waits for human input.

This behavior is **intentional and mandatory**.

---

## Why This Matters

Without a strict stop condition, an AI assistant might:
- assume a preferred implementation
- introduce hidden coupling
- cause unnecessary data access
- break existing behavior silently

The Zero-Hallucination Protocol treats **ambiguity as a blocking error**, not something to smooth over.

---

## Key Takeaway

Under this protocol, the AI does not try to be clever.  
It tries to be **correct**.

When intent is unclear, the system stops, explains, and asks.

No assumption is worth broken code.
