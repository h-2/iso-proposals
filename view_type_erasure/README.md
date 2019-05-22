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

All of these types have in common that they already are "subrange-y" and it would feel natural to preserve the original type if only the size of the "subrange" is changed, i.e. `s | view::drop(1) | view::take(3)` should be equivalent to `s.subspan(1, 3)` if `s` is a `span`.

We propose that `view::drop` and `view::take` return a range of the same type as the input type and not return a nested type if the input is one of
  * a `span` of dynamic extent;
  * a `basic_string_view`;
  * a `ranges::subrange` if it models `std::ranges::RandomAccessRange` and `std::ranges::SizedRange`

We also propose that the `view::counted` range factory return a `span` of dynamic extent (instead of `ranges::subrange`) if the iterator passed to it models `std::ContiguousIterator`.

The proposed changes will strongly reduce the amount of template instantiation in situations where `view::drop` and `view::take` are applied repeatedly.
They increase the integration of the view machinery from `std::ranges` with the views created in the standard independently (`span` and `basic_string_view`).
And they improve the teachability and learnability of views in general, because some of the simplest view operations now return simpler types and behave exactly as the already documented member functions.

## Impact on the Standard

This proposal only includes changes to features from the draft standard library and not the published international standard. It requires no changes to the core language.

It does target C++20, because applying these changes afterwards would be breaking.


## Design Decisions

### Possible arguments against this proposal

Following this proposal means that it will not hold any longer that *"the expression `view::take(E, F)` is expression-equivalent to `take_­view{E, F}`"*, i.e. the adaptor has behaviour that is distinct from the associated view class template.
This may be a source of confusion, however by design of the current view machinery users are expected to interact with "the view" almost exclusively through the adaptor/factory object.
And this is not the first proposal to define such specialised behaviour of an adaptor depending on its input: [TODO insert] already proposes that applying `view::reverse` on a specialisation of `std::ranges::reverse_view` undos the previous reversal instead of applying it a second time.
The added flexibility of the adaptor approach should be seen as a feature.

While specialisations of e.g. `ranges::take_view` provide a `base()` member function that allows accessing the underlying range, this is not true for `span` or the other return types that we are proposing.
However, since the underlying range is of the same type as the returned range, we see little value in re-transforming the range to its original type.
It should also be noted that this "feature" (a `base()` member function) is provided by some of the views in the draft standard, but it is not a part of the general view design and it cannot be relied on in generic contexts and combinations with other views.
No parts of the standard currently make use of it and according to Casey Carter it is not 

### Stronger proposals

This proposal originally included also changing `view::all` to type-erase `basic_string` constants to `basic_string_view`; all forwarding, contiguous ranges to `span`; and all forwarding, sized, random-access ranges to `ranges::subrange`.
`view::all` is the "entry-point" for all non-views in a series of view operations and otherwise wraps non-view-ranges in `ranges::ref_view`.
Changing `view::all` in this manner would have a much stronger type-erasing effect, e.g. `s | view::drop(1) | view::take(3)` would return a `span`, independent of whether s is a `span`, a `vector` or an `array`.
I think that this change would be benificial for the same reasons as the above changes.
However, the possible impact is more significant, because...

## Technical Specifications


## Acknowledgements

## References

