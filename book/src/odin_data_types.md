# Data Types in Odin

This text accompanies a series of videos that cover data types in the Odin programming language as an entry point to the language:

- [Data types in Odin: numbers, booleans, strings, pointers]()
- [Data types in Odin: arrays, slices, maps, structs]()
- [Data types in Odin: unions, error values]()

We assume the audience already has some familiarity with C or other languages with pointers (e.g. C++, Rust, Zig, Go).

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

Data types in Odin have a concept of a “zero value” that is significant in a few contexts. For example, when a variable is left uninitialized, Odin by default will zero out its bytes, giving it the zero value.

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

Literals in Odin have their own distinct types, which a bit confusingly are called the “untyped” types. Integer literals are untyped integers, floating-point literals are untyped floats, boolean literals are untyped booleans, and string literals are untyped strings.
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

s: string = “hello”        // untyped string implicitly cast to string
s16: string16 = “hello”    // untyped string implicitly cast to string16
cs: cstring = “hello”      // untyped string implicitly cast to cstring
```

When a variable's declared type is inferred from a literal:

```go
i := 3         // inferred to be int
f := 3.5       // inferred to be f64
b := true      // inferred to be bool
s := “hi”      // inferred to be string
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
    1 = "banana”" 
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
    4 = “apple”,
    10..=12 = “banana”,   // 10 through 12 (inclusive)
    80..<82 = “orange”,   // 30 through 30 (non-inclusive)
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

What Odin calls a slice is a value that represents a subrange of an array (or alternatively, an array-like buffer that stores contiguous, homogeneous values). Concretely, a slice contains a pointer to the start of the subrange, and then an integer for the length of the subrange.

For anyone coming from Go, it’s important to note that unlike Go slices, Odin slices do not contain a capacity, and there is no append operation for slices. For that in Odin, you instead want what Odin calls a dynamic array, which we’ll cover later in this video.

Anyway, here we’re declaring the variable s to be a slice of ints, as indicated by the empty square brackets. If we have an array of ints, we can use the slice operator to produce a slice of ints value. The slice operator looks like the index operator, but with a colon surrounded by two integers representing the start and end of the returned slice’s subrange. In this example, we’re getting a slice representing the subrange of the array that starts at index 30 and ends at index 40 (not inclusive), so the slice value will store a pointer to index 30 and a length 10.

If we then assign a value to index 0 of the slice, this is logically the same as assigning to index 30 of the array.

As a convenience, the first integer of a slice operation can be omitted, in which case it defaults to 0, so here the slice now runs from index 0 of the array up to (but not including) index 40 and has a length of 40.

The second integer expression in the slice operator can also be omitted, in which case it defaults to the length of the array, so here the slice now runs from index 20 of the array up to (but not including) index 100 and has a length of 80.

Commonly we want to get a slice with a certain length, so often we’ll compute the end index as the starting offset plus the desired length. Here for example we want a slice that starts at index 5 and has a length of 9, so the end index is computed as offset + length.

A nice idiom that lets us express this a little more elegantly is to actually do two slice operations: first we get a slice running from the desired offset to the end of the array, then we get a slice of the slice, starting from its first index up to our desired length. The end result is the same and the compiled code is probably the same, but this way we can write the offset expression just once instead of twice.


## Allocations

Before completing our discussion of slices and before introducing dynamic arrays, we need to  talk a bit about allocators in Odin. This is a larger topic than we can cover here, but we’ll hit the key ideas.
 
Because Odin is not a garbage collected language, you, the programmer, are responsible for allocating and deallocating any heap memory you want to use.

For example, if we want to create a slice whose referenced data resides on the heap, we can call the make_slice function from the base library, which returns a slice that references newly allocated heap memory, and then when we’re done with the slice, we should call delete_slice from the base library to deallocate the slice’s heap memory. Whereas before we were creating slices that referenced the memory of stack-allocated arrays, here the slice references heap allocated memory with no array involved. (And be clear that the slice variable itself is still stack allocated.)

