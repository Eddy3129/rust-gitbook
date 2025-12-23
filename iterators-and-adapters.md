---
description: >-
  This page discusses Rust as a functional programming language. After learning
  these shorthands, it becomes much easier to perform iterations without relying
  on traditional control structures.
cover: .gitbook/assets/iterators_&_adaptors_banner (1).png
coverY: -5.028724888392858
layout:
  width: default
  cover:
    visible: true
    size: full
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
  metadata:
    visible: true
---

# Iterators & Adapters

## Iterators

Iterators can be defined via .iter(), .iter\_mut(), and .into\_iter(), which each serving different ownership rules on the iterating variable.&#x20;

<figure><img src=".gitbook/assets/image.png" alt="" width="563"><figcaption></figcaption></figure>

### .iter()

Allows for borrowing the original values immutably, without mutation capabilities. Most suitable for read-only operations like summing or printing, which leaves the original data intact.

```rust
let val = vec![1, 2, 3, 4, 5];
let mut sum = 0;

// .iter() yields references (&i32), so 'val' is not consumed
for i in val.iter() {
    sum += i; // Rust automatically handles the reference here
}

println!("Sum: {}", sum);
// 'val' is still available here because we only borrowed it!
println!("Original list: {:?}", val);
```

### .iter\_mut()

If we want to modify existing values without dropping the original data, the best solution is `.iter_mut()`. It excels in performing in-place updates where you need the collection to remain valid afterwards.

```rust
// 1. Variable must be 'mut' to allow changes
let mut val = vec![1, 2, 3, 4, 5];

// 2. We borrow mutable references (&mut i32). 
// No ownership is taken, so 'val' stays safe.
for i in val.iter_mut() {
    // 3. Important! We must dereference (*) the pointer to change the value it points to
    *i += 1; 
}

// 4. The original vector is updated and still accessible
assert_eq!(val, vec![2, 3, 4, 5, 6]);
```

### .into\_iter()&#x20;

Transfers ownership to the iterator, allowing you to manipulate or consume values directly, but _**permanently invalidates the original variable**_.&#x20;

A perfect example is sorting Tinder profiles into separate "left" and "right" vectors. Once swiped, the profile is dropped from the original "discover" list, ensuring it is never seen again.

```rust
// The "Feed": A stack of profiles to review
let profiles = vec!["Alex".to_string(), "Bob".to_string(), "Charlie".to_string()];
    
// The "Matches": Where we move the people we like
let mut matches = Vec::new();

// We use into_iter() to SWIPE through the feed.
// We take full ownership of each profile to decide its fate.
for profile in profiles.into_iter() {
    if profile != "Bob" {
        println!("Swiped Right on {}!", profile);
        matches.push(profile); // MOVED to matches
    } else {
        println!("Swiped Left on {}...", profile);
        // 'profile' is dropped (deleted) here
    }
}

// The result: We have our matches.
 println!("Matches: {:?}", matches); 

// The consequence: The feed is empty/gone. We can't swipe again.
// println!("{:?}", profiles); // ❌ Error: value used here after move
```

***

## Adapters

Adapters are lazy instructions that transform iterators. They allow you to simplify complex looping logic into concise, functional chains.

Key concept: "Lazy" means they do no work on their own. They essentially create a "recipe" for processing data that only executes when a consumer (like `.collect()`) pulls the trigger. Mastering these few common adapters will handle 90% of your data manipulation needs.

<figure><img src=".gitbook/assets/image (1).png" alt="" width="563"><figcaption></figcaption></figure>

### &#x20;.map()

One of the most common adapters in iterator logic, which takes a closure to transform each item. It can be iterated over directly in a `for` loop, allowing you to process values one by one without creating a temporary list.

However, since `.map()` transforms the value, the original input is often lost. To access _both_ the original and the transformed value, we can map them into a tuple:

```rust
let val = vec![1, 2, 3, 4, 5];

// Map returns a tuple: (original, squared)
for (x, square) in val.iter().map(|x| (x, x * x)) {
    println!("The square of {} is {}", x, square);
}
```

### .collect()

`.map()` is helpful for transforming data logic, but the values are not actually computed unless they are intentionally collected. Imagine if `.map()` is an additional section of pipe that changes the water flow; we still need a bucket to catch the output. This is when `.collect()` comes into play.

```rust
let mut val = vec![1, 2, 3, 4, 5];

// The previous 'val' is consumed by into_iter()...
// ...and completely replaced by the new collected vector.
val: Vec<i32> = val.into_iter().map(|x| x * x).collect();

assert_eq!(val, vec![1, 4, 9, 16, 25]);
```

### .filter()

This is a straightforward adapter. It evaluates a condition and removes values that do not fulfill it (i.e., where the closure returns `false`).

