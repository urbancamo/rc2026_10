# CLAUDE.md

Project notes and conventions for Claude Code in this repository.

## Project

Urbancamo's Retrochallenge 2026/10 entry — a collection of small, standalone C
utilities built for the [October 2026 Retrochallenge](https://retrochallenge.org).
See README.md for the project blog link.

## Specifications

Before implementing a new utility or any non-trivial feature, write a spec in
`specs/` first and get it agreed before starting the build.

- One file per utility/feature: `specs/YYYY-MM-DD-<kebab-case-name>.md`
- Start from `specs/TEMPLATE.md`
- Keep it short — enough to fix scope and approach, not a design document
- If the implementation ends up diverging from the spec, update the spec to match

## Repository layout

Currently empty aside from this scaffolding. Intended layout as utilities are
added — adjust here as the real structure settles:

- `specs/` — one spec per utility/feature, written before implementation
- `src/<utility-name>/` — source for each standalone C utility
- `common/` — code shared across utilities (add only once a second utility needs it)
- `tests/<utility-name>/` — tests for each utility

## Target Systems

Code that is generated should be written in C and compilable on any posix-compliant system.
The following systems should be supported:

 - VAX C Compiler from OpenVMS 7.3
 - Linux
 - MacOS
 - BSD Unix
 - Ultrix
 - Digital Unix 4.0E for Alpha

The build system should similarly be supported by each of these systems, assume that GNU Make is 
available, but not necessarily a recent version. The aim here is to keep the C-code as compatible 
as possible. Unless specified only standard C-libraries supported by these systems should be used.

## Build

## Code style

VAX C (OpenVMS) and Ultrix's `cc` are the binding constraints — both are
K&R-derived compilers with ANSI features added rather than conformant C89
implementations. Write to the level they can actually handle; the modern
targets (Linux/macOS/BSD) and Digital Unix will accept anything this strict
subset allows.

### Language dialect

- Target **ANSI C89** only — no C99/C11/C17 features anywhere.
- No `//` comments — always `/* ... */`.
- Declare all variables at the top of their block; never mix declarations and
  statements.
- No variable-length arrays, designated initializers, compound literals,
  `inline`, `restrict`, or anonymous struct/union members.
- No `stdint.h`, `stdbool.h`, or `long long` (all C99+). Use `int` for values
  that need a guaranteed 32-bit width; don't assume the width of `long` (VAX
  is 32-bit, Alpha and 64-bit Linux/macOS/BSD are 64-bit).
- Always write full ANSI prototypes (return type + parameter types) in a
  header, even though VAX C checks them loosely — the stricter modern
  compilers will catch mismatches during day-to-day development.
- Function *definitions* use ANSI-style parameter lists everywhere
  (`int foo(int a, int b)`), not old-style K&R definitions. No K&R fallback.

### Portability discipline

- No compiler-specific extensions: no `__attribute__`, `#pragma pack`, GCC
  statement expressions, or similar.
- Standard `#ifndef` / `#define` / `#endif` header guards only — no
  `#pragma once` (not guaranteed on VAX C or Ultrix's `cc`).
- OpenVMS is not a POSIX/Unix system. Anything touching the filesystem,
  process control, or sockets needs to go behind a small `#ifdef`-guarded
  shim (e.g. `platform_vms.c` vs `platform_unix.c`) rather than being called
  directly from utility code, which should stay OS-agnostic.
- Stick to the standard C library (`stdio.h`, `stdlib.h`, `string.h`,
  `ctype.h`) for anything that isn't inherently OS-specific, per the
  standard-libraries-only rule above.

### Naming and formatting

- `snake_case` for functions and variables, `UPPER_SNAKE_CASE` for macros and
  constants.
- 2-space indentation, no tabs.
- Small, single-purpose functions — easier to audit for portability issues
  across six toolchains that can't all be built and tested locally.

### Local verification

VMS/Ultrix/Digital Unix builds can't run in regular local testing, so use
GCC/Clang as the closest practical proxy: build with
`-std=c89 -pedantic -Wall -Wextra` to catch anything that strays outside the
strict subset above before it reaches the real target compilers.
