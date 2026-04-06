# 🚀 RAML Complete Masterclass - Part 2: Advanced Features & Patterns

> **Master every advanced RAML concept with real-world examples**

---

## 📚 Table of Contents

1. [Resource Typing](#resource-typing)
2. [Method Typing](#method-typing)
3. [Response Typing](#response-typing)
4. [External Types](#external-types)
5. [Annotations](#annotations)
6. [Libraries & Reusability](#libraries--reusability)
7. [Documentation](#documentation)
8. [Examples](#examples)
9. [Advanced Patterns](#advanced-patterns)
10. [Design Best Practices](#design-best-practices)
11. [Common Issues & Solutions](#common-issues--solutions)

---

## 🎨 Resource Typing

### **What is Resource Typing?**

Resource Typing = Define a template for how resources should behave
- Instead of defining each endpoint separately, create a template
- Apply template to multiple resources

### **Basic Resource Type**

```yaml
resourceTypes:
  item:
    displayName: Item Resource
    get:
      responses:
        200:
          body:
            application/json:
              type: <<resourcePathName>>  # <<resourcePathName>> = singular form
        404:
          body:
            type: Error
    post:
      responses:
        201:
          body:
            application/json:
              type: <<resourcePathName>>
    put:
      responses:
        200:
          body:
            application/json:
              type: <<resourcePathName>>
    delete:
      responses:
        204:

# Usage:
/students:
  type: item

# This creates:
# GET /students, POST /students, PUT /students, DELETE /students
# All with correct response types automatically!
```

### **Resource Type with Parameters**

```yaml
resourceTypes:
  collection:
    displayName: Collection
    get:
      is: [paged, sorted]
      responses:
        200:
          body:
            type: <<resourcePathName>>[]
    post:
      responses:
        201:
          body:
            type: <<resourcePathName>>

  member:
    displayName: Member
    get:
      responses:
        200:
          body:
            type: <<resourcePathName>>
        404:
          body:
            type: Error
    put:
      responses:
        200:
          body:
            type: <<resourcePathName>>
    delete:
      responses:
        204:

# Usage:
/students:
  type: collection

/students/{studentId}:
  type: member
```

### **Resource Type with Custom Parameters**

```yaml
resourceTypes:
  searchable:
    get:
      queryParameters:
        <<searchKey>>:
          type: string
          description: Search by <<searchKey>>
          required: false

# Usage:
/students:
  type: searchable
  uriParameters:
    searchKey: name

/products:
  type: searchable
  uriParameters:
    searchKey: category
```

---

## 🎯 Method Typing

### **What is Method Typing?**

Method Typing = Define reusable method definitions

```yaml
methodTypes:
  paginatedGet:
    displayName: Paginated GET
    is: [paged, sorted]
    responses:
      200:
        body:
          type: <<responseType>>[]

  restrictedPost:
    displayName: Restricted POST
    securedBy: [oauth2: [write]]
    responses:
      201:
        body:
          type: <<responseType>>
      401:
        body:
          type: Error

# Usage:
/students:
  get:
    is: [paginatedGet]
    parameters:
      responseType: Student

/courses:
  post:
    is: [restrictedPost]
    parameters:
      responseType: Course
```

---

## 📋 Response Typing

### **Defining Response Type**

```yaml
responseTypes:
  standard:
    200:
      body:
        application/json:
          type: <<dataType>>

  errorResponse:
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
```

---

## 📂 External Types

### **Organizing in Multiple Files**

**Project Structure:**
```
api/
├── student-api.raml       (Main file)
├── types/
│   ├── student.raml
│   ├── course.raml
│   └── error.raml
├── traits/
│   └── common.raml
└── examples/
    └── samples.json
```

### **Defining External Types (types/student.raml)**

```yaml
#%RAML 1.0 DataType

Student:
  type: object
  properties:
    studentId:
      type: string
    name:
      type: string
    email:
      type: string
    age:
      type: integer

StudentList:
  type: array
  items:
    $ref: '#/Student'
```

### **Using External Types (student-api.raml)**

```yaml
#%RAML 1.0
title: Student API
version: v1
baseUri: https://api.example.com/{version}

uses:
  studentTypes: types/student.raml
  errorTypes: types/error.raml
  commonTraits: traits/common.raml

/students:
  get:
    is: [commonTraits.paged]
    responses:
      200:
        body:
          type: studentTypes.StudentList
      400:
        body:
          type: errorTypes.Error
```

---

## 🏷️ Annotations

### **What are Annotations?**

Annotations = Add metadata/tags to document additional information

```yaml
# Define custom annotation
annotationTypes:
  deprecated:
    type: string
    description: Marks endpoint as deprecated
    
  rate-limit:
    type: object
    properties:
      limit:
        type: integer
      window:
        type: string

# Use annotation
/students:
  (deprecated): "Use /api/v2/students instead"
  
  get:
    (rate-limit):
      limit: 1000
      window: "1 hour"
```

### **Common Annotations**

```yaml
annotationTypes:
  experimental:
    type: string
    description: Feature is experimental, may change
    
  confidential:
    type: string
    description: API contains confidential data
    
  requiresApproval:
    type: string
    description: Requires special approval to use
    
  supportedIn:
    type: string
    description: List of supported API versions

# Usage:
/admin/users:
  (requiresApproval): "Requires admin approval"
  (confidential): "Contains user personal data"

/experimental/feature:
  (experimental): "May change in future releases"
  (supportedIn): "v2 and above only"
```

---

## 📚 Libraries & Reusability

### **Creating a Library (library.raml)**

```yaml
#%RAML 1.0 Library

traits:
  paged:
    queryParameters:
      page:
        type: integer
        default: 1
      limit:
        type: integer
        default: 20

  authenticated:
    headers:
      Authorization:
        type: string
        required: true

resourceTypes:
  standardResource:
    get:
      is: [paged]
    post:
    put:
    delete:

types:
  Error:
    type: object
    properties:
      error: string
      message: string
      timestamp: datetime

  Pagination:
    type: object
    properties:
      page: integer
      limit: integer
      total: integer
```

### **Using Library (main-api.raml)**

```yaml
#%RAML 1.0
title: My API
version: v1
baseUri: https://api.example.com/{version}

uses:
  lib: library.raml

/students:
  type: lib.standardResource
  get:
    is: [lib.paged]
    responses:
      200:
        body:
          type: Student[]

/courses:
  type: lib.standardResource
  get:
    is: [lib.authenticated]
```

---

## 📝 Documentation

### **Adding Descriptions**

```yaml
/students:
  displayName: Students
  description: |
    Manage student records.
    
    This endpoint handles:
    - Retrieving student lists
    - Creating new students
    - Updating student information
    - Deleting student records
    
    **Important:** All operations require authentication.
    
  get:
    displayName: Get All Students
    description: |
      Retrieves a paginated list of all students.
      
      **Example:**
      ```
      GET /students?page=1&limit=20
      ```
      
      Returns up to 20 students on page 1.
    
    responses:
      200:
        description: Successful retrieval
```

### **Using Markdown in Descriptions**

```yaml
/students:
  description: |
    # Student Management
    
    This API provides complete student management functionality.
    
    ## Features
    - List students with pagination
    - Create new students
    - Update student records
    - Delete students
    - Search by name
    
    ## Authentication
    All endpoints require Bearer token authentication.
    
    ```
    Authorization: Bearer <your-token>
    ```
    
    ## Rate Limiting
    - 1000 requests per hour
    - Returns 429 if exceeded
```

---

## 🎬 Examples

### **Single Example**

```yaml
/students:
  post:
    body:
      application/json:
        type: Student
        example: |
          {
            "name": "John Doe",
            "email": "john@example.com",
            "age": 22,
            "department": "IT"
          }
    responses:
      201:
        body:
          application/json:
            type: Student
            example: |
              {
                "studentId": "S001",
                "name": "John Doe",
                "email": "john@example.com",
                "age": 22,
                "department": "IT",
                "createdDate": "2024-03-31T10:30:00Z"
              }
```

### **Multiple Examples (Named)**

```yaml
/students:
  post:
    body:
      application/json:
        type: Student
        examples:
          minimalist: |
            {
              "name": "Jane",
              "email": "jane@example.com",
              "age": 20,
              "department": "CS"
            }
          complete: |
            {
              "name": "John Doe",
              "email": "john@example.com",
              "age": 22,
              "department": "IT",
              "gpa": 3.8
            }
```

### **External Examples**

```yaml
uses:
  examples: examples/samples.json

/students:
  post:
    body:
      application/json:
        type: Student
        example: !include examples/student-example.json
```

---

## 🎯 Advanced Patterns

### **Pattern 1: Nested Resources**

```yaml
/students:
  displayName: Students
  
  get:
    responses:
      200:
        body:
          type: Student[]
  
  /{studentId}:
    displayName: Student Detail
    uriParameters:
      studentId: string
    
    get:
      responses:
        200:
          body:
            type: Student
    
    /courses:
      displayName: Student Courses
      
      get:
        responses:
          200:
            body:
              type: Course[]
      
      /{courseId}:
        displayName: Student Course
        
        get:
          responses:
            200:
              body:
                type: Enrollment
        
        /grades:
          displayName: Course Grades
          
          get:
            responses:
              200:
                body:
                  type: Grade[]
```

### **Pattern 2: Polymorphic Types**

```yaml
types:
  Document:
    type: object
    discriminator: type  # Use 'type' field to determine subtype
    properties:
      id: string
      type:
        type: string
        enum: [PDF, Word, Excel]
      content: string

  PDFDocument:
    type: Document
    properties:
      pages: integer

  WordDocument:
    type: Document
    properties:
      version: string

# Usage:
/documents:
  post:
    body:
      type: Document  # Could be any subtype
    responses:
      201:
        body:
          type: Document
```

### **Pattern 3: Versioning**

```yaml
#%RAML 1.0
title: API
version: v2
baseUri: https://api.example.com/{version}

types:
  StudentV1:
    type: object
    properties:
      studentId: string
      name: string

  StudentV2:
    type: object
    properties:
      studentId: string
      name: string
      email: string  # New field in v2
      phone: string  # New field in v2

/students:
  get:
    responses:
      200:
        body:
          application/json:
            type: StudentV2[]
```

### **Pattern 4: Batch Operations**

```yaml
types:
  BatchRequest:
    type: object
    properties:
      requests:
        type: array
        items:
          type: object
          properties:
            method: string
            url: string
            body: object

  BatchResponse:
    type: object
    properties:
      responses:
        type: array
        items:
          type: object
          properties:
            status: integer
            body: object

/batch:
  post:
    displayName: Batch Request
    description: Submit multiple requests in one call
    body:
      type: BatchRequest
    responses:
      200:
        body:
          type: BatchResponse
```

### **Pattern 5: Soft Delete**

```yaml
types:
  Student:
    type: object
    properties:
      studentId: string
      name: string
      email: string
      isDeleted: boolean  # Soft delete flag
      deletedDate:
        type: datetime
        required: false  # Only if deleted

/students:
  get:
    queryParameters:
      includeDeleted:
        type: boolean
        default: false
        description: Include soft-deleted records?
    responses:
      200:
        body:
          type: Student[]

  /{studentId}:
    delete:
      responses:
        204:
          description: Student soft-deleted (marked deleted, not removed)
```

### **Pattern 6: Search/Filter**

```yaml
/students:
  get:
    displayName: Search Students
    queryParameters:
      name:
        type: string
        description: Search by name (partial match)
        required: false
      department:
        type: string
        enum: [IT, CS, HR, Finance]
        required: false
      minGpa:
        type: number
        minimum: 0
        maximum: 4.0
        required: false
      maxAge:
        type: integer
        required: false
      status:
        type: string
        enum: [active, inactive, suspended]
        required: false
      sort:
        type: string
        enum: [name, -name, gpa, -gpa, createdDate, -createdDate]
        default: name
        required: false
      page:
        type: integer
        default: 1
      limit:
        type: integer
        default: 20
```

---

## 📊 Design Best Practices

### **1. Resource-Oriented Design**

```yaml
# ✅ CORRECT - Resources, not actions
GET /students              # Get list
POST /students             # Create
GET /students/{id}         # Get one
PUT /students/{id}         # Update
DELETE /students/{id}      # Delete

# ❌ WRONG - Action-oriented
GET /getStudents
POST /createStudent
GET /getStudentById?id=1
POST /updateStudent
POST /deleteStudent?id=1
```

### **2. Consistent Naming**

```yaml
# ✅ CORRECT - Consistent plural, lowercase
/students
/courses
/departments
/enrollments

# ❌ WRONG - Inconsistent
/student
/Courses
/DEPARTMENTS
/enrollment_records
```

### **3. Proper HTTP Methods**

```yaml
# ✅ CORRECT - Proper methods
GET /students                 # Retrieve list
POST /students                # Create new
GET /students/{id}            # Retrieve one
PUT /students/{id}            # Replace
PATCH /students/{id}          # Partial update
DELETE /students/{id}         # Delete

# ❌ WRONG - Wrong methods
GET /students/create          # Should be POST
POST /students/get            # Should be GET
POST /students/{id}/delete    # Should be DELETE
```

### **4. Status Codes**

```yaml
# ✅ CORRECT - Proper status codes
200 OK                 # Successful GET/PUT/PATCH
201 Created            # Successful POST
204 No Content         # Successful DELETE
400 Bad Request        # Invalid input
401 Unauthorized       # Auth required/failed
403 Forbidden          # Auth OK but not allowed
404 Not Found          # Resource doesn't exist
409 Conflict           # Resource already exists
422 Unprocessable      # Validation failed
429 Too Many Requests  # Rate limited
500 Server Error       # Server error
503 Service Unavailable  # Temporarily down

# ❌ WRONG - Wrong status codes
200 OK                 # Don't use for DELETE (use 204)
500 Server Error       # Don't use for bad input (use 400)
200 OK                 # Don't use for resource not found (use 404)
```

### **5. Error Responses**

```yaml
# ✅ CORRECT - Structured error
{
  "error": "VALIDATION_FAILED",
  "message": "Input validation failed",
  "details": [
    {
      "field": "email",
      "issue": "Invalid email format"
    },
    {
      "field": "age",
      "issue": "Must be 18 or older"
    }
  ],
  "timestamp": "2024-03-31T15:30:00Z",
  "path": "/students"
}

# ❌ WRONG - Vague error
{
  "message": "Error"
}
```

### **6. Pagination**

```yaml
# ✅ CORRECT - Pagination info
{
  "data": [
    {...}, {...}, ...
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 1250,
    "pages": 63
  }
}

# ❌ WRONG - No pagination info
[{...}, {...}, ...]  # How many total? What page?
```

### **7. API Versioning**

```yaml
# ✅ CORRECT - Version in URL
GET /v1/students
GET /v2/students    # Breaking changes in v2

# Also acceptable - Version in header
GET /students
Header: API-Version: 2

# ❌ WRONG - No versioning
GET /students       # What happens when API changes?
```

### **8. Documentation**

```yaml
# ✅ CORRECT - Clear documentation
/students/{studentId}:
  displayName: Student Detail
  description: |
    Get, update, or delete a specific student.
    
    **Requires:** Authentication
    **Rate Limit:** 1000 requests/hour
    
  get:
    displayName: Get Student
    description: Retrieve student details by ID
    responses:
      200:
        description: Success
      404:
        description: Student not found
      500:
        description: Server error

# ❌ WRONG - No documentation
/students/{id}:
  get:
    responses:
      200:
      404:
```

---

## ⚠️ Common Issues & Solutions

### **Issue 1: Circular Type References**

```yaml
# ❌ PROBLEM
Student:
  properties:
    courses: Course[]

Course:
  properties:
    students: Student[]  # Circular!

# ✅ SOLUTION - Use nested IDs
Student:
  properties:
    courseIds: array
    items: string

Course:
  properties:
    studentIds: array
    items: string
```

### **Issue 2: Type Reuse Across Files**

```yaml
# library.raml
#%RAML 1.0 Library
types:
  Error:
    type: object
    properties:
      error: string
      message: string

# main-api.raml
#%RAML 1.0
uses:
  lib: library.raml

types:
  ErrorResponse:
    type: lib.Error

/students:
  post:
    responses:
      400:
        body:
          type: ErrorResponse
```

### **Issue 3: Optional Fields in Response**

```yaml
# ✅ Make field optional
Student:
  type: object
  properties:
    studentId: string
    name: string
    middleName:
      type: string
      required: false
    email: string
  required: [studentId, name, email]
```

### **Issue 4: Conditional Requirements**

```yaml
# ✅ Document conditional logic
Address:
  type: object
  properties:
    country: string
    state:
      type: string
      description: Required only if country is USA
    province:
      type: string
      description: Required only if country is Canada
```

### **Issue 5: Large Payload Handling**

```yaml
# ✅ Document streaming for large data
/reports:
  get:
    description: |
      Returns large report file.
      
      **Note:** This endpoint returns data in chunks.
      For files > 10MB, consider pagination or filtering.
    responses:
      200:
        headers:
          Content-Length:
            type: integer
          Transfer-Encoding:
            type: string
```

---

## 🎓 Summary - Advanced RAML

| Feature | Purpose | Example |
|---------|---------|---------|
| Resource Types | Reuse resource patterns | item, collection |
| Method Types | Reuse method patterns | paginatedGet |
| Traits | Reuse behaviors | paged, authenticated |
| Libraries | Share types/traits | common library |
| Annotations | Add metadata | @deprecated |
| External Types | Organize code | types/student.raml |
| Patterns | Common solutions | versioning, nested |

---

**Next:** Learn hands-on with complete projects in Part 3!
