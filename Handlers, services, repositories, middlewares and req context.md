# Backend Request Lifecycle

![Image](https://outcomeschool.com/static/images/blog/go-backend-arch-diagram.png)

![Image](https://miro.medium.com/1%2AKeV7gpYOYgGK8ph8Scua5w.jpeg)

This explains **what happens inside a backend server** when a request comes in and **where each piece of code lives**.

---

## High-Level Flow

> **Request → Middleware → Router → Handler → Service → Repository → Response**

---

## 1. Request Enters the Server

A client sends an HTTP request:

```http
POST /users
Content-Type: application/json
```

The Go HTTP server receives it.

---

## 2. Middleware (Runs First)

![Image](https://drstearns.github.io/tutorials/gomiddleware/img/flow.png)

![Image](https://media2.dev.to/dynamic/image/width%3D1000%2Cheight%3D420%2Cfit%3Dcover%2Cgravity%3Dauto%2Cformat%3Dauto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F18938d07sxa6bj64zr02.png)

Middleware runs **before the handler**.
It is used for logic that applies to **many routes**.

### Example: Logging Middleware (Go)

```go
func LoggingMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		log.Println("Incoming request:", r.Method, r.URL.Path)
		next.ServeHTTP(w, r)
	})
}
```

Used for:

* Logging
* Authentication
* Rate limiting
* CORS
* Error handling

Middleware can:

* Modify request
* Stop request early
* Pass it forward

---

## 3. Routing (Find the Correct Handler)

The router decides **which handler function** should run.

Example:

```go
router.HandleFunc("/users", CreateUserHandler).Methods("POST")
```

Routing is based on:

* HTTP method
* URL path

---

## 4. Handler / Controller (HTTP Layer)

![Image](https://media.geeksforgeeks.org/wp-content/uploads/20250705152348042640/Request-and-Response-Cycle.webp)

This is the **entry point of your business code**.

### Responsibilities of a Handler

* Read request data
* Deserialize JSON
* Validate input
* Transform data
* Return HTTP response
* Call the service

### Example: Handler in Go

```go
type CreateUserRequest struct {
	Email string `json:"email"`
	Age   int    `json:"age"`
}

func CreateUserHandler(w http.ResponseWriter, r *http.Request) {
	var req CreateUserRequest

	// Deserialize JSON
	if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
		http.Error(w, "invalid JSON", http.StatusBadRequest)
		return
	}

	// Validation
	if req.Email == "" {
		http.Error(w, "email is required", http.StatusBadRequest)
		return
	}

	// Transformation
	req.Email = strings.ToLower(req.Email)

	// Call service
	user, err := userService.CreateUser(r.Context(), req)
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
		return
	}

	json.NewEncoder(w).Encode(user)
}
```

### What handlers should NOT do

* Database queries
* Business rules
* Complex logic

---

## 5. Service Layer (Business Logic)

The service layer decides **what should happen**.

### Responsibilities

* Business rules
* Orchestration
* Calling repositories
* Calling other services

### Example: Service in Go

```go
type UserService struct {
	repo UserRepository
}

func (s *UserService) CreateUser(ctx context.Context, req CreateUserRequest) (*User, error) {
	if req.Age < 18 {
		return nil, errors.New("user must be at least 18")
	}

	return s.repo.SaveUser(ctx, req.Email, req.Age)
}
```

### Important rule

> **Services do not know about HTTP.**

No `http.ResponseWriter`, no status codes.

---

## 6. Repository Layer (Database Access)

The repository talks **only to the database**.

### Responsibilities

* Build queries
* Execute queries
* Return data

### Example: Repository in Go

```go
type UserRepository interface {
	SaveUser(ctx context.Context, email string, age int) (*User, error)
}

type PostgresUserRepo struct {
	db *sql.DB
}

func (r *PostgresUserRepo) SaveUser(ctx context.Context, email string, age int) (*User, error) {
	query := `INSERT INTO users (email, age) VALUES ($1, $2) RETURNING id`
	var id int
	err := r.db.QueryRowContext(ctx, query, email, age).Scan(&id)
	if err != nil {
		return nil, err
	}
	return &User{ID: id, Email: email, Age: age}, nil
}
```

### Important rule

> **Repository methods should do one thing only.**

---

## 7. Context (Passed Everywhere)

The `context.Context` flows through:

* Middleware
* Handler
* Service
* Repository

Used for:

* Timeouts
* Cancellation
* Request IDs
* Auth info

```go
ctx := r.Context()
```

---

## Full Flow (Mapped to Code)

```
HTTP Request
   ↓
Middleware (logging, auth)
   ↓
Router
   ↓
Handler (parse + validate)
   ↓
Service (business rules)
   ↓
Repository (DB)
   ↓
Service
   ↓
Handler
   ↓
HTTP Response
```

---

## Why This Structure Is Important

* Clean separation of concerns
* Easy testing
* Easy scaling
* Easier debugging
* Production-ready design

---

> **Handlers handle HTTP, services handle business logic, repositories handle data, and middleware handles cross-cutting concerns.**
