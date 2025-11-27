# LBP Example Programs

This folder contains example Label BASIC Pro (LBP) programs that demonstrate practical usage of the language and its structured constructs. These examples are intended to be readable, instructive, and representative of real-world usage.

Each `.lbp` file can be transpiled via LBP into LB, then tokenised by BASLOAD into standard BASIC V2.

---

## File List

### **ex_hero_client_tracker.lbp**
A practical, business-oriented example that simulates a simple **client activity tracker**.  
Demonstrates:
- CONSTANT/ENUM usage  
- Nested IF structures  
- WHILE loops  
- User interaction  
- A maintainable “business logic” flow  

Good for showing how LBP can support light productivity utilities on the Commander X16.

---

### **ex_hero_stardock_patrol.lbp**
A retro-scifi mini-scenario inspired by classic Trek-style patrol missions.  
Demonstrates:
- REPEAT/UNTIL loop  
- EXIT/CONTINUE LOOP  
- Random encounters  
- Structured control flow to manage states  
- Clean separation of logic using readable blocks  

Good for showing how LBP can power small interactive games.

---

### **ex_logging_and_rempass.lbp**
A minimal demonstration of:
- `REM ##OPTION REMPASS ON`
- `REM ##OPTION DEFINE ON`
- Logging syntax (if enabled in your build)
- How option lines appear in the output depending on settings  
- The effect of DEFINE on numeric constants and enums  

Useful for verifying behaviour of directives.

---

### **ex_demo_if_while_repeat.lbp**
A compact, readable demonstration of all the core structured constructs:
- Multi-line IF / ELSE / END IF  
- WHILE / WEND  
- REPEAT / UNTIL  
- Simple EXIT/CONTINUE  
- Clean indentation patterns  

Ideal as a beginner-friendly introduction to LBP’s structured control flow.

---

## How to Use

To transpile any example:

1. Launch the LBP transpiler:

```
    RUN
```

2. Enter the filename when prompted:

```
    ex_hero_client_tracker.lbp
```

3. Accept or override the default output filename.
4. Run BASLOAD on the generated `.bas` file.

Example:

```
    BASLOAD “ex_hero_client_tracker.bas”
    RUN
```

---

## Purpose of These Examples

These examples exist to:

- Show idiomatic LBP usage  
- Validate that the transpiler handles real-world structure  
- Act as reference material for new users  
- Provide quick manual regression tests  
- Demonstrate readability and maintainability compared to LB or raw BASIC V2  
