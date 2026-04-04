# Arc Programming Language

## Definition

Signature: `<name>: <type> : <default>`

An immutable variable

```arc
a: i32 : 100;

// Infer from value
b :: 50;
```

## Variable & Field

Signature: `<name>: <type> = <default>`

A mutable variable

```arc
a: i32 = 100;

// Infer from value
// NOTE: Fields must have a type
b := 50;

_const: const i32 = 100; // This is the same as a Definition
```

## Function

Signature: `fn([parameter, ...]) [type] [{ <body> }]`

```arc
main :: fn() {}
add :: fn(a: i32, b: i32) i32 {
  return a + b;
}
```

## Struct

Signature: `struct { <fields/definitions> }`

```arc
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

```arc
Example :: enum {
  A, // 0
  B, // 1
  C = 10,
  D, // 11
}
```

## Namespace

Signature: `{ <definitions> }`

```arc
Debug :: {
  println :: fn(...) { ... }
}
```

## Return

Signature: `return [expression];`

Returns the enclosing function with `[expression]`

```arc
return a + b;
```

## If, Else, Else-If

Signature: `if (<condition>) { ... } [else <else_body>]`

```arc
result: bool = true;
if result {
  // Then body
} else if false {
  // Else-If Body
} else {
  // Else Body
}
```

## While

Signature: `while (<condition>) { ... }`

```arc
while true {
  // Do something
}
```

## Switch

Signature: `switch (<input>) { <cases> }`

```arc
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

Signature: `comptime <function/expression>`

Functions or expressions with `comptime` appended are evaluated at compile-time.  

A compile-time error is emitted if an expression cannot be evaluated at compile-time.

```arc
comptime {
  // Executes at compile-time
}

_comptime_function :: comptime fn() {
  // Executes at compile-time, when called
}

value: i32 = comptime (1 + 2); // Evaluates to '3' at compile-time
```

## Assembly

Signature: `asm { <instruction> [<%reg/var/literal>, ...]; ... }`

Instructions are architecture specific and cannot be mixed.

NOTE: Assembly cannot execute at compile-time.

```arc
a: const u64 = 10;
result: u64;
asm { // RISC-V instructions
  li %t1, 5;
  add result, a, %t1;
}
```

## Uninitialized

Signature: `---`

Used as a field's value to not zero-initialize the field.  
It is also used as the body of a function to mark it as externally defined.  

```arc
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

### Constant

Signature: `const <type>`

Constant types cannot be assigned to,  
but constant types can be assigned to non-constant types.

```arc
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

```arc
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

| Symbol | Description |
| ------ | ----------- |
| `+` | Addition |
| `-` | Subtraction |
| `*` | Multiplication |
| `/` | Division |
| `%` | Modulo |
| `\|` | Bitwise Or |
| `^` | Bitwise Xor |
| `&` | Bitwise And |
| `<<` | Bitwise Left Shift |
| `>>` | Bitwise Right Shift |
| `\|\|` | Logical Or |
| `&&` | Logical And |

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

| Symbol | Description |
| ------ | ----------- |
| `-` | Minus |
| `!` | Logical Not |
| `~` | Bitwise Not |
| `&` | Reference |
| `*` | Dereference |

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

Signature: `@<name>(<value>)`

Sets `name` attribute to `value` for the next definition

NOTE: attributes are compile-time

## Builtin

Signature: `#<name>`

### typeid_of

Signature: `#typeid_of(<type>)`
Execution: `Compile Time`

Returns the type's unique `usize` identifier

### typeinfo_of

Signature: `#typeinfo_of(<type>)`
Execution: `Compile Time`

Returns a struct containing info about the type,  
including: it's name, struct fields, function parameters, etc

### size_of

Signature: `#size_of(<type>)`
Execution: `Compile Time`

Returns the size in bytes, of the inputted type

### align_of

Signature: `#align_of(<type>)`
Execution: `Compile Time`

Returns the alignment in bytes, of the inputted type

## Compiler

### Evaluator

Resolves names, infers types, executes compile-time expressions, etc

### Name Mangling

Section: `<name length><name>`

A mangled name is comprised of a file section, any amount of parent section's, and a definition section

Examples:

```
8main.arc4main
"main" in file "main.arc"

11example.arc5Tests4Data
"Data" in "Tests" in file "example.arc"
```
