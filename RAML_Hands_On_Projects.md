# 🛠️ RAML Hands-On: 6 Complete Projects with Step-by-Step Instructions

> **Learn RAML by building real projects. Each project teaches specific concepts.**

---

## 📚 Project Index

1. **PROJECT 1:** Simple Student Management API (Basics)
2. **PROJECT 2:** E-commerce Products API (Advanced)
3. **PROJECT 3:** Social Media API (Complex Relationships)
4. **PROJECT 4:** Hospital Management API (Real-world)
5. **PROJECT 5:** Library Management System (Full-featured)
6. **PROJECT 6:** Multi-tenant SaaS API (Enterprise)

---

## 🎯 PROJECT 1: Simple Student Management API

### **What You'll Learn**
- Basic RAML structure
- Simple resources and methods
- Query parameters
- URI parameters
- Basic types and validation
- Error handling

### **API Overview**

This API manages students in a university system:
- List all students (with pagination)
- Get specific student
- Create new student
- Update student
- Delete student

### **Step 1: Create RAML File**

Create a file: `student-api-project1.raml`

```yaml
#%RAML 1.0
title: University Student Management API
version: v1
baseUri: https://api.university.edu/{version}
mediaType: application/json

# Define types
types:
  Student:
    type: object
    properties:
      studentId:
        type: string
        description: Unique student identifier (auto-generated)
      firstName:
        type: string
        minLength: 2
        maxLength: 50
      lastName:
        type: string
        minLength: 2
        maxLength: 50
      email:
        type: string
        pattern: ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$
      age:
        type: integer
        minimum: 18
        maximum: 65
      gpa:
        type: number
        minimum: 0
        maximum: 4.0
        multipleOf: 0.01
      enrollmentStatus:
        type: string
        enum: [active, inactive, graduated, suspended]
        default: active
      enrollmentDate:
        type: date-only
      department:
        type: string
        enum: [Computer Science, Engineering, Business, Medicine, Arts]
    required: [studentId, firstName, lastName, email, age, enrollmentStatus]

  StudentInput:
    type: object
    properties:
      firstName:
        type: string
        minLength: 2
        maxLength: 50
      lastName:
        type: string
        minLength: 2
        maxLength: 50
      email:
        type: string
        pattern: ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$
      age:
        type: integer
        minimum: 18
        maximum: 65
      gpa:
        type: number
        minimum: 0
        maximum: 4.0
      department:
        type: string
        enum: [Computer Science, Engineering, Business, Medicine, Arts]
    required: [firstName, lastName, email, age, department]

  Error:
    type: object
    properties:
      error:
        type: string
        enum: [BAD_REQUEST, NOT_FOUND, CONFLICT, INTERNAL_ERROR]
      message:
        type: string
      timestamp:
        type: datetime
      path:
        type: string

  StudentList:
    type: object
    properties:
      students:
        type: Student[]
      pagination:
        type: object
        properties:
          page:
            type: integer
          limit:
            type: integer
          total:
            type: integer

# Define traits
traits:
  pageable:
    displayName: Pageable
    queryParameters:
      page:
        type: integer
        description: Page number (starts at 1)
        default: 1
        minimum: 1
      limit:
        type: integer
        description: Records per page
        default: 20
        minimum: 1
        maximum: 100

  searchable:
    displayName: Searchable
    queryParameters:
      search:
        type: string
        description: Search by firstName or lastName
        required: false

  sortable:
    displayName: Sortable
    queryParameters:
      sort:
        type: string
        description: Sort field (prefix - for descending)
        enum: [firstName, -firstName, lastName, -lastName, gpa, -gpa, enrollmentDate, -enrollmentDate]
        required: false

# Define resources
/students:
  displayName: Students Collection
  description: Manage student records

  get:
    displayName: List Students
    description: Get paginated list of all students with optional filtering
    is: [pageable, searchable, sortable]
    queryParameters:
      department:
        type: string
        enum: [Computer Science, Engineering, Business, Medicine, Arts]
        description: Filter by department
        required: false
      status:
        type: string
        enum: [active, inactive, graduated, suspended]
        description: Filter by enrollment status
        required: false
    responses:
      200:
        description: Successfully retrieved students
        body:
          type: StudentList
          example: |
            {
              "students": [
                {
                  "studentId": "STU001",
                  "firstName": "John",
                  "lastName": "Doe",
                  "email": "john@university.edu",
                  "age": 22,
                  "gpa": 3.8,
                  "enrollmentStatus": "active",
                  "enrollmentDate": "2022-09-01",
                  "department": "Computer Science"
                },
                {
                  "studentId": "STU002",
                  "firstName": "Jane",
                  "lastName": "Smith",
                  "email": "jane@university.edu",
                  "age": 21,
                  "gpa": 3.9,
                  "enrollmentStatus": "active",
                  "enrollmentDate": "2022-09-01",
                  "department": "Engineering"
                }
              ],
              "pagination": {
                "page": 1,
                "limit": 20,
                "total": 150
              }
            }
      400:
        description: Invalid query parameters
        body:
          type: Error
      500:
        description: Server error
        body:
          type: Error

  post:
    displayName: Create Student
    description: Create a new student record
    body:
      type: StudentInput
      example: |
        {
          "firstName": "Alice",
          "lastName": "Johnson",
          "email": "alice@university.edu",
          "age": 19,
          "gpa": 3.5,
          "department": "Business"
        }
    responses:
      201:
        description: Student created successfully
        body:
          type: Student
          example: |
            {
              "studentId": "STU003",
              "firstName": "Alice",
              "lastName": "Johnson",
              "email": "alice@university.edu",
              "age": 19,
              "gpa": 3.5,
              "enrollmentStatus": "active",
              "enrollmentDate": "2024-03-31",
              "department": "Business"
            }
      400:
        description: Invalid input data
        body:
          type: Error
      409:
        description: Student with this email already exists
        body:
          type: Error
      500:
        description: Server error
        body:
          type: Error

  /{studentId}:
    displayName: Student Detail
    description: Get, update, or delete a specific student
    uriParameters:
      studentId:
        type: string
        description: The unique student identifier (e.g., STU001)

    get:
      displayName: Get Student
      description: Retrieve a specific student by ID
      responses:
        200:
          description: Student found
          body:
            type: Student
        404:
          description: Student not found
          body:
            type: Error
            example: |
              {
                "error": "NOT_FOUND",
                "message": "Student with ID STU999 not found",
                "timestamp": "2024-03-31T15:30:00Z",
                "path": "/students/STU999"
              }
        500:
          description: Server error
          body:
            type: Error

    put:
      displayName: Update Student
      description: Update complete student record (all fields required)
      body:
        type: StudentInput
        example: |
          {
            "firstName": "John",
            "lastName": "Updated",
            "email": "john.updated@university.edu",
            "age": 23,
            "gpa": 3.9,
            "department": "Computer Science"
          }
      responses:
        200:
          description: Student updated successfully
          body:
            type: Student
        400:
          description: Invalid input data
          body:
            type: Error
        404:
          description: Student not found
          body:
            type: Error
        409:
          description: Email already in use by another student
          body:
            type: Error
        500:
          description: Server error
          body:
            type: Error

    patch:
      displayName: Partial Update Student
      description: Update specific fields of student (only send changed fields)
      body:
        type: object
        properties:
          firstName:
            type: string
          lastName:
            type: string
          email:
            type: string
          gpa:
            type: number
          enrollmentStatus:
            type: string
          department:
            type: string
      responses:
        200:
          description: Student partially updated
          body:
            type: Student
        400:
          description: Invalid input data
          body:
            type: Error
        404:
          description: Student not found
          body:
            type: Error
        500:
          description: Server error
          body:
            type: Error

    delete:
      displayName: Delete Student
      description: Permanently delete a student record
      responses:
        204:
          description: Student deleted successfully
        404:
          description: Student not found
          body:
            type: Error
        500:
          description: Server error
          body:
            type: Error
```

