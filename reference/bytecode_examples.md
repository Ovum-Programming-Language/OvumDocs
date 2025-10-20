# Example Bytecode Programs

This documents contains example bytecode programs that demonstrate the compilation of Ovum source code to OIL bytecode.

## Simple Program

**OIL Bytecode Target:**
```oil
// Simple program that prints numbers 1 to 5
function _Global_Main_StringArray {
    PushInt 1
    SetLocal 0
    
    while {
        LoadLocal 0
        PushInt 5
        IntLessEqual
    } then {
        LoadLocal 0
        Call _Global_ToString_Int
        Call _Global_PrintLine_String
        
        LoadLocal 0
        PushInt 1
        IntAdd
        SetLocal 0
    }
    
    PushInt 0
    Return
}
```

**Ovum Source Code:**
```ovum
fun Main(args: StringArray): Int {
    var i: Int = 1
    
    while i <= 5 {
        sys::PrintLine(ToString(i))
        i = i + 1
    }
    
    return 0
}
```

## Function with Multiple Arguments

**OIL Bytecode Target:**
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
        LoadLocal 5
        LoadLocal 2
        IntLessThan
    } then {
        LoadLocal 4
        LoadLocal 3
        StringConcat
        SetLocal 4
        
        LoadLocal 5
        PushInt 1
        IntAdd
        SetLocal 5
    }
    
    LoadLocal 4
    Return
}
```

**Ovum Source Code:**
```ovum
fun CalculateArea(width: Int, height: Int): Int {
    return width * height
}

fun ProcessStrings(first: String, second: String, repeatCount: Int): String {
    val concatenated: String = first + second
    var result: String = ""
    var counter: Int = 0
    
    while counter < repeatCount {
        result = result + concatenated
        counter = counter + 1
    }
    
    return result
}
```

## Class Declarations and Method Definitions

**OIL Bytecode Target:**
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

**Ovum Source Code:**
```ovum
interface IComparable {
    fun IsLess(other: Object): Bool
}

interface IStringConvertible {
    fun ToString(): String
}

class Point implements IComparable, IStringConvertible {
    public var x: Int
    public var y: Int
    
    public fun Point(x: Int, y: Int): Point {
        this.x = x
        this.y = y
        return this
    }
    
    public fun GetDistance(): Float {
        return CalculateDistance(this.x, this.y)
    }
    
    private fun CalculateDistance(x: Int, y: Int): Float {
        return sys::Sqrt(x * x + y * y)
    }
    
    public override fun IsLess(other: Object): Bool {
        // Implementation of comparison logic
        return false  // Placeholder
    }
    
    public override fun ToString(): String {
        // Implementation of string conversion
        return "Point(" + x.ToString() + ", " + y.ToString() + ")"
    }
}

fun Main(args: StringArray): Int {
    val point: Point = Point(3, 4)
    val distance: Float = point.GetDistance()
    sys::PrintLine(distance.ToString())
    
    return 0
}
```

## Interface-Based Polymorphism

**OIL Bytecode Target:**
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

**Ovum Source Code:**
```ovum
interface IShape {
    fun GetArea(): Float
    fun GetPerimeter(): Float
}

class Rectangle implements IShape {
    public var Width: Float
    public var Height: Float
    
    public fun Rectangle(width: Float, height: Float): Rectangle {
        this.Width = width
        this.Height = height
        return this
    }
    
    public override fun GetArea(): Float {
        return Width * Height
    }
    
    public override fun GetPerimeter(): Float {
        return 2.0 * (Width + Height)
    }
}

class Circle implements IShape {
    public var Radius: Float
    
    public fun Circle(radius: Float): Circle {
        this.Radius = radius
        return this
    }
    
    public override fun GetArea(): Float {
        return 3.14159 * Radius * Radius
    }
    
    public override fun GetPerimeter(): Float {
        return 2.0 * 3.14159 * Radius
    }
}

fun ProcessShape(shape: IShape): (Float, Float) {
    val area: Float = shape.GetArea()
    val perimeter: Float = shape.GetPerimeter()
    return (area, perimeter)
}

fun Main(args: StringArray): Int {
    val rectangle: Rectangle = Rectangle(5.0, 3.0)
    val (rectArea, rectPerimeter): (Float, Float) = ProcessShape(rectangle)
    
    val circle: Circle = Circle(2.5)
    val (circleArea, circlePerimeter): (Float, Float) = ProcessShape(circle)
    
    return 0
}
```

## Pure Functions with Mixed Argument Types

**OIL Bytecode Target:**
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

**Ovum Source Code:**
```ovum
pure fun CalculateQuadratic(a: Int, b: Int, c: Int): Int {
    // Calculate discriminant: b^2 - 4ac
    return b * b - 4 * a * c
}

pure fun BitwiseOperations(first: Int, second: Int): Int {
    val andResult: Int = first & second
    val orResult: Int = first | second
    val xorResult: Int = first ^ second
    
    // Return XOR result (most interesting)
    return xorResult
}

fun Main(args: StringArray): Int {
    val discriminant: Int = CalculateQuadratic(2, 5, 3)
    sys::PrintLine(discriminant.ToString())
    
    val xorResult: Int = BitwiseOperations(15, 7)
    sys::PrintLine(xorResult.ToString())
    
    return 0
}
```

## Summary

These examples demonstrate how Ovum source code compiles to the OIL bytecode format:

1. **Simple Program**: Basic control flow with while loops and local variables
2. **Function with Multiple Arguments**: Functions with different argument types (copy vs reference)
3. **Class Declarations**: Object-oriented features with constructors, methods, and interfaces
4. **Interface-Based Polymorphism**: Runtime polymorphism through interface dispatch
5. **Pure Functions**: Functions marked as pure for optimization, with mixed argument types

The compilation process transforms high-level Ovum constructs into stack-based bytecode instructions, with proper name mangling for functions and vtables for object-oriented features.
