[LuaPP](https://github.com/echaozh/luapp) is a library I wrote to ease the embedding of Lua scripts into C++ applications. It's more C++-ish, and much easier to use.

The library makes Lua tables, functions, and other primitive types directly accessible from the C++ code. Instead of manipulating the stack, you can directly get and set values to a global variable, or access fields of a table, or invoke a function as if it was native. You can pass in arguments of the std::string type, and it will be converted. You can even pass in Lua tables as arguments. Variadic arguments are used in the function interface to make invocations look and feel more native.

When you embed Lua into your application, you normally want to load external Lua scripts, execute them, and interact with them (say invoke functions defined in the scripts). LuaPP helps you interact with the scripts easily.

I wrote another library _taveren_ using LuaPP to embed Lua scripts into strings. It makes expansions in string literals like in Bash or Ruby possible.
