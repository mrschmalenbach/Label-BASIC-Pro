# LBP Test Suite

This folder contains all automated and manual test cases for **Label BASIC Pro (LBP)**.

There are two subfolders:

---

## `good/`
Contains **valid** LBP programs that should transpile cleanly with no errors.

These files are designed to test:
- Correct handling of structured `IF/ELSE/END IF`
- Looping (`WHILE/WEND`, `REPEAT/UNTIL`)
- `EXIT`/`CONTINUE`
- CONSTANT and ENUM blocks
- Option handling
- General correctness and stability

All files in `good/` should produce a valid `.BAS` output file.

---

## `evil/`
Contains **intentionally invalid** LBP programs designed to trigger specific compiler errors.

These help verify:
- Correct error detection
- Meaningful error messages
- Protection against malformed structured constructs
- Prevention of illegal label reuse, malformed keywords, or syntax violations

If any `evil/` test transpiles successfully, something has gone wrong.

---

## How to Use

1. Run each test via LBP:

    RUN

2. Provide the appropriate `good/*.lbp` or `evil/*.lbp` filename.
3. Verify behaviour:
- **good/** tests must pass with no errors
- **evil/** tests must produce the expected error

Use these tests before each release (RC or stable) to ensure LBP behaves consistently.

