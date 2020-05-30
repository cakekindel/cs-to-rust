# C# to Rust
learn Rust by transforming C# into Rust, step by step

This is a thought piece that is meant to be read top-to-bottom in order,
and is a "what if" exercise, changing the core values of C# to those of
Rust, with the end goal of arriving at the Rust language.

## Table of Contents
- [Why this guide exists](#why-this-guide-exists)
    - [If this guide helped you](#if-this-guide-helped-you)
    - [If this guide is missing something](#if-this-guide-is-missing-something)
- [Language Values](#language-values)
    - [List of values](#list-of-values)
    - [C# or Java](#cs-or-java)
    - [Rust](#rust)
    - [How to interpret](#how-to-interpret)
- [Mutability & Data Races](#mutability-and-data-races)
    - [How can we determine that a value is thread-safe at compile time?](#how-can-we-determine-that-a-value-is-thread-safe-at-compile-time)
- [To Kill a Garbage Collector](#to-kill-a-garbage-collector)
    - [How can we determine when a value is safe to delete from memory at compile time?](#how-can-we-determine-when-a-value-is-safe-to-delete-from-memory-at-compile-time)
- [Enter the Rust Commandments](#enter-the-rust-commandments)
    - [Mutability](#mutability)
    - [Data Ownership](#data-ownership)
    - [Borrowing](#borrowing)
- [Expression oriented](#expression-oriented)
- [Enums](#enums)
    - [Variants of different types](#variants-of-different-types)
    - [Variant "Instances"](#variant-instances)
    - [Tuple Variants](#tuple-variants)
    - [Struct Variants](#struct-variants)

## Why this guide exists
As someone who was at one point only comfortable
in languages and ecosystems heavily geared towards OOP,
Rust can be very difficult to appreciate and fully understand.

Rust is an incredibly powerful, and very unique
programming language that can have a very steep learning curve.

So, let's contextualize it by experimenting with
language design!

### If this guide helped you
please consider doing one or more of the following
- star this repo
- file an issue to say thanks
- share this with other rust-curious individuals

### If this guide is missing something
please [file an issue](https://www.github.com/cakekindel/cs-to-rust/issues/new)!

## Language Values
It's important that we understand the **motivation** for the changes to come.

This section is intended to outline the **core values** of the two languages,
with the intent of pointing out the virtues that each is unwilling to compromise on.

You don't have to agree with every decision that is made, but please make an
effort to understand & internalize the core values of both languages.

### List of values
Here is a very rough list of programming language core values,
shamelessly ripped from [bcantrill](https://github.com/bcantrill)'s
[awesome 2018 talk on Rust](https://www.youtube.com/watch?v=2wZ1pCpJUIM),
and programming values in general.

| | | |
|---|---|---|
|Approachability|Integrity|Robust|
|Availability|Maintainability|Safety|
|Compatibility|Measurability|Security|
|Composability|Operability|Simplicity|
|Debuggability|Performance|Stability|
|Expressiveness|Portability|Thoroughness|
|Extensibility|Resiliency|Transparency|
|Interoperability|Rigor|Velocity|

### CS or Java

| | | |
|---|---|---|
|✅Approachability|Integrity|Robust|
|Availability|Maintainability|Safety|
|✅Compatibility|Measurability|Security|
|Composability|Operability|Simplicity|
|✅Debuggability|✅Performance|Stability|
|Expressiveness|✅Portability|Thoroughness|
|Extensibility|Resiliency|Transparency|
|✅Interoperability|Rigor|✅Velocity|

### Rust
| | | |
|---|---|---|
|Approachability|✅Integrity|✅Robust|
|Availability|Maintainability|✅Safety|
|Compatibility|Measurability|✅Security|
|✅Composability|Operability|Simplicity|
|Debuggability|✅Performance|Stability|
|✅Expressiveness|Portability|Thoroughness|
|✅Extensibility|Resiliency|Transparency|
|✅Interoperability|✅Rigor|Velocity|

### How to interpret
C# / Java values being approachable, classical (in the C-influenced, OOP imperative sense),
and performant. The two languages are geared towards moving quickly with a moderate amount
of type safety.

Rust is to C#, as C# is to Python. Rust introduces some rules of significant
complexity to guarantee memory & thread safety at compile time. This means that in exchange
for some cognitive load & extra development time, you are rewarded with an extremely fast
program that is most likely going to work exactly as you expect it to.

## Mutability and Data Races
In C# / Java, data structures are mutable by default, with the ability for developers
to opt-out via the `readonly` keyword in some scenarios.

This feature works fine in a world where thread safety is not a core concern,
because we have:
- thread-safe structures available that we can opt-in to (e.g. `ConcurrentHashMap`)
- a garbage collector keeping track of data that is in use, and freeing data that is not.

However, if our new Java wants to prioritize thread safety as a primary goal of the
language, we need to rethink this model.

Shared mutability is the scourge of paralellism - it results in defects that are
extremely difficult to diagnose and reason about.

Rust took this problem, and posed this question:

### How can we determine that a value is thread-safe at compile time?

Let's keep this question in our pocket for a moment. We've got bigger fish to fry.

## To Kill a Garbage Collector
If our new Java wanted to prioritize interoperability with unmanaged code (C / C++),
and eke out every bit of performance we can from our programs,
the GC undoubtedly gets in our way.

Pretend for a moment we do not have a Garbage Collector doing its **absolute best**
to prevent our program from going _yonkers_ gobbling up memory.

A question that **needs** to be answered arises:

### How can we determine when a value is safe to delete from memory at compile time?

If we prematurely delete things from memory, we'll have difficult to debug use-after-free errors.

In contrast, if we never delete things from memory, our program will happily run and gobble
up system resources until there aren't any left.

## Enter the Rust Commandments
1. **All values shall be recursively immutable by default**
1. **Only one function or data structure or variable may _own_ a given value**
1. **When an owned value goes out of scope, it is immediately freed in memory**
1. **There can be any number of read-only windows to an owned value**
1. **There can be at most 1 piece of code with a mutable access to a value**

This may sound restrictive, and it is!

If you follow these rules, however, you'll be rewarded with a lightning-fast binary that is
_guaranteed_ to be free from a slew of common memory defects and _extremely_ predictable.

### Mutability
Mutability in Rust can be achieved in 2 ways:
- Mutable ownership
- Mutable reference

Note that there can only be 1 variable / function / structure with mutable access
to a value at any given time, in the name of thread safety.

### Data Ownership
A piece of code _owning_ a piece of data means a few things:
- The owner can issue any number of read-only references (windows) to the owned data
- The owner can issue 1 mutable reference (borrow) of the owned data
- The owner can choose to step into mutating its owned value, but only if there are no
    mutable borrows existing at that time

An example
```rust
pub fn main() {
    let my_name = String::from("Orion Kindel");
    my_name = "This doesn't work, because `my_name` is not mutable in this context";

    let mut my_name_mut = my_name;
    my_name = "This works now, because I _moved_ my_name into my_name_mut, and my_name_mut takes mutable ownership of its String."

    println!("{}", my_name) // This won't compile, because my_name was moved into my_name_mut!
}
```

One idea introduced in the above example is Rust's `move` semantics.

It may be helpful when reasoning about rust's ownership and move semantics, to think
of a variable as being a **_location_** that data is kept in.

When you reference one of these **locations** you're wilfully taking the data out
of that location, and putting it into a new one.

Please check out the [move semantics section](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html?highlight=move#ways-variables-and-data-interact-move)
of the rust book for more information on this language feature.

### Borrowing
A value is _borrowed_, rather than moved, with the ampersand `&` keyword.

Borrowing is not dissimilar to a C / C++ pointer; it indicates a
"window into", or reference of, someone else's owned value.

"Borrow" and "reference" may be used interchangeably.

There are 2 kinds of borrows in Rust; immutable and mutable.

A mutable borrow is taken by modifying the `&` keyword with the `mut`
keyword, like so:

```rust
fn add_world(s: &mut String) {
    s.push_str(", World!");
}

fn main() {
    let hw = add_world("Hello".to_string());
    println!("{}", hw) // prints: "Hello, World!"
}
```

## Expression Oriented
In Java & C#, the _statements_ in a code block do not resolve to values.

For example:
```cs
public class Dice
{
    public void Roll()
    {
        var random = new Random();
        int diceVal = random.Next(1, 6);

        return diceVal;
    }
}
```

In our example, `new Random()` is a C# _expression_ that resolves to an instance
of the `Random` class. `var random = new Random()` however is a _statement_ that
does not resolve to a value, not even `void`. In this language, there is a rough
tacit rule that each line in your program's code is an instructional step,
not an expression that resolves to a value.

Believe it or not, you already know this! If you're familiar with a ternary
statement, you can reason about `if / else` statements as being the **statement**
based conditional idiom, while ternary statements are the **expression-based**
conditional idiom.

Imagine if **both** methods in the following code snippet were OK with the
C# compiler, and semantically equivalent:

```cs
public class Game
{
    private bool canMove = true;
    
    public void GetSpacesToMove_Ternary()
    {
        return this.canMove ? new Dice().Roll() : 0;
    }

    public void GetSpacesToMove_IfStatements()
    {
        return if (this.canMove)
        {
            new Dice().Roll();
        }
        else
        {
            0;
        }
    }

    // <snip>
}
```

If you're wondering "why would you make this change? We already have ternary statements,
and this is _more_ verbose!", with some syntactic changes to `if` statements, you
could argue that ternary statements are wholly unnecessary:

```cs
// ternary
this.canMove ? new Dice().Roll() : 0

// vs our tersified if statements:
if this.canMove { new Dice().Roll() } else { 0 }
```

You don't have to prefer the expression-based terse if statements, but now you're familiar with
the semantics of Rust's conditional statements.

This is important, because if everything resolves to a value,
then you can make the `return` statements **optional** if the last line in a function
body is an expression.

```cs
public int Roll()
{
    var random = new Random();

    random.Next(1, 6)
}

// real C# equivalent:
public int Roll()
{
    var random = new Random();

    return random.Next(1, 6);
}
```

You don't have to love this language feature, but in a world where everything
is an expression, you'll probably come across **function bodies** that
end with an expression rather than a return statement.

## Enums
The Rust `enum` idiom, and the C# `enum` idiom are _very_ different.

Given a C#-style enum like this:
```cs
public enum Coin
{
    Quarter = 1,
    Nickel = 2,
    Dime = 3
}
```

It may be helpful to think of an "instance" of this `Coin` enum as `1` OR `2` OR `3`.

### Variants of different types
What if enums supported each member, or _Variant_, having different types?

```cs
public enum Coin
{
    Quarter = 0.25, // float
    Nickel = 0.05, // float
    Dollar = 1, // int
    Hapenny = 0.005, // float
    Default = null // always null
}
```

Now this `Coin` enum means "0.25 or 0.05 or null or 1"

When handling this `Coin` enum, one might use the
pattern-matching features in C# 8 like so:

```cs
public int AppraiseCoin(Coin coin)
{
    return coin switch
    {
        Coin.Quarter => /* algorithm to appraise quarters? */,
        // <snip>
    };
}
```

### Variant "Instances"
What if the types contained in enums did not have to be constant values?

Let's say you're dealing with IP Addresses and want to have a strongly-typed
differentiation between IPV4 addresses & IPV6:
```cs
public enum IpAddress
{
    V4(string),
    V6(string),
}
```

In this example, if one had a variable of type `IpAddress`,
the variable contains a string that represents the full IP address.

This allows one to use patterns like this:
```cs
public class TcpHelper
{
    public TcpConnection Connect(IpAddress addr)
    {
        return addr switch
        {
            IpAddress.V4(addrV4) => ConnectIpV4(addrV4),
            IpAddress.V6(addrV6) => ConnectIpV6(addrV6),
        }
    }

    private TcpConnection ConnectIpV4(string addrRaw);
    private TcpConnection ConnectIpV6(string addrRaw);
}
```

Contrast this with a vanilla C# implementation:
```cs
public class TcpHelper
{
    public TcpConnection Connect(IpAddress addr)
    {
        if (IsIpV4(addr))
        {
            return ConnectIpV4(addr);
        }
        else if (IsIpV6(addr))
        {
            return ConnectIpV6(addr);
        }
    }

    private bool IsIpV4(string addrRaw) { /* <snip> */ } 
    private bool IsIpV6(string addrRaw) { /* <snip> */ } 
}
```

A couple things to note here:
- In the second example, readers of the code or receivers of `string addr`
    have no hints at the type level that the given string is an Ip Address
    v4, v6, or even an Ip Address at all.

- By using Enums this way, if a hypothetical IPV8 was introduced, **all code**
    pattern-matching over IpAddress _immediately_ breaks, telling the maintainers
    all the places where the new case is not considered.

**PLEASE STOP READING** until that all makes logical sense as a language feature, and as a construct that you can imagine using.

### Tuple Variants
That's neat and all, but an IPV4 is a 4-segment string of integers, not a string.
How could we better represent that constraint with **types** so that requirement
is enforced at compile-time?

What if these constructs could contain **multiple positional** elements?

```cs
public enum IpAddress
{
    V4(int, int, int, int),
    V6(string),
}
```

### Struct Variants
let's expand the idea that a variant can contain multiple positional values,
to multiple **named** values.

Imagine an embedded `struct` in each "instance" of an enum variant:
```cs
TODO: EXAMPLE :)
```

