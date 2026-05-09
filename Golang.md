
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