The hidden detail here is that make_slice, delete_slice, and all other Odin functions that allocate and deallocate let you specify an allocator. Different allocators can track their allocations in different ways, and some allocators may perform better than others in different use cases. In our example, we didn’t specify the allocator, so both functions default to Odin’s default allocator. This allocator is normally accessible as context.allocator, so here we get the same result by passing context.allocator explicitly.

The other allocator that’s most commonly used is normally accessible as context.temp_allocator. This temp allocator does not track each allocation individually: instead it just tracks how much of its allotted space has been used, such that its allocations can only be freed all together instead of individually.

Here if we use the temp allocator, attempting to deallocate the slice individually will trigger a segmentation fault. Instead what we should do is eventually call free_all to deallocate all the allocations at once. This effectively resets the temp allocator, making all of its memory available again for subsequent allocations.

In practice, temp allocations are useful in situations where you know that a group of allocations can all be safely freed at the same time. For example, video games typically allocate many objects in a frame that can be neatly deallocated all at once at the end of the frame. For these allocations, the game can use the temp allocator, but for things that need to live longer than an individual frame, the game may need to use an allocator that individually tracks allocations, such as Odin’s default allocator.


## Dynamic Arrays

Now that we have some understanding of allocators, we can talk about dynamic arrays.

Whereas a normal Odin array is fixed in size, a dynamic array has no fixed size and so can grow and shrink. 

Here we’re declaring a variable arr which is a dynamic array of ints, as denoted by the reserved work dynamic inside the square brackets.

Concretely a dynamic array value resembles a slice in that it consists of a pointer and a length, but in addition a dynamic array has an integer representing its capacity and a reference to an allocator.

This capacity and allocator allows a dynamic array to behave more like what Go calls a slice, in that we’re able to append values to a dynamic array.

Whereas slices often reference subranges of stack allocated arrays, that is not an intended use case for dynamic arrays. Instead, the data referenced by a dynamic array is normally heap allocated via base library functions, such as here where this call allocates a block of 7 ints and returns a dynamic array pointing to this block, with a length of 4, capacity of 7, and a reference to the default allocator. When we’re done with this dynamic array, we should call delete_dynamic_array, which uses the referenced allocator to know which allocator to deallocate the block from.

As a sidenote, the fact that slice values do not include an allocator reference can create a bit of hassle because you must separately track the allocators used for each of your allocated slices. In contrast, dynamic arrays conveniently reference their allocator, and this in fact is generally the recommended pattern for any data types that require allocations.

Now as promised, we can append to a dynamic array, as here where we use the base library function append_elems to append three values. The function requires a pointer to the dynamic array so that it may update its length and potentially also its pointer and capacity. In this case, the block referenced by the dynamic array already had sufficient capacity for three more values, so the pointer and capacity remain the same.

But if a call to append exceeds the capacity, then a new larger block is allocated, the data is copied, and the original block is freed before the values are appended, and the pointer and capacity are updated to match the new allocation. Here, the second append exceeds the capacity, so a new allocation is made with a capacity that is at least large enough to accommodate the appended values.


## Maps


One more type of collection built into Odin are maps, which are hashmaps of key-value pairs. Concretely a map value consists of a pointer to a block of memory where the key-value pairs reside, an integer indicating the number of key-value pairs, and a reference to an allocator.

Here this variable m is declared to be a map of string keys with int values.

Before using the map, we must allocate it, and like all allocated things, we should eventually deallocate it when we no longer need it.

Once the map is allocated, we can add, set, and read key-value pairs with the index operator. Here we add a key “hi” with the value 5 to our empty map, increasing its length to 1. When we assign to an existing key, we replace its existing value, and the length of the map remains unchanged. To remove a key, we call a base library function, passing a pointer to the map and the key to remove.

For more things you can do with arrays and maps, there are a number of base library functions which we won’t cover here. I recommend browsing the documentation, starting with this list.


## Structs

Like in C and other C-like languages, a struct in Odin is a composite data type that consists of named members called fields. 

