# r3note

This is kiy's note on Pablo Hugo Reda's r3 forth language.
2025-10-01 - 2025-10-09

Taken mainly from 
	**A Concatenative Language Derived from ColorForth** 
	*with English Translation and Corrections*
	Pablo H. Reda - 2025*
	pabloreda@gmail.com
	Repository: https://github.com/phreda4/r3

## Why

I have a project that consists mostly of numerical calculation, not that heavy-weight, and hope it to be portable to resource-constrained devices.
A natural choice of the language for implementation would be ANSI C, but I don't find programming in C enjoyable.
So I looked for a simple language and came across R3 while considering Charles Moor's ColorForth.

### Pros

- **Pablo Hugo Reda**: helpful.
- **Curiosity**: R3 is inspired by ColorFORTH.
- **Portabiliry**: The virtual machine (VM) is written in C++ and R3 itself.
- **Fixed point**: 48.16 fixed point in place of floats.
- **Typeless**: write less.

### Cons

- **Documentation**: Scarce, scattered, and outdated. 
- **Low numerical accuracy**: No floating point. *This is my biggest concern.*
- **No REPL**: FreeForth2-style REPL should be possible.
- **Case insensitive**: This may be an advantage.
- **Windows libraries**: Tend not to work under Linux.

## Hello world

Suppose the executable file `r3` is in a directory. In the same directory, make a text file `hw.r3` containing

```forth
^r3/editor/code-print.r3
: "Hello world!" .println ;
```

Typing

```
./r3 hw.r3
```

in the said directory produces

```
Hello world!
```
on the screen. 
The first line in `hw.r3` includes (`^`) the file in which `.println`  is defined.
The second line tells the VM to define (`:`) a nameless word that prints "Hello world!" as  a line (`.println`), execute it, and stop execution (`;`).

- **`:f ...`** without a blank between `:` and `f` , with or without a semicolon `;` at the end, defines `f`.
- **`:f ... ;`** without a blank between `:` and `f` , with a semicolon `;` at the end, defines `f` which returns to the caller after being called.
- **`: f ;`** with a blank between colon `:` and `f` calls `f` and then stops execution.

## Dictionary

The VM comes with a predefined dictionary. 
Based on the primitives, new words are defined with sigils `:` and `#`.
The new words are added to the dictionary.

Programming in r3 consists of defining new words from old words.
The VM searches the dictionary from the latest to the oldest. 
Words with the same name can be defined; the VM finds only the last defined word of the same name.

The inclusion sigil `^` takes the indicated text file.
The VM adds to the dictionary all words marked to be *export*ed: 
**`::`** for code and **`##`** for data.
Other words defined by single `:` or `#` are local to the file. 

### Example

```forth
#side 5          | line 1
:square dup * ;  | line 2
: side square ;  | line 3
```
Line 1 defines a variable named `side` with value 5.  
Line 2 defines an anonymous word that duplicates the top of stack and multiplies these two numbers.  
Line 3 is where the program execution begins: 
pushes 5 (the value of variable 'side'), then calls square.

At the end of the program, the stack will have the number 25.

- The semicolon indicates not that the word definition ends there but that execution terminates there.
  A word can have multiple end points and even no end, continuing to the next defined word.

## Parsing

- The VM reads the source code word by word.
- The words are separated by  `whitespace characters` = {space, tab, newline}
- The word names are case insensitive: `wordname` is the same as `WORDNAME` or `WordName`.

1. If the word is a number, it is pushed onto the data stack and VM parses the next word.
2. Otherwise, the VM looks for the word in the dictionary. If the word is found, it is executed.
3. Otherwise, the VM checks the word's first character.
    If the word begins with any of the 8 characters in  `sigils` = {`|`, `^`, `"`, `:`, `#`, `$`, `%`, `'`} , 
    then the word is interpreted accordingly, as listed below.
4. Otherwise, the program terminates indicating an error with its location

| Prefix | Meaning | Example | Description |
|--------|---------|---------|-------------|
| `|` | Comment | `| This is a comment` | Not executed to the line end |
| `^` | Include | `^r3/lib/console.r3` | Include code from a file |
| `"` | String | `"Hello"` | Define text string ending with `"` |
| `:` | Action | `:wordname` | Define action |
| `#` | Data | `#varname 5` | Define variable |
| `$` | Hexadecimal | `$FF` | Hexadecimal number |
| `%` | Binary | `%1010` | Binary number |
| `'` | Address | `'wordname` | Address of a word |

