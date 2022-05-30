# Variable Member Assignment

## Summary

Provide a way to assign a member function (which uses `:`) with a variable key.

## Motivation

There is currently a feature gap in member assignment.
- Want to assign a statically-named value? You can use `object.key = value`
- Want to assign a dynamically-named value? You can use `object[key] = value`
- Want to assign a statically-named member function? You can use `function object:func()`
- Want to assign a dynamically-named member function? You can't do that.

Member function, in this sense, meaning "a function which has an implicit `self` argument".

## Design

The design simply merges the member function assignment pattern with the dynamic assignment pattern.
```lua
function object:[func]()
   -- function body
end
```

In this example, `object` is any table and `func` is any variable. An example use case would be an object where several members functions have an identical (or at least easily-adaptable) body:

```lua
for _, name in {'open', 'close', 'toggle'} do
    function object:[name]()
        error(name.." is not implemented")
    end
end
```

In this example, the object expects that those functions would be implemented later on in some unspecified dependant code.

## Drawbacks

It's potentially confusing to read. `object:[name]()` is not very friendly to people who don't already know all the details of the other assignment methods.

Additionally, it may be non-trivial to determine what member functions an object has (or can have) if they can be dynamically assigned.

## Alternatives

One alternative is to use a function pattern, for example `table.assignMember(object, index, func)`. This was not selected because there aren't any other examples of direct assignment through functions. `insert` is probably the closest, but it has additional behavior that necessitates its implementation as a function. Using a function for this is also non-obvious, as those familiar with the language would likely expect to dynamically assign a member function using a recognizable combination of existing assignment patterns.

Another alternative is a new assignment pattern, for example `function object:{key}()`. This wasn't selected because an additional pattern seemed unnecessary and non-obvious.