Here we’re defining a type named Cat which is a struct consisting of two fields, an int named a and an f32 named b. 

If we declare a Cat variable, we can assign to its individual fields with the dot operator.

We can also create Cat values with a literal syntax, where each field can be provided a value by name. Any omitted fields default to their zero values, and optionally, the struct name on the literal can be left inferred from the assignment target.

anonymous structs

Here we’re creating a variable whose type is an unnamed, anonymous struct.

We can explicitly cast to and from any named struct type that has the exact same field names and types.

Anonymous structs are particularly convenient for fields in other structs. Here this Dog struct has a field named nested that is itself an anonymous struct, and we can then read and write the fields of the nested struct individually or as a complete struct.

While using a named struct instead wouldn’t change the semantics, the anonymous struct effectively allows us to logically group fields without the hassle of defining a separate named struct type.

# `using` fields of structs

Another option with nested struct fields is to mark them with the reserved word using. This doesn’t change the structure of the data at all, but it makes the members of the nested struct directly accessible as if they were fields of the containing struct itself.

Here for example, we have a Cat struct that contains a Pet struct field marked with using. We can still access a cat’s pet field and its members as normal, but we can also as a stylistic convenience directly access the fields of Pet as if they are directly fields of Cat itself.

As a further convenience, we can also use a Cat value in any place where the compiler expects a Pet value, and the compiler will understand this as shorthand for the Pet within the Cat.

In this last assignment, for example, we’re not actually assigning the full Cat to the Pet variable–something which is logically impossible–but rather just assigning the Cat’s Pet field to the Pet variable.

Similarly, if we call a function that expects a Pet argument, we can seemingly pass a Cat value, but in actuality what is passed at runtime is just the Pet field of the Cat.

Finally, a struct field marked with using can be given the special name _, which makes the full Pet field inaccessible by name. We can otherwise still access the Pet members as if they are members of Cat itself, and we can still use Cat values where a Pet is expected.


## Enums
An enum in Odin is an integer type with discretely named compile time values.

Here we have a type Direction which is defined as a u32 enum with four named values: North with the value 0, East with the value 1, South with the value 2, and West with the value 3.

We then here create a Direction variable and assign it the Direction value South, which is equivalent to the u32 value 2.

If we don’t specify an enums specific integer type, it defaults to int.

If we omit the value for the first named value, it defaults to 0, and then any subsequent omitted value will default to 1 greater than the prior value.

Effectively, if we omit all the values, they will run from 0 up through 1 less than the count of named values.

In a context where an enum type is expected, such as in an assignment to an enum variable, we can omit the name of the enum type before the dot as shorthand.

Normally we only want to use the named values of an enum, but we can actually cast any integer value into an enum type, such as here where we make a Direction value from 9 even though Direction has no named value for 9.

We can even do arithmetic with enum values, though there aren’t many cases where this is useful.

In a for loop, we can loop over every named value of enum type in the order they listed in the enum definition. Here, as specified by the in clause, this loop will iterate over each named value of the Direction enum, and each iteration will assign the direction value to the first variable, d, and assign the index of the iteration to the second variable, index.

We can also switch on enum values, such as here where this switch will execute the case corresponding to the value of this Direction variable. Note that we can use shorthand for the enum values in each case. 

By default, Odin strictly demands that an enum switch have a separate case for every named value, so here when we omit cases for North and West, we’ll get a compilation error. However, if we add the #partial directive to our switch, Odin will allow us to omit cases, and we can also then have a default case.

To get an enum value name as a string, we can call a function from the reflect package. Enum_name_from_value returns the name of an enum value as a string. The function also returns a boolean that will be false if the enum value has no name. For instance, if the direction value here is 9, the returned boolean will be false because 9 has no name in the Direction enum.

Using another function from the reflect package allows us to go the other way: we can get an enum value from a string matching its name. Here we’re getting the Direction value named South from a string matching the name. If the string doesn’t match a named value of the specified enum type, the returned boolean will be false.

# enumerated arrays

