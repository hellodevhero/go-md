# Go Fundamentals — A Complete Self-Study Book
### Phase 1: From Zero to Confident Go Developer

---

> **How to use this book**
> Read each section, then type every code example yourself — don't copy-paste.
> Complete the exercise at the end of each week before moving on.
> Each chapter builds on the last. Don't skip.

---

## Table of Contents

- [Week 1 — Setup, Syntax & Basic Types](#week-1--setup-syntax--basic-types)
- [Week 2 — Structs, Methods & Pointers](#week-2--structs-methods--pointers)
- [Week 3 — Interfaces & Error Handling](#week-3--interfaces--error-handling)
- [Week 4 — Packages, Modules & Testing](#week-4--packages-modules--testing)
- [Week 5 — Goroutines & Channels](#week-5--goroutines--channels)
- [Week 6 — Consolidation & Capstone](#week-6--consolidation--capstone)

---

# Week 1 — Setup, Syntax & Basic Types

## 1.1 Installing Go & Your First Program

Download Go from https://go.dev/dl and install it. Then verify:

```bash
go version
# go version go1.22.0 linux/amd64
```

Set up VS Code with the official Go extension (`golang.go`). It gives you autocompletion, formatting on save, and inline error hints.

### Your First Program

Create a folder, initialize a module, and write hello world:

```bash
mkdir learn-go && cd learn-go
go mod init learn-go
```

Create `main.go`:

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}
```

Run it:

```bash
go run main.go
# Hello, World!
```

Build it into a binary:

```bash
go build -o myapp main.go
./myapp
```

**Key things to notice:**
- Every Go file starts with `package name`
- `package main` is special — it marks the entry point of a program
- `func main()` is where execution starts
- `import` brings in packages. `fmt` is the formatting/printing package

---

## 1.2 Variables & Types

Go is statically typed. Every variable has a type, and types don't convert automatically.

### Declaring Variables

```go
// Long form — explicit type
var name string = "Alice"
var age int = 30
var active bool = true

// Short form — type inferred (only inside functions)
name := "Alice"
age := 30
active := true

// Multiple variables
var x, y int = 10, 20
a, b := "hello", "world"

// Block declaration
var (
    host string = "localhost"
    port int    = 8080
)
```

**Rule:** Use `:=` inside functions (most of the time). Use `var` for package-level variables or when you want to declare without assigning.

### Zero Values

In Go, variables always have a value — even if you don't assign one. The default is the **zero value** of the type.

```go
var i int     // 0
var f float64 // 0.0
var b bool    // false
var s string  // ""  (empty string, not null)
var p *int    // nil
```

This is intentional. Go has no `undefined` or `null` for basic types. Embrace it.

### Basic Types

```go
// Integer types
var a int    = 42       // platform size (32 or 64 bit)
var b int8   = 127      // -128 to 127
var c int16  = 32767
var d int32  = 2147483647
var e int64  = 9223372036854775807
var u uint   = 42       // unsigned (no negatives)

// Float types
var f32 float32 = 3.14
var f64 float64 = 3.141592653589793  // prefer float64

// Boolean
var yes bool = true
var no  bool = false

// String
var greeting string = "Hello, Go!"

// Rune (a Unicode code point, alias for int32)
var letter rune = 'A'

// Byte (alias for uint8)
var b byte = 255
```

### Type Conversion

Go does NOT do implicit conversion. You must convert explicitly:

```go
var i int = 42
var f float64 = float64(i)   // must convert
var u uint = uint(f)

// This would NOT compile:
// var f float64 = i  // error: cannot use i (type int) as type float64
```

### Constants

```go
const Pi = 3.14159
const MaxUsers = 1000
const AppName = "MyApp"

// Typed constant
const TypedPi float64 = 3.14159

// Block of constants
const (
    StatusOK    = 200
    StatusError = 500
)
```

### iota — Auto-incrementing Constants

```go
type Direction int

const (
    North Direction = iota // 0
    East                   // 1
    South                  // 2
    West                   // 3
)

// More creative iota usage
const (
    _           = iota // skip 0
    KB = 1 << (10 * iota) // 1024
    MB                    // 1048576
    GB                    // 1073741824
)
```

---

## 1.3 Control Flow

### if / else

```go
age := 20

if age >= 18 {
    fmt.Println("adult")
} else if age >= 13 {
    fmt.Println("teenager")
} else {
    fmt.Println("child")
}
```

**Go's special `if` — init statement:**

```go
// The variable declared in the init statement is scoped to the if block
if n := 10; n > 5 {
    fmt.Println("n is greater than 5")
}
// n is not accessible here

// Very common pattern with errors:
if err := doSomething(); err != nil {
    fmt.Println("error:", err)
    return
}
```

### for — The Only Loop in Go

Go has no `while`. The `for` loop handles everything:

```go
// Classic C-style for
for i := 0; i < 5; i++ {
    fmt.Println(i)
}

// Like a while loop
n := 0
for n < 5 {
    n++
}

// Infinite loop
for {
    // runs forever
    break // use break to exit
}

// Range over a slice
fruits := []string{"apple", "banana", "cherry"}
for i, fruit := range fruits {
    fmt.Printf("%d: %s\n", i, fruit)
}

// Range — ignore index
for _, fruit := range fruits {
    fmt.Println(fruit)
}

// Range over a map
scores := map[string]int{"Alice": 90, "Bob": 85}
for name, score := range scores {
    fmt.Printf("%s: %d\n", name, score)
}

// Range over a string (gives runes, not bytes)
for i, ch := range "hello" {
    fmt.Printf("%d: %c\n", i, ch)
}
```

### switch

```go
day := "Monday"

switch day {
case "Monday", "Tuesday", "Wednesday", "Thursday", "Friday":
    fmt.Println("Weekday")
case "Saturday", "Sunday":
    fmt.Println("Weekend")
default:
    fmt.Println("Unknown")
}

// Switch with no expression (like if/else if chains)
score := 75
switch {
case score >= 90:
    fmt.Println("A")
case score >= 80:
    fmt.Println("B")
case score >= 70:
    fmt.Println("C")
default:
    fmt.Println("F")
}
```

**Note:** Unlike C, Go switch cases do NOT fall through by default. Use `fallthrough` keyword explicitly if you need it (rare).

### defer

`defer` schedules a function call to run just before the surrounding function returns. Deferred calls run in LIFO order (last in, first out).

```go
func readFile() {
    file, err := os.Open("data.txt")
    if err != nil {
        return
    }
    defer file.Close() // will run when readFile() exits, no matter what

    // work with file...
}

// Deferred calls run in LIFO
func counting() {
    for i := 0; i < 3; i++ {
        defer fmt.Println(i)
    }
    // Output: 2, 1, 0
}
```

**Common uses:** closing files, releasing locks, closing database connections.

---

## 1.4 Arrays, Slices & Maps

### Arrays

Arrays have a fixed size. Size is part of the type. Rarely used directly in Go.

```go
var arr [5]int          // [0 0 0 0 0]
arr[0] = 10

primes := [5]int{2, 3, 5, 7, 11}
auto := [...]int{1, 2, 3}  // compiler counts the size

fmt.Println(len(primes))   // 5

// Arrays are value types — copying creates a full copy
a := [3]int{1, 2, 3}
b := a   // b is a separate copy
b[0] = 99
fmt.Println(a[0]) // still 1
```

### Slices

Slices are the workhorse of Go. A slice is a view into an underlying array — it has a length and a capacity.

```go
// Creating slices
s := []int{1, 2, 3, 4, 5}
empty := []int{}
withMake := make([]int, 5)       // length 5, cap 5, all zeros
withCap  := make([]int, 3, 10)   // length 3, cap 10

// Accessing and modifying
fmt.Println(s[0])  // 1
s[0] = 99

// Slicing a slice [low:high] — includes low, excludes high
fmt.Println(s[1:3])  // [2 3]
fmt.Println(s[:2])   // [1 2]  (from beginning)
fmt.Println(s[2:])   // [3 4 5] (to end)

// Appending — returns a new slice (may reallocate)
s = append(s, 6, 7)
other := []int{8, 9}
s = append(s, other...)  // spread another slice

// Length vs Capacity
s2 := make([]int, 3, 5)
fmt.Println(len(s2), cap(s2))  // 3 5

// Copy
src := []int{1, 2, 3}
dst := make([]int, len(src))
copy(dst, src)
```

**The mental model:** A slice is a struct with three fields: a pointer to an array, a length, and a capacity. When you append beyond capacity, Go allocates a new larger array and copies the data.

```go
// Iterating slices
nums := []int{10, 20, 30}
for i, v := range nums {
    fmt.Printf("index %d, value %d\n", i, v)
}

// 2D slice
matrix := [][]int{
    {1, 2, 3},
    {4, 5, 6},
    {7, 8, 9},
}
fmt.Println(matrix[1][2]) // 6
```

### Maps

Maps are Go's built-in hash tables.

```go
// Creating maps
ages := map[string]int{
    "Alice": 30,
    "Bob":   25,
}

empty := make(map[string]int)

// Reading — missing key returns zero value (0 for int)
fmt.Println(ages["Alice"])   // 30
fmt.Println(ages["Charlie"]) // 0 — key doesn't exist, no panic

// Check if key exists — comma-ok idiom
val, ok := ages["Charlie"]
if !ok {
    fmt.Println("Charlie not found")
}

// Writing
ages["Charlie"] = 35
ages["Alice"] = 31   // update

// Deleting
delete(ages, "Bob")

// Iterating — order is random, don't rely on it
for name, age := range ages {
    fmt.Printf("%s is %d\n", name, age)
}

// Length
fmt.Println(len(ages))
```

---

## 1.5 Functions

Functions are first-class citizens in Go.

### Basic Functions

```go
// Simple function
func add(a int, b int) int {
    return a + b
}

// Shorthand when params share a type
func add(a, b int) int {
    return a + b
}

// No return value
func greet(name string) {
    fmt.Println("Hello,", name)
}
```

### Multiple Return Values

This is one of Go's most distinctive features.

```go
func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, fmt.Errorf("cannot divide by zero")
    }
    return a / b, nil
}

// Calling it
result, err := divide(10, 3)
if err != nil {
    fmt.Println("Error:", err)
    return
}
fmt.Println(result) // 3.3333...

// Ignore a return value with _
result, _ := divide(10, 3)
```

### Named Return Values

```go
func minMax(nums []int) (min, max int) {
    min, max = nums[0], nums[0]
    for _, n := range nums[1:] {
        if n < min { min = n }
        if n > max { max = n }
    }
    return // "naked" return — returns named values
}
```

Use sparingly. Named returns help in short functions but hurt readability in longer ones.

### Variadic Functions

```go
func sum(nums ...int) int {
    total := 0
    for _, n := range nums {
        total += n
    }
    return total
}

sum(1, 2, 3)        // 6
sum(1, 2, 3, 4, 5)  // 15

// Spread a slice
nums := []int{1, 2, 3}
sum(nums...)  // spread the slice
```

### Functions as Values

```go
// Assign a function to a variable
double := func(n int) int {
    return n * 2
}
fmt.Println(double(5)) // 10

// Pass a function as an argument
func apply(nums []int, fn func(int) int) []int {
    result := make([]int, len(nums))
    for i, n := range nums {
        result[i] = fn(n)
    }
    return result
}

doubled := apply([]int{1, 2, 3}, func(n int) int { return n * 2 })
// [2 4 6]
```

### Closures

A closure captures variables from its surrounding scope.

```go
func counter() func() int {
    count := 0
    return func() int {
        count++
        return count
    }
}

c := counter()
fmt.Println(c()) // 1
fmt.Println(c()) // 2
fmt.Println(c()) // 3

// Each call to counter() creates an independent counter
c2 := counter()
fmt.Println(c2()) // 1 — independent from c
```

---

## Week 1 — Project

**Build a CLI calculator**

Requirements:
1. Accept two numbers and an operator (+, -, *, /) as command-line arguments
2. Perform the calculation
3. Handle division by zero with an error message
4. Handle invalid operators
5. Print the result

```bash
go run main.go 10 + 5   # 15
go run main.go 10 / 0   # Error: cannot divide by zero
go run main.go 10 % 5   # Error: unknown operator: %
```

Hints:
- Use `os.Args` to read command-line arguments (`os.Args[0]` is the program name)
- Use `strconv.ParseFloat()` to convert strings to numbers
- Use a `switch` on the operator string

---

---

# Week 2 — Structs, Methods & Pointers

## 2.1 Structs

Structs are Go's way of grouping related data together. Go has no classes — structs plus methods fill that role.

```go
// Defining a struct
type Person struct {
    Name string
    Age  int
    Email string
}

// Creating a struct
p1 := Person{Name: "Alice", Age: 30, Email: "alice@example.com"}
p2 := Person{"Bob", 25, "bob@example.com"}  // positional (avoid this — fragile)

// Zero value struct
var p3 Person  // Name: "", Age: 0, Email: ""

// Accessing fields
fmt.Println(p1.Name)   // Alice
p1.Age = 31            // modify

// Struct as function argument
func greet(p Person) {
    fmt.Printf("Hello, %s!\n", p.Name)
}
```

### Struct Literals

```go
type Point struct {
    X, Y float64
}

origin := Point{X: 0, Y: 0}
p := Point{X: 3.5, Y: -1.2}

// Anonymous struct — useful for quick grouping
config := struct {
    Host string
    Port int
}{
    Host: "localhost",
    Port: 8080,
}
```

### Comparing Structs

Structs are comparable if all their fields are comparable types:

```go
a := Point{1, 2}
b := Point{1, 2}
fmt.Println(a == b)  // true

// Structs with slices/maps are NOT comparable
type Team struct {
    Name    string
    Members []string  // slice — not comparable
}
// t1 == t2  // this would not compile
```

### Embedded Structs (Composition)

Go favors composition over inheritance. You embed one struct inside another:

```go
type Animal struct {
    Name string
}

func (a Animal) Speak() string {
    return a.Name + " makes a sound"
}

type Dog struct {
    Animal          // embedded — not a named field
    Breed string
}

d := Dog{
    Animal: Animal{Name: "Rex"},
    Breed:  "Labrador",
}

fmt.Println(d.Name)     // promoted field — access directly
fmt.Println(d.Speak())  // promoted method — access directly
fmt.Println(d.Breed)
```

The embedded struct's fields and methods are **promoted** to the outer struct. This is not inheritance — it's promotion.

---

## 2.2 Pointers

A pointer holds the memory address of a value. Go uses pointers explicitly, unlike Java where object references are hidden.

```go
x := 42
p := &x        // p is a *int, holds the address of x
fmt.Println(p)  // something like 0xc00001e090
fmt.Println(*p) // 42 — dereference to get the value

*p = 100       // change the value through the pointer
fmt.Println(x) // 100 — x is now 100
```

### Why Pointers Matter

```go
// Without pointer — function gets a COPY
func doubleVal(n int) {
    n = n * 2  // only changes the local copy
}

// With pointer — function gets the real variable
func doublePtr(n *int) {
    *n = *n * 2  // changes the original
}

x := 5
doubleVal(x)
fmt.Println(x)  // still 5

doublePtr(&x)
fmt.Println(x)  // 10
```

### Pointers to Structs

```go
type User struct {
    Name  string
    Score int
}

u := User{Name: "Alice", Score: 100}
p := &u

// Go auto-dereferences struct pointer fields
p.Score = 200       // same as (*p).Score = 200
fmt.Println(u.Score) // 200

// new() allocates a zeroed struct and returns a pointer
u2 := new(User)
u2.Name = "Bob"
```

### nil Pointers

```go
var p *int  // nil — doesn't point to anything

// Dereferencing nil PANICS
// fmt.Println(*p)  // panic: runtime error: invalid memory address

// Always check before dereferencing
if p != nil {
    fmt.Println(*p)
}
```

---

## 2.3 Methods

A method is a function with a **receiver** — the struct (or type) it belongs to.

```go
type Rectangle struct {
    Width  float64
    Height float64
}

// Method with a VALUE receiver
func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

func (r Rectangle) Perimeter() float64 {
    return 2 * (r.Width + r.Height)
}

rect := Rectangle{Width: 10, Height: 5}
fmt.Println(rect.Area())       // 50
fmt.Println(rect.Perimeter())  // 30
```

### Value Receiver vs Pointer Receiver

This is one of the most important distinctions in Go:

```go
type Counter struct {
    count int
}

// VALUE receiver — receives a COPY, mutations don't stick
func (c Counter) IncrementBad() {
    c.count++  // only changes the copy
}

// POINTER receiver — receives the real struct, mutations stick
func (c *Counter) Increment() {
    c.count++  // changes the original
}

func (c Counter) Value() int {
    return c.count
}

c := Counter{}
c.IncrementBad()
fmt.Println(c.Value())  // 0 — mutation didn't stick

c.Increment()
fmt.Println(c.Value())  // 1 — mutation stuck
```

**Rule of thumb:**
- Use a **pointer receiver** when the method mutates state, or when the struct is large
- Use a **value receiver** for small read-only operations
- Be consistent: if any method on a type uses a pointer receiver, use pointer receivers for all methods on that type

### Methods on Non-Struct Types

You can define methods on any type you define — not just structs:

```go
type Celsius float64
type Fahrenheit float64

func (c Celsius) ToFahrenheit() Fahrenheit {
    return Fahrenheit(c*9/5 + 32)
}

func (f Fahrenheit) ToCelsius() Celsius {
    return Celsius((f - 32) * 5 / 9)
}

boiling := Celsius(100)
fmt.Println(boiling.ToFahrenheit())  // 212
```

### The Stringer Interface

Implement `String() string` to control how your type prints:

```go
type Point struct {
    X, Y int
}

func (p Point) String() string {
    return fmt.Sprintf("(%d, %d)", p.X, p.Y)
}

p := Point{3, 4}
fmt.Println(p)        // (3, 4) — uses our String()
fmt.Printf("%v\n", p) // (3, 4)
```

---

## Week 2 — Project

**Library book tracker**

Requirements:
1. Define a `Book` struct: `Title`, `Author`, `Year`, `Available bool`
2. Add methods: `Checkout() error` and `Return() error` (return errors if book state is wrong)
3. Define a `Library` struct holding a slice of `*Book`
4. Add `Library` methods: `AddBook()`, `FindBook(title string) (*Book, error)`, `ListBooks()`
5. Build a menu loop: list books, check out, return, quit

Example interaction:
```
1) List books
2) Checkout book
3) Return book
4) Quit
> 1
[Available] The Go Programming Language (Donovan, 2015)
[Checked Out] Clean Code (Martin, 2008)
```

---

---

# Week 3 — Interfaces & Error Handling

## 3.1 Interfaces

An interface defines a set of methods. Any type that has those methods **automatically** implements the interface — no explicit declaration needed.

```go
// Define an interface
type Shape interface {
    Area() float64
    Perimeter() float64
}