### **Step 2: Test with Examples**

**Example 1: List students (page 1, 20 per page)**
```
GET /v1/students?page=1&limit=20
Response: 200
{
  "students": [...],
  "pagination": {"page": 1, "limit": 20, "total": 150}
}
```

**Example 2: Create student**
```
POST /v1/students
Body: {
  "firstName": "Alice",
  "lastName": "Johnson",
  "email": "alice@university.edu",
  "age": 19,
  "gpa": 3.5,
  "department": "Business"
}
Response: 201 Created
```

**Example 3: Get specific student**
```
GET /v1/students/STU001
Response: 200
{Student details}
```

### **Key Concepts Learned**
✅ Basic RAML structure with title, version, baseUri
✅ Type definitions (Student, StudentInput, Error)
✅ Query parameters (pagination, filtering, sorting)
✅ URI parameters ({studentId})
✅ HTTP methods (GET, POST, PUT, PATCH, DELETE)
✅ Proper status codes (200, 201, 204, 400, 404, 500)
✅ Request and response bodies with examples

---

## 🏪 PROJECT 2: E-Commerce Products API

### **What You'll Learn**
- Resource relationships
- Nested resources
- Multiple content types
- Rate limiting
- Authentication headers
- Advanced filtering

### **API Overview**

```
/products                          # List all products
/products/{productId}              # Specific product
/products/{productId}/reviews      # Product reviews
/products/{productId}/inventory    # Stock information
/categories                        # Product categories
/orders                           # Customer orders
/orders/{orderId}/items           # Order line items
```