- **Primitives** are words defined in the virtual machine.
- `'` doesn't work for primitives since they have no addresses. 
- Generally it is a bad idea to define a word the name of which consists of a single sigil, but this can be done with respect to some of the sigils.

## Data Stack

- Top-of-stack (TOS) is the 0th element on the stack and next-of-stack (NOS) is the 1st element on the stack.
- Stack is pictured with older elements to the left, newer elements to the right: `x[n] x[n-1] .. x[1] x[0]`.
- Stack effect is illustrated as `old[m] .. old[0] -- new[n] .. new[0]`.

| Word | Stack Effect | Description |
|------|--------------|-------------|
| `DUP` | `a -- a a` | Duplicate TOS |
| `DROP` | `a --` | Remove TOS |
| `OVER` | `a b -- a b a` | Duplicate NOS |
| `PICK2` | `a b c -- a b c a` | Duplicate 3rd element |
| `PICK3` | `a b c d -- a b c d a` | Duplicate 4th element |
| `PICK4` | `a b c d e -- a b c d e a` | Duplicate 5th element |
| `SWAP` | `a b -- b a` | Exchange TOS and NOS |
| `NIP` | `a b -- b` | Remove NOS |
| `ROT` | `a b c -- b c a` | Rotate 3 elements |
| `-ROT` | `a b c -- c a b` | Rotate 3 elements |
| `2DUP` | `a b -- a b a b` | Duplicate 2 values |
| `2DROP` | `a b --` | Remove 2 elements |
| `3DROP` | `a b c --` | Remove 3 elements |
| `4DROP` | `a b c d --` | Remove 4 elements |
| `2OVER` | `a b c d -- a b c d a b` | Duplicate 2 elements from 3rd position |
| `2SWAP` | `a b c d -- c d a b` | Exchange 4 elements |

- Since the names are historical, simpler names are possible but with execution penalty.

```forth
| Consistent names  
:_4  pick4 ;
:_3  pick3 ;
:_2  pick2 ;
:_1  over  ;  :pick1 over ;
:_   dup   ;  :pick0 dup  ;
:_-  drop  ;  :2_- 2drop ;  :3_- 2_- _- ;  :4_- 2_- 2_- ;
:_1- nip ;
:_2- rot drop ;
:_3- >r rot drop r> ;
:_4- >r >r rot drop r> r> ;
:~   swap  ;  :2~  2swap ;
:~!  swap ! ;
```

- R3 does not check stack underflow.
- Upon stack overflow the VM freezes or stops.

Pablo says, 251009:
> la pila es solamente variables temporales y pasaje de parámetros, no una estructura de datos, Por eso no existe pick con parámetros.. el acceso esta limitado.. si necesitas acceder mas abajo, algo anda mal en el diseño, seguramente se puede simplificar.
Esto permite análisis estático, puedo calcular que hace con la pila sin ejecutar el código.

It might mean that the data stack is actually a small number of registers; I don't know.

## Return Stack

| Word | Stack Effect | Description |
|------|--------------|-------------|
| `>R` | `a -- rstack: -- a` | Push to return stack |
| `R>` | `-- a rstack: a --` | Pop from return stack |
| `R@` | `-- a rstack: a -- a` | Read top of return stack |

Stores the return address used whenever `;` is executed.

When a word is called (a code definition, not data), the VM pushes onto the return stack the location where it should return once the called word's execution terminates.

- Try to avoid using this stack since an imbalance between calls will cause the code to break. 
- It can be used as a place to save or retrieve values. Balance `>r` and `r>` carefully.
- It is possible to alter this stack to change execution flow, but it is dangerous.

## Arithmetic

### Unary

| Word | Stack Effect | Description |
|------|--------------|-------------|
| `NEG` | `a -- -a` | Negate value |
| `ABS` | `a -- |a|` | Absolute value |
| `SQRT` | `a -- sqrt(a)` | Square root |
| `CLZ` | `a -- n` | Count leading zeros |

### Addition and Subtraction

| Word | Stack Effect | Description |
|------|--------------|-------------|
| `+` | `a b -- c` | c = a + b |
| `-` | `a b -- c` | c = a - b |

### Multiplication and Division

| Word | Stack Effect | Description |
|------|--------------|-------------|
| `*` | `a b -- c` | c = a * b |
| `/` | `a b -- c` | c = a / b |
| `MOD` | `a b -- c` | c = a mod b |
| `/MOD` | `a b -- c d` | c = a/b, d = a mod b |
| `*/` | `a b c -- d` | d = a*b/c without bit loss |

