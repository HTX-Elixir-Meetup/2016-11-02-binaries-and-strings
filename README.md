# Binaries and strings

## What are binaries? 
  Strings are UTF-8 Encoded binaries.
  
  Binaries are a sequence of bytes, denoted between the `<<` and `>>` angle brackets. 


  A Binary is a bitstring where the number of bits is divisible by 8.

  Binaries that are valid UTF - 8 will be represented as strings.

  Strings have a data structure and a printable representation.

   ```
  iex> <<71, 45, 71, 45, 71, 45, 71, 72, 79, 83, 84>>
  "G-G-G-GHOST"
  ```

  This example shows when you enter the codepoints into IEX, you receive a representation as a string. 

  You can also get a character's codepoint by using the `?` operator.

  ```
  iex> ?👻
  128123
  ```

  Elixir also supports a shorthand for referring to unicode characters:

  ```
  iex> "\u0068"
  "h"
  ```

## Graphemes vs. codepoints
  The data structure is composed of codepoints, which can combine to create individual graphemes.
  In unicode, some codepoints are used to modify other characters, like `é`.

  ```
  iex> string = "\u0065\u0301"
  "é"
  iex> String.codepoints string
  ["e", "́"]
  ```

  Keep in mind that in order to know a string's length (`String.length/1`, Elixir will have to traverse this sequence of bytes to return the number of printable graphemes from your data (and O(N) operation). You can also use `Kernel.byte_size/1` to know the size of a given binary, which is an O(1) operation.  

  ```
  iex> byte_size("é")
  2
  iex> String.length("é")
  1
  ```


 ### There be dragons 🐉

  Some characters can be represented as one codepoint or as a combination of multiple codepoints:
  
  ```
  iex> {combined, individual} = {"e\u0308", "\u00EB"}
  {"ë", "ë"}

  iex> combined == individual
  false

  iex> String.equivalent?(combined, individual)
  true
  ```


## Charlists
  Charlists are typically used for compatability reasons with Erlang. They are represented as single-quoted strings.

  ```
  iex> "Spooky scary skeletons" |> String.to_charlist
  'Spooky scary skeletons'
  ```

  One advantage/feature of this is that you can iterate over the sequence with functions you would use to work with lists.

  ```
  iex> spooky_charlist = 'Spooky scary skeletons'
  'Spooky scary skeletons'

  iex> hd spooky_charlist
  83

  iex> <<83>>
  "S"

  iex> spooky_string = "Spooky scary skeletons"
  "Spooky scary skeletons"

  iex> hd spooky_string
  ** (ArgumentError) argument error
      :erlang.hd("Spooky scary skeletons")

  iex> is_list spooky_string
  false

  iex> is_list spooky_charlist
  true
  ```

## Large binary space

  Large binaries (> 64 bytes) stored in separate area from a process' heap.
  
  This allows for fast message passing as only pointer sent between processes

  Can save a lot of memory as well (no needless copying)
  
  Can be long delay before being reclaimed by GC. The large binary has a counter of all of the other processes that are using it. In order to be garbage collected, all processes which have “seen” the binary must first do a garbage collection.
  
  If garbage collection does not happen often enough, the large binaries can grow and crash system.


  If your process is only referencing a tiny part of a large binary, it might be worthwhile to use `:binary.copy` to explicitly copy the portion you need from a large binary so that the large binary can be garbage collected sooner. [More info in erlang's docs](http://erlang.org/doc/man/binary.html#copy-1).

  See also Evan Miller's [Template of Doom](http://www.evanmiller.org/elixir-ram-and-the-template-of-doom.html) post.


## IO Lists

  Concatenating strings costs memory to compose items into a single string. Since our data is immutable, it can be cheaper and faster to work with lists of references that can easily be appended. 

  IO lists allow you to create sequences of binaries and other IO lists, which will ultimately be flattened when the list is written (to disk for example). This has advantages because that final string never requires memory within your application. It only exists in the destination. Additionally, IO lists work with lists have have layers of nesting. 

  ```
  iex> first = "David "
  "David "
  iex> mi = "S. "
  "S. "
  iex(41)> last = "Pumpkins"
  "Pumpkins"
  iex> IO.puts [[first, [mi]], last]
  David S. Pumpkins
  :ok
  ```


