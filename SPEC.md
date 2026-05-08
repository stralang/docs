# Stra Programming Language

## Definition

Signature: `<name>: <type> : <default>`

An immutable variable

```stra
a: i32 : 100;

// Infer from value
b :: 50;
```

## Variable & Field

Signature: `<name>: <type> = <default>`

A mutable variable

```stra
a: i32 = 100;

// Infer from value
// NOTE: Fields must have a type
b := 50;

_const: const i32 = 100; // This is the same as a Definition
```

## Function

Signature: `fn([parameter, ...]) [type] [{ <body> }]`

```stra
main :: fn() {}
add :: fn(a: i32, b: i32) i32 {
  return a + b;
}
```

## Struct

Signature: `struct { <fields/definitions> }`

```stra
Vector3 :: struct {
  x: f32,
  y: f32,
  z: f32,

  // You can `<self>.add(<lhs>)`, -
  // if the first parameter is a pointer to the struct (`&Vector3` or `&const Vector3`)
  add :: fn(self: &Vector3, lhs: &const Vector3) {
    ...
  }
}
```

## Enum

Signature: `enum [integer_type] { <enumerators/definitions> }`

```stra
Example :: enum { // Default tag is `u32`
  A, // 0
  B, // 1
  C = 10,
  D, // 11
}
```

## Union

Signature: `union [integer_type/void] { <variants/definitions> }`

The tag is placed after the data in memory  
`void` is used to make C-style (tag-less) unions  

```stra
Example :: union { // Default tag is `u32`
  integer: i32,
  float: f32,
}
```

## Namespace

Signature: `{ <definitions> }`

```stra
Debug :: {
  println :: fn(...) { ... }
}
```

## Return

Signature: `return [expression];`

Returns the enclosing function with `[expression]`

```stra
return a + b;
```

## If, Else, Else-If

Signature: `if (<condition>) { ... } [else <else_body>]`

```stra
result: bool = true;
if result {
  // Then body
} else if false {
  // Else-If Body
} else {
  // Else Body
}
```

## For

Signatures: `for <conditions> { ... }`

Examples:

```stra
for i in 0..<100 {
  // Iterates from 0 to 99
}

for i in 0..<100; value := iter.next(); value != 100 {
  // Iterates until `value` is null, `100`, or `i` finishes iterating
  // NOTE: the output of `iter.next()` must be a pointer
}

// This is equivalent to the above
i: usize = 0;
for { // Forever loop
  // Check `i`
  if i == 100 {
    break;
  }

  // Get and Check `value`
  value := iter.next();
  if value == null || value == 100 {
    break;
  }

  // Other Code
  i = i + 1; // Increment `i`
}
```

## Switch

Signature: `switch (<input>) { <cases> }`

```stra
switch 1 {
  0 => { ... }
  1 => { ... }
  2 => { ... }
  _ => { ... } // default
}
```

## Break

Signature: `break [name];`

Breaks out of the enclosed loop

## Continue

Signature: `continue [name];`

Jumps to the beginning of the enclosing loop, rechecking the conditional

## Defer

Signature: `defer <statement;/body>`

Copies the `statement`/`body` to the exits of the enclosing body

## Import

Signature: `import "[<package>:]<path>";`

Imports the source code at `<path>`, relative to the importer or `<package>`, as a namespace (`{ ... }`)

## Comptime

Signature: `comptime <statement/expression>` `$<statement/expression>`

Functions or expressions with `comptime` appended are evaluated at compile-time.  

A compile-time error is emitted if an expression cannot be evaluated at compile-time.

```stra
comptime {
  // Executes at compile-time, when it's scope is referenced
}

_comptime_function :: comptime fn() {
  // Executes at compile-time, when called
}

value: i32 = comptime (1 + 2); // Evaluates to '3' at compile-time

// A compile-time variable with no runtime variable
// any reference to the variable is replaced with it's value
$comptime_expr: i32 = 1 + 2;
```

## Assembly

Signature: `asm { <instruction> [<%reg/var/literal>, ...]; ... }`

Instructions are architecture specific and cannot be mixed.

NOTE: Assembly cannot execute at compile-time.

```stra
a: const u64 = 10;
result: u64;
asm { // RISC-V instructions
  li %t1, 5; // load `5` into register `t1`
  add =result, a, %t1; // add `a` and register `t1` and output to `result`
}
```

## Undefined

Signature: `---`

Used as a field's value to not zero-initialize the field.  
It is also used as the body of a function to mark it as externally defined.  

```stra
// the value of `field` is undefined.
// for global variables, it will be externally defined
// NOTE: Uninitialized fields MUST have a known type
field: i32 = ---;

external_fn :: fn() ---; // Externally defined function
```

## Types

### Primitives

| Symbol | Bits | Description |
| ------ | ---- | ----------- |
| `void` | 0 | |
| `bool` | 1 | |
| `uN` | N* | unsigned integer |
| `iN` | N* | signed integer |
| `usize` | native** | unsigned integer |
| `isize` | native** | signed integer |
| `f16` | 16 | floating-point |
| `f32` | 32 | floating-point |
| `f64` | 64 | floating-point |
| `f128` | 128 | floating-point |

*N being the user-defined bit-count  
**native being the bit-count of the target architecture (e.g. 64-bit for x86_64)

### Pointer

Signature: `&<type>`

### Array/Slice

Signature: `[<length>]<type>`

