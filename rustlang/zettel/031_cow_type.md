# Cow (Clone on Write)

## Core Insight
Cow<T> provides smart copy-on-write functionality, deferring cloning until mutation is needed, optimizing for read-heavy workloads.

```rust
use std::borrow::Cow;

fn process_string(input: &str) -> Cow<str> {
    if input.contains("replace") {
        // Clone only when modification needed
        Cow::Owned(input.replace("replace", "with"))
    } else {
        // No allocation for unmodified case
        Cow::Borrowed(input)
    }
}

let result = process_string("hello");
// No allocation happened - still borrowed
```

Cow enum structure:
```rust
pub enum Cow<'a, B: ?Sized + 'a> where B: ToOwned {
    Borrowed(&'a B),
    Owned(<B as ToOwned>::Owned),
}
```

Common patterns:
```rust
let mut cow: Cow<str> = Cow::Borrowed("hello");
cow.to_mut().push_str(" world"); // Clones on first mutation
// Now Cow::Owned(String)
```

Perfect for APIs that sometimes need to modify data - avoids unnecessary allocations in the common case.

## Connections
← [[008_clone_trait]]
→ [[081_to_owned_trait]]
→ [[082_deref_coercion]]

---
Level: L4
Date: 2025-08-15
Tags: #cow #smart-pointer #optimization #allocation