Title: Hacking Elixir's Syntax - PR TEST - THE NEXT GENERATION
Date: 2015-06-16 17:00
Category: Code
Slug: elixir-hacking
Tags: elixir, hacking, functional programming
Author: Roger Braun
Summary: Elixir won't allow us to use underscores in binary numbers. Let's add this feature.

[Elixir](http://elixir-lang.org/) is quickly becoming one of my favorite languages. It's a functional language that runs on the Erlang VM and has lots of things going for it, like fast response times, a Ruby-like syntax and pattern matching.

While toying around with the language, I needed to test the operation of a function on binary numbers. For example, if you OR two numbers, the resulting number will be number that results from all the bits in either number, like this:

```elixir
assert binary_or(0b10000000000000000, 0b0000000000000001) == 0b1000000000000001
```

If you are as bad at counting consecutive zeroes as I am, you'll probably have missed the error in the statement above[^1]: The first number has one to many zeroes. In Ruby, you can use underscores in all numbers to make them easier to read. This is how the above code would look like if Elixir supported this:

```ruby
assert binary_or(0b1000_0000_0000_0000_0, 0b0000_0000_0000_0001) == 0b1000_0000_0000_0001
```

The error becomes glaringly obvious. While Elixir does support underscores in decimal numbers, it doesn't support it in binary notation. Adding the support turned out to be not that hard, though.

The Elixir codebase was extremely easy to get running, just clone the repo and run `make clean test`. It worked out of the box for me, which is already a great experience. Some projects are extremely hard to build, but Elixir shines in this regard.

To find out where to look, I just used grep. The binary notation starts with '0b', so I hoped that there would be a test with these characters inside somewhere. Sure enough, this is the output of `grep 0b1 -R`:

```console
~/r/elixir (master ⚡=) grep -R '0b1' 
lib/elixir/lib/hash_dict.ex:  @node_bitmap 0b111
lib/elixir/lib/hash_set.ex:  @node_bitmap 0b111
lib/elixir/test/elixir/inspect_test.exs:    assert inspect(86, base: :binary) == "0b1010110"
lib/elixir/test/elixir/integer_test.exs:    assert Integer.undigits([1, 1], 2) == 0b11
lib/elixir/test/erlang/tokenizer_test.erl:  [{number, {1,1,5}, 3}] = tokenize("0b11"),
```

The tokenizer is what we are looking for. As you can see by the `.erl` extension, it's written in Erlang. Elixir's source code is a mix of Erlang and Elixir: The low-level parts are implemented in Erlang, but things like the standard library are written in Elixir itself.

In the file we find the following lines of testing code:

```erlang
hex_bin_octal_test() ->
  [{number, {1,1,5}, 255}] = tokenize("0xFF"),
  [{number, {1,1,5}, 63}] = tokenize("0o77"),
  [{number, {1,1,5}, 3}] = tokenize("0b11").
```

This is exactly what I was looking for, so I added the underscored variants, which should tokenize to the same result:


```erlang
hex_bin_octal_test() ->
  [{number, {1,1,5}, 255}] = tokenize("0xFF"),
  [{number, {1,1,5}, 255}] = tokenize("0xF_F"),
  [{number, {1,1,5}, 63}] = tokenize("0o77"),
  [{number, {1,1,5}, 63}] = tokenize("0o7_7"),
  [{number, {1,1,5}, 3}] = tokenize("0b11"),
  [{number, {1,1,5}, 3}] = tokenize("0b1_1").
```

Looking into the Makefile, we can see that `make test_erlang` will only run the Erlang tests. Using this we get the expected error:

```console
~/r/elixir (master ⚡=) make test_erlang
==> elixir (compile)
==> elixir (eunit)
tokenizer_test: hex_bin_octal_test...*failed*
in function tokenizer_test:hex_bin_octal_test/0 (lib/elixir/test/erlang/tokenizer_test.erl, line 32)
**error:{badmatch,[{number,{1,1,4},15},{identifier,{1,4,6},'_F'}]}


=======================================================
  Failed: 1.  Skipped: 0.  Passed: 197.
  Makefile:207: recipe for target 'test_erlang' failed
  make: *** [test_erlang] Error 1
```

The tokenizer choked on the `_F`, as expected. Let's try to make it work.

Guessing that the tokenizer would be implemented in a properly named file, I found `elixir_tokenizer.erl`. Reading through the code, I found the approriate functions around line 130. Here they are:

```erlang
% Base integers

tokenize([$0, $x, H|T], Line, Column, Scope, Tokens) when ?is_hex(H) ->
  {Rest, Number, Length} = tokenize_hex([H|T], []),
  tokenize(Rest, Line, Column + 2 + Length, Scope, [{number, {Line, Column, Column + 2 + Length}, Number}|Tokens]);

tokenize([$0, $b, H|T], Line, Column, Scope, Tokens) when ?is_bin(H) ->
  {Rest, Number, Length} = tokenize_bin([H|T], []),
  tokenize(Rest, Line, Column + 2 + Length, Scope, [{number, {Line, Column, Column + 2 + Length}, Number}|Tokens]);

tokenize([$0, $o, H|T], Line, Column, Scope, Tokens) when ?is_octal(H) ->
  {Rest, Number, Length} = tokenize_octal([H|T], []),
  tokenize(Rest, Line, Column + 2 + Length, Scope, [{number, {Line, Column, Column + 2 + Length}, Number}|Tokens]);
```

This code calls the `tokenize_{hex, bin, octal}` functions when the number starts with 0x, 0b or 0o. Let's take a look at those functions, further down in the file.

```erlang
tokenize_hex([H|T], Acc) when ?is_hex(H) -> tokenize_hex(T, [H|Acc]);
tokenize_hex(Rest, Acc) -> {Rest, list_to_integer(lists:reverse(Acc), 16), length(Acc)}.

tokenize_octal([H|T], Acc) when ?is_octal(H) -> tokenize_octal(T, [H|Acc]);
tokenize_octal(Rest, Acc) -> {Rest, list_to_integer(lists:reverse(Acc), 8), length(Acc)}.

tokenize_bin([H|T], Acc) when ?is_bin(H) -> tokenize_bin(T, [H|Acc]);
tokenize_bin(Rest, Acc) -> {Rest, list_to_integer(lists:reverse(Acc), 2), length(Acc)}.
```

The three functions are very similar. They put the current character into an accumulator list as long as it's a valid number for the given base. After that, they reverse the resulting accumulator and just call the `list_to_integer` function with the respective base.

The `is_bin`, `is_hex` and `is_octal` functions are actually macros. They are defined in `elixir.hrl`:

```erlang
%% Numbers
-define(is_hex(S), (?is_digit(S) orelse (S >= $A andalso S =< $F) orelse (S >= $a andalso S =< $f))).
-define(is_bin(S), (S >= $0 andalso S =< $1)).
-define(is_octal(S), (S >= $0 andalso S =< $7)).
```

It's easy to see why parsing `0xf_f` fails, `_` is neither a digit nor a letter from a to f. My first instinct was to add the underscore as a possible character to the `is_x` functions, but that seems wrong. Instead, I added another pattern to function definitions, one that would just drop any `_` that would be encountered while parsing these numbers:


```erlang
tokenize_hex([H|T], Acc) when ?is_hex(H) -> tokenize_hex(T, [H|Acc]);
tokenize_hex([$_|T], Acc) -> tokenize_hex(T, Acc);
tokenize_hex(Rest, Acc) -> {Rest, list_to_integer(lists:reverse(Acc), 16), length(Acc)}.

tokenize_octal([H|T], Acc) when ?is_octal(H) -> tokenize_octal(T, [H|Acc]);
tokenize_octal([$_|T], Acc) -> tokenize_octal(T, Acc);
tokenize_octal(Rest, Acc) -> {Rest, list_to_integer(lists:reverse(Acc), 8), length(Acc)}.

tokenize_bin([H|T], Acc) when ?is_bin(H) -> tokenize_bin(T, [H|Acc]);
tokenize_bin([$_|T], Acc) -> tokenize_bin(T, Acc);
tokenize_bin(Rest, Acc) -> {Rest, list_to_integer(lists:reverse(Acc), 2), length(Acc)}.
```

As you can see, if we now encounter an underscore, we don't modify the accumulator at all. This in effect strips out all underscores in our numbers. Let's run the tests:

```console
~/r/elixir (master ⚡=) make test_erlang
==> elixir (compile)
Compiled src/elixir_tokenizer.erl
==> elixir (eunit)
  All 198 tests passed.
```

And it works! Running `make clean test` shows that nothing else broke. The binaries in the `bin` folder now support the underscore notation:

```iex
~/r/elixir (master ⚡=) bin/iex
Erlang/OTP 17 [erts-6.4] [source] [64-bit] [smp:4:4] [async-threads:10] [hipe] [kernel-poll:false]

Interactive Elixir (1.1.0-dev) - press Ctrl+C to exit (type h() ENTER for help)
iex(1)> 0b1001_0001
145
```

Finding my way around Elixir's codebase was surprisingly easy. The source files are layed out logically and the code itself is clean and easy to understand.

This also shows how important it is for a project to have a comprehensive test suite. Thanks to the existing specs, it was easy to find the relevant pieces of code I needed to work on.

This exercise resulted in a [Pull Request](https://github.com/elixir-lang/elixir/pull/3395) and a slightly different version got merged, so look forward to this feature in upcoming versions of Elixir!

Please send me an [Email](mailto:blog@rogerbraun.net) if you want to comment!

[^1]: In fact, I made an error while producing this example, accidentally making it correct...