// These types implicitly implement Shape
type Circle struct {
    Radius float64
}

func (c Circle) Area() float64 {
    return 3.14159 * c.Radius * c.Radius
}

func (c Circle) Perimeter() float64 {
    return 2 * 3.14159 * c.Radius
}

type Rectangle struct {
    Width, Height float64
}

func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

func (r Rectangle) Perimeter() float64 {
    return 2 * (r.Width + r.Height)
}
```

Now any function that accepts a `Shape` works with both:

```go
func printShapeInfo(s Shape) {
    fmt.Printf("Area: %.2f, Perimeter: %.2f\n", s.Area(), s.Perimeter())
}

printShapeInfo(Circle{Radius: 5})
printShapeInfo(Rectangle{Width: 4, Height: 6})
```

### Interface Values

An interface value has two components: a concrete type and a value of that type.

```go
var s Shape
fmt.Println(s)        // <nil>
fmt.Println(s == nil) // true

s = Circle{Radius: 3}
fmt.Println(s)         // {3}  (uses Stringer if defined)
fmt.Println(s == nil)  // false
```

### Small Interfaces Are Better

Go's standard library uses tiny interfaces. The most famous:

```go
// io.Reader — just one method
type Reader interface {
    Read(p []byte) (n int, err error)
}

// io.Writer — just one method
type Writer interface {
    Write(p []byte) (n int, err error)
}

