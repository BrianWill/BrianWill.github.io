# Polymorphism in Odin

This text accompanies two videos that cover polymorphism in the Odin programming language:

- [Compile-time Polymorphism in Odin]()
- [Runtime Polymorphism in Odin]()

See also the pre-requisite topic, [Data Types in Odin](odin_data_types.md)


# parameteric polymorphic structs

What Odin calls a parametric polymorphic struct is a near equivalent of what other languages would call a generic struct (or a templated struct in C++). Here the struct Cat has two parameters, T and U, both typeids, that function effectively as type parameters.

The dollar sign on the parameter names indicates that the values must be compile time expressions. When we use the Cat type, we must then provide compile time type arguments.

Here we’re declaring two Cat variables, one for which T is an f32 and U is a string, and another variable for which T is an int and U is a string. Because they are composed of different types, they are actually completely distinct struct types that just share the same name, so we cannot assign one of these variables to the other, and there is no option to cast between them.

Also be clear that Cat with no type arguments is not a valid type.

Aside from compile type typeids, another kind of parameter we can give a struct is a compile time integer value, which is useful for specifying sizes of arrays in the struct. Here for example, the parameter N is used to specify the size of the array field c. Like with the typeids arguments, structs with different integer arguments are considered to be completely distinct types with different sizes and memory layouts.

A parametric polymorphic struct can optionally have a where clause. A where clause contains a compile time boolean expression that is evaluated for each variant of the type, and if the expression evaluates false, the variant is rejected by the compiler. Here for example, the where clause specifies that N must be less than 10, and so a Cat where N is 6 is OK, but a Cat where N is 11 triggers a compilation error.



## Parametric polymoprhic procs (generic functions)


### parameteric polymorphic unions

Like structs, unions can also have compile time typeid and integer parameters, which effectively allows us to create variants from a single union definition.

Here we create a variable of type Pet, where the variant types are f32, int, and 4-string arrays. We can then assign any value of these types to the variable without an explicit cast.

Just be clear, like with parametric polymorphic structs, each unique variant is a distinct type, and thus, say here, a Pet union with 6-string arrays as a variant type is distinct from a Pet union with 4-string arrays as a variant type.


### compile time arguments

Like we’ve already seen with structs and unions, functions in Odin can have compile time arguments, as denoted by a dollar sign prefix on the parameter name. 

Here, for example, this function foo has a single parameter, an int named x which has the $ prefix, meaning it requires a compile time expression for its argument. So if we call this function with a number literal, that’s valid, but if we try to pass the function a variable, we get a compilation error.

One way a compile time argument can be useful is to specify array sizes. Here this function returns an array whose size is determined by a compile time argument.

Another way a compile time argument might help is if they allow some expressions to be evaluated at compile time. Here if the second argument, val, is also a compile time argument, then the calculation of val * val can be computed at compile time, thus avoiding some work at runtime.

Lastly, we can use compile time typeids as a way to specify types, much like we saw with structs and unions. Here this function returns a pointer to T, where T is determined by a compile time argument, and thus this single function can effectively return any kind of pointer.

By the way, if you’re wondering about the implementation, the simplest explanation is that each function call with a unique combination of compile time arguments necessitates the compiler to generate a variant of the function. For instance, if our program calls this function sometimes with int and sometimes with string, those calls will invoke two different compiled versions of the function: one for ints and one for strings.

The same can be said of structs and unions with compile time parameters: each unique combination of arguments necessitates the compiler to generate a variant of the type.

In this way, you could think of this feature as analogous to templates in C++ and similar meta-programming features in other languages.


### compile time parameter types

When a parameter’s type is prefixed with a dollar sign, that indicates that the parameter’s type is determined at compile time by the type of the argument.

Here for example we have a function repeat_five, which has a single parameter of type $T and returns an array of size 5 and type T.  In the function, the argument value is assigned to every index of the returned array.

If our program calls this function with, say, a bool value, the compiler will generate a version of the function for which T is bool, and if our program calls this function with a string value, the compiler will generate a version of the function for which T is string. So here the first call, which passes true, returns an array of 5 bools, all set to true, and the second call, which passes a string “hi”, returns an array of 5 strings, all set to that string “hi”.

For another example, here’s a function clamp where the first parameter has type $T, and the other two parameters use the same type T. If instead we wanted the function to have different compile time types that didn’t necessarily match, we could give the other parameters their own dollar sign types, but in this case, we want the parameter types all to match. As a rule, the dollar sign should go on the first T in the parameter list and only the first T; otherwise we’ll get a compile error.

Anyway, if we then call this clamp function with int arguments, then the compiler invokes a version of clamp in which T is int, and so the call returns an int. Likewise, if we call clamp with f32 arguments, then the compiler invokes a version of clamp in which T is f32, and so the call returns an f32. 

