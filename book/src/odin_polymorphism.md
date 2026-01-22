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

### Parametric polymoprhic procs (generic functions)

*Parametric polymorphic procedures* are Odin's semi-equivalent of generic functions in other languages. A procedure is parameteric polymorphic if it has any parameters whose arguments and/or types are fixed for each call at compile time.

#### parameters with compile time arguments

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

#### parameters with compile time types

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
> Don't be confused that we used "T" as the name for the typeid parameter name earlier but here now use "T" as the name for the parameter type itself. In the former case, the type "T" is determined by the *typeid value* passed as argument; in the latter case, the type "T" is determined by the *type* of the passed argument.

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

####  parameters with both compile time arguments and compile time types

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

#### where clauses

In some cases, we may wish to restrict which compile time types and arguments are allowed for a function, and we can do this with a where clause. The where clause of a function has a compile time boolean expression which is evaluated for each call of the function. If the expression evaluates false, it triggers a compilation error. 

For an example, we can add a where clause to the clamp function to ensure that the arguments are a numeric type. A call to clamp with int arguments is still OK because int is numeric, but a call with string arguments now fails because strings are not numeric, making the where clause here evaluate false.

#### specialization

Another way to express restrictions of the parameters’ compile time types is with what is called specialization. For the most part, specialization is just shorthand syntax for what you otherwise can express in a where clause, but unlike a where clause, specialization can introduce new type parameters. 

Here for example we have a sum function that returns the sum of all elements in a slice. To ensure that the argument is a slice of a numeric type, we use both specialization and a where clause. Specialization is denoted by a slash after the parameter type, followed by another type. In this case the type after the slash is a slice of $E, where E is an additional type parameter. We couldn’t just use T here because then we’d be saying that T must be a slice of itself, which isn’t logically possible. Instead, we introduce an additional type parameter so that it can be some other type. The additional type parameter is then used in the where clause to test if the slice element type is numeric.

(By the way, ‘E’ stands for Element, as in ‘element of a slice’, so it is the conventional name in this situation.) 

Without specialization, we could still express the same thing using just a where clause, but the code is then arguably harder to read.



### parameteric polymorphic structs

What Odin calls a parametric polymorphic struct is a near equivalent of what other languages would call a generic struct (or a templated struct in C++). Here the struct Cat has two parameters, T and U, both typeids, that function effectively as type parameters.

The dollar sign on the parameter names indicates that the values must be compile time expressions. When we use the Cat type, we must then provide compile time type arguments.

Here we’re declaring two Cat variables, one for which T is an f32 and U is a string, and another variable for which T is an int and U is a string. Because they are composed of different types, they are actually completely distinct struct types that just share the same name, so we cannot assign one of these variables to the other, and there is no option to cast between them.

Also be clear that Cat with no type arguments is not a valid type.

Aside from compile type typeids, another kind of parameter we can give a struct is a compile time integer value, which is useful for specifying sizes of arrays in the struct. Here for example, the parameter N is used to specify the size of the array field c. Like with the typeids arguments, structs with different integer arguments are considered to be completely distinct types with different sizes and memory layouts.

A parametric polymorphic struct can optionally have a where clause. A where clause contains a compile time boolean expression that is evaluated for each variant of the type, and if the expression evaluates false, the variant is rejected by the compiler. Here for example, the where clause specifies that N must be less than 10, and so a Cat where N is 6 is OK, but a Cat where N is 11 triggers a compilation error.

polymorphism via parapoly struct

The most obvious use case for generic types are collections. Here for example we define a parapoly struct named Stack that is composed of a dynamic array of T. Odin has no concept of methods, but we can define parapoly procedures to implement the methods of this stack. The Make_stack procedure fills the role of a constructor which simply takes the desired type as argument, while push and pop both require a pointer to the stack. This is expressed as a type parameter which is specialized to be a Stack of type E, where E itself is another type parameter. As we said earlier, specialization is often just a shorthand for what can be expressed in a where clause, but in these cases, we want to enforce that the second argument of push and the return type of pop must match the element type of the stack argument, and the only way to express this is with specialization.

Anyway, having defined this stack type and its operations, we can then create a stack with any kind of element without having to write more than one stack definition.

