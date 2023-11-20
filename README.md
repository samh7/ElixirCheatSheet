# Elixir Cheat Sheet
---

## Introduction

Elixir is a dynamic functional compiled language that runs over an Erlang Virtual Machine called BEAM.
<br>

Erlang and its BEAM is well known for running low-lattency, distributed and fault-tolerant applications
<br>

Elixir data types are immutable.
<br>

In Elixir a function is usually described with its arity (number of arguments), such as `is_boolean/1`
<br>

## File Types
---

- `.exs` => Elixir Script File
- `.ex` => Elixir Regular File
- `.beam` => Compiled Elixir File
<br>

## Compilating & Running Elixir Code

- `elixir <script_file>.exs` => run script code
- `elixirc <file>.ex` => compile a file into an `Elixir.<File>.beam`
<br>

## Elixr Convetions

- `function foo return tuple` => the result of a foo funtion is usually {:ok, result} ot , {:error, reason}.
- `function foo! may raise an error` =>  the result of foo! is not wrapped in a tuple and may raise an exception.
- **Exceptions/Erros** are not used to controlling flow.
- Elixir uses **fail** **fast** idea and the supervision trees to control process health and if possible, restart processes.
<br>

## Comments

- `#` => Single line comments
- No Multi-line comments
<br>

## Code Documentation
- `@moduledoc` => Module documentation
- `@doc` => funtion doc.
- `@spec` => function args/ return specification (types of the args and return of the function)
- `@typedoc` =>  type doc.
- `@type` => type definition.
- `@typep` => private type defintion

### Eg.
```elixir
defmodule Math d0
    @moduledoc """
    provides math-related functions.
    ## Examples
    iex> Math.sum(1,2)
    3
    """""
    @spec sum(number, number) :: number
    @doc"""
    Calculates the sum of two numbers.
    """
    def sum(a,b), do: a + b
end

```
<br>

## Elixir Special Variables

- `_` => unbound variable (discard)
- `^var` => It pins the variable on its value and prevent any assignment to this variable when using pattern matching.
    ### For Example
```elixir
    a = 1
    b = 2
    a = b # a now has a value of 2
    a == b # return true or false
    ^a = b = #pattern matches the two, if not equal, throws an error
** (SyntaxError) iex:13:10: unexpected expression after keyword list. Keyword lists must always come last in lists and maps. Therefore, this is not allowed:

    [some: :value, :another]
    %{some: :value, another => value}

Instead, reorder it to be the last entry:

    [:another, some: :value]
    %{another => value, some: :value}

Syntax error after: ','
    |
 13 | %{age: 30, :name => "Mary"}
    |          ^
    (iex 1.15.7) lib/iex/evaluator.ex:294: IEx.Evaluator.parse_eval_inspect/4
    (iex 1.15.7) lib/iex/evaluator.ex:187: IEx.Evaluator.loop/1
    (iex 1.15.7) lib/iex/evaluator.ex:32: IEx.Evaluator.init/5
    (stdlib 5.1.1) proc_lib.erl:241: :proc_lib.init_p_do_apply/3   
``` 
- `|` =>
   1. used to update a single key in a map or a truct
```elixir

#The a key of the same name should already exist otherwise it errors
# iex> map = %{name: Alice}
# iex> map %{map} is not valid syntax you use it with the `|` operator to update a key

map = %{name: "Alice", age: 25}
map = %{map | name: "Bob"}

struct = %User{name: "Alice", age: 25}
struct = %{struct | name: "Bob"}
```
  2. To prepend an element to a list
```elixir
    list = [2,3,4]
    list = [1 | list] # list now equals [1,2,3,4]
```
<br>

## Elixir/Erlang Virtual Machine inspection

- `:observer.start` => Start a gui too for inspection
- `:erlang.memory` => inspect memory
- `:c.memory` => inspect memory
```elixir
    :c.memory
    # [
    #   total: 19262624,
    #   processes: 4932168,
    #   processes_used: 4931184,
    #   system: 14330456,
    #   atom: 256337,
    #   atom_used: 235038,
    #   binary: 43592,
    #   code: 5691514,
    #   ets: 358016
    # ]
```
<br>

## Interactive Elxir/IEX

- `iex` => open Interactive Elixir
- `iex file` => open iex loading a file
- `<Ctrl>c + a` => close iex
- `i <object>` => information about an object
- `h <function/arity>` => help for a function
- `h <operator/arity>` => help for an operator
- `s <function/arity>` => specification for a function
- `s <operator/arity>` => specification for an operator
- `t <function/arity>` => type for a function
- `c <file>` => Load and compile a .ex file
<br>

# Basic Types
---
<br>

