# Example Bytecode Programs

This documents contains example bytecode programs that demonstrate the compilation of Ovum source code to OIL bytecode.

## Simple Program

**OIL Bytecode Target:**
```oil
// Simple program that prints numbers 1 to 5
function:1 _Global_Main_StringArray {
    PushInt 1
    SetLocal 1
    
    while {
        LoadLocal 1
        PushInt 5
        IntLessEqual
    } then {
        LoadLocal 1
        CallConstructor _Int_int
        Call _Int_ToString_<C>
        PrintLine
        LoadLocal 1
        PushInt 1
        IntAdd
        SetLocal 1
    }
    
    PushInt 0
    Return
}
```

**Ovum Source Code:**
```ovum
fun Main(args: StringArray): Int {
    var i: int = 1
    
    while (i <= 5) {
        sys::PrintLine(Int(i).ToString())
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
function:2 _Global_CalculateArea_int_int {
    LoadLocal 0  // width
    LoadLocal 1  // height
    IntMultiply
    Return
}

// Function that processes multiple string arguments
// Arguments: Ref (String), Ref (String), Copy:8 (int)
function:3 _Global_ProcessStrings_String_String_int {
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
    var counter: int = 0
    
    while (counter < repeatCount) {
        result = result + concatenated
        counter = counter + 1
    }
    
    return result
}
```

## VTable Declarations and Method Definitions

**OIL Bytecode Target:**
```oil
// VTable definition for Point class
vtable Point {
    size: 24 // 4 bytes (vtable index) + 4 bytes (badge) + 2 * int (8 bytes each)
    interfaces {
        IComparable, IStringConvertible
    }
    methods {
        _IsLess_<C>_Object: _Point_IsLess_<C>_Object
        _ToString_<C>: _Point_ToString_<C>
    }
    vartable {
        x: int@8
        y: int@16
    }
}

// Constructor implementation
function:3 _Point_int_int {
    LoadLocal 0  // this pointer: put on stack by VM for constructor
    LoadLocal 1  // x argument
    SetField 0
    LoadLocal 0  // this pointer
    LoadLocal 2  // y argument
    SetField 1
    Return
}

// Method: GetDistance
function:1 _Point_GetDistance_<C> {
    LoadLocal 0  // this pointer
    GetField 0
    LoadLocal 0  // this pointer
    GetField 1
    Call _Point_CalculateDistance_<C>_int_int
    CallConstructor _Float_float
    Return
}

// Method: CalculateDistance (private)
function:2 _Point_CalculateDistance_<C>_int_int {
    LoadLocal 0  // x
    LoadLocal 1  // y
    IntMultiply
    LoadLocal 0  // x
    LoadLocal 1  // y
    IntMultiply
    IntAdd
    IntToFloat
    FloatSqrt
    Return
}

// Interface method: IsLess (IComparable)
function:2 _Point_IsLess_<C>_Object {
    LoadLocal 0  // this pointer
    if {
        LoadLocal 1  // other object
        IsType Point
        BoolNot
    } then {
        PushBool false
        Return
    }
    if {
        LoadLocal 0
        GetField 0
        LoadLocal 1
        GetField 0
        IntNotEqual
    } then {
        LoadLocal 0
        GetField 0
        LoadLocal 1
        GetField 0
        IntLess
        Return
    }
    LoadLocal 0
    GetField 1
    LoadLocal 1
    GetField 1
    IntLess
    Return
}

// Interface method: ToString (IStringConvertible)
function:1 _Point_ToString_<C> {
    PushString "Point("
    LoadLocal 0
    GetField 0
    IntToString
    StringConcat
    PushString ", "
    StringConcat
    LoadLocal 0
    GetField 1
    IntToString
    StringConcat
    PushString ")"
    StringConcat
    Return
}

// Usage example
function:1 _Global_Main_StringArray {
    // Create Point object
    PushInt 3
    PushInt 4
    CallConstructor _Point_int_int
    SetLocal 0
    LoadLocal 0
    Call _Point_GetDistance_<C> // Call regular methods
    Call _Float_ToString_<C>
    PrintLine
    LoadLocal 0
    Call _Point_ToString_<C>
    PrintLine
    
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
    public var x: int
    public var y: int
    
    public fun Point(x: int, y: int): Point {
        this.x = x
        this.y = y
        return this
    }
    
    public fun GetDistance(): Float {
        return CalculateDistance(this.x, this.y)
    }
    
    private fun CalculateDistance(x: int, y: int): float {
        return sys::Sqrt(float(x * x + y * y))
    }
    
    public override fun IsLess(other: Object): Bool {
        if (!(other is Point)) {
            return false
        }
        
        val otherPoint: Point = (other as Point) ?: Point(0, 0)
        
        if (this.x != otherPoint.x) {
            return this.x < otherPoint.x
        }
        
        return this.y < otherPoint.y
    }
    
    public override fun ToString(): String {
        return "Point(" + Int(x).ToString() + ", " + Int(y).ToString() + ")"
    }
}

fun Main(args: StringArray): Int {
    val point: Point = Point(3, 4)
    val distance: Float = point.GetDistance()
    sys::PrintLine(distance.ToString())
    sys::PrintLine(point.ToString())
    
    return 0
}
```

