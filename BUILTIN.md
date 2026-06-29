# Stra Programming Language

Builtin Functions and Global Variables

## Atomics

Signature: `atomic*(<pointer>, [...], <order>)`  
Execution: `Runtime`

### Load, Store

- `atomicLoad(<pointer>, <order>)`  
- `atomicStore(<pointer>, <data>, <order>)`  

### Compare Exchange

- `atomicCompareExchange(<pointer>, <expected>, <desired>, <success_order>, <failure_order>)`  

### Read-Modify-Write

- `atomicFetchAdd(<pointer>, <increment>, <order>)`  
- `atomicFetchSub(<pointer>, <decrement>, <order>)`
- `atomicFetchAnd(<pointer>, <rhs>, <order>)`  
- `atomicFetchOr(<pointer>, <rhs>, <order>)`  
- `atomicFetchNand(<pointer>, <rhs>, <order>)`  
- `atomicFetchXor(<pointer>, <rhs>, <order>)`  

## Target

### Arch

Name: `TARGET_ARCH`
Type: `u8`; integer or enum

The target's architecture

### OS

Name: `TARGET_OS`
Type: `u8`; integer or enum

The target's operating system

### Sub OS

Name: `TARGET_SUB_OS`
Type: `u8`; integer or enum

The target's sub operating system

### Vendor

Name: `TARGET_VENDOR`
Type: `[]u8`

A string of the target system's vendor

### Endian

Name: `TARGET_ENDIAN`
Type: `u8`; integer or enum

The target system's endianness

## Type

### typeInfoOf

Signature: `typeInfoOf(<type>)`  
Execution: `Compile Time`

Returns a struct containing info about the type,  
including: it's name, struct fields, function parameters, etc

### sizeOf

Signature: `sizeOf(<type>)`  
Execution: `Compile Time`

Returns the size in bytes, of the inputted type

### alignOf

Signature: `alignOf(<type>)`  
Execution: `Compile Time`

Returns the alignment in bytes, of the inputted type

## Compilation

### Link Library

Signature: `linkLibrary(<name>, <Static, Dynamic>)`  
Execution: `Compile Time`

Links a static/dynamic library to this binary

### Link Directory

Signature: `linkDirectory(<path>)`  
Execution: `Compile Time`

Adds a path to a library directory to the linker

### Linker Script

Signature: `linkerScript(<path>)`  
Execution: `Compile Time`

Specifies a linker script to be used during the linking phase

NOTE: This should never be used by a library unless explicitly mentioned, or built separately

### Linker Flags

Signature: `linkerFlags(<flags>)`  
Execution: `Compile Time`

Passes user-provided flags to the linker
