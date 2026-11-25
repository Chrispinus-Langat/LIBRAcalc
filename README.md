# fx-991 Style Scientific Calculator

A scientific calculator modelled on the Casio fx-991 series, built as a **single self-contained HTML file**: HTML, CSS and vanilla JavaScript, with no external libraries, no network requests and no `eval()`. It works offline and on both phone and desktop screens.

**Developer:** Chrispinus Langat

## Getting started

1. Download `casio-calculator.html`.
2. Open it in any modern browser (Chrome, Edge, Firefox, Safari). No server or install is needed.
3. To confirm the engine works, open the browser console and run `selfTest()`. The checks also run automatically on page load.

## Features

### Calculate mode
- **Basics:** `+ − × ÷`, brackets, decimal point, `%`, sign change, `Ans`, AC and DEL.
- **Editing:** ◀ ▶ move the cursor so you can edit the middle of an expression. ▲ recalls the previous expression, and SHIFT+▲ moves back down.
- **Precedence:** `^` is right-associative and `-2^2 = -4`. Implicit multiplication works (`2π`, `3(4+5)`, `2sin(30)`) and binds tighter than division, so `1÷2π = 1/(2π)`. Missing closing brackets are closed automatically.
- **Errors:** malformed input gives `Syntax ERROR`. Invalid maths (divide by zero, log of a non-positive number, `tan(90)` in DEG, square root of a negative number, factorial of a negative or non-integer) gives `Math ERROR`.
- **Number display:** results are rounded to about 12 significant digits, so `0.1+0.2` shows `0.3` and `sin(180°)` shows `0`. Very large or small values use scientific notation with a superscript exponent.
- **Display modes:** SHIFT+DRG cycles Norm, FIX 2, FIX 4 and SCI.
- **Angle modes:** DEG, RAD and GRAD, with an on-screen indicator that applies to every trig function.
- **Memory:** M+, M−, MR (SHIFT+7) and MC (SHIFT+8), with an `M` indicator while memory is non-zero.
- **SHIFT and ALPHA:** SHIFT shows an `S` indicator and swaps every key to its yellow function. ALPHA gives variables A–F, X, Y and M. STO is SHIFT+RCL, followed by a variable key.
- **Scientific functions:**
  - Trigonometry: sin, cos, tan and their inverses.
  - Hyperbolic: sinh, cosh, tanh and their inverses (use the `hyp` key).
  - Logarithms and exponentials: log, ln, `logb(base, x)`, 10ˣ and eˣ.
  - Powers and roots: x², x³, xʸ, √, ∛, ˣ√ and x⁻¹.
  - Other: x!, nPr, nCr, Abs, Rnd, Ran#, π, e and ×10ˣ.
- **Fractions:** the `a b/c` key enters fractions (`1⌟2⌟3` is 1 and 2/3). S⇔D toggles between fraction and decimal results, and SHIFT+S⇔D switches between mixed and improper form.
- **Degrees, minutes, seconds:** the `°′″` key cycles ° ′ ″ as you type, and SHIFT+`°′″` converts the current value to DMS.
- **Conversions:** `Pol(x, y)` and `Rec(r, θ)`, which also store their results in X and Y.
- **History:** every calculation is listed in the history panel. Click an expression to reuse it, or a result to insert it.

### Mode menu (dropdown in the top bar)
Each mode opens its own panel in place of the keypad.

| Mode | What it does |
|---|---|
| **Statistics** | 1-variable and 2-variable data with an optional frequency column. Gives n, Σx, Σx², mean, population and sample standard deviation, min, max, median, Q1 and Q3. For 2-variable data it adds Σy, Σy², Σxy, ȳ, and a linear, quadratic, exponential, logarithmic or power regression with an estimate for any x. |
| **Equation** | Quadratic, cubic, and 2- and 3-unknown linear systems, with complex roots shown. Singular systems give `Math ERROR`. |
| **Complex** | Type expressions such as `(1+2i)×(3−i)`. Supports `+ − × ÷ ^`, implicit multiplication, `i`, `π`, `e`, `√`, `Abs`, `Arg`, `Conj`, `Re` and `Im`. Results appear in rectangular and polar form. |
| **Base-N** | Convert between DEC, BIN, OCT and HEX, with `+ − × ÷ AND OR XOR XNOR NOT NEG` on signed 32-bit values. |
| **Table** | Tabulates f(X) over a start, end and step, up to 100 rows. Use the letter `X` in the formula. |