## Logical

| Word | Stack Effect | Description | Example |
|------|--------------|-------------|-------|
| `AND` | `a b -- c` | c = a AND b | `$ff $55 AND` → $55 |
| `NAND` | `a b -- c` | c = a NAND b | `$2 $1 NAND` → 0 |
| `OR` | `a b -- c` | c = a OR b | `$2 $1 OR` → 3 |
| `XOR` | `a b -- c` | c = a XOR b | `$3 2 XOR` → 1 |
| `NOT` | `a -- b` | b = NOT a | `0 NOT` → -1 |

## Shift

| Word | Stack Effect | Description |
|------|--------------|-------------|
| `<<` | `a b -- c` | Left bit shift |
| `>>` | `a b -- c` | Right bit shift |
| `>>>` | `a b -- c` | Right bit shift without sign |
| `*>>` | `a b c -- d` | d = (a*b)>>c without bit loss |
| `<</` | `a b c -- d` | d = (a<<c)/b without bit loss |

```forth
5 2 <<    | pushes 20 since in binary 101 becomes 10100
5 1 >>    | pushes 2 since in binary 101 becomes 10
-2 1 >>   | pushes -1 since in binary 1111 ... 1111 1101 becomes 1111 ... 1111 1110
-1 1 >>>  | pushes 9223372036854775807 since in binary 1111 ... 1111 1110 becomes 0111 ... 1111 1111
```

## Fixed Point

Numbers with decimal points are recognized as 48.16 fixed point numbers: 48 bits for integer part, 16 bits for fractional part. 
Avoids floating point complexity.
Makes the VM easily portable to devices with no floating point. 

With these numbers, addition and subtraction are the same as integers, but multiplication and division need words defined in `r3/lib/math.r3`:

| Word | Stack Effect | Description |
|------|--------------|-------------|
| `int.` | `f -- n=floor(f)` | fixed point to number |
| `cos` | `f -- cos(f)` |  |
| `sin` | `f -- sin(f)` |  |
| `tan` | `f -- tan(f)` |  |
| `sqrt.` | `f -- sqrt(f)` | |
| `exp.` | `f -- exp(f)` |  |
| `ln.` | `f -- log_e(f)` | Low accuracy. As of 2025-10-06, `1.0 ln.` gives 0.0149. |
| `*.` | `f g -- f*g` | multiply |
| `/.` | `f g -- f/g` | divide |
| `root.` | `f g -- f**(1/g)` | 16.0 2.0 root.  gives  4.000 |

## Registers

R6 VM has two registers, `A` and `B` .
X := A xor B

| Size | Load | Push | Add | Fetch | Store | `@` and +size | `!` and +size | Name |
|:----:|:-----:|:-----:|:-----:|:----:|:----:|:---:|:--:|----|
| 8 bits | - | - | - | `cX@` | `cX!` | `cX@+` | `cX!+` | byte or char |
| 16 bits | - | - | - | - | - | - | - | word |
| 32 bits | - | - | - |`dX@` | `dX!` | `dX@+` | `dX!+` | dword |
| 64 bits | `>X` | `X>` | `X+` | `X@` | `X!` | `X@+` | `X!+` | qword (default) |

- Registers A and B are implemented with hardware registers, hence faster in execution.
- When using a register, save and restore its original value to avoid destruction.

## Memory

The default **cell** size is 64 bits, but other sizes are supported. See the section on *memory* .

| Size | Fetch | Store | `@` and +size | `!` and +size | Name |
|------|-------|-------|------|------|-------------|
| 8 bits | `c@` | `c!` | `c@+` | `c!+` | byte or char |
| 16 bits | `w@` | `w!` | `w@+` | `w!+` | word |
| 32 bits | `d@` | `d!` | `d@+` | `d!+` | dword |
| 64 bits | `@` | `!` | `@+` | `!+` | qword (default) |

## Memory Block

| Word | Stack Effect | Description |
|------|--------------|-------------|
| `MOVE` | `d s c --` | Copy `s` to `d`, `c` qwords |
| `MOVE>` | `d s c --` | Copy `s` to `d`, `c` qwords in reverse |
| `FILL` | `d v c --` | Fill `d`, `c` qwords with `v` |
| `CMOVE` | `d s c --` | Copy `s` to `d`, `c` bytes |
| `CMOVE>` | `d s c --` | Copy `s` to `d`, `c` bytes in reverse |
| `CFILL` | `d v c --` | Fill `d`, `c` bytes with `v` |
| `DMOVE` | `d s c --` | Copy `s` to `d`, `c` dwords |
| `DMOVE>` | `d s c --` | Copy `s` to `d`, `c` dwords in reverse |
| `DFILL` | `d v c --` | Fill `d`, `c` dwords with `v` |
| `MEM` | `-- a` | Start of free memory |

