# Ovum Intermediate Language (Bytecode)

The Ovum Intermediate Language (Ovum bytecode) is a stack-based virtual machine instruction set that serves as the compilation target for Ovum source code. This document describes the bytecode syntax, structure, and available commands.

## General Syntax

Ovum bytecode follows a stack-based execution model where operations consume values from the stack and push results back onto the stack. The syntax uses WASM-like code blocks enclosed in curly braces `{}`.

### Control Flow Structures

#### If Statements

If statements consist of separate condition blocks and execution blocks. The syntax supports `if`, `else if`, and `else` branches:

```oil
{
    // Condition block - pushes Bool value to stack
    {
        LoadLocal 0
        PushInt 0
        IntGreaterThan
    }
    
    // If condition is true, execute this block
    if {
        PushString "Positive number"
        Call Print
    }
    
    // Optional else if branches
    else if {
        // Condition block for else if
        {
            LoadLocal 0
            PushInt 0
            IntLessThan
        }
        
        // Execution block for else if
        if {
            PushString "Negative number"
            Call Print
        }
    }
    
    // Default else block
    else {
        PushString "Zero"
        Call Print
    }
}
```

#### While Loops

While loops consist of separate condition blocks and execution blocks. For loops are reduced to while loops during compilation:

```oil
{
    // Initialize loop variable
    PushInt 0
    SetLocal 0
    
    // While loop
    while {
        // Condition block - pushes Bool value to stack
        {
            LoadLocal 0
            PushInt 10
            IntLessThan
        }
        
        // Execution block
        {
            LoadLocal 0
            Call ToString
            Call Print
            
            // Increment loop variable
            LoadLocal 0
            PushInt 1
            IntAdd
            SetLocal 0
        }
    }
}
```

#### Functions

Functions are defined with optional keywords and contain a sequence of bytecode instructions:

```oil
// Basic function
function CalculateSum {
    LoadLocal 0
    LoadLocal 1
    IntAdd
    Return
}

// Pure function with argument descriptions
// Arguments: Copy:8 (Int), Copy:8 (Int)
pure(Copy:8, Copy:8) function IsEven {
    LoadLocal 0
    PushInt 2
    IntModulo
    PushInt 0
    IntEqual
    Return
}

// Pure function with mixed argument types
// Arguments: Ref (String), Copy:8 (Int)
pure(Ref, Copy:8) function StringRepeat {
    LoadLocal 0  // String reference
    LoadLocal 1  // Int copy
    Call StringRepeatImpl
    Return
}

// Function with no JIT optimization
no-jit function DebugPrint {
    LoadLocal 0
    Call ToString
    Call Print
    Return
}

// Main function (entry point)
function Main {
    PushString "Hello, World!"
    Call Print
    PushInt 0
    Return
}
```

## Class Declarations and VTable Structure

### Class Declaration Syntax

Class declarations in Ovum bytecode specify the class metadata, size, and vtable information separately from method implementations:

```oil
class ClassName {
    // Class metadata
    size: <size> bytes  // Includes vtable pointer (8 bytes) + field sizes
    vtable: <VTableName>
    
    // Field layout (offsets start after vtable pointer)
    field "<fieldName>": <type> (offset <offset>)
    field "<fieldName>": <type> (offset <offset>)
    ...
}
```

### VTable Definition Syntax

VTable definitions map interface methods, class methods, and field information to their implementations:

```oil
vtable <VTableName> {
    interface <InterfaceName> {
        <methodName>: <mangledFunctionName>
        <methodName>: <mangledFunctionName>
    }
    methods {
        <methodName>: <mangledFunctionName>
        <methodName>: <mangledFunctionName>
    }
    vartable {
        "<fieldName>": <offset>
        "<fieldName>": <offset>
    }
}
```

### Field Layout and Memory Management

* **VTable pointer**: Each object starts with an 8-byte pointer to its vtable (offset 0)
* **Field offsets**: Each field has a specific byte offset within the object (starting after vtable pointer)
* **Size calculation**: Total class size = 8 bytes (vtable pointer) + sum of all field sizes + alignment
* **Memory alignment**: Fields are aligned according to their type size
* **Vartable**: Maps field names to their byte offsets for runtime field access

## Function Name Mangling

Function names are mangled to include fully qualified class names for method resolution:

### Mangling Rules

* **Global functions**: `_Global_<functionName>_<arg1Type>_<arg2Type>_..._<argNType>`
* **Class methods**: `_<ClassName>_<methodName>_<constFlag>_<arg1Type>_<arg2Type>_..._<argNType>`
* **Nested classes**: `_<OuterClass>_<InnerClass>_<methodName>_<constFlag>_<arg1Type>_<arg2Type>_..._<argNType>`
* **Pure functions**: `_Pure_<functionName>_<arg1Type>_<arg2Type>_..._<argNType>` or `_Pure_<ClassName>_<methodName>_<constFlag>_<arg1Type>_<arg2Type>_..._<argNType>`

**Argument mutability indicators:**
* `<M>` - Mutable argument (`var` parameter)
* `<C>` - Const argument (default, immutable parameter)

**Const method flag:**
* `<C>` - Const method (cannot modify object state)
* `<M>` - Mutable method (default, can modify object state)

### Examples

```oil
// Source code
fun Foo(a: Int, b: bool, var s: String): String { ... }
fun Bar(a: Int, b: bool, var s: String): String { ... }  // const method

class MathUtils implements IComparable {
    fun Add(a: Int, b: Int): Int { ... }
    pure fun IsEven(n: int): Bool { ... }
    mut fun IncrementCounter(): Void { ... }
    override fun IsLess(other: Object): Bool { ... }
}

fun Main(args: StringArray): Int { ... }

// Mangled names
_Global_Foo_Int_bool_<M>String
_Global_Bar_<C>_Int_bool_<M>String
_MathUtils_Add_<M>_Int_Int
_Pure_MathUtils_IsEven_<C>_int
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
