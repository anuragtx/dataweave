# 🎓 DataWeave Complete Masterclass - Part 1: Foundations & Theory
## Crystal Clear Theory with Every Concept, Workaround & Edge Case

> **Goal:** Master DataWeave fundamentals so deeply that you can solve ANY transformation problem. This guide covers EVERYTHING with no shortcuts.

---

## TABLE OF CONTENTS
1. [What is DataWeave - Deep Dive](#what-is-dataweave---deep-dive)
2. [How DataWeave Works - Under the Hood](#how-dataweave-works---under-the-hood)
3. [Data Types - Complete Reference](#data-types---complete-reference)
4. [Variables & Scope - Advanced](#variables--scope---advanced)
5. [Operators - Every Type Explained](#operators---every-type-explained)
6. [The Pipe Operator - Most Important](#the-pipe-operator---most-important)
7. [Functions - Deep Dive](#functions---deep-dive)
8. [Type Coercion & Conversion](#type-coercion--conversion)
9. [Control Flow - if, when, otherwise](#control-flow---if-when-otherwise)
10. [Common Mistakes & Workarounds](#common-mistakes--workarounds)

---

# PART 1: FOUNDATIONS

## What is DataWeave - Deep Dive

### The Problem It Solves
You have data in one format (JSON) and need it in another (XML, CSV, etc.). Manually converting is:
- ❌ Error-prone
- ❌ Time-consuming
- ❌ Hard to maintain

**DataWeave** is MuleSoft's **domain-specific language (DSL)** that transforms data declaratively.

### Key Characteristics

#### 1. **Declarative, Not Imperative**
```dataweave
// ❌ Imperative (HOW to do it) - NOT used in DataWeave
for item in list:
    print item * 2

// ✅ Declarative (WHAT you want) - DataWeave way
[1, 2, 3, 4] map $ * 2  // [2, 4, 6, 8]
```

**Why it matters:** You describe the desired output, not the steps. DataWeave optimizes execution.

#### 2. **Functional Programming Language**
```dataweave
// Pure functions (no side effects)
sum = (a, b) -> a + b  // Given same inputs, always same output
sum(5, 3)  // Always 8

// Immutability (values don't change)
var x = 10
x = x + 5  // This creates NEW value, doesn't modify existing
```

#### 3. **Lazy Evaluation**
```dataweave
// Only calculates what's needed
var expensive = (1 to 1000000) map sqrt($)  // Not evaluated yet!
expensive[0]  // Now it's calculated, but only up to what's needed
```

#### 4. **Strongly Typed**
```dataweave
// Types are checked at runtime
"hello" * 3  // Valid: "hellohellohello"
"hello" * "world"  // ERROR: Can't multiply two strings

// Type checking before execution helps catch bugs
```

### Execution Model

```
INPUT DATA
    ↓
DataWeave Script (Parse & Validate)
    ↓
Compiled to Java Bytecode (Fast!)
    ↓
Executed on JVM
    ↓
OUTPUT DATA
```

**Key Point:** DataWeave compiles to native Java, so it's FAST even for complex transformations.

---

## How DataWeave Works - Under the Hood

### DataWeave Script Structure

```dataweave
%dw 2.0                              // Version directive (always first)
input payload application/json       // Input type declaration (optional, auto-detected)
output application/json              // Output format (REQUIRED)
import java!java.util::ArrayList     // Optional: Import Java classes
var itemCount = sizeOf(payload)      // Variable declarations
var threshold = 1000
---                                  // Separator: everything above is header, below is body
// Transformation logic starts here
payload map {
    item: $.name,
    total: $.price * $.quantity,
    qualified: $.price * $.quantity > threshold
}
```

### Component Breakdown

#### **1. Version Directive** (`%dw 2.0`)
```dataweave
%dw 2.0  // Always use 2.0 (v1 is deprecated)
```
**Why:** MuleSoft may release v3 in future with different syntax. This locks your code to known version.

#### **2. Input Declaration** (Optional but Good Practice)
```dataweave
// Explicit input type
input payload application/json
input variables.config application/xml

// DataWeave auto-detects if omitted, but explicit is BETTER:
// ✅ Makes intention clear
// ✅ Helps catch data format bugs early
// ✅ Auto-transforms input to declared type
```

#### **3. Output Declaration** (REQUIRED)
```dataweave
output application/json    // Most common
output application/xml     // Must have single root
output text/plain          // Raw text
output text/csv            // Comma-separated
output application/java    // POJO objects
```

**Critical:** Without output declaration, DataWeave throws error.

#### **4. Imports** (Optional)
```dataweave
// Import Java classes for advanced operations
import java!java.util::ArrayList
import java!java.util::HashMap

// Import DataWeave modules
import * from dw::core::Strings
import { upper, lower } from dw::core::Strings
```

#### **5. Variables & Functions** (Header Section)
```dataweave
var myVar = "value"                 // Variable
fun myFunc(x, y) = x + y            // Function definition
```

#### **6. Separator** (`---`)
```dataweave
---  // Everything below this is the transformation body
```

**Important:** Three hyphens exactly. This marks where logic begins.

#### **7. Body** (Transformation Logic)
```dataweave
---
// Return value is the final transformed data
payload map {
    name: $.name,
    active: $.status == "active"
}
```

**Rule:** The LAST expression in the body is returned. Everything else must be in variables/functions above.

---

## Data Types - Complete Reference

### Primitive Types

#### **1. STRING - Text Data**

```dataweave
// Literals
"Hello World"
'Single quotes also work'
""  // Empty string

// Special characters with escape sequences
"Line 1\nLine 2"      // \n = newline
"Tab\there"           // \t = tab
"Quote: \"Hello\""    // \" = escaped quote
"Backslash: \\"       // \\ = escaped backslash
"Unicode: \u0041"     // \uXXXX = Unicode character (A)

// String interpolation (embedding values)
var name = "John"
var age = 30
"Name is $(name) and age is $(age)"  // "Name is John and age is 30"

// EDGE CASE: Multiline strings
"This is
a multiline
string"  // Works! Preserves newlines
```

**Important String Facts:**
- Immutable: "hello"[0] = "h", can't change it
- Zero-indexed: "hello"[0] = "h", [1] = "e", etc.
- Last index: "hello"[-1] = "o"
- Slicing: "hello"[1 to 3] = "ell"

#### **2. NUMBER - Integer or Decimal**

```dataweave
// Integers
42
-100
0

// Decimals
3.14
-0.5
0.0

// Scientific notation
1e10      // 10,000,000,000
1.5e-3    // 0.0015

// EDGE CASES & Workarounds
// Large numbers - use strings to avoid precision loss
"999999999999999999999" as Number  // May lose precision

// Negative zero (technically different from 0)
-0 == 0  // true, but -0 as String = "-0"

// Special values
(1/0)        // ERROR: Can't divide by zero
(0 / 0)      // ERROR: NaN not supported in DataWeave

// Number precision
0.1 + 0.2 == 0.3  // false! (0.30000000000000004)
// Workaround: round or use BigDecimal
round((0.1 + 0.2) * 100) / 100  // 0.3
```

#### **3. BOOLEAN - True or False**

```dataweave
true
false

// Falsy vs Truthy (IMPORTANT!)
// In DataWeave, ONLY false and null are falsy
if (0) "zero is truthy"           // Outputs: zero is truthy
if ("") "empty string is truthy"  // Outputs: empty string is truthy
if (false) "never"                // Doesn't output
if (null) "never"                 // Doesn't output

// This is DIFFERENT from JavaScript!
```

#### **4. NULL - No Value**

```dataweave
null  // Absence of value

// Checking for null
payload.field == null        // true if null
payload.field != null        // true if NOT null
payload.field is Null        // Alternative syntax

// EDGE CASE: null propagation
null.field              // null (doesn't error!)
null + 5                // null
null map $ * 2          // null

// Workaround to handle null
(payload.field default []) map $ * 2
```

#### **5. DATE - Temporal Data**

```dataweave
// Date only (ISO 8601)
|2024-03-31|

// Date with time UTC
|2024-03-31T14:30:00Z|

// Date with time and timezone
|2024-03-31T14:30:00-05:00|    // EST (UTC-5)
|2024-03-31T14:30:00+05:30|    // IST (UTC+5:30)

// CRITICAL: Understanding ISO 8601
|2024-03-31|           // Midnight UTC on that date
|2024-03-31T00:00:00Z| // Explicitly midnight UTC
|2024-03-31T00:00:00|  // Midnight (no timezone info)

// Date arithmetic
|2024-03-31| + |P5D|   // Add 5 days
|2024-03-31| - |P2D|   // Subtract 2 days
now + |PT2H|           // Add 2 hours

// Duration notation (ISO 8601)
|P5D|       // 5 days
|PT5H|      // 5 hours
|PT30M|     // 30 minutes
|PT30S|     // 30 seconds
|P1DT2H30M| // 1 day, 2 hours, 30 minutes
```

#### **6. REGULAR EXPRESSION - Pattern Matching**

```dataweave
// Regex pattern
/[0-9]{3}-[0-9]{3}-[0-9]{4}/

// Using regex
"123-456-7890" matches /[0-9]{3}-[0-9]{3}-[0-9]{4}/  // true

// Extract patterns
"abc123def456" match /([a-z]+)([0-9]+)/
// Returns array: [abc, 123] and [def, 456]

// Replace with regex
"hello world" replace /o/ with "0"  // "hell0 world"
"hello world" replace /o/g with "0" // "hell0 w0rld" (all occurrences)
```

### Complex Types

#### **7. ARRAY - Ordered Collection**

```dataweave
// Indexed from 0
[1, 2, 3, 4, 5]
["apple", "banana", "cherry"]
[true, false, true]

// Array of objects
[
  { "id": 1, "name": "Alice" },
  { "id": 2, "name": "Bob" }
]

// Mixed types (NOT recommended, but possible)
[1, "text", true, null, 3.14]

// Accessing elements
var arr = [10, 20, 30, 40, 50]
arr[0]      // 10 (first element)
arr[-1]     // 50 (last element)
arr[-2]     // 40 (second from last)
arr[10]     // null (out of bounds returns null, not error)

// Array slicing
arr[1 to 3]    // [20, 30, 40] (inclusive on both ends!)
arr[0 to 2]    // [10, 20, 30]
arr[2 to -1]   // [30, 40, 50]

// EDGE CASE: Slicing with out-of-bounds
arr[0 to 100]  // [10, 20, 30, 40, 50] (doesn't error)

// Empty array
[]

// Nested arrays
[[1, 2], [3, 4], [5, 6]]
```

#### **8. OBJECT - Key-Value Pairs**

```dataweave
// Object literal
{
  "name": "John",
  "age": 30,
  "active": true
}

// Keys can be strings or computed
{
  "first-name": "John",      // String key with hyphen
  ("key" ++ "2"): "value",   // Computed key
  (var1): var2               // Key from variable
}

// Accessing fields
var user = { "name": "John", "age": 30 }
user.name       // "John"
user["name"]    // "John" (alternative syntax)
user.missing    // null (missing fields don't error)

// Nested objects
{
  "user": {
    "firstName": "John",
    "lastName": "Doe",
    "contact": {
      "email": "john@example.com",
      "phone": "123-456-7890"
    }
  }
}

// Accessing nested
user.contact.email       // "john@example.com"
user["contact"]["email"] // Same thing

// EDGE CASE: Optional field access
user.contact?.email    // null if contact is null (no error)
user.missing?.field    // null (doesn't error)

// Empty object
{}

// Object with numeric keys (stored as strings)
{ "1": "a", "2": "b" }  // Keys are strings "1", "2"
```

### Type Coercion - Automatic Conversions

```dataweave
// String concatenation converts to string
"Hello " ++ 123              // "Hello 123"
"Age: " ++ (payload.age as String)

// Array/Object concatenation doesn't auto-convert
[1, 2] + [3, 4]             // [1, 2, 3, 4] (works)
{a: 1} + {b: 2}             // {a: 1, b: 2} (works)
[1, 2] + "text"             // ERROR (can't combine different types)

// Null in operations
null + 5                     // null (propagates)
"text" ++ null              // "textnull" (converts to string)
```

---

## Variables & Scope - Advanced

### Variable Declaration

```dataweave
// Basic variable
var name = "John"

// Can use expressions
var age = 30
var nextYear = age + 1

// Multiple variables
var firstName = "John"
var lastName = "Doe"
var fullName = firstName ++ " " ++ lastName

// SCOPE: Variables are scoped to where they're declared
var x = 10
if (true) {
  var x = 20  // Different x (shadows outer x)
  x           // 20
}
x             // 10 (original x unchanged)
```

### Variable Scope - Important Rules

```dataweave
// 1. Header variables are available in body
var taxRate = 0.18
---
payload map {
  price: $.price,
  tax: $.price * taxRate  // Can access header variable
}

// 2. Variables defined in map are LOCAL to that iteration
payload map {
  var itemTotal = $.price * $.qty  // Local to this iteration
  {
    item: $.name,
    total: itemTotal
  }
}

// 3. Variables in one branch don't leak to another
if (condition) {
  var x = 10
} else {
  var x = 20
}
x  // ERROR: x not defined outside if block

// 4. Function parameters shadow outer variables
var x = 10
var func = (x) -> x + 5  // Parameter x shadows outer x
func(20)                 // 25 (uses parameter x = 20, not outer)
```

### IMPORTANT: Variable Naming Conventions

```dataweave
// Use camelCase for variables
var myVariable = 10
var firstName = "John"

// Avoid reserved keywords
var class = 10      // ERROR (class is reserved)
var if = 5          // ERROR (if is reserved)

// Reserved words: class, fun, if, else, match, case, not, and, or, as, is, etc.
```

---

## Operators - Every Type Explained

### 1. ARITHMETIC OPERATORS

```dataweave
// ADDITION
5 + 3                    // 8 (numbers)
"Hello " + "World"       // ERROR! Use ++ for string concatenation
"Hello " ++ "World"      // "Hello World" (correct)
[1, 2] + [3, 4]          // [1, 2, 3, 4] (arrays)
{ a: 1 } + { b: 2 }      // { a: 1, b: 2 } (objects)

// Edge case: Adding incompatible types
5 + "text"               // ERROR
[1, 2] + "text"          // ERROR

// SUBTRACTION
10 - 3                   // 7 (numbers)
[1, 2, 3, 4] - [2, 4]   // [1, 3] (remove elements)
{ a: 1, b: 2 } - "a"    // { b: 2 } (remove key)

// MULTIPLICATION
5 * 3                    // 15 (numbers)
"a" * 3                  // "aaa" (repeat string)
[1, 2] * 2               // [1, 2, 1, 2] (repeat array)

// DIVISION
10 / 2                   // 5.0 (always returns decimal)
10 / 3                   // 3.3333... (repeating)
1 / 0                    // ERROR (division by zero)

// MODULO (Remainder)
10 % 3                   // 1
-10 % 3                  // -1 (result has same sign as dividend)
5.5 % 2                  // 1.5 (works with decimals)
```

### 2. COMPARISON OPERATORS

```dataweave
// EQUALITY
5 == 5              // true
"hello" == "hello"  // true
[1, 2] == [1, 2]    // true (structural equality)
{ a: 1 } == { a: 1 } // true
null == null        // true
5 == "5"            // ERROR (different types)

// INEQUALITY
5 != 3              // true
"hello" != "world"  // true
null != 5           // true

// GREATER THAN / LESS THAN
5 > 3               // true
5 < 10              // true
5 >= 5              // true
5 <= 5              // true

// Comparing strings (alphabetical)
"apple" < "banana"  // true
"zebra" > "apple"   // true

// Comparing arrays and objects - NOT supported for <, >, <=, >=
[1, 2] < [3, 4]     // ERROR

// Type checking
5 is Number         // true
"text" is String    // true
[] is Array         // true
{} is Object        // true
null is Null        // true
```

### 3. LOGICAL OPERATORS

```dataweave
// AND - Both must be true
true and true       // true
true and false      // false
5 > 3 and 10 < 20   // true and true = true

// OR - At least one must be true
true or false       // true
false or false      // false
5 > 10 or 10 < 20   // false or true = true

// NOT - Reverse boolean
not true            // false
not false           // true
not (5 > 10)        // true

// Short-circuit evaluation
// 'and' stops at first false
// 'or' stops at first true
payload.users != null and payload.users[0].name != null
// If users is null, second condition not evaluated
```

### 4. STRING OPERATORS

```dataweave
// Concatenation
"Hello " ++ "World"            // "Hello World"
"Name: " ++ payload.name       // "Name: John"

// Interpolation (embedding values)
var name = "John"
"Hello $(name)"                // "Hello John"
"2 + 3 = $(2 + 3)"             // "2 + 3 = 5"

// String comparison
"apple" == "apple"             // true
"Apple" == "apple"             // false (case-sensitive!)

// Pattern matching with regex
"123-456-7890" matches /[0-9]{3}-[0-9]{3}-[0-9]{4}/  // true
"hello" matches /^h/           // true (starts with h)
```

---

## The Pipe Operator - MOST IMPORTANT

The pipe operator (`|>`) is fundamental to DataWeave. It's what makes it so powerful.

### What It Does

The pipe operator takes the LEFT side result and passes it as input to the RIGHT side function.

```dataweave
// Without pipe
lower("HELLO")           // "hello"

// With pipe (same result, different style)
"HELLO" |> lower         // "hello"

// Why use pipe? It reads left-to-right like natural language
"HELLO" |> lower |> trim |> split(" ")  // Much clearer flow
```

### Pipe Chains - The Real Power

```dataweave
// Process data through multiple transformations
"  HELLO WORLD  "
  |> trim                           // "HELLO WORLD"
  |> lower                          // "hello world"
  |> splitBy(" ")                   // ["hello", "world"]
  |> map(upper($))                  // ["HELLO", "WORLD"]
  |> joinBy("-")                    // "HELLO-WORLD"

// Real-world: Processing array of data
[
  { "name": "  john smith  ", "active": true },
  { "name": "  jane doe  ", "active": false }
]
  |> filter($.active == true)       // Keep only active
  |> map(trim($.name))              // Trim names
  |> map(upper($))                  // Uppercase
```

### Pipe with $ (Current Value)

Inside a piped expression, `$` represents the value being piped.

```dataweave
payload
  |> filter($ > 5)         // $ is each element
  |> map($ * 2)            // $ is each filtered element
  |> sum                    // $ is the array being summed

// Real example:
[1, 2, 3, 4, 5]
  |> filter($ > 2)         // [3, 4, 5]
  |> map($ * 10)           // [30, 40, 50]
  |> sum                    // 120
```

### Pipe vs Without Pipe - Same Result, Different Style

```dataweave
// Style 1: Nested functions (hard to read)
sum(map(filter([1,2,3,4,5], $ > 2), $ * 10))

// Style 2: Using pipe (easy to read)
[1,2,3,4,5]
  |> filter($ > 2)
  |> map($ * 10)
  |> sum

// Both give same result: 120
```

---

## Functions - Deep Dive

### Function Definition

```dataweave
// Simple function
fun add(a, b) = a + b

// With body
fun greet(name) = "Hello, " ++ name

// Multiple statements (last is returned)
fun calculateTotal(price, qty, tax) = 
  var subtotal = price * qty
  var totalTax = subtotal * tax
  subtotal + totalTax
```

### Function Parameters & Default Values

```dataweave
// Basic parameters
fun add(x, y) = x + y

// Parameters with type hints (validation)
fun add(x: Number, y: Number): Number = x + y

// WORKAROUND: Default parameter values
fun greet(name = "Guest") = "Hello, " ++ name
greet()              // "Hello, Guest"
greet("John")        // "Hello, John"

// IMPORTANT: DataWeave doesn't support default values directly
// Workaround: Use if-else or default operator
fun greetSafe(name) = 
  if (name == null) "Hello, Guest" else "Hello, " ++ name
```

### Higher-Order Functions (Functions that take functions)

```dataweave
// Function that takes a function
fun applyTwice(func, value) = func(func(value))

// Usage
fun double(x) = x * 2
applyTwice(double, 5)  // 20 (5 -> 10 -> 20)

// With lambda/inline functions
applyTwice((x) -> x + 1, 5)  // 12 (5 -> 6 -> 7)
```

### Lambda Functions (Anonymous Functions)

```dataweave
// Syntax: (param) -> expression
(x) -> x * 2

// Multiple parameters
(x, y) -> x + y

// No parameters
() -> "constant value"

// Usage in map
[1, 2, 3, 4] map ((x) -> x * 2)  // [2, 4, 6, 8]

// Shorthand with $
[1, 2, 3, 4] map $ * 2           // [2, 4, 6, 8]
```

---

## Type Coercion & Conversion

### Explicit Type Conversion with `as`

```dataweave
// String to Number
"42" as Number              // 42
"3.14" as Number            // 3.14
"invalid" as Number         // ERROR

// Number to String
42 as String                // "42"
3.14 as String              // "3.14"

// String to Boolean
"true" as Boolean           // true
"false" as Boolean          // false
"anything" as Boolean       // ERROR (must be "true" or "false")

// String to Date
"2024-03-31" as Date {format: "yyyy-MM-dd"}
// |2024-03-31|

// Date to String
|2024-03-31| as String {format: "yyyy-MM-dd"}
// "2024-03-31"

// Object to JSON string
{ "name": "John" } as String
// "{\"name\":\"John\"}"

// JSON string to Object
'{"name": "John"}' as Object
// { "name": "John" }
```

### Type Coercion in Operations (Automatic)

```dataweave
// String concatenation auto-converts
"Age: " ++ 30              // "Age: 30"
"Total: " ++ (5 * 3)       // "Total: 15"

// Array/Object concatenation doesn't auto-convert
[1, 2] + [3, 4]            // [1, 2, 3, 4]
{a: 1} + {b: 2}            // {a: 1, b: 2}

// Arithmetic doesn't auto-convert strings
"5" + 3                    // ERROR (use as Number)
("5" as Number) + 3        // 8
```

### Type Checking

```dataweave
// Is operator
5 is Number                // true
"text" is String           // true
true is Boolean            // true
[1, 2] is Array            // true
{a: 1} is Object           // true
null is Null               // true
|2024-01-01| is Date       // true

// Conditional logic based on type
if (payload is Array)
  payload map $.name
else if (payload is Object)
  payload.name
else
  "Unknown type"
```

---

## Control Flow - if, when, otherwise

### if-else Statement

```dataweave
// Simple if
if (condition) "yes" else "no"

// Multi-branch
if (age < 13) "Child"
else if (age < 18) "Teen"
else if (age < 65) "Adult"
else "Senior"

// Nested if
if (user.active)
  if (user.admin) "Admin User" else "Regular User"
else
  "Inactive User"
```

### when-otherwise (Alternative Syntax)

```dataweave
// Equivalent to if-else
score > 90 when "A"
score > 80 when "B"
score > 70 when "C"
"F" otherwise

// This is often clearer than nested if-else
```

### Edge Cases in Conditionals

```dataweave
// Only null and false are falsy
if (0) "truthy"           // "truthy" (surprising!)
if ("") "truthy"          // "truthy"
if ([]) "truthy"          // "truthy"
if ({}) "truthy"          // "truthy"
if (null) "never"         // (nothing)
if (false) "never"        // (nothing)

// Safe field access
if (payload.user != null and payload.user.name != null)
  "Name: " ++ payload.user.name
else
  "No user data"

// Using default for safer code
payload.user.name default "Unknown"
```

### default - Provide Fallback Values

```dataweave
payload.nickname default "No nickname"

// Chain defaults
payload.nickname default payload.title default "Unknown"

// In object construction
{
  "name": payload.name,
  "email": payload.email default "not-provided@example.com",
  "phone": payload.phone default "N/A"
}
```

---

## Common Mistakes & Workarounds

### Mistake 1: Forgetting `as` for Type Conversion

```dataweave
// ❌ WRONG
var count = "42" + 8  // ERROR

// ✅ CORRECT
var count = ("42" as Number) + 8  // 50
```

### Mistake 2: String Concatenation with +

```dataweave
// ❌ WRONG
"Hello " + "World"  // ERROR

// ✅ CORRECT
"Hello " ++ "World"  // "Hello World"
```

### Mistake 3: Not Handling null

```dataweave
// ❌ WRONG
payload.user.name |> upper  // ERROR if user is null

// ✅ CORRECT - Option 1: Check for null
if (payload.user != null) 
  payload.user.name |> upper
else
  null

// ✅ CORRECT - Option 2: Use default
(payload.user.name default "") |> upper

// ✅ CORRECT - Option 3: Null-safe navigation (DataWeave 2.4+)
payload.user?.name |> upper
```

### Mistake 4: Array Indexing Out of Bounds

```dataweave
// ✅ DOESN'T ERROR (returns null instead)
[1, 2, 3][10]  // null

// ✅ But check first to be safe
if (sizeOf(arr) > 0) 
  arr[0]
else
  null
```

### Mistake 5: Missing Output Declaration

```dataweave
// ❌ WRONG - No output declaration
%dw 2.0
---
payload map $.name

// ✅ CORRECT
%dw 2.0
output application/json
---
payload map $.name
```

### Mistake 6: Misunderstanding Slicing

```dataweave
// ❌ WRONG - Thinking like array languages
arr[1...3]  // ERROR

// ✅ CORRECT - DataWeave uses "to"
arr[1 to 3]  // Elements at indices 1, 2, 3 (inclusive!)

// Remember: [1 to 3] includes BOTH endpoints
[10,20,30,40,50][1 to 3]  // [20, 30, 40]
```

### Mistake 7: Object Key with Hyphen/Spaces

```dataweave
// ❌ WRONG
payload.first-name  // ERROR (subtraction)

// ✅ CORRECT
payload."first-name"        // Use quotes
payload["first-name"]       // Or bracket notation
```

### Mistake 8: Comparing Incompatible Types

```dataweave
// ❌ WRONG
5 == "5"  // ERROR

// ✅ CORRECT
5 == ("5" as Number)  // true
(5 as String) == "5"  // true
```

---

## Summary of Key Concepts

| Concept | Key Point | Common Mistake |
|---------|-----------|----------------|
| **Variables** | Immutable, scoped | Scoping rules differ from traditional languages |
| **Pipe** | Left → Right flow | Not using pipes (hard to read) |
| **Types** | Static checking | Forgetting `as` for conversion |
| **Operators** | `++` for strings, `+` for arrays | Using `+` for string concatenation |
| **null** | Propagates, only falsy value | Not handling null values |
| **Slicing** | Inclusive on both ends `[a to b]` | Using JavaScript-style syntax |
| **Default** | Fallback when null | Not using for safety |
| **Functions** | Declarative, pure | Side effects not recommended |

---

## Next Steps

You've mastered the FOUNDATIONS. Ready for:
- **Part 2:** Advanced Topics, Patterns, Real-World Scenarios
- **Part 3:** 100+ Practice Problems with Solutions

