# Aqua Language Specification: Grammar

## 1. Program

An Aqua program consists of a single top-level block expression.

```
block <eof> | block <export> + export_list 
```

> Note: Export list would be defined in 13.1.

The top-level block defines the module scope.

If the export clause is present, the listed identifiers are exported from the module.
If no export clause is provided, the program executes normally without exporting symbols.
The end of file (EOF) terminates the program.

## 2. Statements and Expressions

Aqua is an expression-oriented language.

Most syntactic constructs are expressions and produce a value. 
Syntactic constructs that are not a expression are a part of one or several types of expressions

### 2.1. Statement-like Constructs

Certain constructs exist primarily for control flow and may not produce meaningful values (e.g. return, break, continue, raise).
However, syntactically they are treated within expression contexts as defined by the grammar.

## 3. Block Expression

A block expression is a fundamental unit in Aqua.
It is a sequence of expressions in order and an optional `catch` block. 
The termination of a block depends on the context in which it is used.

```
block: $ending = [expression] + (<catch> + <ident> + block $ending) + $ending
```

### 3.1. Separate Block Expression 

Block expressions are not always a part of an other syntactic construct. 
They could be used as an expression for scope controls

```
blockExpression = <begin> + block <end>
```

## 4. Variables, Declaration

Variables are accessed by identifiers

```
ident_expression = <ident>
```

To declare a variable use a `let` expression

```
let_expression = <let> + <ident>
```

> Note: let declares only one variable at a time

## 5. Assigment

Assigment expressions are used to modify a value, or values.

Aqua specifies two types of assigment expressions:

### 5.1 Direct Assigment

Direct assigment modifies one value and optionally applies a `binary expression`

```
assigment_bin = <plus> | <minus> | <multiply> | <divide> | <modulo> | <strong divide> | <and> <or> <xor> <shr> | <shl> | <question> |  <in> | <bind>
direct_assigment = expression + (assigment_bin)? + <eq> + expression
```

### 5.2 Pattern Assigment

Pattern assigment uses **pattern matching** to target object and array properties 

```
pattern_assigment = expression + (<column> + <ident>)? + [<comma> + expression + (<column> + <ident>)?] 
    + <eq> + expression
```

```
assigment_expression = direct_assigment | pattern_assigment
```

## 6. If Expression

If expressions implements conditional block execution.

If expressions contains of a main if block with a condition, a sequence of elif (else if) blocks with conditions in order and an optional else block.

```
else_block: $ending = block <else> + block $ending
elif_block = block <elif> + [expression + block <elif>] + (block <end> | else_block <end>)
if_expression = <if> + expression + (block <end> | elif_block | else_block <end>)
```

## 7. While/Repeat-Until Loop

While and repeat-until loops implements conditional block repetitive execution.
Both while and repeat-until have an optional else blocks that are executed if break signal wasn't triggered.

```
while_expression = <while> + expression + (else_block <end> | block <end>)
```

```
repeat_until_expression = <repeat> + (else_block <until> | block <until>) + expression
```

## 8. Arguments

Arguments is a syntactic construct used in other syntactic constructs to define a template for mapping a list of values to variables.
Arguments is a sequence of identifiers in order with an optional default value and an optional identifier to store the rest of given values.

```
arguments = [<ident> + (<eq> + expression)?] + (<dots> + expression)?
```

## 9. For Loop

For loop implements iteration over an `iterable` value.
For als has an optional else blocks that are executed if break signal wasn't triggered.


```
for_expression = <for> + arguments + <in> + expression + (block <end> | else_block <end>)
```

## 10. Switch expression

Switch expression implements comparing a value to multiple values

```
switch_part = block <end> | block <case> + expression + switch_part | block <default> + block<end>
switch_expression = <switch> + expression + <case> + expression + switch_part
```

## 11. Using Expression

Using expression implements a temporarily introduce of variable into a block scope.

```
using_expression = <using> + expression + (<as> + <ident>)? + block <end>
```

## 12. Subroutines

Subroutines in Aqua are first-class functions with arguments, block expression and an optional prototype value.

```
subroutine_expression = <sub> + (<ident>)? + <parenthesis opened> + arguments + <parenthesis opened> + (block <end> | block <with> + expression)
```

## 13. Modules And Importing

