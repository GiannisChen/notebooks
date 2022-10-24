## TODO List

### Tuples-Primitives

1. *Recap*: Add the `fmt::Display` trait to the `Matrix` struct in the above example, so that if you switch from printing the debug format `{:?}` to the display format `{}`, you see the following output:

   ```text
   ( 1.1 1.2 )
   ( 2.1 2.2 )
   ```

   You may want to refer back to the example for [print display](https://doc.rust-lang.org/stable/rust-by-example/hello/print/print_display.html).

2. Add a `transpose` function using the `reverse` function as a template, which accepts a matrix as an argument, and returns a matrix in which two elements have been swapped. For example:

   ```rust
   println!("Matrix:\n{}", matrix);
   println!("Transpose:\n{}", transpose(matrix));
   ```

   results in the output:

   ```text
   Matrix:
   ( 1.1 1.2 )
   ( 2.1 2.2 )
   Transpose:
   ( 1.1 2.1 )
   ( 1.2 2.2 )
   ```



[`attributes`](https://doc.rust-lang.org/stable/rust-by-example/attribute.html), and [destructuring](https://doc.rust-lang.org/stable/rust-by-example/flow_control/match/destructuring.html)

[`match`](https://doc.rust-lang.org/stable/rust-by-example/flow_control/match.html), [`fn`](https://doc.rust-lang.org/stable/rust-by-example/fn.html), and [`String`](https://doc.rust-lang.org/stable/rust-by-example/std/str.html), ["Type alias enum variants" RFC](https://rust-lang.github.io/rfcs/2338-type-alias-enum-variants.html)

[Library documentation](https://doc.rust-lang.org/stable/rust-by-example/meta/doc.html)



### Formatted Print

[`std::fmt`](https://doc.rust-lang.org/std/fmt/) contains many [`traits`](https://doc.rust-lang.org/std/fmt/#formatting-traits) which govern the display of text. The base form of two important ones are listed below:

- `fmt::Debug`: Uses the `{:?}` marker. Format text for debugging purposes.
- `fmt::Display`: Uses the `{}` marker. Format text in a more elegant, user friendly fashion.

Here, we used `fmt::Display` because the std library provides implementations for these types. To print text for custom types, more steps are required.

Implementing the `fmt::Display` trait automatically implements the [`ToString`](https://doc.rust-lang.org/std/string/trait.ToString.html) trait which allows us to [convert](https://doc.rust-lang.org/stable/rust-by-example/conversion/string.html) the type to [`String`](https://doc.rust-lang.org/stable/rust-by-example/std/str.html).

#### Activities

- Fix the issue in the above code (see FIXME) so that it runs without error.
- Try uncommenting the line that attempts to format the `Structure` struct (see TODO)
- Add a `println!` macro call that prints: `Pi is roughly 3.142` by controlling the number of decimal places shown. For the purposes of this exercise, use `let pi = 3.141592` as an estimate for pi. (Hint: you may need to check the [`std::fmt`](https://doc.rust-lang.org/std/fmt/) documentation for setting the number of decimals to display)

#### See also:

1. [`std::fmt`](https://doc.rust-lang.org/std/fmt/), [`macros`](https://doc.rust-lang.org/stable/rust-by-example/macros.html), [`struct`](https://doc.rust-lang.org/stable/rust-by-example/custom_types/structs.html), and [`traits`](https://doc.rust-lang.org/std/fmt/#formatting-traits)
2. [`attributes`](https://doc.rust-lang.org/reference/attributes.html), [`derive`](https://doc.rust-lang.org/stable/rust-by-example/trait/derive.html), [`std::fmt`](https://doc.rust-lang.org/std/fmt/), and [`struct`](https://doc.rust-lang.org/stable/rust-by-example/custom_types/structs.html)
3. [`derive`](https://doc.rust-lang.org/stable/rust-by-example/trait/derive.html), [`std::fmt`](https://doc.rust-lang.org/std/fmt/), [`macros`](https://doc.rust-lang.org/stable/rust-by-example/macros.html), [`struct`](https://doc.rust-lang.org/stable/rust-by-example/custom_types/structs.html), [`trait`](https://doc.rust-lang.org/std/fmt/#formatting-traits), and [`use`](https://doc.rust-lang.org/stable/rust-by-example/mod/use.html)
4. [`for`](https://doc.rust-lang.org/stable/rust-by-example/flow_control/for.html), [`ref`](https://doc.rust-lang.org/stable/rust-by-example/scope/borrow/ref.html), [`Result`](https://doc.rust-lang.org/stable/rust-by-example/std/result.html), [`struct`](https://doc.rust-lang.org/stable/rust-by-example/custom_types/structs.html), [`?`](https://doc.rust-lang.org/stable/rust-by-example/std/result/question_mark.html), and [`vec!`](https://doc.rust-lang.org/stable/rust-by-example/std/vec.html)
5. [The `const`/`static` RFC](https://github.com/rust-lang/rfcs/blob/master/text/0246-const-vs-static.md), [`'static` lifetime](https://doc.rust-lang.org/stable/rust-by-example/scope/lifetime/static_lifetime.html)
6. 