## System Interface

| Word | Stack Effect | Description |
|------|--------------|-------------|
| `LOADLIB` | `"name" -- liba` | Load dynamic library |
| `GETPROC` | `liba "name" -- a` | Get function address |
| `SYS0` | `a -- r` | Call function with 0 parameter |
| `SYS1` | `p0 a -- r` | Call function with 1 parameter |
| `SYS2` | `p0 p1 a -- r` | Call function with 2 parameters |
| ... | ... | ...|
| `SYS10` | `p0 p1 p2 p3 p4 p5 p6 p7 p8 p9 a -- r` | Call function with 10 parameters |

## Execution Control

| Word | Stack Effect | Description |
|------|--------------|-------------|
| `;` | `--` | End word execution |
| `(` | `--` | Begin code block; needs a conditional befor or after |
| `)` | `--` | End code block; needs a preceding `)` |
| `[` | `-- vec` | Begin anonymous definition |
| `]` | `vec --` | End anonymous definition |
| `EX` | `vec --` | Execute word by address |

- `[ ... ]` is quotation without immediate execution.

```forth
:**2 dup * ;
: '**2 ex ; 
```
is the same as
```forth
: [ dup * ; ] ex ;
```

- *Condition*s with *code block*s `( .. )` build *control structure*s.
- There are two types of control structures:
	- conditionals
	```forth
	condition ( code )
	```
	in which the code is executed only if the condition gives True, and
	- repetitions
	```forth
	( condition code )
	```
	in which the code  is executed if the condition gives True and then the control loops back to `(`.

So if a condition comes immediately before `(` then the combination is a conditional while if a condition comes immediately after `(` then the combibnation is a repetition.

## Condition

- There are two types of conditions, TOS conditions and *comparison*s of TOS against NOS.
- A condition is either **True** or **False** recognized by the VM but *no result is left on the data stack*.

### TOS Conditions

These condition words check the top-of-stack (TOS) without modifying it:

| Word | Stack Effect | Description |
|------|--------------|-------------|
| `0?` | `a -- a` | True when a = 0 |
| `1?` | `a -- a` | True when a ≠ 0 |
| `-?` | `a -- a` | True when a < 0 |
| `+?` | `a -- a` | True when a ≥ 0 |

### Comparisons of TOS Against NOS

These compare top-of-stack (TOS) with next-of-stack (NOS), *consuming only TOS*:

| Word | Stack Effect | Description |
|------|--------------|-------------|
| `=?` | `a b -- a` | True if a = b |
| `<?` | `a b -- a` | True if a < b |
| `<=?` | `a b -- a` | True if a ≤ b |
| `>?` | `a b -- a` | True if a > b |
| `>=?` | `a b -- a` | True if a ≥ b |
| `<>?` | `a b -- a` | True if a ≠ b |
| `AND?` | `a b -- c` | True if a AND b |
| `NAND?` | `a b -- c` | True if a NAND b |
| `IN?` | `a b c -- a` | True if a≤b≤c (removes TOS and NOS) |

- 0?`, `1?`, `+?`, `-?` don't consume TOS.
- Others don't consume NOS.
- No result (True or False) is left on TOS.

```forth
:=<? <=? ; | alias
```

## Conditional

A conditional is built with a condition word followed by a code block that executes only if the condition is met:

```forth
: 3
	4 >? ( "Greater than 4" .print )
	4 <? ( "Less than 4" .print )
	drop ;
```

Line 2 asks if the NOS = 3 is greater than TOS = 4; if so prints the text.  The NOS is not consumed, so conditions can be chained as in line 3.

### Early Exit

```forth
:min | a b -- c
    over >? ( drop ; ) nip ;
```

### ELSE

ELSE is not provided suggesting factorization.

```forth
| Instead of: A ?? ( B ) else ( C ) D
| Use:
:condition ?? ( B ; ) C ;
A condition D
```

### SWITCH or CASE

**For sequential integers, use jump tables:**

```forth
:a0 "action 0" ;
:a1 "action 1" ;
:a2 "action 2" ;
:a3 "action 3" ;
:a4 "action 4" ;

