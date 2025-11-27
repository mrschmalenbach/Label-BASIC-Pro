# Evil Test Suite for LBP v1.0 RC1

This folder contains deliberately malformed, structurally invalid, or borderline-illegal LBP programs.  
They are designed to stress-test:

- **Error detection**
- **STRICT_ERR behaviour**
- **Colon-shape syntactic guards**
- **Loop stack correctness**
- **IF stack correctness**
- **Reserved-label detection**
- **Option parsing robustness**
- **LBP’s ability to abort cleanly**

These files should **never run successfully**; they are expected to trigger one or more error messages.

If you are working on LBP itself or validating a port (e.g., C/LLVM-MOS), these tests help ensure you haven’t broken the error-reporting logic.

---

## How to Use These Tests

1. Copy this folder to your X16 environment or emulator disk image.  
2. Run LBP normally and select an `.lbp` file from this folder as input.  
3. Observe:
   - Which error is triggered
   - Whether the error line number is correct
   - Whether STRICT_ERR behaves correctly (aborts after error)
4. You may temporarily turn `STRICT_ERR OFF` inside the file to see the full cascade of errors, though some tests are designed to fail early.

---

## Test Files

### `evil_colons_01.lbp`
**Purpose:**  
Tests the colon-shape parser (`LBP.CHECK.COLON.STRUCTURED`).  
Contains:
- Legal colon usage
- Colons inside strings
- Illegal forms such as:
  - `PRINT "ok": IF X THEN ...`
  - `... : ELSE`  
  - `... : END IF`  
  - `... : WHILE`, `REPEAT`, `UNTIL`, etc.
- Colons in CONST/ENUM headers and bodies

**Expected:**  
First illegal structured keyword after a colon should trigger:  
`SYNTAX ERROR: STRUCTURED KEYWORD AFTER COLON AT LBP LINE n`  
With STRICT_ERR ON, transpilation aborts immediately.

---

### `evil_if_while_repeat_01.lbp`
**Purpose:**  
Stress-test IF/WHILE/REPEAT stacks.

Includes:
- Unterminated IF/WHILE/REPEAT
- ELSE without IF
- END IF without IF
- Multiple ELSEs for single IF
- WEND without WHILE
- UNTIL without REPEAT
- Wrong block terminator types

**Expected:**  
Various structural error messages such as:
- `ELSE WITHOUT IF`
- `END IF WITHOUT IF`
- `MULTIPLE ELSE`
- `WEND WITHOUT WHILE`
- `UNTIL WITHOUT REPEAT`
- `MISSING END IF` / `MISSING WEND/UNTIL`

---

### `evil_exit_continue_01.lbp`
**Purpose:**  
Verify EXIT/CONTINUE correctness and loop-type matching.

Contains:
- EXIT WHILE without a loop  
- CONTINUE REPEAT without a REPEAT  
- EXIT LOOP with no loop at all  
- UNTIL inside a WHILE loop  
- WEND inside a REPEAT loop

**Expected:**  
Errors such as:
- `EXIT WHILE WITHOUT WHILE`
- `UNTIL FOR NON-REPEAT LOOP`
- `WEND WITHOUT WHILE`
- `WEND FOR NON-WHILE LOOP`

---

### `evil_const_enum_01.lbp`
**Purpose:**  
Test CONST/ENUM blocks including colon restrictions.

Includes:
- `CONSTANT:` on same line (illegal)
- Colons inside CONST or ENUM block lines
- Missing ENDCONST
- Missing END ENUM
- ENUM header with colon
- ENUM member with colon
- Inconsistent terminators

**Expected:**  
Early colon-shape errors, block-mismatch errors at EOF:
- `COLON NOT ALLOWED ON STRUCTURED LINE`
- `MISSING ENDCONST`
- `MISSING END ENUM`

---

### `evil_reserved_labels_01.lbp`
**Purpose:**  
Ensure LBP detects attempts to define labels beginning with `Z__`.

`Z__` is reserved exclusively for LBP-generated labels.

Includes:
- User-defined labels like `Z__MYLABEL:`
- Attempt to define internal-like labels such as `Z__IF1_END:`

**Expected:**  
`SYNTAX ERROR: USER LABEL MAY NOT BEGIN WITH RESERVED PREFIX 'Z__'`  
Transpilation aborts with STRICT_ERR ON.

---

### `evil_options_01.lbp`
**Purpose:**  
Test the option parser, including unknown options and STRICT_ERR behaviour.

Contains:
- Unknown option names
- Unrecognised options before STRICT_ERR ON (non-fatal)
- Repeated LOGGING or REMPASS options
- Unknown option after STRICT_ERR ON (fatal)

**Expected:**  
- `WARNING: UNKNOWN OPTION '<name>'` (if STRICT_ERR OFF)  
- `SYNTAX ERROR: UNKNOWN OPTION '<name>'` (if STRICT_ERR ON)  
- Once STRICT_ERR ON, further unknown options stop the transpiler.

---

## Final Notes

- These files intentionally contain *multiple* errors.  
- Do not correct or “fix” them — they are here to break things on purpose.  
- Contributions of additional evil test cases are welcome!  

If an evil file *doesn’t* cause LBP to complain, that is probably a bug.