## Interface-Based Polymorphism and Static Variables

**OIL Bytecode Target:**
```oil
init-static {
    PushFloat 3.14159
    SetStatic 0
}

// Rectangle VTable definition
vtable Rectangle {
    size: 24 // 4 bytes (vtable index) + 4 bytes (badge) + 2 * Ref (8 bytes each)
    interfaces {
        IShape
    }
    methods {
        _Rectangle_GetArea_<C>: _Rectangle_GetArea_<C>
        _Rectangle_GetPerimeter_<C>: _Rectangle_GetPerimeter_<C>
    }
    vartable {
        Width: Float@8
        Height: Float@16
    }
}

// Circle VTable definition
vtable Circle {
    size: 16 // 4 bytes (vtable index) + 4 bytes (badge) + 1 * Ref (8 bytes)
    interfaces {
        IShape
    }
    methods {
        _Circle_GetArea_<C>: _Circle_GetArea_<C>
        _Circle_GetPerimeter_<C>: _Circle_GetPerimeter_<C>
    }
    vartable {
        Radius: Float@8
    }
}

// Rectangle constructor implementation
function:3 _Rectangle_float_float {
    LoadLocal 0  // this pointer
    LoadLocal 1  // width argument
    CallConstructor _Float_float
    SetField 0
    LoadLocal 0  // this pointer
    LoadLocal 2  // height argument
    CallConstructor _Float_float
    SetField 1
    Return
}

// Rectangle interface method: GetArea
function:1 _Rectangle_GetArea_<C> {
    LoadLocal 0  // this pointer
    GetField 0
    Unwrap
    LoadLocal 0  // this pointer
    GetField 1
    Unwrap
    FloatMultiply
    CallConstructor _Float_float
    Return
}

// Rectangle interface method: GetPerimeter
function:1 _Rectangle_GetPerimeter_<C> {
    LoadLocal 0  // this pointer
    GetField 0
    Unwrap
    LoadLocal 0  // this pointer
    GetField 1
    Unwrap
    FloatAdd
    PushFloat 2.0
    FloatMultiply
    CallConstructor _Float_float
    Return
}

// Circle constructor implementation
function:2 _Circle_Constructor_<M>_float {
    LoadLocal 0  // this pointer
    LoadLocal 1  // radius argument
    CallConstructor _Float_float
    SetField 0
    Return
}

// Circle interface method: GetArea
function:1 _Circle_GetArea_<C> {
    LoadStatic 0  // PI
    LoadLocal 0  // this pointer
    GetField 0
    Unwrap
    FloatMultiply
    LoadLocal 0  // this pointer
    GetField 0
    Unwrap
    FloatMultiply
    CallConstructor _Float_float
    Return
}

// Circle interface method: GetPerimeter
function:1 _Circle_GetPerimeter_<C> {
    PushFloat 2.0
    LoadStatic 0  // PI
    FloatMultiply
    LoadLocal 0  // this pointer
    GetField 0
    Unwrap
    FloatMultiply
    CallConstructor _Float_float
    Return
}

// Polymorphic usage through interface
function:1 _Global_ProcessShape_IShape {
    LoadLocal 0  // IShape object
    CallVirtual _GetArea_<C>  // GetArea method by name
    SetLocal 1
    LoadLocal 0
    CallVirtual _GetPerimeter_<C>  // GetPerimeter method by name
    SetLocal 2
    PushString "Area: "
    LoadLocal 1
    Call _Float_ToString_<C>
    StringConcat
    PushString ", Perimeter: "
    StringConcat
    LoadLocal 2
    Call _Float_ToString_<C>
    StringConcat
    Return
}

// Main function demonstrating interface-based polymorphism
function:1 _Global_Main_StringArray {
    // Create Rectangle object
    PushFloat 5.0
    PushFloat 3.0
    CallConstructor _Rectangle_float_float
    SetLocal 1
    // Create Circle object
    PushFloat 2.5
    CallConstructor _Circle_float
    SetLocal 2
    LoadLocal 1
    Call _Global_ProcessShape_IShape
    SetLocal 3
    LoadLocal 2
    Call _Global_ProcessShape_IShape
    SetLocal 4
    LoadLocal 3
    PrintLine
    LoadLocal 4
    PrintLine
    PushInt 0
    Return
}
```

