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
        PushInt 5  // right operand
        LoadLocal 1  // left operand
        IntLessEqual
    } then {
        LoadLocal 1
        CallConstructor _Int_int
        Call _Int_ToString_<C>
        PrintLine
        PushInt 1  // right operand
        LoadLocal 1  // left operand
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
    LoadLocal 1  // height (right operand)
    LoadLocal 0  // width (left operand)
    IntMultiply
    Return
}

// Function that processes multiple string arguments
// Arguments: Ref (String), Ref (String), Copy:8 (int)
function:3 _Global_ProcessStrings_String_String_int {
    LoadLocal 1  // second string (right operand)
    LoadLocal 0  // first string (left operand)
    StringConcat
    LoadLocal 2  // repeat count
    
    // Implementation of string repeat
    SetLocal 3  // store concatenated string
    PushString ""  // result string
    SetLocal 4
    
    PushInt 0
    SetLocal 5  // counter
    
    while {
        LoadLocal 2  // repeatCount (right operand)
        LoadLocal 5  // counter (left operand)
        IntLessThan
    } then {
        LoadLocal 3  // concatenated (right operand)
        LoadLocal 4  // result (left operand)
        StringConcat
        SetLocal 4
        
        PushInt 1  // right operand
        LoadLocal 5  // left operand
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
    GetField 1  // y (rightmost argument)
    LoadLocal 0  // this pointer
    GetField 0  // x (leftmost argument)
    Call _Point_CalculateDistance_<C>_int_int
    CallConstructor _Float_float
    Return
}

// Method: CalculateDistance (private)
function:2 _Point_CalculateDistance_<C>_int_int {
    // Calculate x * x
    LoadLocal 0  // x (right operand)
    LoadLocal 0  // x (left operand)
    IntMultiply  // x * x, stack: [x*x]
    // Calculate y * y (pushes on top of x*x)
    LoadLocal 1  // y (right operand)
    LoadLocal 1  // y (left operand)
    IntMultiply  // y * y, stack: [y*y, x*x] (y*y on top)
    // Add: x*x + y*y (addition is commutative, so y*y + x*x = x*x + y*y)
    // IntAdd computes top op bottom: y*y + x*x
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
        LoadLocal 1  // other (right operand)
        GetField 0
        LoadLocal 0  // this (left operand)
        GetField 0
        IntNotEqual  // this.x != other.x
    } then {
        LoadLocal 1  // other (right operand)
        GetField 0
        LoadLocal 0  // this (left operand)
        GetField 0
        IntLess  // this.x < other.x
        Return
    }
    LoadLocal 1  // other (right operand)
    GetField 1
    LoadLocal 0  // this (left operand)
    GetField 1
    IntLess  // this.y < other.y
    Return
}

// Interface method: ToString (IStringConvertible)
function:1 _Point_ToString_<C> {
    // "Point(" + Int(x).ToString()
    LoadLocal 0
    GetField 0
    IntToString  // Int(x).ToString() (right operand)
    PushString "Point("  // left operand
    StringConcat  // "Point(" + Int(x).ToString(), stack: [result]
    
    // previous + ", "
    // Stack: [previous]
    PushString ", "  // right operand, stack: [", ", previous]
    Swap  // stack: [previous, ", "]
    StringConcat  // previous + ", ", stack: [result]
    
    // previous + Int(y).ToString()
    // Stack: [previous]
    LoadLocal 0
    GetField 1
    IntToString  // Int(y).ToString() (right operand), stack: [Int(y).ToString(), previous]
    Swap  // stack: [previous, Int(y).ToString()]
    StringConcat  // previous + Int(y).ToString(), stack: [result]
    
    // previous + ")"
    // Stack: [previous]
    PushString ")"  // right operand, stack: [")", previous]
    Swap  // stack: [previous, ")"]
    StringConcat  // previous + ")", stack: [result]
    Return
}

// Usage example
function:1 _Global_Main_StringArray {
    // Create Point object
    PushInt 4  // y (rightmost argument)
    PushInt 3  // x (leftmost argument)
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
    size: 24 // 4 bytes (vtable index) + 4 bytes (badge) + 2 * Object (8 bytes each)
    interfaces {
        IShape
    }
    methods {
        _Rectangle_GetArea_<C>: _Rectangle_GetArea_<C>
        _Rectangle_GetPerimeter_<C>: _Rectangle_GetPerimeter_<C>
    }
    vartable {
        Width: Object@8
        Height: Object@16
    }
}

