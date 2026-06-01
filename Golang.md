
easy way - https://gobyexample.com

## SLICE TYPE & make()
```
================================================================
            GOLANG: SLICE TYPE & make() NOTES
================================================================

----------------------------------------------------------------
1. WHAT IS A SLICE TYPE?
----------------------------------------------------------------
A slice is a **dynamic sequence data type** in Go.

It represents:
  - A collection of elements of the same type
  - Flexible size (can grow/shrink)
  - Built on top of arrays internally

Syntax of slice type:
  []T

Examples:
  []int
  []string
  []float64

Key idea:
  - []int = "slice of integers"
  - []string = "slice of strings"

IMPORTANT:
  - Each []T is a DIFFERENT TYPE
  - []int ≠ []string

----------------------------------------------------------------
2. DECLARING A SLICE
----------------------------------------------------------------
You can declare a slice like this:

  var s []int

What happens:
  - s is created
  - s is NIL (no memory allocated)

Properties:
  len(s) = 0
  cap(s) = 0
  s == nil → true

IMPORTANT:
  - This slice is NOT usable for direct indexing
  - No underlying storage exists yet

----------------------------------------------------------------
3. WHAT IS make()?
----------------------------------------------------------------
make() is a built-in function used to:

  → CREATE and INITIALIZE a slice
  → ALLOCATE memory for its elements

It makes the slice READY TO USE

----------------------------------------------------------------
4. SYNTAX OF make()
----------------------------------------------------------------
  make([]T, length, capacity)

Where:
  []T       → slice type
  length    → number of usable elements
  capacity  → total allocated space (optional)

----------------------------------------------------------------
5. EXAMPLE OF make()
----------------------------------------------------------------
  s := make([]int, 3)

What happens internally:
  1. Memory is allocated for 3 integers
  2. Slice is created pointing to that memory
  3. Values initialized to zero

Result:
  s = [0 0 0]
  len(s) = 3
  cap(s) = 3
  s != nil

----------------------------------------------------------------
6. WITH CAPACITY
----------------------------------------------------------------
  s := make([]int, 3, 5)

What happens:
  - Underlying storage = 5 elements
  - Usable length = 3

Result:
  len(s) = 3
  cap(s) = 5

----------------------------------------------------------------
7. DIFFERENCE: var vs make()
----------------------------------------------------------------
  var s []int
    - nil slice
    - no memory
    - cannot assign directly

  s := make([]int, 3)
    - initialized slice
    - memory allocated
    - ready to use

Example:

  var s []int
  s[0] = 10   ❌ panic

  s := make([]int, 3)
  s[0] = 10   ✅ works

----------------------------------------------------------------
8. KEY BEHAVIOR OF SLICES
----------------------------------------------------------------
1. Dynamic Size
   - Slices can grow using append()

2. Zero Initialization
   - make() initializes elements to zero values
     int → 0
     string → ""
     bool → false

3. Type Safety
   - Slice type is fixed
   - []int can only store integers

----------------------------------------------------------------
9. WHY make() IS IMPORTANT
----------------------------------------------------------------
- Allocates memory upfront
- Avoids runtime errors
- Improves performance (less reallocation)
- Required when you want fixed initial size

----------------------------------------------------------------
10. MENTAL MODEL
----------------------------------------------------------------
  []T        → defines the TYPE of slice
  var s []T  → declares a NIL slice (no memory)
  make()     → creates slice + allocates memory

----------------------------------------------------------------
FINAL SUMMARY
----------------------------------------------------------------
- Slice = dynamic array-like type (written as []T)
- make() = initializes slice with memory
- var []T = only declaration (nil, not usable for indexing)
- make([]T, n) = ready-to-use slice with n elements

================================================================
```