// io.ReadWriter — composed from both
type ReadWriter interface {
    Reader
    Writer
}
```

Prefer 1–2 method interfaces. The more methods, the fewer types implement it.

### The Golden Rule

> **Accept interfaces, return structs**

```go
// BAD — accepting a concrete type limits who can call this
func SaveUser(db *PostgresDB, user User) error { ... }

// GOOD — accepting an interface lets you test with a fake DB
type Database interface {
    Save(user User) error
}
func SaveUser(db Database, user User) error { ... }

// Return structs, not interfaces (callers can always use interface)
func NewPostgresDB(url string) *PostgresDB { ... }  // return concrete type
```

---

## 3.2 Type Assertions & Type Switches

### Type Assertions

Extract the concrete type from an interface value:

```go
var s Shape = Circle{Radius: 5}

// Assert — panics if wrong type
c := s.(Circle)
fmt.Println(c.Radius)  // 5

// Safe assert — comma-ok idiom
c, ok := s.(Circle)
if ok {
    fmt.Println("It's a circle with radius", c.Radius)
} else {
    fmt.Println("Not a circle")
}
```

### Type Switch

Handle multiple types cleanly:

```go
func describe(i interface{}) {
    switch v := i.(type) {
    case int:
        fmt.Printf("int: %d\n", v)
    case string:
        fmt.Printf("string: %q\n", v)
    case bool:
        fmt.Printf("bool: %v\n", v)
    case Circle:
        fmt.Printf("circle with radius %.2f\n", v.Radius)
    default:
        fmt.Printf("unknown type: %T\n", v)
    }
}