In both of these calls, the compiler infers the type of T from the first argument and then expects the types of the other two arguments to match. So the untyped integers 2 and 5 are being implicitly cast to int in the first call and f32 in the second call.

In the case of the first call, we could actually leave out the cast, because the untyped integer 8 would be presumed to represent an int in this context.

Now, if we try calling this function with most non-numeric types, like bool, we’ll get compile errors because the version of clamp where T is bool cannot compile: bools cannot be compared with the less than or greater than operators. 

The only non-numeric types that work for T here are strings because strings in Odin actually can be compared with the less than and greater than operators. So this call is valid, even though calling clamp with strings doesn’t really produce a meaningful result.


### using a parameter with both a compile time value and compile time type

An individual parameter of a function can have both a compile time argument and a compile time type, and this is useful in a few cases.

Here we have a function with a single parameter where both the name and type have a dollar sign, so both the argument value and type are determined for each call at compile time.

Because the parameter n is used to specify an array size, it must be an integer. In this first example call, the argument is an int value 3, so the function returns a pointer to an array of 3 ints. In this second example call, the argument is a u8 value 5, so the function returns a pointer to an array of 5 u8s.

(Note, again, that an untyped integer is assumed to be an int when passed to a parameter with a compile time type.)

[todo less artificial example? something more useful?]


### where clauses

In some cases, we may wish to restrict which compile time types and arguments are allowed for a function, and we can do this with a where clause. The where clause of a function has a compile time boolean expression which is evaluated for each call of the function. If the expression evaluates false, it triggers a compilation error. 

For an example, we can add a where clause to the clamp function to ensure that the arguments are a numeric type. A call to clamp with int arguments is still OK because int is numeric, but a call with string arguments now fails because strings are not numeric, making the where clause here evaluate false.

Another way to express restrictions of the parameters’ compile time types is with what is called specialization. For the most part, specialization is just shorthand syntax for what you otherwise can express in a where clause, but unlike a where clause, specialization can introduce new type parameters. 

Here for example we have a sum function that returns the sum of all elements in a slice. To ensure that the argument is a slice of a numeric type, we use both specialization and a where clause. Specialization is denoted by a slash after the parameter type, followed by another type. In this case the type after the slash is a slice of $E, where E is an additional type parameter. We couldn’t just use T here because then we’d be saying that T must be a slice of itself, which isn’t logically possible. Instead, we introduce an additional type parameter so that it can be some other type. The additional type parameter is then used in the where clause to test if the slice element type is numeric.

(By the way, ‘E’ stands for Element, as in ‘element of a slice’, so it is the conventional name in this situation.) 

Without specialization, we could still express the same thing using just a where clause, but the code is then arguably harder to read.


## polymorphism


### any and typeid

Odin has one more pointer type called any, which is like a rawptr but which additionally contains a typeid. What is a typeid?

Well when you compile an Odin program, every unique type in your program is given a unique integer id called a typeid. In your code, you can get the typeid of any type by calling the builtin function typeid_of . In this example, we’re assigning the typeid of bool pointer to a typeid variable named t.

So anyway, back to any. Every type in the language can be implicitly cast to any, and what we get back in this case is a value of type any that contains the typeid of int and the address of i.

Note that this is actually a bit odd and inconsistent with the rest of the language, syntactically, because despite not using the address operator, the resulting any value contains the address of the int variable, not its value.

The reason for this inconsistency is that it allows casting from a pointer to produce an any value with the same address but the typeid of the pointer type. So here when the pointer to i is cast to any, the resulting value has the typeid of int pointer instead of int. 

Even more strange, we can actually cast an arbitrary expression into an any, and the compiler will allocate space on the call stack for the resulting value. Here the result of the expression 3 + i will be stored somewhere in the call stack frame, and the any value assigned to this variable ‘a’ will point to that location.


In contrast, the address operator can only be used on variables and array indexing expressions, not just any expression. If we try to use the address operator on this expression to get an int pointer, we’ll just get a compilation error.



Now that we’ve covered the concrete mechanisms of Odin’s type system, it’s useful to summarize how the various options in the language can enable polymorphism. This is an important topic because Odin diverges in this area from other, more widely-known languages, such as Java and C#. In these other languages, polymorphism is enabled through features like inheritance, interfaces, overloading, overriding, and generics, but Odin outright omits most of these features and offers inexact substitutes for the rest.

First, though, what exactly does the umbrella term ‘polymorphism’ mean that it can encompass so much? Well wikipedia puts it succinctly: “...polymorphism allows a value type to assume different types.” Where otherwise a single thing would only expect one concrete type or behave in one way, it can be made to expect potentially multiple types and behave in different ways.

