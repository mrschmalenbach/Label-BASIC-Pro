# List BASIC Pro / 'LBP' – Label BASIC Plus Transpiler
A transpiler for a superset of 'List BASIC' as implemented by the X16 BASLOAD command / utility.

**Status:** v1.0 RC1  
**Target:** Commander X16 (LB → BASIC V2 via BASLOAD)  
**Implementation language:** Label BASIC (LB) as supported by the X16 utility 'BASLOAD'

---

## Overview

**LBP** is a **source‑to‑source transpiler** that converts a higher‑level dialect called **Label BASIC+ (LBP)** into **Label BASIC (LB)**.  
LB is then fed to the Commander X16 ROM tool **BASLOAD**, which generates standard **Microsoft/Commodore BASIC V2** with line numbers and 2‑character variable names.

So the stack is:

```text
LBP source  -->  LBP transpiler  -->  LB source  -->  BASLOAD  -->  Tokenised BASIC V2
```

This program (LBP) is about making your source more **structured, readable, and maintainable**, while still ending up with plain old BASIC V2 at the end.

---

## What’s implemented in this codebase

The current code you’re looking at implements:

- A robust **line reader** that handles:
  - CR / LF / CRLF
  - Final line without newline
  - Embedded NULs
  - End‑of‑file via the CBM/X16 `ST` EOI bit
- A **driver loop** that:
  - Reads each line into `BUF$`
  - Maintains a logical input line counter `LBP.LINE%`
  - Dispatches each line to the appropriate handler
- A simple **symbol table** (for now used by constants and enums):
  - `SYM.NAME$()`, `SYM.TYPE%()` (1 = CONST, 2 = ENUM)
  - `SYM.IVALUE%()` for numeric values
  - `SYM.SVALUE$()` for string constants
- **Built‑in logical constants**:
  - `FALSE = 0`
  - `TRUE  = 65535` (all bits set in 16‑bit logic, which plays nicely with NOT)
- **Directives** via special comment lines:
  - `REM ##OPTION DEFINE ON|OFF`
  - `REM ##OPTION REMPASS ON|OFF`
  - `REM ##OPTION STRICT_ERR ON|OFF`
- **Constants**:
  - Single‑line: `CONSTANT NAME = EXPR`
  - Block form:
    ```basic
    CONSTANT
      NAME1 = 10
      MSG$  = "HELLO"
    ENDCONST
    ```
  - Numeric constants in 0..65535 become `#DEFINE` when `DEFINE` is ON, otherwise an assignment is emitted.
  - String constants always become assignment statements.
- **Enums**:
  - Block only:
    ```basic
    ENUM COLOUR
      RED
      GREEN = 5
      BLUE
    END ENUM
    ```
  - Auto‑numbering starts at 1.
  - Values are stored in the symbol table and emitted as either `#DEFINE` or `NAME = <value>` depending on range and `DEFINE` option.
- **Structured IF / ELSE / END IF** lowering to labels + GOTOs:
  - Recognises only **block‑style** IFs of the form:
    ```basic
    IF <condition> THEN
    ```
    with **nothing else** on that line.
  - `ELSE` and `END IF` must each be alone on their line.
  - It generates labels like:
    ```text
    Z__IF<n>_ELSE
    Z__IF<n>_END
    ```
  - IF nesting is tracked on a dedicated IF stack.
- **WHILE / WEND** loops:
  - `WHILE <condition>`  … `WEND`
  - Lowered via a loop stack with labels:
    ```text
    Z__WH<n>_TOP
    Z__WH<n>_CHK
    Z__WH<n>_END
    ```
- **REPEAT / UNTIL** loops:
  - `REPEAT` … `UNTIL <condition>`
  - Also uses the same loop stack with type flag (`WH.TYPE.STACK%()`):
    - `1` = WHILE/WEND
    - `2` = REPEAT/UNTIL
- **EXIT / CONTINUE** for loops:
  - `EXIT WHILE`
  - `CONTINUE WHILE`
  - `EXIT REPEAT`
  - `CONTINUE REPEAT`
  - `EXIT LOOP` (nearest enclosing loop, any type)
  - `CONTINUE LOOP` (nearest enclosing loop, any type)
- **Strict error mode**:
  - Controlled via: `REM ##OPTION STRICT_ERR ON` (or OFF).
  - Many structural errors (e.g. `ELSE` without `IF`, `WEND` without `WHILE`, `UNTIL` without `REPEAT`, excessive nesting) set an error message and, if `STRICT_ERR` is ON, also set `ABORT.FLAG%` and stop processing cleanly.
