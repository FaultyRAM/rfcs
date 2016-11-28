- Feature Name: crate_item_stability
- Start Date: 2016-11-28
- RFC PR: 
- Rust Issue: 

# Summary
[summary]: #summary

This RFC introduces the concept of *stability* for crates and items, and adds two attributes,
`#[unstable]` and `#[stable]`, to control the stability of items.

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

Crates and items have a property called *stability*. The stability of a crate is either *stable*
or *unstable*. The stability of an item is either *stable*, *unstable* or *unspecified*. The
stability of items is controlled via the stability attributes (described below), while the
stability of crates is based on a number of factors. The attributes and rules that govern
stability are detailed below.

## Stability Attributes
[stability-attributes]: #stability-attributes

A publically-visible item can be marked with one or more of the *stability attributes*, listed
below.

* An `#[unstable]` item is unstable, and may change in future versions of the crate to which the
  item belongs.
* A `#[stable]` item is stable, and not expected to change before the next major version of the
  crate to which the item belongs.
* Additionally, [`#[deprecated]`][RFC 1270] shall henceforth be recognised as a stability
  attribute. A `#[deprecated]` item is unstable, and expected to be removed in a future version
  of the crate to which the item belongs.

Each of the stability attributes has the following fields:

* `since` is a semver-compliant version number, representing the version of the crate at the
  time the item was marked with the given attribute. This field is required, but for now if the
  item is marked with exactly one stability attribute, the compiler shall instead emit a warning
  if this field is absent; a future RFC shall promote this to a hard error. It is an error for
  two or more of an item's stability attributes to have identical values for this field.
* `note` is a human-readable string briefly summarizing why the item is marked with the given
  attribute. This field is optional, but authors are strongly encouraged to use it. For now, it
  shall be interpreted as plain unformatted text so that `rustdoc` can include it without
  breaking any formatting.

## Item Stability Rules
[item-stability-rules]: #item-stability-rules

The stability of an item is determined by following the steps below:

1. Sort the item's stability attributes from earliest version to latest version.
2. If the latest stability attribute is a `#[stable]` attribute, the item is stable.
3. If the latest stability attribute is any other stability attribute, the item is unstable.
4. Otherwise, the item is unspecified.

In the event that an unstable item is used, the compiler shall obey the following rules:

* If the item belongs to a stable crate, emit a warning.
* If the item belongs to an unstable crate, do not emit a warning unless the user requests that
  the compiler do so in such situations. (A future RFC may alter this behavior so that a warning
  is issued regardless of a crate's stability).

In the event that an unspecified item is used, the compiler shall obey the following rules:

* If the item belongs to a stable crate, do not emit a warning. (Currently such items are
  considered stable by library users, so it is inapproriate to consider their usage as
  problematic.)
* If the item belongs to an unstable crate, do not emit a warning unless the user requests that
  the compiler do so in such situations. (A future RFC may alter this behavior so that a warning
  is always issued in this situation).

## Crate Stability Rules
[crate-stability-rules]: #crate-stability-rules

A crate is stable if it meets both of the following requirements:

* Its version is >= `1.0.0` (a stable release per semver).
* It contains either no publically-visible items, or at least one publically-visible stable
  item. It is an error for a >= `1.0.0` crate to not obey this requirement, but for now the
  compiler shall only emit a warning when compiling such a crate; a future RFC shall promote
  this to a hard error.

Otherwise, it is unstable.

In the event that an unstable crate is used, the compiler shall not emit a warning unless the
user requests that the compiler do so in such situations. (While a future RFC may change this
behavior, note that currently a large number of crates on [crates.io][crates-io] are unstable
per the above requirements, and any such change would be a significant breaking change.)

# Drawbacks
[drawbacks]: #drawbacks

* Breaking changes: the `since` field of `#[deprecated]` is now required, and the compiler now
  ensures that its value is semver-compliant. Stable crates must also mark all
  publically-visible items with one of the stability attributes, or risk being broken by a
  future version of Rust.
* Once this feature is public, its design is forever set in stone.

# Alternatives
[alternatives]: #alternatives

* Do nothing.

# Unresolved questions
[unresolved]: #unresolved-questions

* Rust uses its own stability attributes internally. What will happen to these?

[RFC 1270]: 1270-deprecation.md
[crates-io]: https://crates.io
