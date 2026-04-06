# 📚 RAML Complete Reference & Quick Lookup

> **All RAML syntax in one place. Quick reference for interviews and production.**

---

## 🎯 Quick Reference by Task

### "I need to create a GET endpoint"
```yaml
/todos:
  get:
    displayName: Get Todos
    description: Retrieve all todos
    responses:
      200:
        body:
          type: Todo[]
```

### "I need to create a POST endpoint"
```yaml
/todos:
  post:
    body:
      type: TodoInput
    responses:
      201:
        body:
          type: TodoOutput
      400:
        body:
          type: Error
```

### "I need to validate a string"
```yaml
properties:
  email:
    type: string
    pattern: ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$
    minLength: 5
    maxLength: 254
```

### "I need to validate a number"
```yaml
properties:
  age:
    type: integer
    minimum: 18
    maximum: 120
  gpa:
    type: number
    minimum: 0
    maximum: 4.0
    multipleOf: 0.01
```

### "I need pagination"
```yaml
traits:
  paged:
    queryParameters:
      page:
        type: integer
        default: 1
        minimum: 1
      limit:
        type: integer
        default: 20
        minimum: 1
        maximum: 100
```

### "I need authentication"
```yaml
traits:
  authenticated:
    headers:
      Authorization:
        type: string
        required: true
        description: Bearer token
```

### "I need error handling"
```yaml
types:
  Error:
    type: object
    properties:
      error:
        type: string
        enum: [BAD_REQUEST, NOT_FOUND, UNAUTHORIZED, SERVER_ERROR]
      message: string
      timestamp: datetime
      path: string
```

---

## 📋 RAML Syntax Cheat Sheet

### File Header
```yaml
#%RAML 1.0
title: API Name
version: v1
baseUri: https://api.example.com/{version}
mediaType: application/json
description: Optional API description
```

### Types - Primitive
```yaml
types:
  StringType:
    type: string
    minLength: 1
    maxLength: 100
    pattern: ^[A-Z][a-z]+$
    enum: [value1, value2]
    default: value1

  NumberType:
    type: number
    minimum: 0
    maximum: 1000
    multipleOf: 0.1

  IntegerType:
    type: integer
    minimum: 1
    maximum: 100

  BooleanType:
    type: boolean
    default: false

  DateType:
    type: date-only

  DateTimeType:
    type: datetime

  ArrayType:
    type: array
    items:
      type: string
    minItems: 1
    maxItems: 10
    uniqueItems: true
```

### Types - Objects
```yaml
# Simple object
SimpleObject:
  type: object
  properties:
    name: string
    age: integer

# Required fields
WithRequired:
  type: object
  properties:
    name: string
    email: string
  required: [name, email]

# Nested object
NestedObject:
  type: object
  properties:
    name: string
    address:
      type: object
      properties:
        street: string
        city: string

# Object inheritance
Extended:
  type: SimpleObject
  properties:
    email: string
```

### Traits
```yaml
traits:
  paged:
    queryParameters:
      page: integer
      limit: integer

  authenticated:
    headers:
      Authorization: string

  notFound:
    responses:
      404:
        body:
          type: Error

# Apply traits
/endpoint:
  get:
    is: [paged, authenticated, notFound]
```

### Resources & Methods
```yaml
/todos:
  displayName: Todos
  description: Todo management
  
  get:
    displayName: List
    is: [paged]
    responses:
      200:
        body:
          type: Todo[]

  post:
    body:
      type: TodoInput
    responses:
      201:
        body:
          type: Todo
      400:
        body:
          type: Error

  /{todoId}:
    uriParameters:
      todoId: string
    
    get:
      responses:
        200:
          body:
            type: Todo
        404:
          body:
            type: Error

    put:
      body:
        type: TodoInput
      responses:
        200:
          body:
            type: Todo
        404:
          body:
            type: Error

    patch:
      body:
        type: object
        properties:
          title: string
          completed: boolean
      responses:
        200:
          body:
            type: Todo

    delete:
      responses:
        204:
        404:
          body:
            type: Error
```

