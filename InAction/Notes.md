# Windows PowerShell In Action 3rd edition

## Core PowerShell Concepts

PowerShell projects ambition:

- Create a toolkit of simple pieces that can create complex solutions

PowerShell is:

- A command line shell
- A scripting environment
- An automation engine
- Optimal DevOps tool on Windows
- A tool for admins & developers

What is a shell?

- Enables access to the Operating Systems functions
- Historically sits over the OS kernel
- Kernel / Shell, pun
- A command-line interpreter
- Has a UI
- Focus on end user experience (alias, history, customizable)

What is a scripting language?

- Might have REPL (Read-evaluate-print-loop) making it feel like a shell
- Usually lacks UI
- Helps you develop larger scripts
- Runs with more efficiency then a shell language
- More oriented towards writing an application rather then typing a command

What is PowerShell?

- Both

Why did Microsoft create PowerShell?

- Increased complexity on Windows needed to be solved with a management tool that scales
- GUI was not the answer when Windows moved into the datacenter

PowerShell Patterns

- Apply cmdlet patters to solve problems
- General patters to different solutions
  - `| sort -dec propertyName | select -first 3` grabs largest 3 properties
  - `cmdlet | where {$_.b -like "*propertyName*"}`

PowerShell is based on: IEEE POSIX 1003.2 Standard

Flow-control statements

- if, else, elseif
- switch
- for, foreach
- while, do

Remote Administration

- Invoke-Command -ComputerName computer1,computer2 -scriptblock { stuff }
- Enter-PSSession -ComputerName computer1

Syntax style: C#

- Easy to convert PowerShell -> C#, for performance
- Easy to convert C# -> PowerShell when needed

Concepts and terminology

- `command -parameter1 -parameter2 arg1 arg2` command name, switch parameter, parameters with arguments, positional argument
- Parameter is the receiver of information
- Argument is the information itself
- In python: powershell parameters = keywords, powershell arguments = values

Type of "commands" in PowerShell:

- cmdlets (binary developed specific commands)
  - PowerShell specific
  - implemented by .net class `Cmdlet` in PowerShell SDK
  - dynamic link library (dll)
  - compiled code is loaded into the powershell process (faster then disk based commands)
  - Verb-Noun naming standard
  - Run example: Invoke-Cmdlet -Param Argument

- Functions
  - Named piece of PowerShell script code
  - Lives in memory as the interpreter is running
  - Discarded on exit
  - Parsed representation (is preserved when ran)
  - Run example: MyFunction -Param Argument

- Script Commands
  - A .ps1 text file with PowerShell code
  - Loaded and parsed every time they run (bit slower then a function)
  - Once loaded/parsed, the runtime speed is the same as a function
  - Run example: .\script.ps1 -Param Argument

- Native Windows Commands (ipconfig, dcdiag, net)
  - configurations (dsc)
  - Starts its own process
  - Parameter processing by itself (can missmatch PowerShell)

## Elastic type system

- To turn a very verbose language into a shell language
- Aliases (dir -> get-childitem)
- Transitional aliases (transition from cmd.exe/unix shells)
- Convenience aliases (Get-ChildItem -> gci, Invoke-Item -> ii, etc)
- Aliases does not contain parameters (if more advanced, use functions/scripts)
- The elastic type system lets you do:
  
```powershell

gcm|?{$_.parametersets.count -gt 3}|fl name

```

   and
  
  ```powershell
  Get-Command |
    Where-Object {$_.ParameterSets.Count -gt 3} |
        Format-List Name
  ```

- Type out something fast (interactively), or
- Write code that you/other people can read easily
- Parameter Alias (To combat versioning issues)

PowerShell Parser

