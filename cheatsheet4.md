## Random notes

- `=~` validates against a regex, `==` validates for an exact match


## Behaviors vs Protocols

**Behaviours** and **protocols** are similar in that they both define a set of functions that must be implemented. However, the main difference between them is that **behaviours** are tied to a module, while **protocols** are tied to a data type 1. Behaviours are used to ensure that a module implements a certain set of functions, while protocols are used to define a set of functions that a data type must implement 2.

## Behaviours

```elixir
defmodule Parser do
  @callback parse(String.t) :: any
  @callback extensions() :: [String.t]
end
```
---
```elixir
defmodule JSONParser do
  @behaviour Parser

  def parse(str), do: # ... parse JSON
  def extensions, do: ["json"]
end
```


## Protocols 

```elixir
defprotocol Blank do
  @doc "Returns true if data is considered blank/empty"
  def blank?(data)
end
```
---
```elixir
defimpl Blank, for: List do
  def blank?([]), do: true
  def blank?(_), do: false
end

Blank.blank?([])  # → true
```

## Any
```elixir
defimpl Blank, for: Any do ... end

defmodule User do
  @derive Blank     # Falls back to Any
  defstruct name: ""
end
```

## Sigils

```elixir
~r/regexp/
~w(list of strings)
~s|strings with #{interpolation} and \x20 escape codes|
~S|no interpolation and no escapes|
~c(charlist)

# Allowed chars: / | " ' ( [ { < """
```

## Type specs and custom types

```elixir
@spec round(number) :: integer

@type number_with_remark :: {number, String.t}
@spec add(number, number) :: number_with_remark
```

## Regexp

```elixir
exp = ~r/hello/
exp = ~r/hello/i
"hello world" =~ exp
```

## Metaprogramming
```elixir
__MODULE__
__MODULE__.__info__

@after_compile __MODULE__
def __before_compile__(env)
def __after_compile__(env, _bytecode)
def __using__(opts)    # invoked on `use`

@on_definition {__MODULE__, :on_def}
def on_def(_env, kind, name, args, guards, body)

@on_load :load_check
def load_check
```