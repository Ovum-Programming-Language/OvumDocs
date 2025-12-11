# Types

Ovum has a rich type system with primitive types and user-defined types. The type system is static and does not permit implicit type coercions (an `Int` won't automatically become a `Float` without an explicit cast, for example).

## Fundamental Types

Fundamental types are passed by value and represent the basic building blocks of the language:

### Numeric Types

* **`int`** (8 bytes) - 64-bit signed integer
  * Literals: `42`, `-17`, `0x1A` (hex), `0b1010` (binary)
  
* **`float`** (8 bytes) - 64-bit floating-point number (IEEE 754 double precision)
  * Literals: `3.14`, `2.0e10`, `1.5E-3`, `.5`, `5.`
  * Special values: `Infinity`, `-Infinity`, `NaN`

* **`byte`** (1 byte) - 8-bit unsigned integer
  * Literals: `255`, `0x00`, `0b11111111`

### Character and Boolean Types

* **`char`** - single Unicode character (UTF-32)
  * Literals: `'A'`, `'中'`, `'\n'`, `'\t'`, `'\0'`

* **`bool`** - Boolean value (`true`, `false`)
  * Any value can be explicitly cast to `bool`

### Low-Level Types

* **`pointer`** - raw memory address *(only meaningful in `unsafe` code)*
  * Used for FFI and low-level memory operations

> **Fundamental Type Constraints**: Fundamental types cannot be made nullable (`int?` is invalid). They are not `Object` types and cannot be stored in `ObjectArray` or cast to `Object`. To convert nullable primitives to fundamentals, cast to primitive first: `(nullableInt as Int) as int`.

## Primitive Reference Types

Primitive reference types are built-in reference wrappers around fundamental types, passed by reference:

### Numeric Reference Types

* **`Int`** - reference wrapper for `int` values
* **`Float`** - reference wrapper for `float` values  
* **`Byte`** - reference wrapper for `byte` values

### Character and Boolean Reference Types

* **`Char`** - reference wrapper for `char` values
* **`Bool`** - reference wrapper for `bool` values

### Low-Level Reference Types

* **`Pointer`** - reference wrapper for `pointer` values *(only meaningful in `unsafe` code)*

> **Nullable Primitives**: Any primitive reference type can be made nullable by appending `?` (e.g., `Int?`, `Float?`, `Bool?`).

## Reference Types

### Built-in Reference Types

* **`String`** - immutable text data (UTF-8 encoded)
  * Literals: `"Hello"`, `"Multi-line\nstring"`, `""` (empty string)
  * Concatenation: `"Hello" + " " + "World"`

* **`File`** - file operations
  * Literals: `sys::OpenFile("data.txt", "r")`
  * Methods: `Read(): ByteArray`, `ReadLine(): String`, `Write(data: ByteArray): Bool`, `WriteLine(text: String): Bool`, `Close(): Void`, `IsOpen(): Bool`

* **`Object`** - root of all reference types
  * Implicit base class for all user-defined types
  * Contains only a virtual destructor

### Array Types

Ovum provides specialized array classes for different element types (no generics/templates):

**Primitive Arrays:**
* `IntArray` - array of `int` values
* `FloatArray` - array of `float` values
* `BoolArray` - array of `bool` values
* `CharArray` - array of `char` values
* `ByteArray` - array of `byte` values
* `PointerArray` - array of `pointer` values

**Object Arrays:**
* `ObjectArray` - array of any `Object`-derived types
* `StringArray` - convenience array of `String` (used for `Main` function arguments)

**Array Creation:**
```ovum
val numbers: IntArray = IntArray(10, -1)        // Create array of int with default value -1
val names: StringArray = StringArray(5, "Null")     // Create string array of size 5 with default value "Null"
val objects: ObjectArray = ObjectArray(3, "Test")   // Create object array of size 3 with default value String "Test"
```

## Type Aliases

Create type aliases for better code readability:

```ovum
typealias UserId = Int
typealias UserName = String

fun ProcessUser(id: UserId, name: UserName): Void {
    // Implementation
}
```


## Assignment Operators

### Reference Assignment (`=`)

The standard assignment operator assigns references for reference types:

```ovum
val original: String = "Hello"
val reference: String = original  // Both variables point to the same string
```

### Copy Assignment (`:=`)

The copy assignment operator performs deep copy for reference types:

```ovum
val original: String = "Hello"
val copy: String := original  // Creates a new string with the same content
```

## Type Casting

### Explicit Casting

Use the `as` operator for explicit casting:

```ovum
val intValue: int = 42
val floatValue: float = (intValue as float)  // int to float

val floatNum: float = 3.14
val intNum: int = (floatNum as int)  // float to int (truncates)

// Implicit bidirectional casting between fundamental and primitive reference types
val fundamentalInt: int = 42
val primitiveInt: Int = fundamentalInt  // Implicit: int -> Int
val backToFundamental: int = primitiveInt  // Implicit: Int -> int

// Implicit conversion from literals to primitive types
val count: Int = 0  // Implicit: int literal -> Int
val flag: Bool = true  // Implicit: bool literal -> Bool
val pi: Float = 3.14  // Implicit: float literal -> Float

// Arithmetic works seamlessly
val sum: Int = 10 + 20  // Int + Int = Int (implicit conversion from literals)
val result: int = sum + 5  // Int + int = int (implicit conversion)
```

### Boolean Casting

Any value can be explicitly cast to `bool`:

```ovum
val intVal: int = 42
val boolVal: bool = (intVal as bool)  // true (non-zero)

val zeroInt: int = 0
val falseBool: bool = (zeroInt as bool)  // false (zero)

val nullString: String? = null
val nullBool: bool = (nullString as bool)  // false (null)

// With primitive reference types (implicit conversion)
val primitiveInt: Int = 42  // Implicit conversion from literal
val primitiveBool: bool = primitiveInt  // Implicit: Int -> bool
val boolRef: Bool = true  // Implicit: bool literal -> Bool
```

**Rules:** Fundamentals and primitive reference types: zero → `false`, non-zero → `true`.
References: `null` → `false`, non-null → `true`

### Unsafe Casting

Some casts require `unsafe` blocks:

```ovum
unsafe {
    val obj: Object = Point(10, 20)
    val bytes: ByteArray = (obj as ByteArray)           // Raw byte view
    val mutableBytes: ByteArray = (obj as var ByteArray) // Mutable byte view
}
```

## Passing Semantics

**Fundamental types** (`int`, `float`, `byte`, `char`, `bool`, `pointer`) are passed by value (copied):
```ovum
fun ModifyInt(x: int): Void {
    x = x + 1  // Only modifies the local copy
}
```

**Primitive reference types** (`Int`, `Float`, `Byte`, `Char`, `Bool`, `Pointer`) and **all other reference types** (including `String`, arrays, and user-defined types) are passed by reference:
```ovum
fun ModifyIntRef(var x: Int): Void {
    x = x + 1  // Implicit conversion: Int + int -> Int
}

fun ModifyArray(arr: IntArray): Void {
    arr[0] := Int(999)  // Use := for deep copy assignment
}
```

**Immutability:** References are immutable by default - use `var` for mutable references:
```ovum
fun CannotReassign(str: String): Void {
    // str = "New value"  // ERROR: Cannot reassign immutable reference
}

fun CanReassign(var str: String): Void {
    str = "New value"  // OK: str is mutable
}
```

## Type System Characteristics

**Static typing:** Every variable and expression has a type checked at compile time
**Limited implicit conversions:** The compiler only performs implicit conversions between a primitive reference type and its matching fundamental (for example, `Int` ↔ `int`). Any conversion across different primitive families—such as `Int` to `Float` or `Float` to `int`—must use an explicit cast.
**Type safety:** Prevents many common errors
**Nullable types:** Any reference type (including primitive reference types) can be made nullable by appending `?`. Fundamental types cannot be nullable.

```ovum
val x: int = 42
val y: String = "Hello"
// val z: int = x + y  // ERROR: Cannot add int and String

val intVal: int = 42
val floatVal: float = 3.14
val result: float = (intVal as float) + floatVal  // OK: Explicit conversion

// Using primitive reference types (implicit conversion between wrappers and fundamentals)
val refInt: Int = 42  // Implicit conversion from literal to Int
val refFloat: Float = 3.14  // Implicit conversion from literal to Float
val sum: Int = refInt + (refFloat as Int)  // Requires explicit narrowing
val fundamentalSum: int = sum + 10  // Implicit: Int -> int when assigning to a fundamental

// Converting nullable primitives to fundamentals
val nullableInt: Int? = 42  // Implicit conversion from literal
val fundamentalFromNullable: int = (nullableInt ?: 0) as int  // Two-step conversion
```

## Pure Function Constraints

Types used as parameters in pure functions must implement `IComparable` for stable ordering:

```ovum
pure fun ProcessData(data: IComparable): Int {
    // data must implement IComparable for stable ordering
    return data.GetHash()
}
```

## Runtime Behavior

Type information is preserved at runtime for reference types:

```ovum
fun ProcessObject(obj: Object): Void {
    if (obj is String) {
        val str: String? = obj as String
        if (str != null) {
            val nonNullStr: String = str ?: "default"  // Use Elvis operator
            sys::Print("String length: " + nonNullStr.Length().ToString())
        }
    } else if (obj is IntArray) {
        val arr: IntArray? = obj as IntArray
        if (arr != null) {
            val nonNullArr: IntArray = arr ?: IntArray(0)  // Use Elvis operator
            sys::Print("Array size: " + nonNullArr.Length().ToString())
        }
    } else if (obj is Int) {
        val intRef: Int? = obj as Int
        if (intRef != null) {
            val nonNullInt: Int = intRef ?: 0  // Use Elvis operator
            sys::Print("Int value: " + nonNullInt.ToString())  // Implicit conversion to string
        }
    }
}
```
