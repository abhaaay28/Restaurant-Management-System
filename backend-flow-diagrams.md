# Restaurant Management System - Backend Flow Diagrams

## 1. Overall System Architecture Flow

```mermaid
graph TB
    subgraph "Frontend (React)"
        A[React App] --> B[API Calls]
    end
    
    subgraph "Backend (Spring Boot)"
        B --> C[Security Layer]
        C --> D[Controller Layer]
        D --> E[Service Layer]
        E --> F[Repository Layer]
        F --> G[Database Layer]
    end
    
    subgraph "External Services"
        G --> H[MySQL Database]
        E --> I[Stripe Payment]
        E --> J[Email Service]
    end
    
    A -.->|HTTP Requests| C
    H --> F
    F --> E
    E --> D
    D --> C
    C --> A
```

## 2. Authentication & Authorization Flow

```mermaid
sequenceDiagram
    participant Client
    participant Security
    participant JWTValidator
    participant AuthController
    participant UserService
    participant Database
    
    Client->>Security: HTTP Request with JWT
    Security->>JWTValidator: Validate Token
    JWTValidator->>Security: Token Valid/Invalid
    
    alt Token Valid
        Security->>AuthController: Forward Request
        AuthController->>UserService: Process Business Logic
        UserService->>Database: Query/Update Data
        Database-->>UserService: Return Data
        UserService-->>AuthController: Return Result
        AuthController-->>Client: HTTP Response
    else Token Invalid
        Security-->>Client: 401 Unauthorized
    end
```

## 3. User Registration Flow

```mermaid
flowchart TD
    A[User Registration Request] --> B{Email Exists?}
    B -->|Yes| C[Return Error: Email Already Used]
    B -->|No| D[Create New User]
    D --> E[Encode Password with BCrypt]
    E --> F[Set User Role]
    F --> G[Save User to Database]
    G --> H[Create User Cart]
    H --> I[Generate JWT Token]
    I --> J[Return Auth Response]
    
    style A fill:#e1f5fe
    style J fill:#c8e6c9
    style C fill:#ffcdd2
```

## 4. User Login Flow

```mermaid
flowchart TD
    A[Login Request] --> B[Load User by Email]
    B --> C{User Found?}
    C -->|No| D[Throw BadCredentialsException]
    C -->|Yes| E[Verify Password with BCrypt]
    E --> F{Password Match?}
    F -->|No| D
    F -->|Yes| G[Create Authentication Token]
    G --> H[Set Security Context]
    H --> I[Generate JWT Token]
    I --> J[Return Auth Response with Role]
    
    style A fill:#e1f5fe
    style J fill:#c8e6c9
    style D fill:#ffcdd2
```

## 5. Restaurant Management Flow

```mermaid
flowchart TD
    A[Restaurant Request] --> B{User Authenticated?}
    B -->|No| C[Return 401 Unauthorized]
    B -->|Yes| D{User Role Valid?}
    D -->|No| E[Return 403 Forbidden]
    D -->|Yes| F[Process Restaurant Operation]
    
    F --> G{Operation Type?}
    G -->|Create| H[Validate Restaurant Data]
    G -->|Update| I[Check Restaurant Ownership]
    G -->|Delete| J[Verify Permissions]
    G -->|Status Update| K[Toggle Restaurant Status]
    
    H --> L[Save to Database]
    I --> M[Update Database]
    J --> N[Remove from Database]
    K --> O[Update Status]
    
    L --> P[Return Restaurant Object]
    M --> P
    N --> Q[Return Success Message]
    O --> P
    
    style A fill:#e1f5fe
    style P fill:#c8e6c9
    style Q fill:#c8e6c9
    style C fill:#ffcdd2
    style E fill:#ffcdd2
```

## 6. Food Ordering Flow

```mermaid
flowchart TD
    A[Customer Browse] --> B[View Restaurants]
    B --> C[Select Restaurant]
    C --> D[View Menu Items]
    D --> E[Add Items to Cart]
    E --> F[Review Cart]
    F --> G[Proceed to Checkout]
    G --> H[Enter Delivery Address]
    H --> I[Payment Processing]
    I --> J{Payment Success?}
    J -->|Yes| K[Create Order]
    J -->|No| L[Return to Cart]
    K --> M[Update Inventory]
    M --> N[Send Confirmation Email]
    N --> O[Order Confirmed]
    
    style A fill:#e1f5fe
    style O fill:#c8e6c9
    style L fill:#ffcdd2
```

## 7. Cart Management Flow

