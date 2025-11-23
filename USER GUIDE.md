# Label BASIC Pro Language & Transpiler User Guide  
**Version: v1.0 RC1**

---

# Contents
1. [Introduction](#introduction)  
2. [Installation & Running](#installation--running)  
   - Requirements  
   - Running LBP  
3. [Writing LBP Source Code](#writing-lbp-source-code)  
4. [Constants & Enums](#constants--enums)  
5. [Enums](#enums)  
6. [Structured Conditionals](#structured-conditionals)  
7. [Looping Constructs & Loop Control](#looping-constructs--loop-control)  
8. [Options (Directives)](#options-directives)  
9. [Error Handling](#error-handling)  
10. [Example LBP Program](#example-lbp-program)  
11. [Output Files](#output-files)  
12. [Limitations of v1.0 RC1](#limitations-of-v10-rc1)  
13. [Notes for Advanced Users](#notes-for-advanced-users)  
14. [Troubleshooting (FAQs)](#troubleshooting-faqs)  
15. [Performance Notes](#performance-notes)  
16. [Summary](#summary)

---

# 1. Introduction

LBP (Label BASIC Pro) is a structured, expressive superset of Label BASIC (LB).  
It transpiles LBP into LB, which is then converted into standard BASIC V2 by the Commander X16’s **BASLOAD** utility.

LBP adds modern conveniences:

- Multi-line IF / ELSE / END IF  
- WHILE / WEND  
- REPEAT / UNTIL  
- EXIT / CONTINUE  
- CONSTANT and ENUM  
- Strict structural error checking  

LBP source is clean and maintainable, while final output remains compatible with BASIC V2.

**Important:**  
LBP and LB are **case-insensitive** for:

- Keywords  
- Statements  
- Labels  
- Variable names  

This matches BASIC V2 behaviour.

---

# 2. Installation & Running

## 2.1 Requirements

- Commander X16 (emulator or real hardware)  
- BASIC with **BASLOAD**  
- `LBP.PRG` on your disk/device  

## 2.2 Running LBP

At the BASIC prompt:

    LOAD "LBP.PRG"
    RUN

Prompts:

1. Device number (RETURN = 8)  
2. Input filename (.LBP)  
3. Output filename  
   - RETURN → use same name but `.BAS`  
   - If file exists → overwrite prompt  

LBP then transpiles and writes the `.BAS` LB output file.

---

# 3. Writing LBP Source Code

LBP accepts ASCII text using CR, LF, or CRLF endings.

### General rules

- Case-insensitive  
- One structured construct per line  
- Labels:
  - Must appear alone on line  
  - Must end with `:`  
- Identifiers:
  - Up to 64 characters  
  - Start with a letter  
  - May include letters, digits, `_` and `.`  
  - Must **not** begin with `Z__` (reserved for LBP)  
- Variable names and label names must not collide  
- X16 system variables (TI$, TI, DA$, ST, MX, MY, MB) must follow X16 rules  

### Structured constructs must appear alone (but may have `: REM` after):

- IF  
- ELSE  
- END IF  
- WHILE  
- WEND  
- REPEAT  
- UNTIL  
- CONSTANT / ENDCONST  
- ENUM / END ENUM  
- REM ##OPTION …  

Other LB statements may be multi-statement lines separated by `:`.

---

# 4. Constants & Enums

## 4.1 Constants

### Single-line constant

    CONSTANT MAX.SIZE = 100

### Block constants

    CONSTANT
      BUF.LEN = 256
      TITLE$  = "HELLO"
    ENDCONST

Notes:

- Supports numeric and string constants.
- With `DEFINE ON`, numeric constants (0–65535) emit as `#DEFINE <name of constant> <16 bit unsigned value>`.
- String constants ALWATS emit as assignments, so a string variable with the same name as the constant and with a trailing "$" string type indicator is created with the stated constant value. 
- Identifiers are case-insensitive.
- Labels, constants and variable names must all have unique identifier names.
- Constants emitted as variables rather than #DEFINEs tend to be more quickly processed by the BASIC V2 interpreter. 

## 4.2 Built-in System Constants

The standard BASIC V2 constant `PI` is available, with the value 3.14159265.

LBP automatically inserts:

    FALSE = 0
    TRUE  = -1

### Why TRUE = -1?

- Commodore BASIC uses all-bits-clear for FALSE and `NOT FALSE` is the same as `TRUE` which has all bits the inverse of all-bits-clear which means all bits are 1, and in signed 16 bit terms this equates to '-1'.  
- Ensures correct behaviour with NOT, AND, OR, because for working with these operators the numeric value, always a float, is translated to a signed 16 bit value for bitwise behaviour. An error is raised by the BASIC V2 interpreter if the numeric value is not in the -32768 to +32767 range for 16 bit signed values.

---

# 5. Enums

Example:

    ENUM COLOUR
      RED            : REM value =1
      GREEN = 5      : REM value =5 as stated
      BLUE           : REM value =6, which is 1 more than the last, 5
    END ENUM

Behaviour:

- First member defaults to 1.
- Members without explicit value auto-increment.
- Emitted as `#DEFINE` or assignment depending on `DEFINE` option.

---

# 6. Structured Conditionals

## 6.1 Multi-line IF

    IF SCORE >= 50 THEN
      PRINT "PASS"
    ELSE
      PRINT "FAIL"
    END IF

Rules:

- `IF <expr> THEN` alone on its line  
- `ELSE` alone  
- `END IF` alone  

LBP transpiler lowers this to sequence of labels and GOTO statements.

## 6.2 Nested IFs (Fully Supported)

    IF A THEN
      IF B THEN
        PRINT "A AND B"
      END IF
    ELSE
      PRINT "NOT A"
    END IF

## 6.3 ELSEIF Style

LBP does not support ELSEIF directly.  
Use nested IF:

    IF X < 0 THEN
      PRINT "NEGATIVE"
    ELSE
      IF X = 0 THEN
        PRINT "ZERO"
      ELSE
        PRINT "POSITIVE"
      END IF
    END IF

---

# 7. Looping Constructs & Loop Control

## 7.1 WHILE / WEND
WHILE <expr> is true, the loop will iterate or loop. **The loop might not execute at all**.

    WHILE COUNT < 10
      PRINT COUNT
      COUNT = COUNT + 1
    WEND

## 7.2 REPEAT / UNTIL
REPEAT... UNTIL <expr> is true, the loop will iterate or loop. **The loop will ALWAYS execute at least once**.

    REPEAT
      INPUT "VALUE:"; V%
    UNTIL V% >= 0

## 7.3 EXIT and CONTINUE

Supported:

    EXIT WHILE
    CONTINUE WHILE

    EXIT REPEAT
    CONTINUE REPEAT

    EXIT LOOP
    CONTINUE LOOP

`EXIT` means that the current stated loop type (WHILE or REPEAT) is exited, meaning code execution jumps immediately out of the loop and picks up with the very first statement after the corresponding `WEND` or `UNTIL` statement, depending on loop type.

`UNTIL` means that the current stated loop type (WHILE or REPEAT) is started from the next iteration, meaning code execution jumps immediately back to the start of the loop and picks up with the very first statement after the corresponding `WHILE` or `REPEAT` statement, depending on loop type.

Example:

    WHILE X < 100
      IF X = 50 THEN EXIT WHILE
      X = X + 1
    WEND

Another:

    REPEAT
      X = X + 1
      IF X MOD 2 = 1 THEN CONTINUE LOOP
      PRINT "EVEN:"; X
    UNTIL X >= 20

LBP searches the loop stack for the nearest matching loop. This is because a specific loop type (`WHILE` or `REPEAT`) has not been specified.

---

# 8. Options (Directives)

Written as:

    REM ##OPTION NAME VALUE

Case-insensitive.

## 8.1 DEFINE ON|OFF

    REM ##OPTION DEFINE ON

Controls lowering of numeric constants/enums (`#DEFINE` vs assignment).

## 8.2 REMPASS ON|OFF

    REM ##OPTION REMPASS ON

Controls whether option lines are emitted.

## 8.3 STRICT_ERR ON|OFF

    REM ##OPTION STRICT_ERR ON

ON → abort on structural errors  
OFF → continue where safe  

---

# 9. Error Handling

LBP performs full structural validation with IF and loop stacks.  
This is the complete error set for v1.0 RC1.

---

## 9.1 ELSE Without IF

Every `ELSE` statement must be part of an earlier `IF-THEN` statement that doesn't already have a prior `ELSE` or `END IF` statement or keyword associated with it. 

**Message:**

    SYNTAX ERROR: ELSE WITHOUT IF AT LINE n

**Cause:** An ELSE appears when no structured IF statement or construct is active.

**Example (invalid)**

    PRINT "HELLO"
    ELSE

**Fix:**
Ensure the IF block exists:

    IF X THEN
      PRINT "HELLO"
    ELSE
      PRINT "WORLD"
    END IF

---

## 9.2 Multiple ELSE For IF

This relates to 9.1 above, and arises when there's already an `ELSE` statement associated with the last IF-THEN statement that hasn't yet been closed with an associated `END IF` statement.

**Message:**

    SYNTAX ERROR: MULTIPLE ELSE FOR IF AT LINE n

**Cause:** An ELSE appears when previous ELSE statements have appeared associated with the same `IF-THEN` construct.

**Example (invalid)**

    IF X THEN
      PRINT "A"
    ELSE
      PRINT "B"
    ELSE
      PRINT "C"    ← invalid
    END IF
    
**Fix:**
Use nested IFs instead of chained ELSEIF (not supported yet):

    IF X THEN
      PRINT "A"
    ELSE
      IF Y THEN
        PRINT "B"
      ELSE
        PRINT "C"
      END IF
    END IF

---

## 9.3 END IF Without IF

**Message:**

    SYNTAX ERROR: END IF WITHOUT IF AT LINE n

**Cause:** No IF block is currently open.

**Example:**

    END IF

**Fix:**
Ensure every `IF-THEN` statement has just one and only one matching/closing `END IF` statement.

---

## 9.4 Missing END IF at EOF

**Message:**

    SYNTAX ERROR: MISSING END IF FOR IF STARTED AT LINE m

**Cause:** Reached end-of-file with one or more IF blocks still open.

**Example:**

    IF X THEN
      PRINT "HELLO"

**Fix:** Add corresponding END IF.

---

## 9.5 WEND Without WHILE

**Message:**

    SYNTAX ERROR: WEND WITHOUT WHILE AT LINE n

**Cause:** WEND appears when no WHILE is active.

---

## 9.6 UNTIL Without REPEAT

**Message:**

    SYNTAX ERROR: UNTIL WITHOUT REPEAT AT LINE n

**Cause:** An UNTIL is found, but there is no matching REPEAT.

---

## 9.7 UNTIL For Non-REPEAT Loop

**Message:**

    SYNTAX ERROR: UNTIL FOR NON-REPEAT LOOP AT LINE n

**Cause:** Top of loop stack is a WHILE loop, not a REPEAT loop.

**Example:**

    WHILE X<10
      PRINT X
    UNTIL X=5     ← invalid: cannot close WHILE with UNTIL

---

## 9.8 Missing Loop Terminator At EOF

**Message:**

WHILE:

    SYNTAX ERROR: MISSING WEND FOR WHILE STARTED AT LINE m

REPEAT:

    SYNTAX ERROR: MISSING UNTIL FOR REPEAT STARTED AT LINE m

**Cause:** Reached EOF with an unterminated WHILE or REPEAT block.

---

## 9.9 Colon Not Allowed On Structured Line

**Message:**

    SYNTAX ERROR: COLON NOT ALLOWED ON STRUCTURED LINE AT LBP LINE n

Applies to:
- CONSTANT
- ENDCONST
- ENUM
- END ENUM
- ELSE
- END IF
- WHILE
- WEND
- REPEAT
- UNTIL
- All REM ##OPTION lines

**Except for structured IF headers, which have special rules.**

---

## 9.10 Structured (multi-line) IF Header Colon Rules

There's a little bit of nuance here so familiarise yourself with this aspect. It can be the source of harder-to-track down errors in your source code.

### Valid:

    IF X THEN
    IF X THEN : REM comment
    IF A=1 THEN : REM begin block

### Invalid:

    IF X THEN : PRINT "NO"
    IF X THEN : A=A+1

**Reason:** BASIC V2 interprets these as single-line IFs, where the THEN-body, the statement after the `THEN` keyword can't be empty.

### Valid single-line IFs:

    IF X THEN PRINT "YES"
    IF X THEN A=A+1 : PRINT "DONE"

### Invalid structured IF headers:**

    IF X THEN : PRINT "NO"      ← invalid
    IF X THEN : A=A+1           ← invalid
    IF X THEN : GOSUB MENU      ← invalid

These are interpreted as single-line IFs, but since the THEN-body would be empty (the expression begins after a colon), BASIC V2 rejects them as malformed.

LBP treats:
- `IF X THEN + comment`   → structured IF
- `IF X THEN : PRINT ...` → illegal for structured IF; passes through or errors depending on context
- `IF X THEN : REM ...`   → structured IF header
   
---

## 9.11 Reserved Label Prefix “Z__”

**Message:**

    SYNTAX ERROR: LABEL MAY NOT BEGIN WITH 'Z__' (RESERVED) AT LINE n

**Cause:** User code attempts to define a label with the prefix reserved for LBP-generated code.

**Example (invalid):**

    Z__LOOP:
    PRINT "HELLO"

LBP uses Z__ exclusively for generated labels (IF/WHILE/REPEAT lowering).
User code must not define or reference labels beginning with Z__.

---

## 9.12 Disk I/O Errors

These come directly from the X16 DOS channel and are not generated by LBP itself.

**Typical messages:**

    DISK ERROR 62 - FILE NOT FOUND
    DISK ERROR 63 - FILE EXISTS
    DISK ERROR 50 - RECORD NOT PRESENT
    DISK ERROR 70 - NO CHANNEL

LBP simply reports them and halts.

---

# 10. Example LBP Program

    REM ##OPTION DEFINE OFF

    CONSTANT
      MAX.SCORE = 100
      PASS.MARK = 60
    ENDCONST

    INPUT "ENTER SCORE:"; S%

    IF S% < 0 THEN
      PRINT "INVALID"
    ELSE
      IF S% > MAX.SCORE THEN
        PRINT "TOO HIGH"
      ELSE
        IF S% >= PASS.MARK THEN
          PRINT "PASS"
        ELSE
          PRINT "FAIL"
        END IF
      END IF
    END IF

    END

---

# 11. Output Files

LBP produces:

- Your original `.LBP` file  
- A generated `.BAS` for BASLOAD  

---

# 12. Limitations of v1.0 RC1

- No ELSEIF  
- No multi-statement structured lines  
- No include files  
- No macro system  
- No expression analysis  
- No compression/expansion  
- No symbol table dump  

---

# 13. Notes for Advanced Users

- Output is plain LB  
- All LBP-generated labels begin with `Z__`  
- TRUE = -1, FALSE = 0  
- Blank lines preserved  
- Case-insensitive  

---

# 14. Troubleshooting (FAQs)

### Q1: Why can’t I write `IF X THEN : PRINT "YES"`?

Because it is treated as a **single-line IF** and has an empty THEN-body.

### Q2: Why does `IF X THEN : REM COMMENT` work?

Because LBP recognises it as a structured IF header.

### Q3: Why does LBP reject `Z__LABEL:`?

`Z__` is reserved for internal autogenerated labels.

### Q4: Why do I get colon errors?

Structured lines cannot contain colon characters except in allowed IF headers.

---

# 15. Performance Notes

LBP is written in LB and runs at BASIC V2 speed.

- Up to ~250 lines → performs well  
- 250–600 lines → noticeable delay  
- 600+ lines → go make a coffee  

Tips:

- Avoid very long lines (>200 chars)  
- Avoid deeply nested constructs  
- Keep constant/enum blocks manageable  

---

# 16. Summary

LBP v1.0 RC1 provides:

- A reliable structured language atop LB  
- Predictable lowering into BASIC V2  
- Robust error checking  
- Clean separation of logic and labels  
- Full support for nested structures and loop control  

It is ready for community use and feedback.