**Ovum Source Code:**
```ovum
static val PI: Float = 3.14159

interface IShape {
    fun GetArea(): Float
    fun GetPerimeter(): Float
}

class Rectangle implements IShape {
    public var Width: Float
    public var Height: Float
    
    public fun Rectangle(width: float, height: float): Rectangle {
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
    
    public fun Circle(radius: float): Circle {
        this.Radius = radius
        return this
    }
    
    public override fun GetArea(): Float {
        return PI * Radius * Radius
    }
    
    public override fun GetPerimeter(): Float {
        return 2.0 * PI * Radius
    }
}

fun ProcessShape(shape: IShape): String {
    val area: Float = shape.GetArea()
    val perimeter: Float = shape.GetPerimeter()
    return "Area: " + area.ToString() + ", Perimeter: " + perimeter.ToString()
}

fun Main(args: StringArray): Int {
    val rectangle: Rectangle = Rectangle(5.0, 3.0)
    val circle: Circle = Circle(2.5)
    val result_rectangle: String = ProcessShape(rectangle)
    val result_circle: String = ProcessShape(circle)
    sys::PrintLine(result_rectangle)
    sys::PrintLine(result_circle)
    
    return 0
}
```

## Pure Functions with Mixed Argument Types

**OIL Bytecode Target:**
```oil
// Pure function that calculates mathematical operations without heap allocation
// Arguments: int (int), int (int), int (int)
pure(int, int, int) function:3 _Global_CalculateQuadratic_int_int_int {
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
// Arguments: int (int), int (int)
pure(int, int) function:2 _Global_BitwiseOperations_int_int {
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
function:1 _Global_Main_StringArray {
    PushInt 2
    PushInt 5
    PushInt 3
    Call _Global_CalculateQuadratic_int_int_int
    SetLocal 1 // this line and line below can possibly be optimized out by the compiler
    LoadLocal 1
    IntToString
    PrintLine
    
    PushInt 15
    PushInt 7
    Call _Global_BitwiseOperations_int_int
    SetLocal 2 // this line and line below can possibly be optimized out by the compiler
    LoadLocal 2
    IntToString
    PrintLine
    
    PushInt 0
    Return
}
```

**Ovum Source Code:**
```ovum
pure fun CalculateQuadratic(a: int, b: int, c: int): int {
    // Calculate discriminant: b^2 - 4ac
    return b * b - 4 * a * c
}

pure fun BitwiseOperations(first: int, second: int): int {
    val andResult: int = first & second
    val orResult: int = first | second
    val xorResult: int = first ^ second
    
    // Return XOR result (most interesting)
    return xorResult
}

fun Main(args: StringArray): Int {
    val discriminant: Int = CalculateQuadratic(2, 5, 3)
    sys::PrintLine(ToString(discriminant))
    
    val xorResult: Int = BitwiseOperations(15, 7)
    sys::PrintLine(ToString(xorResult))
    
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
