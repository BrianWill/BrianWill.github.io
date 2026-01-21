# Data Types in Odin

This text is the supplementary notes for a series of videos that introduces the data types in the Odin programming language:

- [Data types in Odin: numbers, booleans, strings, pointers]()
- [Data types in Odin: arrays, slices, maps, structs]()
- [Data types in Odin: unions, error values]()

> [!WARNING]
> The videos (and text) assume no prior knowledge of Odin itself, but they assume the audience already has some familiarity with C (or other languages with pointers, such as C++, Rust, Zig, or Go). Also be clear that these notes are not intended to be sequentially read on their own without having first watched the videos.

This topic is followed by another video and text: [Polymorphism in Odin](odin_polymorphism.md)

For more on Odin, see also:

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

However, the types `int` and `uint` are generally your default choices, and their sizes depend on the target platform you’re compiling for. For example, when compiling for x64, an `int` or `uint` will be 64 bits.

There’s also the type called `byte`, which is actually just an alias for `u8`.

Floating-point numbers come in three sizes:

|  |   |
|---|---|
| `f16`  | 8 bits  |
| `f32`  | 16 bits  |
| `f64`  | 32 bits  |


## Booleans

A bit surprisingly, booleans also come in multiple sizes:

|  |   |
|---|---|
| `b8`  | 8 bits  |
| `b16`  | 16 bits  |
| `b32`  | 32 bits  |
 
While, in principle, representation of a boolean only requires a single bit, Odin has these multiple sizes mainly to allow easier interop with various binary formats and to allow you to better control padding and alignment in structs. Most of the time, however, you’ll simply default to using the type called `bool`. Like a `b8`, a `bool` is 8-bits in size, but the compiler considers `b8` and `bool` to be distinct types.

## Strings

The primary string type, called `string`, represents a UTF-8 encoded string, and the type called `string16` represents a UTF-16 encoded string.

Concretely, a `string` or `string16` value is actually a pointer to a buffer of characters and an integer representing the length of the text. So when you assign, pass, or return a string value, what’s actually being copied is just a pointer and integer, not the actual character data.

For ease of interop with C, Odin also has types `cstring` and `cstring16`. These `cstring` types have no integer to represent length because they instead use the C convention of signaling the end of the character data with a 0 byte.

Odin also has another integer type called `rune` that represents the unicode codepoint of an individual character.

> [!IMPORTANT]
> Because Odin is not a garbage collected language, you should keep in mind how the character buffers pointed to by strings are allocated. For a string literal, the character buffer is statically allocated, meaning the data resides alongside the code of the executable itself. For any string created at runtime, the character buffer must be [allocated dynamically]().

## Zero values

Data types in Odin have a concept of a "zero value" that is significant in a few contexts. For example, when a variable is left uninitialized, Odin by default will zero out its bytes, giving it the zero value.

Zero values by type:

| Type | Zero value |
|---|---|
| numbers | 0 | 
| booleans | false  |
| strings | empty string |
| pointers  |  nil  |
| structs | a zero value struct has fields that are all their zero values |
| enums | 0 (enums are represented in memory as integers) |
| unions | nil (unless the union type is declared with certain directives)

## Casts

Compared to C and some other languages, Odin is much stricter about explicit casting, disallowing most implicit conversions to help prevent absent-minded mistakes. For example, to assign an i32 value to an i64 variable, the cast cannot be left implicit. 

## Distinct types and aliases

The double colon syntax in Odin denotes a compile-time definition, either of a constant, a function, or type:

```go
Also_Int :: int               // defines Also_Int as an alias for int
My_Int :: distinct int        // defines My_Int as a type that is like int but considered separate by the compiler
```

A distinct type by be explicit cast to and from its doppelganger:

```go
// the explicit cast is required here because My_Int is distinct from int
i: int = 3                   // declare an int variable 'i' and assign it the value 3
j: My_Int = My_Int(i)        // declare a My_Int variable 'j' and assign it the value of i
```

