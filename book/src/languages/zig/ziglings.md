# Zig Intro - Code Examples (Ziglings)

![Under construction](../../construction.gif)

This text is a supplement to a [video](https://ziglang.org/) that introduces the Zig programming language by walking through small code exercises from the [Ziglings project](https://codeberg.org/ziglings/exercises/#ziglings).

This walkthrough assumes the audience has reasonable familiarity with C or other similar languages (*e.g.* C++, Rust, Odin, or Go). If you're new to this kind of programming, it may help to first check out my [Odin Introduction](odin_data_types.md).

> [!NOTE] Not all Ziglings exercises are included. Some exercises are skipped because they are redundant. Several others are skipped because they cover `async/await`, a feature that is not yet available in the main Zig compiler.

The Ziglings exercises present broken code examples that need fixes to pass their tests, but here we present just completed solutions. Rather than focus on the particular problems being solved and the logic of their solutions, the video commentary and the code comments in this text focus just on the Zig language features introduced by each exercise.

> [!WARNING] I strongly recommend working through the Ziglings exercises yourself at some point, say, one or two weeks after watching the video and reading this text. 

### 003_assignment.zig

```rust
// A package namespace is a struct.
// This assigns the "std" package struct to 'std' in the 
// current package struct.
const std = @import("std");

// Program entry point. Returns nothing.
pub fn main() void {
    // local variable 'n' of type u8
    var n: u8 = 50;
    n = n + 5;

    // local constant 'pi' of type u32
    const pi: u32 = 314159;

    // local constant 'negative_eleven' of type i8
    const negative_eleven: i8 = -11;

    // The 'std' package includes 'debug' package,
    // and 'debug' package includes 'print' function.
    // The .{} is an anonymous struct literal, here with three 
    // values assigned to indexes 0, 1, and 2 of the struct.
    // Print's second parameter has type 'anytype'. 
    // Print uses introspection to access the indexes of the struct.
    std.debug.print("{} {} {}\n", .{ n, pi, negative_eleven });
}
```


### 005_arrays2.zig

```rust
const std = @import("std");
// create alias in local package for member of imported package
const assert = std.debug.assert;

pub fn main() void {
    // array of u8s
    // The underscore indicates the array size is 
    // inferred from the number of elements.
    const le = [_]u8{ 1, 3 };
    const et = [_]u8{ 3, 7 };

    // 1 3 3 7
    // ++ concatenates the two [2]u8 arrays into a [4]u8 array
    const leet = le ++ et;
    assert(leet.len == 4);

    // 1 0 0 1 1 0 0 1 1 0 0 1
    // ** concatenates 3 instances of the array together
    const bit_pattern = [_]u8{ 1, 0, 0, 1 } ** 3;
    assert(bit_pattern.len == 12);

    std.debug.print("LEET: ", .{});

    // loop for each element of leet, assiging each element to n
    for (leet) |n| {
        std.debug.print("{}", .{n});
    }

    std.debug.print(", Bits: ", .{});

    for (bit_pattern) |n| {
        std.debug.print("{}", .{n});
    }

    std.debug.print("\n", .{});
}
```


### 006_strings.zig

```rust
const std = @import("std");

pub fn main() void {
    const ziggy = "stardust";

    const d: u8 = ziggy[4];  // the u8 at index 4 of the string

    // concatenate 3 instances of the string together
    const laugh = "ha " ** 3;

    const major = "Major";
    const tom = "Tom";

    // concatenate strings 'major', " ", and 'tom'
    const major_tom = major ++ " " ++ tom;

    // {u} means print as unsigned 
    // {s} means print as string
    std.debug.print("d={u} {s}{s}\n", .{ d, laugh, major_tom });
}

```

### 007_strings2.zig

```rust
const std = @import("std");

pub fn main() void {
    // Multi-line string literals begin with \\ and run to end of line. 
    // Successive lines starting with \\ continue the string.
    const lyrics =
        \\Ziggy played guitar
        \\Jamming good with Andrew Kelley
        \\And the Spiders from Mars
    ;

    std.debug.print("{s}\n", .{lyrics});
}

```

### 010_if2.zig

```rust
const std = @import("std");

pub fn main() void {
    const discount = true;

    // if-else used as an expression: 
    //      if discount is true, evaluates to 17
    //      if discount is false, evaluates to 20
    const price: u8 = if (discount) 17 else 20;

    std.debug.print("With the discount, the price is ${}.\n", .{price});
}
```


### 012_while2.zig

```rust
const std = @import("std");

pub fn main() void {
    var n: u32 = 2;

    // The expression after : is evaluated after each iteration.
    while (n < 1000) : (n *= 2) {
        std.debug.print("{} ", .{n});
    }

    std.debug.print("n={}\n", .{n});
}
```

### 016_for2.zig

```rust
const std = @import("std");

pub fn main() void {
    const bits = [_]u8{ 1, 0, 1, 1 };
    var value: u32 = 0;

    // A for loop can iterate over multiple collections or ranges in tandem.
    // The lengths must match. (If the lengths are not knownable
    // at compile time, the lengths are checked at runtime before 
    // the first iteration and trigger a panic if unequal.)
    // Here, array 'bits' and range 0.. are iterated in tandem.
    // (The upper bound of this range is unspecified, so it 
    // automatically matches the array length.)
    for (bits, 0..) |bit, i| {
        // convert the usize i to a u32 with builtin @intCast()
        const i_u32: u32 = @intCast(i);
        const place_value = std.math.pow(u32, 2, i_u32);
        value += place_value * bit;
    }

    std.debug.print("The value of bits '1101': {}.\n", .{value});
}
```

### 021_errors.zig

```rust
// Defines MyNumberError to be an "error set" type.
// An error set is like an enum, but the members are 
// given unique global ids.
const MyNumberError = error{
    TooBig,
    TooSmall,
    TooFour,
};

const std = @import("std");

pub fn main() void {
    const nums = [_]u8{ 2, 3, 4, 5, 6 };

    for (nums) |n| {
        std.debug.print("{}", .{n});

        const number_error = numberFail(n);

        if (number_error == MyNumberError.TooBig) {
            std.debug.print(">4. ", .{});
        }
        if (number_error == MyNumberError.TooSmall) {
            std.debug.print("<4. ", .{});
        }
        if (number_error == MyNumberError.TooFour) {
            std.debug.print("=4. ", .{});
        }
    }

    std.debug.print("\n", .{});
}

// returns a MyNumberError value
fn numberFail(n: u8) MyNumberError {
    if (n > 4) return MyNumberError.TooBig;
    if (n < 4) return MyNumberError.TooSmall;
    return MyNumberError.TooFour;
}
```


### 022_errors2.zig

```rust
const std = @import("std");

const MyNumberError = error{
    TooSmall
};

pub fn main() void {
    // an "error union" type is two types joined by an !:  
    //        SomeErrorSet ! SomePayloadType
    // ...where the "payload" can be any type (including another error set).

    // The variable's type here is an error 
    // union joining MyNumberError and u8,
    // so this variable can be assigned any 
    // MyNumberError value or any u8 value.
    var my_number: MyNumberError!u8 = 5;

    my_number = MyNumberError.TooSmall;

    std.debug.print("I compiled!\n", .{});
}

```

### 023_errors3.zig

```rust
const std = @import("std");

const MyNumberError = error{
    TooSmall
};

pub fn main() void {
    // If the call returns an error, the catch expression is evaluated 
    // and returned instead of the value returned by the call itself.
    // Generally, a catch clause acts as a default for case of an error.
    const a: u32 = addTwenty(44) catch 22;  // 'a' assigned 64
    const b: u32 = addTwenty(4) catch 22;   // 'b' assigned 22

    std.debug.print("a={}, b={}\n", .{ a, b });
}

// Returns either a MyNumberErorr or a u32
fn addTwenty(n: u32) MyNumberError!u32 {
    if (n < 5) {
        return MyNumberError.TooSmall;
    } else {
        return n + 20;
    }
}
```

### 024_errors4.zig

```rust
const std = @import("std");

const MyNumberError = error{
    TooSmall,
    TooBig,
};

pub fn main() void {
    const a: u32 = makeJustRight(44) catch 0;
    const b: u32 = makeJustRight(14) catch 0;
    const c: u32 = makeJustRight(4) catch 0;

    std.debug.print("a={}, b={}, c={}\n", .{ a, b, c });
}

// (You can ignore the convoluted logic of this 
// example. Just focus on the catch syntax.)

fn makeJustRight(n: u32) MyNumberError!u32 {
    // If the fixTooBig call returns an error, catch clause is evaluated.
    // The catch clause assigns the error to 'err' and executes 
    // its block (the curly braces after |err|).
    return fixTooBig(n) catch |err| {
        return err;  // return from the containing function
    };
}

fn fixTooBig(n: u32) MyNumberError!u32 {
    return fixTooSmall(n) catch |err| {
        if (err == MyNumberError.TooBig) {
            return 20;
        }
        return err;
    };
}

fn fixTooSmall(n: u32) MyNumberError!u32 {
    return actualFix(n) catch |err| {
        if (err == MyNumberError.TooSmall) {
            return 10;
        }
        return err;
    };
}

fn actualFix(n: u32) MyNumberError!u32 {
    if (n < 10) return MyNumberError.TooSmall;
    if (n > 20) return MyNumberError.TooBig;
    return n;
}
```


### 025_errors5.zig

```rust
const std = @import("std");

const MyNumberError = error{
    TooSmall,
    TooBig,
};

pub fn main() void {
    const a: u32 = addFive(44) catch 0;
    const b: u32 = addFive(14) catch 0;
    const c: u32 = addFive(4) catch 0;

    std.debug.print("a={}, b={}, c={}\n", .{ a, b, c });
}

fn addFive(n: u32) MyNumberError!u32 {
    // The 'try' is shorthand for:
    //      detect(n) catch |err| return err;
    const x = try detect(n);
    return x + 5;
}

fn detect(n: u32) MyNumberError!u32 {
    if (n < 10) return MyNumberError.TooSmall;
    if (n > 20) return MyNumberError.TooBig;
    return n;
}

```


### 026_hello2.zig

```rust
const std = @import("std");

pub fn main(init: std.process.Init) !void {
    // std.debug.print writes to standard error, not standard output!

    const io = init.io;
    // Get the standard output file.
    var stdout_file = std.Io.File.stdout();
    // Create a writer for standard output
    var stdout_writer = stdout_file.writer(io, &.{});
    const stdout = &stdout_writer.interface;

    // Writing to standard output can fail with an error,
    // so we use 'try': 
    try stdout.print("Hello world!\n", .{});
}

```

### 027_defer.zig

```rust
const std = @import("std");

pub fn main() void {
    // defer the print call to end of the scope
    // (in this case end of the function)
    defer std.debug.print("Two\n", .{});

    // Should print before the above.
    std.debug.print("One ", .{});
}

```

### 029_errdefer.zig

```rust
const std = @import("std");

var counter: u32 = 0;

const MyErr = error{ GetFail, IncFail };

pub fn main() void {
    // return if we fail to get a number
    const a: u32 = makeNumber() catch return;
    const b: u32 = makeNumber() catch return;

    std.debug.print("Numbers: {}, {}\n", .{ a, b });
}

fn makeNumber() MyErr!u32 {
    std.debug.print("Getting number...", .{});

    // registers deferred print call, 
    // but only executes if function returns an error
    errdefer std.debug.print("failed!\n", .{});

    // These try calls may trigger returning an error.
    var num = try getNumber();
    num = try increaseNumber(num);

    std.debug.print("got {}. ", .{num});
    return num;
}

fn getNumber() MyErr!u32 {
    return 4;
}

fn increaseNumber(n: u32) MyErr!u32 {
    if (counter > 0) return MyErr.IncFail;
    counter += 1;
    return n + 1;
}

```


### 030_switch.zig

```rust
const std = @import("std");

pub fn main() void {
    const lang_chars = [_]u8{ 26, 9, 7, 42 };

    for (lang_chars) |c| {
        // switch on value of 'c'
        switch (c) {
            1 => std.debug.print("A", .{}), // if 1
            2 => std.debug.print("B", .{}), // if 2
            3 => std.debug.print("C", .{}), // etc...
            4 => std.debug.print("D", .{}),
            5 => std.debug.print("E", .{}),
            6 => std.debug.print("F", .{}),
            7 => std.debug.print("G", .{}),
            8 => std.debug.print("H", .{}),
            9 => std.debug.print("I", .{}),
            10 => std.debug.print("J", .{}),
            // ... skip some letters
            25 => std.debug.print("Y", .{}),
            26 => std.debug.print("Z", .{}),
            else => {  // default case
                std.debug.print("?", .{});
            },
        }
    }

    std.debug.print("\n", .{});
}

```

### 031_switch2.zig

```rust
const std = @import("std");

pub fn main() void {
    const lang_chars = [_]u8{ 26, 9, 7, 42 };

    for (lang_chars) |c| {
        // switch as an expression:
        const real_char: u8 = switch (c) {
            1 => 'A',  // evaluate to 'A'
            2 => 'B',  // evaluate to 'B'
            3 => 'C',  // etc...
            4 => 'D',
            5 => 'E',
            6 => 'F',
            7 => 'G',
            8 => 'H',
            9 => 'I',
            10 => 'J',
            // ...
            25 => 'Y',
            26 => 'Z',
            else => '!',
        };

        std.debug.print("{c}", .{real_char});
    }

    std.debug.print("\n", .{});
}
```

### 032_unreachable.zig

```rust
const std = @import("std");

pub fn main() void {
    const operations = [_]u8{ 1, 1, 1, 3, 2, 2 };

    var current_value: u32 = 0;

    for (operations) |op| {
        switch (op) {
            1 => {
                current_value += 1;
            },
            2 => {
                current_value -= 1;
            },
            3 => {
                current_value *= current_value;
            },
            else => unreachable,  // triggers a panic!
            // (useful in development to crash and emit stack trace 
            // if an unexpected code path executes)
        }

        std.debug.print("{} ", .{current_value});
    }

    std.debug.print("\n", .{});
}

```

### 033_iferror.zig

```rust
const MyNumberError = error{
    TooBig,
    TooSmall,
};

const std = @import("std");

pub fn main() void {
    const nums = [_]u8{ 2, 3, 4, 5, 6 };

    for (nums) |num| {
        std.debug.print("{}", .{num});

        const n: MyNumberError!u8 = numberMaybeFail(num);

        // Branches on error union value:
        if (n) |value| {
            // if n is the payload type (u8 in this case)
            std.debug.print("={}. ", .{value});
        } else |err| switch (err) {
            // if n is the error set type (MyNumberError in this case)
            MyNumberError.TooBig => std.debug.print(">4. ", .{}),
            MyNumberError.TooSmall => std.debug.print("<4. ", .{}),
        }
    }

    std.debug.print("\n", .{});
}

fn numberMaybeFail(n: u8) MyNumberError!u8 {
    if (n > 4) return MyNumberError.TooBig;
    if (n < 4) return MyNumberError.TooSmall;
    return n;
}

```

### 035_enums.zig

```rust
const std = @import("std");

// Define type Ops as an enum with three values
// (Unlike in Odin, values of an enum with no specified integer type 
// are not implicitly integers. Rather, they are just distinct names.)
const Ops = enum {
    inc,
    pow,
    dec 
};

pub fn main() void {
    const operations = [_]Ops{
        Ops.inc,
        Ops.inc,
        Ops.inc,
        Ops.pow,
        Ops.dec,
        Ops.dec,
    };

    var current_value: u32 = 0;

    for (operations) |op| {
        // Switch on enum value
        switch (op) {
            Ops.inc => {
                current_value += 1;
            },
            Ops.dec => {
                current_value -= 1;
            },
            Ops.pow => {
                current_value *= current_value;
            },
            // No "else" because already exhaustive
        }

        std.debug.print("{} ", .{current_value});
    }

    std.debug.print("\n", .{});
}

```


### 036_enums2.zig

```rust
const std = @import("std");

// Define enum type Color with three values.
// u32 is the backing "tag" type
const Color = enum(u32) {
    red = 0xff0000,
    green = 0x00ff00,
    blue = 0x0000ff,
};

pub fn main() void {

    //     {x:0>6}
    //      ^
    //      x       type ('x' is lower-case hexadecimal)
    //       :      separator (needed for format syntax)
    //        0     padding character (default is ' ')
    //         >    alignment ('>' aligns right)
    //          6   width (use padding to force width)
    std.debug.print(
        \\<p>
        \\  <span style="color: #{x:0>6}">Red</span>
        \\  <span style="color: #{x:0>6}">Green</span>
        \\  <span style="color: #{x:0>6}">Blue</span>
        \\</p>
        \\
    , .{
        @intFromEnum(Color.red),   // convert from Color to u32
        @intFromEnum(Color.green),
        @intFromEnum(Color.blue), 
    });
}

```


### 037_structs.zig

```rust
const std = @import("std");

const Role = enum {
    wizard,
    thief,
    bard,
    warrior,
};


// Define a struct Character with four fields
const Character = struct {
    role: Role,        // field 'role' of type Role
    gold: u32,         // field 'gold' of type u32
    experience: u32,   // etc...
    health: u8,
};

pub fn main() void {
    // A Character struct literal with specified values for all four fields
    var glorp_the_wise = Character{
        .role = Role.wizard,
        .gold = 20,
        .experience = 10,
        .health = 100,
    };

    glorp_the_wise.gold += 5;
    glorp_the_wise.health -= 10;

    std.debug.print("Your wizard has {} health and {} gold.\n", .{
        glorp_the_wise.health,
        glorp_the_wise.gold,
    });
}
```

### 039_pointers.zig

```rust
const std = @import("std");

pub fn main() void {
    var num1: u8 = 5;

    // *u8 = pointer to u8
    // The & operator here gets a *u8 from the u8 variable.
    const num1_pointer: *u8 = &num1;  

    var num2: u8 = 6;
    num2 = num1_pointer.*;  // dereference the pointer

    // num1_pointer is const, but the pointer itself is not,
    // so we can assign to its dereference
    num1_pointer.* = 9;

    std.debug.print("num1: {}, num2: {}\n", .{ num1, num2 });
}

```

### 040_pointers2.zig

```rust
const std = @import("std");

pub fn main() void {
    var a: u8 = 0;

    // 'b' is a const, so cannot assign a different pointer value to 'b',
    // and the type is 'pointer-to-const-u8'
    // (&a returns a pointer-to-u8, which is 
    // implicitly cast to pointer-to-const-u8)
    const b: *const u8 = &a;

    // cannot assign to deref of a pointer-to-const
    // b.* = 7;

    // Note that a const pointer does NOT guarantee that 
    // the referenced data is immutable!
    a = 12;

    // OK to deref pointer-to-const to read value
    std.debug.print("a: {}, b: {}\n", .{ a, b.* });
}
```


### 045_optionals.zig

```rust
const std = @import("std");

pub fn main() void {
    const result = deepThought();

    // 'orelse' evaluates and returns its 
    // right operand if left operand is null
    const answer: u8 = result orelse 42;

    std.debug.print("The Ultimate Answer: {}.\n", .{answer});
}

// Returns either a u8 or null
// (u8 is not a pointer type, but it's still nullable!)
fn deepThought() ?u8 {
    return null;
}
```


### 046_optionals2_.zig

```rust
const std = @import("std");

const Elephant = struct {
    letter: u8,

    // 'tail' is a pointer-to-Elephant or null
    // (a non-? pointer cannot be null)
    tail: ?*Elephant = null,
    visited: bool = false,
};

pub fn main() void {
    var elephantA = Elephant{ .letter = 'A' };
    var elephantB = Elephant{ .letter = 'B' };
    var elephantC = Elephant{ .letter = 'C' };

    linkElephants(&elephantA, &elephantB);
    linkElephants(&elephantB, &elephantC);

    visitElephants(&elephantA);

    std.debug.print("\n", .{});
}

fn linkElephants(e1: ?*Elephant, e2: ?*Elephant) void {
    
    // panic if e1 or e2 is null, otherwise assign e2 to e1.tail
    (e1 orelse unreachable).tail = e2 orelse unreachable; 

    // shorthand for prior line
    e1.?.tail = e2.?;
}

fn visitElephants(first_elephant: *Elephant) void {
    var e = first_elephant;

    while (!e.visited) {
        std.debug.print("Elephant {u}. ", .{e.letter});
        e.visited = true;

        // break if .tail is null
        e = e.tail orelse break;
    }
}

```

### 047_methods.zig

```rust
const std = @import("std");

const Alien = struct {
    health: u8,

    // .hatch belongs to namespace of Alien
    pub fn hatch(strength: u8) Alien {
        return Alien{
            .health = strength * 5,
        };
    }
};

const HeatRay = struct {
    damage: u8,

    // .zp belongs to namespace of HeatRay
    pub fn zap(self: HeatRay, alien: *Alien) void {
        alien.health -= if (self.damage >= alien.health) 
            alien.health else self.damage;
    }
};

pub fn main() void {
    var aliens = [_]Alien{
        // invoke .hatch like a Java static method
        Alien.hatch(2),
        Alien.hatch(1),
        Alien.hatch(3),
        Alien.hatch(3),
        Alien.hatch(5),
        Alien.hatch(3),
    };

    var n_aliens_alive = aliens.len;
    const heat_ray = HeatRay{ .damage = 7 };

    while (n_aliens_alive > 0) {
        n_aliens_alive = 0;

        // loop through every alien by pointer
        // (both & and * required)
        for (&aliens) |*alien| {

            HeatRay.zap(heat_ray, alien);
            // heat_ray.zap(alien);  // shorthand for prior line

            if (alien.health > 0) {
                n_aliens_alive += 1;
            }
        }

        std.debug.print("{} aliens. ", .{n_aliens_alive});
    }

    std.debug.print("Earth is saved!\n", .{});
}
```


### 050_no_value.zig

```rust
const std = @import("std");

const Err = error{
    Cthulhu
};

pub fn main() void {
    
    // A pointer-to-const-[16]u8
    // Starts undefined (i.e. explicitly uninitialized)
    var first_line1: *const [16]u8 = undefined;
    
    // String literal of 16 characters coerced to *const [16]u8
    first_line1 = "That is not dead";

    // An error union of Err and pointer-to-const-[21]u8
    var first_line2: Err!*const [21]u8 = Err.Cthulhu;

    // String literal of 21 characters coerced to *const [21]u8
    first_line2 = "which can eternal lie";

    // Need "{!s}" format for the error union string.
    std.debug.print("{s} {!s} / ", .{ first_line1, first_line2 });

    printSecondLine();
}

fn printSecondLine() void {
    // Nullable-pointer-to-const-[18]u8
    var second_line2: ?*const [18]u8 = null;
    
    // String literal of 18 characters coerced to *const [18]u8
    second_line2 = "even death may die";

    std.debug.print("And with strange aeons {s}.\n", .{second_line2.?});
}
```


### 051_values.zig

```rust
const std = @import("std");

const Character = struct {
    gold: u32 = 0,
    health: u8 = 100,
    experience: u32 = 0,
};

// global Character constant
const the_narrator = Character{
    .gold = 12,
    .health = 99,
    .experience = 9000,
};

// global Character variable
var global_wizard = Character{};

pub fn main() void {
    var glorp = Character{
        .gold = 30,
    };
    const reward_xp: u32 = 200;

    // local alias of imported function
    const print = std.debug.print;

    var glorp_access1: Character = glorp;
    glorp_access1.gold = 111;
    print("1:{}!. ", .{glorp.gold == glorp_access1.gold});

    var glorp_access2: *Character = &glorp;
    glorp_access2.gold = 222;
    print("2:{}!. ", .{glorp.gold == glorp_access2.gold});

    const glorp_access3: *Character = &glorp;
    glorp_access3.gold = 333;
    print("3:{}!. ", .{glorp.gold == glorp_access3.gold});

    print("XP before:{}, ", .{glorp.experience});

    levelUp(&glorp, reward_xp);

    print("after:{}.\n", .{glorp.experience});
}

fn levelUp(character: *Character, xp: u32) void {
    character.experience += xp;
}
```

### 052_slices.zig

```rust
const std = @import("std");

pub fn main() void {
    var cards = [8]u8{ 'A', '4', 'K', '8', '5', '2', 'Q', 'J' };

    // constants 'hand1' and 'hand2' are slices of u8
    
    // [0..4] gets slice of array from index 0 up to (but not including) 4
    const hand1: []u8 = cards[0..4];

    // [4..] gets slice of array from index 4 up through end of the array
    const hand2: []u8 = cards[4..];

    std.debug.print("Hand1: ", .{});
    printHand(hand1);

    std.debug.print("Hand2: ", .{});
    printHand(hand2);
}

fn printHand(hand: []u8) void {
    for (hand) |h| {
        std.debug.print("{u} ", .{h});
    }
    std.debug.print("\n", .{});
}
```

### 053_slices2.zig

```rust
const std = @import("std");

pub fn main() void {
    const scrambled = "great base for all your justice are belong to us";

    // these are slices of const u8
    // (slicing a string returns a slice of constants)
    const base1: []const u8 = scrambled[15..23];
    const base2: []const u8 = scrambled[6..10];
    const base3: []const u8 = scrambled[32..];
    printPhrase(base1, base2, base3);

    const justice1: []const u8 = scrambled[11..14];
    const justice2: []const u8 = scrambled[0..5];
    const justice3: []const u8 = scrambled[24..31];
    printPhrase(justice1, justice2, justice3);

    std.debug.print("\n", .{});
}

fn printPhrase(part1: []const u8, part2: []const u8, part3: []const u8) void {
    std.debug.print("'{s} {s} {s}.' ", .{ part1, part2, part3 });
}
```


### 054_manypointers.zig

```rust
const std = @import("std");

pub fn main() void {
    const s = "ABCDEFG";

    // Coerce to a pointer-to-const-array
    // (s.len is a compile time expression, so valid for the array size)
    const ptr: *const [s.len]u8 = s;

    // Coerce to a slice
    var slice: []const u8 = s;

    // A "many"-pointer-to-const-u8
    const manyptr: [*]const u8 = ptr;

    // Index the many-item pointer like an array
    const char: u8 = manyptr[5]; // 'F'

    // Get slice from a many pointer
    // Range is from 0 up to (but not including) ptr.len
    slice = manyptr[0..ptr.len];

    std.debug.print("{s} {c}\n", .{ slice, char });
}
```


### 055_unions.zig

```rust
const std = @import("std");

// A Zig union is like a struct where the fields overlap in memory, 
// so assigning to  .ant clobbers .bee and vice versa
const Insect = union {
    ant: Ant,
    bee: Bee,
};

const Ant = struct {
    still_alive: bool,
};

const Bee = struct {
    flowers_visited: u16,
};

// Unlike Odin enums, a Zig enum by default does not include a tag, 
// so we are separately creating this enum to discriminate the union
const Species = enum {
    ant, 
    bee,
};

pub fn main() void {
    const ant = Ant{ .still_alive = true };
    const bee = Bee{ .flowers_visited = 15 };

    // A union literal looks basically like a struct lieral
    var insect = Insect{ .ant = ant };
    printInsect(insect, Species.ant);

    insect = Insect{ .bee = bee };
    printInsect(insect, Species.bee);
}

fn printInsect(insect: Insect, species: Species) void {
    // switch on the enum value
    switch (species) {
        .ant => std.debug.print("Ant alive is: {}. \n",
                .{insect.ant.still_alive}),
        .bee => std.debug.print("Bee visited {} flowers. \n",
                .{insect.bee.flowers_visited}),
    }
}
```

### 056_unions2.zig

```rust
const std = @import("std");

// This Insect union is tagged with the Species enum
const Insect = union(Species) {
    ant: Ant,
    bee: Bee,
};

const Ant = struct {
    still_alive: bool,
};

const Bee = struct {
    flowers_visited: u16,
};

const Species = enum {
    ant,
    bee,
};

pub fn main() void {
    const ant = Ant{ .still_alive = true };
    const bee = Bee{ .flowers_visited = 16 };

    // Insect with .ant value has the ant Species tag
    var insect = Insect{ .ant = ant };
    printInsect(insect);

    // Insect with .bee value has the bee Species tag
    insect = Insect{ .bee = bee };
    printInsect(insect);
}

fn printInsect(insect: Insect) void {
    // switch on enum tag of the Insect union value
    switch (insect) {
        .ant => |a| std.debug.print("Ant alive is: {}. \n", .{a}),
        .bee => |b| std.debug.print("Bee visited {} flowers. \n", .{b}),
    }
}

```

### 057_unions3.zig

```rust
const std = @import("std");

// union of 'enum' means a tag enum is implicitly defined
const Insect = union(enum) {
    ant: Ant,
    bee: Bee,
};

const Ant = struct {
    still_alive: bool,
};

const Bee = struct {
    flowers_visited: u16,
};

pub fn main() void {
    const ant = Ant{ .still_alive = true };
    const bee = Bee{ .flowers_visited = 16 };

    var insect = Insect{ .ant = ant };
    printInsect(insect);

    insect = Insect{ .bee = bee };
    printInsect(insect);
}

fn printInsect(insect: Insect) void {
    switch (insect) {
        // the tag names are same as the union fields
        .ant => |a| std.debug.print("Ant alive is: {}. \n", .{a}),
        .bee => |b| std.debug.print("Bee visited {} flowers. \n", .{b}),
    }
}
```


### 060_floats.zig

```rust
const print = @import("std").debug.print;

pub fn main() void {
    // e in number literal indicates scientific notation
    const shuttle_weight: f32 = 0.453592 * 4480e3;

    // d = decimal
    // .0 = precision
    print("Shuttle liftoff weight: {d:.0} metric tons\n",
            .{shuttle_weight / 1e3});
}

```


### 061_coercions.zig

```rust
// 1. Types can always be made _more_ restrictive.
//
//    var foo: u8 = 5;
//    var p1: *u8 = &foo;
//    var p2: *const u8 = p1; // mutable to immutable
//
// 2. Numeric types can coerce to _larger_ types.
//
//    var n1: u8 = 5;
//    var n2: u16 = n1; // integer "widening"
//
//    var n3: f16 = 42.0;
//    var n4: f32 = n3; // float "widening"
//
// 3. Single-item pointers to arrays coerce to slices and
//    many-item pointers.
//
//    const arr: [3]u8 = [3]u8{5, 6, 7};
//    const s: []const u8 = &arr;  // to slice
//    const p: [*]const u8 = &arr; // to many-item pointer
//
// 4. Single-item mutable pointers can coerce to single-item
//    pointers pointing to an array of length 1.
//
// 5. Payload types and null coerce to optionals (the ? types).
//
// 6. Payload types and errors coerce to error unions.
//
//    const MyError = error{Argh};
//    var char: u8 = 'x';
//    var char_or_die: MyError!u8 = char; // payload type
//    char_or_die = MyError.Argh;         // error
//
// 7. 'undefined' coerces to any type (or it wouldn't work!)
//
// 8. Compile-time numbers coerce to compatible types.
//
//    Just about every single exercise program has had an example
//    of this, but a full and proper explanation is coming your
//    way soon in the third-eye-opening subject of comptime.
//
// 9. Tagged unions coerce to the current tagged enum.
//
// 10. Enums coerce to a tagged union when that tagged field is a
//     zero-length type that has only one value (like void).
//
// 11. Zero-bit types (like void) can be coerced into single-item
//     pointers.
//

const print = @import("std").debug.print;

pub fn main() void {
    var letter: u8 = 'A';

    // rules 4 and 5 apply here
    const my_letter: ?*[1]u8 = &letter;

    print("Letter: {u}\n", .{my_letter.?.*[0]});
}

```

### 062_loop_expressions.zig

```rust
const print = @import("std").debug.print;

pub fn main() void {
    const langs: [6][]const u8 = .{
        "Erlang",
        "Algol",
        "C",
        "OCaml",
        "Zig",
        "Prolog",
    };

    // for loop used as an expression: 
    // evaluates into slice of the values from each break
    const current_lang: ?[]const u8 = for (langs) |lang| {
        if (lang.len == 3) {
            break lang;  // returns lang for this iteration
        }
    } else null;  // evaluates to null if loop never entered

    if (current_lang) |cl| {
        print("Current language: {s}\n", .{cl});
    } else {
        print("Did not find a three-letter language name. :-(\n", .{});
    }
}

```

### 063_labels.zig

```rust
const print = @import("std").debug.print;

const ingredients = 4;
const foods = 4;

const Food = struct {
    name: []const u8,
    requires: [ingredients]bool,
};

//                 Chili  Macaroni  Tomato Sauce  Cheese
// ------------------------------------------------------
//  Mac & Cheese              x                     x
//  Chili Mac        x        x
//  Pasta                     x          x
//  Cheesy Chili     x                              x
// ------------------------------------------------------

const menu: [foods]Food = [_]Food{
    Food{
        .name = "Mac & Cheese",
        .requires = [ingredients]bool{ false, true, false, true },
    },
    Food{
        .name = "Chili Mac",
        .requires = [ingredients]bool{ true, true, false, false },
    },
    Food{
        .name = "Pasta",
        .requires = [ingredients]bool{ false, true, true, false },
    },
    Food{
        .name = "Cheesy Chili",
        .requires = [ingredients]bool{ true, false, false, true },
    },
};

pub fn main() void {
    const wanted_ingredients = [_]u8{ 0, 3 }; // Chili, Cheese

    // outer loop has label :food_loop
    const meal = food_loop: for (menu) |food| {
        for (food.requires, 0..) |required, required_ingredient| {
            if (!required) {
                continue;   // continue innermost loop
            }

            const found = for (wanted_ingredients) |want_it| {
                if (required_ingredient == want_it) {
                    // return true for iteration of innermost loop
                    break true;  
                }
            } else false;

            if (!found) {
                // continue outer loop
                continue :food_loop;  
            }
        }

        // return food for iteration of outer loop
        break food;  
    } else undefined; 
    // a loop expression must have an else, but this 
    // should be unreachable, so we just use undefined  

    print("Enjoy your {s}!\n", .{meal.name});
}
```


### 064_builtins.zig

```rust
const print = @import("std").debug.print;

pub fn main() void {
    //   @addWithOverflow(a: anytype, b: anytype) 
    //          struct { @TypeOf(a, b), u1 }
    //
    //     - 'a' and 'b' are numbers of anytype.
    //     - The return value is a tuple with the result 
    //       and a possible overflow bit.
    //
    const a: u4 = 0b1101;
    const b: u4 = 0b0101;
    const my_result = @addWithOverflow(a, b);

    // Check out our fancy formatting! b:0>4 means, "print
    // as a binary number, zero-pad right-aligned four digits."
    // The print() below will produce: "1101 + 0101 = 0010 (true)".
    print("{b:0>4} + {b:0>4} = {b:0>4} ({s})\n", .{ a, b, my_result[0], if (my_result[1] == 1) "true" else "false" });

    const expected_result: u8 = 0b10010;
    print("Without overflow: {b:0>8}.\n", .{expected_result});

    //   @bitReverse(integer: anytype) T
    //
    //     * 'integer' is the value to reverse.
    //     * The return value will be the same type with the
    //       value's bits reversed
    //
    const input: u8 = 0b11110000;
    const tupni: u8 = @bitReverse(input);
    print("Furthermore, {b:0>8} backwards is {b:0>8}.\n", .{ input, tupni });
}
```


### 065_builtins2.zig

```rust
const print = @import("std").debug.print;

const Narcissus = struct {
    // Default values 
    // (fields with default values can be 
    // omitted in literals of the struct)
    me: *Narcissus = undefined,
    myself: *Narcissus = undefined,
    echo: void = undefined,

    fn fetchTheMostBeautifulType() type {
        // Returns the type of the containing struct
        // (in this case Narcissus)
        // 
        // Note: by convention, the name of a builtin that 
        // returns a type starts with uppercase letter
        return @This(); 
    }
};

fn typeToString(myType: type) []const u8 {
    // Import assigned to a local constant
    const indexOf = @import("std").mem.indexOf;

    // Gets type name as string
    const name = @typeName(myType);
    // Turn "065_builtins2.Narcissus" into "Narcissus"
    return name[indexOf(u8, name, ".").? + 1 ..];
}

pub fn main() void {
    var narcissus: Narcissus = Narcissus{};

    narcissus.me = &narcissus;
    narcissus.myself = &narcissus;

    // Get type
    const Type1: type = @TypeOf(narcissus, narcissus.me.*, 
            narcissus.myself.*);

    const Type2: type = Narcissus.fetchTheMostBeautifulType();

    print("A {s} loves all {s}es. \n", .{
        typeToString(Type1),
        typeToString(Type2),
    });

    // @"foo" is required syntax for identifiers that match a reserved word
    const fields = @typeInfo(Narcissus).@"struct".fields;

    // 'fields' is a slice of StructField, which is defined as:
    //
    //     pub const StructField = struct {
    //         name: [:0]const u8,
    //         type: type,
    //         default_value_ptr: ?*const anyopaque,
    //         is_comptime: bool,
    //         alignment: comptime_int,
    //
    //         defaultValue() ?sf.type  // Function that loads the
    //                                  // field's default value from
    //                                  // `default_value_ptr`
    //     };
    //

    const field0 = if (fields[0].type != void) fields[0].name else " ";
    const field1 = if (fields[1].type != void) fields[1].name else " ";
    const field2 = if (fields[2].type != void) fields[2].name else " ";

    print("He has room in his heart for: {s} {s} {s}", 
            .{ field0, field1, field2 });

    print(".\n", .{});
}
```

### 066_comptime.zig

```rust
const print = @import("std").debug.print;

pub fn main() void {
    // Unique types exist for constant number literals
    // (the types could be left implicit here)
    const const_int: comptime_int = 12345;
    const const_float: comptime_float = 987.654;

    print("Immutable: {}, {d:.3}; ", .{ const_int, const_float });

    // Literals coerced from comptime_int / _float
    var var_int: u32 = 12345;
    var var_float: f32 = 987.654;

    var_int = 54321;
    var_float = 456.789;

    print("Mutable: {}, {d:.3}; ", .{ var_int, var_float });

    print("Types: {}, {}, {}, {}\n", .{
        @TypeOf(const_int),
        @TypeOf(const_float),
        @TypeOf(var_int),
        @TypeOf(var_float),
    });
}

```

### 067_comptime2.zig

```rust
const print = @import("std").debug.print;

// When the compiler processes a statement, it asks two questions:
//
//     1. Should I run this now? (Is it comptime?)
//     2. Should I emit generated code for this? (Is it runtime?)
//
// For some statements it does both.

pub fn main() void {
    // This variable is only *variable* at comptime,
    // however its current value can be baked into 
    // runtime code as a constant.
    comptime var count = 0;   

    count += 1;   // constant count is now 1
    const a1: [count]u8 = .{'A'} ** count; 

    count += 1;   // constant count is now 2
    const a2: [count]u8 = .{'B'} ** count; 

    count += 1;   // constant count is now 3
    const a3: [count]u8 = .{'C'} ** count; 

    count += 1;   // constant count is now 4
    const a4: [count]u8 = .{'D'} ** count; 

    print("{s} {s} {s} {s}\n", .{ a1, a2, a3, a4 });
}
```


### 068_comptime3.zig

```rust
const print = @import("std").debug.print;

const Schooner = struct {
    name: []const u8,
    scale: u32 = 1,
    hull_length: u32 = 143,
    bowsprit_length: u32 = 34,
    mainmast_height: u32 = 95,

    // Parameter 'scale' requires a compile time value argument
    // The function is compiled once for each unique 
    // value passed to 'scale'
    fn scaleMe(self: *Schooner, comptime scale: u32) void {
        const my_scale = if (scale == 0) 1 else scale;
        self.scale = my_scale;
        self.hull_length /= my_scale;
        self.bowsprit_length /= my_scale;
        self.mainmast_height /= my_scale;
    }

    fn printMe(self: Schooner) void {
        print("{s} (1:{}, {} x {})\n", .{
            self.name,
            self.scale,
            self.hull_length,
            self.mainmast_height,
        });
    }
};

pub fn main() void {
    var whale = Schooner{ .name = "Whale" };
    var shark = Schooner{ .name = "Shark" };
    var minnow = Schooner{ .name = "Minnow" };

    // variable only at comptime
    comptime var scale: u32 = undefined;

    scale = 32; // 1:32 scale

    // pass constant value 32
    minnow.scaleMe(scale);
    minnow.printMe();

    scale -= 16; // 1:16 scale

    // pass constant value 16
    shark.scaleMe(scale);
    shark.printMe();

    scale -= 16; // 0

    // pass constant value 0
    whale.scaleMe(scale);
    whale.printMe();
}

```

### 069_comptime4.zig

```rust
const print = @import("std").debug.print;

pub fn main() void {
    const s1 = makeSequence(u8, 3); // creates a [3]u8
    const s2 = makeSequence(u32, 5); // creates a [5]u32
    const s3 = makeSequence(i64, 7); // creates a [7]i64

    print("s1={any}, s2={any}, s3={any}\n", .{ s1, s2, s3 });
}

// First parameter takes a comptime type value
// Second parameter takes a comptime usize value
fn makeSequence(comptime T: type, comptime size: usize) [size]T {
    var arr: [size]T = undefined;
    var i: usize = 0;

    while (i < size) : (i += 1) {
        // This @as coerces the second arg to T
        arr[i] = @as(T, @intCast(i)) + 1;
    }

    return arr;
}
```

### 070_comptime5.zig

```rust
const print = @import("std").debug.print;

const Duck = struct {
    eggs: u8,
    loudness: u8,
    location_x: i32 = 0,
    location_y: i32 = 0,

    fn waddle(self: *Duck, x: i16, y: i16) void {
        self.location_x += x;
        self.location_y += y;
    }

    fn quack(self: Duck) void {
        if (self.loudness < 4) {
            print("\"Quack.\" ", .{});
        } else {
            print("\"QUACK!\" ", .{});
        }
    }
};

const RubberDuck = struct {
    in_bath: bool = false,
    location_x: i32 = 0,
    location_y: i32 = 0,

    fn waddle(self: *RubberDuck, x: i16, y: i16) void {
        self.location_x += x;
        self.location_y += y;
    }

    fn quack(self: RubberDuck) void {
        // required because Zig demands that every 
        // parameter gets used in some way
        _ = self;  
        print("\"Squeek!\" ", .{});
    }

    fn listen(self: RubberDuck, dev_talk: []const u8) void {
        _ = dev_talk;
        self.quack();
    }
};

const Duct = struct {
    diameter: u32,
    length: u32,
    galvanized: bool,
    connection: ?*Duct = null,

    // Returns !void, meaning any kind of error or void (nothing)
    fn connect(self: *Duct, other: *Duct) !void {
        if (self.diameter == other.diameter) {
            self.connection = other;
        } else {
            return DuctError.UnmatchedDiameters;
        }
    }
};

const DuctError = error{UnmatchedDiameters};

pub fn main() void {
    const duck = Duck{
        .eggs = 0,
        .loudness = 3,
    };

    const rubber_duck = RubberDuck{
        .in_bath = false,
    };

    const duct = Duct{
        .diameter = 17,
        .length = 165,
        .galvanized = true,
    };

    print("duck: {} \n", .{isADuck(duck)});
    print("rubber_duck: {} \n", .{isADuck(rubber_duck)});
    print("duct: {}\n", .{isADuck(duct)}); // false
}

// An anytype parameter takes an argument of any type.
// Function is compiled once for each unique
// type of value passed to 'possible_duck'.
fn isADuck(possible_duck: anytype) bool {
    const Type = @TypeOf(possible_duck);
    const walks_like_duck = @hasDecl(Type, "waddle");
    const quacks_like_duck = @hasDecl(Type, "quack");

    const is_duck = walks_like_duck and quacks_like_duck;

    // Condition evaluated at compile time because 
    // both values are constant.
    // The body is only included in the 
    // runtime code if condition was true.
    if (walks_like_duck and quacks_like_duck) {
        possible_duck.quack();
    }

    return is_duck;
}
```


### 071_comptime6.zig

```rust
const print = @import("std").debug.print;

const Narcissus = struct {
    me: *Narcissus = undefined,
    myself: *Narcissus = undefined,
    echo: void = undefined,
};

pub fn main() void {
    print("Narcissus has room in his heart for:", .{});

    // @typeInfo runs at comptime, so this is a comptime const
    // (meaning its value is fixed at compile time)
    const fields = @typeInfo(Narcissus).@"struct".fields;

    // An 'inline for' unrolls the loop at compile time
    // (meaning the body is repeated for each iteration)
    // Inline allowed here because fields is comptime
    inline for (fields) |field| {
        // This condition is evaluable at comptime,
        // so the if body is only included in runtime
        // when the condition is true.
        if (field.type != void) {
            print(" {s}", .{field.name});
        }
    }

    print(".\n", .{});
}
```


### 072_comptime7.zig

```rust
const print = @import("std").debug.print;

pub fn main() void {
    const instructions = "+3 *5 -2 *2";

    var value: u32 = 0;

    comptime var i = 0;

    // Loop unrolled at compile time
    // (note the header expressions are evaluable at comptime)
    inline while (i < instructions.len) : (i += 3) {
        const digit = instructions[i + 1] - '0';

        // This switch is evaluable at comptime,
        // so only one case baked into runtime 
        // of each loop iteration.
        switch (instructions[i]) {
            '+' => value += digit,
            '-' => value -= digit,
            '*' => value *= digit,
            else => unreachable,
        }
    }

    print("{}\n", .{value});
}
```

### 073_comptime8.zig

```rust
const print = @import("std").debug.print;

const llamas = [5]u32{ 5, 10, 15, 20, 25 };

pub fn main() void {
    const my_llama = getLlama(4);
    print("My llama value is {}.\n", .{my_llama});
}

fn getLlama(comptime i: usize) u32 {
    // Execute expression at comptime
    // (allowed when for expressions that 
    // include only comptime values)
    comptime assert(i < llamas.len);

    return llamas[i];
}

fn assert(ok: bool) void {
    if (!ok) unreachable;
}
```

### 074_comptime9.zig

```rust
// File scope (outside of any function) is implicitly comptime

const print = @import("std").debug.print;

const llamas = makeLlamas(5);  // call is impliticly comptime

fn makeLlamas(comptime count: usize) [count]u8 {
    var temp: [count]u8 = undefined;
    var i = 0;

    while (i < count) : (i += 1) {
        temp[i] = i;
    }

    return temp;
}

pub fn main() void {
    print("My llama value is {}.\n", .{llamas[2]});
}
```


### 076_sentinels.zig

```rust
const print = @import("std").debug.print;
const sentinel = @import("std").meta.sentinel;

pub fn main() void {
    // "sentinal-terminated array" of u32 values
    // (0 is the sentinel)
    // This array has 7 u32 values, and the last value is 0.
    // Its .len is 6, so last valid index is 5.
    var nums = [_:0]u32{ 1, 2, 3, 4, 5, 6 };

    // "sentinal-terminated many-item pointer" of u32 values
    // (0 is the sentinel)
    // If ptr's type used a terminator other than 0,
    // this coercion would be illegal.
    const ptr: [*:0]u32 = &nums;

    nums[3] = 0;

    printSequence(nums);
    printSequence(ptr);
}

fn printSequence(my_seq: anytype) void {
    const my_typeinfo = @typeInfo(@TypeOf(my_seq));

    switch (my_typeinfo) {
        .array => {
            print("Array:", .{});

            for (my_seq) |s| {
                print("{}", .{s});
            }
        },
        .pointer => {
            // if a pointer...

            // The sentinel function from the meta package 
            // returns the sentinal value of the type (in this case 0)
            const my_sentinel = sentinel(@TypeOf(my_seq));
            print("Many-item pointer:", .{});

            var i: usize = 0;
            while (my_seq[i] != my_sentinel) {
                print("{}", .{my_seq[i]});
                i += 1;
            }
        },
        else => unreachable,
    }
    print("\n", .{});
}
```


### 077_sentinels2.zig

```rust
// Zig strings are compatible with C strings (which
// are null-terminated) AND can be coerced to a variety of other
// Zig types:
//
//     const a: [5]u8 = "array".*;
//     const b: *const [16]u8 = "pointer to array";
//     const c: []const u8 = "slice";
//     const d: [:0]const u8 = "slice with sentinel";
//     const e: [*:0]const u8 = "many-item pointer with sentinel";
//     const f: [*]const u8 = "many-item pointer";
//
//
const print = @import("std").debug.print;

const WeirdContainer = struct {
    data: [*]const u8,
    length: usize,
};

pub fn main() void {
    const foo = WeirdContainer{
        // A string literal is a "constant pointer to a
        // zero-terminated (null-terminated) fixed-size array of u8"
        // Here the literal is coerced to [*]const u8
        .data = "Weird Data!",
        .length = 11,
    };

    const printable = foo.data[0..foo.length];

    print("{s}\n", .{printable});
}
```

### 078_sentinels3.zig

```rust
const print = @import("std").debug.print;

pub fn main() void {
    const data: [*]const u8 = "Weird Data!";

    // @ptrCast returns type inferred from context
    // (here the assignment target)
    const printable: [*:0]const u8 = @ptrCast(data);
    print("{s}\n", .{printable});
}
```

### 079_quoted_identifiers.zig

```rust
const print = @import("std").debug.print;

pub fn main() void {
    // The @"foo" syntax is a quoted identifier.
    // Allows for identifier names that otherwise would be illegal.
    const @"55_cows": i32 = 55;
    const @"isn't true": bool = false;

    print("Sweet freedom: {}, {}.\n", .{
        @"55_cows",
        @"isn't true",
    });
}
```


### 080_anonymous_structs.zig

```rust
const print = @import("std").debug.print;

// This function returns a generated struct type
fn Circle(comptime T: type) type {
    // An anonymous struct *type* (not a literal)
    return struct {
        center_x: T,
        center_y: T,
        radius: T,
    };
}

pub fn main() void {
    // Define circle1 to be a struct type where T is i32
    const circle1 = Circle(i32){
        .center_x = 25,
        .center_y = 70,
        .radius = 15,
    };

    // Define circle1 to be a struct type where T is f32
    const circle2 = Circle(f32){
        .center_x = 25.234,
        .center_y = 70.999,
        .radius = 15.714,
    };

    print("[{s}: {},{},{}] \n", .{
        @typeName(@TypeOf(circle1)),
        circle1.center_x,
        circle1.center_y,
        circle1.radius,
    });

    print("[{s}: {d:.1},{d:.1},{d:.1}]\n", .{
        @typeName(@TypeOf(circle2)),
        circle2.center_x,
        circle2.center_y,
        circle2.radius,
    });
}
```


### 081_anonymous_structs2.zig

```rust
const print = @import("std").debug.print;

pub fn main() void {
    // Anonymous struct literals can have any 
    // combination of field names and values
    printCircle(.{
        .center_x = @as(u32, 205),
        .center_y = @as(u32, 187),
        .radius = @as(u32, 12),
    });

    printCircle(.{
        .center_x = @as(f32, 205),
        .center_y = @as(f32, 187),
        .radius = @as(u8, 12),
        .something = 5,  // printCircle won't use this field, but that's OK
    });
}

// Accepts any struct with fields .center_x, .center_y, .radius
// having types that can be printed as {} in the format string
fn printCircle(circle: anytype) void {
    print("x:{} y:{} radius:{}\n", .{
        circle.center_x,
        circle.center_y,
        circle.radius,
    });
}
```

### 082_anonymous_structs3.zig

```rust
const print = @import("std").debug.print;

pub fn main() void {
    // Implicit numbered field names: .0, .1, .2, .3
    const foo = .{
        true,
        false,
        @as(i32, 42),
        @as(f32, 3.141592),
    };

    printTuple(foo);
}

fn printTuple(tuple: anytype) void {
    // []const builtin.Type.StructField
    const fields = @typeInfo(@TypeOf(tuple)).@"struct".fields;

    // Iterate over each field
    inline for (fields) |field| {
        print("\"{s}\"({any}):{any} \n", .{
            field.name,
            field.type,
            @field(tuple, field.name), // get value of field by name string
        });
    }
}
```

### 083_anonymous_lists.zig

```rust
const print = @import("std").debug.print;

pub fn main() void {
    // Coerced tuple into array
    const hello: [5]u8 = .{ 'h', 'e', 'l', 'l', 'o' };
    print("I say {s}!\n", .{hello});
}
```


### 092_interfaces.zig

```rust
const std = @import("std");

// Each insect type has a print method

const Ant = struct {
    still_alive: bool,

    pub fn print(self: Ant) void {
        std.debug.print("Ant is {s}.\n", 
            .{if (self.still_alive) "alive" else "dead"});
    }
};

const Bee = struct {
    flowers_visited: u16,

    pub fn print(self: Bee) void {
        std.debug.print("Bee visited {} flowers.\n", 
            .{self.flowers_visited});
    }
};

const Grasshopper = struct {
    distance_hopped: u16,

    pub fn print(self: Grasshopper) void {
        std.debug.print("Grasshopper hopped {} meters.\n", .{self.distance_hopped});
    }
};

const Insect = union(enum) {
    ant: Ant,
    bee: Bee,
    grasshopper: Grasshopper,

    pub fn print(self: Insect) void {
        switch (self) {
            // At compiletime, generates a case
            // for every member of Insect
            // (compile error if a member doesn't
            //  have a print method)
            inline else => |case| return case.print(),
        }
    }
};

pub fn main() !void {
    const my_insects = [_]Insect{
        Insect{ .ant = Ant{ .still_alive = true } },
        Insect{ .bee = Bee{ .flowers_visited = 17 } },
        Insect{ .grasshopper = Grasshopper{ .distance_hopped = 32 } },
    };

    std.debug.print("Daily Insect Report:\n", .{});
    for (my_insects) |insect| {
        Insect.print(insect);
    }
}
```


### 093_hello_c.zig

```rust
const std = @import("std");

// @cImport parses an expression of C code and imports 
// the functions, types, variables, and compatible 
// macro definitions into a new empty struct type,
//  and then returns that type.
const c: type = @cImport(
    // Block of code passed as expression to @cImport
    { 
        // Appends "#include <$path>\n" to the c_import temporary buffer
        // (this function can only be called inside @cImport expression)
        @cInclude("unistd.h");
    }
);

pub fn main() void {
    // Call the imported c function which has this Zig signature:
    //
    //  pub extern fn write(
    //      _Filehandle: c_int, 
    //      _Buf: ?*const anyopaque, 
    //      _MaxCharCount: c_uint) 
    //      c_int;
    //
    const c_res = c.write(2, "Hello C from Zig!", 17);

    std.debug.print(" - C result is {d} chars written.\n", .{c_res});
}

// "-lc" tells Zig compiler to include C libraries
// e.g. "zig run -lc exercises/093_hello_c.zig".
```

### 094_c_math.zig

```rust
const std = @import("std");

const c = @cImport(
    {
        @cInclude("math.h");
    }
);

pub fn main() !void {
    const angle = 765.2;
    const circle = 360;

    // Call C mod function having this Zig signature:
    //
    //  pub extern fn fmod(
    //      _X: f64,
    //      _Y: f64) 
    //      f64;
    //
    const result = c.fmod(angle, circle);

    std.debug.print(
        "The normalized angle of {d: >3.1} degrees is {d: >3.1} degrees.\n", 
            .{ angle, result });
}

```

### 095_for3.zig

```rust
const std = @import("std");

pub fn main() void {
    // range from 1 up to (but NOT including) 21
    for (1..21) |n| {
        if (n % 3 == 0) continue;
        if (n % 5 == 0) continue;
        std.debug.print("{} ", .{n});
    }

    std.debug.print("\n", .{});
}
```


### 096_memory_allocation.zig

```rust
const std = @import("std");

fn runningAverage(arr: []const f64, avg: []f64) void {
    var sum: f64 = 0;

    for (0.., arr) |index, val| {
        sum += val;
        const f_index: f64 = @floatFromInt(index + 1);
        avg[index] = sum / f_index;
    }
}

pub fn main() !void {
    // Pretend this was defined by reading in user input
    const arr: []const f64 = &[_]f64{ 0.3, 0.2, 0.1, 0.1, 0.4 };

    // Initialize new arena allocator derived from std.heap.page_allocator
    var arena = std.heap.ArenaAllocator.init(std.heap.page_allocator);

    // Defer freeing the whole arena's memory
    defer arena.deinit();

    // Get Allocator from ArenaAllocator
    // (ArenaAllocator is the specific allocator type, but we 
    // wrap as an Allocator to use the general Allocator functions)
    const allocator = arena.allocator();

    // Allocate a block of memory 
    // This call returns *[arr.len]f64, which we coerce to []f64.
    const avg: []f64 = try allocator.create([arr.len]f64);

    runningAverage(arr, avg);
    std.debug.print("Running Average: ", .{});
    for (avg) |val| {
        std.debug.print("{d:.2} ", .{val});
    }
    std.debug.print("\n", .{});
}

// For more details on memory allocation and the different types of
// memory allocators, see https://www.youtube.com/watch?v=vHWiDx_l4V0
```

### 098_bit_manipulation2.zig

```rust
const std = @import("std");
const ascii = std.ascii;
const print = std.debug.print;

pub fn main() !void {
    print("Is this a pangram? {}!\n", 
        .{isPangram("The quick brown fox jumps over the lazy dog.")});
}

fn isPangram(str: []const u8) bool {
    if (str.len < 26) {
        return false;
    }

    var bits: u32 = 0;

    for (str) |c| {
        if (ascii.isAscii(c) and ascii.isAlphabetic(c)) {
            // |= performs a bitwise 'or'
            //
            // When shifting a u32 with <<, the right operand must be a u5,
            // (because 2^5 is the number of bits in a u32)
            // To make the right operand a u5, we use @truncate, 
            // which truncates the u8 to u5 (the return
            // type is inferred from the context)
            //
            // Because bits is a u32, it can only be or'd with another
            // u32. By itself, the integer literal could be any integer type,
            // but @truncate needs it to be a u32 to correclty infer its return type.
            bits |= @as(u32, 1) << @truncate(ascii.toLower(c) - 'a');
        }
    }

    return bits == 0x3ff_ffff;
}
```

### 099_formatting.zig

```rust
const std = @import("std");
const print = std.debug.print;

pub fn main() !void {
    const size = 15;

    // header
    print("\n   |", .{});
    for (0..size) |n| {
        // format as a decimal (d), right-aligned (>), 
        // and minimum width 3
        print("{d:>3} ", .{n + 1});
    }
    print("\n", .{});

    // separator
    var n: u8 = 0;
    while (n <= size) : (n += 1) {
        print("---+", .{});
    }
    print("\n", .{});

    // rows
    for (0..size) |a| {
        // format as a decimal (d), right-aligned (>), 
        // and minimum width 2
        print("{d:>2} |", .{a + 1});
        for (0..size) |b| {
            print("{d:3} ", .{(a + 1) * (b + 1)});
        }
        print("\n\n", .{});
    }
}
```

### 100_for4.zig

```rust
const std = @import("std");
const print = std.debug.print;

pub fn main() void {
    const hex_nums = [_]u8{ 0xb, 0x2a, 0x77 };
    const dec_nums = [_]u8{ 11, 42, 119 };

    // Iterate through both arrays in tandem
    // (Allowed because both are same length)
    for (hex_nums, dec_nums) |hex, dec| {
        if (hex != dec) {
            print("Uh oh! Found a mismatch: {d} vs {d}\n", .{ hex, dec });
            return;
        }
    }

    print("Arrays match!\n", .{});
}
```

### 101_for5.zig

```rust
const std = @import("std");
const print = std.debug.print;

const Role = enum {
    wizard,
    thief,
    bard,
    warrior,
};

pub fn main() void {
    const roles = [4]Role{ .wizard, .bard, .bard, .warrior };
    const gold = [4]u16{ 25, 11, 5, 7392 };
    const experience = [4]u8{ 40, 17, 55, 21 };

    // Iterate over multiple arrays in tandem (sizes must match)
    // The range size will automatically match the others.
    for (roles, gold, experience, 1..) |c, g, e, i| {
        const role_name = switch (c) {
            .wizard => "Wizard",
            .thief => "Thief",
            .bard => "Bard",
            .warrior => "Warrior",
        };

        std.debug.print("{d}. {s} (Gold: {d}, XP: {d})\n", .{
            i,
            role_name,
            g,
            e,
        });
    }
}
```


### 102_testing.zig

```rust
// execute with `zig test` instead of `zig run`

const std = @import("std");
const testing = std.testing;

fn add(a: f16, b: f16) f16 {
    return a + b;
}

// A test fails if it returns an error.

test "add" {
    try testing.expect(add(41, 1) == 42);
    try testing.expectEqual(42, add(41, 1));
    try testing.expect(add(5, -4) == 1);
    try testing.expect(add(1.5, 1.5) == 3);
}

fn sub(a: f16, b: f16) f16 {
    return a - b;
}

test "sub" {
    try testing.expect(sub(10, 5) == 5);
    try testing.expect(sub(3, 1.5) == 1.5);
}

fn divide(a: f16, b: f16) !f16 {
    if (b == 0) return error.DivisionByZero;
    return a / b;
}

test "divide" {
    try testing.expect(divide(2, 2) catch unreachable == 1);
    try testing.expect(divide(-1, -1) catch unreachable == 1);
    try testing.expect(divide(10, 2) catch unreachable == 5);
    try testing.expect(divide(1, 3) catch unreachable == 0.3333333333333333);

    try testing.expectError(error.DivisionByZero, divide(15, 0));
}
```

### 103_tokenization.zig

```rust
const std = @import("std");
const print = std.debug.print;

pub fn main() !void {

    // Multi-line string
    const poem =
        \\My name is Ozymandias, King of Kings;
        \\Look on my Works, ye Mighty, and despair!
    ;

    // Returns a TokenIterator(u8, DelimiterType.any),
    // which splits the poem by the delimiters
    const delimiters = ",\n ;!";
    var it = std.mem.tokenizeAny(u8, poem, delimiters);

    var cnt: usize = 0;
    while (it.next()) |word| {
        cnt += 1;
        print("{s}\n", .{word});
    }

    print("This little poem has {d} words!\n", .{cnt});
}

```

### 104_threading.zig

```rust
const std = @import("std");

const total_time = 5;

pub fn main() !void {
    std.debug.print("Starting work...\n", .{});

    // Create a block so that the defers inside exit just this subscope.
    {
        // Spawn thread with parameter value 1
        const handle = try std.Thread.spawn(.{}, thread_function, .{1});
        // join() waits for thread to complete, then cleans up the thread
        defer handle.join();

        // Spawn thread with parameter value 2
        const handle2 = try std.Thread.spawn(.{}, thread_function, .{2});
        defer handle2.join();

        // Spawn thread with parameter value 3
        const handle3 = try std.Thread.spawn(.{}, thread_function, .{3});
        defer handle3.join();

        // While the threads spawned above run, we can do
        // other business on main thread...
        // (though in this case we're just sleeping for total_time seconds)
        var io_instance: std.Io.Threaded = .init_single_threaded;
        const io = io_instance.io();
        try io.sleep(std.Io.Duration.fromSeconds(total_time), .awake);

        std.debug.print("main thread: finished.\n", .{});
    }

    // only reach here after all the joins
    std.debug.print("Zig is cool!\n", .{});
}

// When used as function for new thread, the thread param is passed to 'delay'
fn thread_function(delay: usize) !void {
    // .init_single_threaded is an instance of the file struct but with
    // member values that differ from the default
    var io_instance: std.Io.Threaded = .init_single_threaded;
    const io = io_instance.io();

    // Sleep to delay start
    // (isize is like usize but signed)
    // (sleep expects a std.Io.Clock enum value,
    // so .awake is understood as std.Io.Clock.awake)
    const seconds = 1 * @as(isize, @intCast(delay));
    try io.sleep(std.Io.Duration.fromSeconds(seconds), .awake);

    // Print message after delay
    std.debug.print("thread {d}: {s}\n", .{ delay, "started." });

    // Sleep for the rest of the total_time
    const work_time = total_time - delay;
    try io.sleep(std.Io.Duration.fromSeconds(@intCast(work_time)), .awake);

    std.debug.print("thread {d}: {s}\n", .{ delay, "finished." });
}
```


### 105_threading2.zig

```rust
const std = @import("std");

pub fn main() !void {
    const count = 1_000_000_000;
    var pi_plus: f64 = 0;
    var pi_minus: f64 = 0;

    // We pass pointers to these threads
    {
        const handle1 = try std.Thread.spawn(.{}, thread_pi, 
            .{ &pi_plus, 5, count });
        defer handle1.join();

        const handle2 = try std.Thread.spawn(.{}, thread_pi, 
            .{ &pi_minus, 3, count });
        defer handle2.join();
    }
    // We're reading value from pointer that was computed by the spawned threads
    // (safe here because the two threads have been joined already)
    std.debug.print("PI ≈ {d:.8}\n", .{4 + pi_plus - pi_minus});
}

// Receives pointer (necessary to get result back on main therad)
fn thread_pi(pi: *f64, begin: u64, end: u64) !void {
    var n: u64 = begin;
    while (n < end) : (n += 4) {
        pi.* += 4 / @as(f64, @floatFromInt(n));
    }
}
```


### 106_files.zig

```rust
const std = @import("std");

// std.process.Init represents initial state of the process
pub fn main(init: std.process.Init) !void {
    // Get standard output
    const io: std.Io = init.io;

    // Get current working directory
    const cwd: std.Io.Dir = std.Io.Dir.cwd();

    cwd.createDir(io, "output", .default_dir) catch |e| switch (e) {
        error.PathAlreadyExists => {}, // if error.PathAlreadyExists, do nothing
        else => return e, // propagate other errors
    };

    const output_dir: std.Io.Dir = try cwd.openDir(io, "output", .{});
    defer output_dir.close(io);

    const file: std.Io.File = try output_dir.createFile(io, "zigling.txt", .{});
    defer file.close(io);

    var file_writer = file.writer(io, &.{});
    // We made file_writer a var instead of a const so that
    // the & operator here returns a *Io.Writer instead 
    // of a *const Io.Writer
    // (The write function expects a *Io.Writer)
    const writer = &file_writer.interface;

    const byte_written = try writer.write("It's zigling time!");
    std.debug.print("Successfully wrote {d} bytes.\n", .{byte_written});
}
```

### 107_files2.zig

```rust
const std = @import("std");

pub fn main(init: std.process.Init) !void {
    const io = init.io;
    const cwd = std.Io.Dir.cwd();

    var output_dir = try cwd.openDir(io, "output", .{});
    defer output_dir.close(io);

    const file = try output_dir.openFile(io, "zigling.txt", .{});
    defer file.close(io);

    var content = [_]u8{'A'} ** 64;
    // This should print out : 
    // `AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA`
    std.debug.print("{s}\n", .{content});

    var file_reader = file.reader(io, &.{});
    const reader = &file_reader.interface;

    // Reads data from the file into the content array
    // Returns the number of bytes read
    const bytes_read = try reader.readSliceShort(&content);

    std.debug.print("Successfully Read {d} bytes: {s}\n", .{
        bytes_read,
        content[0..bytes_read],
    });
}

```

### 108_labeled_switch.zig

```rust
const std = @import("std");

const PullRequestState = enum(u8) {
    Draft,
    InReview,
    Approved,
    Rejected,
    Merged,
};

pub fn main() void {
    // Label on switch allows break and continue
    // break = jump out of the switch
    // continue = jump to start of the switch with a new value to switch on
    // (effectively, a continue jumps to a different case)
    pr: switch (PullRequestState.Draft) {
        PullRequestState.Draft => continue :pr PullRequestState.InReview,
        PullRequestState.InReview => continue :pr PullRequestState.Approved,
        PullRequestState.Approved => continue :pr PullRequestState.Merged,
        PullRequestState.Rejected => {
            std.debug.print("The pull request has been rejected.\n", .{});
            return;
        },
        PullRequestState.Merged => break :pr,
    }
    std.debug.print("The pull request has been merged.\n", .{});
}
```


### 109_vectors.zig

```rust
// @Vector returns a vector type
// (The size of vector is not strictly limited, but in practice
// you rarely want vectors of more than 4 elements.)

// The @Vector() expression returns a vecotr type, but then the {} after 
// makes thse literals.
// So v1 is assigned a vector of 3 i32s, with the values 1, 10, 100
const v1 = @Vector(3, i32){ 1, 10, 100 };
// v2 is assigned a vector of 3 f32s, with the values 2.0, 3.0, 5.0
const v2 = @Vector(3, f32){ 2.0, 3.0, 5.0 };

// Component-wise addition and multiplication
const v3 = v1 + v1; // {   2,  20,  200};
const v4 = v2 * v2; // { 4.0, 9.0, 25.0};

// Cast components of vector of i32 into a vector of f32
const v5: @Vector(3, f32) = @floatFromInt(v3); // { 2.0,  20.0,  200.0}

// Component-wise subtraction
const v6 = v4 - v5; // { 2.0, -11.0, -175.0}

// Component-wise absolute values
const v7 = @abs(v6); // { 2.0,  11.0,  175.0}

// @splat(2) returns a vector length is inferred from context and 
// where every element is the argument
// So v8 is assigned a vector of 4 u8s, with the values 2, 2, 2, 2
const v8: @Vector(4, u8) = @splat(2); // { 2, 2, 2, 2}

// @reduce invokes the specified operation on each successive pair, 
// producing a scalar result
const v8_sum = @reduce(.Add, v8); // 8, the result of 2 + 2 + 2 + 2
const v8_min = @reduce(.Min, v8); // 2, the result of min(min(min(2, 2), 2), 2)

// Fixed-length arrays can be automatically assigned to vectors (and vice-versa).
const single_digit_primes = [4]i8{ 2, 3, 5, 7 };
const prime_vector: @Vector(4, i8) = single_digit_primes;

// A calculation with arrays instead of vectors
fn calcMaxPairwiseDiffOld(list1: [4]f32, list2: [4]f32) f32 {
    var max_diff: f32 = 0;
    for (list1, list2) |n1, n2| {
        const abs_diff = @abs(n1 - n2);
        if (abs_diff > max_diff) {
            max_diff = abs_diff;
        }
    }
    return max_diff;
}

// Define Vec4 as a vector type of 4 f32s
const Vec4 = @Vector(4, f32);

// Same as prior function, but uses vectors
fn calcMaxPairwiseDiffNew(a: Vec4, b: Vec4) f32 {
    const abs_diff_vec = @abs(a - b);
    const max_diff = @reduce(.Max, abs_diff_vec);
    return max_diff;
}

const std = @import("std");
const print = std.debug.print;

pub fn main() void {
    const l1 = [4]f32{ 3.141, 2.718, 0.577, 1.000 };
    const l2 = [4]f32{ 3.154, 2.707, 0.591, 0.993 };
    const mpd_old = calcMaxPairwiseDiffOld(l1, l2);
    const mpd_new = calcMaxPairwiseDiffNew(l1, l2);
    print("Max difference (old fn): {d: >5.3}\n", .{mpd_old});
    print("Max difference (new fn): {d: >5.3}\n", .{mpd_new});
}
```