One more enum-related feature are what Odin calls enumerated arrays. An enumerated array is a readonly array that is fixed at compile time and which is indexed not by number but by the named values of an enum type. 

Here we have an enum Direction with 4 named values, and then we have a compile time array called directions_spanish, which is an array of strings and also an enumerated array of the Direction type as denoted by the enum type in place of where the array size usually goes. This means the array will have four values, one for each named value of the Direction enum, and then when we index the array, we do so not with a number but with a Direction value. When we index the array with .North, we get the string “Norte”. If the indexing expression were instead Direction(0), that’s actually the same value, so we get the same string. If, though, we try to use index Direction(4), well that’s a valid Direction value, but it’s out of bounds of the array and so here triggers a compilation error.

Due to some implementation details, enumerated arrays can only use enum types where the values are contiguous. If our Direction enum had non-contiguous values, then the enumerated array would trigger a compilation error. Be clear though that the written order of the enum values doesn’t matter, nor does the range need to start at 0, as long as the range is contiguous. So here the values are 3 through 6 written out of order, but the enumerated array is still valid. 


## Unions

A union is a data type defined as a set of “variant” types such that the union can store values of any of its variant types.

Here for example we have a union named Pet, which is defined to have 3 variant types, Cat, Dog, and Bird, which let’s assume are all struct types.

If we create a Pet variable, the variable has sufficient space to store a value of the largest variant type along with a typeid. We can assign values of any variant type to this variable without an explicit cast.

So first we assign a Cat value to p, which stores the Cat value and Cat typeid in p.

Next though we assign a Dog value to p, which overwrites the Cat value and Cat typeid with the Dog value and Dog typeid.

Again, any variant type can be implicitly cast to the union type…however, we cannot cast, even explicitly, in the other direction.

Instead, to get the stored value out of a union, we need to use a type assertion, just like we used for ‘any’ pointers earlier. This type assertion here tests if the union variable p currently holds a Dog. If so, it returns the Dog value and the boolean true. If not, the type assertion returns the Dog zero value and the boolean false.

We can also use a type switch on a union value.

Here we’re switching on the Pet variable p: if p stores a Cat, then the Cat case is executed and val will have type Cat; if p stores a Dog, then the Dog case is executed and val will have type Dog; and if p stores a Bird, then the Bird case is executed and val will have type Bird.

By default, our type switch must cover all of the variant types individually, but if we add the #partial directive on our typeswitch, then we can omit cases…and we can add a default case. Here the default case is executed when p is either a Dog or Bird, and val will have type Pet.

### union zero values


Normally, the zero value of a union has a nil typeid and the union zero value evaluates as equal to nil.

The proper way to think about it though is that nil is normally a valid value of the union type, so here where we assign nil to p, that’s the same as if we explicitly cast nil to Pet. The important idea is that the compiler considers the nil value of one union type to be a distinct type from the nil values of other union types.

Also regarding nil unions, a type switch on a union can optionally include a case for nil.


### type assertions

Now in many cases, we of course want to get the value referenced by some any-type value. Instead of dereferencing or casting to a different pointer type, though, we instead have to use what Odin calls a type assertion:  

A type insertion is written after the any value expression with a dot and then the expected type inside parens. At runtime, the type assertion triggers a panic if the typeid in the any value doesn’t match the expected type but otherwise returns the referenced value.

In this example, the any variable references an int variable, so the type assertion will return an int value without panicking.

For cases where we’re uncertain about the expected type, we can use the multi-return form of type assertion that returns a boolean instead of panicking. If the expected type matches the typeid in the any value, the type assertion returns true along with the referenced value. Otherwise, the type assertion returns false along with the zero value of the expected type.

Here, again, the any variable does actually reference an int variable, so the type assertion will return the referenced int and the value true. 

### type switches

Another way to get the referenced values from an any value is to use a type switch.

This switch here branches on the typeid of the any expression, in this case the variable a. A new variable val is assigned the referenced value and will differ in type for each case. In the int case, val will be an int variable; in the f32 case, val will be an f32 variable; in the string case, val will be a string variable; and in the default case, which runs when the type matches none of the other cases, val will have the type any.