// Circle VTable definition
vtable Circle {
    size: 16 // 4 bytes (vtable index) + 4 bytes (badge) + 1 * Object (8 bytes)
    interfaces {
        IShape
    }
    methods {
        _Circle_GetArea_<C>: _Circle_GetArea_<C>
        _Circle_GetPerimeter_<C>: _Circle_GetPerimeter_<C>
    }
    vartable {
        Radius: Object@8
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
    GetField 1  // Height (right operand)
    Unwrap
    LoadLocal 0  // this pointer
    GetField 0  // Width (left operand)
    Unwrap
    FloatMultiply  // Width * Height
    CallConstructor _Float_float
    Return
}

// Rectangle interface method: GetPerimeter
function:1 _Rectangle_GetPerimeter_<C> {
    LoadLocal 0  // this pointer
    GetField 1  // Height (right operand)
    Unwrap
    LoadLocal 0  // this pointer
    GetField 0  // Width (left operand)
    Unwrap
    FloatAdd  // Width + Height
    PushFloat 2.0  // right operand
    FloatMultiply  // 2.0 * (Width + Height)
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
    LoadLocal 0  // this pointer
    GetField 0
    Unwrap  // Radius (right operand for first multiply)
    LoadStatic 0  // PI (left operand for first multiply)
    FloatMultiply  // PI * Radius, stack: [PI*Radius]
    LoadLocal 0  // this pointer
    GetField 0
    Unwrap  // Radius (right operand for second multiply)
    FloatMultiply  // (PI * Radius) * Radius, stack: [PI*Radius*Radius]
    CallConstructor _Float_float
    Return
}

// Circle interface method: GetPerimeter
function:1 _Circle_GetPerimeter_<C> {
    LoadStatic 0  // PI (right operand for first multiply)
    PushFloat 2.0  // left operand for first multiply
    FloatMultiply  // 2.0 * PI, stack: [2.0*PI]
    LoadLocal 0  // this pointer
    GetField 0
    Unwrap  // Radius (right operand for second multiply)
    FloatMultiply  // (2.0 * PI) * Radius
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
    // "Area: " + area.ToString()
    LoadLocal 1
    Call _Float_ToString_<C>  // area.ToString() (right operand)
    PushString "Area: "  // left operand
    StringConcat  // "Area: " + area.ToString(), stack: [result]
    
    // previous + ", Perimeter: "
    // Stack: [previous]
    PushString ", Perimeter: "  // right operand, stack: [", Perimeter: ", previous]
    Swap  // stack: [previous, ", Perimeter: "]
    StringConcat  // previous + ", Perimeter: ", stack: [result]
    
    // previous + perimeter.ToString()
    // Stack: [previous]
    LoadLocal 2
    Call _Float_ToString_<C>  // perimeter.ToString() (right operand), stack: [perimeter.ToString(), previous]
    Swap  // stack: [previous, perimeter.ToString()]
    StringConcat  // previous + perimeter.ToString(), stack: [result]
    Return
}

// Main function demonstrating interface-based polymorphism
function:1 _Global_Main_StringArray {
    // Create Rectangle object
    PushFloat 3.0  // height (rightmost argument)
    PushFloat 5.0  // width (leftmost argument)
    CallConstructor _Rectangle_float_float
    SetLocal 1
    // Create Circle object
    PushFloat 2.5  // radius (only argument)
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
    // Calculate b^2
    LoadLocal 1  // b (right operand)
    LoadLocal 1  // b (left operand)
    IntMultiply  // b^2, stack: [b*b]
    SetLocal 3  // save b*b
    
    // Calculate 4 * a * c = (4 * a) * c
    LoadLocal 0  // a (right operand for 4*a)
    PushInt 4  // 4 (left operand for 4*a)
    IntMultiply  // 4*a, stack: [4*a]
    LoadLocal 2  // c (right operand for (4*a)*c)
    IntMultiply  // 4*a*c, stack: [4*a*c]
    
    // Subtract: b^2 - 4*a*c
    LoadLocal 3  // b*b (left operand)
    IntSubtract  // b^2 - 4*a*c
    Return
}

// Pure function that performs bitwise operations
// Arguments: int (int), int (int)
pure(int, int) function:2 _Global_BitwiseOperations_int_int {
    // Calculate AND: first & second
    LoadLocal 1  // second (right operand)
    LoadLocal 0  // first (left operand)
    IntAnd
    SetLocal 2  // AND result
    
    // Calculate OR: first | second
    LoadLocal 1  // second (right operand)
    LoadLocal 0  // first (left operand)
    IntOr
    SetLocal 3  // OR result
    
    // Calculate XOR: first ^ second
    LoadLocal 1  // second (right operand)
    LoadLocal 0  // first (left operand)
    IntXor
    SetLocal 4  // XOR result
    
    // Return XOR result (most interesting)
    LoadLocal 4
    Return
}

// Usage of pure functions
function:1 _Global_Main_StringArray {
    PushInt 3  // c (rightmost argument)
    PushInt 5  // b (middle argument)
    PushInt 2  // a (leftmost argument)
    Call _Global_CalculateQuadratic_int_int_int
    SetLocal 1 // this line and line below can possibly be optimized out by the compiler
    LoadLocal 1
    IntToString
    PrintLine
    
    PushInt 7   // second (rightmost argument)
    PushInt 15  // first (leftmost argument)
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