## VARIADIC, ARRAYS, SLICES & MEMORY
```
================================================================
       GOLANG: VARIADIC, ARRAYS, SLICES & MEMORY NOTES
================================================================

----------------------------------------------------------------
1. VARIADIC FUNCTIONS
----------------------------------------------------------------
A variadic function accepts a VARIABLE NUMBER of arguments
of the same type.

Syntax:
  func name(params ...T) returnType

Example:
  func sum(nums ...int) int {
      total := 0
      for _, n := range nums {
          total += n
      }
      return total
  }

Inside the function:
  - nums behaves as []T (a slice)
  - Can use len(), range, indexing on it

Calling:
  sum()              → 0
  sum(1, 2)          → 3
  sum(1, 2, 3, 4, 5) → 15

Spreading a slice into a variadic call:
  nums := []int{1, 2, 3}
  sum(nums...)       → 6

RULES:
  - Variadic param must be the LAST parameter
  - Only ONE variadic parameter allowed per function

Real-world example:
  fmt.Println(a ...any)

----------------------------------------------------------------
2. ARRAY vs SLICE
----------------------------------------------------------------
ARRAY:
  - Fixed size, baked into the type
  - [3]int and [5]int are DIFFERENT types
  - Passed by VALUE (copied) into functions
  - Zero value = usable array of zeroed elements

  var a [3]int           → [0 0 0]
  b := [3]int{1, 2, 3}
  c := [5]int{1, 2, 3}   // different type from b!

SLICE:
  - Dynamic, grows with append()
  - Type is just []T (size NOT part of type)
  - Internally a struct: { pointer, length, capacity }
  - Passed by header (still shares underlying array)
  - Zero value = nil (len=0, cap=0, but append works)

  s := []int{1, 2, 3}
  s = append(s, 4)       → [1 2 3 4]

KEY DIFFERENCES:
  Size       → array fixed | slice dynamic
  Type       → [N]T includes N | []T does not
  Passing    → array copies all | slice shares array
  Zero value → zeroed array     | nil slice

WHEN TO USE:
  - Arrays: fixed-size buffers ([16]byte hash, [256]bool LUT)
  - Slices: almost everything else

GOTCHA:
  - s[1:3] does NOT copy data — it shares memory
  - Use copy() or slices.Clone() for independent copy

----------------------------------------------------------------
3. DOES make([]T, len, cap) MAKE A SLICE FIXED?
----------------------------------------------------------------
NO. Capacity is a HINT, not a hard limit.

  s := make([]int, 0, 3)  → len=0, cap=3
  s = append(s, 1, 2, 3)  → len=3, cap=3 (no realloc)
  s = append(s, 4)        → len=4, cap=6 (REALLOCATED)

KEY POINT:
  - cap = how much room is preallocated
  - Slice can still grow past cap
  - Go just allocates a bigger array when needed

If you want TRULY FIXED size:
  - Use an array, not a slice
  - var a [3]int
  - append(a, 4) → ❌ compile error

----------------------------------------------------------------
4. SUBTLE GOTCHA: make([]T, 3) vs make([]T, 0, 3)
----------------------------------------------------------------
  a := make([]int, 3)      → len=3, cap=3, [0 0 0]
  b := make([]int, 0, 3)   → len=0, cap=3, []

  a = append(a, 1)         → [0 0 0 1]   ⚠️
  b = append(b, 1)         → [1]         ✅

WHY THE 3 ZEROS APPEAR:
  - make([]int, 3) = make([]int, 3, 3)
  - 2nd argument is LENGTH
  - Go initializes every element within length to zero
  - append always inserts at index = len(s)
  - Since len(a)=3, the 1 goes at index 3 → [0 0 0 1]

RULE OF THUMB:
  - When BUILDING with append → use make([]T, 0, n)
  - When you need n preinitialized zeros → use make([]T, n)

----------------------------------------------------------------
5. PREALLOCATION vs APPEND — PERFORMANCE
----------------------------------------------------------------
WITHOUT PREALLOCATION:
  s := []int{}
  for i := 0; i < 1000; i++ {
      s = append(s, i)  // ~10 reallocations
  }

WITH PREALLOCATION:
  s := make([]int, 0, 1000)
  for i := 0; i < 1000; i++ {
      s = append(s, i)  // never reallocates
  }

WHY PREALLOC IS FASTER:
  - Each realloc allocates new array + copies all elements
  - That copy is O(n)
  - Repeated reallocs add up in hot loops / large slices

WHEN TO PREALLOCATE:
  ✅ Known size or tight upper bound
  ✅ Hot loops, large slices
  ❌ Don't guess wildly "just in case" — wastes memory
  ❌ Unknown size? Just use append, it's fine

COMMON PATTERN — transforming a slice:
  result := make([]string, 0, len(input))
  for _, x := range input {
      result = append(result, transform(x))
  }

----------------------------------------------------------------
6. SIZE COMPARISON: WITH vs WITHOUT PREALLOC
----------------------------------------------------------------
After appending 1000 elements:

WITH PREALLOC — make([]int, 0, 1000):
  len = 1000
  cap = 1000   (exact)
  Memory: one allocation, no slack

WITHOUT PREALLOC — []int{}:
  len = 1000
  cap ≈ 1024 to 1280   (depends on Go version)
  Memory: ~280 unused slots sitting around

COSTS OF NOT PREALLOCATING:
  1. TIME → ~10 reallocs, ~1000 extra element copies
  2. MEMORY → slack capacity until GC

----------------------------------------------------------------
7. HOW append() GROWS CAPACITY
----------------------------------------------------------------
Go does NOT grow capacity by 1 each append.
It grows in CHUNKS (typically doubling).

WHY NOT BY 1?
  - Growing by 1 → every append copies all elements
  - n appends would cost O(n²)
  - For n=1000 → ~500,000 copies. For n=1M → catastrophic.

DOUBLING STRATEGY:
  - Makes append AMORTIZED O(1) per call
  - Most appends free (just write a slot)
  - Occasional realloc is expensive but rare

GROWTH SEQUENCE (starting from cap=0):
  append 1st  → cap 0 → 1   (bootstrap from zero)
  append 2nd  → cap 1 → 2   (double)
  append 3rd  → cap 2 → 4
  append 5th  → cap 4 → 8
  append 9th  → cap 8 → 16
  append 17th → cap 16 → 32
  ...

NOTE on growth formula (Go 1.18+):
  - cap < 256 → double
  - cap ≥ 256 → grow ~25% each time
  - Final capacity rounded to memory size classes

BETWEEN REALLOCS — appends are essentially free:
  s := make([]int, 0, 4)
  s = append(s, 1)  // cap stays 4, write slot 0
  s = append(s, 2)  // cap stays 4, write slot 1
  s = append(s, 3)  // cap stays 4, write slot 2
  s = append(s, 4)  // cap stays 4, write slot 3
  s = append(s, 5)  // NOW cap → 8, copy 4 elements

----------------------------------------------------------------
8. MENTAL MODEL: cap vs len
----------------------------------------------------------------
LENGTH (len):
  - How many elements the slice CURRENTLY contains
  - Elements within length are initialized
  - append writes at index = len

CAPACITY (cap):
  - How many slots the underlying array can hold
  - Before needing to grow (reallocate)
  - cap is a performance HINT, not a hard cap

LANDLORD ANALOGY:
  - cap = rooms the landlord built for you
  - len = rooms you've actually moved into
  - Most appends → just move into an existing room
  - Out of rooms → landlord builds a BIGGER house
    and moves all your stuff. Expensive — done rarely.

----------------------------------------------------------------
FINAL SUMMARY
----------------------------------------------------------------
- Variadic = func f(x ...T), inside it x is []T
- Arrays = fixed size, copied; Slices = dynamic, shared
- make([]T, len, cap) does NOT make slice fixed
- make([]T, n) preinitializes n zero values
- make([]T, 0, n) reserves n slots, len=0 (preferred for append)
- Preallocate when size is known → fewer reallocs, less memory
- append grows in doubling chunks → amortized O(1)
- len = elements in use, cap = slots available

================================================================
```