### **RAML File: ecommerce-api-project2.raml**

```yaml
#%RAML 1.0
title: E-Commerce API
version: v1
baseUri: https://api.ecommerce.com/{version}
mediaType: application/json

types:
  Product:
    type: object
    properties:
      productId: string
      name: string
      description: string
      price:
        type: number
        minimum: 0
      currency:
        type: string
        default: USD
      category: string
      rating:
        type: number
        minimum: 0
        maximum: 5
      inStock: boolean
      createdDate: datetime
    required: [productId, name, price, category]

  Review:
    type: object
    properties:
      reviewId: string
      productId: string
      customerId: string
      rating:
        type: integer
        minimum: 1
        maximum: 5
      comment: string
      createdDate: datetime
    required: [reviewId, productId, customerId, rating]

  Inventory:
    type: object
    properties:
      productId: string
      quantity: integer
      warehouse: string
      lastUpdated: datetime

  Order:
    type: object
    properties:
      orderId: string
      customerId: string
      items: OrderItem[]
      total: number
      status:
        type: string
        enum: [pending, confirmed, shipped, delivered, cancelled]
      createdDate: datetime

  OrderItem:
    type: object
    properties:
      productId: string
      quantity: integer
      price: number

traits:
  authenticated:
    displayName: Requires Authentication
    headers:
      Authorization:
        type: string
        description: Bearer token
        required: true
        pattern: ^Bearer\s[a-zA-Z0-9\-._~\+\/]+=*$

  rateLimited:
    headers:
      X-RateLimit-Limit:
        type: integer
        description: Requests per hour
      X-RateLimit-Remaining:
        type: integer
        description: Requests remaining

  pageable:
    queryParameters:
      page:
        type: integer
        default: 1
      limit:
        type: integer
        default: 20

/products:
  displayName: Products
  get:
    is: [pageable, rateLimited]
    queryParameters:
      category:
        type: string
        required: false
      minPrice:
        type: number
        required: false
      maxPrice:
        type: number
        required: false
      inStock:
        type: boolean
        required: false
      sort:
        type: string
        enum: [price, -price, rating, -rating, name, -name]
        required: false
    responses:
      200:
        body:
          type: Product[]

  post:
    is: [authenticated]
    body:
      type: Product
    responses:
      201:
        body:
          type: Product
      400:
        description: Invalid product data

  /{productId}:
    uriParameters:
      productId: string
    
    get:
      is: [rateLimited]
      responses:
        200:
          body:
            type: Product
        404:
          description: Product not found

    /reviews:
      displayName: Product Reviews
      get:
        is: [pageable]
        responses:
          200:
            body:
              type: Review[]
      
      post:
        is: [authenticated]
        body:
          type: Review
        responses:
          201:
            body:
              type: Review

    /inventory:
      displayName: Product Inventory
      get:
        is: [authenticated]
        responses:
          200:
            body:
              type: Inventory[]

/orders:
  displayName: Orders
  is: [authenticated]
  
  get:
    is: [pageable]
    responses:
      200:
        body:
          type: Order[]
  
  post:
    body:
      type: Order
    responses:
      201:
        body:
          type: Order
  
  /{orderId}:
    uriParameters:
      orderId: string
    
    get:
      responses:
        200:
          body:
            type: Order
    
    /items:
      get:
        responses:
          200:
            body:
              type: OrderItem[]
```