describe(42)
describe("hello")
describe(Circle{5})
```

### The `any` / `interface{}` Type

`any` (alias for `interface{}`) holds any value. Use it sparingly — it sacrifices type safety.

```go
var val any = 42
val = "now a string"
val = []int{1, 2, 3}

// Before using, you must type-assert
if s, ok := val.(string); ok {
    fmt.Println("string:", s)
}
```

---

## 3.3 Error Handling

In Go, errors are values — just a type that implements the `error` interface:

```go
type error interface {
    Error() string
}
```

### The Basic Pattern

```go
func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}

result, err := divide(10, 0)
if err != nil {
    fmt.Println("Error:", err)
    return
}
fmt.Println(result)
```

This pattern appears everywhere. Get comfortable with it immediately.

### Creating Errors

```go
import (
    "errors"
    "fmt"
)

// Simple error
err1 := errors.New("something went wrong")

// Formatted error
name := "config.json"
err2 := fmt.Errorf("file not found: %s", name)

// Wrapping an error — preserves the original for inspection
original := errors.New("connection refused")
wrapped := fmt.Errorf("failed to connect to database: %w", original)
```

### Wrapping & Unwrapping Errors

Use `%w` to wrap errors, and `errors.Is` / `errors.As` to inspect them:

```go
var ErrNotFound = errors.New("not found")

