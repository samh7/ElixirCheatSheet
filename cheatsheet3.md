## Mixing lists and tuples

How to convert two lists into a single list of tuples or vice versa?

```elixir
list1 = ["Hydrogen", "Helium", "Lithium"]
list2 = ["H", "He", "Li"]
list3 = [1, 2 ,3]

element_list = Enum.zip(list1, list2) # [{"Hydrogen", "H"}, {"Helium", "He"}, {"Lithium", "Li"}]
seperate_lists = Enum.unzip(element_list) # {["Hydrogen", "Helium", "Lithium"], ["H", "He", "Li"]}
```

## Struct (tagged map)

From maps to structs: Structs are extensions built on top of maps that provide compile-time checks and default values.

- It is only possible to define a struct per module, as the struct it tied to the module itself
- Its fields:
    - could be a keyword list
    - or, a list of atoms as in this example: in this case, the atoms in the list will be used as the struct’s field names and they will all default to `nil`.
    - About `@derive`:
       
       ### Deriving
       Although structs are maps, by default structs do not implement any of the protocols implemented for maps. For example, attempting to use a protocol with the User struct leads to an error:
```elixir
       john = %User{name: "John"}
       MyProtocol.call(john)
       ** (Protocol.UndefinedError) protocol MyProtocol not implemented for %User{...}
```

`defstruct/1`, however, allows protocol implementations to be derived. This can be done by defining a `@derive` attribute as a list before invoking `defstruct/1`:
```elixir
defmodule User do
  @derive MyProtocol
  defstruct name: nil, age: nil
end

MyProtocol.call(john) # it works!
```
A common example is to `@derive` the Inspect protocol to hide certain fields when the struct is printed:
```elixir
defmodule User do
  @derive {Inspect, only: :name}
  defstruct name: nil, age: nil
end
```
    -   Enforcing keys =>  Elixir also allows developers to enforce that certain keys must always be given when building the struct:
```elixir
defmodule User do
  @enforce_keys [:name]
  defstruct name: nil, age: 10 + 11
end
```

Now trying to build a struct without the name key will fail:

```elixir
%User{age: 21}
** (ArgumentError) the following keys must also be given when building struct User: [:name]
```

Keep in mind `@enforce_keys` is a simple compile-time guarantee to aid developers when building structs. It is not enforced on updates and it does not provide any sort of value-validation.
    - Types:

It is recommended to define types for structs. By convention, such a type is called t. To define a struct inside a type, the struct literal syntax is used:

```elixir
defmodule User do
  defstruct name: "John", age: 25
  @type t :: %__MODULE__{name: String.t(), age: non_neg_integer}
end
```
It is recommended to only use the struct syntax when defining the struct's type. When referring to another struct, it's better to use User.t() instead of %User{}.

### Protocols

1. A protocol is a module in which you declare functions without implementing them
2. Why we need protocol if we already could achieve polymorphism using patter matching? (Remember: polymorphism means you want behavior to vary depending on the data type.)
   1. Consider this example, we have a simple Utility module to tell use the types of input variable:
```elixir
defmodule Utility do
  def type(value) when is_binary(value), do: "string"
  def type(value) when is_integer(value), do: "integer"
  # ... other implementations ...
end
```

This only works well if we implement this code and this code is not shared by multiple apps. Because there would be no easy way to extend its features.

##### Protocol can help us:

- The protocol implementation doesn’t need to be part of any module. It means: you can implement a protocol for a type even if you can’t modify the type’s source code.
- Dispatching on a protocol is available to any data type that has implemented the protocol and a protocol can be implemented by anyone, at any time.
- So, rewrite those features as a protocol

```elixir
defprotocol Utility do
  @spec type(t) :: String.t()
  def type(value)
end

#  spread them over multiple files as needed
defimpl Utility, for: BitString do
  def type(_value), do: "string"
end

defimpl Utility, for: Integer do
  def type(_value), do: "integer"
end
```

Functions defined in a protocol may have more than one input, but the dispatching will always be based on the data type of the first input.

The power of Elixir’s extensibility comes when protocols and structs are used together.

You can `@derive` protocols

***It's possible to implement protocols for all Elixir data types***

#### Implementing `Any`

Manually implementing protocols for all types can quickly become repetitive and tedious. In such cases, Elixir provides two options: we can explicitly derive the protocol implementation for our types or automatically implement the protocol for all types. In both cases, we need to implement the protocol for `Any`.

#### Deriving

Elixir allows us to derive a protocol implementation based on the `Any` implementation. Let's first implement `Any` as follows:

```elixir
defimpl Size, for: Any do
  def size(_), do: 0
end
```

The implementation above is arguably not a reasonable one. For example, it makes no sense to say that the size of a `PID` or an `Integer`is 0.

However, should we be fine with the implementation for `Any`, in order to use such implementation we would need to tell our struct to explicitly derive the `Size` protocol:

```elixir
defmodule OtherUser do
  @derive [Size]
  defstruct [:name, :age]
end 
```

#### Fallback to `Any`

Another alternative to `@derive` is to explicitly tell the protocol to fallback to `Any` when an implementation cannot be found. This can be achieved by setting `@fallback_to_any` to `true` in the protocol definition:

```elixir
defprotocol Size do
  @fallback_to_any true 
  def size(data)
end
```

*** For the majority of protocols, raising an error when a protocol is not implemented is the proper behaviour.***


## How to process binary

# <u>Note<u>

