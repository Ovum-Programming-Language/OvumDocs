# Ovum Intermediate Language (Bytecode)

The Ovum Intermediate Language (Ovum bytecode) is a stack-based virtual machine instruction set that serves as the compilation target for Ovum source code. This document describes the bytecode syntax, structure, and available commands.

See also: [bytecode command list](./bytecode_commands.md) and [bytecode examples](./bytecode_examples.md).

## General Syntax

Ovum bytecode follows a stack-based execution model where operations consume values from the stack and push results back onto the stack. The syntax uses WASM-like code blocks enclosed in curly braces `{}`.

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

#### Functions

Functions are defined with optional keywords and contain a sequence of bytecode instructions:

```oil
// Basic function
function _Global_CalculateSum_int_int {
    LoadLocal 0
    LoadLocal 1
    IntAdd
    Return
}

// Pure function with argument descriptions
// Arguments: Copy:8 (int), Copy:8 (int)
pure(Copy:8, Copy:8) function _Global_IsEven_int_int {
    LoadLocal 0
    PushInt 2
    IntModulo
    PushInt 0
    IntEqual
    Return
}

// Pure function with mixed argument types
// Arguments: Ref (String), Copy:8 (int)
pure(Ref, Copy:8) function _Global_StringRepeat_String_int {
    LoadLocal 0  // String reference
    LoadLocal 1  // Int copy
    Call _Global_StringRepeatImpl_String_int
    Return
}

// Function with no JIT optimization
no-jit function _Global_DebugPrint_Object {
    LoadLocal 0
    CallVirtual _ToString_<C>
    PrintLine
    Return
}

// Main function (entry point)
function _Global_Main_StringArray {
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
        <fieldName>: <Ref|Copy:<size>>@<offset>
        <fieldName>: <Ref|Copy:<size>>@<offset>
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

**Argument mutability indicators:**
* `<M>` - Mutable argument (`var` parameter)
* `<C>` - Const argument (default, immutable parameter)

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
* Function calls preserve the stack state except for return values
* The bytecode is designed to be platform-independent and executed by the Ovum Virtual Machine (OVM)
* JIT compilation optimizes frequently executed bytecode sequences to native machine code