### Query Parameters
```yaml
/todos:
  get:
    queryParameters:
      page:
        type: integer
        required: false
        default: 1
        minimum: 1
        description: Page number

      limit:
        type: integer
        required: false
        default: 20
        minimum: 1
        maximum: 100

      status:
        type: string
        required: false
        enum: [pending, completed, all]

      search:
        type: string
        required: false

      sort:
        type: string
        required: false
        enum: [name, -name, date, -date]
```

### Headers
```yaml
/todos:
  get:
    headers:
      Authorization:
        type: string
        required: true
        pattern: ^Bearer\s.+$
        description: Bearer token

      X-Request-ID:
        type: string
        required: false

      X-API-Version:
        type: string
        required: false
        default: "1.0"
```

### Responses
```yaml
/todos/{todoId}:
  get:
    responses:
      200:
        description: Success
        headers:
          X-Total-Count:
            type: integer
        body:
          type: Todo

      404:
        description: Not found
        body:
          type: Error

      500:
        description: Server error
        body:
          type: Error
```

---

## ❌ Common Mistakes & Fixes

### Mistake 1: Missing #%RAML directive
```yaml
# ❌ WRONG
title: My API

# ✅ CORRECT
#%RAML 1.0
title: My API
```

### Mistake 2: Wrong property name (require vs required)
```yaml
# ❌ WRONG
TodoInput:
  type: object
  properties:
    title: string
  require: [title]

# ✅ CORRECT
TodoInput:
  type: object
  properties:
    title: string
  required: [title]
```

### Mistake 3: Using singular instead of plural for collections
```yaml
# ❌ WRONG
/todo:
  get:
    responses:
      200:
        body:
          type: Todo

# ✅ CORRECT
/todos:
  get:
    responses:
      200:
        body:
          type: Todo[]
```

### Mistake 4: Using verbs in resource names
```yaml
# ❌ WRONG
/getTodos:
/createTodo:
/deleteTodo:

# ✅ CORRECT
/todos:
  get:
  post:
  delete:
```

### Mistake 5: Wrong status codes
```yaml
# ❌ WRONG
POST response with 200

# ✅ CORRECT
POST response with 201
DELETE response with 204 (no body) or 200 (with body)
```

### Mistake 6: Creating instance instead of type
```yaml
# ❌ WRONG
/todos:
  get:
    responses:
      200:
        body:
          application/json:
            title: string
            description: string

# ✅ CORRECT
types:
  Todo:
    type: object
    properties:
      title: string
      description: string

/todos:
  get:
    responses:
      200:
        body:
          type: Todo
```

### Mistake 7: Forgetting required marker on type
```yaml
# ❌ WRONG - Field looks optional but is required
StudentInput:
  type: object
  properties:
    name: string
    email: string

# ✅ CORRECT - Explicitly mark required
StudentInput:
  type: object
  properties:
    name: string
    email: string
  required: [name, email]
```

### Mistake 8: Using POST for retrieval
```yaml
# ❌ WRONG
/todos/search:
  post:
    queryParameters:
      search: string

# ✅ CORRECT
/todos:
  get:
    queryParameters:
      search: string
```

### Mistake 9: Inconsistent error responses
```yaml
# ❌ WRONG - Different error structures
404:
  body:
    message: string
500:
  body:
    error: string
    description: string

# ✅ CORRECT - Consistent structure
types:
  Error:
    type: object
    properties:
      error: string
      message: string
      timestamp: datetime

404:
  body:
    type: Error
500:
  body:
    type: Error
```

### Mistake 10: Missing example values
```yaml
# ❌ WRONG - No examples
/todos:
  post:
    body:
      type: TodoInput

# ✅ CORRECT - Include examples
/todos:
  post:
    body:
      type: TodoInput
      example: |
        {
          "title": "Buy groceries",
          "priority": "high"
        }
```

---

## 🎯 Edge Cases That Break Your RAML

### Edge Case 1: Circular Type References
```yaml
# ❌ PROBLEM
Student:
  properties:
    courses: Course[]

Course:
  properties:
    students: Student[]  # Circular!

# ✅ SOLUTION
Student:
  properties:
    courseIds: string[]  # Store only IDs

Course:
  properties:
    studentIds: string[]
```