- **Unterminated‑block checks at EOF**:
  - Unterminated `IF` blocks reported by `LBP.REPORT.UNTERMINATED.IFS`.
  - Unterminated loops (both WHILE and REPEAT) reported by `LBP.REPORT.UNTERMINATED.WHILES`.

Everything else (lines that don’t match any of the above constructs) is currently treated as **pass‑through LB code**:

```basic
PRINT BUF$
PRINT#OUT.CH%, BUF$
```

No expression parsing or optimisation is performed in this version.

---

## Assumptions and limitations (important!)

To avoid surprising behaviour, the current implementation assumes:

1. **One logical statement per source line** for any line that starts with a structured keyword:
   - `IF`
   - `ELSE`
   - `END IF`
   - `WHILE`
   - `WEND`
   - `REPEAT`
   - `UNTIL`
   - `CONSTANT` / `ENDCONST`
   - `ENUM` / `END ENUM`
   - `REM ##OPTION ...`
2. **No support yet for splitting or joining colon‑separated statements.**
   - If you write:
     ```basic
     WHILE X<10: PRINT X
     ```
     the transpiler currently treats the whole line as if the condition were `X<10: PRINT X`, which will generate invalid output.
   - Until the planned `COMPRESS`/`EXPAND` modes arrive, keep all structured constructs on their own lines.
3. **No `ELSEIF` syntax.**
   - If you need that shape, use nested `IF` / `ELSE` / `END IF` manually in your LBP code.
4. **No symbol table export or macro system yet.**
   - The symbol table is currently internal only, and used to drive constant/enum emission.

---

## Directives

Directives are recognised only when written as:

```basic
REM ##OPTION NAME VALUE
```

Case is ignored for both NAME and VALUE.

### Supported options

Options:

| Option | Purpose | Default |
|--------|----------|---------|
| DEFINE | Use `#DEFINE` for numeric constants | ON |
| REMPASS | Pass LBP comment/directive lines to output | OFF |
| STRICT_ERR | Abort transpilation on structural error | OFF |

### ✓ Robust file I/O  
- Correct CBM-style EOF detection (ST & 64)  
- Handles CR, LF, CRLF, or final line with no terminator  
- Preserves blank lines  

### ✓ Stacks and structural validation  
- IF-stack with line tracking  
- Loop-stack with type tracking and labelled entry/exit points  
- Reports unterminated blocks at EOF  