func findUser(id int) (*User, error) {
    if id <= 0 {
        return nil, fmt.Errorf("findUser(%d): %w", id, ErrNotFound)
    }
    // ...
}

err := findUser(-1)

// errors.Is checks if any error in the chain matches
if errors.Is(err, ErrNotFound) {
    fmt.Println("User was not found")
}

// errors.As extracts a specific type from the chain
var notFound *NotFoundError
if errors.As(err, &notFound) {
    fmt.Println("Resource:", notFound.Resource)
}
```

### Custom Error Types

Define a struct that implements the `error` interface:

```go
type ValidationError struct {
    Field   string
    Message string
}

func (e *ValidationError) Error() string {
    return fmt.Sprintf("validation error on field %q: %s", e.Field, e.Message)
}

func validateAge(age int) error {
    if age < 0 {
        return &ValidationError{Field: "age", Message: "must be non-negative"}
    }
    if age > 150 {
        return &ValidationError{Field: "age", Message: "unrealistically large"}
    }
    return nil
}

err := validateAge(-5)
var ve *ValidationError
if errors.As(err, &ve) {
    fmt.Println("Field:", ve.Field)    // age
    fmt.Println("Message:", ve.Message) // must be non-negative
}
```

### panic & recover

`panic` stops normal execution and unwinds the stack. Use it rarely — only for truly unrecoverable situations (programmer errors, not user errors).

```go
func mustPositive(n int) int {
    if n <= 0 {
        panic(fmt.Sprintf("expected positive, got %d", n))
    }
    return n
}

// Recover from a panic (e.g., in a web server handler)
func safeCall(fn func()) (err error) {
    defer func() {
        if r := recover(); r != nil {
            err = fmt.Errorf("recovered from panic: %v", r)
        }
    }()
    fn()
    return nil
}
```

**Rule:** Return errors for expected failure cases. Panic for bugs that should never happen.

---

## Week 3 — Project

**Shape area calculator with custom errors**

Requirements:
1. Define a `Shape` interface with `Area() float64` and `Perimeter() float64`
2. Implement `Circle`, `Rectangle`, and `Triangle`
3. Create a custom `InvalidDimensionError` type that includes the shape name, dimension name, and invalid value
4. Validate dimensions in constructors: `NewCircle(radius float64) (Circle, error)`, etc.
5. Write a function `PrintSummary(shapes []Shape)` that prints a table
6. In `main`, create shapes using constructors, handle errors, and print the summary

---

---

# Week 4 — Packages, Modules & Testing

## 4.1 Packages

Every Go file belongs to a package. The package is the fundamental unit of code organization.

```go
// math/calc.go
package math   // package declaration

// Exported — starts with uppercase
func Add(a, b int) int {
    return a + b
}

// Unexported — starts with lowercase, only accessible within this package
func clamp(n, min, max int) int {
    if n < min { return min }
    if n > max { return max }
    return n
}
```

### Package Naming Rules

- Short, lowercase, no underscores, no mixedCase
- The package name should match the directory name
- Use singular: `user` not `users`, `model` not `models`
- Avoid generic names: `util`, `common`, `helper` are red flags

### Importing Packages

```go
import "fmt"
import "os"

// Block import
import (
    "fmt"
    "os"
    "strings"
    
    // Third-party packages use their module path
    "github.com/gin-gonic/gin"
)

// Import alias
import (
    "fmt"
    myfmt "github.com/mylib/fmt"  // alias to avoid conflict
    _ "database/sql/driver"       // blank import — runs init() only
)
```

---

## 4.2 Go Modules

A module is a collection of packages with a `go.mod` file at the root.

```bash
# Create a new module
mkdir myproject && cd myproject
go mod init github.com/yourname/myproject
```

This creates `go.mod`:

```
module github.com/yourname/myproject

go 1.22
```

### Managing Dependencies

```bash
# Add a dependency (edits go.mod and go.sum)
go get github.com/gin-gonic/gin

# Add a specific version
go get github.com/gin-gonic/gin@v1.9.1

# Remove unused dependencies, tidy go.mod
go mod tidy

# Download all dependencies
go mod download

