# Aqua Language Specification: Lexical Structure

## 1. Whitespace

Whitespace characters are ignored. Whitespace includes any character classified as Unicode space.

## 3. Comments

Aqua supports two types of comments:

### 3.1 Single-line Comments

A single-line comment starts with `#` and continues until the end of the line.

```aqua
# A comment
let a = 2 + 2 # A comment after code
```

### 3.2 Multi-line Comments

A multi-line comment starts with `#-` and ends with `-#`.

```aqua
#-
This is a
multi-line comment
-#
```

Comments are ignored during lexical analysis.

## 4. Identifiers

- `<indent>`
  
Identifiers name variables, subroutines, modules, and properties.

### Rules:

- Must start with a letter or underscore (`_`)
- May contain letters, digits, or underscores
- Are case-sensitive

Valid examples:

```aqua
myVar
_userName
count123
```
### Bypassing the rules:

- Enclosing a identifier in backticks lets you can bypass all rules of naming identifiers

```aqua
let `any string` = 10
```

- More about the escape rules are at 6.2

## 5. Keywords

The following identifiers are reserved and cannot be used as regular identifiers:

```
let, begin, end, catch,
if, elif, else,
for, while, repeat, until,
switch, case, default,
using,
sub, with,
mod, export, import, as,
return, break, continue,
raise,
enum,
and, or, xor, not,
in,
typeof,
true, false, null, _ 
infinity, nan
```

> Note: null and _ have token `<null>`

## 6. Literals

### 6.1 Numeric Literals

- `<number`
  
Aqua supports:

- Decimal integers
- Floating-point numbers
- Binary literals (`0b`)
- Octal literals (`0o`)
- Hexadecimal literals (`0x`)
- Underscore separators

Examples:

```aqua
42
3.14
1_000_000
0b1010
0o755
0xFF
0b110.011
0o755.755
0xFF.FF
```

### 6.2 String Literals

- `<string>`

String literals are enclosed in double quotes:

```aqua
"Hello"
```

Escape sequences are supported.

- `\"` ``\` `` `\\` are converted into versions without `\` without closing a the string
- `\n` `\r` `\t` `\a` `\b` `\v` are converted into special unicode characters
- `\xHH`  a 8-bit unicode character (asci character) by it's hex value
- `\uHHHH` a 16-bit unicode character by it's hex value
- `\UHHHHHHHH` a 32-bit unicode character by it's hex value
- `\\}` `\@` do not change, but are used in string formatting

### 6.3 Boolean Literals

```
true
false
```

### 6.4 Null Literal

```
null
```

### 6.5 Special Numeric Literals

```
infinity
nan
```

## 7. Operators and Punctuation

```
+     <plus>
-     <minus>
*     <multiply>
/     <divide>
%     <modulus>
//    <strong divide>

++    <increment>
--    <decrement>

==    <eq>
~=    <ne>
>     <gt>
<     <lt>
>=    <ge>
<=    <le>

<<    <shl>
>>    <shr>

=     <assign>

?.    <question dot>
?.>   <question method>
?.~   <question delete>
??    <question>

.     <dot>
,     <comma>
:     <column>
&     <ptr>
$     <clone>

()    <parenthesis opened> / <parenthesis closed>
[]    <square bracket opened> / <square bracket closed>
{}    <brace opened> / <brace closed>

...   <dots>
->    <bind>
.>    <method>
.~    <delete>
```