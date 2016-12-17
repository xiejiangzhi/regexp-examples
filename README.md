# regexp-examples
[![Gem Version](https://badge.fury.io/rb/regexp-examples.svg)](http://badge.fury.io/rb/regexp-examples)
[![Build Status](https://travis-ci.org/tom-lord/regexp-examples.svg?branch=master)](https://travis-ci.org/tom-lord/regexp-examples/builds)
[![Coverage Status](https://coveralls.io/repos/tom-lord/regexp-examples/badge.svg?branch=master)](https://coveralls.io/r/tom-lord/regexp-examples?branch=master)
[![Code Climate](https://codeclimate.com/github/tom-lord/regexp-examples/badges/gpa.svg)](https://codeclimate.com/github/tom-lord/regexp-examples)

Extends the `Regexp` class with the methods: `Regexp#examples` and `Regexp#random_example`

`Regexp#examples` generates a list of all\* strings that will match the given regular expression.

`Regexp#random_example` returns one, random string (from all possible strings!!) that matches the regex.

\* If the regex has an infinite number of possible srings that match it, such as `/a*b+c{2,}/`,
or a huge number of possible matches, such as `/.\w/`, then only a subset of these will be listed.

For more detail on this, see [configuration options](#configuration-options).

If you'd like to understand how/why this gem works, please check out my [blog post](http://tom-lord.weebly.com/blog/reverse-engineering-regular-expressions) about it.

## Usage

#### Regexp#examples

```ruby
/a*/.examples #=> ['', 'a', 'aa']
/ab+/.examples #=> ['ab', 'abb', 'abbb']
/this|is|awesome/.examples #=> ['this', 'is', 'awesome']
/https?:\/\/(www\.)?github\.com/.examples #=> ['http://github.com',
  # 'http://www.github.com', 'https://github.com', 'https://www.github.com']
/(I(N(C(E(P(T(I(O(N)))))))))*/.examples #=> ["", "INCEPTION", "INCEPTIONINCEPTION"]
/\x74\x68\x69\x73/.examples #=> ["this"]
/what about (backreferences\?) \1/.examples
  #=> ['what about backreferences? backreferences?']
/
  \u{28}\u2022\u{5f}\u2022\u{29}
  |
  \u{28}\u{20}\u2022\u{5f}\u2022\u{29}\u{3e}\u2310\u25a0\u{2d}\u25a0\u{20}
  |
  \u{28}\u2310\u25a0\u{5f}\u25a0\u{29}
/x.examples #=> ["(•_•)", "( •_•)>⌐■-■ ", "(⌐■_■)"]
```

#### Regexp#random_example

Obviously, you will get different (random) results if you try these yourself!

```ruby
/\w{10}@(hotmail|gmail)\.com/.random_example #=> "TTsJsiwzKS@gmail.com"
/5[1-5][0-9]{14}/.random_example #=> "5224028604559821" (A valid MasterCard number)
/\p{Greek}{80}/.random_example
  #=> "ΖΆΧͷᵦμͷηϒϰΟᵝΔ΄θϔζΌψΨεκᴪΓΕπι϶ονϵΓϹᵦΟπᵡήϴϜΦϚϴϑ͵ϴΉϺ͵ϹϰϡᵠϝΤΏΨϹϊϻαώΞΰϰΑͼΈΘͽϙͽξΆΆΡΡΉΓς"
/written by tom lord/i.random_example #=> "WrITtEN bY tOM LORD"
```

## Supported ruby versions
* MRI 2.0.x
* MRI 2.1.x
* MRI 2.2.x
* MRI 2.3.x
* MRI 2.4.0-dev

MRI ≤ 1.9.3 are not supported. This is primarily because MRI 2.0.0 introduced a new
regexp engine (`Oniguruma` was replaced by `Onigmo`). Whilst *most* of this gem could
be made to work with MRI 1.9.x (or even 1.8.x), I feel the changes are too significant
to implement backwards compatability (especially since [long-term support for MRI
1.9.3 has now ended](https://www.ruby-lang.org/en/news/2014/01/10/ruby-1-9-3-will-end-on-2015/)).

For example, named properties (e.g. `/\p{Alpha}/`) are illegal syntax on MRI 1.9.3.

Other implementations, such as JRuby, could probably work fine -
but I haven't fully tried/tested it. Pull requests are welcome.

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'regexp-examples'
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install regexp-examples

## Supported syntax

Short answer: **Everything** is supported, apart from "irregular" aspects of the regexp language -- see [impossible features](#impossible-features-illegal-syntax).

Long answer:

* All forms of repeaters (quantifiers), e.g. `/a*/`, `/a+/`, `/a?/`, `/a{1,4}/`, `/a{3,}/`, `/a{,2}/`
  * Reluctant and possissive repeaters work fine, too, e.g. `/a*?/`, `/a*+/`
* Boolean "Or" groups, e.g. `/a|b|c/`
* Character sets, e.g. `/[abc]/` - including:
  * Ranges, e.g.`/[A-Z0-9]/`
  * Negation, e.g. `/[^a-z]/`
  * Escaped characters, e.g. `/[\w\s\b]/`
  * POSIX bracket expressions, e.g. `/[[:alnum:]]/`, `/[[:^space:]]/`
  * Set intersection, e.g. `/[[a-h]&&[f-z]]/`
* Escaped characters, e.g. `/\n/`, `/\w/`, `/\D/` (and so on...)
* Capture groups, e.g. `/(group)/`
  * Including named groups, e.g. `/(?<name>group)/`
  * And backreferences(!!!), e.g. `/(this|that) \1/` `/(?<name>foo) \k<name>/`
  * ...even for the more "obscure" syntax, e.g. `/(?<future>the) \k'future'/`, `/(a)(b) \k<-1>/``
  * ...and even if nested or optional, e.g. `/(even(this(works?))) \1 \2 \3/`, `/what about (this)? \1/`
  * Non-capture groups, e.g. `/(?:foo)/`
  * Comment groups, e.g. `/foo(?#comment)bar/`
* Control characters, e.g. `/\ca/`, `/\cZ/`, `/\C-9/`
* Escape sequences, e.g. `/\x42/`, `/\x5word/`, `/#{"\x80".force\_encoding("ASCII-8BIT")}/`
* Unicode characters, e.g. `/\u0123/`, `/\uabcd/`, `/\u{789}/`
* Octal characters, e.g. `/\10/`, `/\177/`
* Named properties, e.g. `/\p{L}/` ("Letter"), `/\p{Arabic}/` ("Arabic character")
, `/\p{^Ll}/` ("Not a lowercase letter"), `/\P{^Canadian\_Aboriginal}/` ("Not not a Canadian aboriginal character")
  * ...Even between different ruby versions!! (e.g. `/\p{Arabic}/.examples(max_group_results: 999)` will give you a different answer in ruby v2.1.x and v2.2.x)
* **Arbitrarily complex combinations of all the above!**

* Regexp options can also be used:
  * Case insensitive examples: `/cool/i.examples #=> ["cool", "cooL", "coOl", "coOL", ...]`
  * Multiline examples: `/./m.examples #=> ["\n", "a", "b", "c", "d"]`
  * Extended form examples: `/line1 #comment \n line2/x.examples #=> ["line1line2"]`
  * Options toggling supported: `/before(?imx-imx)after/`, `/before(?imx-imx:subexpr)after/`

##Configuration Options

When generating examples, the gem uses 3 configurable values to limit how many examples are listed:

* `max_repeater_variance` (default = `2`) restricts how many examples to return for each repeater. For example:
  * `.*` is equivalent to `.{0,2}`
  * `.+` is equivalent to `.{1,3}`
  * `.{2,}` is equivalent to `.{2,4}`
  * `.{,3}` is equivalent to `.{0,2}`
  * `.{3,8}` is equivalent to `.{3,5}`

* `max_group_results` (default = `5`) restricts how many characters to return for each "set". For example:
  * `\d` is equivalent to `[01234]`
  * `\w` is equivalent to `[abcde]`
  * `[h-s]` is equivalent to `[hijkl]`
  * `(1|2|3|4|5|6|7|8)` is equivalent to `[12345]`

* `max_results_limit` (default = `10000`) restricts the maximum number of results that can possibly be generated. For example:
  * `/c+r+a+z+y+ * B+I+G+ * r+e+g+e+x+/i.examples.length <= 10000` -- Attempting this will NOT freeze your system, even though
  (by the above rules) this "should" attempt to generate **117546246144** examples.

`Rexexp#examples` makes use of *all* these options; `Rexexp#random_example` only uses `max_repeater_variance`, since the other options are redundant.

To use an alternative value, simply pass the configuration option as follows:

```ruby
/a*/.examples(max_repeater_variance: 5)
  #=> [''. 'a', 'aa', 'aaa', 'aaaa' 'aaaaa']
/[F-X]/.examples(max_group_results: 10)
  #=> ['F', 'G', 'H', 'I', 'J', 'K', 'L', 'M', 'N', 'O']
/[ab]{10}/.examples(max_results_limit: 64).length == 64 # NOT 1024
/[slow]{9}/.examples(max_results_limit: 9999999).length == 4 ** 9 == 262144 # Warning - this will take a while!
/.*/.random_example(max_repeater_variance: 50)
  #=> "A very unlikely result!"
```

A sensible use case might be, for example, to generate all 1-5 digit strings:

```ruby
/\d{1,5}/.examples(max_repeater_variance: 4, max_group_results: 10, max_results_limit: 100000)
  #=> ['0', '1', '2', ..., '99998', '99999']
```

Due to code optimisation, `Regexp#random_example` runs pretty fast even on very complex patterns.
(I.e. It's typically a _lot_ faster than using `/pattern/.examples.sample(1)`.)
For instance, the following takes no more than ~ 1 second on my machine:

`/.*\w+\d{100}/.random_example(max_repeater_variance: 1000)`

## Bugs and TODOs

There are no known major bugs with this library. However, there are a few obscure issues that you *may* encounter:

* Conditional capture groups, e.g. `/(group1)? (?(1)yes|no)/.examples` are not yet supported. (This example *should* return: `["group1 yes", " no"]`)
* `\Z` should be interpreted like `\n?\z`; it's currently just interpreted like `\z`. (This basically just means you'll be missing a few examples.)
* Ideally, `regexp#examples` should always return up to `max_results_limit`. Currenty, it usually "aborts" before this limit is reached.
 (I.e. the exact number of examples generated can be hard to predict, for complex patterns.)
* There are some (rare) edge cases where backreferences do not work properly, e.g. `/(a*)a* \1/.examples` -
 which includes `"aaaa aa"`. This is because each repeater is not context-aware, so the "greediness" logic is flawed.
 (E.g. in this case, the second `a*` should always evaluate to an empty string, because the previous `a*` was greedy.)
 However, patterns like this are highly unusual...
* Nested repeat operators are incorrectly parsed, e.g. `/b{2}{3}/` - which *should* be interpreted like `/b{6}/`. (However, there is probably no reason
 to ever write regexes like this!)

Some of the most obscure regexp features are not even mentioned in [the ruby docs](http://ruby-doc.org/core/Regexp.html).
However, full documentation on all the intricate obscurities in the ruby (version 2.x) regexp parser can be found
[here](https://raw.githubusercontent.com/k-takata/Onigmo/master/doc/RE).

## Impossible features ("illegal syntax")

The following features in the regex language can never be properly implemented into this gem because, put simply, they are not technically "regular"!
If you'd like to understand this in more detail, check out what I had to say in [my blog post](http://tom-lord.weebly.com/blog/reverse-engineering-regular-expressions) about this gem.

Using any of the following will raise a `RegexpExamples::IllegalSyntax` exception:

* Lookarounds, e.g. `/foo(?=bar)/`, `/foo(?!bar)/`, `/(?<=foo)bar/`, `/(?<!foo)bar/`
* [Anchors](http://ruby-doc.org/core/Regexp.html#class-Regexp-label-Anchors) (`\b`, `\B`, `\G`, `^`, `\A`, `$`, `\z`, `\Z`), e.g. `/\bword\b/`, `/line1\n^line2/`
  * Anchors are really just special cases of lookarounds!
  * However, a special case has been made to allow `^`, `\A` and `\G` at the start of a pattern; and to allow `$`, `\z` and `\Z` at the end of pattern. In such cases, the characters are effectively just ignored.
* Subexpression calls (`\g`), e.g. `/(?<name> ... \g<name>* )/`

(Note: Backreferences are not really "regular" either, but I got these to work with a bit of hackery.)

## Contributing

1. Fork it ( https://github.com/tom-lord/regexp-examples/fork )
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create a new Pull Request
6. Don't forget to add tests!!
