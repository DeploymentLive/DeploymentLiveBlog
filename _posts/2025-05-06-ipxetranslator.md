---
layout: post
title: "iPXE Pre-Compiler"
categories: ipxe 
---

# iPXE Pre-Compiler

Been working with [iPXE](https://ipxe.org) a lot over the past year. iPXE has it's own scripting language that can perform a wide range of built in intrinsic commands, like downloading files, Interface functions, and other basic test and control primitives.

When running the script, the script interpreter can branch by  testing the return codes of basic commands, and loop using goto statements.

Example:
```
:startinternet
ping 8.8.8.8 && goto foundinternet || goto retryinternet
:retryinternet
echo did not find internet
goto startinternet
:foundinternet
echo Found Internet
```

Well, after using higher level languages for the past couple of decades, it has been a struggle going back to these primitives. I hate goto's, and yes I consider them harmful as well ( Shout out to Dijkstra: https://homepages.cwi.nl/~storm/teaching/reader/Dijkstra68.pdf )

## Pre-Compiler

So I had the idea of writing a pre-compiler to translate more complex routines into iPXE native code, taking high level concepts and translating on the fly.

For example, how could we add be the ping statement above, how could we add an if statement to the command above?

```
:startinternet
if ( ping 8.8.8.8 )
    echo found internet
else
    echo did not find internet
    goto: startinternet
end if
```

with some fancy regex expressions, we can write some code to search for the `if` statement, extract out the command `ping 8.8.8.8` and output the translated command to the output. 

Additionally, we could also add a `NOT` and a `while` statements to make this look even cleaner:

```
while ( NOT ping 8.8.8.8 )
    echo did not find internet
wend
echo found internet
```

## functions

Another feature I wanted to see handled is the concept of a function/subroutine. 

```
sub mycustomfunction
    echo hello ${arg1}
end sub

call mycustomfunction world
```

Calling a function is similar to calling goto, but when we are done in the function, where do we go back to? iPXE doesn't have a call stack where we can return to. So the calling function will need to keep track of where to return.

The above code would look like this:
```
:mycustomfunction
echo hello ${arg1}
goto :mycustomfunction_returnto

set mycustomfunction_returnto call_0001_returnpoint
set arg1 World
goto mycustomfunction
:call_0001_returnpoint
```

Just a bit more complex, but should be possible to create a simple script using the `sub` and `call` commands. 

## Other notes:

* Save your script with the .sh file extension. Visual Studio code will do a pretty good job of showing the iPXE code as a sh script. Then save the output as *.ipxe for execution after running through the pre-compiler.
* Lines with `#region` and `#endregion` are striped out.
* Lines that start with `##` are removed.

# REFERENCE:

Here is a list of the commands the pre-compiler will translate:

## IF 
Standard IF statement against a single iPXE command and test the output.
### Syntax
```
if ( [not] <ipxecommand> [args...] ) # Comments...
    echo true if
else   # Comments...
    echo false if
end if   # Comments...
```
Must include the closing `end if` statement, `else` is optional. `NOT` ( or `!` ) will test the negative case.
There can be any number of arguments passed to the test command. 

## WHILE 
Standard While loop statement against a single iPXE command and test the output.
### Syntax
```
while ( [not] <ipxecommand> [args...] )  # Comments...
    echo Looping...
    if ( ping google.com )
         break # Comments...
    end if
    if ( ping 8.8.8.8 )
        continue # Comments...
    end if
end while  # Comments...
```
Must include the closing `end while` statement, `NOT` ( or `!` ) will test the negative case.
There can be any number of arguments passed to the test command. 
`break` will exit the while loop. `continue` will jump back to the start of the while loop.

## INCLUDE 
Include another iPXE script in this file.
### Syntax
```
#include <path> # Comments...
```
The Path can be relative to the existing script, the current directory, or the path must be absolute. 
The sub-script is pre-parsed before importing.

## SUB 
Subroutine Commands
### Syntax
```
sub <subroutinename>  # Comments...
    return # Comments...
end sub # Comments...
call <subroutinename> [args..] # Comments...
```
Calls a subroutine, really just a goto jump, where the subroutine keeps track of where to jump BACK to. 
call will accept a number of arguments, stored as global variables arg1, arg2, arg3 respectively. 

forexample:
```
call  mysubroutine One Two "Three Tres Drei"
```
turns into:
```
set arg1 One
set arg2 Two
set arg3 "Three Dres Drei"
```

## echo 
console formatting
### Syntax
```
echo  This     Line     will     NOT    show    extra                    spaces
echo "This     Line     will            SHOW    extra                    spaces"
```
For text written to console, iPXE will trim all consecutive whitespace to a single space.
To allow for text alignment, we can add an empty escape:  From: `"  "` to `" ${} "`
This feature is only applicable for `echo` `item` `prompt` lines contain Double Quotes.

# Use

The script can be found at our github repro: https://github.com/DeploymentLive/iPXEPreProcessor 