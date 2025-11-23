# Changelog  
All notable changes to **Label BASIC Pro (LBP)** will be documented in this file.

This project follows a simplified semantic versioning scheme:
- **v1.0 RC** versions are release candidates (feature-complete, stabilisation phase)
- **v1.x** versions are compatible feature releases
- **v2.x** will introduce breaking language changes

---

## [v1.0 RC1] – 2025-03-??  
### First public release candidate (binary-only)

This is the first publicly available version of **LBP**, released as a `.PRG` program for the Commander X16.

#### Major features implemented
- **Fully working transpiler** from LBP → LB (Label BASIC).
- Written entirely in **Label BASIC**, bootstrapped via BASLOAD.
- Output ready for **BASLOAD** → BASIC V2 toolchain.
  
#### Implemented language features
- **Constants**
  - Single-line constants (`CONSTANT NAME = EXPR`)
  - Block constants (`CONSTANT … ENDCONST`)
  - Automatic `#DEFINE` emission for numeric constants within 0..65535 when `DEFINE ON`
  - String constants always emitted as assignments

- **ENUM blocks**
  - Auto-incrementing numeric values starting at 1
  - Optional explicit assignment (`NAME = n`)
  - Emitted as `#DEFINE` or assignment depending on options

- **Boolean built-ins**
  - `FALSE = 0`
  - `TRUE  = -1` (all bits set)

- **Structured control flow**
  - `IF / ELSE / END IF`
  - `WHILE / WEND`
  - `REPEAT / UNTIL`
  - Stack-based nesting tracking
  - Automatic label generation (`Z__IFn_*`, `Z__WHn_*`)

- **Loop control statements**
  - `EXIT WHILE`, `CONTINUE WHILE`
  - `EXIT REPEAT`, `CONTINUE REPEAT`
  - `EXIT LOOP`, `CONTINUE LOOP` (loop-type agnostic)

- **Compiler directives**
  - `REM ##OPTION DEFINE ON|OFF`
  - `REM ##OPTION REMPASS ON|OFF`
  - `REM ##OPTION STRICT_ERR ON|OFF`

- **Structural error handling**
  - Unterminated IF reporting
  - Unterminated loop reporting
  - Missing WEND / UNTIL / ELSE / END IF errors
  - Invalid EXIT/CONTINUE usage
  - Full abort when `STRICT_ERR ON`

- **Colon-shape micro-checks**
  - Structured lines (IF/WHILE/REPEAT/etc.) must not include `:`  
  - Produces syntax error if violated  
  - (Planned relaxations/expansions for v1.5+)

- **Robust line reader**
  - Handles CR, LF, CRLF, embedded NULs
  - Correct EOF detection using the CBM ST register (EOI bit)
  - Preserves blank lines

- **I/O and error paths**
  - Safe open/close for CHRGET/GET#, PRINT#
  - Disk error reporting via channel 15
  - Output file overwrite confirmation

---

## Planned for v1.0 Final  
*(Not yet included in RC1)*

- Documentation polishing
- More sample programs and regression tests
- Early warning system (deep nesting, suspicious colon usage, etc.)
- Minor output formatting improvements

---

## Planned for v1.5  
*(Future work, not implemented yet in RC1)*

- `COMPRESS` and `EXPAND` modes for colon-separated LB output
- `ASSERT` statements (strippable)
- REM_MAP output mode for source‐to‐output cross-referencing
- Symbol table introspection & reporting
- `#INCLUDE` support
- Multi-statement parsing for input lines
- Improved error context displays
- Optional support for ELSEIF lowering

---

## Long-term roadmap (v2.x and beyond)

- Full syntax tree for expression rewriting/optimisation
- Abstract intermediate representation (IR)
- Alternate backend: direct BASIC V2 generation (skipping LB)
- Compiled C version via LLVM-MOS for large program performance
- Experimental “LBP-Lite” for embedded scripting

---

## Earlier versions  
This RC1 release is the first public version; earlier versions were private prototypes.

---