An `array` is a block of memory whose length is compile-time known.  
A `slice` is a pointer whose length is stored with it.  
A `pointer array` is a helper type to allow indexing into pointers.  

If `<length>` is not provided then it's a `slice`.  
If `<length>` is `*` then it's a `pointer slice`.  

An `array`/`slice` can be indexed: `<identifier>[<index>]`, -  
and converted to a `slice`: `<identifier>[<range>]` (requires at least `lhs` or `rhs`)

### SIMD

Signature: `<primitive>x<N>`

Operators treat SIMD as the underlying type

### Constant

Signature: `const <type>`

Constant types cannot be assigned to,  
but constant types can be assigned to non-constant types.

```stra
a: i32 = 100; // Can be assigned
b: const i32 = 50; // Cannot be assigned

// the resulting type of `a + b` is `const i32`
// which can be assigned to `a`
a = a + b;

// `ptr` can be assigned to, but not the pointed-to value
ptr: &const i32;
ptr = &100;
// *ptr = 50; // Error
```

### Meta Type

Signature: `type`

Meta types only get evaluated at compile-time,  
and MUST have a known underlying type during code generation

```stra
_struct_type: type = struct {}
_fn_type: type = fn();
// NOTE: in both cases `type` will be inferred

_struct_usage: _struct_type; // the "_struct_type" meta type becomes a `struct` type
```

## Operators

### Assign

| Symbol | Description |
| ------ | ----------- |
| `=` | Assign |

### Arithmetic

| Symbol | Description | Applies To |
| ------ | ----------- | ---------- |
| `+` | Addition | Integer, Float |
| `-` | Subtraction | Integer, Float |
| `*` | Multiplication | Integer, Float |
| `/` | Division | Integer, Float |
| `%` | Modulo | Integer, Float |
| `\|` | Bitwise Or | Integer |
| `^` | Bitwise Xor | Integer |
| `&` | Bitwise And | Integer |
| `<<` | Bitwise Left Shift | Integer |
| `>>` | Bitwise Right Shift | Integer |
| `\|\|` | Logical Or | Bool |
| `&&` | Logical And | Bool |

### Equality

| Symbol | Description |
| ------ | ----------- |
| `==` | Equal to |
| `!=` | Not equal to |
| `<` | Less then |
| `>` | Greater then |
| `<=` | Less then or Equal to |
| `>=` | Greater then or Equal to |

### Access

| Symbol | Description |
| ------ | ----------- |
| `.` | Member Access |

### Unary

| Symbol | Description | Applies To |
| ------ | ----------- | ---------- |
| `-` | Minus | Integer, Float |
| `!` | Logical Not | Bool |
| `~` | Bitwise Not | Integer |
| `&` | Reference | * |
| `*` | Dereference | Pointer |

### Special

| Name | Description |
| ------ | ----------- |
| `as` | casts `lhs` to type `rhs`* |
| `bitcast` | bitcasts `lhs` to type `rhs`* |

*`rhs` must be compile-time known, and the size, and alignment, must be the same as `lhs`

### Range

The Range operators are only supported in specific contexts

| Name | Description |
| ---- | ----------- |
| `..<` | `lhs` to `rhs`-1 |
| `..=` | `lhs` to `rhs` |

## Attributes

Signature: `@(<name> [= <value>])`

Sets `name` attribute to `value` for the next definition

NOTE: attributes are compile-time

Example:

```stra
@(example = "value")
example_1 :: 1;

@(a = 0, b = 1) // Multiple attributes can be added
example_2 :: 2;
```

## Builtin Package

### Atomics

Signature: `atomic*(<pointer>, [...], <order>)`  
Execution: `Runtime`

#### Load, Store

- `atomicLoad(<pointer>, <order>)`  
- `atomicStore(<pointer>, <data>, <order>)`  

#### Compare Exchange

- `atomicCompareExchange(<pointer>, <expected>, <desired>, <success_order>, <failure_order>)`  

#### Read-Modify-Write

- `atomicFetchAdd(<pointer>, <increment>, <order>)`  
- `atomicFetchSub(<pointer>, <decrement>, <order>)`
- `atomicFetchAnd(<pointer>, <rhs>, <order>)`  
- `atomicFetchOr(<pointer>, <rhs>, <order>)`  
- `atomicFetchNand(<pointer>, <rhs>, <order>)`  
- `atomicFetchXor(<pointer>, <rhs>, <order>)`  

### typeid_of

Signature: `typeid_of(<type>)`  
Execution: `Compile Time`

Returns the type's unique `usize` identifier

### typeinfo_of

Signature: `typeinfo_of(<type>)`  
Execution: `Compile Time`

Returns a struct containing info about the type,  
including: it's name, struct fields, function parameters, etc

### size_of

Signature: `size_of(<type>)`  
Execution: `Compile Time`

Returns the size in bytes, of the inputted type

### align_of

Signature: `align_of(<type>)`  
Execution: `Compile Time`

Returns the alignment in bytes, of the inputted type

## Builtin Attributes

- `link_name` - The name used for linking
- `builtin` - Marks the definition as provided by the compiler

## Name Mangling

Section: `<name length><name>`

A mangled name is comprised of a file section, any amount of parent section's, and a definition section

Examples:

```
9main.stra4main
"main" in file "main.stra"

12example.stra5Tests4Data
"Data" in "Tests" in file "example.stra"
```

## Comments

- `// <comment>` - single-line comment
- `/* <comment> */` -  multi-line comment