## Literals

Literals in Odin have their own distinct types, which a bit confusingly are called the "untyped" types. Integer literals are untyped integers, floating-point literals are untyped floats, boolean literals are untyped booleans, and string literals are untyped strings.
These special untyped types have a few special rules:

- First they only exist at compile time, so you can’t, say, create a variable with one of these untyped types.
- Second, these types can be implicitly cast to their related types.
- Third, casts of a literal perform range checks.

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

A pointer value is *typed* as a specific kind of pointer, *e.g.* an `int` pointer is intended to represent the memory addresses of only ints, or a `string` pointer is intended to represent the memory addresses of only strings, *etc.* By virtue of pointers being typed and static typing, the compiler can know the type of value at the address represented by the pointer. For example, dereferencing an `int` pointer accesses an `int` value rather than some kind of other value.

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
>  Odin uses the ^ symbol instead of C’s traditional asterisk. Also unlike C, Odin puts the dereference operator on the right. Placing it on the right works out nicely when pointers are used in combination with arrays.

### rawptr

A `rawptr` is Odin's closest analog of a C void pointer. Unlike other pointers, a `rawptr` can represent the address of any kind of value.

Other pointer types can be implicitly cast to `rawptr`, but a cast from `rawptr` to other pointer types must be explicit.

### uintptr

Like a `rawptr`, a `uintptr` is a pointer that can represent the address of any kind of value, but unlike a `rawptr`, a `uintptr` can be used as an unsigned integer in arithmetic operations. 

> [!NOTE]
> Unlike C or other C-like languages, Odin doesn’t let us do arithmetic directly on pointers, but instead we can convert a pointer into a uintptr, perform the arithmetic, and then cast back to a pointer.

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
> Always keep in mind that arbitrarily indexing memory is fundamentally unsafe, as in this example where we are jumping to meaningless locations on the call stack. In real use cases, multi-pointers should generally only be used to access addresses within known allocated buffers of memory.

## Arrays

Arrays in Odin are fixed-size, homogenous, and either stack-allocated or globally allocated.

```go
// decare variable 'arr' to be an array of 5 ints
arr: [5]int          

arr = [5]int{1, 2, 3, 4, 5}     // assign to arr a literal of 5 ints
arr = {1, 2, 3, 4, 5}           // shorthand for prior (the size and type inferred is from the target)
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

arr[100] = true                 // compile time bounds check error

i := 100
arr[i] = true                   // runtime bounds check panic
```

When we try to assign to index 100 of this 5 bool array, the compiler gives us a compilation error because it knows the compile time value 100 is out of bounds for this array. If though we index an array with a runtime expression, the bounds check happens at runtime, so indexing this array with a variable whose value will be 100 will trigger a panic.

These runtime bounds checks of course incur some degree of overhead, so in some performance-critical contexts you may wish to disable them with the `#no_bounds_check` directive:

```go
arr: [5]bool       

// no bounds checks will be performed in this block
#no_bounds_check {
    i := 100
    arr[i] = bool              // no panic but unsafe at runtime
}
```


## Slices

A *slice* in Odin is a value that represents a subrange of an array (or alternatively, an array-like buffer that stores contiguous, homogeneous values). Concretely, a slice contains a pointer to the start of the subrange plus an integer for the length of the subrange.

> [!WARNING]
> For Go programmers, it’s important to note that, unlike Go slices, Odin slices do not contain a capacity, and there is no append operation for slices. Odin's closest equivalent of a Go slice is called a [dynamic array]().

```go
s: []int                 // declare 's' to be a slice of ints

arr: [100]int       
s = arr[30:40]           // from the array, get a slice starting at index 30 and ending at index 40

assert(10 == len(s))     // length of the slice is 10

// because index 0 of this slice is the same as index 30 of the array,
// these two assignments assign to the same location in memory
s[0] = -99               
arr[30] = -99            
```

