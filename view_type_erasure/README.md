# Modest type erasure for "subrange-y" types in combination with "subrange-y" view adaptors


| Foo               | Bar                                               |
|-------------------|---------------------------------------------------|
| Document Number:  | tba                                               |
| Date:             | 2019-05-16                                        |
| Project:          | Programming Language C++, Library Working Group   |
| Reply-To:         | Hannes Hauswedell (h2 AT fsfe.org)                |


## Introduction

The current draft standard introduces many range adaptor objects [24.7].
Upon invocation, most of these are specified to return a type that is specified together with the adaptor, e.g. std::view::take returns a specialization of std::ranges::take_view.
Chaining multiple adaptors thus leads to an increasingly nested type.
This proposal suggests avoiding nested return types where this is *reasonable*.

## Motivation and Scope

Given the following example:

```cpp
vector vec{1, 2, 3, 4, 5};
span s{vec.data(), 5};
auto v = s | view::drop(1) | view::take(10)
           | view::drop(1) | view::take(10);
```

The type of `v` will be something similar to
```
ranges::take_view<ranges::drop_view<ranges::take_view<ranges::drop_view<span<int, -1> > > > >
```

Although in fact it could just be `span<int, -1>`, i.e. the same as the original type of `s`.
This is also true if the input to the series of pipe operations is a specialisation of `basic_string_view` and also for certain specialisations of `ranges::subrange`.

All of these types have in common that they already are "subrange-y" and it would feel natural to preserve the original type if only the size of the "subrange" is changed, i.e. `s | view::drop(1) | view::take(3)` should be expression equivalent to `s.subspan(1, 3)` if `s` is a `span`.

We propose that `view::drop` and `view::take` return a range of the same type as the input type and not return a nested type if the input is one of
  * a `span` of dynamic extent;
  * a `basic_string_view`;
  * a `ranges::subrange` if it models `std::ranges::RandomAccessRange` and `std::ranges::SizedRange`

We also suggest that the `view::counted` range factory return a `span` of dynamic extent (instead of `ranges::subrange`) if the iterator passed to it models `std::ContiguousIterator`.

The proposed changes will strongly reduce the amount of template instantiation and code generation in situations where `view::drop` and `view::take` are applied repeatedly.
They increase the integration of the view machinery from `std::ranges` with the views created in the standard independently (`span` and `basic_string_view`).
They increase the teachability and learnability of views in general, because some of the simplest view operations now return simpler types and behave exactly as the already documented member functions.

## Impact on the Standard

This proposal only includes changes to the draft standard library.
It proposes no changes to that would be breaking to the international standard and it requires no changes to the core language.

It does target C++20, because applying these changes afterwards would be breaking to the international standard.


## Design Decisions

## Technical Specifications


## Acknowledgements

## References
