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


## Elastic type system:

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
- [Windows PowerShell Language Specification Version 3.0](https://www.microsoft.com/en-us/download/confirmation.aspx?id=36389)
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


## Working with Types

