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
  
### How to define Lambda function and use it
```elixir
square = fn x ->
  x * x
end

iex(2)> square.(24)
square.(24)
576
```
- The dot operator is to make the code explicit such that you know an anonymous function is being called.
  
- **Capture** makes us to make full function qualifier as lambda
```elixir
Enum.each([1, 2, 3, 4], &IO.puts/1)

iex(4)> Enum.each([1, 2, 3, 4], &IO.puts/1)
1
2
3
4
:ok
```

The closure capture doesn’t affect the previous defined lambda that references the same symbolic name
```elixir
outside_var = 5
lambda = fn -> IO.puts(outside_var) end
outside_var = 6
lambda.() #=> 5
```

### other types

- Time and date
```elixir
date = ~D[2008-09-30]
time = ~T[11:59:12]
naive_datetime = ~N[2018-01-31 11:59:12.000007]
```

- Appeding IO list is 0(1), very usefull to increamentaly builda stream of bytes
```elixir
iolist = []
iolist = [iolist, "This"]
iolist = [iolist, "is"]
iolist = [iolist, "Amazing"]

iex(20)> iolist = []
iex(21)> [[], "This"]
iex(22)> [[[], "This"], "is"]
iex(23)> [[[[], "This"], "is"], "Amazing"]
iex(24)> IO.puts(iolist)
IO.puts(iolist)
ThisisAmazing
:ok
```

#### How to check the and load additional code paths
- load additional code path from command-line when started erlang runtime
```elixir 
$ iex -pa my/code/path -pa another/code/path # from command-line to load additional code path 
```

- once start runtime, check current loaded path
```elixir
:code.get_path # check path 
```

### How to dynamically call a function
```elixir
apply(IO, :puts, ["Dynamic function call."])
```
###  How to run a single script
- Create `.exs` file
```elixir
defmodule MyModule do
  def run  do
    IO.puts("Called Mymodule.run")
  end
end

# Code outside of a module is executed immediately
MyModule.run
```
- On terminal
```elixir
elixir script.exs
``` 
- With `--no-halt`, it will make the BEAM instance keep running. Useful when your script start other concurrent tasks.

###  How to get current time

```elixir
iex(28)> {_, time} = :calendar.local_time()
{{2022, 2, 11}, {13, 32, 10}}
iex(29)> time 
time 
{13, 32, 10}
```

###  How to handle exception error in guard
- If an error is raised from inside the guard, it won’t be propagated. And the guard expression will return false. The corresponding clause won’t match.

### How to match the content of variable, ^ (pin) operator
```elixir
iex(30)> expected_name = "bob"
expected_name = "bob"
"bob"
iex(31)> {^expected_name, age} = {"bob", 25}
{^expected_name, age} = {"bob", 25}
{"bob", 25}
iex(32)> age 
age 
25
```

### How to check the type of a variable

```elixir
#from REPL/IEX
iex(10)> i x
i x
Term
  1
Data type
  Integer
Reference modules
  Integer
Implemented protocols
  IEx.Info, Inspect, List.Chars, String.Chars

#from code

defmodule Util do
    def typeof(a) do
        cond do
            is_float(a)    -> "float"
            is_number(a)   -> "number"
            is_atom(a)     -> "atom"
            is_boolean(a)  -> "boolean"
            is_binary(a)   -> "binary"
            is_function(a) -> "function"
            is_list(a)     -> "list"
            is_tuple(a)    -> "tuple"
            true           -> "idunno"
        end    
    end
end
```


###  How to chain multiple pattern matching using `with`

```elixir
defmodule ChainPattern do
  # define some helper function
  def extract_login(%{"login" => login}) do
    {:ok, login}
  end
  def extract_login(_) do
    {:error, "login missed"}
  end

  def extract_email(%{"email" => email}) do
    {:ok, email}
  end
  def extract_email(_) do
    {:error, "email missed"}
  end

  def extract_password(%{"password" => password}) do
    {:ok, password}
  end
  def extract_password(_) do
    {:error, "password missed"}
  end


  def extract_info(submitted) do
    with {:ok, login} <-extract_login(submitted),
      {:ok, email} <-extract_email(submitted),
      {:ok, password} <-extract_password(submitted) do
      {:ok, %{login: login, email: email, password: password}}
    end
  end
end

submitted = %{
  "login" => "alice",
  "email" => "some_email",
  "password" => "password",
  "other_field" => "some_value",
  "yet_another_not_wanted_field" => "..."
}

# iex(20)> ChainPattern.extract_info(submitted)
# ChainPattern.extract_info(submitted)
# {:ok, %{email: "some_email", login: "alice", password: "password"}}
```

### How to update hierachical data

- In general
  - We can’t directly modify part of it that resides deep in its tree.
  - We have to walk down the tree to particular part that needs to be modified, and then transform it and all of its ancestors.
  - The result is a copy of the entire model.
- Useful macros from Kernel:
  - put_in/2
  - put_in/3
  - get_in/2
  - update_in/2
  - get_and_update_in/2
- Those macros rely on the **Access** module. **i.e**: `ie`

### How to register a process/ name a process

```elixir
Process.register(self(), :some_name)

send(:some_name, :msg)
receive do
  msg -> IO.puts("received #{msg}")
end
```

### How to handle unlimited process mailbox problem