```mermaid
flowchart TD
    A[Add Item to Cart] --> B{Item Already in Cart?}
    B -->|Yes| C[Update Quantity]
    B -->|No| D[Create New Cart Item]
    C --> E[Recalculate Total]
    D --> E
    E --> F[Save Cart to Database]
    F --> G[Return Updated Cart]
    
    H[Remove Item] --> I[Delete Cart Item]
    I --> J[Update Cart Total]
    J --> K[Save Changes]
    
    L[Clear Cart] --> M[Remove All Items]
    M --> N[Reset Cart Total]
    N --> O[Save Empty Cart]
    
    style A fill:#e1f5fe
    style H fill:#e1f5fe
    style L fill:#e1f5fe
    style G fill:#c8e6c9
    style K fill:#c8e6c9
    style O fill:#c8e6c9
```

## 8. Order Processing Flow

```mermaid
stateDiagram-v2
    [*] --> Pending
    Pending --> Confirmed : Payment Success
    Pending --> Cancelled : Payment Failed
    Confirmed --> Preparing : Kitchen Starts
    Preparing --> Ready : Food Prepared
    Ready --> OutForDelivery : Driver Assigned
    OutForDelivery --> Delivered : Delivery Complete
    Delivered --> [*]
    
    Cancelled --> [*]
    
    note right of Pending : Customer places order
    note right of Confirmed : Payment processed
    note right of Preparing : Restaurant starts cooking
    note right of Ready : Food ready for pickup
    note right of OutForDelivery : Driver on the way
    note right of Delivered : Order completed
```

## 9. Payment Integration Flow

```mermaid
sequenceDiagram
    participant Customer
    participant Frontend
    participant Backend
    participant Stripe
    participant Database
    
    Customer->>Frontend: Proceed to Payment
    Frontend->>Backend: Create Payment Intent
    Backend->>Stripe: Create Payment Intent
    Stripe-->>Backend: Payment Intent Created
    Backend-->>Frontend: Payment Intent ID
    Frontend->>Stripe: Confirm Payment (Card Details)
    Stripe->>Stripe: Process Payment
    Stripe-->>Frontend: Payment Result
    Frontend->>Backend: Payment Success/Failure
    Backend->>Database: Update Order Status
    Backend-->>Frontend: Order Confirmation
    Frontend-->>Customer: Order Status
```

## 10. Exception Handling Flow

```mermaid
flowchart TD
    A[Request Processing] --> B{Exception Occurs?}
    B -->|No| C[Continue Processing]
    B -->|Yes| D{Exception Type?}
    
    D -->|UserException| E[Handle User Errors]
    D -->|RestaurantException| F[Handle Restaurant Errors]
    D -->|CartException| G[Handle Cart Errors]
    D -->|FoodException| H[Handle Food Errors]
    D -->|SystemException| I[Handle System Errors]
    
    E --> J[Return User-Friendly Message]
    F --> K[Return Restaurant Error Details]
    G --> L[Return Cart Error Details]
    H --> M[Return Food Error Details]
    I --> N[Return Generic Error]
    
    J --> O[Log Error]
    K --> O
    L --> O
    M --> O
    N --> O
    
    O --> P[Send Error Response]
    
    style A fill:#e1f5fe
    style C fill:#c8e6c9
    style P fill:#ffcdd2
```

## 11. Database Entity Relationships

```mermaid
erDiagram
    USER ||--o{ ORDER : places
    USER ||--o{ ADDRESS : has
    USER ||--o{ CART : owns
    USER ||--o{ FAVORITE : has
    
    RESTAURANT ||--o{ FOOD : serves
    RESTAURANT ||--o{ ORDER : receives
    RESTAURANT ||--|| USER : owned_by
    
    FOOD ||--o{ CART_ITEM : added_to
    FOOD ||--o{ INGREDIENT : contains
    
    CART ||--o{ CART_ITEM : contains
    CART ||--|| USER : belongs_to
    
    ORDER ||--o{ ORDER_ITEM : contains
    ORDER ||--|| USER : placed_by
    ORDER ||--|| RESTAURANT : from
    
    USER {
        Long id PK
        String fullName
        String email
        String password
        USER_ROLE role
        String status
    }
    
    RESTAURANT {
        Long id PK
        String name
        String description
        String cuisineType
        String address
        String phone
        String email
        Boolean open
        Long ownerId FK
    }
    
    FOOD {
        Long id PK
        String name
        String description
        Double price
        String category
        Boolean available
        Long restaurantId FK
    }
    
    ORDER {
        Long id PK
        String orderStatus
        Double totalAmount
        Long customerId FK
        Long restaurantId FK
        Date createdAt
    }
```

## 12. API Endpoint Structure