-  `1` => Integer
-  `1_000` => integer, use `_` to make it easy to read
-  `0x1F` => Hexadecimal integer
-  `0b1010` => Binary integer, base 10  
-  `0o777` => Octadecimal Integer
- `1.0` => Float
- `5.7e-2` => Floar exponent notation 0.057
- `:atom` => atom/symbol
- `true` => boolean (it is an atom)
- `<<97::size(2)>>` =>  bit string
- `<<97,89>>` => Binary
- `"elixir"` => string
- {1,2,3} => tuple
- [1,2,3] => list (actually a linked list)
- `'elixir'` => char list
- `[a: 5, b: 3]` => keyword list (short notation) `[a => 5, b => 3]` is not permited.
- `[{:a, 5},{:b, 3}]`   keyword list (long notation)
- `%{name: "Mary", age: 29}` => Map short notation (keys must be atoms).
- `%{:name => "Mary", :age => 29}` => Map long notation.
```elixir
    # Keyword list(the short notation) must always come last. So if both short and long notation are used in the same map, the short notation should come last
    %{:name => "Mary", age: 30} # Correct, does not error
    %{name: "Mary", age: 30} # Correct, Does not error
    %{age: 30, :name => "Mary"} # Errors
        #** (SyntaxError) iex:13:10: unexpected expression after keyword list. Keyword lists must always come #last in lists and maps. Therefore, this is not allowed:
        #[some: :value, :another]
        #%{some: :value, another => value}
        #Instead, reorder it to be the last entry:
```
- `self()` => Current Process id
- `fn -> :hello end` => Anonymous function
- `make_ref()` => Create a new reference
- `hd Port.list()` => get firt port
- `is_nil/1`
- `is_integer 1`
- `is_float 4.6`
- `is_number 7.8`
- `is_atom :foo`
- `is_boolean false`
- `is_bitstring <<97:2>>`
- `is_binary <<97, 98>>`
- `is_list/1`
- `is_tuple/1`
- `is_map/1`
- `is_pid self()`
- `is_function(fn a, b -> a + b end)` => function
- `is_function(fn a, b -> a + b end, 2)` => function with arity
- `is_port hd Port.list()`
- `is_reference make_ref()`
- `Range.range?(1..3)`
<br>

## Converting Types
<br>

- `to_charlist("hello")`
- `to_string('hello')`
- `Map.to_list(%{:a => 1, 2 => :b})` => map to list of tuples
<br>

## Number Operators
<br>

- `10/2 => 5.0` => always returns a float
- `div(10, 2) => 5` => Integer division
- `rem(10, 3) => 1` => Modulus operation
- `round(3.58) => 4` => float round
- `trunc(3.58) => 3` => float trunc
<br>

## Operators
`==`, `!=`, `===` => strict equal (integer with float), `!==` => strict different (inetger with float), `<`, `<=`, `>`, `>=`, `&&` => truthy and (t-and), `||` => t-or, `!`, `and` => boolean and (b-and), `or` => b-or, `not` => b-not

### <u>NOTE</u>
<br>

It's possible to compare different data types and that's the sorting order: `number < atom < reference < functions< port < pod < tuple < list < bit string` (narf, pptlb)
<br>

## Pipe Operator
<br>

-   `|>` => Pipe Operator. Passes a value as the first argument of the function following it
```elixir
    1.100
    |> Stream.map(&(&1 * 3)) 
    |> Stream.filter(&(rem(&1,2) != 0))
    |> Enum.sum
    # => 7500    
```
<br>

## Pattern Matching

`=` => is not just an assignment operator, but a Match Operator. It matches variables from the right side to the left based on patterns defined by the left one. If a pattern does not match a `MatchError` is raised.

```elixir
    [a, b, 10] = [0,1,10] #does not error instead, assigns values 0 and 1 to variables a and b respectively.
    
    %{a: 10, :b => 20 } => %{:a => 10, b: 20}
```
<br> 
But, for variables, is will most likely be an assignment operator:

```elixir
    a = 1
    b = 2
    a = b # a now is equal to 1
```
<br>

We therefore use a pin operator `^` to pattern match variables:

```elixir
    a = 1
    b = 10
    ^a = b #errors
```
<br>

```elixir
#To pattern match a single item from a struct or a map do:
map = %{name: "Alice", age: 25}
%{name: name} = map
```

```elixir
#empty maps/ structs always match for any map/struct respectively
map = %{name: "Alice", age: 25}
%{} = map # does not error
```
<br>

### So in other words:
- non variables on the left side will be used to restrict a pattern to match
- variables using the pin operator on the left side will have its value to be used to restrict a pattern to match
- variables on the left side will be assigned with right side values
<br>

## Match Operator Limitation
<br>

You cannot make function calls on the left side of a match.
- `length([1, [2], 3]) = 3 #=> ** (CompileError) illegal pattern`
<br>

## Custom Operators
---
<br>

You can customizr an Elixir Operator.
example:
```elixir
    1 + 2 # => 3
    defmodule WrongMath do
        def a + b do
            {a, b}
        end
    end
    import WrongMath
    import Kernel, except: [+: 2]
    1 + 2 #=> {1, 2}
```