- If a message is not match, it will be stored in mailbox with unlimited number. If we don’t process them, they will **slow down the system** and even **crash** the system when all memory is consumed.

### How to implement a general server process

- In general, there are 5 things to do:
  - spawn a seperate process
  - loop to infinite in that process
  - receive message
  - receive message
  - maintain state

#### How to debug using  `inspect`

Check the representation of a struct

```elixir
Fraction.new(1,4)
|> IO.inspect() 
|> Fraction.add(Fraction.new(1,4))
|> IO.inspect()
|> Fraction.value()

# %Fraction{a: 1, b: 4}
# iex(70)> %Fraction{a: 1, b: 4}
# %Fraction{a: 1, b: 4}
# iex(71)> %Fraction{a: 8, b: 16}
# iex(72)> %Fraction{a: 8, b: 16}
# %Fraction{a: 8, b: 16}
# iex(73)> 0.5
```

### How to get the number of currently running process
```elixir
:erlang.system_info(:process_count)
```

### How state is maintained in server process

- In plain server process implementation
State is passed as argument in loop clause. 
- State is modified (new state) as the result of callback module’s message handling.
- This means the callback module’s `handle_call/2` and `handle_cast/2` need to pass state as argument

- In **GenServer**
  - State is passed in from callback module's interface as argument.
  - state is passed in in `handle_cast/2` as argument
  
###  How to create a singleton of a module
- Implement `GenServer` in your module.
```elixir
defmodule Mod do
  def start do
    # locally register the process, make sure only one instance of the database process.
    GenServer.start(__MODULE__, nil, name: __MODULE__)
  end
end
```


### How to use elixir to request access token using `Poison`

```elixir
defmodule Script do
  @secret "84G7Q~JiELHPu3XuNKqckEB1eavVnMpHmnoZh"
  @client_id "2470ca86-3843-4aa2-95b8-97d3a912ff69"
  @tenant "72f988bf-86f1-41af-91ab-2d7cd011db47"
  @scope "https://microsoft.onmicrosoft.com/3b4ae08b-9919-4749-bb5b-7ed4ef15964d/.default"  
  @moduledoc """
  A HTTP client for doing RESTful action for DeploymentService.
  """
  def request_access_token() do
    url = "https://login.microsoftonline.com/#{@tenant}/oauth2/v2.0/token"

    case HTTPoison.post(url, urlencoded_body(), header()) do
      {:ok, %HTTPoison.Response{status_code: 200, body: body}}  ->

        body
        |> Poison.decode
        |> fetch_access_token
        # |> IO.puts

      {:ok, %HTTPoison.Response{status_code: 404}} ->
        IO.puts "Not found :("
      {:error, %HTTPoison.Error{reason: reason}} ->
        IO.inspect reason      
    end
  end

  def trigger_workflow(token) do
    definition_name = "AuroraK8sDynamicCsi"
    url = "https://xscndeploymentservice.westus2.cloudapp.azure.com/api/Workflow?definitionName=#{definition_name}"
    HTTPoison.post(
      url,
      json_body(),
      [
        {"Content-type", "application/json"},
        {"Authorization", "Bearer #{token}"},
        {"accept", "text/plain"}])
  end

  def test() do
    request_access_token()
    |> trigger_workflow
  end

  def fetch_access_token({:ok, %{"access_token" => access_token}}) do
    access_token
  end

  def header() do
    [{"Content-type", "application/x-www-form-urlencoded"}]
  end

  def urlencoded_body() do
    %{"client_id" => @client_id,
      "client_secret" => @secret,
      "scope" => @scope,
      "grant_type" => "client_credentials"}
    |> URI.encode_query
  end

  def json_body() do
    %{
      SubscriptionId: "33922553-c28a-4d50-ac93-a5c682692168",
      DeploymentLocation: "East US 2 EUAP",
      Counter: "1",
      AzureDiskStorageClassAsk: "Random",
      AzureDiskPvcSize: "13"
    }
    |> Poison.encode!
  end
end
```

### How to do OAuth
 - using Pow


### How to check a module’s available functions
- <MODULE_NAME>.__info__(:functions)
```elixir
KeyValueStore.__info__(:functions)
[get: 2, handle_call: 2, handle_cast: 2, init: 0, put: 3, start: 0]
```

### How to represent a grid
1. using Multidimensional Arrays
2. Elixir Forum
3. Own Implementation
   

### How to produce permutation and combination from list 
- using Comb 

### Difference between alias, use, require and import in Elixir

1. `alias` is used to give shortcut names for a model.
2. `import`: Aliases are great for shortening module names but what if we use functions from given module extensively and want to skip using module name part?
`import` imports all public functions and macros from given module.
3. `require` is like import + alias while different from either `import` or `alias`.
   - It is used like `alias`, but different from it that it will compile module first.
   - So, if a module contains a macro, and we want to use as SomeModule.SomeMacro, `require` will work but not `alias`.
4. `use` allow us to **inject** any code in the current module


[Note on Phoenix](image.png)

### Some notes

- Always keep in mind that a Boolean is just an atom that has a value of true or false.
- short-circuit operators: `||`, `&&`, `!`
  - `||` return the first expression that isn't falsy. **Eg**:
```elixir
read_cache || read_from_disk || read_from_database
```

[Link for More](https://zwpdbh.github.io/elixir-programming/elixir-cheatsheet.html)