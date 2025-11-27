# 🚀 Announcing Label BASIC Pro v1.0 RC1
### A structured, readable superset of Label BASIC — now available for the Commander X16 community!

Hello everyone!
I'm very pleased to share the first public release candidate of Label BASIC Pro (LBP) — a structured dialect of Label BASIC that transpiles into LB for use with the Commander X16’s built-in BASLOAD tool.

LBP is written entirely in LB itself and is aimed at anyone who wants cleaner, more maintainable BASIC code that still compiles down to standard Microsoft/Commodore BASIC V2.

------------------------------------------------------------
WHAT IS LABEL BASIC PRO?
------------------------------------------------------------

LBP is a source-to-source transpiler.
You write expressive, structured code like:

    IF SCORE >= 50 THEN
      PRINT "PASS"
    ELSE
      PRINT "FAIL"
    END IF

…and LBP converts it into valid Label BASIC (LB), which BASLOAD then converts into BASIC V2.

Goal:
MAX readability, ZERO runtime cost, 100% V2 compatibility.

------------------------------------------------------------
KEY FEATURES IN v1.0 RC1
------------------------------------------------------------

✓ Multi-line structured constructs
    IF / ELSE / END IF
    WHILE / WEND
    REPEAT / UNTIL

✓ Loop flow control
    EXIT WHILE, CONTINUE WHILE
    EXIT REPEAT, CONTINUE REPEAT
    EXIT LOOP, CONTINUE LOOP

✓ Constants & enums
    CONSTANT MAX.SIZE = 100

    ENUM COLOUR
      RED       : REM 1
      GREEN = 5 : REM 5
      BLUE      : REM 6
    END ENUM

✓ Strong structural error checking
    mismatched IF/ELSE/END IF
    stray WEND/UNTIL
    colon misuse (PRINT A : IF X THEN)
    invalid shapes (END IF X)
    label prefix violations (Z__ reserved)

✓ System constants injected:
    FALSE = 0
    TRUE  = -1

TRUE = -1 ensures compatibility with BASIC’s NOT/AND/OR behaviour.

✓ Options system:
    REM ##OPTION DEFINE ON|OFF
    REM ##OPTION REMPASS ON|OFF
    REM ##OPTION STRICT_ERR ON|OFF

------------------------------------------------------------
REPOSITORY & DOWNLOADS
------------------------------------------------------------

GitHub repository:
https://github.com/mrschmalenbach/Label-BASIC-Pro

Includes:
- full user guide
- hero example programs
- “good” test suite
- “evil” test suite

------------------------------------------------------------
STABILITY (RC1)
------------------------------------------------------------

RC1 is feature-complete for v1.0.  
Needs community testing, especially with:
- deep nesting
- large constants/enums
- complex colon placement
- error message validation
- BASLOAD edge cases

Performance:
- Fast for files under ~250 lines
- Longer files: noticeable pause (LB/BASIC V2 runtime speed)
- Future C/LLVM-MOS rewrite will greatly increase speed

------------------------------------------------------------
PLANNED FOR v1.5+
------------------------------------------------------------

- ELSEIF support
- Multi-statement structured-line handling
- Expanded error system
- Include files
- Macro system
- Better compression/expansion modes
- C rewrite for performance
- Symbol table export
- Improved REM mapping

------------------------------------------------------------
FEEDBACK WELCOME!
------------------------------------------------------------

Please share:
- bugs, crashes, malformed code
- missing features
- syntax suggestions
- programs you’ve ported to LBP
- test results using “good” or “evil” suites

Thank you for testing Label BASIC Pro.
– Martin
