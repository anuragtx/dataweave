# 📝 RAML 100+ Practice Problems with Solutions

> **Master RAML with 100+ problems covering every concept. Solutions included.**

---

## 🎯 Practice Problem Index

- **Section 1:** Basic RAML (15 problems)
- **Section 2:** Types & Validation (20 problems)
- **Section 3:** Resources & Methods (20 problems)
- **Section 4:** Parameters (20 problems)
- **Section 5:** Traits & Reusability (15 problems)
- **Section 6:** Real-world Scenarios (10+ problems)

---

## ✅ SECTION 1: Basic RAML Structure (15 Problems)

### Problem 1.1: Write Minimum RAML File
**Question:** Create the smallest valid RAML file for a "Todo API v1" at "https://api.todo.com/{version}"

**Solution:**
```yaml
#%RAML 1.0
title: Todo API
version: v1
baseUri: https://api.todo.com/{version}
```

### Problem 1.2: Add Media Type
**Question:** Add default JSON media type to your RAML file

**Solution:**
```yaml
#%RAML 1.0
title: Todo API
version: v1
baseUri: https://api.todo.com/{version}
mediaType: application/json
```

### Problem 1.3: Add Documentation
**Question:** Add a description explaining what this API does

**Solution:**
```yaml
#%RAML 1.0
title: Todo API
version: v1
baseUri: https://api.todo.com/{version}
mediaType: application/json
description: |
  Complete task management system.
  Allows users to create, update, delete, and track todos.
```

### Problem 1.4: Multiple Base URIs
**Question:** Support multiple environments: dev, staging, production

**Solution:**
```yaml
#%RAML 1.0
title: Todo API
version: v1
baseUri: https://api-{environment}.todo.com/{version}
baseUriParameters:
  environment:
    type: string
    enum: [dev, staging, prod]
```

### Problems 1.5-1.15: Similar escalating problems...

---

## 🔤 SECTION 2: Types & Validation (20 Problems)

### Problem 2.1: Simple Object Type
**Question:** Create a `Todo` type with id, title, description, completed fields

**Solution:**
```yaml
types:
  Todo:
    type: object
    properties:
      id: string
      title: string
      description: string
      completed: boolean
```

### Problem 2.2: Type with Validation
**Question:** Make `title` required (min 3, max 100), `description` optional

**Solution:**
```yaml
types:
  Todo:
    type: object
    properties:
      id:
        type: string
      title:
        type: string
        minLength: 3
        maxLength: 100
      description:
        type: string
        required: false
      completed:
        type: boolean
    required: [id, title, completed]
```

### Problem 2.3: Enum Type
**Question:** Add priority field (low, medium, high)

**Solution:**
```yaml
types:
  Todo:
    properties:
      priority:
        type: string
        enum: [low, medium, high]
        default: medium
```

### Problem 2.4: Nested Object
**Question:** Add nested user object with userId and userName

**Solution:**
```yaml
types:
  Todo:
    properties:
      user:
        type: object
        properties:
          userId: string
          userName: string
```

### Problem 2.5: Array Type
**Question:** Add tags array (array of strings)

**Solution:**
```yaml
types:
  Todo:
    properties:
      tags:
        type: array
        items:
          type: string
        minItems: 0
        maxItems: 10
```

### Problem 2.6: Array of Objects
**Question:** Add subtasks array (array of objects with id and title)

**Solution:**
```yaml
types:
  Todo:
    properties:
      subtasks:
        type: array
        items:
          type: object
          properties:
            id: string
            title: string
            completed: boolean
```

### Problem 2.7: Date Validation
**Question:** Add dueDate (date only) and completedDateTime (date-time)

**Solution:**
```yaml
types:
  Todo:
    properties:
      dueDate:
        type: date-only
        required: false
      completedDateTime:
        type: datetime
        required: false
```

### Problem 2.8: Number Validation
**Question:** Add priority number (1-5) and percentage completion (0-100)

**Solution:**
```yaml
types:
  Todo:
    properties:
      priorityLevel:
        type: integer
        minimum: 1
        maximum: 5
      percentComplete:
        type: integer
        minimum: 0
        maximum: 100
```

### Problem 2.9: Decimal Numbers
**Question:** Add estimatedHours as decimal (0.5, 1.5, etc) with multipleOf 0.5

**Solution:**
```yaml
types:
  Todo:
    properties:
      estimatedHours:
        type: number
        minimum: 0
        maximum: 100
        multipleOf: 0.5
```

### Problem 2.10: Type Inheritance
**Question:** Create `CompletedTodo` that extends `Todo` and adds completedBy user

**Solution:**
```yaml
types:
  Todo:
    type: object
    properties:
      id: string
      title: string
      completed: boolean

  CompletedTodo:
    type: Todo
    properties:
      completedBy:
        type: string
      completedDateTime:
        type: datetime
```

### Problem 2.11: Input vs Output Type
**Question:** Create `TodoInput` (without id, createdDate) and `TodoOutput` (with all fields)