As a convenience, the first integer of a slice operation can be omitted, in which case it defaults to 0, and the second integer can also be omitted, in which case it defaults to the length of the array.


## Allocations

Because Odin is not a garbage collected language, the programmer is responsible for allocating and deallocating any heap memory they want to use. 
For example, if we want to create a slice whose referenced data resides on the heap, we can call the `make_slice` function (from the base library), which returns a slice that references newly allocated heap memory. When we’re done with a heap-allocated slice, we should call `delete_slice` (from the base library) to deallocate the slice’s heap memory: 

```go
s: []int              

// returns a slice with newly allocated buffer of 10 ints
s = make_slice([]int, 10)

// deallocates the allocated buffer referenced by the slice
delete_slice(s)
```

> [!NOTE]
> The Odin base library also has functions `make` and `delete`. These [proc group]() functions are the generally preferred shorthand for invoking all variants of the `make_X` / `delete_X` functions.

Whereas before we were creating slices that referenced the memory of stack-allocated arrays, here the slice references heap allocated memory with no array involved. (And be clear that the slice variable itself is still stack allocated.)

The `make_slice` function, the `delete_slice` function, and all other allocating or deallcating functions let you pass an allocator. Different allocators can track their allocations in different ways, and some allocators may perform better than others in different use cases. 

When no allocator is explicitly passed to these functions, they implicitly use the allocator provided by the [context](). The code below is functionally the same as the code above:

```go
s: []int              

// explicitly pass the context allocator
s = make_slice([]int, 10, context.allocator)

// explicitly pass the context allocator
delete_slice(s, context.allocator)
```

See more about [the context and allocations in Odin]().

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
> Whereas slices often reference subranges of stack-allocated arrays, that is not an intended use case for dynamic arrays. Instead, the data referenced by a dynamic array is normally heap-allocated *via* base library functions.

By virtue of storing a capacity integer and allocator reference, a dynamic array allows us to append values with the `apppend_elems` function:

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
> The Odin base library also has an `append` [proc group]() function, which is generally preferred shorthand for invoking all variants of the `append_X` functions.

## Maps

*Maps* are hashmaps of key-value pairs. Concretely a map value consists of a pointer to a block of memory where the key-value pairs reside, an integer indicating the number of key-value pairs, and a reference to an allocator.

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

c: Cat        // declare a variable 'c' of type Cat
c.a = 5       // assign to the 'a' field of 'c'
c.b = 3.6     // assign to the 'b' field of 'c'

// assign a Cat literal (where 'b' is 3.6 and 'a' is 5) to 'c'
c = Cat{b = 3.6, a = 5}

// omitted fields default to zero values (so 'a' is 0 and 'b' is 0)
c = Cat{}                       

// the literal type can be inferred from the assignment context
c = {a = 5, b = 3.6}            
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

c: Cat

// cast an anonymous struct value to Cat
// (valid because they have the same set of field names and types)
c = Cat(anon)                            

// cast a cat value to the anonymous struct
anon = struct{a: int, b: f32}(c)         
```

Anonymous structs are particularly convenient for fields in other structs. Here this Dog struct has a field named nested that is itself an anonymous struct, and we can then read and write the fields of the nested struct individually or as a complete struct. 

```go
Dog :: struct {
    x: string,
    nested: struct {a: int, b: f32},     // anonymous struct field
}

d: Dog

// we can assign to individual fields of an anonymous struct member...
d.nested.a = 3

// ... or we can assign a whole anonymouse struct value
d.nested = struct {a = 5, b = 3.6}
``` 

The semantics would be exactly the same if we defined a named struct type to use for the field, but the inner anonymous struct effectively allows us to logically group fields in the outer struct with less hassle.

### struct fields with `using`

Another option with nested struct fields is to mark them with the reserved word using. This doesn’t change the structure of the data at all, but it makes the members of the nested struct directly accessible as if they were fields of the containing struct itself:

```go
Pet :: struct {name: string, weight: f32}

