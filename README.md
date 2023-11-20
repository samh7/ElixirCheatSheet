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

You can customize an Elixir Operator.
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

## Sigils
---

- `~r` => regular expression (modifires: `i`)
- `~r/hello/i` => `i` modifies to case insensitive
- `~w` => list of a string of words (eg. `~w[foo bar]`)
- `~w[foo bar]c` => `c` modifies list of char lists
- `~w[foo bar]a` => `c` modifies to list ot atoms
```elixir
~w(one two three)   #=> ["one", "two", "three"]
~w(one two threee)c #=> ['one', 'two', 'three']
~w(one two three)a  #=> [:one, :two, :three]
```
## Bit Strings
---

- `<<97::4>>` => short notation with 4 bits
- `<<97::size(4)>>` => long notation with 4 bits
- `byte_size(<<5::4>>)` => bit string byte size
  
## Binaries
---

Binaries are 8 bit multiple Bit Strings. Binaries are 8 bits by default.

- `<<97>>` => Shirt notation with 8 bits.
- `<<97::size(8)>>` => Long notation with 8 bits.
- `<>` => concatenate binaries/strings

## Strings
---

String is a binary of code points where all elements are valid characters.

Strings are sorrounded by double quotes and are encoded in `UTF-8` by default.

- `"hello"` => string
- `<<97, 98>>` => string "ab"
- `<<?a, ?b>>` => string "ab"
- `?a` => 97
- `?b` => 98
- `"hello #{:world}"` => string interpolation for "hello world"
- `String.length("hello") #=>5`
- `String.upcase("hello") #=> "HELLO"`
- `"""(begin) multi-line string (end)"""`

### Performance for Strings:

cheap functions => `byte_size("hello")`
expensive functions => `String.length("hello")`

## Tuples 
---

Tuple is a list that is stored contiguously in memory.
- `{:ok, "hello"}`
- `tuple_size({:ok, "hello"})` => tuple size (cheap function)
- `elem({:ok, "hello"}, 0)` => get tuple element by index (cheap function)
- `put_elem({:ok, "hello"}, 1, "world")` (expensive function)
  
## Lists
---

Lists implements Enumerables protocol.

List is a linked list structure where each element points to the next one in memory. When subtraction just the first ocurrence will be removed.

- `[:ok, "hello"]`
- `[97 | [1, 6, 9]]` => prepend (expensive function)
- `[1, 5] ++ [3, 9] # [1, 5, 3, 9]` => concatenation (expensive function)
- `[1, 2, 3, 4, 5, 6] -- [3,4,10,13] # [1,2,5,6]` => Keep all elements in the first list execpt for all the commont elements between the two lists and discard the rest in the second list.
- `hd([1, 5, 7]) #=> 1` => head (cheap function)
- `tl([1,5,7]) #=> [5,7]` =>  tails (cheap function)
- `:foo in [:foo, :bar] #=> true` => in operator (expensive function)
- `length([1,5])` => length of list (expensive function)

## Char List
---

A Char List is a List of code points where all elements are valid characters. Char Lists are surrounded by single-quote and are ***usually used as arguments to some old Erlang code***.

- `ab` => Char list
- `[97, 98]` => 'ab'
- `[?a,?b]` => 'ab'
- `?a` =? 97
-  `'hello' ++ 'world' #helloworld` => Concatenation
- `'hello' -- 'world' #hel` => Keep all elements in the first char list execpt for all the commont elements between the two lists and discard the rest in the second char list.
- `?l in 'hello' #true` => in operator
- `'length('hello')'`
- `[?H | 'ello']`

## Keyword Lists
---

