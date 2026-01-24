# Polymorphism in Odin

This text accompanies two videos that cover polymorphism in the Odin programming language:

- [Compile-time Polymorphism in Odin]()
- [Runtime Polymorphism in Odin]()

This follows from the pre-requisite topic, [Data Types in Odin](odin_data_types.md)


## What is polymorphism?

Wikipedia defines polymorphism succinctly: 

> “...polymorphism allows a value type to assume different types.” 

Where otherwise a single thing could only be one concrete type or behave in one way, polymorphism allows a thing to potentially vary in type and behave in different ways.

Another way to think of it: mechanisms of polymorphism allow us to express that a piece of code has variants that are similar but somehow different. In this sense, mechanisms of polymorphism enable abstraction building beyond what just plain functions and plain data types allow, *i.e.* polymorphism gives us additional ways to generalize.

Now, how much one *should* attempt to generalize, to abstract, in code is a very debatable question, but polymorphism is useful to have in your toolset. One very common case is the need to express heterogenous data in a collection and then operate upon all elements of the collection. In C#, for example, we can create an array of Pets that may store any kind of Pet, whether a Cat or Dog, and then we can iterate through the collection and perform a common operation on every Pet regardless of its concrete type:

```csharp
// C#
Pet[] pets = new Pet[2];
pets[0] = new Cat();
pets[1] = new Dog();

foreach (var p in pets) {
    p.sleep();    // dynamic dispatch
}
```

This is enabled either by virtue of Cat and Dog inheriting from class Pet *or* by Cat and Dog implementing an interface Pet. Odin, however, lacks inheritance, interfaces, and other common polymorphism-related language features. So we'll look at how look at how this and similar problems can be solved in Odin by other means.


## Compile time polymorphism

It’s helpful to distinguish between ***compile time* polymorphism** and ***runtime* polymorphism**, not just because their implementations differ but but also because they serve quite different purposes. Compile time polymorphism serves two purposes:

- deduplicating code
- overloading names

In Odin, compile time polymorphism is enabled through a few features:

- proc groups
- parametric polymorphic procs
- parametric polymorphic structs
- parametric polymorphic unions
- the `using` modifier for struct fields

### Proc groups

*Proc groups*, very simply, are procedures that are defined not as a body of code but rather as a list of other procedures. At compile time, a call to a proc group dispatches to the procedure in its list that matches the number and types of arguments in the call.

```go
sleep_cat :: proc(cat: Cat) { /* ... */ }
sleep_dog :: proc(dog: Dog) { /* ... */ }

// a proc group 'sleep`
sleep :: proc { sleep_cat, sleep_dog }

sleep(Cat{})       // one Cat argument, so invokes sleep_cat
sleep(Dog{})       // one Dog argument, so invokes sleep_dog
```

Proc groups give us the stylistic and organizational convenience of overloading a procedure name so that we can use a single name at the call sites. Unlike overloading in other languages, however, we still have to give the individual overloads their own names.

### Parametric polymorphic procs (generic functions)

*Parametric polymorphic procedures* are Odin's semi-equivalent of generic functions in other languages. A procedure is parameteric polymorphic if it has any parameters whose arguments and/or types are fixed for each call at compile time.

#### Parameters with compile time arguments

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

To get a similar effect as what other languages call a type parameter, we can use `typeid` parameters that receive compile time arguments. 

> [!NOTE]
> Every unique type in your program is given a unique integer id called a `typeid`. Type names themselves are compile time `typeid` expressions, and the builtin function `typeid_of` returns the `typeid` of its single argument's type, *e.g.* `typeid_of(Cat{})` returns the `typeid` of Cat.

```go
// (slightly simplified version of runtime.new)
// 'T' is a compile time typeid expression, so it can be used like a type name
// Effectively, this one function can return any kind of pointer.
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

#### Parameters with compile time types

When a parameter’s *type* is prefixed with a dollar sign, that indicates that the parameter’s *type* is determined at compile time by the type of the argument:

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

The compile time type establisehd by a parameter can be used as the type of subsequent parameters in the parameter list:

```go
// in each call, 'min' and 'max' will have the same type as 'val'
// ($ should only prefix the first T parameter)
clamp :: proc(val: $T, min: T, max: T) -> T {
    // for the function to compile, 
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

#### Parameters with both compile time arguments *and* compile time types

An individual parameter can have both a compile time argument *and* a compile time type:

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

In some cases, we may wish to restrict which compile time types and arguments are allowed for a function, and we can do this by adding a `where` clause. The `where` clause of a function has a compile time boolean expression which is evaluated for each call of the function. If the expression evaluates false, the call triggers a compilation error.

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

In this case, we could express the same thing using just a `where` clause with no specialization, but the code is then arguably harder to read:

```go
// type_elem_type returns the type of the 
sum :: proc(val: $T) -> T where intrinsics.type_is_numeric(intrinsics.type_elem_type(T)) {
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

The most obvious use case for generic types are collections. For example, a stack:

```go
Stack :: struct($T: typeid) {
    data: [dynamic]T,
}

make_stack :: proc($T: typeid) -> Stack(T) { /* ... */ }

push :: proc(stack: $T/^Stack($E), val: E) { /* ... */ }

pop :: proc(stack: $T/^Stack($E)) -> E { /* ... */ }

// make a stack of ints
s := make_stack(int)  

// push 4 then 7 to the stack
push(&s, 4)
push(&s, 7)

// remove and return last value from the stack (7)
i := pop(&s)               // returns 7
```

> [!NOTE]
> Specialization is often just a shorthand for what can be expressed in a `where` clause, but in the example above, we want to enforce that the second argument of push and that the return type of pop must match the element type of the stack argument, and the only way to express this is with specialization.


### Parameteric polymorphic unions

Like structs, unions can also have compile time `typeid` and unsigned integer parameters:

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
> The main use case for a parapoly union is simply to support parapoly structs with type params in a union: if a variant type in a union has type params, then the union itself must have type params so that its type params can be passed to the variant.


### Struct fields with the `using` modifier

A struct field which is itself of a struct type can be marked with the reserved word `using`. This modifier doesn’t change the structure of the data at all, but it makes the members of the nested struct directly accessible as if they were fields of the containing struct itself:

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


## Runtime polymorphism

Whereas compile time polymorphism enables deduplication of code and overloading of names, runtime polymorphism enables us to have dynamically-typed data (including heterogeneous collections) and to operate upon this dynamically-typed data.

Odin enables runtime polymorphism with a few features:

- unions 
- untyped pointers
- proc references

We've already covered unions and two kinds of untyped pointer (`rawptr`, `uintptr`) (see [Data Types in Odin](odin_data_types.md)), but we haven't yet introduced the third kind of untyped pointer, the `any` type, nor have we introduced proc references. We'll explain how these new things work before discussing how they can enable runtime polymorphism.

### The `any` type

Odin has another kind of untyped pointer called `any`, which is like a `rawptr` but which additionally contains a `typeid`. Every type in the language can be implicitly cast to `any`:

```go
a: any
i: int

a = i        // the any value has a typeid of int and points to i
```

> [!NOTE]
> The cast of `i` to `any` is a bit syntactically inconsistent with the rest of the language because, despite not using the address operator, the resulting `any` value contains the address of the int variable, not its value.

When we cast a pointer expression to `any`, the result holds the `typeid` of the pointer type and holds the same address as the pointer (rather than the address of the pointer itself):

```go
a: any
i: int

a = &i       // the any value has a typeid of ^int and points to i
```

Even more odd, a whole expression can be cast to `any`, in which case the `any` value holds a pointer into the stack frame where the expression result is stored:

```go
a: any
i: int

a = 3 + i          // typeid of int and points into the stack frame

// compile error: 
// unlike a cast to `any`, the & operator only works directly
// on variables, struct fields, and array indexing expressions
ip := &(3 + i) 
```


### Proc references

A proc reference is simply what other languges would call a function pointer:

```go
add :: proc(a: int, b: int) -> int {
    return a + b
}

f: proc(a: int, b:int) -> int
f = add
x := f(3, 5)   // same as calling add
```

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

Runtime polymorphism also requires a way to perform dynamic dispatch, which is not provided by proc groups: 

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
// a parapoly function where T must be a variant of Pet
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

We can get closer to actual runtime dispatch with proc refs:

```go
Cat :: struct{
    sleep_proc : proc(Cat)
}
Dog :: struct{
    sleep_proc : proc(Dog)
}
Pet :: union { Cat, Dog }

// psuedo-methods for each Pet type
sleep_cat :: proc(c: Cat) {}
sleep_dog :: proc(d: Dog) {}

dog : = Dog{ sleep_proc = sleep_dog }
cat : = Cat{ sleep_proc = sleep_cat }

pet: Pet = dog

// we cannot access the .sleep_proc field
// without using a type switch (or type asserts), so we
// are still dispatching on type at compile time, not runtime
switch p in pet {
case Cat:
    p.sleep_proc(p)
case Dog:
    p.sleep_proc(p)
}
```

Even if we create a parapoly proc, the dispatch on a union value's type still happens at compile time:

```go
// a parapoly function where T must be a variant of Pet
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
    sleep_proc: proc(Pet)
    data: any    
}

Dog :: struct{}

sleep_dog :: proc(pet: any) {
    dog := pet.(Dog)
    // ...
}

pet : = Pet{ 
    sleep_proc = sleep_dog, 
    data = Dog{} 
}

// dynamically calls sleep_dog
pet.sleep_proc(pet)
```

Effectively, this pattern establishes an extensible Pet interface: a struct can be said to implement Pet if there is a corresponding `sleep_` proc with the correct signature, *e.g.* a Cat struct implements interface Pet if it has a corresponding `sleep_cat`.

> [!NOTE]
> The method-call syntax familiar from other languages, `x.y()`, has no special meaning in Odin. To invoke `x.y()` simply invokes the proc ref stored in field 'y' of 'x', but no arguments are implicitly passed. Hence, in our example above, the pet variable is passed explicitly.

> [!NOTE]
> Despite its name being so short and convenient, the `any` type is not indented to be commonly used except in a handful of niche use cases (including this interface pattern). In particular, you should prefer using a union over using `any` wherever possible.

For an interface that has multiple psuedo-methods proc refs, it is convenient to bundle the proc refs into a single struct:

```go
Dog :: struct{}

// defines the proc refs of the Pet interface
Pet_Procs :: struct {
    sleep: proc(any)
    eat: proc(any, int) -> int
}

Pet :: struct {
    procs: Pet_Procs
    data: any    
}

dog_procs :: Pet_Procs{
    sleep = proc(pet: any) {
        dog := pet.(Dog)
        // ...
    },
    eat = proc(pet: any, i: int) -> int {
        dog := pet.(Dog)
        // ...
    },
}

pet := Pet{ 
    procs = dog_procs, 
    data = Dog{} 
}

// dynamically calls dog_procs.eat
i: int = pet.procs.eat(pet, 4)
```