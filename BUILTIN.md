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
