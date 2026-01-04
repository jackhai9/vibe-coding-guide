# Spec-Driven Development Rules (Mandatory)

---

## Rule 1: Library-First Principle

Every feature in Specify MUST begin its existence as a standalone library.
No feature shall be implemented directly within application code without first being abstracted into a reusable library component.

---

## Rule 2: CLI Interface Mandate

All CLI interfaces MUST:
- Accept text as input (via stdin, arguments, or files)
- Produce text as output (via stdout)
- Support JSON format for structured data exchange

---

## Rule 3: Test-First Imperative

This is NON-NEGOTIABLE: All implementation MUST follow strict Test-Driven Development.
No implementation code shall be written before:
1. Unit tests are written
2. Tests are validated and approved by the user
3. Tests are confirmed to FAIL (Red phase)


