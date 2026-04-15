# Procedural Programming HOWTO

want to give quick positive description of the procedural approach, but need to briefly recap how it differs from other ways of thinking

# OOP critique recap

problems:

Over-enthusiasm for fine-grained modularity:

Trade complexity within modules for complexity between modules:
    low tolerance for intra-module complexity
    high tolerance for inter-module complexity

Conflate data types and modules:
    all data types are modules
    all modules are data types

====
 *Lack of appreciation for flat, sequential code and data*

 easiest mental model, easiest to reason about and debug
 ====


Performance costs of OOP

// only mention these as they come up in contrast to procedural, e.g. mention scattered data layout when describing good data design


On the downside, OOP tends to incur a number of performance costs:
— Scattered data layout: OOP code is often split into many small objects, and the data
often ends up scattered throughout memory (which leads to cache inefficiencies, as
discussed in prior sections)
— Excessive abstraction: Object-oriented design often encourages layers of delegation,
where the higher levels defer the real work to lower levels, resulting in many objects and
methods that do little actual work
— Complex call chains: Thanks to the many layers of abstraction and a preference for
small functions, call chains get very complex
— Virtual calls: Not only do virtual dispatch tables incur overhead over regular function
calls, virtual calls cannot normally be inlined (though some JIT compilers may do so at
runtime)
— Bad allocation patterns: The complex code paths and tangled data relationships that OOP encourages often make it
difficult to reason about object lifetimes, so OOP code tends to rely upon frequent, small
allocations and garbage collection rather than more efficient alternatives
— One-at-a-time processing: Because the code which directly manipulates an object is
part of the object itself, there’s a natural tendency in OOP to process objects one-by-one
rather than in large batches



1 Entangling your data and code makes both of them messier and more complicated

2 Replacing concentrated complexity with scattered complexity increases the overall complexity

3 Objects make it difficult to track which code accesses which data


# mindset


procedural mindset…(but not really about procedural vs. OOP, per se)
pessimistic about fine-grained modularization
pessimistic about code reuse
pessimistic about creating abstractions
pessimistic about using abstractions


tolerant of rewriting code
    exploratory problem solving
    not worried about future, just solve for the problem as currently defined (even if consciously truncated)
    don’t speculate about future needs
    if the requirements change, change the data/code
    good writing is rewriting

APIs != normal code


# data transformation

translating problems that don’t seem like data transformation problems into data transformation

data A → code → data B

what’s example of something that doesn’t seem like a data transformation problem? graphics? game logic? simulation agent behaviour?    

data pipeline model
    data assembly line
    clean macro > clean micro
    an individual function is a mini-pipeline
        macro-structure that most closely mirrors the one proven unit of code abstraction: the function
    example of factorio bus assembly line


data pipeline spaghetti
    still have to worry about sequencing of how the pipeline is fed:
        event sequences
        logic over multiple frames in game loop
            cannot be captured by the pipeline that defines the frame

## Good function design

no data mutation of arguments
no read of globals
no write of globals
no i/o
no alloc
no sync
no async coloring
no exceptions
    no returned errors?

shallow call stacks

minimize scope of data access
    don't pass in things that aren't actually needed
    the larger the scope of data accessed by a function, the simpler its direct logic should be (farm out work to helper functions)
    data scope should generally narrow as you go further down the call stack

    using globals or passing more than you actually need makes it hard to audit the codebase when you need to know:
        1. what data is touched by a certain piece of code?
        2. what parts of code touch a certain piece of data?



## Good data design

spectrum of persistent data to transitory data
    at very least, don’t store transitory state in globals
    pass minimal set of global state up the chain
        similar to argument about exceptions: hassle of returning errors up full chain

clean data > clean code

good usually = simple
    maybe not always simplest option, but generally simple

flat, minimize hierarchy and graphs
    example of factorio bus assembly line

reference into structures by index/keys rather than address

avoid redundancies
    consider the minimal, most compact encoding of the information

## sequences > hierarchies > graphs

common theme about both code design and data design


sometimes you do need hierarchies and graphs
    e.g. hierarchical data and distributed systems
    ...but avoid them when you can
        e.g. do you want to have to think about complex type hierarchies AND runtime object graph AND deep call stacks?
            classic Java style imposes this kind of burden

    ... when you do have hierarchies and graphs, try to keep them simple
        e.g. keep call stack shallow









# DISCARD

pessimistic about dependencies
pessimistic about compilers / toolchains

tolerant of some repetition
tolerant of data in code
    tolerant of code that resembles data: repetition that could be extracted out with macros or other abstractions



purported advantages:

The theoretical benefits of OOP include:
— Composability: Programs made out of objects can be incrementally assembled and
modified
— Reconfigurability: Features can be easily added, removed, and modified by inserting,
removing, and replacing objects
— Code reuse: Objects can be easily reused between programs.
— Intuitiveness: Real-world things and processes naturally correspond to objects.
— Abstraction: Objects allow the programmer to solve problems at a high-level without
being distracted by low-level details