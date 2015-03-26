Fast linear time & space suffix arrays for Rust. Supports Unicode!

[![Build status](https://api.travis-ci.org/BurntSushi/suffix.png)](https://travis-ci.org/BurntSushi/suffix)

Licensed under the [UNLICENSE](http://unlicense.org).


### Documentation

The API is mostly documented with examples:
[http://burntsushi.net/rustdoc/suffix/](http://burntsushi.net/rustdoc/suffix/).

If you just want the details on how construction algorithm used, see the
documentation for the
[`SuffixTable`](http://burntsushi.net/rustdoc/suffix/struct.SuffixTable.html)
type. This is where you'll find info on exactly how much overhead is required.


### Installation

This crate works with Cargo and is on
[crates.io](https://crates.io/crates/suffix). The package is regularly updated.
Add it to your `Cargo.toml` like so:

```toml
[dependencies]
suffix = "*"
```


### Examples

Usage is simple. Just create a suffix array and search:

```rust
extern crate suffix;

use suffix::SuffixTable;

fn main() {
  let st = SuffixTable::new("the quick brown fox was quick.");
  assert_eq!(st.positions("quick"), vec![4, 24]);
}
```

There is also a command line program, `stree`, that can be used to visualize
suffix trees:

```bash
git clone git://github.com/BurntSushi/suffix
cd suffix/stree_cmd
cargo build --release
./target/release/stree "banana" | dot -Tpng | xv -
```

And here's what it looks like:

!["banana" suffix tree](http://burntsushi.net/stuff/banana.png)


### Status of implementation

The big thing missing at the moment is a generalized suffix array. I started
out with the intention to build them into the construction algorithm, but this
has proved more difficult than I thought.

A kind-of-sort-of compromise is to append your distinct texts together, and
separate them with a character that doesn't appear in your document. (This is
technically incorrect, but maybe your documents don't contain any `NUL`
characters.) During construction of this one giant string, you should record
the offsets of where each document starts and stops. Then build a `SuffixTable`
with your giant string. After searching with the `SuffixTable`, you can find
the original document by doing a binary search on your list of documents.

I'm currently experimenting with different techniques to do this.


### Benchmarks

Here are some very rough benchmarks that compare suffix table searching with
searching in the using standard library functions. Note that these benchmarks
explicitly do not include the construction of the suffix table. The premise of
a suffix table is that you can afford to do that once---but you hope to gain
much faster queries once you do.

```
test search_scan_exists_many            ... bench:      4301 ns/iter (+/- 68)
test search_scan_exists_one             ... bench:        26 ns/iter (+/- 0)
test search_scan_not_exists             ... bench:    131761 ns/iter (+/- 255)
test search_suffix_exists_many          ... bench:       371 ns/iter (+/- 3)
test search_suffix_exists_many_contains ... bench:       132 ns/iter (+/- 3)
test search_suffix_exists_one           ... bench:       193 ns/iter (+/- 1)
test search_suffix_exists_one_contains  ... bench:        16 ns/iter (+/- 0)
test search_suffix_not_exists           ... bench:       165 ns/iter (+/- 1)
test search_suffix_not_exists_contains  ... bench:       106 ns/iter (+/- 0)
```

The "many" benchmarks test repeated queries that match. The "one" benchmarks
test a single query that matches. The "not_exists" benchmarks test a single
query that does *not* match. Finally, the "contains" benchmark test existence
rather finding all positions.

One thing you might take away from here is that you'll get a very large
performance boost if many of your queries don't match. A linear scan takes a
long time to fail!

And here are some completely useless benchmarks on suffix array construction.
They compare the linear time algorithm with the naive construction algorithm
(call `sort` on all suffixes, which is `O(n^2 * logn)`).

```
test naive_dna_medium                   ... bench:  24657031 ns/iter (+/- 112718)
test naive_dna_small                    ... bench:   1868072 ns/iter (+/- 3110)
test naive_small                        ... bench:       296 ns/iter (+/- 2)
test sais_dna_medium                    ... bench:  10889339 ns/iter (+/- 42678)
test sais_dna_small                     ... bench:   1021010 ns/iter (+/- 3746)
test sais_small                         ... bench:      5286 ns/iter (+/- 58)
```

These benchmarks might make you say, "Whoa, the special algorithm isn't that
much faster." That's because the data just isn't big enough. And when it *is*
big enough, a micro benchmark is useless. Why? Because using the `naive`
algorithm will just burn your CPUs until the end of the time.

It would be more useful to compare this to other suffix array implementations,
but I haven't had time yet. Moreover, most (all?) don't support Unicode and
instead operate on bytes, which means they aren't paying the overhead of
decoding UTF-8.