# Vendor dependencies (copy into vendor/ folder)
go mod vendor
```

The `go.sum` file contains cryptographic hashes of each module version. Commit both `go.mod` and `go.sum`.

---

## 4.3 Project Structure

For small projects, keep it flat. Don't add structure before you need it.

### Small Project (flat)

```
myapp/
├── go.mod
├── go.sum
├── main.go
├── user.go
├── user_test.go
└── config.go
```

### Growing Project

```
myapp/
├── go.mod
├── go.sum
├── main.go
├── cmd/
│   ├── serve/
│   │   └── main.go   // go run ./cmd/serve
│   └── migrate/
│       └── main.go   // go run ./cmd/migrate
├── internal/         // NOT importable outside this module
│   ├── user/
│   │   ├── user.go
│   │   └── user_test.go
│   └── config/
│       └── config.go
└── pkg/              // importable by external projects
    └── validator/
        └── validator.go
```

The `internal/` directory is enforced by the Go compiler — packages inside it can only be imported by code within the parent directory tree.

---

## 4.4 Testing

Go has a built-in testing package. No external framework needed.

### Basic Test

File must end in `_test.go`. Test functions must start with `Test` and take `*testing.T`:

```go
// math/calc_test.go
package math

import "testing"

func TestAdd(t *testing.T) {
    result := Add(2, 3)
    if result != 5 {
        t.Errorf("Add(2, 3) = %d; want 5", result)
    }
}
```

Run tests:

```bash
go test ./...          # run all tests
go test ./math/...     # run tests in math package
go test -v ./...       # verbose — show each test name
go test -run TestAdd   # run only TestAdd
```

### Table-Driven Tests

The standard Go pattern — write one test with many cases:

```go
func TestAdd(t *testing.T) {
    tests := []struct {
        name     string
        a, b     int
        expected int
    }{
        {"positive numbers", 2, 3, 5},
        {"negative numbers", -1, -2, -3},
        {"zeros", 0, 0, 0},
        {"mixed", -5, 10, 5},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got := Add(tt.a, tt.b)
            if got != tt.expected {
                t.Errorf("Add(%d, %d) = %d; want %d", tt.a, tt.b, got, tt.expected)
            }
        })
    }
}
```

`t.Run` creates subtests — you can run a specific subtest:

```bash
go test -run TestAdd/negative_numbers
```

### Testing Errors

```go
func TestDivide(t *testing.T) {
    tests := []struct {
        name        string
        a, b        float64
        expected    float64
        expectError bool
    }{
        {"normal division", 10, 2, 5.0, false},
        {"divide by zero", 10, 0, 0, true},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            result, err := Divide(tt.a, tt.b)
            if tt.expectError {
                if err == nil {
                    t.Error("expected an error but got nil")
                }
                return
            }
            if err != nil {
                t.Errorf("unexpected error: %v", err)
                return
            }
            if result != tt.expected {
                t.Errorf("got %f; want %f", result, tt.expected)
            }
        })
    }
}
```

### Useful Testing Helpers

```go
// t.Fatal — stops the test immediately (use when continuing makes no sense)
t.Fatal("setup failed, cannot continue")

// t.Skip — skip a test
if runtime.GOOS == "windows" {
    t.Skip("skipping on Windows")
}

// t.Helper — marks a function as a test helper (better error line numbers)
func assertError(t *testing.T, err error) {
    t.Helper()
    if err == nil {
        t.Fatal("expected an error but got nil")
    }
}
```

### Test Coverage

```bash
go test -cover ./...
# coverage: 82.4% of statements

# Generate an HTML coverage report
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out
```

---

## Week 4 — Project

**Multi-package temperature converter with tests**

Directory structure:
```
temperature/
├── go.mod
├── convert/
│   ├── convert.go
│   └── convert_test.go
└── main.go
```

Requirements:
1. `convert` package: functions `CelsiusToFahrenheit`, `FahrenheitToCelsius`, `CelsiusToKelvin`
2. Table-driven tests covering normal cases and edge cases (absolute zero, boiling point)
3. `main.go` CLI: takes a value and unit, prints all conversions
4. Run `go test ./...` — all tests must pass
5. Run `go test -cover ./...` — aim for >90% coverage

---

---

# Week 5 — Goroutines & Channels

## 5.1 Goroutines

A goroutine is a lightweight thread managed by the Go runtime. Creating one is cheap (~2KB of stack space, grows as needed).

```go
func say(msg string) {
    for i := 0; i < 3; i++ {
        fmt.Println(msg)
        time.Sleep(100 * time.Millisecond)
    }
}

func main() {
    go say("world")  // starts a new goroutine
    say("hello")     // runs in the main goroutine
}
```

**Problem:** If `main` returns, ALL goroutines are killed immediately.

```go
func main() {
    go fmt.Println("I might not print!")  // main exits before this runs
}
```

### sync.WaitGroup — Waiting for Goroutines

```go
import "sync"

func main() {
    var wg sync.WaitGroup

    for i := 0; i < 5; i++ {
        wg.Add(1)                          // increment counter before launching
        go func(id int) {
            defer wg.Done()                // decrement when goroutine finishes
            fmt.Printf("Worker %d done\n", id)
        }(i)                               // pass i as argument — avoid closure trap
    }

    wg.Wait()  // block until counter reaches 0
    fmt.Println("All workers done")
}
```

**The closure trap:** Be careful capturing loop variables in goroutines:

```go
// BUG — all goroutines share the same 'i' variable
for i := 0; i < 3; i++ {
    go func() {
        fmt.Println(i)  // might print 3 3 3
    }()
}

// FIX — pass 'i' as an argument to create a copy
for i := 0; i < 3; i++ {
    go func(i int) {
        fmt.Println(i)  // prints 0, 1, 2 (in some order)
    }(i)
}
```

---

## 5.2 Channels

Channels are typed conduits for communication between goroutines. They are the preferred way to share data.

```go
// Create a channel
ch := make(chan int)       // unbuffered
bch := make(chan int, 5)   // buffered with capacity 5