Type switches are of course more convenient than type assertions when the any value might have more than one possible type.


## Error values

Unlike many other languages, Odin has no exception mechanism. It does have runtime panics, which are triggered by some operations, such as failing bounds checks, and these panics will unwind the call stack, but there is no way in the language to catch and recover from these panics except to do some logging and cleanup before the program terminates. Consequently, panics are not a mechanism for normal error handling.

Instead, normal errors in Odin are represented as ordinary data values, and these errors should follow three strong conventions:

First, error values are always represented either as boolean, enum, or union types.

Second, errors are always returned by functions as the last return value. So if a function returns other things in addition to an error, the error should always be the last of the return types.

Third, the zero-value of an error indicates success (a.k.a. the absence of an error). So, say, a function that returns a boolean error value returns false to indicate success and true to indicate an error has occurred.

For some examples of error values, we’ll look at a few functions from the base and core libraries. 

The overflow_add function from the intrinsics package adds two numbers together and returns two values: the result of the addition plus a boolean indicating if overflow occurred. Again, true indicates that an error occurred, so the variable error here will be true in the event of an overflow.

A number of library functions that perform allocations use this Allocator_Error enum to signal allocation errors. Because allocations may fail in multiple ways, it’s useful to convey that information with an enum instead of just using a boolean to signal that some error has occurred. 

So here, say, when we call the alloc function from the mem package, it returns two values: a pointer for the allocated data plus an Allocator_Error value. If the allocation succeeds, the function will return the zero value a.k.a. the named value None. Otherwise, depending on the nature of the failure, the function will return one of the other Allocator_Error named values.

Note that we don’t use a #partial switch here, so the compiler forces us to cover every named value of the enum. It’s unwise to ignore errors, so it’s generally best to avoid #partial switches when processing errors.

While enum errors provide more information than a simple boolean, you sometimes want an error value with other kinds of data, such as string messages, and this is where union errors become useful. The variant types of a union can be anything, such as strings, structs, other unions, or whatever, so a union error can hold any information we need it to.

Here’s an example union error type from the os package, one called Error.  (The #shared_nil directive, by the way, we haven’t covered, but we can ignore it here.)

The function open from the os package is a function that returns a newly opened file plus a value of this union error type. What we should do in response to an error, of course, depends on the specific error and the specific context, which is beyond our scope here, but in the general case, we can account for all the possible kinds of error by using a type switch. Again you should generally be careful about using #partial type switches as they can make it too easy to ignore errors.


### `or_return`

Very commonly with error values, we want to immediately return the error if it is non-zero. Here we’re getting the error returned from the function then immediately returning it if it is non-zero.

This pattern is so common that Odin provides the or_return operator as shorthand for the same logic.

The or_return operator follows its operand, which is always a function call returning at least one value, and the or_return effectively consumes the last returned value and returns if it is non-zero. In this case, the overflow_add call returns a result and an error, but the or_return consumes the error, so the whole or_return expression only returns the result.

Note that because this example may return a boolean, it can only be used in a function that also returns a boolean error. If we use on_return with a function call that returns an enum or union type error, the containing function must also return that same error type.

To use or_return inside a function that returns multiple types, then it must abide the Odin convention of putting the error type last, and it must also make the return types named  variables. Here for example, the return types of function foo are given variable names, x, y, and err, which allows us to then use or_return in the function on any call that returns a bool as its last return type.


### `or_break`

Odin also has an or_break operator, which is just like or_return except it performs a break rather than a return.

### `or_continue`
And there’s also an or_continue operator, which is just like or_break except it continues instead of breaks.
### `or_else`

Lastly there is or_else, which unlike the other operators, takes a second operand on its right. The right operand is only evaluated if the left operand returns a non-zero error, and then the or_else expression evaluates into the right operand value instead of the left operand. In effect, an or_else lets us conveniently substitute a default value in the event of an error.



