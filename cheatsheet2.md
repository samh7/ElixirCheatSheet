## String

Strings in elixir are series of Unicode Characters written between double quotes "".

#### Writing double-quotes inside of a string

you can put double-quotes inside of a string using backslash \

`"a string with \"double quotes\""`

#### String concatenation (adding strings)

`"Brooklin" <> " " <> "Myers" # "Brooklin Myers"`

#### String interpolation

```elixir 
name = "Brooklin"
"Hi #{name}" # Hi Brooklin
"What is 5 + 4? #{5 + 4}." # What is 5 + 4? 9.
```

#### <u>The String module</u>

You can access string-related functionality with the built-in String module.

```elixir
String.length "a string" # 8 
String.contains?("string", "str") # true
String.at("mystring", 2) # s
String.slice("mystring", 2, 3)
String.split("my string", " ") # ["my", "string"]
String.reverse("string") # gnirts
String.upcase("string") # STRING 
```

#### Integer Functions

Elixir provides functions rem and div for working with integer division and remainder.

```elixir
rem(5, 4) # 1
rem(10, 6) # 4
3 / 2 => #1.5
div(3, 2) # 1 (normally 1.5 with divide /)
div(1, 2) # 0 (normally 0.5 with divide /)
1 / 2 #0.5
div(8, 3) # 2 (normally 2.6666 with divide /)
8 / 3 # 2.6666
```

#### Comparison Operator

#### exactly equals ===
```elixir
5 === 5   # true
5 === 5.0 # false
4 === 5   # false
4 === "4" # false
```

#### equals ==

```elixir
5 == 5   # true
5 == 5.0 # true
4 == "4" # false, equals still checks type unlike some languages.
```

#### not exactly equals !==

```elixir
5 !== 5   # false
5 !== 5.0 # true
4 !== 5   # true
```

## Reusing Modules

Elixir provides directives for reusing modules in your project.

It’s important to note that Elixir is a compiled language. It is usually compiled using a tool called `mix`. So unlike a language like Javascript or Ruby, you do not import one file into another. Instead, modules are compiled together and can be accessed from any .ex files in the project.

#### Alias

alias allows you to import a module into another module and access it using a different name. This is commonly used to access nested modules more conveniently.

```elixir
defmodule Me do
  defmodule Info do
    def name do
      "Brooklin"
    end
  end
end

defmodule Greeting do
  alias Me.Info, as: Info
  # You can more conveniently write the above as:
  # alias Me.Info
  def say_hello do
   "Hello " <> Info.name()
  end
end

Greeting.say_hello() # Hello Brooklin

```

#### import

import is used to conveniently access functions from other modules.

For example, here is how you would access the puts/1 function from the IO module.

#### use

use allows you to inject functionality from one module into another.

Initially, you will most often use…use… 😅 with modules from third parties. For example, to write tests with the ExUnit framework, you will use the ExUnit.Case module

```elixir
defmodule AssertionTest do
  use ExUnit.Case, async: true

  test "always pass" do
    assert true
  end
```

use is a reasonably complicated topic, and it’s good enough if you understand it at a surface level early on.