### Interface
- Casio-style casing, solar-panel strip, and a two-line LCD with status indicators (S, A, M, hyp, DEG/RAD/GRAD, FIX, SCI).
- Distinct colours for number, operator and function keys, a yellow SHIFT key, a red ALPHA key, and separate AC and DEL keys. Each key shows its SHIFT and ALPHA labels above it.
- Large touch targets, a press animation, a light and dark theme toggle, and `aria-label` on every key.
- Memory, history, angle mode, display mode, theme and the selected mode are saved to `localStorage`. All access is wrapped in `try/catch`, so the calculator still works if storage is blocked.

## Keyboard shortcuts

| Key | Action |
|---|---|
| `0–9 . + - * / ( ) ^ ! % ,` | Insert the character |
| `Enter` or `=` | Evaluate |
| `Backspace` | DEL |
| `Escape` | AC |
| Arrow keys | Move the cursor (left and right) and recall history (up and down) |
| `s` `c` `t` | `sin(` `cos(` `tan(` |
| `l` `n` `r` | `log(` `ln(` `√(` |
| `p` `e` `a` | `π`, `e`, `Ans` |

Shortcuts are paused while a mode panel is open, so typing in its fields is not affected.

## How it works

The JavaScript is organised in clearly commented sections:

1. **State:** one object holds the expression, cursor, variables, modes and history.
2. **Numeric helpers:** 12-digit rounding, angle conversion, factorial, nPr and nCr.
3. **Function table:** every function with its arity and its Math ERROR checks.
4. **Tokenizer:** a longest-match scanner for numbers, function names, constants and operators.
5. **Parser:** a recursive-descent parser. Grammar, from lowest to highest precedence:
   ```
   expr  := term (('+'|'-') term)*
   term  := imp (('×'|'÷') imp)*
   imp   := perm (perm)*                      implicit multiplication
   perm  := unary (('nPr'|'nCr') unary)*
   unary := ('-'|'+') unary | pow
   pow   := post (('^'|'ˣ√') unary)?          right-associative
   post  := atom ('!'|'%'|'⁻¹')*
   atom  := number | constant | function | '(' expr ')'
   ```
6. **Formatting:** decimal, FIX, SCI, fraction and DMS output.
7. **Self-test:** the checks listed below.
8. **Editing and commands, keypad definition, rendering and persistence.**
9. **Mode menu:** statistics, equations, complex numbers, Base-N and table.

A separate small parser handles complex-number expressions in Complex mode.

## Self-test

Run `selfTest()` in the console. It reports pass or fail for 23 checks, including:

`sin(30)` in DEG = 0.5 · `5!` = 120 · `2^10` = 1024 · `0.1+0.2` = 0.3 · `-2^2` = -4 · `10 nCr 3` = 120 · `10 nPr 3` = 720 · `log(1000)` = 3 · `ln(e)` = 1 · `√(-1)` gives Math ERROR · `1÷0` gives Math ERROR · `5+` gives Syntax ERROR · `1÷3` as a fraction shows 1/3 · `sin⁻¹(0.5)` in DEG = 30 · `1÷2π` = 0.1591549431 · `6÷2(1+2)` = 1.

## Known limitations

- The calculator has been tested with scripted checks in Node.js, not yet in a range of real browsers and devices. Please report any visual or interaction problems.
- Complex numbers work only in Complex mode, not in Calculate mode.
- Base-N handles one operation on two values at a time and is fixed at 32 bits.
- Statistics data entry is a text box, not a table editor, and the data is not saved.
- RCL turns on ALPHA so you press a variable key, rather than opening a recall screen.
- Not included: calculus (∫, d/dx), matrices and vectors, a second table function g(x), Solve, and unit and constant tables.

## Credits

Developed by **Chrispinus Langat**.

Casio and fx-991 are trademarks of Casio Computer Co., Ltd. This project is an independent imitation and is not affiliated with or endorsed by Casio.