```
================================================================
            ANONYMOUS FUNCTIONS
================================================================

----------------------------------------------------------------
1. WHAT IS AN ANONYMOUS FUNCTION?
----------------------------------------------------------------
A function WITHOUT A NAME.

- Defined inline
- Used once OR stored in a variable
- Treated as a VALUE (functions are first-class in Go)

----------------------------------------------------------------
2. SYNTAX
----------------------------------------------------------------
  func(parameters) returnType {
      // body
  }

No name after `func`.

----------------------------------------------------------------
3. ASSIGN TO A VARIABLE
----------------------------------------------------------------
  add := func(a, b int) int {
      return a + b
  }

  add(3, 4)   // 7

Type of add:
  func(int, int) int

----------------------------------------------------------------
4. IMMEDIATELY INVOKED
----------------------------------------------------------------
Call it right where you define it:

  func() {
      fmt.Println("Hello")
  }()

The trailing `()` CALLS the function.

With arguments:

  func(name string) {
      fmt.Println("Hi", name)
  }("Sam")

----------------------------------------------------------------
5. CLOSURES
----------------------------------------------------------------
An anonymous function can access variables from its
SURROUNDING SCOPE. This is called a CLOSURE.

  func counter() func() int {
      count := 0
      return func() int {
          count++
          return count
      }
  }

  c := counter()
  c()   // 1
  c()   // 2
  c()   // 3

Key idea:
  - Inner function "remembers" count
  - count survives after counter() returns

----------------------------------------------------------------
6. COMMON USES
----------------------------------------------------------------
(a) Goroutines
      go func() {
          fmt.Println("background work")
      }()

(b) defer
      defer func() {
          fmt.Println("cleanup")
      }()

(c) Callbacks (passed as arguments)

----------------------------------------------------------------
FINAL SUMMARY
----------------------------------------------------------------
- Anonymous function = function with no name
- Can be assigned, invoked immediately, or passed around
- Supports closures (captures outer variables)
- Common in goroutines, defer, and callbacks

================================================================
```