#list 'a0 'a1 'a2 'a3 'a4

:action | n -- string
    3 <<        | 8 * (cell size)
    'list + @ ex ;
```

**For non-sequential values, use comparison chains:**

```forth
:cases | value -- value string
    5 <? ( "less than 5" ; )
    6 =? ( "is 6" ; )
    7 =? ( "is 7" ; )
    111 <? ( "between 8 and 110" ; )
    "greater or equal to 111" ;
```

- **Stack balance**: make sure that all branches produce the same stack behavior (unless intentionally different).

## Repetition

When the conditional is inside the block, a repetition is constructed. While this condition is met, the block repeats. When false, execution jumps to the next word after the block.

```forth
( code to be executed conditionally )
```
### Countdown

Conditionals that don't consume stack execute faster than those that do. Zero is the preferred end marker:

```forth
10 ( 1? 1- ) drop
```

### Counting Up

```forth
: 1 ( 10 <?
    dup "%d " .print
	1+ ) drop ;
```

This code prints numbers 1 to 9. When TOS becomes 10 on line 2, it jumps to line 3 after the closing parenthesis.

### Nested Loops 

```forth
| Multiplication table  9 x 9
:.tab 9 .emit ; | 9 = ASCII tab
:.2digits 10 <? ( " %d" .print ; ) "%d" .print ; | n --
:.*table
  0 ( 9 <? 1+                     | i
    0 ( 9 <? 1+                   | i j
      2dup * .2digits .tab        | i j
    ) drop .cr                    | i
  ) drop .cr ;                    |
: .*table ;
```
Some may find the `: 'table ... ;` part convoluted and prefer a factored version shown later in conjunction with *tail optimization*.


### Memory Traverse

When traversing memory, it's practical to have a termination marker, usually 0:

```forth
"hello" ( c@+ 1?
    use_each_character ) 2drop
```

- When exiting the loop, we have NOS = address and TOS = 0 .

Using a count would require:

```forth
"hello" 4 ( 1? 1- swap c@+
    use_each_character swap ) 2drop
```

### Multiple Exit

Valid to perform multiple comparisons to exit the loop:

```forth
( c@+ 1?
    13 <>?
    drop )  | 0 or 13
```

- Each loop exit must leave the same stack depth.

### Recursion

Recursion is built by a word calling itself.

```forth
| Fibonacci numbers f(n-2) + f(n-1) =: f(n)
:fibonacci | n -- f
	2 <? ( 1 nip ; )
  1- dup 1-                     | n-1 n-2
	fibonacci swap fibonacci + ;
```
- Watch the termination condition and stack state when the execution terminates.

### Tail Optimization

When a word calls itself immediately before its end of execution `;` , the execution loops back to the beginning of the word. Namely, if the last word calls itself, the recursive call  becomes a simple jump back:

```forth
:loopback | n -- 0
	0? ( ; ) 1- loopback ;
```

This is more efficient than the usual recursion, hence known by a special name  **tail optimization**. 
This can be used to factor loops out. 
For instance, the `:*table ... ;` part of the multiplication table example can also be written this way:

```forth
| Multiplication table factored
:.i*j 2dup * .2digits .tab ;            | i j -- i j   | print entry i*j
:.j   9 >=? ( drop ; ) 1+ .i*j     .j ; | i j -- i j+1 | print a row
:.i   9 >=? ( drop ; ) 1+ 0 .j .cr .i ; | i -- i+1     | print rows
:.*table-factored 0 .i .cr ;
: .*table-factored ;
```

## Memory

Variables define memory for data storage. Variables have a name, a memory address, and a value stored at that memory location. The actual address where each variable is located is obtained when executed - it's not necessary to know this address value, just use its name to represent it.

```forth
#lives 3               | line 1
#positionX #positionY  | line 2
#map * $400  | 1KB     | line 3
#list 3 1 4            | line 4
#energy 1000           | line 5
```

- `#name *` syntax means to allot memory in bytes: `#var 0` is the same as `#var * 8`;
the `*` here is not multiplication.

Using hex address $1000 as an example, this code reflects in memory as:

| Line | Name | Address | Value |
|------|------|---------|--------|
| 1 | lives | $1000 (4096) | 3 |
| 2 | positionx | $1008 (4104) | 0 |
| 2 | positiony | $1010 (4112) | 0 |
| 3 | map | $1018 (4120) | 0 0 0 ... 0 |
| 4 | list | $1418 (5144) | 3 1 4 |
| 5 | energy | $1430 (5168) | 1000 |

