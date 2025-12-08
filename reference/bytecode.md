# Ovum Intermediate Language (Bytecode)

The Ovum Intermediate Language (Ovum bytecode) is a stack-based virtual machine instruction set that serves as the compilation target for Ovum source code. This document describes the bytecode syntax, structure, and available commands.

See also: [bytecode command list](./bytecode_commands.md) and [bytecode examples](./bytecode_examples.md).

## General Syntax

Ovum bytecode follows a stack-based execution model where operations consume values from the stack and push results back onto the stack. The syntax uses WASM-like code blocks enclosed in curly braces `{}`.

### Stack Operation Convention

OIL follows a consistent convention for all stack operations: **operands are pushed from right to left** (the rightmost operand is pushed first, the leftmost operand is pushed last).

This convention applies to:
* **Function calls**: Arguments are pushed right-to-left
* **Binary operations**: Right operand is pushed first, then left operand
* **Comparison operations**: Right operand is pushed first, then left operand
* **All stack-based operations**: Right operand first, left operand last

**Function Arguments:**
* **Argument pushing**: Arguments are pushed onto the stack from **right to left** (the rightmost argument is pushed first, the leftmost argument is pushed last)
* **Argument popping**: When a function executes, it pops arguments from the stack from **left to right** (the leftmost argument is at `LoadLocal 0`, the next at `LoadLocal 1`, etc.)

This means that for a function call `foo(a, b, c)`, the bytecode should:
1. Push `c` (rightmost argument)
2. Push `b` (middle argument)
3. Push `a` (leftmost argument)
4. Call `foo`

Inside the function, `LoadLocal 0` will access `a`, `LoadLocal 1` will access `b`, and `LoadLocal 2` will access `c`.

**Binary Operations:**
For binary operations like `a + b`, `a * b`, `a < b`, etc., the right operand is pushed first, then the left operand:
1. Push `b` (right operand)
2. Push `a` (left operand)
3. Execute operation (e.g., `IntAdd`, `IntMultiply`, `IntLessThan`)

**Examples:**
```oil
// Function definition: fun Add(x: Int, y: Int): Int
function:2 _Global_Add_int_int {
    LoadLocal 0  // x (first argument, leftmost)
    LoadLocal 1  // y (second argument, rightmost)
    IntAdd
    Return
}

// Function call: Add(5, 3)
PushInt 3  // Push rightmost argument first
PushInt 5  // Push leftmost argument last
Call _Global_Add_int_int

// Binary operation: x + y
LoadLocal 1  // y (right operand)
LoadLocal 0  // x (left operand)
IntAdd

// Comparison: i <= 5
PushInt 5  // 5 (right operand)
LoadLocal 1  // i (left operand)
IntLessEqual

// String concatenation: "Hello" + "World"
PushString "World"  // right operand
PushString "Hello"  // left operand
StringConcat
```

### Control Flow Structures

#### If Statements

If statements consist of separate condition blocks and execution blocks connected by the `then` keyword. The syntax supports `if`, `else if`, and `else` branches:

```oil
{
    // If condition is true, execute this block
    if {
        LoadLocal 0
        PushInt 0
        IntGreaterThan
    } then {
        PushString "Positive number"
        Print
    }
    // Optional else if branches
    else if {
        LoadLocal 0
        PushInt 0
        IntLessThan
    } then {
        PushString "Negative number"
        Print
    }
    // Default else block
    else {
        PushString "Zero"
        Print
    }
}
```

#### While Loops

While loops consist of separate condition blocks and execution blocks connected by the `then` keyword. For loops are reduced to while loops during compilation:

```oil
{
    // Initialize loop variable
    PushInt 0
    SetLocal 0
    // While loop
    while {
        LoadLocal 0
        PushInt 10
        IntLessThan
    } then {
        // Execution block
        LoadLocal 0
        IntToString
        Print
        // Increment loop variable
        LoadLocal 0
        PushInt 1
        IntAdd
        SetLocal 0
    }
}
```

#### Init-Static Block

The `init-static` block is a unique block that must appear exactly once in a bytecode file. It contains initialization code that is executed before the `Main` function is called, primarily for setting up static variables:

```oil
// Static initialization block (must be unique, executed before Main)
init-static {
    PushInt 42
    SetStatic 0  // Set static variable at index 0
    PushString "Initialized"
    SetStatic 1  // Set static variable at index 1
    PushFloat 3.14
    SetStatic 2  // Set static variable at index 2
}
```

#### Functions

Functions are defined with optional keywords and contain a sequence of bytecode instructions:

```oil
// Basic function
function:2 _Global_CalculateSum_int_int {
    LoadLocal 0
    LoadLocal 1
    IntAdd
    Return
}

// Pure function with argument descriptions
// Arguments: int (int), int (int)
pure(int, int) function:2 _Global_IsEven_int_int {
    LoadLocal 0
    PushInt 2
    IntModulo
    PushInt 0
    IntEqual
    Return
}

// Pure function with mixed argument types
// Arguments: String, int
pure(String, int) function:2 _Global_StringRepeat_String_int {
    LoadLocal 0  // String reference
    LoadLocal 1  // Int copy
    Call _Global_StringRepeatImpl_String_int
    Return
}

// Function with no JIT optimization
no-jit function:1 _Global_DebugPrint_Object {
    LoadLocal 0
    CallVirtual _ToString_<C>
    PrintLine
    Return
}

// Main function (entry point)
function:1 _Global_Main_StringArray {
    PushString "Hello, World!"
    PrintLine
    PushInt 0
    Return
}
```

## VTable Structure

VTable is a data structure that maps method names to their implementations. 
It is used to resolve method calls at runtime. 
VTable is stored in the object itself and is accessed by the object's vtable index.
VTable is also used to resolve field access at runtime.

### VTable Definition Syntax

VTable definitions map interface methods and field information to their implementations and contains class size information in bytes:

```oil
vtable <ClassName> {
    size: <size>  // Includes vtable index (4 bytes) + badge (4 bytes) + field sizes in bytes
    interfaces {
        <InterfaceName>,<InterfaceName>, ...
    }
    methods {
        <methodNameWithoutClassName>: <realFunctionName>
        <methodNameWithoutClassName>: <realFunctionName>
        ...
    }
    vartable {
        <fieldName>: <typename|Object>@<offset> // typename for fundamental types, Object for user-defined types
        <fieldName>: <typename|Object>@<offset> // typename for fundamental types, Object for user-defined types
        ...
    }
}
```

### Field Layout and Memory Management

* **VTable index**: Each object starts with an 4-byte index to its vtable (offset 0)
* **Service information (badge)**: Each object contains 4-byte field for service information called badge (e.g. GC generation, reference count, etc.). Starts after vtable index.
* **Field offsets**: Each field has a specific byte offset within the object (starting after vtable index and badge)
* **Size calculation**: Total class size = 4 bytes (vtable index) + 4 bytes (badge) + sum of all field sizes + alignment
* **Memory alignment**: Fields are aligned according to their type size
* **Vartable**: Maps field names to their byte offsets for runtime field access

## Function Name Mangling

Function names are mangled to include fully qualified class names for method resolution:

### Mangling Rules

* **Global functions**: `_Global_<functionName>_<arg1Type>_<arg2Type>_..._<argNType>`
* **Class methods**: `_<ClassName>_<methodName>_<constFlag>_<arg1Type>_<arg2Type>_..._<argNType>`
* **Nested classes**: `_<OuterClass>_<InnerClass>_<methodName>_<constFlag>_<arg1Type>_<arg2Type>_..._<argNType>`

> **Note**: Namespaces are included in the class and function names, e.g. `_test_FooBar_Int` for `test::FooBar(a : Int)` function.

**Argument mutability indicators:**
* `<M>` - Mutable argument (`var` parameter)
* `<C>` - Const argument (default, immutable parameter, omitted)

**Const method flag:**
* `<M>` - Mutable method (can modify object state)
* `<C>` - Const method (cannot modify object state)

### Examples

```oil
// Source code
fun Foo(a: Int, b: bool, var s: String): String { ... }

class MathUtils implements IComparable {
    fun Add(a: Int, b: Int): Int { ... }
    pure fun IsEven(n: int): Bool { ... }
    mut fun IncrementCounter(): Void { ... }
    override fun IsLess(other: Object): Bool { ... }
}

fun Main(args: StringArray): Int { ... }

// Mangled names
_Global_Foo_Int_bool_<M>String
_MathUtils_Add_<M>_Int_Int
_MathUtils_IsEven_<C>_int
_MathUtils_IncrementCounter_<M>
_MathUtils_IsLess_<C>_Object
_Global_Main_StringArray
```

## Notes

* All command names use PascalCase convention
* Stack operations are performed on the top elements of the stack
* Local variables are indexed starting from 0
* The bytecode is designed to be platform-independent and executed by the Ovum Virtual Machine (OVM)
* JIT compilation optimizes frequently executed functions to native machine code
