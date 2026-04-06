# 🚀 RAML Complete Masterclass - Part 1: Foundations & Fundamentals

> **Crystal clear, comprehensive guide to RAML (RESTful API Modeling Language) for interviews and production**

---

## 📚 Table of Contents

1. [What is RAML? Deep Dive](#what-is-raml-deep-dive)
2. [Why RAML? Real Business Value](#why-raml-real-business-value)
3. [RAML Basics - File Structure](#raml-basics--file-structure)
4. [Data Types in RAML](#data-types-in-raml)
5. [Resources & Endpoints](#resources--endpoints)
6. [HTTP Methods (GET, POST, PUT, DELETE)](#http-methods-get-post-put-delete)
7. [Request & Response Bodies](#request--response-bodies)
8. [Query Parameters](#query-parameters)
9. [URI Parameters](#uri-parameters)
10. [Headers](#headers)
11. [Status Codes & Error Responses](#status-codes--error-responses)
12. [Type System - Everything](#type-system---everything)
13. [Traits - DRY Principle](#traits---dry-principle)
14. [Security Schemes](#security-schemes)
15. [Common Mistakes & Workarounds](#common-mistakes--workarounds)

---

## 🎯 What is RAML? Deep Dive

### **Simple Explanation (For Non-Technical People)**

**Scenario:** You're building a restaurant ordering app.
- **Without RAML:** Frontend dev says "Send me the order data" → Backend dev says "OK, I'll send name, customerID, items, total" → Frontend dev gets confused: "Is it customerID or customer_id? Is total a number or string?" → Chaos and bugs!
- **With RAML:** One document says: "Order has these exact fields: customerID (STRING, required), items (ARRAY of Objects), total (NUMBER, required)" → Both teams know EXACTLY what to send and receive → No confusion, no bugs!

### **What RAML Actually Is**

**RAML = API Contract in YAML format**

Think of it as a blueprint:
- **Blueprint for a house** = Defines rooms, doors, windows, electrical outlets
- **RAML for an API** = Defines endpoints, what data goes in, what comes out, error codes

### **Key Characteristics**

```yaml
✅ YAML Format (human-readable, not JSON)
✅ Machine-readable (tools can parse it)
✅ RESTful design focused
✅ Reusable components (DRY - Don't Repeat Yourself)
✅ Can be versioned in Git
✅ Creates interactive documentation automatically
✅ Tools can generate client code from it
✅ Tools can generate mock servers from it
✅ Can be used for testing
```

### **How RAML Fits in Your Project**

```
┌─────────────────────────────────┐
│  RAML Specification File        │
│  (API Contract Document)        │
└────────────┬────────────────────┘
             │
    ┌────────┴────────┬─────────────┬──────────────┐
    │                 │             │              │
    ▼                 ▼             ▼              ▼
Frontend Devs   Backend Devs  QA/Testers    Mock Server
Know what to    Know what     Know what     Test without
send            to receive    to test       real backend
```

### **Real-World Usage**

**Example: E-commerce API**
```yaml
# RAML defines:
# 1. GET /products → Returns list of products
# 2. POST /orders → Create new order
# 3. GET /orders/{orderId} → Get specific order
# 4. PUT /orders/{orderId} → Update order
# 5. DELETE /orders/{orderId} → Cancel order

# And for EACH endpoint:
# - What parameters it accepts
# - What data format (JSON/XML)
# - What fields are required
# - What status codes it returns
# - What errors it can throw
```

---

## 💼 Why RAML? Real Business Value

### **Problem RAML Solves**

| Problem | Without RAML | With RAML |
|---------|-------------|----------|
| Documentation | Outdated word docs | Auto-generated, always current |
| Miscommunication | "Is it ID or id?" | Single source of truth |
| Testing | Manual, slow, error-prone | Auto-generated tests |
| Time to Market | Dev + QA + Docs = weeks | RAML + Auto-tools = days |
| Integration | Guess the API format | Exact specification |
| Mocking | Build fake server | Instant mock server |
| Client Code | Manual coding | Auto-generated |

### **Business Benefits**

✅ **Faster Development** - Frontend & backend work in parallel
✅ **Fewer Bugs** - Clear contracts prevent misunderstandings
✅ **Better Documentation** - Always up-to-date
✅ **Easier Testing** - Automated testing possible
✅ **Reusability** - Share API specs across teams
✅ **Compliance** - Document API contracts for audits
✅ **Cost Savings** - Less rework, fewer meetings

---

## 📋 RAML Basics - File Structure

### **Minimum RAML File (The Skeleton)**

```yaml
#%RAML 1.0
title: Student API
version: v1
baseUri: https://api.example.com/{version}
```

**What each line means:**

| Line | Meaning | Required? |
|------|---------|-----------|
| `#%RAML 1.0` | "This is a RAML file version 1.0" | YES |
| `title:` | API name | YES |
| `version:` | API version (v1, v2, etc) | YES |
| `baseUri:` | Base URL for all endpoints | YES |

### **Complete RAML File Structure**

```yaml
#%RAML 1.0
title: Student API
version: v1
baseUri: https://api.student.com/{version}
mediaType: application/json  # Default data format

# Import external files
uses:
  resources: library.raml
  
# Define custom data types
types:
  Student:
    properties:
      studentId: string
      name: string
      email: string
      
# Define reusable traits
traits:
  paged:
    queryParameters:
      page: integer
      limit: integer
      
# Define API resources (endpoints)
/students:
  displayName: Students
  description: All student operations
  
  get:
    displayName: List Students
    description: Get all students
    is: [ paged ]  # Use trait
    responses:
      200:
        body:
          application/json:
            type: Student[]
```

### **Visual Hierarchy**

```
RAML File
├── Metadata (title, version, baseUri)
├── Types (data structures)
├── Traits (reusable behaviors)
├── Resources (endpoints)
│   ├── /students (resource)
│   │   ├── GET (method)
│   │   ├── POST (method)
│   │   └── /{studentId} (sub-resource)
│   │       ├── GET
│   │       ├── PUT
│   │       └── DELETE
└── Security Schemes
```

---

## 🔤 Data Types in RAML

### **Primitive Data Types**

#### **STRING - Text data**

```yaml
name:
  type: string
  
# Optional: Add constraints
email:
  type: string
  pattern: ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$  # Email regex
  minLength: 5
  maxLength: 100
  
# Example constraint usage
username:
  type: string
  minLength: 3
  maxLength: 20
  description: Username must be 3-20 characters
```

**Properties you can add:**
- `pattern:` - Regular expression the string must match
- `minLength:` - Minimum characters
- `maxLength:` - Maximum characters
- `enum:` - List of allowed values

```yaml
status:
  type: string
  enum: [active, inactive, pending]  # Only these values allowed
```

#### **NUMBER & INTEGER - Numeric data**

```yaml
# NUMBER - Can have decimals
price:
  type: number
  minimum: 0
  maximum: 10000
  multipleOf: 0.01  # Must be multiple of 0.01

# INTEGER - Whole numbers only
studentId:
  type: integer
  minimum: 1

age:
  type: integer
  minimum: 1
  maximum: 150
```

**Properties:**
- `minimum:` - Smallest allowed value
- `maximum:` - Largest allowed value
- `multipleOf:` - Value must be multiple of this
- `format:` - Can be `int32`, `int64`, `float`, `double`

#### **BOOLEAN - True/False**

```yaml
isActive:
  type: boolean
  description: Whether account is active

isEmailVerified:
  type: boolean
  default: false  # Default value if not provided
```

#### **DATE - Dates and Times**

```yaml
createdDate:
  type: date-only  # Format: YYYY-MM-DD (example: 2024-03-31)
  
lastLogin:
  type: datetime  # Format: RFC3339 (example: 2024-03-31T15:30:00Z)
  
birthDate:
  type: date-only
  description: Date of birth in format YYYY-MM-DD
```

**Date types:**
- `date-only` - Just the date (YYYY-MM-DD)
- `time-only` - Just the time (HH:mm:ss)
- `datetime-only` - Date and time (no timezone)
- `datetime` - Date, time, AND timezone (RFC3339)

#### **ARRAY - List of values**

```yaml
tags:
  type: array
  items:
    type: string
  description: List of tag names

# With size constraints
items:
  type: array
  items:
    type: object  # Array of objects
  minItems: 1     # At least 1 item
  maxItems: 100   # At most 100 items
  uniqueItems: true  # All items must be different
```

#### **OBJECT - Key-value pairs**

```yaml
address:
  type: object
  properties:
    street: string
    city: string
    zipCode: string
    country: string
```

### **Object Types - Complete Reference**

```yaml
# Simple object definition
address:
  type: object
  properties:
    street:
      type: string
    city:
      type: string
    zip:
      type: string

# Object with required fields
person:
  type: object
  properties:
    firstName:
      type: string
    lastName:
      type: string
    email:
      type: string
  required:        # These fields MUST be present
    - firstName
    - lastName
    - email

# Nested objects
student:
  type: object
  properties:
    name: string
    email: string
    address:       # Another object inside
      type: object
      properties:
        street: string
        city: string
        zip: string
```

### **Type Inheritance - Reusing Types**

```yaml
# Define a base type
person:
  type: object
  properties:
    name: string
    email: string
    age: integer

# Extend the base type
employee:
  type: person      # Inherits all properties from person
  properties:
    employeeId: string
    department: string
    salary: number

# Result: employee has ALL properties from person PLUS new ones
```

### **Union Types - Multiple Options**

```yaml
# A field can be multiple types
phoneNumber:
  type: string | integer
  description: Can be string (with formatting) or integer (numbers only)

# More complex example
contact:
  type: string | object
  # Can be a simple string like "john@example.com"
  # OR a full object with {type: "email", value: "john@example.com"}
```

### **Advanced: Custom Type with Validation**

```yaml
Email:
  type: string
  pattern: ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$

Phone:
  type: string
  pattern: ^\+?1?\d{9,15}$

Percentage:
  type: integer
  minimum: 0
  maximum: 100

Status:
  type: string
  enum: [active, inactive, pending, archived]

# Now use these custom types
user:
  type: object
  properties:
    email: Email         # Uses Email validation
    phone: Phone         # Uses Phone validation
    status: Status       # Only these values allowed
    discount: Percentage # 0-100 only
```

---

## 🌐 Resources & Endpoints

### **Understanding Resources**

**Resource** = An endpoint in your API
- `/students` = Resource for students
- `/students/{id}` = Resource for specific student
- `/orders` = Resource for orders

```yaml
/students:              # This is a resource
  /enrollments:         # This is a sub-resource
    /{enrollmentId}:    # This is a sub-resource with parameter
```

### **Creating Resources**

```yaml
/students:
  displayName: Students
  description: Manage all student records
```

**Best Practices for resource naming:**
- Use **lowercase**
- Use **plural** names (`/students` not `/student`)
- Use **nouns** (`/students`, `/courses`) not verbs (`/getStudent`, `/createOrder`)
- Use **forward slashes** for hierarchy (`/students/{id}/courses`)

### **URI Parameters (Path Variables)**

```yaml
# Single parameter
/students/{studentId}:
  displayName: Student by ID
  description: Get a specific student by their ID
  uriParameters:
    studentId:
      type: string
      description: The unique student identifier
      required: true

# Multiple parameters
/students/{studentId}/courses/{courseId}:
  uriParameters:
    studentId:
      type: string
    courseId:
      type: string
      
# Parameter with constraints
/departments/{deptCode}:
  uriParameters:
    deptCode:
      type: string
      pattern: ^[A-Z]{2,4}$  # 2-4 uppercase letters
      minLength: 2
      maxLength: 4
```

### **Resource with Multiple Methods**

```yaml
/products:
  get:
    description: List all products
    responses:
      200:
        body:
          type: Product[]
          
  post:
    description: Create new product
    body:
      type: Product
    responses:
      201:
        body:
          type: Product

/products/{productId}:
  get:
    description: Get specific product
    responses:
      200:
        body:
          type: Product
          
  put:
    description: Update product
    body:
      type: Product
    responses:
      200:
        body:
          type: Product
          
  delete:
    description: Delete product
    responses:
      204:
        description: Product deleted successfully
```

---

## 🔄 HTTP Methods (GET, POST, PUT, DELETE)

### **GET - Retrieve Data (Read Only)**

```yaml
/students:
  get:
    displayName: List All Students
    description: Retrieve a list of all students
    responses:
      200:
        description: Successfully retrieved students
        body:
          application/json:
            type: Student[]
            
/students/{studentId}:
  get:
    displayName: Get Student
    description: Retrieve a specific student by ID
    uriParameters:
      studentId:
        type: string
    responses:
      200:
        description: Student found
        body:
          application/json:
            type: Student
      404:
        description: Student not found
```

**Rules for GET:**
- ✅ Retrieve data (NEVER modify)
- ✅ Safe (doesn't change data)
- ✅ Idempotent (calling twice = same result)
- ✅ Can have query parameters
- ❌ Cannot have request body
- ❌ Do NOT use for creating/updating/deleting

### **POST - Create New Data**

```yaml
/students:
  post:
    displayName: Create Student
    description: Create a new student record
    body:
      application/json:
        type: Student
        example: |
          {
            "name": "John Doe",
            "email": "john@example.com",
            "age": 22
          }
    responses:
      201:
        description: Student created successfully
        body:
          application/json:
            type: Student
            example: |
              {
                "studentId": "S123",
                "name": "John Doe",
                "email": "john@example.com",
                "age": 22,
                "createdDate": "2024-03-31"
              }
      400:
        description: Invalid input
      409:
        description: Student already exists
```

**Rules for POST:**
- ✅ Create new resource
- ✅ Has request body (data to create)
- ✅ Returns 201 (Created) on success
- ✅ Can have side effects
- ❌ NOT idempotent (calling twice = creates twice)
- ❌ Should NOT use for retrieving data

### **PUT - Update Entire Resource**

```yaml
/students/{studentId}:
  put:
    displayName: Update Student
    description: Update entire student record (must provide all fields)
    uriParameters:
      studentId:
        type: string
    body:
      application/json:
        type: Student
        description: Complete student data
    responses:
      200:
        description: Student updated successfully
        body:
          application/json:
            type: Student
      404:
        description: Student not found
      400:
        description: Invalid input
```

**Rules for PUT:**
- ✅ Update entire resource
- ✅ Requires complete data (all fields)
- ✅ Idempotent (calling twice = same result)
- ✅ Returns 200 (OK)
- ❌ Must provide ALL fields (even unchanged ones)
- ⚠️ Use PATCH for partial updates

### **PATCH - Partial Update**

```yaml
/students/{studentId}:
  patch:
    displayName: Partial Update Student
    description: Update specific fields of student (only send what changed)
    uriParameters:
      studentId:
        type: string
    body:
      application/json:
        type: object
        properties:
          name:
            type: string
            required: false
          email:
            type: string
            required: false
        description: Only provide fields to update
    responses:
      200:
        description: Student updated
        body:
          application/json:
            type: Student
```

**Difference: PUT vs PATCH**

```yaml
Original student:
{
  "studentId": "S1",
  "name": "John",
  "email": "john@example.com",
  "age": 22
}

PUT (replace entire):
Request body MUST have ALL fields:
{
  "name": "Jane",
  "email": "jane@example.com",
  "age": 22
}
Result: If you forget "age", it's deleted!

PATCH (update specific):
Request body has only fields to change:
{
  "name": "Jane"
}
Result: Only name changes, email and age stay same
```

### **DELETE - Remove Resource**

```yaml
/students/{studentId}:
  delete:
    displayName: Delete Student
    description: Remove a student record permanently
    uriParameters:
      studentId:
        type: string
    responses:
      204:
        description: Student deleted successfully (no content)
      404:
        description: Student not found
      409:
        description: Cannot delete - has active enrollments
```

**Rules for DELETE:**
- ✅ Remove resource
- ✅ Typically returns 204 (No Content)
- ✅ Idempotent (calling twice = same result)
- ✅ Cannot have request body
- ❌ Be careful - it's destructive!

---

## 📨 Request & Response Bodies

### **Request Body - What Client Sends**

```yaml
/students:
  post:
    displayName: Create Student
    body:
      application/json:
        type: Student
        description: The student data to create
        example: |
          {
            "name": "John Doe",
            "email": "john@example.com",
            "age": 22,
            "department": "IT"
          }
    responses:
      201:
        description: Created
```

### **Response Body - What Server Sends Back**

```yaml
/students/{studentId}:
  get:
    displayName: Get Student
    responses:
      200:
        description: Success
        body:
          application/json:
            type: Student
            example: |
              {
                "studentId": "S123",
                "name": "John Doe",
                "email": "john@example.com",
                "age": 22,
                "department": "IT",
                "createdDate": "2024-03-31"
              }
      404:
        description: Not found
        body:
          application/json:
            type: object
            properties:
              error:
                type: string
              message:
                type: string
            example: |
              {
                "error": "NOT_FOUND",
                "message": "Student with ID S999 not found"
              }
```

### **Multiple Content Types**

```yaml
/students:
  post:
    body:
      application/json:
        type: Student
      application/xml:
        type: Student
        # XML schema goes here
    responses:
      201:
        body:
          application/json:
            type: Student
          application/xml:
            type: Student
```

---

## 🔍 Query Parameters

### **What are Query Parameters?**

Query parameters are optional filters/options added to the URL after `?`

```
GET /students?page=1&limit=10&department=IT
                │                  │
                Query parameters───┘
```

### **Defining Query Parameters**

```yaml
/students:
  get:
    displayName: List Students
    queryParameters:
      page:
        type: integer
        description: Page number (starts at 1)
        required: false
        default: 1
        
      limit:
        type: integer
        description: Number of results per page
        required: false
        default: 20
        minimum: 1
        maximum: 100
        
      department:
        type: string
        description: Filter by department
        required: false
        enum: [IT, HR, Finance, Marketing]
        
      name:
        type: string
        description: Search by student name (partial match)
        required: false
        
      sort:
        type: string
        description: Sort by field (prefix - for descending)
        required: false
        enum: [name, -name, age, -age, email, -email]
        
      active:
        type: boolean
        description: Filter by active status
        required: false
    
    responses:
      200:
        body:
          application/json:
            type: Student[]
```

### **Query Parameters - Real Examples**

```yaml
# Pagination
GET /students?page=2&limit=50

# Filtering
GET /students?department=IT&active=true

# Searching
GET /students?name=john&department=IT

# Sorting
GET /students?sort=-age (descending age)
GET /students?sort=name (ascending name)

# Combining
GET /students?page=1&limit=20&department=IT&sort=name
```

---

## 🎯 URI Parameters

### **What are URI Parameters?**

URI Parameters are **required** parts of the URL path, not optional

```
GET /students/{studentId}/courses/{courseId}
                │                    │
                URI parameters───────┘
(REQUIRED - cannot be omitted)
```

### **Defining URI Parameters**

```yaml
/students/{studentId}:
  displayName: Specific Student
  uriParameters:
    studentId:
      type: string
      description: Unique student ID (e.g., S001, S002)
      required: true
      
  get:
    displayName: Get Student Details
    responses:
      200:
        body:
          application/json:
            type: Student

/students/{studentId}/courses/{courseId}:
  displayName: Student Course
  uriParameters:
    studentId:
      type: string
      description: The student ID
    courseId:
      type: string
      description: The course ID
      
  get:
    responses:
      200:
        body:
          application/json:
            type: Course
```

### **URI Parameters with Patterns**

```yaml
/users/{userId}:
  uriParameters:
    userId:
      type: integer
      minimum: 1
      description: User ID must be positive integer

/departments/{deptCode}:
  uriParameters:
    deptCode:
      type: string
      pattern: ^[A-Z]{2,4}$  # Only 2-4 uppercase letters
      minLength: 2
      maxLength: 4
      description: Department code (e.g., IT, HR, FIN, MKTG)

/files/{filename}:
  uriParameters:
    filename:
      type: string
      pattern: ^[a-zA-Z0-9._-]+\.[a-zA-Z0-9]+$  # Valid filename
      description: File name with extension
```

### **Query Parameters vs URI Parameters**

```yaml
# URI Parameter (REQUIRED - part of path)
GET /students/{studentId}
- studentId MUST be provided
- Example: /students/S001

# Query Parameter (OPTIONAL - after ?)
GET /students?department=IT
- department is optional
- Can omit: /students

# Mixing both
GET /students/{studentId}/courses?sort=name
- studentId REQUIRED (URI parameter)
- sort is OPTIONAL (query parameter)
```

---

## 📬 Headers

### **What are Headers?**

Headers are extra information sent with request/response, not part of the main body

```
GET /students/S001
Header: Authorization: Bearer token123
Header: Content-Type: application/json
Header: X-Request-ID: req-12345
```

### **Common Request Headers**

```yaml
/students:
  get:
    displayName: List Students
    headers:
      Authorization:
        type: string
        description: Bearer token for authentication
        required: true
        example: "Bearer eyJhbGciOiJIUzI1NiIs..."
        
      X-Request-ID:
        type: string
        description: Unique request identifier for tracking
        required: false
        pattern: ^[a-zA-Z0-9-]{20,}$
        
      Accept-Language:
        type: string
        description: Preferred language
        required: false
        enum: [en, es, fr, de]
        
      X-API-Version:
        type: string
        description: API version to use
        required: false
        default: "1.0"
        
    responses:
      200:
        body:
          application/json:
            type: Student[]
```

### **Response Headers**

```yaml
/students:
  get:
    responses:
      200:
        description: Success
        headers:
          X-Request-ID:
            type: string
            description: The request ID from request
            
          X-RateLimit-Limit:
            type: integer
            description: Total requests allowed in time window
            example: 1000
            
          X-RateLimit-Remaining:
            type: integer
            description: Requests remaining
            example: 950
            
          X-RateLimit-Reset:
            type: integer
            description: Unix timestamp when limit resets
            example: 1704067200
            
          Cache-Control:
            type: string
            description: Caching directive
            example: "max-age=3600, public"
            
        body:
          application/json:
            type: Student[]
```

### **Common Headers Reference**

| Header | Purpose | Example |
|--------|---------|---------|
| `Authorization` | Authentication | Bearer token |
| `Content-Type` | Data format | application/json |
| `Accept` | Desired format | application/json |
| `X-Request-ID` | Request tracking | req-123abc |
| `X-RateLimit-*` | Rate limiting | 1000 requests/hour |
| `Cache-Control` | Caching rules | max-age=3600 |
| `CORS-*` | Cross-origin | Access-Control-Allow-* |

---

## ✋ Status Codes & Error Responses

### **HTTP Status Codes**

```yaml
# 2xx - Success
200: OK                    # Request succeeded
201: Created               # Resource created
202: Accepted              # Request accepted, processing
204: No Content            # Success, no body (DELETE)

# 3xx - Redirection
301: Moved Permanently     # Resource moved
302: Found                 # Temporary redirect
304: Not Modified          # Use cached version

# 4xx - Client Error
400: Bad Request           # Invalid input
401: Unauthorized          # Authentication required
403: Forbidden             # Authorized but not allowed
404: Not Found             # Resource doesn't exist
409: Conflict              # Resource already exists
422: Unprocessable Entity  # Validation failed
429: Too Many Requests     # Rate limit exceeded

# 5xx - Server Error
500: Internal Server Error # Server error
502: Bad Gateway          # Upstream unavailable
503: Service Unavailable  # Temporarily down
```

### **Error Response Examples**

```yaml
/students/{studentId}:
  get:
    responses:
      200:
        description: Success
        body:
          application/json:
            type: Student
            example: |
              {
                "studentId": "S123",
                "name": "John Doe",
                "email": "john@example.com"
              }
              
      404:
        description: Student not found
        body:
          application/json:
            type: object
            properties:
              error:
                type: string
                enum: [NOT_FOUND]
              message:
                type: string
              timestamp:
                type: datetime
              path:
                type: string
            example: |
              {
                "error": "NOT_FOUND",
                "message": "Student with ID S999 not found",
                "timestamp": "2024-03-31T15:30:00Z",
                "path": "/students/S999"
              }
```

### **Complete Error Response Schema**

```yaml
types:
  Error:
    type: object
    properties:
      error:
        type: string
        description: Error code
        enum: [BAD_REQUEST, UNAUTHORIZED, FORBIDDEN, NOT_FOUND, CONFLICT, UNPROCESSABLE_ENTITY, INTERNAL_ERROR]
      message:
        type: string
        description: Human-readable error message
      details:
        type: array
        items:
          type: object
          properties:
            field:
              type: string
            issue:
              type: string
        description: Field-level validation errors
      timestamp:
        type: datetime
        description: When error occurred
      path:
        type: string
        description: API path that caused error

# Usage:
/students:
  post:
    responses:
      400:
        description: Validation failed
        body:
          application/json:
            type: Error
            example: |
              {
                "error": "BAD_REQUEST",
                "message": "Validation failed",
                "details": [
                  {"field": "email", "issue": "Invalid email format"},
                  {"field": "age", "issue": "Must be 18 or older"}
                ],
                "timestamp": "2024-03-31T15:30:00Z",
                "path": "/students"
              }
```

---

## 🎨 Type System - Everything

### **Defining Custom Types**

```yaml
# Simple object type
Student:
  type: object
  properties:
    studentId: string
    name: string
    email: string
    age: integer

# Reusing the type
/students:
  get:
    responses:
      200:
        body:
          application/json:
            type: Student[]  # Array of students
```

### **Types with Inheritance**

```yaml
# Base type
Person:
  type: object
  properties:
    name: string
    email: string
    dateOfBirth: date-only

# Extended type
Employee:
  type: Person         # Inherits from Person
  properties:
    employeeId: string
    department: string
    salary: number

# Result: Employee has name, email, dateOfBirth, employeeId, department, salary
```

### **Types with All Features**

```yaml
Student:
  displayName: Student Record
  description: Complete student information
  type: object
  properties:
    studentId:
      type: string
      description: Unique identifier
      required: true
      
    name:
      type: string
      minLength: 2
      maxLength: 100
      description: Full name
      required: true
      
    email:
      type: string
      pattern: ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$
      required: true
      
    age:
      type: integer
      minimum: 18
      maximum: 80
      description: Age in years
      
    gpa:
      type: number
      minimum: 0
      maximum: 4.0
      multipleOf: 0.01
      
    status:
      type: string
      enum: [active, inactive, suspended]
      default: active
      
    department:
      type: string
      
    courses:
      type: array
      items:
        type: string
      minItems: 1
      maxItems: 10
      
    address:
      type: object
      properties:
        street: string
        city: string
        zip: string
        country: string
        
    createdDate:
      type: datetime
      
  required: [studentId, name, email, age]  # These fields MUST be present
```

---

## 🎯 Traits - DRY Principle

### **What are Traits?**

Traits = Reusable behaviors you can apply to multiple endpoints
- **Problem:** You have 50 endpoints that all need pagination. Do you copy-paste 50 times?
- **Solution:** Define pagination trait ONCE, use it in all 50 endpoints

### **Creating Traits**

```yaml
traits:
  paged:
    displayName: Pageable
    description: Supports pagination
    queryParameters:
      page:
        type: integer
        description: Page number (starts at 1)
        default: 1
        required: false
      limit:
        type: integer
        description: Items per page
        default: 20
        minimum: 1
        maximum: 100
        required: false

  sorted:
    displayName: Sortable
    queryParameters:
      sort:
        type: string
        description: Sort by field (prefix - for descending)
        example: "name or -name"
        required: false

  authenticated:
    displayName: Requires Authentication
    headers:
      Authorization:
        type: string
        description: Bearer token
        required: true
        pattern: ^Bearer\s[a-zA-Z0-9\-._~\+\/]+=*$
```

### **Using Traits**

```yaml
/students:
  get:
    displayName: List Students
    is: [ paged, sorted, authenticated ]  # Apply all three traits
    responses:
      200:
        body:
          application/json:
            type: Student[]

# Now /students endpoint automatically has:
# - page and limit query parameters
# - sort query parameter
# - Authorization header requirement
```

### **Trait with Responses**

```yaml
traits:
  notFound:
    description: Not found error
    responses:
      404:
        description: Resource not found
        body:
          application/json:
            type: object
            properties:
              error: string
              message: string

/students/{studentId}:
  uriParameters:
    studentId: string
  get:
    is: [ notFound ]
    responses:
      200:
        body:
          application/json:
            type: Student
      
# GET automatically includes the 404 response from trait
```

### **Trait with Multiple Properties**

```yaml
traits:
  errorable:
    responses:
      400:
        body:
          application/json:
            type: Error
      401:
        body:
          application/json:
            type: Error
      500:
        body:
          application/json:
            type: Error

/students:
  post:
    is: [ errorable ]  # All error responses included
```

---

## 🔐 Security Schemes

### **Authentication Methods**

#### **API Key Authentication**

```yaml
securitySchemes:
  apiKey:
    type: API Key
    description: API key for authentication
    name: X-API-Key      # Header name
    in: header           # Where it's located

/students:
  get:
    securedBy: [ apiKey ]  # Requires API key
    responses:
      200:
        body:
          application/json:
            type: Student[]
```

#### **Bearer Token (OAuth 2.0)**

```yaml
securitySchemes:
  oauth2:
    type: OAuth 2.0
    description: OAuth 2.0 for user authentication
    authorizationUri: https://api.example.com/oauth2/authorize
    accessTokenUri: https://api.example.com/oauth2/token
    authorizationGrants: [authorization_code, implicit]
    scopes:
      read: Read access
      write: Write access
      delete: Delete access

/students:
  get:
    securedBy: [ oauth2 ]  # Any scope allowed
    responses:
      200:
        body:
          application/json:
            type: Student[]

/students:
  post:
    securedBy: [oauth2: [write]]  # Requires 'write' scope
    responses:
      201:
        body:
          application/json:
            type: Student
```

#### **Basic Authentication**

```yaml
securitySchemes:
  basic:
    type: Basic Authentication
    description: Username and password

/students:
  get:
    securedBy: [ basic ]
    responses:
      200:
        body:
          application/json:
            type: Student[]
```

### **Multiple Security Options**

```yaml
/students:
  get:
    securedBy: [ basic, oauth2 ]  # Either method works
```

---

## ⚠️ Common Mistakes & Workarounds

### **Mistake 1: Using Singular Resource Names**

```yaml
# ❌ WRONG
/student:
  get:
    responses:
      200:
        body:
          type: Student

# ✅ CORRECT
/students:
  get:
    responses:
      200:
        body:
          type: Student[]
```

### **Mistake 2: Using Verbs in Resource Names**

```yaml
# ❌ WRONG
/getStudent:
/createStudent:
/deleteStudent:

# ✅ CORRECT
/students:
  get:
  post:
  delete:
```

### **Mistake 3: Wrong Status Codes**

```yaml
# ❌ WRONG - Using 200 for creation
/students:
  post:
    responses:
      200:  # ← Wrong!
        body:
          type: Student

# ✅ CORRECT - Using 201 for creation
/students:
  post:
    responses:
      201:  # ← Correct
        body:
          type: Student
```

### **Mistake 4: Missing Required Fields**

```yaml
# ❌ WRONG - No required specification
Student:
  type: object
  properties:
    studentId: string
    name: string
    email: string

# ✅ CORRECT - Specify what's required
Student:
  type: object
  properties:
    studentId: string
    name: string
    email: string
  required: [studentId, name, email]
```

### **Mistake 5: Forgetting Error Responses**

```yaml
# ❌ WRONG - No error responses
/students/{studentId}:
  get:
    responses:
      200:
        body:
          type: Student

# ✅ CORRECT - Include error responses
/students/{studentId}:
  get:
    responses:
      200:
        body:
          type: Student
      404:
        body:
          type: Error
      500:
        body:
          type: Error
```

### **Mistake 6: Inconsistent Data Types**

```yaml
# ❌ WRONG - Inconsistent ID format
Student:
  properties:
    studentId: integer    # Number here
    
Course:
  properties:
    courseId: string      # String here (inconsistent!)

# ✅ CORRECT - Consistent format
Student:
  properties:
    studentId: string
    
Course:
  properties:
    courseId: string
```

### **Mistake 7: Not Using Traits for Common Patterns**

```yaml
# ❌ WRONG - Repeating pagination in every endpoint
/students:
  get:
    queryParameters:
      page: integer
      limit: integer
      
/courses:
  get:
    queryParameters:
      page: integer
      limit: integer
      
/departments:
  get:
    queryParameters:
      page: integer
      limit: integer

# ✅ CORRECT - Use trait
traits:
  paged:
    queryParameters:
      page: integer
      limit: integer
      
/students:
  get:
    is: [ paged ]
    
/courses:
  get:
    is: [ paged ]
    
/departments:
  get:
    is: [ paged ]
```

### **Workaround: Optional Body**

```yaml
# If body should sometimes be present and sometimes not
/students:
  patch:
    body:
      application/json:
        type: object
        properties:
          name: string
          email: string
        # No required fields - all are optional
```

### **Workaround: Versioning Your API**

```yaml
# Version 1
#%RAML 1.0
title: Student API
version: v1
baseUri: https://api.example.com/{version}

# Version 2 - Breaking changes
#%RAML 1.0
title: Student API
version: v2
baseUri: https://api.example.com/{version}

# Both v1 and v2 can run simultaneously
# - /v1/students (old API)
# - /v2/students (new API)
```

---

## 📊 Summary - RAML Fundamentals

| Concept | Use Case | Example |
|---------|----------|---------|
| Resources | Define endpoints | `/students`, `/courses` |
| Methods | Define operations | GET, POST, PUT, DELETE |
| Types | Define data structures | `Student`, `Course` |
| Parameters | Pass data | Query, URI, headers |
| Traits | Reuse behaviors | Pagination, authentication |
| Responses | Document outputs | 200, 404, 500 |
| Security | Protect endpoints | API keys, OAuth 2.0 |

---

**Next:** Continue to Part 2 for advanced RAML features and patterns!