```rust
let val = vec![1, 2, 3, 4, 5];

// Filter keeps numbers where the remainder (%) is 0
let evens = val.into_iter().filter(|x| x % 2 == 0).collect();

assert_eq!(evens, vec![2, 4]);
```

### .find()

Finds an element based on a defined condition and returns it immediately when found. It returns an `Option<T>` type (either `Some(value)` or `None`), which makes it ideal for safely checking existence without crashing if the item is missing.

```rust
let val = vec![1, 2, 3, 4, 5];

// use ref &x here due to .iter()
let message = match val.iter().find(|&x| x * x == 9) {
    Some(n) => format!("Found the square root of 9: {}!", n), 
    None => "Not found".to_string(), 
};

assert_eq!(message, "Found the square root of 9: 3!");
```

### .enumerate()

A helpful adaptor for iterating over both the index and the value simultaneously. It wraps the value in a tuple `(index, value)`.

```rust
let val = vec!['a', 'b', 'c', 'd', 'e'];

// use ref &x here due to .iter()
for (index, value) in val.iter().enumerate() {
    println!("Index {} has value {}", index, value);
}
```

### .zip()

A powerful adaptor that stitches two iterators together into pairs (tuples). It links values in order and stops immediately when the shorter iterator runs out. This is super useful for creating HashMaps or Vectors from known lists.

```rust
use std::collections::HashMap; // Import HashMap if we want to use it

fn main() {
    let names = vec!["Alice", "Bob", "Charles"];
    let ages = vec![21, 34]; // shorter list!

    // 1. Loop Example: Stops after "Bob" because 'ages' runs out.
    // Note: We use 'ages' here, not 'scores'
    for (name, age) in names.iter().zip(ages.iter()) {
        println!("{} is {} years old.", name, age);
    }

    // 2. Collection Example:
    // We can collect these pairs directly into a real HashMap
    let people_map: HashMap<_, _> = names.iter().zip(ages.iter()).collect();
    
    // OR into a Vector of tuples
    let people_vec: Vec<(&str, &i32)> = names.iter().zip(ages.iter()).collect();
}
```

### .fold()

Similar to Python's `reduce()`, this method reduces a collection to a single value via aggregation. It maintains an accumulator that carries the result from one step to the next.

```rust
let val = vec![1, 2, 3, 4, 5];

// 1. '0' is the initial value of the accumulator (acc)
// 2. 'x' is the current item from the list
// 3. The result of the block becomes the new 'acc' for the next step
let sum = val.iter().fold(0, |acc, x| acc + x);

assert_eq!(sum, 15);
```

***

## Conclusion

Congratulations on finishing another topic for the Rust programming language!&#x20;

Iterators and adaptors can be quite confusing at first, but they appear in almost every production-level codebase. As you practice, you will realize that your own code can be simplified significantly using these tools.

Here is a final challenge that combines everything we learned:

```rust
use std::collections::HashMap;

fn main() {
    // TASK: Obtain fruits with quantity > 50, exclude "Oranges", 
    // map them for easy data entry, and sum their total value.

    let fruits = vec!["Apples", "Blueberries", "Oranges", "Grapes"];
    let quantities = vec![40, 63, 85, 90];

    // 1. CRITICAL STEP: .zip() FIRST!
    // We must link the name to the quantity before we start removing things.
    // If we filtered 'fruits' first, the indices would misalign with 'quantities'.
    let target_fruits: HashMap<_, _> = fruits.into_iter()
        .zip(quantities.into_iter()) 
        .filter(|(name, qty)| *name != "Oranges" && *qty > 50) // Filter the PAIR
        .collect();

    // 2. Iterate over our new clean HashMap
    for (name, qty) in &target_fruits {
        println!("{} has {}", name, qty);
    }

    // 3. Calculate sum using .fold()
    // Since we are iterating a HashMap, we iterate over .values()
    let total = target_fruits.values().fold(0, |acc, x| acc + x);

    println!("Total quantity of the required food is {}", total);
}
```

We can even further simplify it to:

```rust
fn main() {
    let fruits = vec!["Apples", "Blueberries", "Oranges", "Grapes"];
    let quantities = vec![40, 63, 85, 90];// We chain everything into one logic stream
    // .fold() can handle the summing AND the printing if we want to be clever
    let sum = fruits.iter()
        .zip(quantities.iter())
        .filter(|(name, &qty)| *name != "Oranges" && qty > 50)
        .fold(0, |acc, (name, &qty)| {
            println!("{} has {}", name, qty); // Side effect: Print
            acc + qty // Accumulate
        });
    
    println!("Total quantity: {}", sum);
}
```

}
