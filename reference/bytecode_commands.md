# Ovum Intermediate Language (Bytecode) Commands

This document describes the commands available in the Ovum Intermediate Language (OIL).
All commands expect arguments of proper size on the stack and push return value on stack if applicable.

## Stack Operations

* `PushInt <value>` - Pushes a 64-bit integer literal onto the stack
* `PushFloat <value>` - Pushes a 64-bit floating-point literal onto the stack
* `PushBool <value>` - Pushes a Boolean literal (`true` or `false`) onto the stack
* `PushChar <value>` - Pushes a character literal onto the stack
* `PushString <value>` - Pushes a string literal onto the stack
* `PushNull` - Pushes `null` onto the stack
* `Pop` - Removes the top value from the stack
* `Dup` - Duplicates the top value on the stack
* `Swap` - Swaps the top two values on the stack
* `Rotate <n>` - Rotates the top `n` values on the stack

## Local Variable Operations

* `LoadLocal <index>` - Loads a local variable onto the stack
* `SetLocal <index>` - Stores the top stack value to a local variable
* `LoadGlobal <name>` - Loads a global variable onto the stack
* `SetGlobal <name>` - Stores the top stack value to a global variable

## Array Operations

* `NewArray <type> <size>` - Creates a new array of specified type and size
* `ArrayLength` - Pushes the length of the array (top of stack)
* `ArrayGet` - Gets element at index (array, index on stack)
* `ArraySet` - Sets element at index (array, index, value on stack)

## Arithmetic Operations

## Integer Operations

* `IntAdd` - Adds two integers (a + b)
* `IntSubtract` - Subtracts two integers (a - b)
* `IntMultiply` - Multiplies two integers (a * b)
* `IntDivide` - Divides two integers (a / b)
* `IntModulo` - Computes remainder (a % b)
* `IntNegate` - Negates an integer (-a)
* `IntIncrement` - Increments an integer (a + 1)
* `IntDecrement` - Decrements an integer (a - 1)

## Floating-Point Operations

* `FloatAdd` - Adds two floats (a + b)
* `FloatSubtract` - Subtracts two floats (a - b)
* `FloatMultiply` - Multiplies two floats (a * b)
* `FloatDivide` - Divides two floats (a / b)
* `FloatNegate` - Negates a float (-a)
* `FloatSqrt` - Computes square root (sqrt(a)). NaN if negative.

## Byte Operations

* `ByteAdd` - Adds two bytes (a + b)
* `ByteSubtract` - Subtracts two bytes (a - b)
* `ByteMultiply` - Multiplies two bytes (a * b)
* `ByteDivide` - Divides two bytes (a / b)
* `ByteModulo` - Computes remainder (a % b)
* `ByteNegate` - Negates a byte (-a)
* `ByteIncrement` - Increments a byte (a + 1)
* `ByteDecrement` - Decrements a byte (a - 1)

## Comparison Operations

* `IntEqual` - Tests integer equality (a == b)
* `IntNotEqual` - Tests integer inequality (a != b)
* `IntLessThan` - Tests integer less than (a < b)
* `IntLessEqual` - Tests integer less than or equal (a <= b)
* `IntGreaterThan` - Tests integer greater than (a > b)
* `IntGreaterEqual` - Tests integer greater than or equal (a >= b)
* `FloatEqual` - Tests float equality (a == b)
* `FloatNotEqual` - Tests float inequality (a != b)
* `FloatLessThan` - Tests float less than (a < b)
* `FloatLessEqual` - Tests float less than or equal (a <= b)
* `FloatGreaterThan` - Tests float greater than (a > b)
* `FloatGreaterEqual` - Tests float greater than or equal (a >= b)
* `ByteEqual` - Tests byte equality (a == b)
* `ByteNotEqual` - Tests byte inequality (a != b)
* `ByteLessThan` - Tests byte less than (a < b)
* `ByteLessEqual` - Tests byte less than or equal (a <= b)
* `ByteGreaterThan` - Tests byte greater than (a > b)
* `ByteGreaterEqual` - Tests byte greater than or equal (a >= b)

## Logical Operations

* `BoolAnd` - Logical AND (a && b)
* `BoolOr` - Logical OR (a || b)
* `BoolNot` - Logical NOT (!a)
* `BoolXor` - Logical XOR (a ^ b)

## Bitwise Operations

* `IntAnd` - Bitwise AND (a & b)
* `IntOr` - Bitwise OR (a | b)
* `IntXor` - Bitwise XOR (a ^ b)
* `IntNot` - Bitwise NOT (~a)
* `IntLeftShift` - Left bit shift (a << b)
* `IntRightShift` - Right bit shift (a >> b)

