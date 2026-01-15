# Built-in Reference Types

This document describes the built-in reference types in Ovum, their methods, and the standard interfaces that all types
implement.

## String Type

`String` is a reference type (not primitive) for text data. All strings are immutable by default and implement
`IStringConvertible`, `IComparable`, and `IHashable`.

### String Methods

* `Length(): int` - Returns the number of characters in the string
* `ToString(): String` - Returns the string itself (implements `IStringConvertible`)
* `IsLess(other: Object): bool` - Lexicographic comparison (implements `IComparable`)
* `GetHash(): int` - Returns hash code for the string (implements `IHashable`)
* `+` operator - String concatenation (e.g., `"Hello" + " " + "World"`)

### String Usage

```ovum
val greeting: String = "Hello, World!"
val length: Int = greeting.Length()  // Returns 13
val combined: String = "Hello" + ", " + "World!"

// String comparison
val a: String = "apple"
val b: String = "banana"
val isLess: bool = a.IsLess(b)  // true (lexicographic order)

// String hashing
val hash: int = greeting.GetHash()
```

## Nullable Types

> **Note**: For detailed information about nullable types, see [`nullable.md`](nullable.md). This section only covers
> basic information.

Any primitive type can be made nullable by appending `?` (e.g., `Int?`, `String?`). Nullable types are passed by
reference and can hold either a value or `null`.

**Important**: You cannot directly call methods on nullable types using `.` - you must use the safe call operator `?.`.

```ovum
val nullableString: String? = "Hello"
// val length: Int = nullableString.Length()  // ERROR: Cannot call method directly on nullable
val safeLength: Int = nullableString?.Length() ?: 0  // Correct: Use safe call
```

## Array Types

Ovum provides specialized array classes for different data types. Arrays are reference types and support indexing,
iteration, and length operations. All arrays have dynamic length. All array types implement `IComparable` and
`IHashable`.

### Primitive Arrays

* `IntArray` - Array of 64-bit signed integers
* `FloatArray` - Array of 64-bit floating-point numbers
* `BoolArray` - Array of Boolean values
* `CharArray` - Array of characters
* `ByteArray` - Array of 8-bit unsigned integers
* `PointerArray` - Array of raw memory addresses (unsafe)

### Object Arrays

* `ObjectArray` - Array of any object type (implements `Object`)
* `StringArray` - Convenience array of `String` objects (used for `Main` function)

## File Type

`File` is a reference type for file operations.

### File Methods

* `Read(size: int): ByteArray` - Reads given amount bytes from the file
* `ReadLine(): String` - Reads all text from the file as UTF-8, returns `null` on error
* `Write(data: ByteArray): int` - Writes all bytes to the file, returns number of bytes written
* `WriteLine(text: String): Void` - Writes a string to file with `'\n'`
* `Open(path: String, mode: String): Void` - Opens the file at the given path with the specified mode
* `Close(): Void` - Closes the file handle
* `IsOpen(): bool` - Returns `true` if the file is currently open
* `Eof(): bool` - Returns `true` if the end of the file has been reached
* `Seek(position: int): void` - Sets the file position
* `Tell(): int` - Returns the current file position

### Array Methods

All array types support the following methods:

* `Length(): int` - Returns the number of elements in the array
* `[index]: ElementType` - Indexing operator for element access
* `[index] = value` - Assignment operator for mutable arrays
* `ToString(): String` - String representation of the array (implements `IStringConvertible`)
* `IsLess(other: Object): bool` - Lexicographic comparison of array elements (implements `IComparable`)
* `Equals(other: Object): bool` - Equality comparison of array contents (implements `IComparable`)
* `GetHash(): Int` - Hash code based on array contents (implements `IHashable`)
* `Reserve(newCapacity: int): Void` - Reserves capacity for the array
* `ShrinkToFit(): Void` - Shrinks the array capacity to fit its size
* `Clear(): Void` - Clears all elements from the array
* `Add(value: ElementType): Void` - Appends an element to the end of the array
* `InsertAt(index: int, value: ElementType): Void` - Inserts an element at the specified index
* `RemoveAt(index: int): Void` - Removes the element at the specified index


All of them can be created with a specified length:

```ovum
var intArr: IntArray = IntArray(10, 0)  // Array of 10 integers
var floatArr: FloatArray = FloatArray(5, 0.0)  // Array of 5 floats
```

### Array Usage

```ovum
// Creating and using arrays
val numbers: IntArray = IntArray(3)
numbers[0] = 10
numbers[1] = 20
numbers[2] = 30

val count: Int = numbers.Length()  // Returns 3

// Iteration
for (num in numbers) {
    sys::Print(Int(num).ToString())
}

// Array comparison
val arr1: IntArray = IntArray(2)
arr1[0] = 1
arr1[1] = 2

val arr2: IntArray = IntArray(2)
arr2[0] = 1
arr2[1] = 3

val isLess: Bool = arr1.IsLess(arr2)  // true (lexicographic comparison)

// Array hashing
val hash: Int = numbers.GetHash()

// String array (used in Main)
fun Main(args: StringArray): Int {
    for (arg in args) {
        sys::Print("Argument: " + arg)
    }
    return 0
}
```

## Built-in Interfaces

All types in Ovum implicitly implement certain standard interfaces that provide common functionality.

### Object (Root Interface)

The implicit root interface for all types. Provides:

* `destructor(): Void` - Virtual destructor called by GC during finalization

### IStringConvertible

Provides string conversion capability:

* `ToString(): String` - Converts the object to its string representation

All built-in types implement this interface:

```ovum
val num: Int = 42
val str: String = num.ToString()  // "42"

val flag: Bool = true
val flagStr: String = flag.ToString()  // "true"
```

### IComparable

Provides ordering capability for sorting and comparison:

* `IsLess(other: Object): bool` - Returns true if this object is less than the other
* `Equals(other: Object): bool` - Returns true if this object is equal to the other

Required for user-defined types used as parameters to pure functions (ensures stable ordering).

```ovum
val a: Int = 5
val b: Int = 10
val isLess: bool = a.IsLess(b)  // true
val isEqual: bool = a.Equals(b)  // false
```

### IHashable

Provides hashing capability for use in hash tables and caching:

* `GetHash(): int` - Returns a hash code for the object

```ovum
val text: String = "Hello"
val hash: Int = text.GetHash()
```

## Type Hierarchy

```
Object (implicit root)
├── IStringConvertible
├── IComparable  
└── IHashable

Built-in Types:
├── String (implements all interfaces)
├── File (implements all interfaces)
├── IntArray, FloatArray, etc. (implements all interfaces)
├── Int, Float, Bool, Char, Byte (implements all interfaces)
└── Int?, String?, File?, etc. (nullable versions, implements all interfaces)
```
