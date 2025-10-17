# Example Bytecode Programs

## Simple Program

```oil
// Simple program that prints numbers 1 to 5
function _Global_Main_StringArray {
    PushInt 1
    SetLocal 0
    
    while {
        // Condition block
        {
            LoadLocal 0
            PushInt 5
            IntLessEqual
        }
        
        // Execution block
        {
            LoadLocal 0
            Call _Global_ToString_Int
            Call _Global_PrintLine_String
            
            LoadLocal 0
            PushInt 1
            IntAdd
            SetLocal 0
        }
    }
    
    PushInt 0
    Return
}
```

## Function with Multiple Arguments

```oil
// Function that calculates area of rectangle
// Arguments: Copy:8 (int), Copy:8 (int)
function _Global_CalculateArea_int_int {
    LoadLocal 0  // width
    LoadLocal 1  // height
    IntMultiply
    Return
}

// Function that processes multiple string arguments
// Arguments: Ref (String), Ref (String), Copy:8 (int)
function _Global_ProcessStrings_String_String_int {
    LoadLocal 0  // first string
    LoadLocal 1  // second string
    StringConcat
    LoadLocal 2  // repeat count
    
    // Implementation of string repeat
    SetLocal 3  // store concatenated string
    PushString ""  // result string
    SetLocal 4
    
    PushInt 0
    SetLocal 5  // counter
    
    while {
        // Condition block
        {
            LoadLocal 5
            LoadLocal 2
            IntLessThan
        }
        
        // Execution block
        {
            LoadLocal 4
            LoadLocal 3
            StringConcat
            SetLocal 4
            
            LoadLocal 5
            PushInt 1
            IntAdd
            SetLocal 5
        }
    }
    
    LoadLocal 4
    Return
}
```

## Class Declarations and Method Definitions

```oil
// Class declaration with size and vtable information
class Point {
    // Class metadata
    size: 24 bytes  // 8 bytes (vtable pointer) + 2 * int (8 bytes each)
    vtable: PointVTable
    
    // Field layout (offsets start after vtable pointer)
    field "x": int (offset 8)
    field "y": int (offset 16)
}

// VTable definition for Point class
vtable PointVTable {
    interface IComparable {
        IsLess: _Point_IsLess_<C>_Object
    }
    interface IStringConvertible {
        ToString: _Point_ToString_<C>
    }
    methods {
        GetDistance: _Point_GetDistance_<C>
        CalculateDistance: _Point_CalculateDistance_<C>_int_int
    }
    vartable {
        "x": 8
        "y": 16
    }
}

// Constructor implementation
function _Point_Constructor_<M>_int_int {
    LoadLocal 0  // this pointer
    LoadLocal 1  // x argument
    SetField "x"
    
    LoadLocal 0  // this pointer
    LoadLocal 2  // y argument
    SetField "y"
    Return
}

// Method: GetDistance
function _Point_GetDistance_<C> {
    LoadLocal 0  // this pointer
    GetField "x"
    LoadLocal 0  // this pointer
    GetField "y"
    Call _Point_CalculateDistance_<C>_int_int
    Return
}

// Method: CalculateDistance (private)
function _Point_CalculateDistance_<C>_int_int {
    LoadLocal 0  // x
    LoadLocal 1  // y
    IntMultiply
    LoadLocal 0  // x
    LoadLocal 1  // y
    IntMultiply
    IntAdd
    Call _Global_Sqrt_float
    Return
}

// Interface method: IsLess (IComparable)
function _Point_IsLess_<C>_Object {
    LoadLocal 0  // this pointer
    LoadLocal 1  // other object
    // Implementation of comparison logic
    Return
}

// Interface method: ToString (IStringConvertible)
function _Point_ToString_<C> {
    LoadLocal 0  // this pointer
    // Implementation of string conversion
    Return
}

// Usage example
function _Global_Main_StringArray {
    // Create Point object
    PushInt 3
    PushInt 4
    CallConstructor Point
    
    // Call method on object
    Dup
    CallVirtual GetDistance  // GetDistance method by name
    
    Call _Global_ToString_float
    Call _Global_PrintLine_String
    
    PushInt 0
    Return
}
```

## Interface-Based Polymorphism

