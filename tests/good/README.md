# Good Test Files for Label BASIC Pro (LBP)

This folder contains **valid, representative LBP programs** used for positive regression testing.  
Each file demonstrates a different set of features that LBP supports, including:

- `REM ##OPTION` handling  
- CONSTANT / ENUM blocks  
- Structured `IF / ELSE / END IF`  
- Structured loops: `WHILE/WEND`, `REPEAT/UNTIL`  
- `EXIT` / `CONTINUE` for both loop types  
- String and numeric expressions  
- Symbol table behaviour  
- Output correctness  
- General “good citizen” coding style for LBP  

All files in this directory **should transpile cleanly** under LBP RC1 with no errors.

---

## Files and Descriptions

### `good_if_nesting.lbp`
Demonstrates:
- Deep but valid `IF / ELSE / END IF` nesting  
- Conditions with string and numeric comparisons  
- Multi-block logic broken across several layers  
Useful for verifying correct label generation and stack handling.

---

### `good_loops_exit_continue.lbp`
Covers:
- `WHILE/WEND`
- `REPEAT/UNTIL`
- `EXIT WHILE`, `CONTINUE WHILE`
- `EXIT REPEAT`, `CONTINUE REPEAT`
Useful to verify loop stack behaviour and jump-label correctness.

---

### `good_constants_enums_simple.lbp`
A small demonstration of:
- Single-line CONSTANTs  
- Block CONSTANT/ENDCONST  
- ENUM / END ENUM blocks  
Good for confirming symbol table population and `#DEFINE` emission.

---

### `good_logging_basic.lbp`
Shows:
- Use of `REM ##OPTION LOGGING ON`
- Logging file auto-generation based on input filename  
- Typical usage of REMPASS  
This file helps verify logging configuration and output ordering.

---

## Mixed / Comprehensive Tests

### `good_all_features_status_panel.lbp`
A simple “status panel” simulation showing:
- CONSTANT blocks  
- ENUM usage for modes  
- Nested IF statements  
- WHILE loop with EXIT  
- Dynamic variable updates  
Ideal for validating structured IF lowering and multi-branch logic.

---

### `good_all_features_task_queue.lbp`
Imitates a small task-processing queue:
- Arrays for tasks  
- ENUM-based task types  
- REPEAT/UNTIL for queue iteration  
- EXIT/CONTINUE in natural contexts  
Useful for realistic control-flow testing.

---

### `good_all_features_state_machine.lbp`
A small finite-state machine:
- ENUM for states  
- Multi-tiered IF/ELSE IF logic  
- WHILE control loops  
- Safety break conditions  
Tests structured IF regions and label correctness under real state transitions.

---

### `good_all_features_mixed_structures.lbp`
A full integration test including:
- CONSTANTS  
- ENUMS  
- Multi-level IF/ELSE  
- WHILE and REPEAT loops  
- EXIT/CONTINUE behaviours  
- State progression across levels  
This is a very strong “everything together” sanity test for LBP RC1.

---

## How to Use These Tests

1. Run each file through the transpiler:

```
    LOAD “LBPTRC1.PRG”,8
    RUN
```

2. Supply the test filename (e.g., `good_if_nesting.lbp`)

3. Verify:
- No syntax errors are reported  
- Output `.BAS` file is generated  
- Output matches expectations for structured lowering  

4. (Optional) Turn on logging for additional verification:

Add to the top of the test file:

```
    REM ##OPTION LOGGING ON
```

5. Compare the `.LOG` file to the on-screen output for divergence.

---

## Notes

- All files in this folder are **intentionally valid**.  
- Anything that *should* raise an error is placed in the `evil/` folder instead.  
- If a file in `good/` produces an error, it indicates a regression in:
- structured IF lowering  
- structured loop handling  
- constant/enum parsing  
- option parsing  
- label resolution  
- output ordering or symbol emission  

These tests collectively exercise **most of LBP RC1** and should be re-run before each release.

