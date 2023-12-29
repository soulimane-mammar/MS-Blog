# Variables on the Linux Command-Line

Variables can be thought of as named containers. You can place data into these containers and then refer to the data simply by naming the container

This section describes the following:

- Variable assignment
- Variable substitution
- Indirect variables (namerefs)
- Built-in shell variables
- Arrays
- Prompt strings

Before diving into declaring a variable, it's important to familiarize ourselves with another command: the `echo` command. Essentially, `echo` allows us to output text to the terminal. For instance, by using `echo "Hello World!"`, whatever is enclosed within the double quotes will be displayed when executed.

![echo command](https://github.com/soulimane-mammar/HTG/blob/main/Shell_Variables/echo_hello_world.png?raw=true)

Now let's create a variable. A variable name consist of any number of letters, digits, or underscores. Upper and lowercase letters are distinct, and names may not start with a digit. Variables are assigned values using the `=` operator. **There must not be any whitespace between the variable name and the value.**

```bash
$ myvar="Hello World!"
$ echo $myvar
Hello World!
```

The second command above prints the content stored in the variable `myvar` by referencing it using the `$` sign

You can make multiple assignments on the same line by separating each one with whitespace:

```bash
myFirstName=Soulimane myLastName=Mammar
```

The shell typically considers variable values as strings, regardless of whether the string contains only digits. However, when a value is assigned to an integer variable, defined using `declare -i`, Bash interprets the right-hand side of the assignment as an expression to evaluate.

```bash
$ v1=5+6 ; echo $v1
5+6
$ declare -i v2 ; v2=5+6 ; echo $v2
11
```

The `+=` operator enables the addition or appending of the right-hand side of the assignment to an existing value. For integer variables, the right-hand side is treated as an expression that gets evaluated and added to the value. When used with arrays, it appends the new elements to the existing array.

```bash
$ v=5 ; echo $v
5
$ v+=2+6 ; echo $v
13
```

## Variable substitution

Variable substitution refers to the process of replacing a variable reference with its stored value within a command or string.
We've already seen the **simple substitution** by just preceding the variable name with the `$` sign as in

```bash
$ greet=Hello
$echo $greet World!
Hello World!
```

For more clarity and delimitation we use the curly braces syntax as in

```bash
$ direction=up
$ # Braces are needed here
$ echo ${direction}stairs 
upstairs
```

The `${var:-value}` substitute `var` if set; otherwise, use `value`

```bash
$ echo ${direction:-'up'}stairs
upstairs
$ direction=down
$ echo ${direction:-'up'}stairs
downstairs
```

The `${var:=value}` substitute `var` if set; otherwise, use `value` and assign `value` to `var`

```bash
echo ${direction:='up'}stairs
upstairs
echo $direction
up
```

The `${var:+value}` substitute  `value` if `var` is set; otherwise, use nothing.

```bash
$ echo ${direction:+'downstairs'}

$ direction=up
echo ${direction:+'downstairs'}
downstairs
```

The `${#var}` substitute the length of `var`

```bash
$ greeting="Hello World"
$ echo ${#greeting}
11
```

## Indirect Variables (namerefs)

Indirect variables, known as namerefs, serve as aliases for other variables. Namerefs are created using `declare -n`

``` bash
$ greeting="hello, world"           Variable assignment
$ declare -n message=greeting       Declare the nameref
$ echo $message                     Access using the nameref
hello, world                        
$ message="bye now"                 Assign using the nameref
$ echo $greeting                    Show the change
bye now
```

A nameref can be used as the control variable in a for loop

```bash
$ declare -n i 
$ j=1 
$ for i in var1 var2 var3 
> do
> i=$((j++)) 
> done
$ echo $var1 $var2 $var3
1 2 3
```

## Built-In Shell Variables

Built-in variables are automatically defined and primarily utilized within shell scripts.  The following variables are the most frequently used ones that are available in almost any Bourne-compatible shell:

| Variable          | Meaning                                                                                        |
|-------------------|------------------------------------------------------------------------------------------------|
| $$                | Process ID of the current shell                                                                |
| $#                | Number of command-line arguments.                                                              |
| $?                | Exit status of the last executed command                                                       |
| $!                | Process ID of the last command run in the background                                           |
| $0                | Name of the script                                                                             |
| \$* or $@          | All command-line arguments passed to the script (as a single string or individually quoted)   |
| $1, $2, ... $n    | Individual command-line arguments (first, second, nth argument)                                |
| $USER             | Username of the current user                                                                   |
| $HOME             | Home directory of the current user                                                             |
| $PWD              | Present working directory (current directory)                                                  |
| $RANDOM           | Generates a random number                                                                      |
| $HOSTNAME         | Name of the host                                                                               |
| $SECONDS          | Number of seconds since the script started                                                     |
| $OSTYPE           | Operating system type                                                                          |
| $LINENO           | Current line number in the script                                                              |
| $BASH_VERSION     | Version of Bash                                                                                |

## Arrays

In Bash there are two kinds of arrays, indexed and associative

### Indexed arrays

We can create an indexed array by using one of the following syntaxes:

```bash
numbers=(zero one two three four five)
```

or

```bash
numbers[0]=zero
numbers[1]=one
numbers[2]=two
numbers[3]=three
numbers[4]=four
numbers[5]=five
```

When working with arrays, we utilize `${...}` for referencing. Negative subscripts count from the last index, with `-1` denoting the last element, `-2` the second-to-last, and so on.

```bash
$ arr=(0 1 2 3 4 5 6 7 8) 
$ echo ${arr[4]} 
4
$ echo ${arr[-2]} 
7
```

The variable substitutions for arrays and their elements are outlined as follows:

| variable   | meaning                                          |
|----------  |--------------------------------------------------|
|${array[i]} | Element i of the array                           |
|${array}    | Element 0 of the array                           |
|\${array[*]},${array[@]}| All elements of the array|
|\${#array[*]},${#array[@]}| The number of elements in the array|

### Associative arrays

Associative arrays are arrays in which indices are strings. They are declared using `declare -A` command. There's a special syntax that enables the assignment of values to multiple elements simultaneously within arrays.

```bash
assoc=([one]=1 two=[2] [three]=3)
```

Use `${assoc[one]}`, `${assoc[two]}` and `${assoc[three]}` to access the value

## Prompt Strings

Bash provides five customizable prompt strings:

- `PS0` appears after each command but before any output.
- `PS1`, the primary prompt, precedes every command, making it the most commonly customized prompt.
- `PS2` serves as the secondary prompt, appearing when a command requires additional input, like a multi-line command. By default is set to the string `>`
- `PS3`, less commonly used, is the prompt for Bash's `select` built-in command.
- `PS4`, also uncommonly used, appears when debugging bash scripts, indicating levels of indirection. The first character is repeated to signify deeper levels.

Bash process these variables for the following special escape sequences:

| Escape Sequence | Description                                       |
|-----------------|---------------------------------------------------|
| \u              | Username                                          |
| \h              | Hostname (short)                                  |
| \H              | Full hostname                                     |
| \w              | Current working directory                          |
| \W              | Basename of the current working directory          |
| \t              | Time in 24-hour format (HH:MM:SS)                 |
| \T              | Time in 12-hour format (HH:MM:SS)                 |
| \@              | Time in 12-hour format with AM/PM (hh:mm AM/PM)    |
| \d              | Date in "Weekday Month Day" format (e.g., Fri Sep 1)|
| \n              | Newline                                           |
| \s              | Shell name                                        |
| \A              | Time in 24-hour format without seconds (HH:MM)     |
| \\              | Backslash                                         |
| \@              | Time in 12-hour format with AM/PM (hh:mm AM/PM)    |
| \[              | Start of a sequence of non-printing characters     |
| \]              | End of a sequence of non-printing characters       |

#### Example

![Changing Prompts](https://github.com/soulimane-mammar/HTG/blob/main/Shell_Variables/changing_prompts.png?raw=true)

In the provided example, the initial line defines the `PS1` variable, altering the shell prompt format. Subsequently, as seen in the second line when changing directory, the prompt changes accordingly. Line 4 modifies the `PS2` variable, influencing the prompt displayed during multi-line commands, as shown in the following command execution.