### ✓ Built-in boolean constants  
Hard-wired (not #DEFINEs):
- `FALSE = 0`
- `TRUE  = -1     (all bits set, matches BASIC V2 truth semantics)`

#### `DEFINE`
Controls whether in‑range numeric constants and enums are emitted as `#DEFINE` or as assignments.

- `REM ##OPTION DEFINE ON`
- `REM ##OPTION DEFINE OFF`

Behaviour:

- When ON and value is in 0..65535:
  ```basic
  #DEFINE MAX.SIZE  100
  ```
- Otherwise:
  ```basic
  MAX.SIZE = 100
  ```

#### `REMPASS`

Controls whether the `REM ##OPTION ...` lines themselves are emitted to the output.

- When OFF (default): option lines are **not** written to the output file.
- When ON: option lines are also written out (useful for debugging or self‑documenting source).

#### `STRICT_ERR`

Controls whether structural errors abort the transpilation.

- When OFF (default):
  - Errors are printed, but LBP attempts to continue where possible.
- When ON:
  - Serious structural errors set `ABORT.FLAG%` and trigger the `LBP.ABORT` path, which cleans up and exits.

---

## Language features in detail

### Constants

**Single‑line constant:**
```basic
CONSTANT MAX.SIZE = 100
```

**Block constants:**
```basic
CONSTANT
  BUF.LEN     = 256
  PROMPT$     = "READY."
  ESC.SEQ$    = "{ESC}"
ENDCONST
```

- String constants are stored and emitted as assignment statements:
  ```basic
  PROMPT$ = "READY."
  ```
- Numeric constants are range‑checked for 0..65535.
- With `DEFINE ON`, in‑range numeric constants become `#DEFINE`.
- With `DEFINE OFF`, or when out of range, an assignment is emitted instead.

### Enums

```basic
ENUM ERROR.CODE
  ERR_OK
  ERR_NOTFOUND = 62
  ERR_TIMEOUT
END ENUM
```

This will assign:

- `ERR_OK = 1`
- `ERR_NOTFOUND = 62`
- `ERR_TIMEOUT = 63`

and emit either `#DEFINE` or `NAME = value` according to the `DEFINE` option and numeric range.

### Structured IF / ELSE / END IF

```basic
IF X>10 THEN
  PRINT "BIG"
ELSE
  PRINT "SMALL"
END IF
```

Lowered roughly as:

```basic
IF (X>10)=0 THEN Z__IF1_ELSE
PRINT "BIG"
GOTO Z__IF1_END
Z__IF1_ELSE:
PRINT "SMALL"
Z__IF1_END:
```

Constraints:

- `IF` line must be of the form `IF <cond> THEN` with nothing after `THEN`.
- `ELSE` must be alone on its line.
- `END IF` must be alone on its line.

### WHILE / WEND

```basic
WHILE X<10
  PRINT X
  X = X + 1
WEND
```

Lowered using loop labels:

```basic
Z__WH1_TOP:
IF (X<10)=0 THEN Z__WH1_END
PRINT X
X = X + 1
GOTO Z__WH1_TOP
Z__WH1_END:
```

### REPEAT / UNTIL

```basic
REPEAT
  X = X - 1
UNTIL X=0
```

Lowered to a post‑test loop using the same label scheme:

```basic
Z__WH2_TOP:
X = X - 1
Z__WH2_CHK:
IF (X=0)=0 THEN Z__WH2_TOP
Z__WH2_END:
```

### EXIT and CONTINUE

These work for both WHILE and REPEAT loops, and can also use the generic `LOOP` alias:

```basic
WHILE X<100
  IF X=50 THEN EXIT WHILE
  X = X + 1
WEND
```

Or:

```basic
REPEAT
  X = X + 1
  IF X=10 THEN CONTINUE LOOP
  PRINT X
UNTIL X>20
```

The handlers search the loop stack from the top down for the nearest matching loop type (or any loop, for `EXIT LOOP`/`CONTINUE LOOP`) and emit a `GOTO` to the correct label (`TOP`, `CHK`, or `END`).

---

## Error handling & diagnostics

Error Handling

LBP detects a range of structural errors:
- ELSE without matching IF
- END IF without IF
- WEND without WHILE
- UNTIL without REPEAT
- Unterminated blocks at EOF
- Colon-shape errors on guarded lines
- EXIT/CONTINUE with no matching loop
- Output file and disk errors via channel 15
- DOS status via `LBP.GET.DOS.STATUS` and channel 15 (`ST.CH%`) to report:
  - Error code
  - Error message
  - Track
  - Sector

Examples of error messages you might see:

- `SYNTAX ERROR: ELSE WITHOUT IF AT LINE 120`
- `SYNTAX ERROR: WEND WITHOUT WHILE AT LINE 80`
- `SYNTAX ERROR: MISSING END IF FOR IF STARTED AT LINE 42`
- `DISK ERROR 62 - FILE NOT FOUND (TRACK 0, SECTOR 0)`

With `STRICT_ERR ON`, most of these will cause an abort after the message. With strict mode OFF, LBP tries to continue where it is safe to do so.

---

## File handling

The program:

- Prompts for **device number** (default 8).
- Prompts for **input filename** (must be non‑empty).
- Prompts for **output filename**:
  - If left blank, uses input name with `.BAS` extension.
- Checks for existing output file and asks whether to overwrite.
- Opens:
  - Channel 2 for input (`IN.CH%`)
  - Channel 3 for output (`OUT.CH%`)
  - Channel 15 for DOS status (`ST.CH%`)

It correctly handles:

- EOF on the last character via the EOI bit in `ST`.
- Final lines with or without CR/LF.
- Blank lines (they are preserved and passed through).

---

## How to run it (on X16)

1. Copy `LBP.PRG` to your X16 disk/image.
2. From BASIC:

   ```basic
   LOAD "LBP.BAS"
   RUN
   ```

3. Follow the prompts:
   - Device number (press RETURN for 8)
   - Input filename
   - Output filename (or RETURN to append `.BAS`)

The output will be **LB source** suitable for feeding to BASLOAD or for further tools.

---

## Roadmap (beyond this code)

The current code is a **solid v1.0 RC1 core** with:

- Structural constructs (IF/ELSE/END IF, WHILE/WEND, REPEAT/UNTIL)
- Constants & enums
- Options system
- Strict error handling
- Clean IO and EOF handling

Planned future enhancements include (but are not yet implemented in this code):

- Statement compression/expansion (`COMPRESS ON|OFF|EXPAND`).
- REM‑based source‑map tags (`REM_MAP`) on output lines.
- `ASSERT` macro support, optionally removable.
- A warning system (unused constants, deep nesting, long lines, etc.).
- Symbol table dumping and cross‑reference reports.
- Include files (`#INCLUDE`).
- Expression analysis and optimisations in a later major version.

---

## License

MIT License - see LICENSE.md file.