```
================================================================
       CLOSURES & ANONYMOUS FUNCTIONS — DEEP DIVE
================================================================

----------------------------------------------------------------
1. ANONYMOUS FUNCTION
----------------------------------------------------------------
A function WITHOUT A NAME.

Syntax:
  func(parameters) returnType {
      // body
  }

Three usage patterns:
  (a) Assign to a variable
  (b) Invoke immediately (IIFE)
  (c) Pass as argument / return as value

(a) Assigned:
  add := func(a, b int) int { return a + b }
  add(3, 4)   // 7

(b) Immediately invoked:
  func() {
      fmt.Println("hello")
  }()

(c) Passed as callback:
  apply(nums, func(x int) int { return x * x })

Functions in Go are FIRST-CLASS values
  → can be assigned, passed, returned

----------------------------------------------------------------
2. WHAT IS A CLOSURE?
----------------------------------------------------------------
A closure is a function that REMEMBERS variables from
the scope where it was created — even after that scope
has finished executing.

  Closure = function + captured variables (its "backpack")

Basic example:
  func counter() func() int {
      count := 0
      return func() int {
          count++
          return count
      }
  }

  c := counter()
  c()   // 1
  c()   // 2
  c()   // 3

Why it works:
  - counter() returns the inner function
  - Inner function uses `count`
  - `count` survives because the inner function holds it

----------------------------------------------------------------
3. CAPTURED BY REFERENCE
----------------------------------------------------------------
Closures capture variables BY REFERENCE, not by value.

  x := 10
  f := func() { fmt.Println(x) }
  x = 20
  f()   // prints 20

The closure sees the LATEST value, not a snapshot.

----------------------------------------------------------------
4. EACH CLOSURE HAS ITS OWN STATE
----------------------------------------------------------------
Every call to the outer function creates a NEW backpack.

  c1 := counter()
  c2 := counter()

  c1()   // 1
  c1()   // 2
  c2()   // 1   (independent of c1)

----------------------------------------------------------------
5. LOOP VARIABLE PITFALL
----------------------------------------------------------------
  for i := 0; i < 3; i++ {
      funcs = append(funcs, func() {
          fmt.Println(i)
      })
  }

Before Go 1.22 → prints 3 3 3 (all share same i)
Go 1.22+       → prints 0 1 2 (each iteration gets own i)

Fix for older Go:
  for i := 0; i < 3; i++ {
      i := i              // shadow it
      funcs = append(funcs, func() {
          fmt.Println(i)
      })
  }

================================================================
            MEMORY MODEL — HOW CLOSURES WORK
================================================================

----------------------------------------------------------------
6. STACK vs HEAP
----------------------------------------------------------------
  STACK
    - Fast, temporary
    - Local variables
    - Wiped when function returns

  HEAP
    - Stays alive while something references it
    - Cleaned by garbage collector (GC)

Normal variable:
  func number() {
      a := 10           // lives on STACK
  }
  number()              // a is gone

Captured variable:
  func counter() func() int {
      count := 0        // lives on HEAP (because captured)
      return func() int {
          count++
          return count
      }
  }

----------------------------------------------------------------
7. ESCAPE ANALYSIS
----------------------------------------------------------------
The compiler checks at COMPILE TIME:

  "Is this variable used by a function that outlives
   the current scope?"
       YES → move it to HEAP (variable "escapes")
       NO  → keep it on STACK

You don't write any syntax — compiler decides.

Check it yourself:
  go build -gcflags="-m" main.go

Output:
  moved to heap: count
  func literal escapes to heap

----------------------------------------------------------------
8. MENTAL MODEL
----------------------------------------------------------------
  Stack variable  → sticky note on a desk
                    (thrown away when you leave)

  Heap variable   → sticky note locked in a locker
                    (anyone with the key can return)

  Closure         → the key to the locker

  HEAP                           STACK
  ┌──────────────┐               ┌─────────────────────┐
  │  count = 0   │ ◄─────────────┤ inner function      │
  └──────────────┘               │ (holds pointer ──┐) │
                                 └──────────────────┼──┘
                                                    ▼
                                              points to count

----------------------------------------------------------------
9. WHEN DO CAPTURED VARIABLES DIE?
----------------------------------------------------------------
When NOTHING references them anymore.

  c := counter()   // count alive on heap
  c()              // count = 1
  c = nil          // no references → GC frees it

================================================================
            CLOSURES IN OTHER LANGUAGES
================================================================

----------------------------------------------------------------
10. SAME IDEA, DIFFERENT SYNTAX
----------------------------------------------------------------
JavaScript:
  const counter = () => {
      let count = 0;
      return () => {
          count++;
          return count;
      };
  };

Python:
  def counter():
      count = 0
      def inner():
          nonlocal count
          count += 1
          return count
      return inner

Go:
  func counter() func() int {
      count := 0
      return func() int {
          count++
          return count
      }
  }

Differences:
  - JS: arrow functions (`=>`), short syntax
  - Python: needs `nonlocal` to modify captured vars
  - Go: always use `func`, explicit return types

Mental model is identical across all three.

================================================================
            CLOSURES IN MIDDLEWARE
================================================================

----------------------------------------------------------------
11. THE FRAMEWORK CONSTRAINT
----------------------------------------------------------------
Every framework forces middleware into a FIXED SIGNATURE:

  Fiber    → func(ctx *fiber.Ctx) error
  Express  → function(req, res, next)
  FastAPI  → async def m(request, call_next)

You CANNOT add extra parameters.

Why fixed?
  - Framework calls thousands of handlers generically
  - Doesn't know about your DB, logger, secret key
  - Fixed signature = fast, type-safe, decoupled

Think USB port: one shape, many devices plug in.

----------------------------------------------------------------
12. THE PROBLEM
----------------------------------------------------------------
Real middleware needs extras:
  - logger
  - DB connection
  - secret key
  - rate limit config

But framework only gives `ctx`. Where do the extras go?

----------------------------------------------------------------
13. THE CLOSURE SOLUTION
----------------------------------------------------------------
  func Auth(secret string) fiber.Handler {  // ← extras here
      return func(ctx *fiber.Ctx) error {   // ← framework shape
          validate(ctx.Get("token"), secret)
          return ctx.Next()
      }
  }

  app.Use(Auth("my-secret-key"))

What happens:
  - Outer fn takes extras (your world)
  - Inner fn matches framework's signature
  - Inner fn captures extras via closure
  - Extras live on HEAP until server shuts down

----------------------------------------------------------------
14. TIMELINE
----------------------------------------------------------------
SETUP TIME (once):
  main()
    └─ Auth("secret")
        └─ "secret" escapes to HEAP 🎒
        └─ returns inner fn
        └─ framework stores it

WAITING:
  Auth() has already returned
  But "secret" is still alive on heap

REQUEST TIME (later, many times):
  framework → calls inner fn with ctx
              └─ reaches backpack → finds "secret"
              └─ uses it

Key insight:
  Closures don't pass values DOWN through layers.
  They carry values FORWARD in time.

----------------------------------------------------------------
15. PER-ROUTE CONFIGURATION
----------------------------------------------------------------
Same middleware, different configs:

  app.Use("/api/free",    RateLimiter(10))
  app.Use("/api/premium", RateLimiter(1000))

Two SEPARATE closures, each with its own captured value.
Impossible with global variables.

----------------------------------------------------------------
16. WHY NOT ALTERNATIVES?
----------------------------------------------------------------
(a) Global variables
    - Only one config possible
    - Hidden dependencies
    - Hard to test

(b) Stuffing into ctx.Locals
    - No compile-time safety
    - Type assertions everywhere
    - Dependencies hidden inside handler body

(c) Closures ✅
    - Explicit dependencies in signature
    - Type-safe
    - Per-route configuration
    - Clean separation

================================================================
            FULL EXAMPLE: FIBER API
================================================================

----------------------------------------------------------------
17. COMPLETE WORKING CODE
----------------------------------------------------------------
  package main

  import (
      "fmt"
      "github.com/gofiber/fiber/v2"
  )

  // ---------- CONTROLLER ----------
  func ListEmployees(ctx *fiber.Ctx) error {
      employees := []string{"Alice", "Bob", "Charlie"}
      return ctx.JSON(fiber.Map{"employees": employees})
  }

  // ---------- MIDDLEWARE (closure) ----------
  func LoggerMiddleware(appName string) fiber.Handler {
      return func(ctx *fiber.Ctx) error {
          fmt.Printf("[%s] %s %s\n",
              appName, ctx.Method(), ctx.Path())
          return ctx.Next()
      }
  }

  // ---------- ROUTER ----------
  func SetupRoutes(app *fiber.App) {
      app.Use(LoggerMiddleware("MyAPI"))
      app.Get("/employees", ListEmployees)
  }

  // ---------- MAIN ----------
  func main() {
      app := fiber.New()
      SetupRoutes(app)
      app.Listen(":3000")
  }

Where the closure is:
  - Outer: LoggerMiddleware(appName) — runs ONCE at setup
  - Inner: func(ctx) — runs PER REQUEST
  - `appName` captured from outer, stored on HEAP

----------------------------------------------------------------
18. STRUCT METHODS vs CLOSURES
----------------------------------------------------------------
Both achieve dependency injection.

Struct method (good for many dependencies):
  type Service struct {
      DB  *gorm.DB
      Log *zap.SugaredLogger
  }

  func (s *Service) ListEmployees(ctx *fiber.Ctx) error {
      // s.DB and s.Log are "captured" via receiver
  }

Closure (good for one or two values):
  func Auth(secret string) fiber.Handler {
      return func(ctx *fiber.Ctx) error { ... }
  }

Rule of thumb:
  - Many deps → struct with methods
  - One/two deps → closure

Both are valid; closures are the lighter option.

================================================================
            COMMON USES OF CLOSURES
================================================================

----------------------------------------------------------------
19. WHERE YOU'LL SEE THEM
----------------------------------------------------------------
(a) Middleware (auth, logging, rate limit)
(b) Goroutines:
      go func() { doWork() }()
(c) defer:
      defer func() { cleanup() }()
(d) Function factories:
      multiplier(2), multiplier(3)
(e) Callbacks (sort.Slice, http.HandlerFunc)
(f) Encapsulating private state without structs

================================================================
            FINAL SUMMARY
================================================================

CONCEPTS:
  - Anonymous fn = function with no name
  - Closure      = function + captured variables
  - Captured by REFERENCE, not value
  - Each outer-fn call = new independent closure

MEMORY:
  - Normal locals → STACK → die on return
  - Captured vars → HEAP  → live as long as referenced
  - Compiler decides via ESCAPE ANALYSIS

MIDDLEWARE:
  - Framework forces fixed signature: func(ctx) error
  - Closures smuggle in extra dependencies
  - Setup-time capture, request-time usage
  - Each app.Use() call = independent closure
  - Enables per-route configuration

MENTAL MODEL:
  - Stack variable = sticky note on desk
  - Heap variable  = sticky note in a locker
  - Closure        = the key to the locker

ONE-LINE TAKEAWAY:
  Closures let a function remember variables from its
  birthplace and carry them forward in time, which is
  exactly what frameworks need to combine a fixed handler
  signature with per-route dependencies.

================================================================
```