### 13.1. Modules

Modules implements scope control with exporting 

```
export_list = <ident> + [<comma> + <ident>]
module_expression = <mod> + <ident> + block <export> + export_list
```

### 13.2 

Import expression implements importing values from other files. 

```
import_expression = <import> + <expression> + (<as> + <ident>)? 
```

> Note: probably pattern matching would be added to import and using in future versions


## 14. Array, Object And String Literals

### 14.1. Array Literals

Array literals allow you to define a sequence of elements and to spread other arrays into a new array.

```
array_element = (<dots>)? + expression
array_declaration: $ending =  + (array_element)? + [<comma> + array_element] + $ending
array_expression = <square bracket opened> + array_declaration <square bracket closed>
```

### 14.2. Object literals

Object literals allow you to define key-value mappings, embedded `subroutines` as methods, and spread other objects.

```
object_element = <ident> + <column> + expression | <dots> + expression | subroutine_expression @with-name
object_expression = <brace opened> + (object_element)? + [<comma> + object_element] + <brace closed>
```

### 14.2

String literals allow you to insert values into a string.

```
inserted_value = `@` + `{` + expression + `}`
```

- `\@` would be converted into `@` without inserting a value
- `\}` are converted into `}` inside of a inserted value to use `}` in syntactic constructs
  
## 15. Signals

Signals implement calling different signals to manage exceptions, returns from subroutines and loop controls.
Each signal has a type and an optional value.

```
signal = <raise> | <return> | <continue> | <break>
signal_expression = signal + (<column> + expression)?
```

## 16. Operators

### 16.1 Prefix operators

```
prefix_operator = <not> | <typeof> | <neg> | <ptr> | <clone>
prefix_expression = prefix_operator + expression
```

### 16.2 Postfix operators

```
postfix_operator = <dec> | <inc>
postfix_expression =  expression + postfix_operator
```

### 16.3 Binary operator

```
binary_operator = <and> | <or> | <xor> | <in> | <plus> | <minus> | <multiply> | <divide> | <modulus> | <strong divide> | <shl> | <shr> | <eq> | <ne> | <gt> | <lt> | <ge> | <le> | <question dot> | <question method> | <question delete> | <dot> | <method> | <delete> | <bind> | <question>
binary_expression = expression + binary_operator + expression
```

### 16.4 Index And Slice

```
index_expression = expression + <square bracket opened> + expression + <square bracket closed>
slice_expression = expression + <square bracket opened> + (expression)? + <column> + (expression)? + <square bracket closed>
```

### 16.5 Calling

```
call_expression = expression + <parenthesis opened> + array_declaration <parenthesis closed>
```

### 17. Precedence Table

Operators are listed from **lowest precedence** (evaluated last) to **highest precedence** (evaluated first).

| Level | Operators| Category|
| ----- | ---------------------------------------- | ------------------------ | 
| 1| `=`| PatternAssigment| Right|
| 2| `=`| DirectAssigment | Right|
| 3| `??`| Null coalescing| Right|
| 4| `or`| Logical OR| Left|
| 5| `and`| Logical AND| Left|
| 6| `xor`| Logical XOR| Left|
| 7| `==`, `~=`| Equality| Left|
| 8| `>`, `<`, `>=`, `<=`, `in`| Comparison| Left|
| 9| `<<`, `>>`| Bitwise Shift| Left|
| 10| `+`, `-`| Additive| Left|
| 11| `*`, `/`, `%`, `//`| Multiplicative| Left|
| 12| `not`, `typeof`, unary `-`, `&`, `$`| Prefix Operators| Right|
| 13| `->`| Bind Operator| Right|
| 14| `.`, `.>`, `.~`, `?.`, `?.>`, `?.~`, `[]`, `++`, `--` | Postfix / Member / Index | Left|

- Higher levels bind stronger.
- Assignment and null-coalescing are right-associative.
- Prefix operators bind to the nearest expression on the right.
- Postfix operators bind strongest and apply to the nearest expression on the left.
-  When a bind operator is followed by a function call `(...)`, the call **does not consume the bind**.  
Instead, the bind applies to the function itself **before** the call
- Pattern assigment and direct assigment have different levels for being able to use direct assigment in syntactic constructs that uses `<comma>`  