### Edge Case 2: Optional Required Field
```yaml
# ❌ CONFUSING
Todo:
  type: object
  properties:
    title: string
    optional: string
  required: [title]  # optional is required? Or not?

# ✅ CLEAR
Todo:
  type: object
  properties:
    title:
      type: string
      required: true
    optional:
      type: string
      required: false
```

### Edge Case 3: Empty Response Body
```yaml
# ❌ WRONG
DELETE:
  responses:
    204:
      body:
        type: Empty  # No body in 204!

# ✅ CORRECT
DELETE:
  responses:
    204:
      description: No content
      # No body specification
```

### Edge Case 4: Array Items Without Type
```yaml
# ❌ WRONG
todos:
  type: array
  items:  # No type specified!

# ✅ CORRECT
todos:
  type: array
  items:
    type: Todo
```

### Edge Case 5: Required Array with No Items
```yaml
# ❌ CONFUSING
subtasks:
  type: array
  minItems: 0
  maxItems: 10
  required: true

# ✅ CLEAR
subtasks:
  type: array
  minItems: 1  # At least one
  maxItems: 10
  required: true
```

---

## 🚀 Performance Tips

### Tip 1: Use Traits for DRY
```yaml
# ❌ REPEATING
/students:
  get:
    headers:
      Authorization: string
/courses:
  get:
    headers:
      Authorization: string
/departments:
  get:
    headers:
      Authorization: string

# ✅ BETTER
traits:
  authenticated:
    headers:
      Authorization: string

/students:
  get:
    is: [authenticated]
/courses:
  get:
    is: [authenticated]
/departments:
  get:
    is: [authenticated]
```

### Tip 2: Use Resource Types
```yaml
# ❌ REPEATING for each resource
/students:
  get:
    is: [paged]
  post:
    responses:
      201:
/courses:
  get:
    is: [paged]
  post:
    responses:
      201:

# ✅ BETTER
resourceTypes:
  collection:
    get:
      is: [paged]
    post:
      responses:
        201:

/students:
  type: collection
/courses:
  type: collection
```

### Tip 3: External Files for Organization
```yaml
# main-api.raml
uses:
  types: types/common.raml
  traits: traits/common.raml

# types/common.raml
types:
  Error:
    type: object
    properties:
      error: string

# traits/common.raml
traits:
  paged:
    queryParameters:
      page: integer
```

---

## 🎓 Interview Quick Tips

| Question | Answer |
|----------|--------|
| What is RAML? | RESTful API Modeling Language - API specification format |
| Why use RAML? | Single source of truth, auto-docs, mock servers, testing |
| Main components? | Types, Resources, Methods, Traits, Security |
| Difference: PUT vs PATCH? | PUT = entire resource, PATCH = partial fields |
| Status code for create? | 201 Created |
| Status code for delete? | 204 No Content (empty) or 200 (with body) |
| Required vs optional field? | Use required array or required: true/false |
| How to reuse? | Traits for behaviors, Resource Types for patterns |
| Best practice naming? | Plural nouns, lowercase, /students not /getStudents |
| Error handling? | Consistent Error type across all endpoints |

---

## 📊 RAML Syntax Comparison Table

| Concept | RAML | Example |
|---------|------|---------|
| String | `type: string` | `pattern: ^[a-z]+$` |
| Number | `type: number` | `minimum: 0, maximum: 100` |
| Integer | `type: integer` | `minimum: 1, maximum: 50` |
| Boolean | `type: boolean` | `default: false` |
| Date | `type: date-only` | `2024-03-31` |
| DateTime | `type: datetime` | `2024-03-31T15:30:00Z` |
| Array | `type: array` | `items: type: string` |
| Object | `type: object` | `properties: {name: string}` |
| Required | `required: [...]` | `required: [name, email]` |
| Enum | `enum: [...]` | `enum: [active, inactive]` |
| Pattern | `pattern: '^...$'` | Regex validation |
| Min/Max | `minimum: 0, maximum: 100` | Range validation |

---

**Use this guide while developing RAML APIs and preparing for interviews!**
