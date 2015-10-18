# EGSHEL Specification 0.1

FIRST DRAFT =^.^= 16 October 2015

##### EGSHEL (the Extensible Golfing Scripting High Efficiency Language) is an open source, highly extensible, turing-complete, stack-aware object-oriented language intended partially for golfing, but which can also be used in more useful circumstances as a 'real' scripting language, and which can be built upon and used for almost anything.

*The goal of this specification is to act as an initial guide for following near-future EGSHEL specifications. As such, some traditionally expected language features may appear "incomplete." This is most likely deliberate, as it will be easier to add to the language than to change and introduce further incompatibilities.*

---

## Formatting

### Whitespace

* Spaces are sometimes used to demarcate tokens; most times they are completely ignored and unnecessary.

  * Unnecessary whitespace will be warned by the parser, but not errored.


* For multiple tokens in a math operation, spaces are needed; for statements like `vsb` they are almost completely unnecessary.

* Newlines are mostly needed to denote separate commands; however non-EOL semicolons can serve the same purpose. Most builtins can in either mode be followed by maximum two uppercase hexadecimal digits in place of the usual operator to specify how many of the next characters of source are part of that command.
  ```
  vsbFprinted printed!!A
  ```
  Output:
  ```
  printed printed

  ```
  Because the `!!A` instruction deletes a nonexistent variable A, the program terminates.

* Indentation is primarily ignored except in loops and when using verbose mode.

* Lines which begin or end with semicolons have special compile-time meanings: `verbose n;` tells the compiler the next *n* lines are verbose or that lines in range *x* - *y* are verbose.

* A single line comment is always terminated by a newline, and, in Embedded Concise mode (detected by a entirely Concise block), always initiated by two spaces after the last instruction on a line:
  ```
  vsb'this is a print statement in verbose
  a{
    v'this is a concise print statement  -| this is a comment
    v'close quotes are unnecessary as long as a newline follows\; use an unescaped \; otherwise -| this will cause a syntax error
  }

  ```
* Ending a line in `...` or the Unicode ellipsis character (u2026) allows the line to be broken but continued on the next line. (Verbose only.)

### Comments

In Verbose mode, single line comments are begun by `-|`, and may occur either after a line of code, on a separate line, or following a line of code following a line separator (`...`).

All of the following are valid Verbose single-line comments:

```
a1 -| a = 1

a1... -| a = 1

a1b2...
-| a = 1, b = 2
```

Multi-line comments are begun and ended by `--|` and `|--`.

These are valid multi-line comments:

<pre><code>
&#42;a1 --| this is a long comment block
see, i have more comments here
and here
|-- vsb'this is still printed.
</code></pre>

```
lgv -inf; -| parser now reads from bottom of file to top
vsb a --| this line is read last
still more comment
also a comment
this is still a comment
a"hello world |-- start of multiline comment
```

### Files

#### File Creation

This specification recommends script files have either of the extensions `.egl` (pronounced *eagle*) or `.egs` (pronounced *eggs*), and while both Windows (CRLF) and Unix (LF) newline formats are permitted, Unix is preferred and optimised output files will use LF.

#### Parse Mode