### **Key Concepts Learned**
✅ Nested resources (/products/{id}/reviews)
✅ Traits for reusable behaviors (authenticated, rateLimited)
✅ Headers for rate limiting
✅ Complex filtering (price ranges, categories)
✅ Sorting with descending order

---

## 🤝 PROJECT 3: Social Media API

### **What You'll Learn**
- Resource inheritance
- Polymorphic types
- Batch operations
- WebSocket events (documentation)
- Complex relationships

### **Simplified RAML Structure**

```yaml
#%RAML 1.0
title: Social Media API
version: v1
baseUri: https://api.social.com/{version}

types:
  User:
    type: object
    properties:
      userId: string
      username: string
      email: string
      bio: string
      followers: integer
      following: integer

  Post:
    type: object
    properties:
      postId: string
      authorId: string
      content: string
      timestamp: datetime
      likes: integer
      comments: integer

  Comment:
    type: object
    properties:
      commentId: string
      postId: string
      authorId: string
      content: string
      timestamp: datetime

  Like:
    type: object
    properties:
      likeId: string
      postId: string
      userId: string
      timestamp: datetime

/users:
  /{userId}:
    get:
      responses:
        200:
          body:
            type: User
    
    /posts:
      get:
        responses:
          200:
            body:
              type: Post[]
    
    /followers:
      get:
        responses:
          200:
            body:
              type: User[]
    
    /following:
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
    get:
      responses:
        200:
          body:
            type: Post
    
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
    
    /likes:
      get:
        responses:
          200:
            body:
              type: Like[]
      
      post:
        body:
          type: Like
        responses:
          201:
            body:
              type: Like
```

### **Key Concepts Learned**
✅ Deep nesting (/posts/{id}/comments)
✅ Multiple related resources
✅ Relationship handling

---

## 🏥 PROJECT 4: Hospital Management API

### **What You'll Learn**
- Real-world complexity
- Role-based access
- Multiple entity types
- Status workflows
- Audit trails

### **Simplified Structure**

