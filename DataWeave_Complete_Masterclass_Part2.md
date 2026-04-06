# 🎓 DataWeave Complete Masterclass - Part 2: Advanced Concepts & Patterns
## Complex Transformations, Real-World Scenarios & Workarounds

---

## TABLE OF CONTENTS
1. [Array Operations - Complete Reference](#array-operations---complete-reference)
2. [Object Operations - Complete Reference](#object-operations---complete-reference)
3. [String Operations - Complete Reference](#string-operations---complete-reference)
4. [Numeric Operations - Complete Reference](#numeric-operations---complete-reference)
5. [Date & Time Operations - Complete Reference](#date--time-operations---complete-reference)
6. [Type Conversions - Complete Reference](#type-conversions---complete-reference)
7. [XML Transformations](#xml-transformations)
8. [CSV Operations](#csv-operations)
9. [Advanced Patterns](#advanced-patterns)
10. [Performance & Best Practices](#performance--best-practices)

---

# ARRAY OPERATIONS - COMPLETE REFERENCE

## `map(array, expression)` - Transform Each Element

### How It Works
```dataweave
// For each element in array, apply expression
[1, 2, 3] map $ * 2
// Step 1: First element: 1 * 2 = 2
// Step 2: Second element: 2 * 2 = 4
// Step 3: Third element: 3 * 2 = 6
// Result: [2, 4, 6]
```

### All Variations

```dataweave
// 1. Simple transformation
[1, 2, 3, 4] map $ * 2         // [2, 4, 6, 8]

// 2. Extract field from objects
[
  { "id": 1, "name": "Alice" },
  { "id": 2, "name": "Bob" }
] map $.name                     // ["Alice", "Bob"]

// 3. Create new structure
[
  { "id": 1, "price": 100, "qty": 2 },
  { "id": 2, "price": 50, "qty": 3 }
] map {
  productId: $.id,
  total: $.price * $.qty
}
// Result:
// [
//   { "productId": 1, "total": 200 },
//   { "productId": 2, "total": 150 }
// ]

// 4. Conditional transformation
payload.users map {
  name: $.name,
  status: if ($.active) "Active" else "Inactive"
}

// 5. Nested map (map within map)
[
  { "orderId": "O1", "items": [10, 20, 30] },
  { "orderId": "O2", "items": [5, 15] }
] map {
  orderId: $.orderId,
  itemCount: sizeOf($.items),
  total: $.items map sum
}

// 6. With lambda functions
[1, 2, 3, 4] map ((x) -> x * x)  // [1, 4, 9, 16]
```

### Edge Cases & Workarounds

```dataweave
// Edge case 1: Mapping over null
null map $ * 2                   // null (doesn't error)

// Workaround: Check first
if (payload != null) 
  payload map $ * 2
else
  []

// Edge case 2: Mapping preserves null elements
[1, null, 3] map $ * 2          // [2, null, 6]

// Edge case 3: Empty array
[] map $ * 2                     // [] (returns empty)

// Workaround for falsy values in map
["hello", "", "world"] map {
  value: $,
  isEmpty: $ == ""
}
```

---

## `filter(array, condition)` - Keep Only Matching Elements

### How It Works
```dataweave
// Keep only elements where condition is true
[1, 2, 3, 4, 5] filter $ > 2
// Checks each: 1 > 2? No. 2 > 2? No. 3 > 2? Yes -> keep. etc.
// Result: [3, 4, 5]
```

### All Variations

```dataweave
// 1. Simple numeric filter
[1, 2, 3, 4, 5] filter $ > 3       // [4, 5]
[1, 2, 3, 4, 5] filter $ % 2 == 0  // [2, 4] (even numbers)

// 2. Filter objects by field
[
  { "id": 1, "status": "active", "age": 25 },
  { "id": 2, "status": "inactive", "age": 17 },
  { "id": 3, "status": "active", "age": 30 }
] filter $.status == "active"
// Result: [{ "id": 1, ... }, { "id": 3, ... }]

// 3. Multiple conditions with and/or
payload.users filter ($.age >= 18 and $.status == "active")

payload.users filter ($.role == "admin" or $.role == "manager")

// 4. Complex conditions
[
  { "name": "Alice", "score": 85, "active": true },
  { "name": "Bob", "score": 45, "active": false },
  { "name": "Charlie", "score": 92, "active": true }
] filter ($.score > 80 and $.active)
// Result: Alice and Charlie

// 5. Filter with not
payload.items filter not ($.archived)  // Keep non-archived

// 6. Chained filters
[1, 2, 3, 4, 5, 6, 7, 8, 9]
  filter $ > 3           // [4, 5, 6, 7, 8, 9]
  filter $ < 8           // [4, 5, 6, 7]
  filter $ % 2 == 0      // [4, 6]
```

### Edge Cases & Workarounds

```dataweave
// Edge case 1: Filter with null
[1, null, 3, null, 5] filter $ != null  // [1, 3, 5]

// Edge case 2: Filter empty array
[] filter $ > 5                         // []

// Edge case 3: Filter all out
[1, 2, 3] filter $ > 10                 // []

// Edge case 4: Filter objects with missing fields
[
  { "id": 1, "category": "A" },
  { "id": 2 },
  { "id": 3, "category": "A" }
] filter $.category == "A"              // Only id 1 and 3

// Workaround: Check for field existence
filter ($.category != null and $.category == "A")
```

---

## `reduce(array, initialValue, accumulator)` - Aggregate Values

### How It Works
```dataweave
// Combine all elements into single value
[1, 2, 3, 4] reduce ((item, acc = 0) -> acc + item)
// acc starts at 0
// Iteration 1: item=1, acc=0 -> 0+1=1
// Iteration 2: item=2, acc=1 -> 1+2=3
// Iteration 3: item=3, acc=3 -> 3+3=6
// Iteration 4: item=4, acc=6 -> 6+4=10
// Result: 10
```

### All Variations

```dataweave
// 1. Sum numbers
[1, 2, 3, 4, 5] reduce ((item, acc = 0) -> acc + item)
// Result: 15

// 2. Concatenate strings
["a", "b", "c"] reduce ((item, acc = "") -> acc ++ item)
// Result: "abc"

// 3. Find max value
[3, 7, 2, 9, 1] reduce ((item, acc = 0) -> 
  if (item > acc) item else acc
)
// Result: 9

// 4. Build object from array
[
  { "key": "name", "value": "John" },
  { "key": "age", "value": 30 },
  { "key": "city", "value": "NYC" }
] reduce ((item, acc = {}) -> 
  acc ++ { (item.key): item.value }
)
// Result: { "name": "John", "age": 30, "city": "NYC" }

// 5. Sum prices with quantities
[
  { "product": "Laptop", "price": 1000, "qty": 2 },
  { "product": "Mouse", "price": 50, "qty": 5 }
] reduce ((item, acc = 0) -> 
  acc + (item.price * item.qty)
)
// Result: 2250

// 6. Group items by category and count
payload.items reduce ((item, acc = {}) ->
  var category = item.category
  var count = (acc[category] default 0) + 1
  acc ++ { (category): count }
)

// 7. Build array from multiple items
[1, 2, 3] reduce ((item, acc = []) -> 
  acc + [item * 2]
)
// Result: [2, 4, 6]
```

### Edge Cases & Workarounds

```dataweave
// Edge case 1: Reduce empty array
[] reduce ((item, acc = 0) -> acc + item)  // 0 (returns initial value)

// Edge case 2: Reduce with complex accumulator
[] reduce ((item, acc = { count: 0, sum: 0 }) -> 
  { count: acc.count + 1, sum: acc.sum + item }
)
// Result: { count: 0, sum: 0 }

// Edge case 3: Nested reduce
[
  [1, 2, 3],
  [4, 5],
  [6, 7, 8]
] reduce ((arr, acc = 0) ->
  acc + (arr reduce ((num, subAcc = 0) -> subAcc + num))
)
// Result: 36 (sum of all numbers)
```

---

## `groupBy(array, keyExpression)` - Group Elements

### How It Works
```dataweave
// Organize array elements by key
[
  { "dept": "IT", "name": "Alice" },
  { "dept": "HR", "name": "Bob" },
  { "dept": "IT", "name": "Charlie" }
] groupBy $.dept
// Result:
// {
//   "IT": [{ "dept": "IT", "name": "Alice" }, { "dept": "IT", "name": "Charlie" }],
//   "HR": [{ "dept": "HR", "name": "Bob" }]
// }
```

### All Variations

```dataweave
// 1. Group by string field
[
  { "category": "Electronics", "name": "Laptop" },
  { "category": "Clothing", "name": "Shirt" },
  { "category": "Electronics", "name": "Phone" }
] groupBy $.category
// Result: { Electronics: [...], Clothing: [...] }

// 2. Group by computed value
[
  { "age": 22 },
  { "age": 17 },
  { "age": 25 }
] groupBy (if ($.age >= 18) "Adult" else "Minor")
// Result: { "Adult": [...], "Minor": [...] }

// 3. Group by nested field
payload.employees groupBy $.department.name

// 4. Count items per group
payload.products groupBy $.category map ($.category => sizeOf($))
// Result: { "Electronics": 3, "Clothing": 2 }

// 5. Get group sizes
payload.items groupBy $.status map ($.status => sizeOf($))
```

---

## `orderBy(array, keyExpression)` - Sort Array

### How It Works
```dataweave
// Sort in ascending order (default)
[3, 1, 4, 1, 5] orderBy $  // [1, 1, 3, 4, 5]

// Sort descending (negate with -)
[3, 1, 4, 1, 5] orderBy -$  // [5, 4, 3, 1, 1]
```

### All Variations

```dataweave
// 1. Sort numbers ascending
[5, 2, 8, 1, 9] orderBy $             // [1, 2, 5, 8, 9]

// 2. Sort numbers descending
[5, 2, 8, 1, 9] orderBy -$            // [9, 8, 5, 2, 1]

// 3. Sort strings
["zebra", "apple", "mango"] orderBy $  // ["apple", "mango", "zebra"]

// 4. Sort objects by field
[
  { "name": "Alice", "score": 85 },
  { "name": "Charlie", "score": 92 },
  { "name": "Bob", "score": 78 }
] orderBy $.score                      // Ascending by score
// Result: [Bob (78), Alice (85), Charlie (92)]

// 5. Sort objects descending
payload.students orderBy -$.score      // Highest first

// 6. Sort by multiple fields
payload.employees orderBy [$.department, $.name]  // By dept, then name

// 7. Sort with null values
[3, null, 1, null, 2] orderBy $        // [null, null, 1, 2, 3]

// 8. Sort case-insensitive
["apple", "Zebra", "BANANA"] orderBy lower($)
```

---

## `distinctBy(array, keyExpression)` - Remove Duplicates

### How It Works
```dataweave
// Keep only first occurrence of each unique value
[1, 2, 2, 3, 3, 3, 1] distinctBy $  // [1, 2, 3]

// By object field:
[
  { "id": 1, "name": "Alice" },
  { "id": 2, "name": "Bob" },
  { "id": 1, "name": "Alice" }
] distinctBy $.id                  // [{ id: 1, ... }, { id: 2, ... }]
```

### All Variations

```dataweave
// 1. Remove duplicate numbers
[1, 1, 2, 2, 3, 1] distinctBy $  // [1, 2, 3]

// 2. Remove duplicate strings
["apple", "apple", "banana", "apple"] distinctBy $
// ["apple", "banana"]

// 3. Distinct by object field
[
  { "email": "john@ex.com", "name": "John" },
  { "email": "jane@ex.com", "name": "Jane" },
  { "email": "john@ex.com", "name": "John Doe" }
] distinctBy $.email
// Result: [John, Jane] (duplicate email removed)

// 4. Combine groupBy with distinctBy
payload.users groupBy $.department map distinctBy($.name)

// 5. Distinct by computed value
payload.products distinctBy lower($.category)
```

---

## `flatten(array, depth)` - Flatten Nested Arrays

### How It Works
```dataweave
// Flatten one level deep (default)
[[1, 2], [3, 4], [5]] flatten  // [1, 2, 3, 4, 5]

// Flatten specific depth
[[[1, 2]], [[3, 4]]] flatten(2)  // [1, 2, 3, 4]
```

### All Variations

```dataweave
// 1. Flatten one level
[[1, 2], [3, 4]] flatten         // [1, 2, 3, 4]

// 2. Flatten with depth
[[[1]], [[2]]] flatten(2)        // [1, 2]
[[[1]], [[2]]] flatten(1)        // [[1], [2]]

// 3. Flatten mixed nested
[[1, 2], [3, [4, 5]]] flatten(1)
// Result: [1, 2, 3, [4, 5]]

// 4. Flatten completely
[[[1]], [[2, [3]]]] flatten(99)  // [1, 2, 3]

// 5. Flatten objects in arrays
[
  { "items": [1, 2] },
  { "items": [3, 4] }
] map $.items |> flatten
// Result: [1, 2, 3, 4]
```

---

## `flatMap(array, expression)` - Map then Flatten

### How It Works
```dataweave
// Apply map, then automatically flatten one level
[1, 2, 3] flatMap ((x) -> [x, x*2])
// Map creates: [[1, 2], [2, 4], [3, 6]]
// Flatten: [1, 2, 2, 4, 3, 6]
```

### All Variations

```dataweave
// 1. Expand each item into multiple items
["a", "b", "c"] flatMap ((x) -> [x, x, x])
// Result: ["a", "a", "a", "b", "b", "b", "c", "c", "c"]

// 2. Flatten nested structure (orders with items)
[
  { "orderId": "O1", "items": ["Laptop", "Mouse"] },
  { "orderId": "O2", "items": ["Monitor", "Keyboard"] }
] flatMap ((order) ->
  order.items map ((item) -> {
    orderId: order.orderId,
    item: item
  })
)
// Result: Each item as separate object with orderId

// 3. Real-world: Flatten CSV-like data
[
  { "user": "Alice", "roles": ["admin", "editor"] },
  { "user": "Bob", "roles": ["viewer"] }
] flatMap ((user) ->
  user.roles map ((role) -> {
    user: user.user,
    role: role
  })
)

// 4. Equivalent to flatten after map
[1, 2, 3] flatMap ((x) -> [x * 2])
// Same as: [1, 2, 3] map (x -> [x * 2]) |> flatten
```

---

## `sizeOf(array)` - Get Array Length

```dataweave
// Count elements
[1, 2, 3, 4, 5] sizeOf    // 5
[] sizeOf                 // 0

// Works with strings (character count)
"hello" sizeOf            // 5

// Works with objects (count keys)
{ "a": 1, "b": 2 } sizeOf  // 2

// With null
null sizeOf               // null (doesn't error)

// Get total characters in array of strings
["hello", "world"] map sizeOf  // [5, 5]
(["hello", "world"] map sizeOf) reduce $ // 10 total characters
```

---

## `isEmpty(array)` - Check if Array is Empty

```dataweave
// Check for empty
[] isEmpty               // true
[1, 2] isEmpty           // false

// With null
null isEmpty             // true

// Safe checks
if (payload.items isEmpty)
  "No items"
else
  payload.items map $.name
```

---

## `reverse(array)` - Reverse Array Order

```dataweave
// Reverse array
[1, 2, 3, 4] reverse     // [4, 3, 2, 1]

// Reverse strings
"hello" reverse          // "olleh"

// Real-world: Show newest first
payload.messages orderBy -$.timestamp reverse  // Already most recent
// Or just: orderBy $.timestamp (ascending to descending)
```

---

# OBJECT OPERATIONS - COMPLETE REFERENCE

## `mapObject(object, expression)` - Transform Each Property

### How It Works
```dataweave
// Apply function to each key-value pair
{ "a": 1, "b": 2 } mapObject ($ + 10)
// Result: { "a": 11, "b": 12 }
```

### All Variations

```dataweave
// 1. Transform all values
{ "x": 1, "y": 2, "z": 3 } mapObject $ * 2
// Result: { "x": 2, "y": 4, "z": 6 }

// 2. Transform with key and value
{ "first": "john", "last": "doe" } mapObject ((value, key) ->
  upper(value)
)
// Result: { "first": "JOHN", "last": "DOE" }

// 3. Add prefix to keys
{ "name": "John", "age": 30 } mapObject ((value, key) ->
  ("user_" ++ key): value
)
// Result: { "user_name": "John", "user_age": 30 }

// 4. Transform conditionally
{ "a": 1, "b": 2, "c": 3 } mapObject ((value, key) ->
  if (value > 1) value * 100 else value
)
// Result: { "a": 1, "b": 200, "c": 300 }

// 5. Change object structure
{ "firstName": "John", "lastName": "Doe" } mapObject ((value, key) ->
  { (lower(key)): value }
)
// Result: { "firstname": "John", "lastname": "Doe" }
```

---

## `filterObject(object, condition)` - Keep Only Matching Properties

```dataweave
// 1. Remove null values
{ "name": "John", "phone": null, "email": "j@ex.com", "age": null } 
filterObject $ != null
// Result: { "name": "John", "email": "j@ex.com" }

// 2. Keep only strings
{ "name": "John", "age": 30, "city": "NYC" } 
filterObject $ is String
// Result: { "name": "John", "city": "NYC" }

// 3. Keep only numeric values
{ "a": 1, "b": "text", "c": 3, "d": "more" } 
filterObject $ is Number
// Result: { "a": 1, "c": 3 }

// 4. Filter by key pattern
{ "public_name": "John", "private_email": "j@ex.com", "public_age": 30 } 
filterObject (startsWith($ as String, "public"))  // Wrong approach

// Better:
filterObject ((value, key) -> startsWith(key, "public"))
// Result: { "public_name": "John", "public_age": 30 }

// 5. Remove empty strings and nulls
payload filterObject ($ != null and $ != "")
```

---

## `keys(object)` - Get All Property Names

```dataweave
// Extract keys
{ "name": "John", "age": 30, "city": "NYC" } keys
// Result: ["name", "age", "city"]

// Find specific key
if (({ "name": "John" } keys) contains "name") 
  "Has name field"

// Check field existence
var hasEmail = (payload keys) contains "email"
if (hasEmail) 
  "Email provided"
else
  "No email"
```

---

## `values(object)` - Get All Property Values

```dataweave
// Extract values
{ "name": "John", "age": 30, "city": "NYC" } values
// Result: ["John", 30, "NYC"]

// Get all values from nested structure
{
  "user1": { "age": 25 },
  "user2": { "age": 30 }
} values
// Result: [{ "age": 25 }, { "age": 30 }]

// Sum all numeric values
{ "revenue": 1000, "expenses": 300, "profit": 700 } values
reduce ((item, acc = 0) -> if (item is Number) acc + item else acc)
```

---

## `pluck(array, keyFunction)` - Convert Object Entries to Array

```dataweave
// Transform each key-value pair
{ "it": 5, "hr": 3, "finance": 7 } pluck ((value, key) -> {
  department: key as String,
  count: value
})
// Result: [
//   { "department": "it", "count": 5 },
//   { "department": "hr", "count": 3 },
//   { "department": "finance", "count": 7 }
// ]

// Or simpler with values/keys:
var obj = { "it": 5, "hr": 3 }
(obj keys) map ((key) -> {
  name: key,
  value: obj[key]
})
```

---

## `hasKey(object, key)` - Check Key Existence

```dataweave
// Check if key exists
{ "name": "John", "age": 30 } hasKey "name"    // true
{ "name": "John", "age": 30 } hasKey "email"   // false

// Safe field access
if (payload hasKey "user")
  payload.user.name
else
  "No user object"
```

---

# STRING OPERATIONS - COMPLETE REFERENCE

## String Manipulation Functions

```dataweave
// CASE CONVERSION
"hello" |> upper                  // "HELLO"
"HELLO" |> lower                  // "hello"
"hello world" |> capitalize       // "Hello World"

// TRIMMING
"  hello  " |> trim               // "hello"
"  hello  " |> trimLeft           // "hello  "
"  hello  " |> trimRight          // "  hello"

// CHECKING
"hello world" |> contains("world")     // true
"hello world" |> startsWith("hello")   // true
"hello world" |> endsWith("world")     // true

// EXTRACTING
"hello"[0]                        // "h"
"hello"[1 to 3]                   // "ell"
"hello"[-1]                       // "o"

// REPLACING
"hello world" replace "world" with "DataWeave"  // "hello DataWeave"
"hello world" replace /o/ with "0"             // "hell0 w0rld"

// SPLITTING & JOINING
"apple,banana,cherry" split ","           // ["apple", "banana", "cherry"]
["a", "b", "c"] joinBy "-"                // "a-b-c"

// ENCODING
"hello" ++ "world"                // "helloworld"
```

---

# NUMERIC OPERATIONS - COMPLETE REFERENCE

## Math Functions

```dataweave
// ROUNDING
3.7 round           // 4
3.2 round           // 3
3.5 round           // 4
3.4 floor           // 3
3.9 ceil            // 4

// ABSOLUTE VALUE
-42 abs             // 42
-3.14 abs           // 3.14

// MIN/MAX
[5, 2, 8, 1] min    // 1
[5, 2, 8, 1] max    // 8

// AGGREGATION
[1, 2, 3, 4] sum    // 10
[10, 20, 30] avg    // 20

// MODULO
10 % 3              // 1
-10 % 3             // -1

// SQUARE ROOT (requires import)
import * from dw::core::Math
sqrt(16)            // 4.0
```

---

# DATE & TIME OPERATIONS - COMPLETE REFERENCE

## Date Functions

```dataweave
// CURRENT TIME
now                 // |2024-03-31T14:30:45.123Z|
today               // |2024-03-31|

// PARSING
"2024-03-31" as Date {format: "yyyy-MM-dd"}
"31/03/2024" as Date {format: "dd/MM/yyyy"}

// FORMATTING
|2024-03-31| as String {format: "yyyy-MM-dd"}
now as String {format: "dd-MMM-yyyy HH:mm:ss"}

// EXTRACTING PARTS
|2024-03-31| dayOfMonth     // 31
|2024-03-31| dayOfWeek      // "Sunday"
|2024-03-31| month          // 3
|2024-03-31| year           // 2024
|2024-03-31| quarter        // 1

// ARITHMETIC
|2024-03-31| + |P5D|        // Add 5 days
|2024-03-31| - |P2W|        // Subtract 2 weeks
now + |PT2H|                // Add 2 hours

// BOUNDARY FUNCTIONS
|2024-03-31T14:30:00| atBeginningOfDay      // |2024-03-31T00:00:00|
|2024-03-31T14:30:00| atEndOfDay            // |2024-03-31T23:59:59|
|2024-03-31| atBeginningOfMonth             // |2024-03-01|
|2024-03-31| atEndOfMonth                   // |2024-03-31T23:59:59|
```

---

# XML TRANSFORMATIONS

## XML to JSON

```dataweave
// Input XML:
// <employees>
//   <employee>
//     <id>1</id>
//     <name>John</name>
//   </employee>
// </employees>

// Solution:
payload.employees.employee map {
  id: $.id,
  name: $.name
}

// OR using selector:
payload.employees.*employee map {
  id: $.id,
  name: $.name
}
```

## JSON to XML with Attributes

```dataweave
// Input JSON:
{
  "product": {
    "id": "P001",
    "name": "Laptop",
    "price": 1000,
    "currency": "USD"
  }
}

// Solution (attributes with @):
output application/xml
---
{
  item @(id: payload.product.id): {
    name: payload.product.name,
    price @(currency: payload.product.currency): payload.product.price
  }
}
```

---

# CSV OPERATIONS

## CSV to JSON

```dataweave
// Input CSV (auto-parsed to array of objects):
// name,age,city
// John,30,NYC
// Jane,25,LA

// Solution:
output application/json
---
payload map {
  fullName: $.name,
  years: $.age as Number,
  location: $.city
}
```

## JSON to CSV

```dataweave
// Input JSON:
[
  { "product": "Laptop", "qty": 2, "price": 1000 },
  { "product": "Mouse", "qty": 5, "price": 50 }
]

// Solution:
output text/csv
---
payload map {
  Product: $.product,
  Quantity: $.qty,
  UnitPrice: $.price,
  Total: $.qty * $.price
}
```

---

# ADVANCED PATTERNS

## Pattern 1: Merge Two Arrays by Common Field

```dataweave
// Problem: Merge employees with salaries by emp ID
var employees = [
  { "id": 1, "name": "Alice" },
  { "id": 2, "name": "Bob" }
]
var salaries = [
  { "empId": 1, "salary": 75000 },
  { "empId": 2, "salary": 60000 }
]
---
employees map ((emp) -> {
  id: emp.id,
  name: emp.name,
  salary: (salaries filter ($.empId == emp.id))[0].salary
})
```

## Pattern 2: Build Hierarchical Structure

```dataweave
// Problem: Group orders with items
[
  { "orderId": "O1", "item": "Laptop", "qty": 1 },
  { "orderId": "O1", "item": "Mouse", "qty": 2 },
  { "orderId": "O2", "item": "Monitor", "qty": 1 }
]
groupBy $.orderId map ((items, orderId) -> {
  orderId: orderId as String,
  items: items map {
    name: $.item,
    quantity: $.qty
  }
})
```

## Pattern 3: Pivot Table Structure

```dataweave
// Problem: Create pivot from rows
[
  { "year": 2022, "month": "Jan", "sales": 1000 },
  { "year": 2022, "month": "Feb", "sales": 1200 },
  { "year": 2023, "month": "Jan", "sales": 1500 }
]
groupBy $.year map ((yearData, year) ->
  {
    year: year as String,
    months: (yearData groupBy $.month map ((monthData, month) -> {
      month: month as String,
      sales: monthData[0].sales
    }))
  }
)
```

---

## Pattern 4: Recursive Data Processing

```dataweave
// Problem: Process deeply nested structure
fun processNode(node) = {
  id: node.id,
  children: node.children default [] map processNode($)
}
---
processNode(payload)
```

## Pattern 5: Conditional Object Construction

```dataweave
// Problem: Build object with optional fields
payload map {
  id: $.id,
  name: $.name,
  email: $.email when $.hasEmail == true otherwise null,
  phone: $.phone default "N/A"
}
```

---

# PERFORMANCE & BEST PRACTICES

## Best Practice 1: Use Variables for Reusable Expressions

```dataweave
// ❌ Bad: Recalculating
payload map {
  subtotal: $.price * $.qty,
  tax: $.price * $.qty * 0.18,
  total: $.price * $.qty * 1.18
}

// ✅ Good: Variable for reuse
payload map {
  var subtotal = $.price * $.qty
  {
    subtotal: subtotal,
    tax: subtotal * 0.18,
    total: subtotal * 1.18
  }
}
```

## Best Practice 2: Use Pipe for Readable Chains

```dataweave
// ❌ Hard to read
sizeOf(map(filter(payload, $ > 5), $ * 2))

// ✅ Easy to read
payload
  |> filter($ > 5)
  |> map($ * 2)
  |> sizeOf
```

## Best Practice 3: Avoid Nested Maps When Possible

```dataweave
// ❌ Slow: Triple nested map
payload map ($.orders map ($.items map $.price))

// ✅ Better: Use flatMap
payload flatMap ((order) ->
  order.items map ($.price)
)
```

## Best Practice 4: Filter Early to Reduce Processing

```dataweave
// ❌ Processing all, then filtering
payload map complexFunction($) filter $.valid

// ✅ Filter first, then process
payload
  |> filter($.valid == true)
  |> map(complexFunction($))
```

## Best Practice 5: Handle Null Early

```dataweave
// ❌ Risky: Null propagation
payload.user.name |> upper

// ✅ Safe: Handle null
(payload.user.name default "") |> upper

// ✅ Safe: Check before access
if (payload.user != null) 
  payload.user.name |> upper
else
  null
```

---

## Summary

You now have complete reference for:
- ✅ All array operations with edge cases
- ✅ All object operations
- ✅ String manipulation
- ✅ Numeric operations
- ✅ Date/time handling
- ✅ XML/CSV transformations
- ✅ Advanced patterns
- ✅ Performance best practices

**Next:** Practice with 100+ problems!