##### Verbose mode:

  * First, the parser looks for a `verbose ...;`, `vrb ...;`, or `...;` (the `verbose` can be usually be omitted since it's inferred) statement within the first and last three lines of the file: if it is not found, execution continues in default verbose mode.

  * If there are more than two semicolon-terminated lines in the file, (Note that verbose mode will execute nearly all concise code but not the inverse).

  * If `...` is a space separated list of options or an undelimited string of lowercase ASCII alphanumeric characters...
    * containing a base10 or base16 integer *n*, only *n* lines below or above (respective to whether the statement is at the top or bottom) will be read.

    * containing a base10 or base16 integer *n* which begins with a hyphen (`-10`, `-3Ah`) and is at the top of the file, the file will be parsed bottom to top. Lines will be executed left to right, as normal.

    * containing exactly two base10 or base16 integers *x* and *y* separated by a hyphen (`10-0Fh`), then that range of lines will be read. If the result of the second number subtracted from the first would result in a negative number, that range of lines will be read upwards starting at the bottom rather than downwards.

    * containing the string `prcd`, `prc` or `p` (precedence) then, provided another semicolon-terminated statement at the end of the file is present, this "side" of the file is granted execute precedence and will always have a priority higher than another statement.

    * containing the string `inf` or the letter `i` (infinite) then the entire file will be read directionally in the implied direction as if it were omitted; if `inf<` or `i<` are present then the file will be read top to bottom, then bottom to top.

  ```
  pida; -| this semicolon still counts as EOL even with comment
  'a executed thrice
  vsb a
  vsb'hai world!
  vsb *a2
  'a executed twice
  -2;
  ```
  Output:
  ```
  executed thrice
  hai world!
  executed thrice
  executed thrice
  executed twice
  executed twice
  ```

##### Concise mode:
(Note that this mode is the default and this line is only necessary if a different reading mode is desired.)
  * The parser looks for `<options>;` statements at the beginning or end of the file, where `<options>` may be, in order:
    * `p`: must be used with `i`; denotes `precedence`
    * `i`: short for `inf`; reads all of the given file in the optionally given direction. (see example.) Combine with `_` for `-`.
    * `n` takes the place of `i` where *n* is =< 3 hex digits, to read next *n* chars, or with a leading `_` to only read in reverse the last *n* characters.
    * `d`: can be `o`, `h`, `d` or `b`; explicit respective output base for integers of any given type. Extensiblity allows other bases to be added later.
    * `s`: `strct`: makes all reserved names (builtins) unmutable (non-reassignable) and causes the parser to ignore techincally incorrect syntaxes while allowing the program to continue execution without exiting fatally. Omission of this switch is known as `lazy`.
    * `a`: asynchronous: recommended for this application of `strct`; allows bidirectional program execution to be different for different directions. Omission of this flag will throw more errors on bidirectional noncompliant code.
  * As above, if none are found, the program is run as normal.

  Source: (22 bytes)
  ```
  pi<Fdsa;{n:v:0210:v:n}
  ```
  Output:
  ```
  0
  1
  2
  3
  4
  ...
  136
  0
  1
  2
  3
  4
  ...
  80
  (repeated 15 more times)
  ```
  Explanation:

  `pi<` sets precedence (where to start) to left and states that when the right side of the program is reached, go left again (`<`), then repeat this `iF(h)` (0x0F, or 15) *more* (non-inclusive) times
  `dsa;` flags in order: `d`: converts all octal, hex and bin numbers to decimal before printing; `s`: similar to javascript's 'use strict'; this makes all reserved names unmutable and causes the parser to ignore techincally incorrect syntaxes while allowing the program to continue execution; `a`: in the case of bidirectional `i`, allow asynchronous behaviours.
  `{n:v:0210:v:n}` because v is both Nil (as a variable) and a reserved name (as both a function and var), it is ignored on the relative-first side<br>

  We can expand this line as a whole into pseudocode:
  ```
    while repeats < 0xF:
      for n in range(oct(0210)) and o in range(oct(0120)):
        print n
        print o
      repeats += 1
  ```

#### Ignored Characters
Characters not explicitly part of EGSHEL will be ignored, especially if `strct` is specified. This makes writing polyglots with EGSHEL quite easy.

---

## Variables

### Scope

If a variable is uninitialised before its first reference, its type is automatically cast to NilType and thus its value before being set is Nil.
Uninitialised variables are local to their initialising scope, unless explicitly defined with `gbl` or `pbl`.
Variables initialised with `gbl var` or `pbl var` are completely global regardless of defining scope, and any scope may read or write to or from them without dropping their global status.
Variables initialised with `lcl var` or `prv var` are completely local to the defining scope and even child scopes do not have access to them.
Variables may be referenced above their definition in the enclosing scope and still be defined but uninitialised variables may not, and referencing uninitialised variables will only return NilType.

### Naming

Variable and function names may not contain the digits 0-9, punctuation or other EGSHEL operators; i.e., they are restricted to alphabeticals.
In Concise mode, identifiers for single-letter variables must be in uppercase. In Verbose mode, because lowercase alphabetical letters are not explicitly reserved but can become such if Concise syntax is detected in the enclosing scope, using lowercase single-letter variables is not recommended. Variable and function identifiers are case-sensitive, but this can be toggled in Verbose mode with the `wcs` (watch-case) switch.

### Declaration and Assignment

#### Concise Mode
In Concise mode, variables can be assigned in a few ways: (note that typecasting is automatic, implicit and inferred, but is still manually possible.)
```
A1 -| varnames may not contain numbers, so this assigns a value of dec(1) to a.
A40h -| assigns hex 040 (dec 64) to A
B,A -| copy A into B
B,A1 -| copy 1 into both A and B
A'hello world -| assign the string "hello world" to A
gb_nt C;B*,C"A -| declare C as being global and Nil; copy the value of A into B and cast B's type to int, which will "techincally" fail and cast B as an Untyped (but not a NilType) Bytearray, (type:ub,scope:all); and copy the value of A into C as a raw string.
C")t -| regardless of previous type, set C to a string equal to its own previous type
1~C -| set C to whatever it was before it was last set. If C has not been set, cast NilType to C and assign it the value Nil.
2~C"A -| set the 2nd history index of C to a copy of A which has had its type cast to string
AB~C -| set A to the Bth history index of C
A=*2 2 -| set A to 2 * 2 = 4
B,A[1~A,1~B,C] -| set B and A each equal to an array containing copies of the immediately previous versions of themselves, and C
A,B[X,Y]N -| set A and B equal to the respective indicies X and Y of array N
A['hello,'world] -| set A to an array containing two values: [0]A as string "hello" and [1]A as string "world"
v*~C -| print a newline separated list of every previous value of C starting with Nil
A`v*2 7` -| define A as a macro which prints 2 7* (14) twice, separated by a space, newline or nothing: this is a simple function in Concise.
B"A -| take A's string literal macro and copy it into B as a string, not a callable function
B=A -| copy the macro A into B
C~|B -| make C a listener to B - that is, its value is always identical to that of B at a given time. if C and B have different scope access, C's attributes and scope is preserved as what it was in 1~C. (note that this is only bidirectional if |~| is used instead.)
$C{`~C`:vI~C -| define C as a for statement which prints each previous value of C in a list
...
```
Note that in different cases does the `<int><tilde><var>` (`1~C` format) have different meanings: when the statement contains exactly one variable, the variable gets assigned to a previous versions of itself; when other variables are present in the equation then that index of the var is cast to the other variables.

---

## Types

The variable types that LOLCODE currently recognizes are: strings (YARN), integers (NUMBR), floats (NUMBAR), and booleans (TROOF) (Arrays (BUKKIT) are reserved for future expansion.) Typing is handled dynamically. Until a variable is given an initial value, it is untyped (NOOB). ~~Casting operations operate on TYPE types, as well.~~

### Untyped

The untyped type (NOOB) cannot be implicitly cast into any type except a TROOF. A cast into TROOF makes the variable FAIL. Any operations on a NOOB that assume another type (e.g., math) results in an error.

Explicit casts of a NOOB (untyped, uninitialized) variable are to empty/zero values for all other types.

### Booleans

The two boolean (TROOF) values are WIN (true) and FAIL (false). The empty string (""), an empty array, and numerical zero are all cast to FAIL. All other values evaluate to WIN.

### Numerical Types

A NUMBR is an integer as specified in the host implementation/architecture. Any contiguous sequence of digits outside of a quoted YARN and not containing a decimal point (.) is considered a NUMBR. A NUMBR may have a leading hyphen (-) to signify a negative number.

A NUMBAR is a float as specified in the host implementation/architecture. It is represented as a contiguous string of digits containing exactly one decimal point. Casting a NUMBAR to a NUMBR truncates the decimal portion of the floating point number. Casting a NUMBAR to a YARN (by printing it, for example), truncates the output to a default of two decimal places. A NUMBAR may have a leading hyphen (-) to signify a negative number.

Casting of a string to a numerical type parses the string as if it were not in quotes. If there are any non-numerical, non-hyphen, non-period characters, then it results in an error. Casting WIN to a numerical type results in "1" or "1.0"; casting FAIL results in a numerical zero.

### Strings

String literals (YARN) are demarked with double quotation marks ("). Line continuation and soft-command-breaks are ignored inside quoted strings. An unterminated string literal (no closing quote) will cause an error.

Within a string, all characters represent their literal value except the colon (:), which is the escape character. Characters immediately following the colon also take on a special meaning.

* :) represents a newline (\n)
* :> represents a tab (\t)
* :o represents a bell (beep) (\g)
* :" represents a literal double quote (")
* :: represents a single literal colon (:)