### Access

| Word | Stack Effect | Description |
|------|--------------|-------------|
| `!` | `value address --` | STORE - write value to address |
| `@` | `address -- value` | FETCH - read value from address |
| `+!` | `val address --` | Add val to value at address |
| `!+` | `value address -- address+8` | Store and increment address |
| `@+` | `address -- address+8 value` | Fetch and increment address |

- **Memory layout**: each number in memory is stored in 8 byte *cell*s (64 bits). If there's a sequence of numbers, they will be at this distance apart.

### Usage

```forth
:listshow
    'list
    @+ "%d " .print  | prints 3
    @+ "%d " .print  | prints 1  
    drop ;

'list 8 + @         | pushes 1
1 'positionX +!     | add 1 to positionX
listshow            | prints 3 1

5 6                 | 5 6
'list               | 5 6 $1418
!+                  | 5 $1420
!                   |
listshow            | prints 6 5
```

### Buffer

```forth
| Memory buffer with pointer
#buffer> 'buffer

:+element | element --
    buffer> !+ 'buffer> ! ;

:traverse
    'buffer ( buffer> <?
        @+ "%d " .println
    ) drop ;

3 +element
4 +element  
5 +element
traverse | prints 3 4 5
```

## Text

- The text sigil is `"`;  ends with another double quote.
- Text is implemented as byte array ending with a 0.
- Text handling is the same as memory handling, but the unit is bytes instead of qwords (8 bytes).
- No UTF-8.

### String Examples

```forth
"example text"
" example text "  | with spaces
"Say ""HELLO"" to everyone"  | embedded quotes
"Mac US keyboard:  ¿ = shift-alt-?  i ≠ ¡ = rightalt-1 !"
```

- Text is stored in separate memory area from variables unless the text is in a variable definition.

```forth
#lives 3
#text "BEWARE of the dog"
#dogs 4
```

### Character-by-Character Processing

```forth
^r3/editor/code-print.r3
:.ascii | t --
		( c@+ 1? "%d " .print ) 2drop ;
"AB" .ascii waitEsc .cr | 65 66
		| waitEsc waits for ESCape
```

### Formatted Output with `.print`

The `.print` word (^r3/editor/code-print.r3) processes text with % placeholders:

| Format | Description |
|--------|-------------|
| `%d` | Print number in decimal |
| `%b` | Print number in binary |
| `%h` | Print number in hexadecimal |
| `%s` | Print text from address |
| `%%` | Print % sign |

```forth
253 254 255 "%d %b %h" .print | 255 11111110 fc
```
- `.print` consumes TOS.

```forth
| Shorthands
^r3/editor/code-print.r3
:.pr .print ;  :. "%d " .print ;  :.. "%f " .print ;  :.ln .println ;
:_.  _ . ;  :@.  @ . ;  :@..  @ .. ;
```

### String Processing Libraries

Useful definitions for text handling are in libraries `r3/lib/str.r3` and `r3/lib/parse.r3`.

To count characters in text:

```forth
::count | s1 -- s1 cnt
    0 over ( c@+ 1? | until 0 byte is found
        drop swap 1+ swap ) 2drop ;
```

## Register Usage

Regiters A  and B are convenient for traversing memory, to read or write values.

### Find the Minimum Values of Two Lists

```forth
#list1 * $ffff  | 64kb
#list2 * $ffff  | 64kb  
#minimum

:update  a@+ minimum <? ( 'minimum ! ; ) drop ; | use register A

'list1 >a | A has list1
	a@ 'minimum !  1000 ( 1? 1- update ) drop     | look up

'list2 >a | A has list2
	a@ 'minimum !  1000 ( 1? 1- update ) drop     | look up
```

The word `update` uses the address in register A to search. 
First A has `list1` and then ` list2`. 
This could be passed as a parameter on the stack but would require moving the element counter.
The variable `minimum` will have the minimum value of each list when the loop terminates.

### Nested Loops with Registers

Nested loop using registers avoids stack manipulation to retrieve address, recover value, and advance:

```forth
^r3/lib/rand.r3
#array * 80  | 10 x 8 bytes

: 0 ( 9 <? dup 8 * 'array + rand8 dup .print swap ! 1+ ) drop .cr ;
: 0 ( 9 <? dup 8 * 'array + @ .print 1+ ) drop .cr ;

: a> >r 
  'array >a
  0 ( 9 <? 1+ a@+ .print ) drop .cr ;
r> >a ;
```
## Operating System Connection