Or another way to think of it: mechanisms of polymorphism allow us to express that a piece of code has variants that are similar but somehow different. In this sense, mechanisms of polymorphism enable abstraction building beyond what just plain functions and plain data types allow. Polymorphism gives us additional ways to generalize.
Now, how much one should attempt to generalize—to abstract—in code is a very debatable question, but there clearly are some expressions of polymorphism that are useful to have in your toolset. One very common case is the need to express heterogenous data in a collection and then operate upon all elements of the collection. In C#, for example, we can create an array of Pets that may store any kind of Pet, whether a Cat or Dog, and then we can iterate over every element of the array and perform a common operation on every Pet regardless of its concrete type. This is enabled either by virtue of Cat and Dog inheriting from class Pet or by Cat and Dog implementing an interface Pet. Odin, however, lacks both inheritance and interfaces as language features, so we’re going to look at how this and similar problems can be solved in Odin by other means.



## Compile time polymorphism

When we look at the options for polymorphism in a language, it’s helpful to distinguish between compile time polymorphism and runtime polymorphism, not just because of how they are implemented but also because they serve different purposes.

First we’ll look at compile time polymorphism, which serves two purposes: deduplicating code and overloading names.

In Odin, compile time polymorphism is enabled through a few features: proc groups, parapoly functions, structs, and unions, and the using modifier which can be applied to fields of structs.


polymorphism via proc groups

Proc groups, very simply, are procedures that are defined not as a body of code but rather as a list of other procedures. At compile time, a call to a proc group dispatches to the procedure in its list that matches the number and types of arguments in the call.

For example, here we have a proc group named sleep, which lists two procedures: sleep_cat and sleep_dog, which importantly take different types of argument. When we then call sleep with a Cat argument, it resolves at compile time as a call to sleep_cat, and likewise when we call sleep with a Dog argument, it resolves at compile time as a call to sleep_dog.

Really then, proc groups just give us the stylistic and organizational convenience of overloading a procedure name so that we can use a single name at the call sites. Unlike overloading in other languages, however, we still have to give the individual overloads their own names.

Again be clear that proc groups do not enable runtime dispatch: if we have, say, a union that can be either a Cat or Dog, then to invoke the appropriate sleep operation, we have to use a type switch and separately call sleep for each case.

As we’ll see later, proper runtime dispatch will require other mechanisms.

polymorphism via parapoly function

Parametric polymorphic functions offer another kind of compile time polymorphism. Here this function sleep takes a single argument with a compile time type, so it effectively can be called with any kind of argument as long as the function can properly compile with that type. Because the function accesses a field called name on the argument, an acceptable argument must be a struct that has such a field. Here then, calls with Cat and Dog arguments are OK as long as the Cat and Dog structs have a .name field.

Optionally, we can further narrow the set of allowed argument types so as to discourage erroneous uses of the function. Here we’re using a where clause to require that T must be a variant of the Pet union. Assuming Cat and Dog are variants of this union, then these two calls are still valid.

However, a Pet value itself is not a variant of Pet, so we cannot call sleep with a Pet argument. 

Like with proc groups, then, parametric polymorphic functions do not enable runtime dispatch. A unique variant of the function is generated for each unique argument type, but which specific variant gets called at each call site is fully locked in at compile time. 


polymorphism via parapoly struct

The most obvious use case for generic types are collections. Here for example we define a parapoly struct named Stack that is composed of a dynamic array of T. Odin has no concept of methods, but we can define parapoly procedures to implement the methods of this stack. The Make_stack procedure fills the role of a constructor which simply takes the desired type as argument, while push and pop both require a pointer to the stack. This is expressed as a type parameter which is specialized to be a Stack of type E, where E itself is another type parameter. As we said earlier, specialization is often just a shorthand for what can be expressed in a where clause, but in these cases, we want to enforce that the second argument of push and the return type of pop must match the element type of the stack argument, and the only way to express this is with specialization.

Anyway, having defined this stack type and its operations, we can then create a stack with any kind of element without having to write more than one stack definition.

Aside from collections, another possible use case for a parapoly struct is to imitate inheritance, at least in part. This Pet struct defines a few fields common to all pets but then also has a field whose type is plugged in from the type parameter T.  If we then define structs to represent the data that is unique for each specific kind of pet, we can make specific variants of Pet. Here a dog is represented as a Pet with Dog_Data, and a cat is represented as a Pet with Cat_Data. Effectively, then, the data field of a dog has dog-specific data while the data field of a cat has cat-specific data.

To define a function that operates on any kind of Pet, we again can define parapoly functions. Here the sleep function takes any argument which can be any variant of Pet.

