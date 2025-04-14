+++
title = "Rust Type-level programming for coding interviews"
date = "2025-04-13"
[taxonomies]
tags = ["rust", "code"]
+++

The other day I watched a nice YouTube video called [Let‘s Solve The Most Popular Leetcode Problem only using TypeScript Types - Reverse Linked Lists](https://www.youtube.com/watch?v=tE5EHyFgA94) by Typed Rocks and it got me thinking, can I do the same in Rust? I remember reading articles about [Rust type system being Turing complete](https://sdleffler.github.io/RustTypeSystemTuringComplete/) so it should be doable, but can **I** do it?

## Definition
**This is the exercise:**
Given a [singly linked list](https://en.wikipedia.org/wiki/Linked_list#Singly_linked_list), reverse it. \
**Constraint:** Use only Rust types.

Let's start by completing the exercise using normal means, how do you reverse a singly linked list?
A quick implementation would be to start from the head of the linked list, iterate through the list, and reverse the pointers as you go. To achieve it we will need to keep track of two variables: `prev` and `curr`.

To be more clear, this is a visual representation of the process
```
Step 1
// Start
prev = NOTHING
curr = [1] -> [2] -> [3] -> NOTHING

Step 2
// Remove [1] from `curr`
// Set `prev` (NOTHING) as `next` of [1]
prev = [1] -> NOTHING
curr = [2] -> [3] -> NOTHING

Step 3
// Remove [2] from `curr`
// Set `prev` ([1] -> NOTHING) as `next` of [2]
prev = [2] -> [1] -> NOTHING
curr = [3] -> NOTHING

Step 4
// Remove [3] from `curr`
// Set `prev` ([2] -> [1] -> NOTHING) as `next` of [3]
prev = [3] -> [2] -> [1] -> NOTHING
curr = NOTHING

Step 5
// `curr` is NOTHING, use `prev` as final result
[3] -> [2] -> [1] -> NOTHING
```

This is the Rust implementation of the above logic
```Rust
struct Node {
    value: u32,
    next: Option<Box<Node>>,
}

fn reverse_list(head: Box<Node>) -> Box<Node> {
    let mut prev = None;
    let mut curr = Some(head);

    while let Some(mut node) = curr {
        curr = node.next.take();
        node.next = prev;
        prev = Some(node);
    }

    prev.unwrap()
}
```

## Building blocks
Now that we're all on the same page, let's move on the fun part of the exercise, let's implement the same logic using **only Rust types**.

{% admonition(type="question") %}
🙋 What does it mean to use only types? Aren't we using types in the standard solution, for example `Box<Node>`?
{% end %}

Great question imaginary reader! When I say **only types** I mean **only types**, meaning no functions, no while loop, no variables, no runtime execution. Everything will be done at compile time, and the input, output and logic only exists as types.

Let me show you what I mean, by defining our singly linked list
```Rust
trait List {}

struct Nothing;
struct Node<const DATA: usize, Next: List = Nothing>(PhantomData<Next>);

impl List for Nothing {}
impl<const DATA: usize, Next: List> List for Node<DATA, Next> {}
```
**This is a lot**, let's try to digest it one concept at a time.

A sinlgly linked list is defined by a `Node` with some data and a pointer to either the next `Node` or to nothing. Out of this very simplistic definition we can identify two building blocks of our solution: `Node` and `Nothing`.
```Rust
struct Nothing;
struct Node<const DATA: usize, Next: List = Nothing>(PhantomData<Next>);
```
{% admonition(type="info") %}
In type-level programming each building block of your solution needs to be its own type.
{% end %}

There are two main concepts in here that may be confusing, even for readers that dabbled in Rust: [const generics](https://practice.course.rs/generics-traits/const-generics.html) and [PhantomData](https://doc.rust-lang.org/std/marker/struct.PhantomData.html)

I'll be brief in the explanation here, *const generics* are a way to hold constant values as generic arguments of a type, *PhantomData* is a zero-sized type needed because otherwise Rust will complain that we don't use a generic type.

In our case, we use *const generics* to hold our node data, and *PhantomData* is used to silence Rust's complaints about us not using the generic parameter Next.

The rest is easy, `Node` needs `Next` to be either `Nothing` or another `Node`, to allow this we use the trait `List`
```Rust
trait List {}
impl List for Nothing {}
impl<const DATA: usize, Next: List> List for Node<DATA, Next> {}
```
{% admonition(type="info") %}
`List` is an empty trait because where we're going we don't need functions.
{% end %}

So... how do we declare a singly linked list, for example `[1] -> [2] -> [3]`? Easy!
```Rust
type MyList = Node<1, Node<2, Node<3>>>;
```
As you can see, no variables, no functions, just types!

## The Rev trait
Now that we got the linked list definition out of the way, we need to start implementing the logic for reversing the list, and we all know that when writing a new piece of logic you always start from a ~function~ **trait**.

Before starting, I have to admit that I'm not an expert in type-level programming, even less in Rust type-level programming.
I took heavy inspiration from how the *Rust being Turing complete* article approaches solving the problem: having a trait with an [associated type](https://doc.rust-lang.org/rust-by-example/generics/assoc_items/types.html) to bake the logic into.
```Rust
trait Rev<Prev: List>: List {
    type Output: List;
}
```
{% admonition(type="info") %}
It's necessary for the `Rev` trait to implement the `List` trait due to how the logic for Output is implemented, we'll see it later.
{% end %}

In the standard solution for reversing the linked list we identified two variables: `prev` and `curr`. This trait does the same, it's meant to be implemented for `curr` and has the Prev generic parameter to represent `prev`.
Output is an associated type that defines the next step in the iteration logic.

## Entry point
{% admonition(type="question") %}
🙋 A trait? Functions can be called e.g. `reverse(my_list)`, how do you use this trait to reverse the linked list?
{% end %}

Remember, we're doing type-level programming, let's use a type!

```Rust
type Reverse<Head> = <Head as Rev<Nothing>>::Output;
// Usage:
type MyReversedList = Reverse<MyList>;
```
This is a bit of an esoteric formula, but if we read it spelled out it should all make sense. When defining the trait `Rev` we said that it has to be implemented for `curr` right? This means that in `<Head as Rev<Nothing>>` the `Head` is our `curr` and `Nothing` is our `prev`, starting our process that will eventually give us the reversed list in the `Output` associated type.

{% admonition(type="tip") %}
If you notice, this type defines `Step 1` from the visual representation of the process, by setting `Nothing` as `Prev` in the first iteration.
{% end %}

## Halting condition
Now that we have an entry point, let's start implementing one case at a time, starting with the case of an **empty list**
```Rust
type TheListReversed = Reverse<Nothing>;
```
As expected, when the type `TheListReversed` is used we get this compilation error

{% admonition(type="failure", title="Compilation error") %}
the trait `Rev<Nothing>` is not implemented for `Nothing`
{% end %}

That's right! We created the trait `Rev` but we didn't implement it for any of our building blocks, let's do it for the Nothing struct
```Rust
impl<Prev: List> Rev<Prev> for Nothing {
    type Output = Prev;
}
```
This implementation is equivalent to `Step 5` of the visual representation: when reaching the end of the list, the final result is Prev.

Does it work? Yes!
```
# Test cases (Original -> Reversed)
Nothing -> Nothing
```
{% admonition(type="tip") %}
All our test cases are printed out using [`println!("{}", std::any::type_name::<MyType>())`](https://doc.rust-lang.org/std/any/fn.type_name.html)
{% end %}

Even though it works as expected, this is underwhelming, `Reverse<Nothing>` is nothing more than an identity type. Don't worry though, this specific implementation will become useful later on as a halting condition of our recursion.

## Reversing single Node
What's our next step? Oh yeah, a list made of a single node
```Rust
type TheListReversed = Reverse<Node<1>>;
```
Again, the compiler complains
{% admonition(type="failure", title="Compilation error") %}
the trait `Rev<Nothing>` is not implemented for `Node<1>`
{% end %}

Let's do that, more generally we'll implement the trait `Rev<Nothing>` for any `Node<DATA>`
```Rust
impl<const DATA: usize> Rev<Nothing> for Node<DATA> {
    type Output = /*???*/;
}
```
Ok we have the boilerplate for the trait implementation, what should we replace the question marks with? This is an implementation of `Rev<Nothing>` for `Node<DATA>` (i.e. `Node<DATA, Nothing>`), a very specific case where there is no node before (`Prev` is `Nothing`) and no node after (`Next` is `Nothing`). The reverse of a list made of only one node is a list with that single node!

```Rust
impl<const DATA: usize> Rev<Nothing> for Node<DATA> {
    type Output = Node<DATA>;
}
```

And voilà
```
# Test cases (Original -> Reversed)
Nothing -> Nothing
Node<1> -> Node<1>
```
Fantastic, 3 more lines of code to again achieve the identity type. Please bear with me, these two implementations could be considered both base cases, everything will fit into place soon.

Next step, two nodes!
```Rust
type TheListReversed = Reverse<Node<1, Node<2>>>;
```
Aaaand...
{% admonition(type="failure", title="Compilation error") %}
the trait `Rev<Nothing>` is not implemented for `Node<1, Node<2>>`
{% end %}

That's true, the trait `Rev<Nothing>` is implemented only for the terminal node `Node<DATA, Nothing>`! Let's generalise our trait implementation to cover both terminal and non-terminal nodes. To do that we need to change our `Next` from being `Nothing` to be a generic type of our implementation
```Rust
impl<
    const DATA: usize,
    Next: List
> Rev<Nothing> for Node<DATA, Next> {
    type Output = /*???*/;
}
```
We're here again, what should we replace the question marks with?
Let's look back at our visual representation, we've covered `Step 1` already, is this `Step 2`? Let's see
```
curr = [1] -> [2] -> [3] -> NOTHING

Step 2
// Remove [1] from `curr`
// Set `prev` (NOTHING) as `next` of [1]
prev = [1] -> NOTHING
curr = [2] -> [3] -> NOTHING
```
So... in the next iteration `curr` should be `Next` (`[2] -> [3] -> NOTHING`) and `prev` needs to become `curr` with `Nothing` as `Next`.

Let's try and do the same in our trait implementation!
```Rust
impl<
    const DATA: usize,
    Next: Rev<Node<DATA, Nothing>>
> Rev<Nothing> for Node<DATA, Next> {
    type Output =
        <Next as Rev<Node<DATA, Nothing>>>::Output;
}
```
Wow, it was easy!
As a reminder again, the trait `Rev` defines a generic `Prev` and the implementor is the `curr`.
In the implementation we're saying that the `Next` is now `curr` (because it implements `Rev` in the `<Next as Rev...>`) and the new `Prev` is `Node<DATA, Nothing>`, matching perfectly with what has been defined in `Step 2`!

...wait what?
{% admonition(type="failure", title="Compilation error") %}
the trait `Rev<Node<1>>` is not implemented for `Node<2>`
{% end %}

That's right, when implementing `Rev` for a `Node` we only have a specific implementation `impl<...> Rev<Nothing> for Node<...>`, targeting the head of a linked list, a node that has `Nothing` as a `Prev`!

## Reversing the list
We're getting close to the end, we implemented steps 1, 2 and 5, and looking at it, steps 3 and 4 are actually the same recursion step!

{% admonition(type="question") %}
🙋 Wait, isn't 2 also the same recursion step as 3 and 4?
{% end %}

Not really... wait... yes you're right, great point imaginary reader! But can we really generalise the trait implementation to apply for any of the steps 2/3/4?

The difference between them lies in `Prev`, step 2 has `Prev = Nothing` and steps 3/4 have `Prev = Node`. We have a trait precisely for this case, the trait `List`!

Let's do it then, let's change our trait implementation for the last time!

```Rust
impl<
    const DATA: usize,
    Prev: List,
    Next: Rev<Node<DATA, Prev>>
> Rev<Prev> for Node<DATA, Next> {
    type Output = <Next as Rev<Node<DATA, Prev>>>::Output;
}
```

The only change has been adding a generic type parameter `Prev` and use it instead of the hardcoded `Nothing`, pretty nice right?

Are we ready to try it?
```
# Test cases (Original -> Reversed)
Nothing -> Nothing
Node<1> -> Node<1>
Node<1, Node<2>> -> Node<2, Node<1>>
Node<1, Node<2, Node<3>>> -> Node<3, Node<2, Node<1>>>
```

We did it! Here is the final code

```Rust
use std::{any::type_name, marker::PhantomData};

trait List {}

struct Nothing;
struct Node<const DATA: usize, Next: List = Nothing>(PhantomData<Next>);

impl List for Nothing {}
impl<const DATA: usize, Next: List> List for Node<DATA, Next> {}

trait Rev<Prev: List>: List {
    type Output: List;
}

type Reverse<Head> = <Head as Rev<Nothing>>::Output;

impl<
    const DATA: usize,
    Prev: List,
    Next: Rev<Node<DATA, Prev>>
> Rev<Prev> for Node<DATA, Next> {
    type Output =
        <Next as Rev<Node<DATA, Prev>>>::Output;
}

impl<Prev: List> Rev<Prev> for Nothing {
    type Output = Prev;
}

macro_rules! print_type_and_reverse {
    ($t:ty) => {
        println!(
            "\x1b[92m{}\x1b[0m -> \x1b[92m{}\x1b[0m",
            type_name::<$t>().replace("testing::", ""),
            type_name::<Reverse<$t>>().replace("testing::", "")
        );
    };
}

fn main() {
    print_type_and_reverse!(Nothing);
    print_type_and_reverse!(Node<1>);
    print_type_and_reverse!(Node<1, Node<2>>);
    print_type_and_reverse!(Node<1, Node<2, Node<3>>>);
}
```

## Final thoughts
Sometimes I wonder, am I wasting my time? Surely I could have used my time better instead of solving this exercise with type-level programming. In this day and age, I should probably experiment more with LLMs or learn how to get into blockchain development.

But then I stop thinking, reboot my brain, and just solve the problem in front of me.

Is it useful? No.\
Is it fun? Oh yeah it's fun, my love for writing code is greater than any anxiety I can ever have, and if I can share it, even better!

```Rust
thank_you(Reason::Reading);
// TODO: write next article
```
