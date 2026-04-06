# 🔥 DataWeave Quick Reference & Common Mistakes
## All Edge Cases, Workarounds & Gotchas Explained

---

## TABLE OF CONTENTS
1. [Ultimate Cheat Sheet](#ultimate-cheat-sheet)
2. [Common Mistakes & Fixes](#common-mistakes--fixes)
3. [Edge Cases That Will Break Your Code](#edge-cases-that-will-break-your-code)
4. [Performance Optimization](#performance-optimization)
5. [Workarounds & Hacks](#workarounds--hacks)
6. [Interview Tips & Tricks](#interview-tips--tricks)

---

# ULTIMATE CHEAT SHEET

## Syntax Quick Reference

```dataweave
// HEADER
%dw 2.0
output application/json
import * from dw::core::Strings
var x = 10
fun add(a, b) = a + b
---
// BODY (transformation logic)

// VARIABLES
var x = 10                    // Immutable
x = x + 5                    // Creates new, doesn't modify
var result = (x |> func)     // With pipe

// OPERATORS
5 + 3                        // 8
"a" ++ "b"                   // "ab"
[1,2] + [3,4]                // [1,2,3,4]
{a:1} + {b:2}                // {a:1, b:2}
5 > 3                        // true
true and false               // false

// STRING OPERATIONS
"HELLO" |> lower             // "hello"
"hello" splitBy " "          // ["hello"]
["a","b"] joinBy ","         // "a,b"
"hi" ++ "there"              // "hithere"

// ARRAY OPERATIONS
[1,2,3] map $ * 2            // [2,4,6]
[1,2,3,4] filter $ > 2       // [3,4]
[1,2,3] reduce ((x,acc=0)->acc+x)  // 6
payload groupBy $.field      // {key: [...]}
[5,2,8] orderBy $            // [2,5,8]
[[1,2],[3,4]] flatten        // [1,2,3,4]
[1,1,2,3] distinctBy $       // [1,2,3]

// OBJECT OPERATIONS
{a:1,b:2} mapObject $*2      // {a:2,b:4}
{a:1,b:2} filterObject $>1   // {b:2}
{a:1,b:2} keys               // ["a","b"]
{a:1,b:2} values             // [1,2]
{a:1,b:2} hasKey "a"         // true

// CONDITIONAL
if (x > 5) "yes" else "no"
x > 5 when "yes" otherwise "no"
payload.field default "default"

// TYPE CHECKING & CONVERSION
"42" as Number               // 42
42 as String                 // "42"
"true" as Boolean            // true
"2024-01-01" as Date {format: "yyyy-MM-dd"}

// DATES
now                          // Current datetime
today                        // Current date
|2024-01-01| year            // 2024
|2024-01-01| month           // 1
|2024-01-01| dayOfWeek       // "Friday"
|2024-01-01| + |P5D|         // Add 5 days
|2024-01-01| as String {format: "dd/MM/yyyy"}

// ADVANCED
payload..field               // Deep search
$                            // Current value (in context)
$.field                      // Access field of current
(expr |> func)              // Pipe
{ (key): value }            // Dynamic key
```

---

# COMMON MISTAKES & FIXES

## Mistake 1: Using `+` for String Concatenation

**❌ WRONG:**
```dataweave
"Hello " + "World"  // ERROR
```

**✅ CORRECT:**
```dataweave
"Hello " ++ "World"  // "Hello World"

// OR with interpolation:
"Hello $(name)"
```

---

## Mistake 2: Forgetting Type Conversion

**❌ WRONG:**
```dataweave
"42" + 8  // ERROR

payload.age + 5  // ERROR if age is string
```

**✅ CORRECT:**
```dataweave
("42" as Number) + 8  // 50

(payload.age as Number) + 5
```

---

## Mistake 3: Not Handling Null Values

**❌ WRONG:**
```dataweave
payload.user.name |> upper  // ERROR if user is null
```

**✅ CORRECT - Option 1:**
```dataweave
if (payload.user != null) 
  payload.user.name |> upper
else
  null
```

**✅ CORRECT - Option 2:**
```dataweave
(payload.user.name default "") |> upper
```

**✅ CORRECT - Option 3 (DataWeave 2.4+):**
```dataweave
payload.user?.name |> upper
```

---

## Mistake 4: Array Index Out of Bounds

**❌ WRONG EXPECTATION:**
```dataweave
[1, 2, 3][10]  // Expects ERROR
```

**✅ ACTUAL BEHAVIOR:**
```dataweave
[1, 2, 3][10]  // Returns null, NOT error!

// Solution: Check first
if (sizeOf(arr) > 10) arr[10] else null
```

---

## Mistake 5: Wrong Slicing Syntax

**❌ WRONG:**
```dataweave
"hello"[0...3]  // ERROR (wrong syntax)
arr[1..3]       // ERROR
```

**✅ CORRECT:**
```dataweave
"hello"[0 to 3]  // "hell" (includes both endpoints!)
arr[1 to 3]      // Elements at indices 1, 2, 3
```

**Important:** `[a to b]` includes BOTH `a` and `b`.

---

## Mistake 6: Object Key with Special Characters

**❌ WRONG:**
```dataweave
payload.first-name     // ERROR (subtraction!)
payload["first-name"]  // Still might error
```

**✅ CORRECT:**
```dataweave
payload."first-name"   // Quoted key
payload["first-name"]  // Bracket notation
```

---

## Mistake 7: Comparing Incompatible Types

**❌ WRONG:**
```dataweave
5 == "5"        // ERROR (types don't match)
[1,2] < [3,4]   // ERROR (comparison not supported)
```

**✅ CORRECT:**
```dataweave
5 == ("5" as Number)   // true
(5 as String) == "5"   // true
```

---

## Mistake 8: Missing Output Declaration

**❌ WRONG:**
```dataweave
%dw 2.0
---
payload map $.name  // ERROR
```

**✅ CORRECT:**
```dataweave
%dw 2.0
output application/json
---
payload map $.name
```

---

## Mistake 9: Falsy Values Are Not What You Think

**❌ WRONG ASSUMPTION:**
```dataweave
if (0) "truthy"     // DataWeave: YES, outputs "truthy"!
if ("") "truthy"    // DataWeave: YES, outputs "truthy"!
if ([]) "truthy"    // DataWeave: YES, outputs "truthy"!
```

**✅ In DataWeave, Only These Are Falsy:**
```dataweave
if (null) "skip"    // Skipped
if (false) "skip"   // Skipped
```

---

## Mistake 10: Map Not Creating Copies

**❌ MISUNDERSTANDING:**
```dataweave
payload map {
  var x = $.price * 2
  // x is available here
}
// x is NOT available here (scoped to map)
```

**✅ CORRECT:**
```dataweave
var doubled = payload map $.price * 2
doubled[0]  // Now accessible
```

---

# EDGE CASES THAT WILL BREAK YOUR CODE

## Edge Case 1: Negative Indexing

```dataweave
// Last element
[1,2,3,4,5][-1]      // 5 ✅

// Second from last
[1,2,3,4,5][-2]      // 4 ✅

// Out of bounds negative
[1,2,3][- 100]       // null (doesn't error)
```

---

## Edge Case 2: Empty Operations

```dataweave
// Empty array operations
[] map $ * 2         // [] (returns empty)
[] filter $ > 5      // [] (returns empty)
[] reduce ((x,acc=0)->acc+x)  // 0 (returns initial value)
[] sizeOf            // 0

// Empty object
{} keys              // []
{} values            // []
{} filterObject $!=null  // {}
```

---

## Edge Case 3: Null Propagation

```dataweave
null + 5             // null
null map $ * 2       // null
null.field           // null (doesn't error!)
null[0]              // null
null filter $ > 5    // null
```

---

## Edge Case 4: Slicing Edge Cases

```dataweave
// Out of bounds slicing doesn't error
[1,2,3][0 to 100]    // [1,2,3] (returns what's available)

// Reverse slicing
[1,2,3,4,5][3 to 1]  // [] (empty, start > end)

// Last element slicing
"hello"[2 to -1]     // "llo"

// Single element
"hello"[2 to 2]      // "l"
```

---

## Edge Case 5: String Index Out of Bounds

```dataweave
"hello"[10]          // null (not error)
"hello"[-10]         // null (not error)
"hello"[0]           // "h"
"hello"[-1]          // "o"
```

---

## Edge Case 6: Type Coercion in Operators

```dataweave
// String concatenation converts types
"Value: " ++ 123              // "Value: 123"
"Value: " ++ null             // "Value: null"
"Value: " ++ {a:1}            // "Value: {a:1}"

// But NOT in arithmetic
"5" + 3                       // ERROR
5 + "3"                       // ERROR
```

---

## Edge Case 7: Truthiness in Filter

```dataweave
// This will return EVERYTHING, not filter
[1,2,3,4] filter true        // [1,2,3,4]

// This filters correctly
[1,2,3,4] filter $ > 2       // [3,4]

// Empty conditions return empty
[1,2,3,4] filter false       // []
```

---

## Edge Case 8: GroupBy with Null Keys

```dataweave
[
  { "dept": "IT", "name": "Alice" },
  { "dept": null, "name": "Bob" },
  { "dept": "IT", "name": "Charlie" }
] groupBy $.dept
// Result: { "IT": [...], "null": [...] }
// null becomes string "null" as key!
```

---

## Edge Case 9: Reduce with Empty Array

```dataweave
[] reduce ((x, acc = 100) -> acc + x)
// 100 (returns initial value, not 0!)

// Important: Initial value matters
[] reduce ((x, acc = {}) -> acc)
// {} (returns empty object)
```

---

## Edge Case 10: OrderBy with Null Values

```dataweave
[3, null, 1, null, 2] orderBy $
// [null, null, 1, 2, 3] (nulls come first)

// In descending
[3, null, 1, null, 2] orderBy -$
// ERROR! Can't negate null
```

---

## Edge Case 11: Map Over Null

```dataweave
null map $ * 2       // null (doesn't error)

// Not the same as empty array
[] map $ * 2         // []

// Solution: Handle both
(payload default []) map $ * 2
```

---

## Edge Case 12: Filter Returns New Array

```dataweave
var original = [1,2,3,4,5]
var filtered = original filter $ > 2
// filtered is [3,4,5]
// original is still [1,2,3,4,5] (immutable!)
```

---

## Edge Case 13: Variables in Conditions

```dataweave
var isActive = true
if (isActive) "yes" else "no"  // "yes"

// But this doesn't work as expected:
if (var x = payload.status) x else "default"
// ERROR (can't declare var in condition)

// Solution:
var x = payload.status
if (x != null) x else "default"
```

---

## Edge Case 14: Functions with Side Effects

```dataweave
// This WON'T work as expected (no side effects)
fun log(x) = (x |> write) ++ " done"

// Better approach:
var logged = payload map logEntry($)
```

---

## Edge Case 15: Accessing Keys with Numbers

```dataweave
{
  "1": "apple",
  "2": "banana"
}["1"]  // "apple" (treated as string)
```

---

# PERFORMANCE OPTIMIZATION

## Tip 1: Use Variables for Repeated Calculations

**❌ SLOW:**
```dataweave
payload map {
  subtotal: $.price * $.qty,
  tax: $.price * $.qty * 0.18,
  total: $.price * $.qty * 1.18
}
```

**✅ FAST:**
```dataweave
payload map {
  var subtotal = $.price * $.qty
  {
    subtotal: subtotal,
    tax: subtotal * 0.18,
    total: subtotal * 1.18
  }
}
```

---

## Tip 2: Filter Before Map

**❌ SLOW (processes all, then filters):**
```dataweave
payload map complexFunction($) filter $.valid
```

**✅ FAST (filters first):**
```dataweave
payload filter $.valid map complexFunction($)
```

---

## Tip 3: Use Reduce Instead of Multiple Maps

**❌ SLOWER (multiple iterations):**
```dataweave
payload
  map $.price
  map $ * quantity
  reduce ((x,acc=0) -> acc+x)
```

**✅ FASTER (single iteration):**
```dataweave
payload reduce ((item, acc=0) -> acc + (item.price * item.quantity))
```

---

## Tip 4: Lazy Evaluation

```dataweave
// DataWeave is lazy - only computes what's needed
var expensive = (1 to 1000000) map sqrt($)
expensive[0]  // Only computes what's accessed!
```

---

## Tip 5: Avoid Deep Nesting

**❌ SLOW:**
```dataweave
payload
  filter $.valid
  filter $.active
  filter $.verified
  map $.name
```

**✅ FAST:**
```dataweave
payload 
  filter ($.valid and $.active and $.verified)
  map $.name
```

---

# WORKAROUNDS & HACKS

## Workaround 1: Default Parameters (Not Directly Supported)

```dataweave
// DataWeave doesn't support default parameters
// Solution: Use if-else or default operator

fun greet(name) = 
  "Hello, " ++ (name default "Guest")

greet(null)      // "Hello, Guest"
greet("John")    // "Hello, John"
```

---

## Workaround 2: Function Overloading

```dataweave
// DataWeave doesn't support overloading
// Solution: Use conditional logic

fun add(x, y) = 
  if (x is Number and y is Number)
    x + y
  else if (x is String and y is String)
    x ++ y
  else if (x is Array and y is Array)
    x + y
  else
    null
```

---

## Workaround 3: Optional Chaining (Pre 2.4)

```dataweave
// DataWeave 2.4+ has ?. operator
payload.user?.name

// Before 2.4, use:
if (payload.user != null) 
  payload.user.name 
else 
  null
```

---

## Workaround 4: Try-Catch (Error Handling)

```dataweave
// No try-catch in DataWeave
// Solution: Validate before use

if (payload.value is Number)
  payload.value * 2
else
  null
```

---

## Workaround 5: Swap Values (No Destructuring)

```dataweave
// DataWeave doesn't have destructuring
// Traditional approach:
var temp = a
a = b
b = temp

// But since immutable, just use directly:
[b, a]  // Returns swapped
```

---

## Workaround 6: Range with Step

```dataweave
// No step parameter like range(1, 10, 2)
// Solution: Filter
range(1, 11) filter ($ % 2 == 1)  // [1, 3, 5, 7, 9]
```

---

## Workaround 7: Unique with Custom Logic

```dataweave
// distinctBy only works with simple selectors
// For complex uniqueness:

payload reduce ((item, obj = {}) ->
  var key = item.id ++ "-" ++ item.type
  obj + { (key): item }
) values  // Convert back to array
```

---

## Workaround 8: Flatten Specific Depth

```dataweave
// Flatten with depth control
fun flattenDepth(arr, depth) =
  if (depth <= 0)
    arr
  else
    flattenDepth(arr flatten, depth - 1)

flattenDepth([[[1,2]], [[3,4]]], 1)  // [[1,2], [3,4]]
```

---

## Workaround 9: Merge Multiple Objects

```dataweave
// Merge objects from array
[{a:1}, {b:2}, {c:3}] reduce ((obj, acc={}) -> acc + obj)
// Result: {a:1, b:2, c:3}
```

---

## Workaround 10: Safe Array Access

```dataweave
// Function for safe access
fun safeGet(arr, index, default = null) =
  if (arr != null and index < sizeOf(arr))
    arr[index]
  else
    default

safeGet([1,2,3], 10)  // null
```

---

# INTERVIEW TIPS & TRICKS

## Tip 1: Always Declare Output Type

```dataweave
// ✅ ALWAYS include this
output application/json
// or application/xml, text/csv, etc.
```

---

## Tip 2: Use Pipes for Readability

**Interviewer's Perspective:** "Can you read this code?"

```dataweave
// ❌ Hard to read
map(filter(map(payload, $ * 2), $ > 5), $ ++ "!")

// ✅ Crystal clear
payload
  map $ * 2
  filter $ > 5
  map $ ++ "!"
```

---

## Tip 3: Handle Edge Cases Explicitly

```dataweave
// Show you're thinking about edge cases
if (payload != null and sizeOf(payload) > 0)
  payload map $.name
else
  []  // Explicit handling
```

---

## Tip 4: Explain Your Variable Names

```dataweave
// Instead of:
var x = payload.price * 0.18
var y = x + payload.price

// Write:
var tax = payload.price * 0.18
var total = tax + payload.price
// Now it's self-documenting!
```

---

## Tip 5: Use Comments for Complex Logic

```dataweave
// Calculate tier-based discount
var tierDiscount = 
  if (payload.tier == "gold") 0.20
  else if (payload.tier == "silver") 0.10
  else 0.05
```

---

## Tip 6: Test Your Solutions Mentally

Before presenting, trace through:
- What if input is null?
- What if array is empty?
- What if string has special characters?
- What if numbers are very large?

---

## Tip 7: Show Multiple Solutions

```dataweave
// Solution 1: Direct
payload filterObject $ != null

// Solution 2: Using filter
payload filterObject ((value, key) -> value != null)

// "Both work, but Solution 1 is simpler"
```

---

## Tip 8: Performance Awareness

```dataweave
// Mention optimization if relevant:
// "I filtered first to reduce map iterations"

payload
  filter $.valid     // Reduce dataset early
  map transform($)   // Then transform
```

---

## Tip 9: Reference the Problem Requirements

```dataweave
// Show you're addressing specific requirements:
// "The problem asked for only active users over 25"
payload filter ($.active and $.age > 25)
```

---

## Tip 10: Have a Fallback Plan

If you don't know the exact syntax:

```dataweave
// "I'm not 100% sure of the exact syntax for groupBy,
// but the logic would be:
var grouped = groupByDepartment(payload)
var counted = grouped map { dept: $, count: sizeOf($) }
```

---

# QUICK LOOKUP TABLE

| Task | Syntax | Notes |
|------|--------|-------|
| Map | `arr map $ * 2` | Transform each element |
| Filter | `arr filter $ > 5` | Keep matching |
| Reduce | `arr reduce ((x,a=0)->a+x)` | Aggregate |
| Group | `arr groupBy $.field` | Creates object |
| Sort | `arr orderBy $` | Ascending |
| Sort Desc | `arr orderBy -$` | Descending |
| Flatten | `arr flatten` | Flatten one level |
| Distinct | `arr distinctBy $.id` | Remove duplicates |
| Size | `sizeOf(arr)` | Get length |
| Join | `arr joinBy ","` | String join |
| Split | `str splitBy ","` | String split |
| Upper | `str \| upper` | Uppercase |
| Lower | `str \| lower` | Lowercase |
| Trim | `str \| trim` | Remove spaces |
| Contains | `str contains "x"` | Has substring |
| Replace | `str replace "a" with "b"` | String replace |
| Round | `num round` | Round to int |
| Ceil | `num ceil` | Round up |
| Floor | `num floor` | Round down |
| Abs | `num abs` | Absolute value |
| If-Else | `if (x) a else b` | Conditional |
| Default | `val default x` | Fallback value |
| Type Cast | `str as Number` | Type conversion |
| Keys | `obj keys` | Get object keys |
| Values | `obj values` | Get object values |
| HasKey | `obj hasKey "x"` | Check key exists |

---

## Final Wisdom

> **"DataWeave mastery comes from:**
> 1. **Understanding the fundamentals deeply** (variables, operators, types)
> 2. **Practicing transformations repeatedly** (until patterns become automatic)
> 3. **Anticipating edge cases** (null, empty, boundaries)
> 4. **Writing readable code** (for yourself and others)
> 5. **Testing your solutions** (mentally and with examples)"

**YOU NOW HAVE EVERYTHING YOU NEED TO BECOME A DATAWEAVE EXPERT!** 🎓

---

## Recommended Practice Schedule

**Week 1:**
- Master Part 1 (Fundamentals)
- Solve Problems 1-20

**Week 2:**
- Master Part 2 (Advanced)
- Solve Problems 21-50

**Week 3:**
- Solve Problems 51-80
- Review edge cases

**Week 4:**
- Solve Problems 81-115
- Work on real-world scenarios
- Mock interviews

**After Week 4:**
- Re-solve everything from memory
- Create your own problems
- Ready for production code!