The colon may also introduce more verbose escapes enclosed within some form of bracket.

* :(`<hex>`) resolves the hex number into the corresponding Unicode code point.
* :{`<var>`} interpolates the current value of the enclosed variable, cast as a string.
* :[`<char name>`] resolves the `<char name>` in capital letters to the corresponding Unicode [normative name](http://www.unicode.org/Public/4.1.0/ucd/NamesList.txt).

### Arrays

*Array and dictionary types are currently under-specified. There is general will to unify them, but indexing and definition is still under discussion.*

### Types

The TYPE type only has the values of TROOF, NOOB, NUMBR, NUMBAR, YARN, and TYPE, as bare words. They may be legally cast to TROOF (all true except for NOOB) or YARN.

*TYPEs are under current review. Current sentiment is to delay defining them until user-defined types are relevant, but that would mean that type comparisons are left unresolved in the meantime.*

---

## Glossary

### Synonyms

* `gbl` and `pbl` are synonymous.
  * In Concise mode, use `gb` and `pb` instead.
<br>
* `lcl` and `prv` are synonymous.
  * In Concise mode, use `lc` and `pv` instead.
<br>
* "Initialise" and "declare" are synonymous.
<br>
* "Defining scope," "initialising scope," and "enclosing scope" are all synonymous.
<br>

### Definitions

* **Semicolon-terminated:**
  A line which ends in a semicolon, legally or otherwise.
  ```
  verbose precedence inf; -| this line is legally semicolon-terminated
  vsb'blah blah -| this line is illegally semicolon-terminated
  v'hello world;q -| this line contains a legal semicolon
  -2; -| this line is legally semicolon-terminated
  ```

### Lists

#### List of Builtins

##### Concise Control Structures


----------