// Send a value (blocks if receiver isn't ready — unbuffered)
ch <- 42

// Receive a value (blocks until a value is sent)
val := <-ch

// Close a channel — signals no more values will be sent
close(ch)

// Check if channel is closed (comma-ok)
val, ok := <-ch
if !ok {
    fmt.Println("channel closed")
}
```

### Unbuffered vs Buffered

```go
// Unbuffered — sender blocks until receiver is ready (synchronous)
ch := make(chan int)
go func() {
    ch <- 1  // blocks here until main receives
}()
val := <-ch  // unblocks the goroutine
fmt.Println(val)

// Buffered — sender blocks only when buffer is full (asynchronous up to capacity)
bch := make(chan int, 3)
bch <- 1  // doesn't block
bch <- 2  // doesn't block
bch <- 3  // doesn't block
// bch <- 4  // would block — buffer full

fmt.Println(<-bch)  // 1
fmt.Println(<-bch)  // 2
fmt.Println(<-bch)  // 3
```

### Ranging Over a Channel

```go
func generate(nums ...int) <-chan int {
    out := make(chan int)
    go func() {
        for _, n := range nums {
            out <- n
        }
        close(out)  // must close to end the range loop
    }()
    return out
}

for n := range generate(1, 2, 3, 4, 5) {
    fmt.Println(n)
}
```

### Directional Channels

Restrict what a function can do with a channel:

```go
func producer(out chan<- int) {  // send-only
    out <- 42
}

func consumer(in <-chan int) {   // receive-only
    val := <-in
    fmt.Println(val)
}

ch := make(chan int)
go producer(ch)
consumer(ch)
```

---

## 5.3 select Statement

`select` lets a goroutine wait on multiple channel operations simultaneously:

```go
ch1 := make(chan string)
ch2 := make(chan string)

go func() {
    time.Sleep(1 * time.Second)
    ch1 <- "one"
}()
go func() {
    time.Sleep(2 * time.Second)
    ch2 <- "two"
}()

// Whichever channel is ready first — that case runs
select {
case msg := <-ch1:
    fmt.Println("Received from ch1:", msg)
case msg := <-ch2:
    fmt.Println("Received from ch2:", msg)
}
```

### Timeout Pattern

```go
func fetchWithTimeout(url string) (string, error) {
    result := make(chan string, 1)
    go func() {
        // simulate a slow HTTP call
        time.Sleep(3 * time.Second)
        result <- "response body"
    }()

    select {
    case res := <-result:
        return res, nil
    case <-time.After(2 * time.Second):
        return "", errors.New("request timed out")
    }
}
```

### Done Channel — Cancellation

```go
func worker(done <-chan struct{}) {
    for {
        select {
        case <-done:
            fmt.Println("worker: stopping")
            return
        default:
            fmt.Println("worker: working...")
            time.Sleep(500 * time.Millisecond)
        }
    }
}

func main() {
    done := make(chan struct{})
    go worker(done)

    time.Sleep(2 * time.Second)
    close(done)  // signal all workers to stop
    time.Sleep(500 * time.Millisecond)
}
```

---

## 5.4 Mutexes & Shared State

When goroutines share memory instead of communicating through channels, protect it with a mutex:

```go
import "sync"

type SafeCounter struct {
    mu    sync.Mutex
    count int
}

func (c *SafeCounter) Increment() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.count++
}

func (c *SafeCounter) Value() int {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.count
}

// Usage
counter := SafeCounter{}
var wg sync.WaitGroup

for i := 0; i < 100; i++ {
    wg.Add(1)
    go func() {
        defer wg.Done()
        counter.Increment()
    }()
}

wg.Wait()
fmt.Println(counter.Value())  // 100 — always
```

### RWMutex — Multiple Readers, One Writer

```go
type Cache struct {
    mu   sync.RWMutex
    data map[string]string
}

// Multiple goroutines can read at the same time
func (c *Cache) Get(key string) (string, bool) {
    c.mu.RLock()
    defer c.mu.RUnlock()
    val, ok := c.data[key]
    return val, ok
}

// Only one goroutine can write at a time
func (c *Cache) Set(key, val string) {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.data[key] = val
}
```

### The Race Detector

Always run with `-race` during development:

```bash
go run -race main.go
go test -race ./...
```

It instruments your binary to detect data races at runtime. If it reports one, it's a real bug.

---

## Week 5 — Project

**Concurrent URL health checker**

Requirements:
1. Define a `Result` struct: `URL`, `StatusCode int`, `Latency time.Duration`, `Err error`
2. Function `checkURL(url string) Result` — makes an HTTP GET and returns the result
3. Function `checkAll(urls []string, concurrency int) []Result` — checks all URLs concurrently with a worker pool (max `concurrency` workers at once)
4. Use channels for the worker pool: a jobs channel and results channel
5. Sort results by latency before printing
6. Run with `go run -race` — no data races

Test with at least 10 URLs including some that are invalid.

---

---

# Week 6 — Consolidation & Capstone

## 6.1 Generics (Go 1.18+)

Generics let you write functions and types that work with multiple types.

```go
// Before generics — you'd need separate functions or use interface{}
func SumInts(nums []int) int {
    total := 0
    for _, n := range nums { total += n }
    return total
}

func SumFloat64s(nums []float64) float64 {
    total := 0.0
    for _, n := range nums { total += n }
    return total
}