Keyword list is a list of tuples where first elements are atoms. When fetching by key the first match will return. If a keyword list is the last argument of a function the square brackets [ are optional.

- `[{:a, 6} | [b: 7]] # [a: 6, b: 7]` => prepend
- `[a: 5] ++ [a: 7] # [a: 5, a: 7]`
- `([a: 5, a: 7])[:a] # 5` => fetch by key
- `length([a: 5, b: 7])`
  
  ### <u>Note<u>

- `[{:a, 6} | [b: 7]]` => `[a: 6, b: 7]`
- `[[a: 6] | [b: 7]]` => `[[a: 6], {:b, 7}]` 

## Maps
---

Maps implements Enumerable Protocol.
Map holds a key value structure.
- `%{name: "Mary", age: 29}[:name] #=> "Mary`
- `%{name: "Mary", age: 29}[:born] #=> nil`
- `%{name: "Mary", age: 29}.name #=> "Mary`
- `%{name: "Mary", age: 29}.name #=> ** (KeyError)`
- `map_size(%{name: "Mary"}) #=> 1`
  
## Structs
---
Structs are built on top of maps
`defstruct` => define a struct

```elixir
defmodule User do
  defstruct name: "John", age: 27
end
john = %User{} #=> %User{age: 27, name: "John"}
mary = %User{name: "Mary", age: 25} #=> %User{age: 25, name: "Mary"}
meg = %{john | name: "Meg"} #=> %User{age: 27, name: "Meg"}
bill = Map.merge(john, %User{name: "Bill", age: 23}) #=> %User{name: "Bill", age: 23}

john.name #=> John
john[:name] #=> ** (UndefinedFunctionError) undefined function: User.fetch/2
is_map john #=> true
john.__struct__ #=> User
Map.keys(john) #=> [:__struct__, :age, :name]
```

## Ranges
---

Ranges are `struct`
- `range = 1..10`
- `Enum.reduce(1..3, 0, fn i, acc -> i + acc end) #=> 6`
- `Enum.count(range) #=> 10`
- `Enum.member?(range, 11) #=> false`

## Protocols
---

- `defprotocol Foo do ... end` => define protocol `Foo`
- `defimpl Blank, for: Integer do ... end` => implement that protocol `Integer`
- Here are all native data types that you can use: `Atom`, `BitString`, `Float`, `Function`, `Integer`, `List`, `Map`, `PID`, `Port`, `Reference`, `Tuple`.
```elixir
defprotocol Blank do
  @doc "Returns true if data is considered blank/empty"
  def blank?(data)
end

defimpl Blank, for: Integer do
  def blank?(_), do: false
end

defimpl Blank, for: List do
  def blank?([]), do: true
  def blank?(_),  do: false
end

defimpl Blank, for: Map do
  def blank?(map), do: map_size(map) == 0
end


# Structs do not share Protocol implementations with Map.
defimpl Blank, for: User do
  def blank?(_), do: false
end

defimpl Blank, for: Atom do
  def blank?(false), do: true
  def blank?(nil),   do: true
  def blank?(_),     do: false
end

Blank.blank?(0) #=> false
Blank.blank?([]) #=> true
Blank.blank?([1, 2, 3]) #=> false
Blank.blank?("hello") #=> ** (Protocol.UndefinedError)
```
```elixir
# You can also implement a Protocol for Any. And in this case you can derive any Struct

defimpl Blank, for: Any do
  def blank?(_), do: false
end

defmodule DeriveUser do
  @derive Blank
  defstruct name: "john", age: 27
end
```

Elixir built-in most common used protocols: `Enumerable`, `String.Chars`, `Inspect`.

## Nested data Structures
- `put_in/2` => Puts a value in a nested structure via the given path.
- `update_in/2` => Updates a nested structure via the given path.
- `get_and_update_in/2` => Gets a value and updates a nested data structure via the given
path.
```elixir
users = [
  john: %{name: "John", age: 27, languages: ["Erlang", "Ruby", "Elixir"]},
  mary: %{name: "Mary", age: 29, languages: ["Elixir", "F#", "Clojure"]}
]
users[:john].age #=> 27

users = put_in users[:john].age, 31
users = update_in users[:mary].languages, &List.delete(&1, "Clojure")
```

## Enums and Streams
---

Lists and Maps are Enumerables
`Enum` module perform **eager** operations, meanwhile `Stream` module perform **lazy** operations.   
```elixir
# eager Enum
1..100 |> Enum.map(&(&1 * 3)) |> Enum.sum #=> 15150

# lazy Stream
1..100 |> Stream.map(&(&1 * 3)) |> Enum.sum #=> 15150
```

## do/end Keyword List and Block Syntax
---

In Elixir **Keyword List** syntax or do/end **Block** syntax:
```elixir
sky = :gray

if sky == :blue do
  :sunny
else
  :cloudy
end

if sky == :blue, do: :sunny, else: :cloudy

if sky == :blue, do: (
  :sunny
), else: (
  :cloudy
)
```

## Conditional Flows (if/else/case/cond)

### if/else

```elixir
sky = :gray
if sky == :blue, do: :sunny, else: :cloudy
```

### unless / else

```elixir
sky = :gray
unless sky != :blue, do: :sunny, else: :cloudy
```

### case / when

```elixir
sky = {:gray, 75}
case sky, do: (
  {:blue, _}         -> :sunny
  {_, t} when t > 80 -> :hot
  _                  -> :check_wheather_channel
)
```

On **when guards** short-circuiting operators &&, || and ! are not allowed.

### cond

`cond` is equivalent as `if/else-if/else` statements.

```elixir 
sky = :gray
cond do: (
  sky == :blue -> :sunny
  true         -> :cloudy
)
```

## The `with` macro
---

- `with` => macro to combine multiple match clauses
- `<-` => a matching clause, on the left
- `=` => bare expression is allowed
- `else` => if some matching clause fails

<!-- <- vs =  vs <=-->

```elixir
opts = %{width: 10, height: 20}
with {:ok, width} <- Map.fetch(opts, :width),
     {:ok, height} <- Map.fetch(opts, :height) do
  {:ok, width * height}
else
  :error ->
    {:error, :wrong_data}
end

opts = %{width: 10, height: 20}
with {:ok, width} <- Map.fetch(opts, :width),
     {:ok, height} <- Map.fetch(opts, :height) do
  {:ok, width * height}
else
  :error ->
    {:error, :wrong_data}
end

#=> {:ok, 200}
```

## Recursion
---

There is traditional no for loop in Elixir, due to Elixir immutability There is a macro `for` that it's also called as `Comprehension` but it works differently from a traditional for loop. If you want a simple loop iteration you'll need to use `recursion`:
```elixir
defmodule Logger do
  def log(msg, n) when n <= 0, do: ()
  def log(msg, n) do
    IO.puts msg
    log(msg, n - 1)
  end
end
Logger.log("Hello World!", 3)
# Hello World!
# Hello World!
# Hello World!
```

In functional programming languages map and reduce are two major algorithm concepts. They can be implemented with recursion or using the `Enum` module.

`reduce` will reduce the array into a single element.

```elixir 
defmodule Math do
  def sum_list(list, sum \\ 0)
  def sum_list([], sum), do: sum
  def sum_list([head | tail], sum) do
    sum_list(tail, head + sum)
  end
end
Math.sum_list([1, 2, 3]) #=> 6

Enum.reduce([1, 2, 3], 0, &+/2) #=> 6
```

map modifies an existing array (new array with new modified values):

```elixir
defmodule Math do
  def double([]), do: []
  def double([head | tail]) do
    [head * 2 | double(tail)]
  end
end
Math.double([1, 2, 3]) #=> [2, 4, 6]

Enum.map([1, 2, 3], &(&1 * 2)) #=> [2, 4, 6]
```

## Comprehension -> the for loop

`Comprehension`  is a syntax sugar for the very powerful `for special form`. You can have **generators** and **filters** in that.
- `for` => `comprehension`
- `->` => `generators`
- `:into` => Change return to another `Collectable` type.
  
You can iterate over `Enumrable` whats makes so close to a regular `for` loop on other languages:
```elixir
for n <- [1, 2, 3, 4], do: n * n
#=> [1, 4, 9, 16]
```

You can also iterate over multiple `Enumerable` and then create a combination between them:

```elixir
for i <- [:a, :b, :c], j <- [1, 2], do:  {i, j}
#=> [a: 1, a: 2, b: 1, b: 2, c: 1, c: 2]
```

You can pattern match each element

```elixir
values = [good: 1, good: 2, bad: 3, good: 4]
for {:good, n} <- values, do: n * n
#=> [1, 4, 16]
```

Generators use `->` and they have on the right an `Enumerable`  and on the left a pattern matchable element variable.

You can have **filters** to filter **truthy elements**:
```elixir
for dir  <- [".", "/"],
    file <- File.ls!(dir),
    path = Path.join(dir, file),
    File.regular?(path) do
  "dir=#{dir}, file=#{file}, path=#{path}"
end
#=> ["dir=., file=README.md, path=./README.md", "dir=/, file=.DS_Store, path=/.DS_Store"]
```        

You can `:into` to change the return type:

```elixir
for k <- [:foo, :bar], v <- 1..5, into: %{}, do: {k, v}
#=> %{bar: 5, foo: 5}
for k <- [:foo, :bar], v <- 1..5, into: [], do: {k, v}
#=> [foo: 1, foo: 2, foo: 3, foo: 4, foo: 5, bar: 1, bar: 2, bar: 3, bar: 4, bar: 5]
```

## Anonymous Functions
- `fn` => define functions
- `->` => one line function definition
- `.` => call a function
- `when` => guards

```elixir
add = fn a, b -> a + b end
add.(4, 5) #=> 9
```

We can have multiple clauses and guards inside functions.

```elixir
calc = fn
  x, y when x > 0 -> x + y
  x, y -> x * y
end
calc.(-1, 6) #=> -6
calc.(4, 5) #=> 9
```

## Modules And Named Functions
- `defmodule`
- `def` => functions (inside Modules)
- `defp` => private functions (")
- `when` => guard
- `@` => module attribute
- `__info__(:functions)` => list functions inside a module
- `defdelagate` => delegate functions
```elixir
defmodule Math do
  def sum(a, b) when is_integer(a) and is_integer(b), do: a + b
end

Math.sum(1, 2) #=> 3

Math.__info__(:functions) #=> [sum: 2]
```

Module attribute works as constants, evaluated at compilation time:

```elixir
defmodule Math do
  @foo :bar
  def print, do: @foo
end

Math.print #=> :bar
```

Special Module attributes:

`@nsv`, `@moduledoc`,`@doc`,`@behaviour`, `@before_compile`

## Default Argument
```elixir
defmodule Concat do
  def join(a, b, sep \\ " ") do
    a <> sep <> b
  end
end

IO.puts Concat.join("Hello", "world")      #=> Hello world
IO.puts Concat.join("Hello", "world", "_") #=> Hello_world
```

Default values are **evaluated runtime**.

So this is correct syntax:

```elixir
defmodule DefaultTest do
  def dowork(x \\ IO.puts "hello") do
    x
  end
end
DefaultTest.dowork #=> :ok
# hello
DefaultTest.dowork 123 #=> 123
DefaultTest.dowork #=> :ok
# hello
```

Function with multiple clauses and a default value requires a function head where default values are set:

```elixir
defmodule Concat do
  def join(a, b \\ nil, sep \\ " ") # head function

  def join(a, b, _sep) when is_nil(b) do
    a
  end

  def join(a, b, sep) do
    a <> sep <> b
  end
end

IO.puts Concat.join("Hello")               #=> Hello
IO.puts Concat.join("Hello", "world")      #=> Hello world
IO.puts Concat.join("Hello", "world", "_") #=> Hello_world
```

## Function Capturing
- `&` => Function capturing
- `&1` => first argument
- `&2` => Secong argument and so on.

`&` could be used with a module function.

When capturing you can use the function/operator with its arity.

```elixir
&(&1 * &2).(3, 4) #=> 12
(&*/2).(3, 4) #=> 12

(&Kernel.is_atom(&1)).(:foo) #=> true
(&Kernel.is_atom/1).(:foo) #=> true
(&{:ok, [&1]}).(:foo) #=> {:ok, [:foo, :bar]}
(&[&1, &2]).(:foo, :bar) #=> [:foo, :bar]
(&[&1 | [&2]]).(:foo, :bar) #=> [:foo, :bar]
```


## Behaviours
---

- `@callbacks` => defines a function to be implemented by other modules
- `::` => Separates the function defintion to its return type

```elixir
defmodule Parser do
  @callback parse(String.t) :: any
  @callback extensions() :: [String.t]
end

defmodule JSONParser do
  @behaviour Parser

  def parse(str), do: # ... parse JSON
  def extensions, do: ["json"]
end
```

## Exceptions/Errors => raise/try/rescue
---

Exceptions/Errors in Elixir are `structs`
- `raise "oops" #=> ** (RuntimeError) oops
- `raise ArgumentError #=> ** (ArgumentError) argument error`
- `raise ArgumentError, message: "oops" #=> ** (ArgumentError) oops`
- `defexception` => define an exception
- `try/rescue` => catches an error
- `throw/try/catch` =>  can be used as circuit breaking, but should be avoided
- `exit("my reason")` => exiting current process
- `after` =>  ensures some resource is cleaned up even if an exception was raised

```elixir
defmodule MyError do
  defexception message: "default message"
end

is_map %MyError{} #=> true
Map.keys %MyError{} #=> [:__exception__, :__struct__, :message]

raise MyError #=> ** (MyError) default message
raise MyError, message: "custom message" #=> ** (MyError) custom message
```

You can rescue an error with:
```elixir
defmodule MyError do
  defexception message: "default message"
end

is_map %MyError{} #=> true
Map.keys %MyError{} #=> [:__exception__, :__struct__, :message]

raise MyError #=> ** (MyError) default message
raise MyError, message: "custom message" #=> ** (MyError) custom message
```

`throw/catch` sometime is used for circuit breaking, but you can usually use another better way:

```elixir
try do
  Enum.each -50..50, fn(x) ->
    if rem(x, 13) == 0, do: throw(x)
  end
  "Got nothing"
catch
  x -> "Got #{x}"
end
#=> "Got -39"

Enum.find -50..50, &(rem(&1, 13) == 0)
#=> -39
```

`exit` can be caught but this is rare in Elixir:
```elixir
try do
  exit "I am exiting"
catch
  :exit, _ -> "not really"
end
#=> "not really"
```

You can ommit `try` inside functions and use `rescue`, `catch`, `after` directly:

```elixir
def without_even_trying do
  raise "oops"
after
  IO.puts "cleaning up!"
end
```

## IO
---

- `IO.puts/1 "Hello"` => prints to stdout
- `IO.puts :stderr, "Hello"` => print to stderr
- `IO.gets "yes/no: "` => reads an user input

##StringIO
---

- `{:ok, pid} = StringIO.open("my-file.md")` => open a file
-  `IO.read(pid, 2) #=> "he"` => read first 2 lines

## File
---

- `{:ok, file} = File.open "hello", [:write]` => open a file for writing.
- `IO.binwrite(file, "world")` => Writes into file
- `File.close(file)` => close file
- `File.read("my-file.md")` => read a file
- `File.stream!("my-file.md") |> Enum.take(10)` => read the first 10 lines
```elixir
{:ok, file} = File.open "my-file.md", [:write]
IO.binwrite file, "hello world"
File.close file
File.read "my-file.md" #=> {:ok, "hello world"}
```