- While **all binaries are bitstrings**, **not all bitstrings are binaries**.
- Strings are binaries
- Binaries have have values which are divisible by 8 **i.e** `bytes` **e.g** 8, 16, 32, 64, 128
- Bitstrings have  values which are not divisible by 8 **e.g** 11,23,43,13, 7

- A binary is a chunk of byte
- Create binary by enclosing the byte sequence
`<<1,2,3>>`
    - Each number represents  the value of the correspoding byte.
    - If the value is bigger that 255, it's truncated to the byte size:
```elixir
<<255>> #=> <<255>>
<<256>> #=> <<0>>
<<257>> #=> <<1>>
```
    - Specify the size of each value and tell the compiler how many bits to use for that particular value
```elixir
# note 1 byte  == 8 bits
# Bit is the smallest unit of the computer. It is usually represented with digits 0 and 1.
# Byte is made of 8 bits. It is used in determining the system storage. Byte is the most common term used in computing. It is used to represent 2^8 = 256 different values. 
<<234::16>> # => <<0, 234>>, used 2 bytes, the first has value 0, the second is 234 
<<1234::32>> # => <<0, 0, 4, 210>>
```
Byte values can be specified using decimal, octal (0o), or hexidecimal (0x) notation. Here's an example.

```elixir
iex> <<1, 0, 0o04, 0x12>>
<<1, 0, 4, 18>>
```

Binaries can be concatenated using the `<>` operator.

```elixir
iex> <<3, 12>> <> <<4, 8>>
<<3, 12, 4, 8>>
```
The `byte_size` function can be used to retrieve the number of bytes in a binary value
```elixir
iex> byte_size(<<1, 0, 4, 18>>)
4
iex> byte_size(<<1, 2, 128, 96, 255, 4>>)
6
```

Variables can also be used to specify byte values in a binary literal.

```elixir
iex> byte1 = 34
34
iex> byte2 = 154
154
iex> binary = <<byte1, byte2, 87>>
<<34, 154, 87>>

<<234::16>> # => <<0, 234>>, used 2 bytes, the first has value 0, the second is 234

<<1234::32>> # => <<0, 0, 4, 210>>


## Note: Word binary is interchangeably the summary below to mean either Elixir Binary or base 2 numbers

# this is so beacause the binary rep. of 1234 is 10011010010. Now since the max a binary can take is 8 bits(1 byte),  and the size given is 32 bits (4 bytes), we split the binary rep. to 4 8 bits i.e 00000000 00000000 00000100 11010010
#first section = 0, second = 0, third = 4 since 2 ^ 0 + 2 ^ 1 + 2 ^ 2 = 4 and the last part = 210 therefoe the binary is represented as: <<0, 0, 4, 210>>
```


###  How to view a string’s binary representation

```elixir
# A common trick in Elixir when you want to see the inner binary representation of a string is to concatenate the null byte <<0>> to it:
iex> "hełło" <> <<0>>
<<104, 101, 197, 130, 197, 130, 111, 0>>

# Alternatively, you can view a string’s binary representation by using IO.inspect/2:
iex> IO.inspect("hełło", binaries: :as_binaries)
<<104, 101, 197, 130, 197, 130, 111>>
```
## How to (pattern) match on a binary of unknown size
```elixir
iex> <<0, 1, x::binary>> = <<0, 1, 2, 3>>
<<0, 1, 2, 3>>
iex> x
<<2, 3>>
```
  - Matching on arbitrary length can only be done at end of the pattern and not anywhere else.
  - If you have the data which can be arbitrary bit length then you can add `bitstring` instead, so the pattern now looks like.

```elixir
<<header :: size(8), data :: bitstring>>
```

###  How to match n bytes in a binary
```elixir
iex> <<head::binary-size(2), rest::binary>> = <<0, 1, 2, 3>>
<<0, 1, 2, 3>>
iex> head
<<0, 1>>
iex> rest
<<2, 3>>
```

## How to pattern match on string with multibyte characters

```elixir
iex> <<x::utf8, rest::binary>> = "über"
"über"
iex> x == ?ü
true
iex> rest
"ber"
```

  - Therefore, when pattern matching on strings, it is important to use the utf8 modifier.
  
#### Example: chuck from png
- chunck format
```
  +--------------+----------------+-------------------+
  |  Length (32) | Chunk type (32)| Data (Length size)|
  +--------------+----------------+-------------------+
  |   CRC (32)   |
  +--------------+
```
- Pattern matching the chunk format

```<<length     :: size(32),
  chunk_type :: size(32),
  chunk_data :: binary - size(length),
  crc        :: size(32),
  chunks     :: binary>>
```
- Another way of defining n byte length is `binary - size(n)`
- Note: we matched `length` in pattern and used in the pattern as well. In Elixir pattern matching you can use the assigned variable in the pattern following it, thats why we are able to extract the `chunk_data` based on the `length`


###  How to represent string

- String in elixir is either a binary or a list type.
- String inter – evaluate values in string template
```elixir
"embedded expression: #{1 + 3}" #=>"embedded expression: 4"
```
- How to include quote inside string
  
```elixir
~s("embedded expression": #{1 + 3}) #=> "\"embedded expression\": 4"

""" 
embedded expression: "#{1 + 3}" 
"""
# => "embedded expression: \"4\"\n"
```

- Aother way to represent string is use single-quote
```elixir
'ABC'
[65, 66, 67] 
# => they both produce 'ABC'
```
- The runtime doesn’t distinguish between a list of integers and a character list.
- A **Char List** is a List of code points where all elements are valid characters. Char Lists are surrounded by single-quote and are ***usually used as arguments to some old Erlang code***.