// With generics — one function for both
func Sum[T int | float64](nums []T) T {
    var total T
    for _, n := range nums { total += n }
    return total
}

Sum([]int{1, 2, 3})           // 6
Sum([]float64{1.1, 2.2, 3.3}) // 6.6
```

### Constraints

```go
import "golang.org/x/exp/constraints"

// Built-in constraints
func Max[T constraints.Ordered](a, b T) T {
    if a > b { return a }
    return b
}

// Custom constraint
type Number interface {
    int | int8 | int16 | int32 | int64 |
    float32 | float64
}

func Abs[T Number](n T) T {
    if n < 0 { return -n }
    return n
}
```

### Generic Data Structures

```go
type Stack[T any] struct {
    items []T
}

func (s *Stack[T]) Push(item T) {
    s.items = append(s.items, item)
}

func (s *Stack[T]) Pop() (T, bool) {
    var zero T
    if len(s.items) == 0 {
        return zero, false
    }
    top := s.items[len(s.items)-1]
    s.items = s.items[:len(s.items)-1]
    return top, true
}

// Type-safe stacks for any type
intStack := Stack[int]{}
strStack := Stack[string]{}
```

---

## 6.2 Common Go Patterns

### Functional Options

Solves the problem of structs with many optional configuration fields:

```go
type Server struct {
    host    string
    port    int
    timeout time.Duration
}

type Option func(*Server)

func WithHost(host string) Option {
    return func(s *Server) {
        s.host = host
    }
}

func WithPort(port int) Option {
    return func(s *Server) {
        s.port = port
    }
}

func WithTimeout(d time.Duration) Option {
    return func(s *Server) {
        s.timeout = d
    }
}

func NewServer(opts ...Option) *Server {
    s := &Server{
        host:    "localhost",   // defaults
        port:    8080,
        timeout: 30 * time.Second,
    }
    for _, opt := range opts {
        opt(s)
    }
    return s
}

// Clean call site
s := NewServer(
    WithPort(9090),
    WithTimeout(10 * time.Second),
)
```

### sync.Once — Run Exactly Once

```go
var (
    instance *DB
    once     sync.Once
)

func GetDB() *DB {
    once.Do(func() {
        instance = connectToDB()
    })
    return instance
}
```

Safe for concurrent use — `connectToDB` is called exactly once, even if `GetDB` is called from multiple goroutines simultaneously.

---

## 6.3 Capstone Project

**In-memory task queue with worker pool**

This project combines everything from Phase 1.

### Directory Structure

```
taskqueue/
├── go.mod
├── main.go
├── queue/
│   ├── queue.go
│   └── queue_test.go
└── worker/
    └── worker.go
```

### Requirements

**`queue` package:**
```go
type Status string
const (
    StatusPending   Status = "pending"
    StatusRunning   Status = "running"
    StatusCompleted Status = "completed"
    StatusFailed    Status = "failed"
)

type Task struct {
    ID        int
    Payload   string
    Status    Status
    CreatedAt time.Time
    Error     error
}

type Queue struct { /* mutex, slice */ }

func NewQueue() *Queue
func (q *Queue) Add(payload string) *Task
func (q *Queue) Next() (*Task, bool)           // returns next pending task
func (q *Queue) Complete(id int)
func (q *Queue) Fail(id int, err error)
func (q *Queue) Stats() (pending, running, completed, failed int)
func (q *Queue) All() []*Task
```

**`worker` package:**
```go
// Start n workers. Each worker:
// 1. Reads from the queue
// 2. Simulates work with time.Sleep(random 100-500ms)
// 3. Randomly fails 20% of tasks
// 4. Marks task complete or failed
func StartPool(q *Queue, n int, done <-chan struct{}, wg *sync.WaitGroup)
```

**`main.go`:**
```go
// 1. Create a queue
// 2. Add 20 tasks
// 3. Start a worker pool with 4 workers
// 4. Every second, print the current stats
// 5. Stop after all tasks are done
// 6. Print final summary
```

**Tests for `queue` package:**
- `TestAdd` — add tasks, check IDs are assigned
- `TestNext` — next returns pending tasks in order
- `TestStats` — stats count correctly after completions/failures
- Table-driven test for concurrent access (run with `-race`)

### Running

```bash
go test -race ./queue/...
go run -race main.go
```

---

## Phase 1 Completion Checklist

Before moving to Phase 2, you should be able to:

- [ ] Write a Go program from scratch without looking up syntax
- [ ] Understand value vs pointer receivers and use them correctly
- [ ] Define and implement interfaces without an "implements" keyword
- [ ] Handle errors properly — wrap with `%w`, check with `errors.Is` / `errors.As`
- [ ] Organize code into packages with correct exports
- [ ] Write table-driven tests and run `go test ./...`
- [ ] Spawn goroutines and coordinate with `WaitGroup` and channels
- [ ] Protect shared state with a mutex
- [ ] Detect and fix data races with `go run -race`
- [ ] Complete the capstone project (task queue) without looking at solutions

---

## What Comes Next — Phase 2 Preview

In Phase 2 you will build on these fundamentals to work with the real world:

- **`net/http`** — build HTTP servers and clients
- **`encoding/json`** — serialize and deserialize JSON
- **`context`** — timeouts and cancellation for HTTP requests
- **`os` and `io`** — reading files, writing output
- **`database/sql`** — connect to a real PostgreSQL database
- **CLI tools** — parsing flags, building real command-line programs

Everything in Phase 1 is used daily in Phase 2. The stronger your foundation here, the faster Phase 2 goes.

---

*Built for self-study. Read, type, build. Good luck.*