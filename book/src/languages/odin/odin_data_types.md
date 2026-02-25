# Odin Intro - Data Types and Polymorphism

This text is the supplementary notes for a series of videos that introduce data types and polymorphism in the Odin programming language:

- [part 1: Data Types](https://youtu.be/6aIpaUKE37I)
- [part 2: Polymorphism](https://youtu.be/p5KF99mtph0)

> [!WARNING]
> The videos and this text assume no prior knowledge of Odin itself, but they assume the audience already has some familiarity with C (or other languages with pointers, such as C++, Rust, Zig, or Go). Also be clear that this text is intended to be read after having first watched the videos.

For more about Odin, see also:

- [the Odin docs overview](http://odin-lang.org/docs/overview/)
- [the Odin docs demo](http://odin-lang.org/docs/demo/)
- [*Understanding the Odin Programming Language* by Karl Zylinski](http://odinbook.com)

## Basic Number Types

Integers in Odin come in five different sizes with both signed and unsigned types:

|  | |   |
|---|---|---|
| `i8`  | signed   | 8 bits  |
| `i16`  | signed   | 16 bits  |
| `i32`  | signed   | 32 bits  |
| `i64`  | signed   | 64 bits  |
| `i128`  | signed   | 128 bits  |


|  | |   |
|---|---|---|
| `u8`  | unsigned   | 8 bits  |
| `u16`  | unsigned   | 16 bits  |
| `u32`  | unsigned   | 32 bits  |
| `u64`  | unsigned   | 64 bits  |
| `u128`  | unsigned   | 128 bits  |

There's also the types `int` and `uint`, which are generally your default choices. Their sizes depend on the target platform you’re compiling for, *e.g.* when compiling for x64, an `int` or `uint` will be 64 bits.

There’s also the type called `byte`, which is actually just an alias for `u8`.

Floating-point numbers come in three sizes:

|  |   |
|---|---|
| `f16`  | 8 bits  |
| `f32`  | 16 bits  |
| `f64`  | 32 bits  |


## Booleans

Booleans also come in multiple sizes:

|  |   |
|---|---|
| `b8`  | 8 bits  |
| `b16`  | 16 bits  |
| `b32`  | 32 bits  |
| `b64`  | 32 bits  |
 
While, in principle, representation of a boolean only requires a single bit, Odin has these multiple sizes mainly to allow easier interop with various binary formats and to allow you to better control padding and alignment in structs. Most of the time, however, you’ll simply default to using the type called `bool`. Like a `b8`, a `bool` is 8-bits in size (though the compiler considers `b8` and `bool` to be distinct types).

## Strings

The primary string type, called `string`, represents a UTF-8 encoded string, and the type called `string16` represents a UTF-16 encoded string.

Concretely, a `string` or `string16` value is actually a pointer to a buffer of characters and an integer representing the length of the text. So when you assign, pass, or return a string value, what’s actually being copied is just a pointer and integer, not the actual character data.

For ease of interop with C, Odin also has types `cstring` and `cstring16`. These `cstring` types have no integer to represent length because they instead use the C convention of signaling the end of the character data with a 0 byte.

Odin also has another integer type called `rune` that represents the unicode codepoint of an individual character.

> [!IMPORTANT]
> Because Odin is not a garbage collected language, you should keep in mind how the character buffers pointed to by strings are allocated. For a string literal, the character buffer is statically allocated, meaning the data resides alongside the code of the executable itself. For any string created at runtime, the character buffer must be [allocated dynamically]().

## Zero values

Data types in Odin have a concept of a "zero value", meaning the value of the type where every bit is 0. When a variable is left uninitialized, it defaults to the zero value.

Zero values by type:

| Type | Zero value |
|---|---|
| numbers | 0 | 
| booleans | false  |
| strings | empty string |
| pointers  |  nil  |
| structs | all fields are all their zero values |
| enums | 0 (enums are represented in memory as integers) |
| unions | nil (unless the union type is declared with certain directives)

## Casts

Compared to C and some other languages, Odin is much stricter about explicit casting, disallowing most implicit conversions to help prevent absent-minded mistakes. For example, to assign an i32 value to an i64 variable, the cast cannot be left implicit. 

## Distinct types and aliases

The double colon syntax in Odin denotes a compile-time definition, either of a constant, a procedure, or type:

```go
// defines Also_Int as an alias for int
Also_Int :: int               

// defines My_Int as a type that is like int but 
// considered separate by the compiler
My_Int :: distinct int        
```

A distinct type by be explicit cast to and from its doppelganger:

```go
// the explicit cast is required here because My_Int is distinct from int

// declare an int variable 'i' and assign it the value 3
i: int = 3                   

// declare a My_Int variable 'j' and assign it the value of i
j: My_Int = My_Int(i)        
```

## Literals

Literals in Odin have their own distinct types, which a bit confusingly are called the "untyped" types. Integer literals are untyped integers, floating-point literals are untyped floats, boolean literals are untyped booleans, and string literals are untyped strings.
These special untyped types have a few special rules:

1. They only exist at compile time, so you can’t, say, create a variable with one of these untyped types.
1. These types can be implicitly cast to their related types.
1. Casts of a literal perform range checks.

Some example casts:

```go
x: f32 = 14          // untyped int implicitly cast to f32
y: u8 = 9            // untyped int implicitly cast to u8
y = 1000             // compile error! 1000 is not in the range of a u8

a: bool = false        // untyped boolean implicitly cast to bool
b: b16 = true          // untyped boolean implicitly cast to b16
c: b64 = true          // untyped boolean implicitly cast to b64

s: string = "hello"        // untyped string implicitly cast to string
s16: string16 = "hello"    // untyped string implicitly cast to string16
cs: cstring = "hello"      // untyped string implicitly cast to cstring
```

When a variable's declared type is inferred from a literal:

```go
i := 3         // inferred to be int
f := 3.5       // inferred to be f64
b := true      // inferred to be bool
s := "hi"      // inferred to be string
```


## Pointers

A pointer is a value that represents a memory address. To *dereference* a pointer is to access the value at the memory address represented by the pointer.

A pointer value is *typed* as a specific kind of pointer, *e.g.* an `int` pointer is intended to represent the memory addresses of only ints, or a `string` pointer is intended to represent the memory addresses of only strings, *etc.* Thanks to pointers being typed and Odin's static typing, the compiler can know the type of value at the address represented by the pointer. For example, dereferencing an `int` pointer accesses an `int` value rather than some kind of other value.

The `^` operator on the right side of a pointer expression dereferences the pointer. The `&` (reference) operator on the left side of a storage location expression (*e.g.* a variable) returns its address:

```go
p: ^int        // declare a variable 'p' which is an int pointer
i: int = 7
p = &i         // assign the address of i to p 

x: int
x = p^         // assign 7 (the dereference of p) to x
p^ = 3         // assign 3 to the dereference of p (a.k.a. the address stored in p)
```

> [!NOTE]
>  Odin uses the `^` symbol instead of C’s traditional `*`. Also unlike C, Odin puts the dereference operator on the right. Placing it on the right works out nicely when pointers are used in combination with arrays.

### rawptr

A `rawptr` is Odin's closest analog of a C void pointer. Unlike other pointers, a `rawptr` can represent the address of any kind of value.

Other pointer types can be implicitly cast to `rawptr`, but a cast from `rawptr` to other pointer types must be explicit.

### uintptr

Like a `rawptr`, a `uintptr` is a pointer that can represent the address of any kind of value, but unlike a `rawptr`, a `uintptr` can be used as an unsigned integer in arithmetic operations. 

> [!NOTE]
> Unlike C or other C-like languages, Odin doesn’t let us do arithmetic directly on pointers, but instead we can convert a pointer into a uintptr, perform the arithmetic, and then cast back to a pointer. More commonly, though, a multi-pointer is used instead.

### multi-pointers

What Odin somewhat oddly calls *multi-pointers* are pointers that can be indexed like an array to do pointer arithmetic. (Compared to a `uintptr`, a multi-pointer is a bit more convenient and less error prone.)

```go
m: [^]int      // a multi-pointer to int
i: int
m = &i         // implicit cast from ^int to [^]int

m[3] = 100     // unsafe! store int 100 at address of m + size_of(int) * 3
i = m[-5]      // unsafe! assing to i the int read from the address of m - size_of(int) * 5
```

> [!WARNING]
> Always keep in mind that arbitrarily indexing memory is fundamentally unsafe, as in this example where we are jumping to meaningless locations on the call stack. In real use cases, multi-pointers should generally only be used to access addresses within known allocated blocks of memory.

## Arrays

Arrays in Odin are fixed-size, homogenous, and either stack-allocated or globally allocated.

```go
// decare variable 'arr' to be an array of 5 ints
arr: [5]int          

// assign to arr a literal of 5 ints
arr = [5]int{1, 2, 3, 4, 5}     

// shorthand for prior (the size and type inferred is from the target)
arr = {1, 2, 3, 4, 5}           
```

```go
// declare a variable 'nums' to be an array of 3 ints
// (the size is inferred from the number of values)
nums := [?]int{11, 22, 33}      
```

```go
arr: [100]string  // an array of 100 strings

// an array literal with explicit indexes (can be partial and out of order)
arr = {
    4 = "apple",
    1 = "banana"" 
    3 = "orange",
}

// same effect as above
arr[4] = "apple"
arr[1] = "banana"
arr[3] = "orange"
```


```go
arr: [100]string  // an array of 100 strings

// indexes in an array can be ranges
arr = {
    4 = "apple",
    10..=12 = "banana",   // 10 through 12 (inclusive)
    80..<82 = "orange",   // 30 through 30 (non-inclusive)
}    

// same effect as above
arr[4] = "apple"
arr[10] = "banana"
arr[11] = "banana"
arr[12] = "banana"
arr[80] = "orange"
arr[81] = "orange"
```

Whereas an array variable in C is actually a constant pointer value, this is not the case in Odin. An Odin array is a proper value unto itself, and so arrays are assigned, passed, compared, and returned by value, not by reference. When we assign one array variable to another, the entire array is copied, and if we compare two arrays for equality, all of their corresponding indexes are compared for equality.

When you do want to assign, pass, or return arrays by reference, you can do so with array pointers or with *slices*, which we’ll cover in a moment.

By default, array indexing in Odin is bounds checked both at compile time and runtime:

```go
arr: [5]bool       

arr[100] = true          // compile time bounds check error

i := 100
arr[i] = true            // runtime bounds check panic
```

When we try to assign to index 100 of this 5 bool array, the compiler gives us a compilation error because it knows the compile time value 100 is out of bounds for this array. If though we index an array with a runtime expression, the bounds check happens at runtime, so indexing this array with a variable whose value will be 100 will trigger a panic.

These runtime bounds checks of course incur some degree of overhead, so in some performance-critical contexts you may wish to disable them with the `#no_bounds_check` directive:

```go
arr: [5]bool       

// no bounds checks will be performed in this block
#no_bounds_check {
    i := 100
    arr[i] = bool        // no panic but unsafe at runtime
}
```


## Slices

A *slice* in Odin is a value that represents a subrange of an array (or alternatively, an array-like buffer that stores contiguous, homogeneous values). Concretely, a slice contains a pointer to the start of the subrange plus an integer for the length of the subrange.

> [!WARNING]
> For Go programmers, it’s important to note that, unlike Go slices, Odin slices do not contain a capacity, and there is no append operation for slices. Odin's closest equivalent of a Go slice is called a [dynamic array]().

```go
// declare 's' to be a slice of ints
s: []int            

arr: [100]int       
// from the array, get a slice starting at index 30 and ending at index 40
s = arr[30:40]      

// length of the slice is 10
assert(10 == len(s))     

// because index 0 of this slice is the same as index 30 of the array,
// these two assignments assign to the same location in memory
s[0] = -99               
arr[30] = -99            
```

As a convenience, the first integer of a slice operation can be omitted, in which case it defaults to 0, and the second integer can also be omitted, in which case it defaults to the length of the array.


## Allocations

Because Odin is not a garbage collected language, the programmer is responsible for allocating and deallocating any heap memory they want to use. 
For example, if we want to create a slice whose referenced data resides on the heap, we can call the `make_slice` procedure (from the base library), which returns a slice that references newly allocated heap memory. When we’re done with a heap-allocated slice, we should call `delete_slice` (from the base library) to deallocate the slice’s heap memory: 

```go
s: []int              

// returns a slice with newly allocated buffer of 10 ints
s = make_slice([]int, 10)

// deallocates the allocated buffer referenced by the slice
delete_slice(s)
```

> [!NOTE]
> The Odin base library also has procedures `make` and `delete`. These [proc group]() procedures are the generally preferred shorthand for invoking all variants of the `make_x` / `delete_x` procedures.

Whereas before we were creating slices that referenced the memory of stack-allocated arrays, here the slice references heap allocated memory with no array involved. (And be clear that the slice variable itself is still stack allocated.)

The `make_slice` procedure, the `delete_slice` procedure, and all other allocating or deallcating procedures let you pass an allocator. Different allocators can track their allocations in different ways, and some allocators may perform better than others in different use cases. 

When no allocator is explicitly passed to these procedures, they implicitly use the allocator provided by the [context](https://odin-lang.org/docs/overview/#implicit-context-system). The code below is functionally the same as the code above:

```go
s: []int              

// explicitly pass the context allocator
s = make_slice([]int, 10, context.allocator)

// explicitly pass the context allocator
delete_slice(s, context.allocator)
```

See more about [the context](https://odin-lang.org/docs/overview/#implicit-context-system) and [allocations](https://odin-lang.org/docs/overview/#allocators) in Odin.

## Dynamic Arrays

Whereas a normal Odin array is fixed in size, a dynamic array has no fixed size and so can grow and shrink. Concretely, a dynamic array value resembles a slice in that it consists of a pointer and a length, but in addition, a dynamic array also has an integer representing its capacity and a reference to an allocator:

```go
// the reserved word 'dynamic' makes this a dynamic array
arr: [dynamic]int     

// returns a dynamic array with a newly allocated buffer of 7 ints,
// a logical length of 4, and a reference to the context allocator
arr = make_dynamic_array_len_cap([dynamic]int, 4, 7)

assert(4 == len(arr))                
assert(7 == cap(arr))
assert(context.allocator == arr.allocator)                

delete_dynamic_array(arr)       // deallocate from the referenced allocator
```

> [!NOTE]
> Whereas slices often reference subranges of stack-allocated arrays, that is not an intended use case for dynamic arrays. Instead, the data referenced by a dynamic array is normally heap-allocated *via* base library procedures.

By virtue of storing a capacity integer and allocator reference, a dynamic array allows us to append values with the `apppend_elems` procedure:

```go
arr: [dynamic]int
            
arr = make_dynamic_array_len_cap([dynamic]int, 4, 7)

// this append stays within the existing capacity
append_elems(&arr, 100, 101, 102)
assert(7 == len(arr))
assert(7 == cap(arr))                

// this append exceeds the capacity, so:
// 1. a new, larger buffer is allocated
// 2. the existing values are copied into this new buffer
// 3. the new elements are added to the new buffer
// 4. the original buffer is deallocated
append_elems(&arr, 123, 456)   
assert(9 == len(arr))                
assert(9 <= cap(arr))
```

> [!NOTE]
> The Odin base library also has an `append` [proc group]() procedure, which is generally preferred shorthand for invoking all variants of the `append_x` procedures.

## Maps

*Maps* are hashmaps of key-value pairs. Concretely, a map value consists of a pointer to a block of memory where the key-value pairs reside, an integer indicating the number of key-value pairs, and a reference to an allocator.

Before using a map, we must allocate it. Any time new keys are added to the map, the map's memory may be reallocated. Like all allocated things, we generally should eventually deallocate it when we no longer need it.

```go
// declare a variable 'm' which is a map of string keys and int values
m: map[string]int       

m = make_map(map[string]int)    // allocate memory for the map

m["hi"] = 5                     // adds new key "hi" with value 5 (may reallocate)
assert(1 == len(m))

m["hi"] = 7                     // sets value of the existing key

delete_key(&m, "hi")            // removes the existing key and its value
assert(0 == len(m))

delete_map(m)                   // deallocate the map when it's no longer needed
```


## Structs

Like in C and other C-like languages, a *struct* in Odin is a composite data type that consists of named members called *fields*. 

```go
// define a type named 'Cat' which is a struct consisting of two fields
Cat :: struct {
    a: int,   // field 'a' is an int
    b: f32,   // field 'b' is an f32
}

cat: Cat        // declare a variable 'cat' of type Cat
cat.a = 5       // assign to the 'a' field of 'cat'
cat.b = 3.6     // assign to the 'b' field of 'cat'

// assign a Cat literal (where 'b' is 3.6 and 'a' is 5) to 'cat'
cat = Cat{b = 3.6, a = 5}

// omitted fields default to zero values (so 'a' is 0 and 'b' is 0)
cat = Cat{}                       

// the literal type can be inferred from the assignment context
cat = {a = 5, b = 3.6}            
```

### Anonymous structs

Rather than give every struct type a name, it's sometimes more convenient to use anonymous struct types:

```go
// declare variable 'anon' with anonymous struct type having two fields
anon: struct {a: int, b: f32}

// assign an anonymous struct literal to 'anon'
anon = {a = 5, b = 3.6}

Cat :: struct {
    a: int,
    b: f32,
}

cat: Cat

// cast an anonymous struct value to Cat
// (valid because they have the same set of field names and types)
cat = Cat(anon)                            

// cast a cat value to the anonymous struct
anon = struct{a: int, b: f32}(cat)         
```

Anonymous structs are particularly convenient for fields in other structs. Here this Dog struct has a field named nested that is itself an anonymous struct, and we can then read and write the fields of the nested struct individually or as a complete struct. 

```go
Dog :: struct {
    x: string,
    nested: struct {a: int, b: f32},     // anonymous struct field
}

dod: Dog

// we can assign to individual fields of an anonymous struct member...
dod.nested.a = 3

// ... or we can assign a whole anonymouse struct value
dod.nested = struct {a = 5, b = 3.6}
``` 

The semantics would be exactly the same if we defined a named struct type to use for the field, but the inner anonymous struct effectively allows us to logically group fields in the outer struct with less hassle.

## Enums

An enum in Odin is an integer type with discretely named compile time values.


```go
// declare an enum type 'Direction' with four named u32 values
Direction :: enum u32 {
    North = 0,                 
    East = 1,                  
    South = 2,                
    West = 3,
}

// declare a variable 'd' of type Direction
d: Direction
d = Direction.South    // assign .South (2) to 'd'
assert(2 == u32(d))    
```

If an enum's integer type is left unspecified, it defaults to int.

If we omit the value for the first named value, it defaults to 0, and then any subsequent omitted value will default to 1 greater than the prior value.

```go 
Direction :: enum {    // defaults to int
    North,             // first value defaults to 0
    East = 1337,                  
    South,             // defaults to 1338 (prior value plus 1)
    West = -100,
}
```

Effectively, if we omit all the values, they will run from 0 up through 1 less than the count of named values.

```go
Direction :: enum {    // defaults to int
    North,             // 0
    East,              // 1       
    South,             // 2
    West,              // 3
}
```

In a context where an enum value is expected, such as in an assignment to an enum variable, we can omit the name of the enum type before the dot as shorthand:

```go
d: Direction
d = .South            // Direction.South
```

Normally we only want to use the named values of an enum, but we can actually cast any integer value into an enum type.:

```go
d: Direction
d = Direction(9)       // OK, even though there is no named Direction value for 9
```

We can even do arithmetic with enum values (though there aren’t many cases where this is useful):

```go
d: Direction
d = Direction.West + Direction.East    // 1 + 3 is Direction(4)
```

In a `for` loop, we can loop over every named value of enum type in the order they listed in the enum definition:

```go
// loop over all named values of an enum type, printing:
// North 0
// East 1
// South 2
// West 3
for d, index in Direction {
    fmt.println(d, index)
}
```

We can also switch on enum values, such as here where this switch will execute the case corresponding to the value of this Direction variable. Note that we can use shorthand for the enum values in each case:

```go
d: Direction

// ...assign a value to d

switch d {
case .East:
    // d is .East
case .North:
    // d is .North
case .South:
    // d is .South
case .West:
    // d is .West
}
```

By default, Odin strictly demands that an enum switch have a separate case for every named value, so here when we omit cases for North and West, we’ll get a compilation error. However, if we add the #partial directive to our switch, Odin will allow us to omit cases, and we can also then have a default case:

```go
d: Direction

// ...assign value to d

// #partial required here because we do not 
// have a case for every named value
#partial switch d {      
case .East:
    // d is .East
case .South:
    // d is .South
case:   
    // the default case (allowed by #partial)
    // d is either .North or .West
}
```

To get an enum value name as a string, we can call a procedure from the reflect package. The procedure `enum_name_from_value` returns the name of an enum value as a string. The procedure also returns a boolean that will be false if the enum value has no name:

```go
d: Direction = .South

if name, ok := reflect.enum_name_from_value(d); ok {
    fmt.println(name)   // prints "South"
}
```

Using procedure `enum_from_name` from the reflect package allows us to go the other way: we can get an enum value from a string matching the value's name.

```go
// if the string doesn’t match a named value of the 
// specified enum type, the returned boolean will be false.
if d, ok := reflect.enum_from_name(Direction, "South"); ok {
    fmt.println(int(d), d)     // prints "2 South"
}
```

### Enumerated arrays

An *enumerated array* is a readonly array with values fixed at compile time and which is indexed not by number but by the named values of an enum type:

```go
Direction :: enum { North, East, South, West } 

// Declare 'direcitons_spanish' as an enumerated array of four strings.
// The four indices correspond to the named values of the Direction enum.
directions_spanish :: [Direction]string {
    .North = "Norte",
    .East = "Este",
    .South = "Sur",
    .West = "Oeste",
}

str: string
str = directions_spanish[.North]   // "Norte"
```

## Unions

A union is a data type defined as a set of "variant" types:

- A union value can contain a single value of any of its variant types.
- The size of a union value is large enough to store the union type's largest variant.
- By default, a union value also stores a "tag", an integer that indicates the variant stored in the value.

```go
Cat :: struct {}
Dog :: struct {}
Bird :: struct {}

// declare a 'Pet' as a union of Cat, Dog, and Bird
Pet :: union { Cat, Dog, Bird }

// assume that Cat is denoted by tag 1, 
// Dog by tag 2, and Bird by tag 3

pet: Pet

// variants of Pet can be implicitly cast to Pet

// assign 'pet' a Pet value containing the zero Cat value and tag 1
pet = Cat{}     

// assign 'pet' a Pet value containing the zero Dog value and tag 2
pet = Dog{}     
```

> [!NOTE]
> In our example, the variant types of the union are all structs, but other kinds of types can also be variants in a union: numbers, strings, pointers, enums, etc. Even unions themselves can be variants of other unions.

While variants of a union can be implicitly cast to the union type, we cannot cast the other way around, even explicitly. Instead, to get the variant value held in a union value, we must use a *type assertion*:

```go
pet: Pet
dog: Dog

// implicit cast from Dog to Pet
pet = dog            

// this type assertion gets the Dog from the union value 
dog = pet.(Dog)      

bird: Bird

// this type assertion panics because the union 
// value does not hold a Bird
bird = pet.(Bird)     

ok: bool
// returns the Bird zero value and false because 
// the union value does not hold a Bird
bird, ok = pet.(Bird)    

// returns the held Dog value and true
dog, ok = pet.(Dog)
```

By default, a union's zero value is `nil`, which has tag 0.

```go
// an uninitialized union variable has value nil
pet: Pet                
assert(pet == nil)
```

When we want to handle multiple variants stored in a union value, it's generally more convenient to use a *type switch*:

```go
pet: Pet

// ... assign a value to pet

// this type switch stores the variant value from 'pet' in new variable 'p',
// whose type differs in each case
switch p in pet {
case Cat:
    // p is a Cat
case Dog:
    // p is a Dog
case Bird:
    // p is a Bird
}
```

The `#partial` directive allows a type switch to omit variants of the union and optionally include a default case:

```go
pet: Pet

// ... assign a value to pet

#partial switch val in pet {
case Cat:
    // p is a Cat
case:  
    // the default case (covers Dog, Bird, and nil)
    // p is a Pet
}
```

## Error values

Unlike many other languages, Odin has no exception mechanism. It does, though, have runtime "panics", which are triggered by some operations, such as failing bounds checks. Panics will unwind the call stack, but there is no way in the language to catch and recover from these panics except to do some logging and cleanup before the program terminates.

> [!WARNING] Panics are not a mechanism for normal error handling! An *error* represents a non-ideal eventuality beyond your programs's control. A *panic* represents a bug in your code.

Normal errors in Odin are represented as ordinary data values, and these errors should follow three strong conventions:

1. Error values are always represented either as boolean, enum, or union types.
1. Procedures which return multiple values should return the error (if any) as the last return type.
1. The zero-value of an error indicates success (*i.e.* the absence of an error). A non-zero value indicates some kind of error occurred.

### Example of a boolean error

```go
import "base:strconv"

num, err := strconv.parse_f64("-52.97")
if ok {
    // could not parse string as an f64
}
```

### Example of an enum error

A number of library procedures that perform allocations use this `Allocator_Error` enum to signal allocation errors. Because allocations may fail in multiple ways, it’s useful to convey that information with an enum instead of just using a boolean to signal that some error has occurred: 

```go
// declared in package base:runtime
Allocator_Error :: enum u8 {
    None                 = 0, 
    Out_Of_Memory        = 1, 
    Invalid_Pointer      = 2, 
    Invalid_Argument     = 3, 
    Mode_Not_Implemented = 4, 
}
```

Typically the enum error value returned by a procedure should be handled by a switch:

```go
import "core:mem"

data, err := mem.alloc(100) 
switch err {
case .None: 
    // ...  (.None indicates no error occurred)
case .Out_Of_Memory: 
    // ...  
case .Invalid_Pointer: 
    // ...  
case .Invalid_Argument: 
    // ...  
case .Mode_Not_Implemented: 
    // ...  
}
```

> [!IMPORTANT]
> Note that we don’t use a #partial switch here, so the compiler forces us to cover every named value of the enum. It’s unwise to ignore errors, so it’s generally best to avoid #partial switches when processing enum and union error values.


### Example of a union error

While enum errors provide more information than a simple boolean, we sometimes want an error value with other kinds of information, such as string messages, and this is where union errors become useful. The variant types of a union can be anything, such as strings, structs, other unions, or whatever, so a union error can hold any information we need it to:

```go
// declared in package core:os
Error :: union #shared_nil {
    os.General_Error, 
    io.Error, 
    runtime.Allocator_Error, 
    os.Platform_Error, 
}
```

Typically the union error value returned by a procedure should be handled by a type switch:

```go
import "core:os"
import "core:io"
import "base:runtime"

file, err := os.open("path/to/file")
switch e in err {
case os.General_Error: 
    // ...  
case io.Error: 
    // ...  
case runtime.Allocator_Error: 
    // ...  
case os.Platform_Error: 
    // ...  
}
```


### `or_return`

Very commonly with error values, we want to immediately return the error if it is non-zero. Here we’re getting the error returned from the procedure then immediately returning it if it is non-zero:

```go
num, okr := strconv.parse_f64("-52.97")
if !ok {
    return ok     // return the error
}
```

This pattern is so common that Odin provides the `or_return` operator as shorthand for the same logic:

```go
// same effect as prior example
num := strconv.parse_f64("-52.97") or_return
```

In a single-return procedure, the left operand of an `or_return` must match the procedure's return type.

In a multi-return procedure, the return values must be named and the error type must be last:

```go
foo :: proc() -> (x: int, y: string, err: bool) {
    x = 3
    y = "hi"
    err = false

    // if bar returns true, this returns 3, "hi", and true
    bar() or_return   
    // ...
}
```


### `or_break`

The `or_break` operator is basically like `or_return` except it performs a break rather than a return:

```go
num, ok := strconv.parse_f64("-52.97")
if !ok {
    break
}
```

```go
// same effect as above
num := strconv.parse_f64("-52.97") or_break
```

### `or_continue`

The `or_continue` operator is like `or_break` except it performs a continue rather than a break:

```go
num, ok := strconv.parse_f64("-52.97")
if !ok {
    continue
}
```

```go
// same effect as above
num := strconv.parse_f64("-52.97") or_continue
```

### `or_else`

Lastly there is `or_else`, which unlike the other operators, takes a second operand on its right. The right operand is only evaluated if the left operand returns a non-zero error, and then the `or_else` expression evaluates into the right operand value instead of the left operand. In effect, an `or_else` lets us conveniently substitute a default value in the event of an error:

```go
num, ok := strconv.parse_f64("-52.97")
if !ok {
    result = 123
}
```

```go
// same effect as above
num := strconv.parse_f64("-52.97") or_else 123
```

--- 

## What is polymorphism?

Wikipedia defines polymorphism succinctly: 

> “...polymorphism allows a value type to assume different types.” 

Where otherwise a single thing could only be one concrete type or behave in one way, polymorphism allows a thing to potentially vary in type and behave in different ways.

Another way to think of it: mechanisms of polymorphism allow us to express that a piece of code has variants that are similar but somehow different. In this sense, mechanisms of polymorphism enable abstraction building beyond what just plain procedures and plain data types allow, *i.e.* polymorphism gives us additional ways to generalize.

Now, how much one *should* attempt to abstract is a very debatable question, but polymorphism is useful to have in your toolset. In particular, we very commonly need the ability to store heterogenous data in a collection and then also need the ability to operate upon these heterogenous elements when iterating the collection. In C#, for example, we can create an array of Pets that may store any kind of Pet, whether a Cat or Dog, and then we can iterate through the collection and perform a common operation on every Pet regardless of its concrete type:

```csharp
// C#
Pet[] pets = new Pet[2];
pets[0] = new Cat();
pets[1] = new Dog();

foreach (var p in pets) {
    p.sleep();    // dynamic dispatch
}
```

This is enabled either by virtue of Cat and Dog inheriting from class Pet *or* by Cat and Dog implementing an interface Pet. Odin, however, lacks inheritance, interfaces, and other common polymorphism-related language features. So we'll look at how this and similar problems can be solved in Odin by other means.


## Compile time polymorphism

It’s helpful to distinguish between ***compile time* polymorphism** and ***runtime* polymorphism**, not just because their implementations differ but but also because they serve quite different purposes. Compile time polymorphism serves two purposes:

- deduplicating code
- overloading names

In Odin, compile time polymorphism is enabled through a few features:

- procedure groups
- parametric polymorphic procedures
- parametric polymorphic structs
- parametric polymorphic unions
- the `using` modifier for struct fields

### Procedure groups

*Procedure groups*, very simply, are procedures that are defined not as a body of code but rather as a list of other procedures. At compile time, a call to a procedure group dispatches to the procedure in its list that matches the number and types of arguments in the call.

```go
sleep_cat :: proc(cat: Cat) { /* ... */ }
sleep_dog :: proc(dog: Dog) { /* ... */ }

// a proc group 'sleep`
sleep :: proc { sleep_cat, sleep_dog }

sleep(Cat{})       // one Cat argument, so invokes sleep_cat
sleep(Dog{})       // one Dog argument, so invokes sleep_dog
```

Proc groups give us the stylistic and organizational convenience of overloading a procedure name so that we can use a single name at the call sites. Unlike overloading in other languages, however, we still have to give the individual overloads their own names.

### Parametric polymorphic procedures (generic functions)

*Parametric polymorphic procedures* are Odin's semi-equivalent of generic functions in other languages. A procedure is parameteric polymorphic if it has any parameters whose arguments and/or types are fixed for each call at compile time.

#### Parameters that require compile time arguments

A parameter which requires a compile time expression argument is denoted by a `$` prefix on the parameter name: 

```go
foo :: proc($x: int) { /* ... */ }

foo(3)        // valid because 3 is a compile time expression

i := 3
foo(i)        // compile error: argument is not a compile time expression
```

One way a compile time argument can be useful is to specify array sizes:

```go
// the argument for 'n' must be compile time expression, 
// but this allows us to use 'n' as an array size
make_array :: proc($n: uint) -> [n]f32 {
    arr: [n]f32
    return arr
}

arr_A := make_array(3)   // returns an array of 3 float 32s
arr_B := make_array(7)   // returns an array of 7 float 32s
```

A compile time argument can also allow some expressions to be evaluated at compile time:

```go
mul :: proc($val: f32) -> f32 {
    return val * val;     // val * val is evaluated at compile time
}
```

To get a similar effect as what other languages call a type parameter, we can use `typeid` parameters that require compile time arguments: 

> [!NOTE]
> Every unique type in your program is given a unique integer id called a `typeid`. Type names themselves are compile time `typeid` expressions, and the builtin procedure `typeid_of` returns the `typeid` of its single argument's type, *e.g.* `typeid_of(Cat{})` returns the `typeid` of Cat.

```go
// (slightly simplified version of runtime.new)
// 'T' is a compile time typeid expression, so it can be used like a type name
// Effectively, this one procedure can return any kind of pointer.
my_new :: proc($T: typeid) -> (^T, runtime.Allocator_Error) {
   return runtime.new_aligned(T, align_of(T))
}

int_ptr: ^int
int_ptr, _ := my_new(int)

bool_ptr: ^bool
bool_ptr, _ := my_new(bool)
```

> [!NOTE]
> For a parameteric polymorphic procedure, separate versions of procedure are compiled for each unique call signature. For instance, in the above example, the two calls to `my_new` actually invoke different code: one for which T is an int and one for which T is a bool.

#### Parameters with caller-determined types

When a parameter’s *type* is prefixed with a dollar sign, that indicates that the parameter’s *type* is determined at compile time by the type of the argument from the caller:

```go
// the arguments to 'val' can be a runtime expression of any type, 
// and T can be used as a type name
repeat_five :: proc(val: $T) -> [5]T {
    arr: [5]T
    for _, i in arr {
        arr[i] = val
    }
    return arr
}

bool_arr: [5]bool
bool_arr = repeat_five(true)
assert(bool_arr == [5]{true, true, true, true, true})

str_arr: [5]string
str_arr = repeat_five("hi")
assert(str_arr == [5]{"hi", "hi", "hi", "hi", "hi"})
```

> [!NOTE]
> Again, separate versions of a procedure are compiled for each unique call signature, so in the above example, the two calls to `repeat_five` invoke different code: one for which T is a bool and one for which T is a string.

> [!WARNING] 
> Don't be confused that we used "T" as the name for the `typeid` parameter name earlier but here now use "T" as the name for the parameter type itself. In the former case, the type "T" is determined by the `typeid` value passed as argument; in the latter case, the type "T" is determined by the type of the passed argument.

A caller-determined parameter type can be used as the type of subsequent parameters in the parameter list:

```go
// in each call, 'min' and 'max' will have the same type as 'val'
// ($ should only prefix the first T parameter)
clamp :: proc(val: $T, min: T, max: T) -> T {
    // for the procedure to compile, 
    // T must be valid operands of <= and =>
    if val <= min {
        return min
    }
    if val >= max {
        return max
    }
    return val
}

clamped_int := clamp(int(8), 2, 5)               // T is int
clamped_float := clamp(f32(8.3), 2, 5)           // T is f32

// compile error: T cannot be a boolean
clamped_bool := clamp(true, false, false)       
```

#### Parameters with both compile time arguments *and* caller-determined types

An individual parameter can both require a compile time argument *and* get its type from the caller's argument:

```go
// the array size is determined by the value passed to 'n',
// and the type of the array is determined by the type passed for 'n'
array_n :: proc($n: $T) -> ^[n]T {
    return runtime.new([n]T)
}

arr_A: ^[3]int
arr_A = array_n(3)           

arr_B: ^[5]u8
arr_B = array_n(u8(5))
```

#### `where` clauses

A `where` clause effectively allows us to restrict which types or compile time values are allowed for a procedure. The `where` clause of a procedure takes a compile time boolean expression which is evaluated for each call of the procedure. If the expression evaluates false, the call triggers a compilation error.

```go
// the where clause's boolean expression determines 
// if each call is valid at compile time
// (type_is_numeric returns true if its argument is a numeric type)
clamp :: proc(val: $T, min: T, max: T) -> T where intrinsics.type_is_numeric(T) {
    if val <= min {
        return min
    }
    if val >= max {
        return max
    }
    return val
}

// OK: int is valid for T because it is numeric
clamped_int := clamp(8, 2, 5)                          

// compile error: string is invalid for T because it is not numeric
clamped_string := clamp("banana", "orange","apple")     
```

#### Specialization

For the most part, *specialization* is just shorthand syntax for what you can otherwise express in a `where` clause, but unlike a `where` clause, specialization can introduce new type parameters: 

```go
// slash after T indicates a specialization of T,
// in this case the additional requirement that T is a slice of E
// (where E is its own type parameter)
sum :: proc(val: $T/[]$E) -> T where intrinsics.type_is_numeric(E) {
    // ...
}

// valid: []int is a slice of a numeric type
i := sum([]int{8, 2, 5})                   

// compile error: not numeric
i = sum([]bool{true, false})               

// compile error: not a slice
i = sum(8)                                 
```

> [!NOTE]
> ‘E’ stands for Element, as in ‘element of a slice’, so it is the conventional name in this situation.

In this case, though, we could express the same thing more simply with just a `where` clause and no specialization:

```go
// type_elem_type returns the type of the 
sum :: proc(val: $T[]) -> T where intrinsics.type_is_numeric(T) {
    // ...
}
```

### Parameteric polymorphic structs

What Odin calls a parametric polymorphic struct is a near equivalent of what other languages would call a generic struct (or a templated struct in C++). The type parameters are expressed as `typeid` params with `$`-prefixed name.

```go
// T and U act as effective type parameters
Cat :: struct ($T: typeid, $U: typeid) {
    x: T,
    y: int,
    z: [5]U,
}

c: Cat(f32, string)         // variant where $T is f32 and $U is string
c2: Cat(int, string)        // variant where $T is int and $U is string

// compile error: c and c2 are different variants of Cat
c = c2                      

// compile error: Cat itself is not a type
cat: Cat                    
```

> [!IMPORTANT]
> Despite sharing the same name, variations of the same parameteric struct are distinct, incompatible types.

Aside from compile time `typeid` params, a struct can also have compile time unsigned integer params, which can be used to specify sizes of arrays in the struct:

```go
// N is a compile time integer parameter, so
// it can be used to specify array sizes
Cat :: struct ($T: typeid, $U: typeid, $N: uint) {
    x: T,
    y: int,
    z: [N]U,
}

c: Cat(f32, string, 4)         // variant where $N is 4
c2: Cat(f32, string, 6)        // variant where $N is 6

arr: [4]string = c.z
```

A struct can also optionally have a `where` clause, whose boolean expression is evaluated for each variant at compile time:


```go
Cat :: struct ($T: typeid, $U: typeid, $N: uint) where N < 10 {
    a: T,
    b: int,
    c: [N]U,
}

// valid because 6 is less than 10
c: Cat(f32, string, 6)       

// compile error: invalid because 11 is greater than 10
c2: Cat(int, string, 11)     
```

The most obvious use case for generic types are collections, such as a stack:

```go
Stack :: struct($T: typeid) {
    data: [dynamic]T,
}

make_stack :: proc($T: typeid) -> Stack(T) { /* ... */ }

push :: proc(stack: ^Stack($T), val: T) { /* ... */ }

pop :: proc(stack: ^Stack($T)) -> T { /* ... */ }

// make a stack of ints
s := make_stack(int)  

// push 4 then 7 to the stack
push(&s, 4)
push(&s, 7)

// remove and return last value from the stack (7)
i := pop(&s)              
```


### Parameteric polymorphic unions

Like structs, unions can also take compile time `typeid` and unsigned integer parameters:

```go
Pet :: union ($T: typeid, $U: typeid, $N: uint) {
    T,
    int,
    [N]U,
}

p: Pet(f32, string, 4)         // variant with [4]string
p2: Pet(f32, string, 6)        // variant with [6]string

// implicit cast to Pet(f32, string, 4)
p = [4]string{}                

// implicit cast to Pet(f32, string, 6)
p2 = [6]string{}                

// compile error: p and p2 are different variants of Pet
p = p2        

// compile error: Pet itself is not a type
pet: Pet  
```
> [!NOTE]
> The main use case for a parapoly union is simply to support parapoly structs with type params in a union: if a variant type in a union has non-concrete type params, then the union itself must have type params that are passed to the variant.


### Struct fields with the `using` modifier

A struct field which is itself of a struct type can be marked with the reserved word `using`. This modifier doesn’t change the structure of the data at all, but it makes the members of the nested struct directly accessible as if they were fields of the containing struct itself:

```go
Pet :: struct {name: string, weight: f32}

Cat :: struct {
    a: int,
    b: f32,
    using pet: Pet,    
}

cat: Cat
cat.pet.name = "Mittens"
cat.name = "Mittens"       // same as prior line
```

Marking a nested struct field with `using` also means the containing struct type can be used where the nested type is expected as syntatic shorthand for the nested struct:

```go
pet: Pet
pet = cat.pet

// same as prior line (assigns the Pet inside cat, not cat itself)
pet = cat            

// assume that procedure feed_pet requires a Pet argument
feed_pet(c.pet)

// same as prior line (actually passes the nested Pet, not the Cat)
feed_pet(c)      
```

A nested struct field marked with `using` can be given the special name `_`, which makes the nested struct itself inaccessible by name (though its members can still be accessed individually as if they were members of the containing struct):

```go
Pet :: struct {
    x: bool
    y : int
}

Cat :: struct {
    a: int,
    b: f32,
    using _: Pet,         // this Pet field itself has no name
}

// can still accesss members of the nested Pet as if they belong to Cat directly
cat: Cat
i: int = cat.y             
```


## Runtime polymorphism

Whereas compile time polymorphism enables deduplication of code and overloading of names, runtime polymorphism enables us to have dynamically-typed data (including heterogeneous collections) and to operate upon this dynamically-typed data.

Odin enables runtime polymorphism with a few features:

- unions 
- untyped pointers
- procedure references

### Heterogenous collections

The preferred way to represent collections containing mixed types is with unions:

```go
Cat :: struct{}
Dog :: struct{}

Pet :: union { Cat, Dog }

pets: [10]Pet

for p in pets {
    switch p in pet {
    case Cat:
        sleep_cat(p)
    case Dog:
        sleep_dog(p)
    }
}
```

> [!TIP]
> When dealing with larger variant types, it may be preferable to include pointers in the union instead of the type itself, `e.g.` `union { ^Cat, ^Dog }` instead of `union { Cat, Dog }`. On the other hand, using pointers introduces the complication of managing the referenced memory.`    


### Extensible interfaces

As a solution for runtime polymorphism, unions have two limitations:

- A union is not extensible: you cannot add additional variants to a union without redefining the original definition. This makes it impossible to extend a union brought in from a library whose source you cannot edit (or prefer not to edit).
- The variants held in a union value can only be accessed *via* type switches or type asserts, `e.g.` in the example above, accessing the value held in a Pet required a type switch with explicit cases for Cat and Dog. So even if a union type could be extended with new variants, all existing code that uses the type would have to be edited to account for the new variants.

Runtime polymorphism also requires a way to perform dynamic dispatch, which is *not* provided by proc groups: 

```go
Cat :: struct{}
Dog :: struct{}

Pet :: union { Cat, Dog }

sleep_cat :: proc(cat: Cat) { /* ... */ }
sleep_dog :: proc(dog: Dog) { /* ... */ }
sleep_group :: proc { sleep_cat, sleep_dog }

pet: Pet = Dog{}

switch p in pet {
case Cat:
    // compile time type of p is Cat, so sleep_group
    // resolves at compile time to sleep_cat
    sleep_group(p)
case Dog:
    // compile time type of p is Dog, so sleep_group
    // resolves at compile time to sleep_dog
    sleep_group(p)
}
```

Parapoly procs don't provide runtime dispatch either:

```go
// a parapoly procedure where T must be a variant of Pet
para_sleep :: proc(pet: $T) where intrinsics.type_is_variant_of(Pet, T) { 
    // a 'when' code block is included in the compiled code only if true
    when T == Cat {
        fmt.println("cat")
    }
    when T == Dog {
        fmt.println("dog")
    }
}

// resolves at compile time to the specialization where T is Dog
para_sleep(Dog{})   
```

We can get closer to actual runtime dispatch with procedure references (which are simply what other languages would call function pointer):

```go
add :: proc(a: int, b: int) -> int {
    return a + b
}

// variable f is a proc ref
// with signature (int, int) -> int
f: proc(a: int, b:int) -> int

f = add

x := f(3, 5)   // same as calling add
```

So we can use proc references in our Pet example:

```go
Cat :: struct{
    // the field 'sleep' is a proc ref for proc that takes a Cat and returns nothing
    sleep : proc(Cat)    
}
Dog :: struct{
    // the field 'sleep' is a proc ref for proc that takes a Dog and returns nothing
    sleep : proc(Dog)
}
Pet :: union { Cat, Dog }

// psuedo-methods for each Pet type
sleep_cat :: proc(c: Cat) {}
sleep_dog :: proc(d: Dog) {}

dog : = Dog{ sleep = sleep_dog }
cat : = Cat{ sleep = sleep_cat }

pet: Pet = dog

// we cannot access the .sleep field
// without using a type switch (or type asserts), so we
// are still dispatching on type at compile time, not runtime
switch p in pet {
case Cat:
    p.sleep(p)
case Dog:
    p.sleep(p)
}
```

Even if we create a parapoly proc, the dispatch on a union value's type still happens at compile time:

```go
// a parapoly procedure where T must be a variant of Pet
// and must have a field .sleep_proc that is a proc with a parameter of type T
sleep_para :: proc(pet: $T) where intrinsics.type_is_variant_of(Pet, T) { 
    pet.sleep_proc(pet)
}

// at compile time, sleep_para requires a Cat or Dog argument, not a Pet,
// so we still need a type switch (or type asserts)
switch p in pet {
case Cat:
    sleep_para(pet)
case Dog:
    sleep_para(pet)
}
```

For actual dynamic dispatch, we need not just proc refs but also untyped pointers. Because the runtime type of the pointer's referant must be tracked, it makes sense to use an `any` pointer:

```go
Pet :: struct {
    sleep: proc(Pet)
    data: rawptr    
}

Dog :: struct{}

sleep_dog :: proc(pet: Pet) {
    dog := (^Dog)(pet.data)
    // ...
}

pet : = Pet{ 
    sleep = sleep_dog, 
    data = Dog{} 
}

// dynamically calls sleep_dog
pet.sleep(pet)
```

Effectively, this pattern establishes an extensible Pet interface: a struct can be said to implement Pet if there is a corresponding `sleep_x` proc with the correct signature, *e.g.* a Cat struct implements interface Pet if it has a corresponding `sleep_cat`.

> [!NOTE]
> The method-call syntax familiar from other languages, `x.y()`, has no special meaning in Odin. To invoke `x.y()` simply invokes the proc ref stored in field 'y' of 'x', but no arguments are implicitly passed. Hence, in our example above, the pet variable is passed explicitly.

For an interface that has multiple psuedo-methods proc refs, it is convenient to bundle the proc refs into a single struct:

```go
Dog :: struct{}

// defines the proc refs of the Pet interface
Pet_Procs :: struct {
    sleep: proc(Pet)
    eat: proc(Pet, int) -> int
}

Pet :: struct {
    procs: Pet_Procs
    data: rawptr    
}

// a constant representing the Dog 
// implementation of the Pet interface
dog_procs :: Pet_Procs{
    sleep = proc(pet: Pet) {
        dog := (^Dog)(pet.data)
        // ...
    },
    eat = proc(pet: Pet, i: int) -> int {
        dog := (^Dog)(pet.data)
        // ...
    },
}

// a Pet instance wrapping a Dog
pet := Pet{ 
    procs = dog_procs, 
    data = Dog{} 
}

// dynamically calls dog_procs.eat
i: int = pet.procs.eat(pet, 4)
```

Another quality of life affordance with this pattern is to create procedures for 'conversions' to the interface wrapper type:

```go
pet_from_dog :: proc(dog: ^Dog) -> Pet {
    return Pet { sleep = sleep_dog, data: dog }
}

// proc group for all pet_from_x procs
pet_from :: proc { pet_from_dog }

// get a Pet wrapping a Dog
dog := Dog{}
pet = pet_from(&dog)

pet.sleep(pet)
```

Not only is this pattern more concise when you need to wrap an implementing type as the interface type, it can help avoid accidents where, say, you accidentally create a Pet with a mismatch of procedures and implementing type:

```go
dog := Dog{}
// danger! mismatch of procedure reference and data type:
// calling sleep on this Pet will call sleep_cat with the 
// rawptr referencing a Dog, leading to bad behaviour
return Pet { sleep = sleep_cat, data: &dog }
```

