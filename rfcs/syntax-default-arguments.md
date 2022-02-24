# Default Argument Values

## Summary

Default argument values are a common feature of many programming languages.

```lua
function foo(bar:number=1)
    print(bar)
end
foo() --> 1
```

## Motivation

### Development Speed

The following is difficult to read.
* If you refer to the arguments later you need to just know that they have defaults written later on.
* If you're reading the defaults, you have to parse each argument name three times and make sure your logical expression is correct each time you want to express a possible default.
```lua
function foo(a, b, c, d)
    local a = a or foo()
    local b = b or a or 3
    local d = d or 123
```

Compare to the proposal. The default values of each argument and readily apparent at the place they are declared. It's faster to write and read and leaves less room for user error.
```lua
function foo(a=foo(), b=a or 3, c, d=123)
```

## Design

### Semantics
A default value is declared with a preceding `=` and a right-hand expression, which is evaluated and used as the argument if the input value is `nil`. Explicit and implicit nil both activate this behavior:

```lua
function foo(a=1)
    print(a)
end

foo(nil) --> 1
foo() --> 1
```

If only implicit nil activated the default value, then any argument to the left of an explicit argument wouldn't be able to take its default value. There is also precedence in `table.unpack` and other global functions that explicit nil should activate default values.

If an argument wouldn't receive the default value (i.e. the argument is nil), that argument's default expression is not evaluated.

### Evaluation Order
The values are executed in order as the function runs, as though they were a part of the function body. Only the arguments to the left of each expression are available as it runs, the rest are set to nil until they're evaluated.

```lua
function foo(a=1, b=a) -- a will default to 1, then b will default to a
function foo(a=b, b=1) -- a will default to nil because b is not defined yet, then b will default to 1
```

### Vararg
If the default value evaluates to multiple values (such as the `next` function), only the first argument is used. The vararg is an exception to this rule. If the vararg's default value evaluates to multiple values, they will all be passed into the vararg.

```lua
function foo()
    return 1, 2
end

function bar(a=foo(), ...=foo())
    print(a) --> 1
    print(...) --> 1 2
end

bar()
```

It may also be beneficial to simply write an expression that evaluates to multiple values as the default value of vararg, like so:

```lua
function foo(...=1,2)
```

This currently would error because the comma in this expression would be interpreted as a new argument. The new behavior will be that after the vararg declaration and the default assignment `=`, everything within the function arguments will be interpreted as a part of the vararg's default value.

## Drawbacks

There is potential weirdness with the handling of `nil`. It is not necessarily obvious that sending an explicit nil argument will activate the default value, and this could be confusing in a situation where the developer calls a function using a value that they expect to be non-nil, then it gets overridden by the default and the value appears to be something wildly different than intended.

## Alternatives

### Keyword
Another possible alternative is using a keyword rather than `=`.

```lua
function foo(a or 1)
```

The `or` keyword would make lexical sense and it's an existing keyword, so it could fit here. This is not the primary suggestion because `or` has a different meaning on the right-hand side of this structure, so something like `function foo(a or x() or y())` may be confusing to read. It also would seem to imply that the right-side expression is evaluated when the function is defined, which is not the case.

### Vararg compound default expression
Vararg's multiple default behavior may be confusing. It may make sense to force the developer to wrap vararg's expression in parentheses if they want to express multiple values.

```lua
function foo(...=(1,2))
```