Aside from collections, another possible use case for a parapoly struct is to imitate inheritance, at least in part. This Pet struct defines a few fields common to all pets but then also has a field whose type is plugged in from the type parameter T.  If we then define structs to represent the data that is unique for each specific kind of pet, we can make specific variants of Pet. Here a dog is represented as a Pet with Dog_Data, and a cat is represented as a Pet with Cat_Data. Effectively, then, the data field of a dog has dog-specific data while the data field of a cat has cat-specific data.

To define a function that operates on any kind of Pet, we again can define parapoly functions. Here the sleep function takes any argument which can be any variant of Pet.

Now is this actually a good solution? Maybe not. Unlike proper inheritance in other languages, this solution doesn’t allow for runtime dispatch, nor does it allow for anything like method overriding.

### parameteric polymorphic unions

Like structs, unions can also have compile time typeid and integer parameters, which effectively allows us to create variants from a single union definition.

Here we create a variable of type Pet, where the variant types are f32, int, and 4-string arrays. We can then assign any value of these types to the variable without an explicit cast.

Just be clear, like with parametric polymorphic structs, each unique variant is a distinct type, and thus, say here, a Pet union with 6-string arrays as a variant type is distinct from a Pet union with 4-string arrays as a variant type.

polymorphism via parapoly union

One more mechanism in Odin for compile time polymorphism is parapoly unions. Compared to parapoly structs, the use cases for parapoly unions are less obvious aside from two:

First, a Result type, as it’s usually called, is a way of expressing the result of an operation that might not be available because an error occurred. Here this Result union is either a value of type parameter T or an Error. By virtue of the type parameter, we can define Result just once instead of separately for every possible kind of result we want to use.

In practice, however, Odin code generally won’t make use of a Result type because the Odin convention is to separately return errors from multi-return functions instead of mixing errors with other values in a union.

Probably, then, the only real use case for parapoly unions in Odin is for cases where a union includes other parapoly types. Here this Pet union includes variant types Dog and Cat which themselves are parapoly structs, and so in order to pass on type arguments to these variants, the union itself must have type parameters.

Outside of these cases, though, uses for parapoly unions are generally quite rare.






### polymorphism via using in a struct

Another way in Odin that you might try to approximate type inheritance is through simple composition. Here we’ve inverted the pattern of the prior example: instead of the Pet containing a more specific kind of Pet, the specific types of Pet themselves contain an instance of Pet. Effectively, both Cat and Dog can have their own unique data while sharing in common the data that defines a Pet. As we did with the parapoly struct, we can define a parapoly function that accepts any kind of pet as input. 

We can improve on this pattern a bit, at least stylistically, by adding the using modifier to the pet fields of our Cat and Dog structs. This simply makes the members of the Pet type directly accessible as if they were members of the Cat or Dog directly, even though in truth the layout of the data remains the same. With this change, the sleep function as we wrote it still works the same, but we can also access the name field directly on the argument, which remember will concretely be either a Cat or Dog. Again, this is just a stylistic affordance of the compiler: nothing about the data or executing code has actually changed here.

Alternatively, though, thanks to these using modifiers, the compiler will let us use a Cat or Dog as a proxy for a Pet value, so instead of making this function generic, we can define it to take a Pet argument directly but then pass a Cat or Dog as proxy for a Pet. In terms of actual effect, this is exactly the same as passing the pet field explicitly, but the shorthand is arguably a nice stylistic affordance.

Be clear, though, that once again this is just a compile time trick that does nothing to support runtime dispatch.





```go
// T must be a variant of the Pet union
sleep :: proc(pet: $T) where intrinsics.type_is_variant_of(Pet, T) {      
    fmt.println(pet.name, “is sleeping“)
}


sleep(Cat{})             // OK if Cat has .name
sleep(Dog{})             // OK if Dog has .name

pet: Pet
sleep(pet)               // compile error Pet does not have .name
```


Here this function sleep takes a single argument with a compile time type, so it effectively can be called with any kind of argument as long as the function can properly compile with that type. Because the function accesses a field called name on the argument, an acceptable argument must be a struct that has such a field. Here then, calls with Cat and Dog arguments are OK as long as the Cat and Dog structs have a .name field.