- Parsing is: turn human readable code into computer understandable code
- Script text breaks into tokens by tokenizer (lexical analyzer)
- A token can be, symbol, number, keyword, variable
- Processed into structures in the language through syntactic analysis
- [Windows PowerShell Language Specification Version 3.0](https://www.microsoft.com/download/confirmation.aspx?id=36389)
- Single quoting turns a token with special meaning into a string `Write-Output '-InputObject'`
- Backquote character ie: 'Program` files'
- Escape character: $v = files
- Write-Output "`$v is $v" -> $v is files
- Doublequote: Variables are expanded
- Single/Double quotes can contain each other
- New like character: `n
- Escape/new-line processing is only supported in doublequotes

Parsing mode:

- Command mode
- Expression Mode

Parsing is working slightly different on each mode

Statement termination:

- ;
- New line (If previous text is syntactically complete, else it's treated as whitespace)

If backtick escape is the last character, the newline will be reated as a simple breaking space instead of a new line

Pipeline:

- PowerShell was the first language to pass objects through its pipeline, previous shells passed only strings
- Streaming behavior (prints asap, not waiting for the whole result set)
- Parser figures out what cmdlets and parameters to work with, pipeline processor

Pipeline processor:

- runs begin block on all commands used simontenously
- then runs the process block on the first cmdlet, and moves on with process on the 2nd cmdlet
- If objects are generated at any point, it gets passed down the pineline
- If the last cmdlet generates output, its passed back to the powershell host
- The host is then responsible for further processing

Parameters and Parameter binder

Binding steps:

1. Bind all named parameters
2. Bind all positional parameters
3. Bind from the pipeline by value with exact match
4. If not bound, then bind from the pipeline by value with conversion
5. If not bound, then bind from the pipeline by name with exact match
6. If not bound, then bind from the pipeline by name with conversion

Trace-Command can help with debugging parameter binding errors

```powershell
Trace-Command -Name ParameterBinding -Option All `
-Expression { 123 | Write-Output } -PSHost
```

( Description at page 38 )

Formatting:

- Formating database is stored in $PSHOME
- Format-* cmdlets help to display information
- Format-Wide, great for reading a large single property array
- Format-Custom, create specific formatting rules that are not table/list
- Out-Default prints to host
- Piping to Out-Null does the same as invoking $null but 40 times slower

## Working with (object) Types

- Type
  - Description of an object
- Class
  - Keyword for creating a type in PowerShell
- Property
  - piece of data describing the object
- Method
  - Define behaviors on a class
- Member
  - Generic term for Property and Method
- Event
  - Method that based on conditions might be invoked
- Generic Type
  - Contains instances of other types

Strings:

String literals:

- Single-quoted strings
- Double-quoted strings
- Single-quoted here-strings
- Double-quoted here-strings

Split up like this because of the expression/command-mode

Strings in PowerShell are a sequence of 16-bit unicode characters
Directly implemented using System.String type (.net)

Force single quote variable expansion:
`$ExecutionContext.InvokeCommand.ExpandString( 'a is $var' )`

Here-String example:

```powershell
$a = @"
One is "1"
Two is '2'
Three is $(2+1)
The date is "$(Get-Date)"
"@
```

Here-strings are useful when:

- Generating output for another program
- Used to embed large chunks of text inline in a script
- With Add-Type, to generate C#, or C code
- CSS, to set a specific style

Numbers:

- int, system.int32
  - 1, 0x1FE4
- long, system.int64
  - 100000000
- double, system.double
  - 1.1, 1e3
- single, system.single
- float, system.single (float is a class in single namespace)
- decimal, system.decimal, 1.123d

Multiplier suffixes

- kb or KB
- mb or MB
- gb or GB
- tb or TB
- pb or PB

Dictionaries and hashtables:

- Hashtable initiation starts with '@{' and ends with }
- Property notation: $hash.firstname
- Array access notation $hash['firstname','lastname']
- $hash[$hash.keys] will access all values from the keys
- or use $hash.Values to access all values from the keys
- To iterate over key-value pairs, call the .GetEnumerator() method first
- PowerShell treats hashtables as scalar objects (pointer-to-member)
- To create an ordered hashtable, cast the ordered class before assigning
- Hashtables are references (if you assign it to another variable, its in sync with each other)
- use .clone() to copy a hashtable, without the reference link

Arrays:

- PowerShell has no array initiation, its dynamic
- Arrays in PowerShell are created using the object[] class
- Call the GetType() method to view the type name
- Polymorphic by default (can store any object type) ($a = 1, "string", 1.24d)
- As with hashtables, arrays are referenced ($a=$b will sync updates)
- If you update $b, the reference will be replaced with a copy of $a's content + what you added, and no longer update

Type literals:

- Used to specify a particular type
- Used to cast (convert object to specific type)
- Part of type-constrained variable declaration
- As an object
- Static types methods are private (can only be invoked on already created object)
- .Net types provide rich functionality, best to re-use a type before creating something yourself

Type Conversions:

- Standard (built in) conversions is done by PowerShell Engine (not normal .net type-conversion)

## Operators and Expressions

Arithmetic operators:

+, -, *, /, %

Assignment operators:

=, +=, -=, *=, /=, %=

Comparison operators:

eq, ne, gt, ge, ,lt ,le

ieq..

ceq..

Containment operators:

contains, notcontains, in notin
icontains
ccontains

Pattern-matching and text-manipulation operators:
-like, notlike, match, notmatch, replace, split, ilike, inotlike, imatch, inotmatch, ireplace, isplit, cline, cnotlike, cmatch, cnotmatch, creplace, csplit, join

Local and bitwise operators:
-and, -or, not, xor, shl, band, nor, bnot, bxor, shr

- Works on many types of objects (polymorphic)

Addition operator:

- Array concatenation
- Hashtable "concatenation"
- "Left-hand" rule, if multiple operators, left one determines the operation

Multiple types addition behavior:

```powershell

$a = [int[]](1,2,3,4) # creates an array with type constrained elements (int)
$a[0] = 'hej' # will fail because of element being type constrained

# You can create a new variable that is of type Object[] that is not  type constrained:

$a = $a + 'hej'
$a[0] = 'hej' now works ($a.gettype())

```

Comparison operators:

- ieq, ceq, case-insensitive, case-sensitive
- ieq and eq has the same behavior, ieq was designed to allow script authors implicitly express what they
- PowerShell does not have symbols as comparison (==, >, <) because of the i/o redirection operators, "asd" > asd.txt was prioritized

- Comparison of different types will type-convert from the left:
  - 123 -eq '123' , converts '123' to int 123, etc

Using comparison operators with collections:

if left operand is array or collection, comparison will return only the elements that match the right operand.

1,2,3,4 -eq 3 outputs: 3

- contains & in returns boolean value if any element in the array or collection matches the corresponding operand:
- contains, array CONTAINS value
- in, value is IN array

Wildcard Operators:

- ? matches any single character
- [A-D*] matches anything that starts with a,b,c,d
- [AD*], matches anything that starts with A or D

Regular Expressions 👀:

- Wildcard operators gets translated to corresponding regex internally
- ,*; in regex is wildcard, and any single character (?), is a dot (.)
- Regex operators in powershell are: Match, replace, split
- Match operator generates a variable $Matches, that contains elements of what matched
- Use parentheses around expression when you -join, if one string is desired

- Split unary form splits on whitespace
- Split binary form (with args) can take multiple arguments
- Delimiter, Amount to split, mode (SimpleMatch)
- Split modes: SimpleMatch, IgnoreCase, CultureInvariant, IgnorePatternWhiteSpace, MultiLine, SingleLine, ExplicitCapture

- Where() method is faster then Where-Object
- ForEach() method is NOT faster then foreach statement

## Advanced Operators & Variables

- The greatest challenge to any thinker is stating the problem in a way that will allow a solution.