Cat :: struct {
    a: int,
    b: f32,
    using pet: Pet,    
}

c: Cat
c.pet.name = "Mittens"
c.name = "Mittens"       // same as prior line
```

Marking a nested struct field with `using` also means the containing struct type can be used where the nested type is expected as syntatic shorthand for the nested struct:

```go
p: Pet
p = c.pet
p = c            // same as prior line (actually assigns the nested Pet, not the Cat)

// assume that function feed_pet requires a Pet argument
feed_pet(c.pet)
feed_pet(c)      // same as prior line (actually passes the nested Pet, not the Cat)
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
c: Cat
i: int = c.y             
```


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
d = .South                             // Direction.South
```

Normally we only want to use the named values of an enum, but we can actually cast any integer value into an enum type.:

```go
d: Direction
d = Direction(9)       // OK, even though there is no named Direction value for 9
```

We can even do arithmetic with enum values (though there aren’t many cases where this is useful):

```go
d: Direction
d = Direction.West + Direction.East    // Direction(4)
```

In a `for` loop, we can loop over every named value of enum type in the order they listed in the enum definition:

```go
// loop over all named values of an enum type, printing:
// 0 North 
// 1 East
// 2 South
// 3 West
for d, index in Direction {
    fmt.println(index, d)
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
case:   // the default case (allowed by #partial)
    // d is either .North or .West
}
```

To get an enum value name as a string, we can call a function from the reflect package. The function `enum_name_from_value` returns the name of an enum value as a string. The function also returns a boolean that will be false if the enum value has no name:

```go
d: Direction = .South

if name, ok := reflect.enum_name_from_value(d); ok {
    fmt.println(name)   // prints "South"
}
```

Using function `enum_from_name` from the reflect package allows us to go the other way: we can get an enum value from a string matching the value's name.

```go
// if the string doesn’t match a named value of the 
// specified enum type, the returned boolean will be false.
if d, ok := reflect.enum_from_name(Direction, "South"); ok {
    fmt.println(int(d), d)     // prints "2 South"
}
```

### enumerated arrays

An *enumerated array* is a readonly array with values fixed at compile time and which is indexed not by number but by the named values of an enum type:

```go
Direction :: enum { North, East, South, West } 

// declare 'direcitons_spanish' as an enumerated array of four strings
// The four indices correspond to the named values of the Direction enum
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

p: Pet
// variants of Pet can be implicitly cast to Pet
p = Cat{}     // assign 'p' a Pet value containing the zero Cat value and tag 1
p = Dog{}     // assign 'p' a Pet value containing the zero Dog value and tag 2
```

> [!NOTE]
> In our example, the variant types of the union are all structs, but other kinds of types can also be variants in a union: numbers, strings, pointers, enums, etc. Even unions themselves can be variants of other unions.

While variants of a union can be implicitly cast to the union type, we cannot cast the other way around, even explicitly. Instead, to get the variant value held in a union value, we must use a *type assertion*:

```go
p: Pet
d: Dog

p = d            // implicit cast from Dog to Pet
d = p.(Dog)      // this type assertion gets the Dog from the union value 

b: Bird

// this type assertion panics because the union value does not hold a Bird
b = p.(Bird)     

ok: bool
// returns the Bird zero value and false because the union value does not hold a Bird
b, ok = p.(Bird)    

// returns the held Dog value and true
d, ok = p.(Dog)
```

By default, a union's zero value is `nil`, which has tag 0.

```go
// an uninitialized union variable has value nil
p: Pet                
assert(p == nil)
```

When we want to handle multiple variants stored in a union value, it's generally more convenient to use a *type switch*:

```go
p: Pet
// ... assign a value to p

// this type switch stores the variant value from 'p' in new variable 'val',
// whose type differs in each case
switch val in p {
case Cat:
    // val is a Cat
case Dog:
    // val is a Dog
case Bird:
    // val is a Bird
case nil:  // the nil case is optional
    // val is nil  
}
```

The `#partial` directive allows a type switch to omit variants of the union and optionally include a default case:

```go
p: Pet
// ... assign a value to p

#partial switch val in p {
case Cat:
    // val is a Cat
case:  // the default case (covers Dog, Bird, and nil)
    // val is a Pet
}
```

## Error values

Unlike many other languages, Odin has no exception mechanism. It does, though, have runtime "panics", which are triggered by some operations, such as failing bounds checks. Panics will unwind the call stack, but there is no way in the language to catch and recover from these panics except to do some logging and cleanup before the program terminates.

> [!WARNING] Panics are not a mechanism for normal error handling! An *error* represents a non-ideal eventuality beyond your code's control. A *panic* represents a bug in your code.

Normal errors in Odin are represented as ordinary data values, and these errors should follow three strong conventions:

- First, error values are always represented either as boolean, enum, or union types.
- Second, functions which return multiple values should return the error (if any) as the last return type.
- Third, the zero-value of an error indicates success (*i.e.* the absence of an error). A non-zero value indicates some kind of error occurred.

### Example of a boolean error

```go
import "base:intrinsics"

a := 999999999999999999
b := 999999999999999999

// overflow_add returns true if addition of its operands overflows
result, error := intrinsics.overflow_add(a, b)
if error {
    // overflow occurred
}
```

### Example of an enum error

A number of library functions that perform allocations use this `Allocator_Error` enum to signal allocation errors. Because allocations may fail in multiple ways, it’s useful to convey that information with an enum instead of just using a boolean to signal that some error has occurred: 

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

Typically the enum error value returned by a function should be handled by a switch:

```go
import "core:mem"

data, error := mem.alloc(100) 
switch error {
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

Typically the union error value returned by a function should be handled by a type switch:

```go
import "core:os"
import "core:io"
import "base:runtime"

file, error := os.open("path/to/file")
switch e in error {
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

Very commonly with error values, we want to immediately return the error if it is non-zero. Here we’re getting the error returned from the function then immediately returning it if it is non-zero:

```go
result, error := intrinsics.overflow_add(a, b)
if error {
    return error     // return the error
}
```

This pattern is so common that Odin provides the `or_return` operator as shorthand for the same logic:

```go
// same effect as prior example
result := intrinsics.overflow_add(a, b) or_return

```

In a single-return function, the left operand of an `or_return` must match the function's return type.

In a multi-return function, the return values must be named:

```go
foo :: proc() -> (x: int, y: string, error: bool) {
    x = 3
    y = "hi"
    error = false

    bar() or_return   // returns 3, "hi", and the boolean returned by bar()
    // ...
}
```


### `or_break`

The `or_break` operator is basically like `or_return` except it performs a break rather than a return:

```go
result, error := intrinsics.overflow_add(a, b)
if error {
    break
}
```

```go
// same effect as above
result := intrinsics.overflow_add(a, b) or_break
```

### `or_continue`

The `or_continue` operator is like `or_break` except it performs a continue rather than a break:

```go
result, error := intrinsics.overflow_add(a, b)
if error {
    continue
}
```

```go
// same effect as above
result := intrinsics.overflow_add(a, b) or_continue
```

### `or_else`

Lastly there is `or_else`, which unlike the other operators, takes a second operand on its right. The right operand is only evaluated if the left operand returns a non-zero error, and then the `or_else` expression evaluates into the right operand value instead of the left operand. In effect, an `or_else` lets us conveniently substitute a default value in the event of an error:

```go
result, error := intrinsics.overflow_add(a, b)
if error {
    result = 123
}
```

```go
// same effect as above
result := intrinsics.overflow_add(a, b) or_else 123
```