```mermaid
graph TD
    A[API Root /api] --> B[Auth /auth]
    A --> C[Customer APIs]
    A --> D[Restaurant APIs]
    A --> E[Admin APIs]
    
    B --> B1[POST /signup]
    B --> B2[POST /signin]
    B --> B3[POST /reset-password]
    B --> B4[POST /reset-password-request]
    
    C --> C1[GET /restaurants]
    C --> C2[GET /restaurants/{id}]
    C --> C3[POST /cart/add]
    C --> C4[GET /cart]
    C --> C5[POST /orders]
    C --> C6[GET /orders]
    
    D --> D1[POST /admin/restaurants]
    D --> D2[PUT /admin/restaurants/{id}]
    D --> D3[DELETE /admin/restaurants/{id}]
    D --> D4[POST /admin/food]
    D --> D5[PUT /admin/food/{id}]
    
    E --> E1[GET /admin/users]
    E --> E2[PUT /admin/users/{id}]
    E --> E3[GET /admin/analytics]
    E --> E4[POST /admin/categories]
    
    style A fill:#e3f2fd
    style B fill:#f3e5f5
    style C fill:#e8f5e8
    style D fill:#fff3e0
    style E fill:#fce4ec
```

## 13. Security Filter Chain

```mermaid
flowchart TD
    A[HTTP Request] --> B[CORS Filter]
    B --> C[JWT Token Validator]
    C --> D{Token Valid?}
    D -->|No| E[Return 401 Unauthorized]
    D -->|Yes| F[Extract User Details]
    F --> G[Set Security Context]
    G --> H[Role-based Authorization]
    H --> I{User Has Required Role?}
    I -->|No| J[Return 403 Forbidden]
    I -->|Yes| K[Forward to Controller]
    
    style A fill:#e1f5fe
    style K fill:#c8e6c9
    style E fill:#ffcdd2
    style J fill:#ffcdd2
```

## 14. Service Layer Business Logic Flow

```mermaid
flowchart TD
    A[Service Method Call] --> B[Input Validation]
    B --> C{Validation Passed?}
    C -->|No| D[Throw Validation Exception]
    C -->|Yes| E[Business Logic Processing]
    
    E --> F{Complex Operation?}
    F -->|Yes| G[Transaction Management]
    F -->|No| H[Direct Processing]
    
    G --> I[Database Operations]
    H --> I
    
    I --> J{Operation Success?}
    J -->|No| K[Rollback Transaction]
    J -->|Yes| L[Commit Transaction]
    
    K --> M[Throw Service Exception]
    L --> N[Return Result]
    
    style A fill:#e1f5fe
    style N fill:#c8e6c9
    style D fill:#ffcdd2
    style M fill:#ffcdd2
```

## 15. Data Flow Between Layers

```mermaid
graph LR
    subgraph "Presentation Layer"
        A[Controllers]
    end
    
    subgraph "Business Layer"
        B[Services]
        C[DTOs]
    end
    
    subgraph "Data Layer"
        D[Repositories]
        E[Entities]
    end
    
    subgraph "Database"
        F[MySQL Tables]
    end
    
    A -->|Request DTOs| B
    B -->|Business Logic| C
    B -->|Entity Objects| D
    D -->|JPA Queries| E
    E -->|Hibernate| F
    
    F -->|Result Sets| E
    E -->|Entity Objects| D
    D -->|Entity Objects| B
    B -->|Response DTOs| C
    C -->|Response DTOs| A
    
    style A fill:#e3f2fd
    style B fill:#e8f5e8
    style C fill:#fff3e0
    style D fill:#f3e5f5
    style E fill:#fce4ec
    style F fill:#e0f2f1
```

---

## Key Features of the Flow Diagrams:

1. **Color Coding**: 
   - 🟦 Blue: Input/Start points
   - 🟩 Green: Success/Completion points
   - 🟥 Red: Error/Exception points
   - 🟨 Yellow: Processing/Intermediate points

2. **Flow Types**:
   - **Sequence Diagrams**: Show time-based interactions
   - **Flowcharts**: Show decision-based logic
   - **State Diagrams**: Show state transitions
   - **Entity-Relationship**: Show data structure
   - **Graphs**: Show hierarchical relationships

3. **Architecture Patterns**:
   - **Layered Architecture**: Clear separation of concerns
   - **MVC Pattern**: Model-View-Controller separation
   - **Repository Pattern**: Data access abstraction
   - **Service Layer**: Business logic encapsulation
   - **Security Filter Chain**: Authentication & authorization

These diagrams provide a comprehensive understanding of how data flows through the system, how different components interact, and how the overall architecture is structured.