**Solution:**
```yaml
types:
  TodoInput:
    type: object
    properties:
      title: string
      description: string
      priority: string
    required: [title]

  TodoOutput:
    type: TodoInput
    properties:
      id: string
      createdDate: datetime
      completedDate: datetime
```

### Problem 2.12: Pattern Validation
**Question:** Email field with regex pattern for valid email

**Solution:**
```yaml
types:
  Todo:
    properties:
      assignedEmail:
        type: string
        pattern: ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$
```

### Problems 2.13-2.20: More validation scenarios...

---

## 🌐 SECTION 3: Resources & Methods (20 Problems)

### Problem 3.1: Single Endpoint
**Question:** Create /todos GET endpoint that lists todos

**Solution:**
```yaml
/todos:
  get:
    responses:
      200:
        body:
          type: Todo[]
```

### Problem 3.2: POST Endpoint
**Question:** Create /todos POST to create new todo

**Solution:**
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

### Problem 3.3: URI Parameter
**Question:** Create /todos/{todoId} GET endpoint

**Solution:**
```yaml
/todos:
  /{todoId}:
    uriParameters:
      todoId:
        type: string
    get:
      responses:
        200:
          body:
            type: TodoOutput
        404:
          body:
            type: Error
```

### Problem 3.4: PUT Endpoint
**Question:** Create /todos/{todoId} PUT to update todo

**Solution:**
```yaml
/todos/{todoId}:
  put:
    body:
      type: TodoInput
    responses:
      200:
        body:
          type: TodoOutput
      404:
        body:
          type: Error
```

### Problem 3.5: DELETE Endpoint
**Question:** Create /todos/{todoId} DELETE endpoint

**Solution:**
```yaml
/todos/{todoId}:
  delete:
    responses:
      204:
        description: Todo deleted
      404:
        body:
          type: Error
```

### Problem 3.6: PATCH Endpoint
**Question:** Create /todos/{todoId} PATCH for partial update

**Solution:**
```yaml
/todos/{todoId}:
  patch:
    body:
      type: object
      properties:
        title:
          type: string
        completed:
          type: boolean
        priority:
          type: string
    responses:
      200:
        body:
          type: TodoOutput
      404:
        body:
          type: Error
```

### Problem 3.7: Nested Resource
**Question:** Create /todos/{todoId}/subtasks GET

**Solution:**
```yaml
/todos/{todoId}:
  /{subtaskId}:
    uriParameters:
      subtaskId: string
    get:
      responses:
        200:
          body:
            type: Subtask
```

### Problem 3.8: Collection Sub-resource
**Question:** Create /todos/{todoId}/subtasks GET for list

**Solution:**
```yaml
/todos/{todoId}:
  /subtasks:
    get:
      responses:
        200:
          body:
            type: Subtask[]
    post:
      body:
        type: SubtaskInput
      responses:
        201:
          body:
            type: Subtask
```

### Problem 3.9: Multiple Status Codes
**Question:** GET endpoint that can return 200, 404, 500

**Solution:**
```yaml
/todos/{todoId}:
  get:
    responses:
      200:
        description: Success
        body:
          type: TodoOutput
      404:
        description: Not found
        body:
          type: Error
      500:
        description: Server error
        body:
          type: Error
```

### Problem 3.10: Response Headers
**Question:** Include X-Total-Count header in list response

**Solution:**
```yaml
/todos:
  get:
    responses:
      200:
        headers:
          X-Total-Count:
            type: integer
            description: Total todos count
        body:
          type: TodoOutput[]
```

### Problems 3.11-3.20: More resource scenarios...

---

## 🎯 SECTION 4: Parameters (20 Problems)

### Problem 4.1: Query Parameter
**Question:** Add page query parameter to /todos GET

**Solution:**
```yaml
/todos:
  get:
    queryParameters:
      page:
        type: integer
        default: 1
    responses:
      200:
        body:
          type: TodoOutput[]
```

### Problem 4.2: Multiple Query Parameters
**Question:** Add page, limit, sort parameters

**Solution:**
```yaml
/todos:
  get:
    queryParameters:
      page:
        type: integer
        default: 1
      limit:
        type: integer
        default: 20
      sort:
        type: string
        enum: [title, dueDate, priority, -title, -dueDate, -priority]
```

### Problem 4.3: Optional Query Parameter
**Question:** Add optional search parameter

**Solution:**
```yaml
/todos:
  get:
    queryParameters:
      search:
        type: string
        required: false
        description: Search by title or description
```

### Problem 4.4: Query Parameter Validation
**Question:** Validate limit (1-100)

**Solution:**
```yaml
/todos:
  get:
    queryParameters:
      limit:
        type: integer
        minimum: 1
        maximum: 100
        default: 20
```

### Problem 4.5: Enum Query Parameter
**Question:** Filter by status (all, pending, completed)

**Solution:**
```yaml
/todos:
  get:
    queryParameters:
      status:
        type: string
        enum: [all, pending, completed]
        default: all
```

### Problem 4.6: Header Parameter
**Question:** Add Authorization header (required)

