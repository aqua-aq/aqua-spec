# Aqua Language Specification: Introduction


**Aqua** is a dynamic, expression-oriented programming language for general-purpose development.  
It combines a flexible, readable syntax inspired by Lua and Ruby with a lightweight object system and functional programming features.


## Key Features

- **Dynamic Typing:** Variables do not require explicit type declarations.
- **Expression-Oriented:** Almost every construct, including `if`, `while`, `begin` blocks, and loops, returns a value.
- **Objects & `it`:** Functions can act as constructors. Each function has a base object (`it`) which represents the function's default context.  
  While it allows an OOP-like style, Aqua is primarily function-based.
- **Built-in Data Types:** 
  - `number`, `string`, `boolean`, `null`, `array`, `object`, `error`, `subroutine`, `method` (subroutine with context)
- **Standard Library (planned for v1.0.0):** `os`, `strings`, `lists`, `math`
- **Operator overloading**


## Current Status

- **Core:** Parser, runtime, expression evaluation, partial built-in functions.
  - CLI: `REPL`, `creating projects`, `running projects`, `executing files`
- **Work In Progress:** Standard library, bug fixes, more language features.
- **Planned v1.0.0:** Full standard library, stable constructs, improved error handling.
- **Planned v2.0.0:** Rewriting runtime from tree-runing to bytecode-running

## Example: Hello World

```aqua
println("Hello World!")
```

## Resources

- [Aqua-Core Repository](https://github.com/aqua-aq/aqua-core)
- [Aqua-Docs](https://github.com/aqua-aq/aqua-docs)
- [Aqua-Spec](https://github.com/aqua-aq/aqua-spec)