Optionally, we can further narrow the set of allowed argument types so as to discourage erroneous uses of the function. Here we’re using a where clause to require that T must be a variant of the Pet union. Assuming Cat and Dog are variants of this union, then these two calls are still valid.

However, a Pet value itself is not a variant of Pet, so we cannot call sleep with a Pet argument. 

Like with proc groups, then, parametric polymorphic functions do not enable runtime dispatch. A unique variant of the function is generated for each unique argument type, but which specific variant gets called at each call site is fully locked in at compile time. 



## Runtime polymorphism

Whereas compile time polymorphism enables deduplication of code and overloading of names, runtime polymorphism enables us to have dynamically-typed data (including heterogeneous collections) and to operate upon this dynamically-typed data.

Odin enables runtime polymorphism with a few features:

- untyped pointers 
- unions 
- proc references


The burden of static typing is that the compiler demands to know the specific type of every variable, every field, and every expression, but runtime polymorphism opens a door where a single compile time type may encompass a set of possible types at runtime.



While untyped pointers offer ultimate flexibility, they undermine any pretensions of static type safety, so generally unions are the preferred option.

When it comes time to use dynamically-typed data, code very often needs to perform runtime type checks (such as Odin’s type assertions and type switches). This suffices for most purposes, but in more advanced cases, we may need one more mechanism: procedure references, which are Odin’s analogue of function pointers. As we’ll see later, procedure references can help us fully simulate the inheritance and interface features of other languages which Odin does not directly support.

First, though, we’ll discuss how untyped pointers and unions can represent dynamically-typed data.

polymorphism via untyped pointers

Once again we’ll consider the use case of a Pet type with pseudo-subtypes Cat and Dog. If our Pet struct contains a field data of type any, then it can contain any kind of Pet, whether a Cat or Dog, or in fact anything whatsoever.  When we now implement operations on our Pet type, we can use a type switch to handle all the possible values of the data field, like this sleep function that can process both Cats and Dogs.

Unlike all of our prior solutions using just compile time polymorphism, we can finally create collections that contain a heterogeneous mix of any kind of Pet and process them based on their subtype. Here we invoke sleep on all Pets in this array, which can contain any mix of Cats and Dogs or other Pets.

Be clear, though, this still isn’t proper dynamic dispatch because the set of possible Pet types that can sleep is hardcoded in the typeswitch. With actual dynamic dispatch, the set of types is open for extension, even across program module boundaries.

Another problem here, as mentioned, is that an untyped pointer is too unrestrictive: the data field of our Pet type can contain literally anything even though we actually only want it to contain Cats or Dogs or other valid subtypes of Pet. Potentially, then, users of this Pet struct may end up assigning inappropriate values to the data field, and the compiler will not identify these errors.


### any

Odin has one more pointer type called `any`, which is like a `rawptr` but which additionally contains a `typeid`.

Every type in the language can be implicitly cast to any, and what we get back in this case is a value of type any that contains the typeid of int and the address of i.

Note that this is actually a bit odd and inconsistent with the rest of the language, syntactically, because despite not using the address operator, the resulting any value contains the address of the int variable, not its value.

The reason for this inconsistency is that it allows casting from a pointer to produce an any value with the same address but the typeid of the pointer type. So here when the pointer to i is cast to any, the resulting value has the typeid of int pointer instead of int. 

Even more strange, we can actually cast an arbitrary expression into an any, and the compiler will allocate space on the call stack for the resulting value. Here the result of the expression 3 + i will be stored somewhere in the call stack frame, and the any value assigned to this variable ‘a’ will point to that location.


In contrast, the address operator can only be used on variables and array indexing expressions, not just any expression. If we try to use the address operator on this expression to get an int pointer, we’ll just get a compilation error.


polymorphism via a union

The better solution, in most cases, is to use a union type instead of an untyped pointer. Here now the data field is this Pet_Data union type, which has variants Cat and Dog. If the Cat and Dog types are significantly large, we might prefer to instead define its variants as pointer to Cat and pointer to Dog. Either way, we can still use a typeswitch to handle the different kinds of Pet, but now the compiler can assure us that this data field will only ever be a Cat or Dog rather than just anything.