Now is this actually a good solution? Maybe not. Unlike proper inheritance in other languages, this solution doesn’t allow for runtime dispatch, nor does it allow for anything like method overriding.

polymorphism via using in a struct

Another way in Odin that you might try to approximate type inheritance is through simple composition. Here we’ve inverted the pattern of the prior example: instead of the Pet containing a more specific kind of Pet, the specific types of Pet themselves contain an instance of Pet. Effectively, both Cat and Dog can have their own unique data while sharing in common the data that defines a Pet. As we did with the parapoly struct, we can define a parapoly function that accepts any kind of pet as input. 

We can improve on this pattern a bit, at least stylistically, by adding the using modifier to the pet fields of our Cat and Dog structs. This simply makes the members of the Pet type directly accessible as if they were members of the Cat or Dog directly, even though in truth the layout of the data remains the same. With this change, the sleep function as we wrote it still works the same, but we can also access the name field directly on the argument, which remember will concretely be either a Cat or Dog. Again, this is just a stylistic affordance of the compiler: nothing about the data or executing code has actually changed here.

Alternatively, though, thanks to these using modifiers, the compiler will let us use a Cat or Dog as a proxy for a Pet value, so instead of making this function generic, we can define it to take a Pet argument directly but then pass a Cat or Dog as proxy for a Pet. In terms of actual effect, this is exactly the same as passing the pet field explicitly, but the shorthand is arguably a nice stylistic affordance.

Be clear, though, that once again this is just a compile time trick that does nothing to support runtime dispatch.


polymorphism via parapoly union

One more mechanism in Odin for compile time polymorphism is parapoly unions. Compared to parapoly structs, the use cases for parapoly unions are less obvious aside from two:

First, a Result type, as it’s usually called, is a way of expressing the result of an operation that might not be available because an error occurred. Here this Result union is either a value of type parameter T or an Error. By virtue of the type parameter, we can define Result just once instead of separately for every possible kind of result we want to use.

In practice, however, Odin code generally won’t make use of a Result type because the Odin convention is to separately return errors from multi-return functions instead of mixing errors with other values in a union.

Probably, then, the only real use case for parapoly unions in Odin is for cases where a union includes other parapoly types. Here this Pet union includes variant types Dog and Cat which themselves are parapoly structs, and so in order to pass on type arguments to these variants, the union itself must have type parameters.

Outside of these cases, though, uses for parapoly unions are generally quite rare.




## Runtime polymorphism


Finally, now, what about runtime polymorphism? Whereas compile time polymorphism enables deduplication of code and overloading of names, runtime polymorphism enables us to have dynamically-typed data, including heterogeneous collections.

The burden of static typing is that the compiler demands to know the specific type of every variable, every field, and every expression, but runtime polymorphism opens a door where a single compile time type may encompass a set of possible types at runtime.

In Odin, dynamically-typed data can be expressed in one of two ways: 

either with untyped pointers 
or with unions. 

While untyped pointers offer ultimate flexibility, they undermine any pretensions of static type safety, so generally unions are the preferred option.

When it comes time to use dynamically-typed data, code very often needs to perform runtime type checks (such as Odin’s type assertions and type switches). This suffices for most purposes, but in more advanced cases, we may need one more mechanism: procedure references, which are Odin’s analogue of function pointers. As we’ll see later, procedure references can help us fully simulate the inheritance and interface features of other languages which Odin does not directly support.

First, though, we’ll discuss how untyped pointers and unions can represent dynamically-typed data.

polymorphism via untyped pointers

Once again we’ll consider the use case of a Pet type with pseudo-subtypes Cat and Dog. If our Pet struct contains a field data of type any, then it can contain any kind of Pet, whether a Cat or Dog, or in fact anything whatsoever.  When we now implement operations on our Pet type, we can use a type switch to handle all the possible values of the data field, like this sleep function that can process both Cats and Dogs.

Unlike all of our prior solutions using just compile time polymorphism, we can finally create collections that contain a heterogeneous mix of any kind of Pet and process them based on their subtype. Here we invoke sleep on all Pets in this array, which can contain any mix of Cats and Dogs or other Pets.

Be clear, though, this still isn’t proper dynamic dispatch because the set of possible Pet types that can sleep is hardcoded in the typeswitch. With actual dynamic dispatch, the set of types is open for extension, even across program module boundaries.

Another problem here, as mentioned, is that an untyped pointer is too unrestrictive: the data field of our Pet type can contain literally anything even though we actually only want it to contain Cats or Dogs or other valid subtypes of Pet. Potentially, then, users of this Pet struct may end up assigning inappropriate values to the data field, and the compiler will not identify these errors.

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