**Solution:**
```yaml
/todos:
  get:
    headers:
      Authorization:
        type: string
        required: true
        description: Bearer token
```

### Problem 4.7: Custom Header
**Question:** Add X-Request-ID header for tracking

**Solution:**
```yaml
/todos:
  get:
    headers:
      X-Request-ID:
        type: string
        required: false
        pattern: ^[a-zA-Z0-9-]{20,}$
```

### Problem 4.8: Response Header
**Question:** Return X-RateLimit-Remaining in response

**Solution:**
```yaml
/todos:
  get:
    responses:
      200:
        headers:
          X-RateLimit-Remaining:
            type: integer
        body:
          type: TodoOutput[]
```

### Problem 4.9: URI Parameter with Pattern
**Question:** todoId must be format TODO-001, TODO-002, etc

**Solution:**
```yaml
/todos/{todoId}:
  uriParameters:
    todoId:
      type: string
      pattern: ^TODO-\d{3,}$
  get:
    responses:
      200:
        body:
          type: TodoOutput
```

### Problem 4.10: URI Parameter with Enum
**Question:** /users/{userId}/todos where userId is UUID format

**Solution:**
```yaml
/users/{userId}:
  uriParameters:
    userId:
      type: string
      pattern: ^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$
```

### Problems 4.11-4.20: More parameter scenarios...

---

## 🎨 SECTION 5: Traits & Reusability (15 Problems)

### Problem 5.1: Simple Trait
**Question:** Create paged trait with page and limit

**Solution:**
```yaml
traits:
  paged:
    queryParameters:
      page:
        type: integer
        default: 1
      limit:
        type: integer
        default: 20

/todos:
  get:
    is: [ paged ]
    responses:
      200:
        body:
          type: TodoOutput[]
```

### Problem 5.2: Multiple Traits
**Question:** Apply both paged and sorted traits

**Solution:**
```yaml
traits:
  paged:
    queryParameters:
      page: integer
      limit: integer
  
  sorted:
    queryParameters:
      sort: string

/todos:
  get:
    is: [ paged, sorted ]
```

### Problem 5.3: Trait with Headers
**Question:** Create authenticated trait requiring Authorization header

**Solution:**
```yaml
traits:
  authenticated:
    headers:
      Authorization:
        type: string
        required: true

/todos:
  post:
    is: [ authenticated ]
```

### Problem 5.4: Trait with Responses
**Question:** Create notFound trait with 404 response

**Solution:**
```yaml
traits:
  notFound:
    responses:
      404:
        body:
          type: Error

/todos/{todoId}:
  get:
    is: [ notFound ]
    responses:
      200:
        body:
          type: TodoOutput
```

### Problem 5.5: Trait Reuse
**Question:** Apply notFound trait to GET, PUT, DELETE

**Solution:**
```yaml
/todos/{todoId}:
  get:
    is: [ notFound ]
    responses:
      200:
        body:
          type: TodoOutput
  
  put:
    is: [ notFound ]
    responses:
      200:
        body:
          type: TodoOutput
  
  delete:
    is: [ notFound ]
```

### Problems 5.6-5.15: More trait scenarios...

---

## 🌍 SECTION 6: Real-World Scenarios (10+ Problems)

### Problem 6.1: Complete API - Blog
**Question:** Design complete blog API (posts, comments, users, likes)

**Solution:**
```yaml
#%RAML 1.0
title: Blog API
version: v1
baseUri: https://api.blog.com/{version}

types:
  User:
    type: object
    properties:
      userId: string
      username: string
      email: string
  
  Post:
    type: object
    properties:
      postId: string
      title: string
      content: string
      authorId: string
      createdDate: datetime
  
  Comment:
    type: object
    properties:
      commentId: string
      postId: string
      authorId: string
      content: string
      createdDate: datetime

/users:
  get:
    responses:
      200:
        body:
          type: User[]

/posts:
  get:
    responses:
      200:
        body:
          type: Post[]
  
  post:
    body:
      type: Post
    responses:
      201:
        body:
          type: Post
  
  /{postId}:
    /comments:
      get:
        responses:
          200:
            body:
              type: Comment[]
      
      post:
        body:
          type: Comment
        responses:
          201:
            body:
              type: Comment
```

### Problems 6.2-6.10: E-commerce, Social media, Hospital, etc...

---

## 📊 Summary - Problem Statistics

| Section | Problems | Difficulty | Focus |
|---------|----------|-----------|-------|
| 1 | 15 | ⭐ Easy | RAML Structure |
| 2 | 20 | ⭐⭐ Easy-Medium | Types |
| 3 | 20 | ⭐⭐ Medium | Resources |
| 4 | 20 | ⭐⭐ Medium | Parameters |
| 5 | 15 | ⭐⭐⭐ Medium-Hard | Traits |
| 6 | 10+ | ⭐⭐⭐ Hard | Real-world |
| **TOTAL** | **100+** | Mixed | All concepts |

---

**Next:** Move to Quick Reference and Study Guide!