The computer only works with numbers, and any communication with the user or the rest of the world is done through words that the operating system resolves, independent of the language but dependent on its connection to it.

Access to external libraries is fundamental because it allows us to access hardware capabilities not directly available, either by manufacturer criteria that doesn't want its internal workings seen, or by resource complexity that doesn't want to be reprogrammed.

Generally, library documentation is needed to know what functions to import, and the use and functioning of these words will be the library's responsibility. Once we have access to the library, we can build our program from there.

## Console

The VM starts with a console where only text can be displayed.

#### ^r3/lib/console.r3

| Word | Stack Effect | Description |
|------|--------------|-------------|
| `::cls` | `--` | Clear console, erase all characters |
| `::.write` | `"text" --` | Write text to console |
| `::.print` | `.. "text with %" --` | Write formatted text, extract values from stack for % |
| `::.home` | `--` | Move cursor to console start |
| `::.at` | `x y --` | Position cursor at row y, column x |
| `::.fc` | `color --` | Set foreground color for text |
| `::.bc` | `color --` | Set background color for text |
| `::.input` | `--` | Wait and save a line of text entered from keyboard |
| `::.inkey` | `-- key` | Return pressed key, zero if none |
| ##pad | variable |  Contains the entered text after `.input` |

```forth
^r3/lib/console.r3
: "hello world" .println ;
```
### Advanced Operating System Connection

The OS connection is built through function calls to dynamic libraries. In Windows these are called .DLL (dynamic link libraries).

The current distribution, besides connecting to the OS, uses the hardware's graphics capabilities through SDL version 2 libraries, since it's multiplatform.

| Windows DLL | R3 Library |
|-------------|------------|
| SDL2.dll | `^r3/lib/sdl2.r3` |
| SDL2_image.dll | `^r3/lib/sdl2image.r3` |
| SDL2_mixer.dll | `^r3/lib/sdl2mixer.r3` |
| SDL2_net.dll | `^r3/lib/sdl2net.r3` |
| SDL2_ttf.dll | `^r3/lib/sdl2ttf.r3` |

- **Platform Support:** R3 is being prepared for use under Linux, MAC (Macintossh), or RPI (Raspberry Pi), though some are incomplete.

### Library Loading

The way to connect to a library is through 2 words: one to load the library and one to get the address of the function to execute, plus a set of words that make the call.

| Word | Stack Effect | Description |
|------|--------------|-------------|
| `LOADLIB` | `"name" -- liba` | Load dynamic library |
| `GETPROC` | `liba "name" -- aa` | Get function address |

Words to call these functions take parameters from the stack according to parameter count and leave the OS response:

| Word | Stack Effect | Description |
|------|--------------|-------------|
| `SYS0` | `aa -- r` | Call function with 0 parameters |
| `SYS1` | `a aa -- r` | Call function with 1 parameter |
| `SYS2` | `a b aa -- r` | Call function with 2 parameters |
| ... | ... | ... |
| `SYS10` | `a b c d e f g h i j aa -- r` | Call function with 10 parameters |

---

## Libraries

Most common words from main libraries.

### Operating System Communication

#### ^r3/lib/core.r3

| Word | Stack Effect | Description |
|------|--------------|-------------|
| `::msec` | `-- msec` | Push system milliseconds since program start |
| `::time` | `-- hms` | Push current time (hours, minutes, seconds) in one value |
| `::date` | `-- ymd` | Push current date (year, month, day) in one value |

##### Date Stamp

```forth
^r3/lib/posix/core.r3
:around2k | yyyy -- yy | get 2 last digits near 2000
  1900 - 100 >=? ( 100 - ; ) ;
:.0n|nn 10 <? ( "0%d" .print ; ) "%d" .print ; | n --
:.today                    | date stamp
  date dup 16 >> $ffff and | yyyy
  around2k .0n|nn          | .yy
  dup 8 >> $ff and .0n|nn  | .mm
  $ff and .0n|nn           | .dd
;
: "Today is " .print .today .cr ;
```

### Random Numbers

#### ^r3/lib/rand.r3

There are a few options to choose from, so check them.

| Word | Stack Effect | Description |
|------|--------------|-------------|
| `::rerand` | `s1 s2 --` | Initialize random generator with two seeds |
| `::rand` | `-- rand` | Push a 64-bit random number |
| `::randmax` | `max -- value` | Push random number between 0 and MAX-1 |