## Byte Bitwise Operations

* `ByteAnd` - Bitwise AND (a & b)
* `ByteOr` - Bitwise OR (a | b)
* `ByteXor` - Bitwise XOR (a ^ b)
* `ByteNot` - Bitwise NOT (~a)
* `ByteLeftShift` - Left bit shift (a << b)
* `ByteRightShift` - Right bit shift (a >> b)

## String Operations

* `StringConcat` - Concatenates two strings (a + b)
* `StringLength` - Gets string length
* `StringSubstring` - Extracts substring (string, start, length on stack)
* `StringCompare` - Compares two strings lexicographically
* `StringToInt` - Converts string to integer
* `StringToFloat` - Converts string to float
* `IntToString` - Converts integer to string
* `FloatToString` - Converts float to string

## Control Flow

* `Call <function>` - Calls a function
* `CallIndirect` - Calls function at address on stack
* `CallVirtual <methodName>` - Calls virtual method using vtable dispatch by method name.
* `Return` - Returns from current function

## Object Operations

* `GetField <name>` - Gets object field (object on stack)
* `SetField <name>` - Sets object field (object, value on stack)
* `CallConstructor <class>` - Calls object constructor (arguments on stack)
* `Unwrap <class>` - Unwraps value from primitive to fundamental
* `GetVTable <class>` - Gets vtable for specified class
* `SetVTable <class>` - Sets vtable for object instance
* `IsNull` - Tests if top of stack is null
* `IsNotNull` - Tests if top of stack is not null

## Nullable Operations

* `SafeCall <method>` - Safe method call on nullable object
* `NullCoalesce` - Null coalescing operator (a ?? b)
* `Unwrap` - Unwraps nullable value

## System Library Commands

## Basic I/O

* `Print` - Prints string to standard output
* `PrintLine` - Prints string followed by newline
* `ReadLine` - Reads line from standard input
* `ReadChar` - Reads single character from standard input

## Time Operations

* `UnixTime` - Gets current Unix timestamp
* `UnixTimeMs` - Gets current Unix timestamp in milliseconds
* `UnixTimeNs` - Gets current Unix timestamp in nanoseconds
* `NanoTime` - Gets high-resolution monotonic time
* `FormatDateTime` - Formats timestamp using format string
* `ParseDateTime` - Parses date string to timestamp

## File Operations

* `OpenFile` - Opens file with specified mode
* `CloseFile` - Closes file
* `FileExists` - Checks if file exists
* `DirectoryExists` - Checks if directory exists
* `CreateDirectory` - Creates directory
* `DeleteFile` - Deletes file
* `DeleteDirectory` - Deletes directory
* `MoveFile` - Moves/renames file
* `CopyFile` - Copies file
* `ListDirectory` - Lists directory contents
* `GetCurrentDirectory` - Gets current working directory
* `ChangeDirectory` - Changes current directory

## Process Control

* `SleepMs` - Sleeps for specified milliseconds
* `SleepNs` - Sleeps for specified nanoseconds
* `Exit` - Terminates process with exit code
* `GetProcessId` - Gets current process ID
* `GetEnvironmentVariable` - Gets environment variable
* `SetEnvironmentVariable` - Sets environment variable

## Random Number Generation

* `Random` - Generates random 64-bit integer
* `RandomRange` - Generates random integer in range
* `RandomFloat` - Generates random float [0.0, 1.0)
* `RandomFloatRange` - Generates random float in range
* `SeedRandom` - Seeds random number generator

## Memory and Performance

* `GetMemoryUsage` - Gets current memory usage
* `GetPeakMemoryUsage` - Gets peak memory usage
* `ForceGarbageCollection` - Forces garbage collection
* `GetProcessorCount` - Gets number of CPU cores

## System Information

* `GetOsName` - Gets operating system name
* `GetOsVersion` - Gets operating system version
* `GetArchitecture` - Gets CPU architecture
* `GetUserName` - Gets current username
* `GetHomeDirectory` - Gets user's home directory

## Error Handling

* `GetLastError` - Gets last system error description
* `ClearError` - Clears last error state

## Foreign Function Interface

* `Interope` - Calls external function via FFI (unsafe)

## Type Operations

* `TypeOf` - Gets type of value on stack
* `IsType <type>` - Tests if value is of specific type
* `SizeOf <type>` - Gets size of type in bytes
