# Assignment Expression

## Summary

Allow assignment as an expression so that it can be performed within condition checks, function arguments, and anywhere else expressions are evaluated.

```lua
if local value = foo() then
    bar(value)
end
```

## Motivation

Assignment as an expression is not an uncommon feature in programming languages.

Consider the following: You must check if something exists and, if it does, perform an operation on it. There are currently three options:

### Wrong Block Solution
Option 1 is to put the variable in the wrong block. This can get extremely messy with a significant amount of elseifs, where you would need to declare several variables above the block series that you likely won't even be using. This is not ideal because the variable isn't even used in the block it's declared in, except to perform the condition check to enter the sub-block.
```lua
local value = foo() -- value is never used in this block, except for within the condition!
if value then
    bar(value)
end
```

### Get Twice Solution
Option 2 is to call the getter twice. This assumes that the getter is strictly returning a new value and not performing any operations that shouldn't be performed twice. This is very difficult to track and maintain. It's not practical for a sufficiently-complicated codebase.
```lua
if foo() then
    bar(foo()) -- foo is called twice!
end
```

### Callback Function Solution
Option 3 is to use some contrived function in place of an assignment operator. This type of construction is hard to understand if you don't already recognize it. It also must be constructed differently for each use. For instance, the function in this example doesn't work for a while loop.
```lua
function ifthen(value, f)
    if value then f(value) end
end

ifthen(foo(), function(value) -- time to consult documentation!
    bar(value)
end)
```

### Environment Modifier Solution
Option 4 is to implement assignment expressions yourself by modifying the function environment. This disposes of internal optimizations and introduces untrackable changes to the environment. This is not practical for any serious project.
```lua
function assign(name, value)
    getfenv(1)[name] = value
    return value
end

if assign("value", foo()) then -- environment editing!!!
    bar(value)
end
```

## Design

Enter the assignment expression. It looks like an assignment, but you use it where you would use an expression. The example above becomes trivial to write and painless to read.

All assignment operators work with it and it can be used in a variety of situations.

```lua
-- while loops!
while local value = foo() do
    bar(value)
end

-- repeat loops!
local value = 1
repeat
    bar(value)
until value *= 4 > 1234

-- for loops!
for i, v in local value = foo() do
    bar(value[i-1], v)
end

-- or even use it without starting a block!
bar(local value = foo())

-- you can even use multiple at once!
if local a = foo() and local b = foo2() then
    bar(a, b)
end

-- multiple uses in one expression apply as executed
bar(local a = foo() * local b = foo() + a/b)

-- the variable doesn't have to be local
if value = foo() then
    bar(value)
end

-- table members work too
if humanoid.Health -= 5 <= 0 then
    bar(humanoid.Health)
end

-- as well as function calls
assert(local value = foo(), "Foo returned falsey")
```

### Precedence

Assignment operators take precedence over all numerical and logical operators. `if x = 5 + 5` will evaluate to `x = 5` and then `if x + 5`.

## Drawbacks

### Assignment within until
The behavior of a `local` assignment within an `until` evaluation might be surprising initially. The value won't be available during the next loop cycle because it gets added to the scope and then immediately terminated when the condition evaluation completes.

### Precedence
Also, the precedence order of the operation may be confusing. Since assignment takes precedence over numerical and logical operators, one might assume `if x = 5 + 5` will read as `if x = (5 + 5)` when it actually reads as `if (x = 5) + 5`. This is a relatively simple fix with parentheses, however.

## Alternatives

The alternatives users implement can be found in the Motivation section above.

### = Ambiguity
A more explicit approach to the syntax may reduce confusion.

```lua
if assign value as foo() then
```

This has the drawback of introducing keywords that are certainly in use in user code. Another alternative is a global function which could take a similar form to the environment modifier solution, but this breaks a convention that variables aren't accessed as strings. To fix that we'd need a pointer to a variable, which is very much out of scope for this proposal.

```lua
if assign("value", foo()) then
```