#### Random Number Usage

```forth
time msec rerand        | Initialize with variable time values
10.0 randmax            | Random number 0 to 9.999...
5.0 randmax 5.0 -       | Random number -5.0 to 0
5.0 randmax 5.0 +       | Random number 5.0 to 10.0
```

### Graphics with SDL2 Library

The SDL2 library allows access to graphics capabilities. https://www.libsdl.org/

This library allows starting a graphics window and drawing on it. Besides the graphics window, mechanisms are needed to respond to KEYBOARD and MOUSE buttons.

#### Graphics Window - ^lib/sdl2.r3

| Word | Stack Effect | Description |
|------|--------------|-------------|
| `::SDLinit` | `"title" w h --` | Start graphics window w×h pixels with title |
| `::SDLfull` | `--` | Set window to fullscreen |
| `::SDLquit` | `--` | Exit graphics window |
| `::SDLcls` | `color --` | Clear screen with chosen color |
| `::SDLredraw` | `--` | Refresh screen |
| `::SDLshow` | `'word --` | Execute WORD each time screen redraws |
| `::exit` | `--` | Exit from SHOW |

#### Input Variables

| Variable | Description |
|----------|-------------|
| `##SDLkey` | Code of pressed key, zero if no key |
| `##SDLchar` | Character code representation |
| `##SDLx`, `##SDLy` | Mouse cursor position x and y in window |
| `##SDLb` | Mouse button state, zero when none pressed |

#### Loading Graphics Files - ^lib/sdl2image.r3

| Word | Stack Effect | Description |
|------|--------------|-------------|
| `::loadimg` | `"file" -- img` | Load image file (PNG with transparency, JPG without) |
| `::unloadimg` | `img --` | Remove image from memory |

#### Drawing Graphics - ^lib/sdl2gfx.r3

| Word | Stack Effect | Description |
|------|--------------|-------------|
| `::SDLColor` | `col --` | Set drawing color |
| `::SDLPoint` | `x y --` | Draw a pixel |
| `::SDLLine` | `x1 y1 x2 y2 --` | Draw line from x1,y1 to x2,y2 |
| `::SDLFRect` | `x y w h --` | Draw filled rectangle |
| `::SDLRect` | `x y w h --` | Draw rectangle outline |
| `::SDLFEllipse` | `rx ry x y --` | Draw filled ellipse |
| `::SDLEllipse` | `rx ry x y --` | Draw ellipse outline |
| `::SDLTriangle` | `x y x y x y --` | Draw filled triangle |

#### Image Drawing

| Word | Stack Effect | Description |
|------|--------------|-------------|
| `::SDLimagewh` | `img -- w h` | Get image width and height |
| `::SDLImage` | `x y img --` | Draw image at position |
| `::SDLImages` | `x y w h img --` | Draw image at position with size |
| `::SDLImageb` | `box img --` | Draw image in defined box |
| `::SDLImagebb` | `box box img --` | Draw part of image in defined box |
| `::SDLspriteZ` | `x y zoom img --` | Draw image at position with scale |
| `::SDLSpriteR` | `x y ang img --` | Draw image at position with rotation |
| `::SDLspriteRZ` | `x y ang zoom img --` | Draw image with rotation and scale |

#### Tile Sheets

| Word | Stack Effect | Description |
|------|--------------|-------------|
| `::tsload` | `w h filename -- ts` | Load image as tile sheet |
| `::tscolor` | `rrggbb 'ts --` | Set tile color (tints white color) |
| `::tsdraw` | `n 'ts x y --` | Draw tile on screen |
| `::tsdraws` | `n 'ts x y w h --` | Draw tile on screen with size |

#### Sprite Sheets

Sprites are parts of an image, drawn from the center of defined size.

| Word | Stack Effect | Description |
|------|--------------|-------------|
| `::ssload` | `w h file -- ss` | Load sprite sheet |
| `::ssprite` | `x y n ss --` | Draw sprite N at centered position |
| `::sspriter` | `x y ang n ss --` | Draw sprite N centered with rotation |
| `::sspritez` | `x y zoom n ss --` | Draw sprite N centered with scale |
| `::sspriterz` | `x y ang zoom n ss --` | Draw sprite N centered with rotation and scale |

## Examples

For more examples, explore the `r3/demo` directory in the R3 repository and try various library combinations.