```yaml
types:
  Doctor:
    properties:
      doctorId: string
      name: string
      specialization: string
      available: boolean

  Patient:
    properties:
      patientId: string
      name: string
      age: integer
      medicalHistory: string[]

  Appointment:
    properties:
      appointmentId: string
      doctorId: string
      patientId: string
      dateTime: datetime
      status:
        enum: [scheduled, completed, cancelled]
      diagnosis: string
      prescription: string

  Prescription:
    properties:
      prescriptionId: string
      patientId: string
      doctorId: string
      medicines: Medicine[]
      createdDate: datetime

  Medicine:
    properties:
      medicineId: string
      name: string
      dosage: string
      frequency: string
      duration: string

traits:
  doctorOnly:
    description: Requires doctor role
    headers:
      X-User-Role:
        type: string
        enum: [DOCTOR]

  patientOnly:
    description: Requires patient role
    headers:
      X-User-Role:
        type: string
        enum: [PATIENT]

/doctors:
  get:
    responses:
      200:
        body:
          type: Doctor[]
  
  /{doctorId}:
    /appointments:
      get:
        is: [doctorOnly]
        responses:
          200:
            body:
              type: Appointment[]

/patients:
  /{patientId}:
    get:
      responses:
        200:
          body:
            type: Patient
    
    /appointments:
      get:
        is: [patientOnly]
        responses:
          200:
            body:
              type: Appointment[]
      
      post:
        is: [patientOnly]
        body:
          type: Appointment
        responses:
          201:
            body:
              type: Appointment
    
    /prescriptions:
      get:
        is: [patientOnly]
        responses:
          200:
            body:
              type: Prescription[]

/appointments:
  get:
    responses:
      200:
        body:
          type: Appointment[]
  
  post:
    body:
      type: Appointment
    responses:
      201:
        body:
          type: Appointment
  
  /{appointmentId}:
    put:
      is: [doctorOnly]
      body:
        type: Appointment
      responses:
        200:
          body:
            type: Appointment
```

---

## 📚 PROJECT 5: Library Management System

### **What You'll Learn**
- Inventory management
- Lending/return workflows
- Fine calculation
- Search and filtering

### **Key Resources**

```
/books                    # Catalog
/books/{bookId}/copies    # Physical copies
/members                  # Library members
/loans                    # Borrowing records
/reservations             # Book reservations
/fines                    # Late fee tracking
```

---

## 🏢 PROJECT 6: Multi-Tenant SaaS API

### **What You'll Learn**
- Tenant isolation
- Resource scoping
- API versioning
- Rate limiting per tier
- Feature flags

### **Key Concepts**

```yaml
# Every resource scoped to tenant
/tenants/{tenantId}/users
/tenants/{tenantId}/projects
/tenants/{tenantId}/data
/tenants/{tenantId}/analytics

# Rate limits vary by subscription
queryParameters:
  subscriptionTier:
    enum: [free, pro, enterprise]
    # Free: 100 req/hour
    # Pro: 1000 req/hour
    # Enterprise: Unlimited

# Features availability by tier
responses:
  403:
    description: Feature not available in your subscription tier
```

---

## 📊 Project Comparison Table

| Project | Complexity | Key Focus | Resources | Endpoints |
|---------|-----------|-----------|-----------|-----------|
| Project 1 | Basic | Fundamentals | Students | 5 |
| Project 2 | Intermediate | E-commerce | Products, Orders, Reviews | 8 |
| Project 3 | Intermediate | Social | Posts, Users, Comments | 10 |
| Project 4 | Advanced | Real-world | Hospital data | 12 |
| Project 5 | Advanced | Workflows | Library system | 15 |
| Project 6 | Expert | Enterprise | Multi-tenant | 20+ |

---

## 🎯 How to Use These Projects

### **Learning Path**

```
1. Start with Project 1
   - Read RAML file
   - Understand structure
   - Write similar API (Student Management v2)

2. Move to Project 2
   - Study nested resources
   - Understand traits
   - Add reviews to your Project 1 API

3. Tackle Project 3
   - Learn complex relationships
   - Practice filtering
   - Add comments/likes to your API

4. Study Project 4
   - Real-world patterns
   - Role-based access
   - Status workflows

5. Build Project 5
   - Combine all concepts
   - Build from scratch
   - Create Library API

6. Master Project 6
   - Multi-tenancy
   - Advanced security
   - Enterprise patterns
```

### **Practice Exercises**

**For Each Project:**
1. ✅ Read and understand the RAML
2. ✅ Modify it (change field names, add new fields)
3. ✅ Add new endpoints (think about what's missing)
4. ✅ Add new error cases
5. ✅ Add rate limiting or authentication
6. ✅ Create mock server from RAML
7. ✅ Test endpoints

---

**Next:** Move to RAML_100_Practice_Problems.md for theory practice!