One more possible solution is that we flip the composition: rather than a Pet struct containing Cat or Dog data, Cat and Dog structs can contain Pet data. When we use a type switch now, we switch on the Pet value itself because Pet is directly the union with variant types Cat or Dog.

So if unions are so great, why would we ever use untyped pointers instead? Well it’s important to note that a union’s set of variants is fixed at compile time and therefore the importers of a package cannot add variants to the package’s unions. For example, if a dependency we’re importing defines a Pet union, we cannot add our own variants of Pet.

The other issue with unions is that we can only process their values with type asserts and type switches, both of which are hardcoded, in the sense that they cannot be extended: for example, if a function in my package uses a typeswitch, importers of my package cannot add new cases to the typeswitch.

Ultimately, the only way in Odin to have truly dynamic data is to use untyped pointers.


polymorphism via function pointers

To process fully dynamic data, we need proc references, which are Odin’s equivalent of C function pointers. 

Let’s say that we again have a Pet struct with two subtypes, Cat and Dog, which both embed a Pet struct with the using modifier. To allow each instance of pet to customize how it performs its sleep operation, we’ll add a sleep field to Pet which is a proc ref. The proc requires a parameter to receive the pet data, and because the pet might be a Cat, Dog, or any other kind of Pet, the parameter must be of type any.

When we then define sleep functions for cat and dog, we expect them to only be called with their respective type of pet, so first thing, these functions both type assert that their parameters are actually Cats and Dogs. We could try to handle the error case for these type asserts, but we actually want to panic in the event of such errors anyway. We should just ensure in our code that cat_sleep is only called with Cat arguments and dog_sleep is only called with Dog arguments.

So here now, when we create a cat value, we assign its sleep field the appropriate proc reference to the cat_sleep function, and so then we can invoke cat_sleep by name or via the sleep field.

Likewise for Dogs.


So now we finally have a form of dynamic function call because calling a proc reference invokes the function that it currently stores.

However, our Pet data itself is not dynamic at all, and so we still can’t in this arrangement create a heterogeneous collection of Pets on which we could invoke each pet’s respective sleep operation. If for example we create a Pet array, that array can only contain Pet structs, not Cat or Dog structs.

If we use a union of the pet types, that allows us to create a heterogenous collection of pets, but it creates two new problems:

First, the set of Pet_Union variants is fixed in this definition, which is fine as long as you can freely edit the union definition, but that is something which downstream code that imports this package cannot do.

Second, there’s no way to access the sleep proc references of the pets without first doing a type assert or type switch, which defeats our whole purpose.

Instead of a union then, we need a struct, which we’ll call Pet_Wrapper, that stores any kind of Pet while also separately storing its sleep proc ref. Also as a convenience, we’ll define a function that takes any kind of Pet and creates a Pet_Wrapper.

Now we can create a heterogeneous collection of pets which, unlike the union solution, is open for extension with new types of Pet, and which allows us to invoke the sleep operation without a type assert or type switch.

Here we create a Cat and Dog, which we wrap and store in an array, and then we can invoke sleep on every pet in the array.

So this finally is working dynamic dispatch in Odin. It’s certainly not as convenient as the inheritance and interface mechanisms built into other languages, but it does solve the problem. So you might ask, why doesn’t Odin support these features directly? Well the philosophy is that these things are not actually commonly needed and so shouldn’t be presented as a default in the language. In the large majority of cases where you might use dynamic dispatch, the code will be simpler and better performing if you don’t.

Before we end, one quick tweak of this dynamic dispatch pattern:

For cases where our Pets have many possible operations beyond just sleep, our structs would get bloated if they have to store many proc references. So rather than store a bunch of individual proc refs, we can instead group all of them into another struct which we then store by pointer. In this example, the sleep proc reference is moved into a Pet_Procs struct, which is stored as a pointer in the Pet struct. If we want additional operations for our pets, we then add additional proc reference fields in Pet_Procs. 

For each pet type, we’ll create a global Pet_Procs containing all the operations for that type of Pet…

…and when we create our Cat and Dog instances, we assign pointers of these globals to their pet_procs fields.

The Pet_Wrapper now includes a Pet_Procs pointer, and the invocation code is slightly more verbose, but it’s still the same idea, just now the pet types can have all of their operation proc references grouped into one struct.