```oil
// Interface definition
interface IShape {
    fun GetArea(): Float
    fun GetPerimeter(): Float
}

// Rectangle class declaration
class Rectangle {
    // Class metadata
    size: 24 bytes  // 8 bytes (vtable pointer) + 2 * Float (8 bytes each)
    vtable: RectangleVTable
    
    // Field layout (offsets start after vtable pointer)
    field "Width": Float (offset 8)
    field "Height": Float (offset 16)
}

// Rectangle VTable definition
vtable RectangleVTable {
    interface IShape {
        GetArea: _Rectangle_GetArea_<C>
        GetPerimeter: _Rectangle_GetPerimeter_<C>
    }
    vartable {
        "Width": 8
        "Height": 16
    }
}

// Circle class declaration
class Circle {
    // Class metadata
    size: 16 bytes  // 8 bytes (vtable pointer) + 1 * Float (8 bytes)
    vtable: CircleVTable
    
    // Field layout (offsets start after vtable pointer)
    field "Radius": Float (offset 8)
}

// Circle VTable definition
vtable CircleVTable {
    interface IShape {
        GetArea: _Circle_GetArea_<C>
        GetPerimeter: _Circle_GetPerimeter_<C>
    }
    vartable {
        "Radius": 8
    }
}

// Rectangle constructor implementation
function _Rectangle_Constructor_<M>_Float_Float {
    LoadLocal 0  // this pointer
    LoadLocal 1  // width argument
    SetField "Width"
    
    LoadLocal 0  // this pointer
    LoadLocal 2  // height argument
    SetField "Height"
    Return
}

// Rectangle interface method: GetArea
function _Rectangle_GetArea_<C> {
    LoadLocal 0  // this pointer
    GetField "Width"
    LoadLocal 0  // this pointer
    GetField "Height"
    FloatMultiply
    Return
}

// Rectangle interface method: GetPerimeter
function _Rectangle_GetPerimeter_<C> {
    LoadLocal 0  // this pointer
    GetField "Width"
    LoadLocal 0  // this pointer
    GetField "Height"
    FloatAdd
    PushFloat 2.0
    FloatMultiply
    Return
}

// Circle constructor implementation
function _Circle_Constructor_<M>_Float {
    LoadLocal 0  // this pointer
    LoadLocal 1  // radius argument
    SetField "Radius"
    Return
}

// Circle interface method: GetArea
function _Circle_GetArea_<C> {
    LoadLocal 0  // this pointer
    GetField "Radius"
    LoadLocal 0  // this pointer
    GetField "Radius"
    FloatMultiply
    PushFloat 3.14159
    FloatMultiply
    Return
}

// Circle interface method: GetPerimeter
function _Circle_GetPerimeter_<C> {
    LoadLocal 0  // this pointer
    GetField "Radius"
    PushFloat 2.0
    FloatMultiply
    PushFloat 3.14159
    FloatMultiply
    Return
}

// Polymorphic usage through interface
function _Global_ProcessShape_IShape {
    LoadLocal 0  // IShape object
    
    // Call interface method GetArea
    Dup
    CallVirtual GetArea  // GetArea method by name
    
    // Call interface method GetPerimeter
    Dup
    CallVirtual GetPerimeter  // GetPerimeter method by name
    
    Return
}

// Main function demonstrating interface-based polymorphism
function _Global_Main_StringArray {
    // Create Rectangle object
    PushFloat 5.0
    PushFloat 3.0
    CallConstructor Rectangle
    
    // Process as IShape (interface-based polymorphism)
    Call _Global_ProcessShape_IShape
    
    // Create Circle object
    PushFloat 2.5
    CallConstructor Circle
    
    // Process as IShape (interface-based polymorphism)
    Call _Global_ProcessShape_IShape
    
    PushInt 0
    Return
}
```

## Pure Functions with Mixed Argument Types

```oil
// Pure function that calculates mathematical operations without heap allocation
// Arguments: Copy:8 (int), Copy:8 (int), Copy:8 (int)
pure(Copy:8, Copy:8, Copy:8) function _Pure_CalculateQuadratic_int_int_int {
    LoadLocal 0  // a (copy)
    LoadLocal 1  // b (copy)
    LoadLocal 2  // c (copy)
    
    // Calculate discriminant: b^2 - 4ac
    LoadLocal 1  // b
    LoadLocal 1  // b
    IntMultiply  // b^2
    
    LoadLocal 0  // a
    LoadLocal 2  // c
    IntMultiply  // a*c
    PushInt 4
    IntMultiply  // 4ac
    
    IntSubtract  // b^2 - 4ac
    Return
}

// Pure function that performs bitwise operations
// Arguments: Copy:8 (int), Copy:8 (int)
pure(Copy:8, Copy:8) function _Pure_BitwiseOperations_int_int {
    LoadLocal 0  // first number (copy)
    LoadLocal 1  // second number (copy)
    
    // Calculate AND, OR, XOR results
    IntAnd
    SetLocal 2  // AND result
    
    LoadLocal 0
    LoadLocal 1
    IntOr
    SetLocal 3  // OR result
    
    LoadLocal 0
    LoadLocal 1
    IntXor
    SetLocal 4  // XOR result
    
    // Return XOR result (most interesting)
    LoadLocal 4
    Return
}

// Usage of pure functions
function _Global_Main_StringArray {
    PushInt 2
    PushInt 5
    PushInt 3
    Call _Pure_CalculateQuadratic_int_int_int
    Call _Global_ToString_int
    Call _Global_PrintLine_String
    
    PushInt 15
    PushInt 7
    Call _Pure_BitwiseOperations_int_int
    Call _Global_ToString_int
    Call _Global_PrintLine_String
    
    PushInt 0
    Return
}
```