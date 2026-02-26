# Aqua Language Specification: Semantics

## 1. Values, Cloning Values

Values are a pointer to 8 different types. Aqua by default uses *deep cloning* meaning cloning a value will clone recursively all values stored in it. 

### 1.1. Numbers

Numbers are stored as a 64-bit floating-point.

### 1.2. Strings

Strings are a immutable sequence of **bytes** or **characters**. It also stores it's length.

### 1.3. Booleans

Booleans have two states: **true** and **false** 

### 1.4 Arrays

Arrays are a mutable sequence of **values** with a length. Depending on the runtime arrays may also have a capacity.

While cloning arrays it will create a new array with the same length and the same but cloned elements.

### 1.5. Objects

Objects are stored as a key-value data structure. Keys are always stored as a **string**. Values are stored as a **value**. Objects should have a length value that counts the number of key-values.

While cloning arrays it will create a new object with the same keys and corresponding values clones. 

### 1.6. Subroutines And Methods

Subroutines are callable Values, storing **arguments template** for mapping an array of arguments, the scope where the subroutine was created, the prototype **value** and executable code. 

Executable code depends on the runtime for example: bytecode, AST, JavaScript, precompiled code for built-in functions

Subroutines are stored with an additional pointer and cloning a subroutine should not clone anything stored inside.

Methods are subroutines with context added via **bind**, **method** operators. 

Cloning methods doesn't clone the context. 

### 1.7. Errors

Errors is a immutable type storing a unsigned 16-bit integer as a error code and a string as an error message.

### 1.8 Null

Null is a one-state value that is mostly used as default values. 

----------------

Depending on the runtime different value types may store additional information for example subroutines may store it's name for better stack trace.

## 2. Evaluation Model

### 2.1. Expression-Oriented Nature

Aqua is a expression-oriented language. It means everything is a expression with an exception of parts of expressions like **arguments**. 

- Every expression returns a **expression result**. 
- **Block expressions** returns the result of the last expression
- **If expressions** and **switch expressions** returns the result of the block that was executed
- **For**, **while**, **repeat-until** returns a result depending on the actions
    1. if **break** was triggered it returns the value of the break **expression result**
    2. if **break** was not triggered and the else block exists the result of the else block is returned
    3. if at the last iteration **continue** was triggered it returns the value of the continue **expression result**
    4. at least it returns the value of the block that was executed at the last iteration
- **Module expressions**, **non-anonymous subroutine declarations** returns null, but adds a value to current scope

### 2.2. Order of evaluation

The language uses a strict, left-to-right evaluation order.

Unless explicitly stated otherwise, all subexpressions are evaluated from left to right before the enclosing expression is evaluated.

#### Binary expressions

In binary expressions the left operand is evaluated first, followed by the right operand unless the operator requires a non-evaluated value: usually member access operators.  
After both operands are evaluated, the operator is applied to the resulting values.

#### Function Calls

In a function call expression:

```aqua
f(arg1, arg2, ..., argN)
```

evaluation proceeds as follows:

1. The callee expression `f` is evaluated.
2. Arguments are evaluated from left to right.
3. The function is invoked with the evaluated argument values.

All argument expressions are evaluated before the function body begins execution.

#### Assignment

In an assignment expression:

the right-hand side is evaluated first, and then the assignment to **value**/**pattern** is performed.

### 2.3. Strict Evaluation

Aqua uses strict (eager) evaluation.

All expressions are fully evaluated before their results are used.  
Operands of operators and arguments of function calls are always evaluated prior to applying the operator or invoking the function.

The language does not provide lazy evaluation semantics.

Every subexpression produces a value unless evaluation is interrupted by a **none-None signal**

The only exception is the binary operator `??` that evaluates the right value only if the left is null

### 2.4. Block Evaluation

- Blocks are a ordered sequence of expressions and the block evaluates expressions one by one starting with the first one and evaluating every expression unless is interrupted by a **none-None signal**
- Blocks return the result of the last expression
- An empty block returns **null**

## 3. Expression And Subroutine Results

Expressions and subroutines return results. A result is required and by default will be null.

### 3.1 Signals

Expression and subroutine results have different signals that are used for managing returning, breaking, continuing, raising. 

Expression result signals
0. None
1. Return
2. Raise
3. Break
4. Continue

Subroutine result signals

0. Return
1. Raise

Expression results into subroutine and subroutine into subroutine
- ExpressionNone, ExpressionReturn -> SubroutineReturn
- ExpressionRaise -> SubroutineRaise
- ExpressionBreak, ExpressionContinue -> Illegal, raising an invalid signal error
  
- SubroutineReturn -> ExpressionNone
- SubroutineRaise -> ExpressionRaise

None is the only signal that continues evaluating expressions. Break and Continue could be cached in **for**, **while**, **repeat-until** loops. Return could be cached in function execution. Raise could be cached in catch blocks

### 3.2 Creating results

For expression results with signals that are not none a special expression type exists that allows to return, raise, break, continue. Also they may be created due to different runtime causes like unsupported types for a certain operators etc.

None expression results are created when the expression was executed without any other signals called.

### 3.3 Examples

Creating signals:

```aqua
raise
break
continue
return

raise: error(ValueError)
break: 10
continue: 37
return: obj
```

Catching signals

```aqua
begin 
    raise: error(ValueError, "something went wrong")
catch err
    println(err)
end
```

```aqua
let a = for i in range(100)
    if i % 5 == 0 and i % 7 == 2 
        break: i
    end
else
    -1
end
```

```aqua
sub a()
    return: 10
end

println(a())
```

----------------

Expression and subroutine results may store other data depending on the runtime, for example the stack trace.

## 4. Environments and Scope 

### 4.1. Lexical Scoping

Aqua uses lexical (static) scoping.

The meaning of a variable is determined by the environment in which it is defined, not by the environment from which a function is called.

### 4.2 Environment Model

An environment is a mapping from identifiers to values.

Each block and subroutine introduces a new environment.
Environments form a chain, where each environment may have a parent environment.

### 4.3. Variable Resolution

When an identifier is evaluated, the runtime searches for a binding in the current environment.
If not found, the search continues in the parent environment.
If no binding is found, a runtime error occurs.

### 4.4. Shadowing

If a binding with the same name exists in an outer environment, a new binding in the current environment shadows the outer binding.

The outer binding remains unchanged.

```aqua
let a = 10

begin
    let a = 20
    println(a)
end

println(a)
```

> Output:
> 
> 20
> 
> 10

### 4.5. Closures

Subroutines capture the lexical environment in which they are defined.

A captured environment remains accessible for the lifetime of the subroutine value.

## 5. Specific Expression Types

### 5.1 Block Expression

Block expressions are used to execute a ordered sequence of expressions.

```aqua
begin
    println("first")
    println("second")
end
```

> Output:
>
> first
>
> second

Block expressions have **catch blocks** that allow to catch errors

```aqua
begin 
    raise
catch e
    println(e)
end
```

> Output:
>
> null
If no exceptions were raised catch does not evaluate 

```aqua
begin 
    println("ok")
catch e
    println(e)
end
```

> Output:
>
> ok

### 5.2 Let Expression

Let expression creates a new binding in the current environment with value `null` bypassing one cloning step

```aqua
let a 
println(a)
```

> Output:
>
> null

```aqua
let a = 10
println(a)
```

> Output:
>
> 10

The result of a let expression could be used as an argument in a subroutine

```aqua
sub a(b)
    b = "hello"
end

a(let x)
println(x)
```

> Output:
>
> hello

Using let with the same name in the same scope redeclares the value

```aqua
let a = 10
let a = 20
println(a)
```

> Output:
>
> 20

### 5.3 Array Declaration

Array declaration uses a sequence of array elements witch is a subexpression and a flag is the element spreading or not. 

A normal element is just appended to the array

A spread element should be type of **array** and each of it elements should be appended to the result array.

Each value is cloned before appending.

```aqua
let a = [1, 2, 3, 5]
let b = [0, 1, ...a, 6, 7]
println(a, b)
a[0] = 0
println(a, b)
```

> Output:
>
> [1, 2, 3, 5] [0, 1, 1, 2, 3, 5, 6, 7]
>
> [0, 2, 3, 5] [0, 1, 1, 2, 3, 5, 6, 7]


### 5.4. Subroutine Declaration

Subroutines can be anonymous and not anonymous.

Anonymous subroutines returns it's value after declaration.

Non anonymous subroutines creates a new binding in the current environment with it's value.

The default values in subroutine arguments are evaluated before it's prototype.
Default values and the prototype are cloned.

```aqua
let a = sub() println("anonymous") end
sub b() println("not anonymous") end

let obj = {a: 10}
sub Obj() it.b = 10 _ with obj
obj.a = 11
println(Obj())
```

> Output:
>
> {a: 10, b: 10}

### 5.5. Object Declaration

Object declaration contains of a sequence of object elements witch are either a spreading value or a key-value pair.

A key-value pair is adding a property *key* with value *value* in the result object

A spread element should be type of **object** and each of it elements should be appended to the result array. 

Each value is cloned adding properties.

Elements and spreed elements can shadow each other. It depends on the order in the sequence of elements

```aqua
let a = {a: 1, b: 2}
let b = {...a, b: 3}
println(b)
```

> Output:
>
> {a: 1, b: 3}

### 5.6 If Expression

If expressions contain of a base if block: condition and block expression. A sequence of elif blocks: condition and block expression. And a optional else block

If expressions evaluate in this order:
1. Calculates the if condition and converts into **boolean** via **into bool**
2. If the condition is true the if block is evaluated and the if expression returns it's result
3. If the condition is false and there is a remaining elif block returns to step 1 but using the next elif block
4. If no elif blocks remains and else block exists returns the result of the else block.
5. If the else block does not exists returns **null**

### 5.7 For Expression

For expression iterates over an **iterable value** with mapping arguments as in subroutines. It provides standard for Aqua value returning in loops logic with **else**, **break**, **continue**.

It also provides an optional enumerating for each iteration, if enumerating is including the first argument will be the iteration number starting from 0.

- Transforming into **iterable**
    - For strings iterating over each **character** of the string as a separate string. Since strings are immutable changing the character does not affect the original string
    - For arrays iterating over each element of the array. The elements are not cloned and could be mutated affecting the elements in the array.
    - For objects iterating over each key and value of the object. If the object has a method `__next__` it would used as the iterable (next part for subroutines). Objects are unordered. Mutating the key does not affect the object, however mutating the value will change it in the object
    - For subroutines they would be called unless the subroutine raised error with code №7 `IteratorStop`
    - Other types are not iterable
  
```aqua
for i in "hello world"
    print(i)
end
println()

for i in [1, 2, 3]
    print(i, "")
end
println()

for k, v in {a: 1, b: 2} 
    print("@{k}: @{v} ")
end 

println()

sub range(max) 
    it.max = max
    return
with {
    max, current: 0,
    sub __next__() 
        if it.current >= it.max 
            raise: stop 
        end
        it.current++ -1
    end
}

for i in enum range(10)
    print(i, "")
end
println()
```

> Output:
>
> hello world
>
> 1 2 3
> 
> a: 1 b: 2
>
> 0 1 2 3 4 5 6 7 8 9

### 5.8 While And Repeat-Until Expressions

While expressions executes the same block while the condition is **true**. Condition is converted into **boolean** via **into bool**

While first checks the condition and then executes the block. 

Repeat-until expressions execute the same block while the condition is **false**. Condition is converted into **boolean** via **into bool**

Repeat-until first executes and the checks the condition.


Both while expressions and repeat-until expressions provides standard for Aqua value returning in loops logic with **else**, **break**, **continue**.

### 5.9 Call Expressions 

Call expressions process by these steps:

1. The callee expression is evaluated
2. The arguments are evaluated
3. Checking that the callee is callable (or transforming into callable via `__call__` method in objects)
4. Local scope based on saved scope in the subroutine
5. Arguments mapped and `it` declared.
    If the function has a context it is used without cloning
    If the function has a prototype the prototype is used with cloning
    Otherwise sets to null
6. Body of the function executed
7. The result of the function transformed into a **subroutine result** 

### 5.10 Assigment Expression

Assigment expression divide in two types: direct and pattern assigment.
The right value in every type of assigment expression should be cloned.

In direct assigment:
1. The left value evaluates without cloning.
2. Optionally a binary expression applies between left & right. 
3. Right value is assigned to the left value.
4. Returns the left value bypassing one step of cloning

Direct assigment allows more advanced features with using new declared values as function arguments.

```aqua
sub add2(a)
    a +=2
end

add2(let x = 10)
println(x)
```

> Output: 
>
> 12

In pattern assigment the right value must be either an array or an objet.

For arrays: For each i-th **pattern element** the pattern element value evaluates and is assigned to the i-th element of the array. Arrays If the pattern argument includes a key-name an error would raise.

For objects: For each **pattern element** the pattern element value evaluates and if it has a key-name the objects property with key of key-name would be assign to the value. If the is no key-name, it will try to figure the key by using **let expression** or **ident expression**

```aqua
let a, let b = [1, 2, 3]
println(a, b)

let a, let b, let c = [1]
println(a, b, c)

let a, let x: b, let y: c = {a: 1, b: 2}
println(a, x, y)
```

> Output:
>
> 1 2
> 
> 1 null null
> 
> 1 2 null

### 5.11 Mod expression

Mod expression allows to execute a block with exporting some of bins as a object.

```
mod math
    let PI = 3.1415 # not accurate
    sub add(a, b)
        a + b
    end
export PI, add

println(math.PI, math.add(math.PI, 10))
```

> Output:
>
> 3.1415 13.1415

### 5.12 Import expression

Import expression allows to import file-modules from standard library, files, external libraries.

Import use string values via **into string** as an import string, and optional identifier to define the name the module would be bind in. If this identifier isn't specified it will be calculated from the import string.

1. If the import string is a predefined path as a standard library, it is resolved as a standard library import.
2. If the import string is a name of a dependency in the project configuration, it is resolved as a root dependency import.
3. If the import string does not end with `.aq`, the extension is appended. The value of the result of the expression of the import string would not be changed it is an internal mutation.
4. If the import string contains `:`, it is split at the first `:` into first and second.
5. If second is non-empty and first is a name of a dependency in the project configuration, it is resolved as a dependency subpath import.
6. Otherwise, the import string is resolved as a file import relative to the directory of current, and the resulting path is converted to an absolute path.

Resolution priority is: standard library → root dependency → dependency subpath → file.

For a root dependency import it executes and gets the exports of the **main** file in the library.

Already executed files would not be executed again, but used the pre-saved results.

### 5.13 Using Expression

Using expression evaluates an expression and disposes the result not depending on the result of the block.

```aqua
sub needDispose() with {
    disposed: false,
    sub __dispose__()
        it.disposed = true
    end
}
let d = needDispose()
using d
    println(d)
end

println(d)
```

> Output: 
>
> {\_\_dispose\_\_: (), disposed: false}
>
> {\_\_dispose\_\_: (), disposed: true}

### 5.14 Switch Expression

Switch expressions compares a value to several different values. 

1. Evaluating the main expression
2. Checking the equality of the main expression and the expression of the current case block
3. If the main expression is equal to the expression of the current case block return the result of the current case block
4. If the main expression is not equal to the expression of the current block and there are case blocks left go to 2. with the next case block
5. If the default block exists return it's result
6. Otherwise return null

```aqua
let a = 5
switch a
case 1
    println(2)
case 2 
    println(2)
case 3 
    println(3)
case 4
    println(4)
default
    println("unexpected", a)
end
```

> Output:
>
> unexpected 5

### 5.15 Slice expression

Slice expressions makes a slice of an array. Type of the slicing value should be **array** or **string** unless operator overloading isn't used.

For array [e_1, e_2, e_3, ..., e_n-1, e_n] slicing on start = x, end = y the array will look like [e_x, e_x+2, ..., e_y-1, e_y-2]. 

Start and end can be any 64-bit integer, if start or end is more then length of the array it will have the same effect as if it was the length of the array. If -start or -end is more than length of the array it will have the same effect as uf ut was 0. If start or end is less that zero it will have the same effect as if it would be length + start or end. If end is less than start it would return an empty array. Default for start is 0, default for end is length of the array.

Slices of arrays should be copied, not copying slices is considered unsafe and the behavior depends on the runtime.

```aqua
let a = [1, 2, 3, 5, 6]
println(a[1:3])
println(a[2:1])
println(a[7:])
println(a[:7])
println(a[:-1])
println(a[-1:])
println(a[:-7])
println(a[-7:])

let b = &a[1:4] # unsafe
```

> Output:
>
> [2, 3]
> []
> 
> []
> 
> [1, 2, 3, 5, 6]
>
> [1, 2, 3, 5]
> 
> [6]
>
> []
> 
> [1, 2, 3, 5, 6]

Strings work the same with the difference of using the characters even if it is stored as bytes

## 7. Into Types

### 7.1. Into Bool

1. Number ~= 0
2. String ~= ""
3. Array ~= []
4. Object ~= {}
5. Null -> false
6. Bool
7. Error, Subroutine -> true

### 7.2 Into String

Depends on the runtime. 

## 6. Overwriting Operators

Most operators may or must have custom logic for objects witch curtain methods

### 6.1. Converting
`__bool__` Called when converting value into bool

`__number__` Called when converting value into number

`__iter__` Called when converting value into iterator

`__next__` Called when an object is iterated

`__display__` Called when converting value into string

`__call__` Called when converting value into callable

### 6.2. Other
`__slice__` Called when slicing value

`__length__` Called when getting length of value

`__dispose__` Called when disposing a value

### 6.3. Unary/Prefix/Postfix Expressions
`__neg__` Operator `-`

`__not__` Operator `not`

`__dec__` Operator `--`

`__inc__` Operator `++`

### 6.4. Binary Expressions
`__add__` Operator `+`

`__sub__` Operator `-`

`__mul__` Operator `*`

`__div__` Operator `/`

`__mod__` Operator `%`

`__intdiv__` Operator `//`

`__and__` Operator `and`

`__or__` Operator `or`

`__xor__` Operator `xor`

`__shr__` Operator `>>`

`__shl__` Operator `<<`

`__eq__` Operator `==`

`__ne__` Operator `~=`

`__gt__` Operator `>`

`__ge__` Operator `>=`

`__lt__` Operator `<`

`__le__` Operator `<=`

`__in__` Operator `in`

`__index__` Operator `[]`

`__bind__` Operator `->`

## 7. Unary/Prefix/Postfix Expressions

Unary expressions is a expression that process a value and returns a result. It may mutate a value.

### 7.1. `&`: Pointer

The pointer operator does not clone a value even if it should be cloned bypassing one cloning step.

### 7.2. `$`: Clone

The clone operator clones a value even if it shouldn't be cloned. It doesn't clone twice one cloning step.

### 7.3. `-`: Negative

The negative operator works only for **numbers** and returns the negative representation of the value (* -1)

### 7.4. `not`: Not 
The not operator works for **numbers** (only integers) and **booleans**. It returns opposite value:
1. For **numbers** (only integers) it is bitwise not
2. For **booleans** it is logical not: not true = false, not false = true

### 7.5. `++`: Increment
The increment operators works only for **numbers** and increases the value by one, returning the mutated value.

### 7.6. `--`: Decrement

The decrements operator works for **numbers**, **arrays**, **strings**

1. For **numbers** decreases the value by one, returning the mutated value.
2. For **arrays** removes the last element and returns it. If array is empty returns **null**
3. For **strings** removes the last character and returns it as a new string. If string is empty returns **null**


### 7.7. `typeof`: Typeof

The typeof operator returns the type of a value as a lowercase string.

## 8. Binary operators

Binary expressions is a expression that process a value with one arguments and returns a result. It may mutate a value.

Most binary expressions use the left value as the main value and the right value as a argument , however, some use the right value as the main value and the left value as a argument 

### 8.1. Member access operators

The following operators deal with member access meaning the left value must be an object and right must be an ident expression and shouldn't be evaluated. Each member operator has a null-safe option that returns null if the left value is null

1. `.`, `?.`: Dot. Gets the property of an object. If the object does not has this property returns null.
2. `.>`, `?.>`: Method. Gets the method of the object by getting a property and binding the object to the property. Expects the property to be a subroutine.
3. `.~`, `?.~`: Delete. Deletes a property and returns the deleted property's value. If the object does not has this property returns null.

### 8.2. `??`: Question/Null coalescing

If the left is not null returns left, if left is null evaluates right and returning it

### 8.3. `->`: Bind operator

Bind operator binds a context into a subroutine. The main value (subroutine) is the right value, and the argument is the left value.

Binding a value to a function that already has a context value replaces the current context value to the left value.

### 8.4. `in`: In Operator

In operator checks is:
1. A substring in a string
2. A element in a array
3. A key in a object

### 8.5. `==`, `~=`: Equality Operators.

Equality operators checks if two values are equal or not equal with deep equality that checks every element or value in arrays and object, subroutines by pointers, other value types by value. In subroutines context doesn't affect equality.

```aqua
sub a() println(it ?? "Hello world") end
let b = "Hello Aqua" -> a
a()
b()
println( a == b)
```

> Output: 
>
> Hello world
>
> Hello Aqua
>
> true

### 8.6. Operators For Numbers

1. `+`: Plus adds two numbers
2. `-`: Subtracts right from left
3. `*`: Multiplies two numbers
4. `/`: Left divided by right
5. `%`: The floating-point remaining of left divided by right
6. `//`: Floor left divided by right
7. `and`: Bitwise and, only for integers
8. `or`: Bitwise not, only for integers
9. `xor`: Bitwise xor, only for integers
10. `>>`: Right shift, only for integers
11. `<<`: Left shift, only for integers
12. `>`, `<`, `>=`, `<=`: Comparing left and right
    
### 8.7.Operators For Booleans

Converting booleans to numbers: the same as if true=1, false=0

1. `+`, `-`, `/`, `//`, `%`, `>>`, `<<`, : Converting booleans to numbers and dong the same as 8.6.
2. `*`, `and`: Logical and
3. `or`: Logical or
4. `xor`: Logical xor
5. `>`, `<`, `>=`, `<=`: Comparing left and right. 

### 8.8. Operators For Arrays And Strings

1. `+`: Creates a new array/string with appending the right as the last element or creating a new string of added the left and right strings, without cloning the array but cloning the string. Even if it doesn't clone the array when appending, if the result array isn't cloned the results may depend on the runtime and are unsafe.
2. `[]`: Gets the right'th element of an array or character of string, right should be an integer. If index is out of range returns null. Mutating array element mutates the array but mutating the string character doesn't mutate the string

### 8.9. Operators For Objects: `[]`

Get's the value with the from right. The same as member, but dynamic.

## 9. Cloning

Different expression evaluations are cloned or not. Was the value really cloned depends by the expression type for example let, prefix ptr, direct assigment expressions bypass even required cloning. Usually if cloning the value isn't needed it doesn't require cloning like in binary expressions since cloning is always deep and may cost too much. Sometimes bypassing cloning might be unsafe and the behavior will depend on the runtime.
   
