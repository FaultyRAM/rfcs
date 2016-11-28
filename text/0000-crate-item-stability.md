- Feature Name: crate_item_stability
- Start Date: 2016-11-28
- RFC PR: 
- Rust Issue: 

# Summary
[summary]: #summary

This RFC introduces the concept of *stability* for crates and items, and adds three attributes,
`#[unstable]`, `#[stable]` and `#[reserved]` to control the stability of items.

# Motivation
[motivation]: #motivation

Over time libraries evolve, adding new features, stabilizing mature features, and deprecating old
ones. The ability to catagorise APIs accordingly is therefore highly desirable to library authors.
[RFC 1270][RFC 1270] introduces the ability to mark an item as *deprecated*, but
the concept of *stability* remains unknown to Rust. This RFC aims to rectify that, by clearly
defining the rules of stability and providing library authors with simple, easy-to-use methods of
controlling stability.

# Detailed design
[design]: #detailed-design

Crates and items are either *stable* or *unstable*. In short, the stability of items is
controlled via the stability attributes (described below), while the stability of crates is
based on a number of factors. The attributes and rules that govern stability are detailed below.

## Stability Attributes
[stability-attributes]: #stability-attributes

A publically-visible item can be marked with one of the *stability attributes*: `#[unstable]`,
`#[stable]` or `#[reserved]`. Additionally, [`#[deprecated]`][RFC 1270] shall henceforth be
recognised as a stability attribute. It is an error for an item to be marked with more than one
stability attribute. `#[unstable]`, `#[stable]` and `#[reserved]` each have the following fields,
all of which are optional:

* `since` is a semver-compliant version number, representing the version of the crate at the time
  the item was marked with the given attribute.
* `note` is a human-readable string briefly summarizing why the item is marked with the given
  attribute. Even though it is currently optional, authors are strongly encouraged to use it. For
  now, it shall be interpreted as plain unformatted text so that `rustdoc` can include it without
  breaking any formatting.

## Item Stability Rules
[item-stability-rules]: #item-stability-rules

An item is *stable* if either of the following is true:

* It belongs to an unstable crate and is marked with `#[stable]`.
* It belongs to a stable crate and is not marked with any stability attribute, other than
  `#[stable]`.

Otherwise, it is *unstable*.

In the event that an unstable item is used, the compiler shall obey the following rules:

* If the item belongs to a stable crate, emit a warning.
* If the item belongs to an unstable crate, do not emit a warning unless the user requests that
  the compiler do so in such situations. (A future RFC may alter this behavior so that a warning
  is issued regardless of a crate's stability).

## Crate Stability Rules
[crate-stability-rules]: #crate-stability-rules

A crate is *stable* if it meets both of the following requirements:

* Its version is >= `1.0.0` (a stable release per semver).
* It contains either no publically-visible items, or at least one publically-visible stable
  item.

Otherwise, it is *unstable*. It is an error for a crate to obey the first requirement but not
the second.

In the event that an unstable crate is used, the compiler shall not emit a warning unless the
user requests that the compiler do so in such situations. (While a future RFC may change this
behavior, note that currently a large number of crates on [crates.io][crates-io] are unstable
per the above requirements, and any such change would be a significant breaking change.)

# Drawbacks
[drawbacks]: #drawbacks

* Once this feature is public, its design is forever set in stone.

# Alternatives
[alternatives]: #alternatives

* Do nothing.

# Unresolved questions
[unresolved]: #unresolved-questions

* Rust uses its own stability attributes internally. What will happen to these?

[RFC 1270]: text/1270-deprecation.md
[crates-io]: https://crates.io
