# Common Bash Patterns

A helpful reference of common bash script features and functions.
If the answer isn't here, Stack Overflow is your friend.

## If statement

```
if [ true == true ]; then
  echo "Yes, it is true"
fi
```

Alternative formatting

```
if [ true == true ]
then        # "then" is placed on its own line
  echo "Yes, it is true"
fi
```

## If/else statement

```
if [ false == true ]; then
  echo "Yes, it is true"
else
  echo "No, it is false"
fi
```

## Variables

Variables are easy to declare, use, and concatenate

```
a="hello"
b="world"
echo "$a"
echo "$a $b"       # concatenation
```

Boolean variables are not recognized. In the example below, false returns 0,
which allows the if statement to run.

```
if [ false ]; then
  echo "Yes, it is true"
fi
```

Variables are always referenced using a $ symbol, but are not declared using
this symbol. In general, it is best to reference the variable inside of double
quotes to make sure it is interpreted as a variable and not a string. However,
it is not always necessary to use double quotes. This example shows both
approaches to referencing a variable.

```
counter="4"
cat /etc/passwd | head -n "$counter"
echo $counter
```

Bash has special variables which will not be covered in detail here.
Some useful special variables are:
- `$0`: current shell environment
- `$$`: PID of current shell process
- `$_`: The name of the file from which the code is being run
- `$1`, `$2`, `$n`, etc.: the value of the nth argument provided to the bash
command or function. Useful in functions that accept inputs.

## Check if variable is null or empty

See `man test` for additional logic tests and explanations.
Technically there's some distinction between a variable being unset vs. null,
(`foo=` vs. `foo=""`), but this distinction usually doesn't matter in practice.

```
z_test_variable="Not an empty variable"
if [ -z "$z_test_variable" ]; then
      echo "the length of z_test_variable is zero"
else
      echo "the length of z_test_variable is not zero"
fi
```


```
n_test_variable="Not an empty variable"
if [ -n "$n_test_variable" ]; then
      echo "the length of n_test_variable is not zero"
else
      echo "the length of n_test_variable is zero"
fi
```

## Creating an array from a string

```
list="a b c d e f"
my_array=($list)
echo $list            # print out entire string
echo ${my_array[1]}   # print out single character from array
echo ${my_array[3]}
```

## Storing command output as variable

Running a command inside of `$()` lets you store the command output as a
variable. This is helpful when a command's output is later used as input.

```
passwordfile=$(cat /etc/passwd)
echo "$passwordfile"
```

## Integer Arithmetic

Integer arithmetic should occur inside of double parentheses.
Otherwise, integers are treated as strings and concatenated.

```
c=12
d=15
echo "$c+$d"          # integers are concatenated, not added
echo "$(($c+$d))"     # integers are added
```

## For loop

```
for i in $(seq 1 3 30); do
   echo "Number: $i"
done
```

## Iterate over multiline file using for loop

Observe how the following example stumbles over whitespace.
It is usually easier to iterate over a variable using a while loop.

```
password_file="/etc/passwd"
password_file_lines=$(cat $password_file)

for single_line in $password_file_lines; do
    echo "$single_line"
done
```

## Iterate over a variable using for loop

```
list="a b c d e f"
my_array=($list)        # convert string to array
for i in "${my_array[@]}"; do
  echo "$i"
done
```

## Iterate over multiline file using while loop

```
while read myline; do
  echo "$myline"
done < /etc/passwd
```

## Iterate over multiline variable using while loop

```
while IFS= read -r thisfile; do
  cat "$thisfile"
done <<< $(ls -1)
```

## Creating custom function without input

```
print_hello_world () {
  echo "Hello world!"
}

print_hello_world
```

## Creating custom function with input

Use the special variable names `$1`, `$2`...`$n` to use input arguments

```
add_two_numbers () {
    number1=$1
    number2=$2
  echo "$(($number1+$number2))"
}

add_two_numbers 20 9
```

## Substring function

```
small_string="hello world!"
echo "${small_string:6:6}"
```

## Iterate through specific files in current directory

Combine `ls -1` (lists files on separate lines), `ls -p` (appends / to the
end of directory names), and `grep -v /` (removes lines with / character) to
get an iterable list of only files, not directories. Combine with
[iterating over multiline variable](Iterate-over-multiline-variable-using-while-loop)

```
files_list=$(ls -p -1 /var/log/ | grep -v /)  # Each filename on a separate line
while IFS= read -r thisfile; do
  cat "/var/log/$thisfile"
done <<< $files_list
```

## Using head and tail to remove lines at start or end of multiline text

If a bash command prints a row (or rows) at the start or end that you want to
remove, you can use the head and tail commands to remove these rows.

```
ls -l /var/log/ | tail -n+2        # remove the first row of text
cat /etc/passwd | head -n 5        # print only the first 5 rows of text
```

## Extract specific column of text output

The `awk` utility is very useful as extracting individual columns of text, even
when there is variable whitespace in different rows of text. The following
example first removes the first row of text, then prints out the owner of all
files in /var/log/ (data found in column 3).

```
ls -l /var/log/ | tail -n+2 | awk '{ print $3 }'
```

## Removing duplicate rows or data

The `sort` utility allows duplicate rows of data to be deleted. For example, the
previous example can be reused but with duplicate usernames remove using the
`-u` (unique) flag of the sort utility. Another useful flag is `-b`, which
ignores whitespace.

```
ls -l /var/log/ | tail -n+2 | awk '{ print $3 }' | sort -ub
```

## Cutting text into substring using delimiter

The `cut` utility allows data to be parsed using a delimiter of a single
character.

```
# Using delimiter " ", cut the first word of each line
ls -l /var/log/ | cut -d " " -f 1
# A range of data to cut using this delimiter can be specified
ls -l /var/log/ | cut -d " " -f 1-3
# To cut all text after the 4th word of a line, use
# an open-ended hyphen to indicate a range
ls -l /var/log/ | cut -d " " -f 4-
```

## Parsing columns right to left (for example, to extract file types)

Most Linux tools act on data from left to right, not right to left. If you need
to parse data in the other direction, the `rev` utility is the best approach.
It reverses rows of data, and it can be used a second time to return the data
to the proper orientation. The following example shows how

```
ls -1 -p /etc/ | grep -v /                  # prints out file names
# This first attempt (below) at extracting file extensions fails
# because not every line contains a delimiter.
ls -1 -p /etc/ | grep -v / | cut -d "." -f 2
# Solve this by grepping for delimiter
ls -1 -p /etc/ | grep -v / | grep "\." | cut -d "." -f 2
# Now use sort to remove duplicate rows
ls -1 -p /etc/ | grep -v / | grep "\." | cut -d "." -f 2 | sort -u
# Another solution would be to use `rev` to parse from right to left
# This is useful if any file has two delimited (i.e. .tar.gz)
# Observe how `rev` is used twice
ls -1 -p /etc/ | grep -v / | grep "\." | rev | cut -d "." -f 1 | rev | sort -u
```

## Replacing or removing a specific character

Use the `tr` utility to remove or replace a single character in text.

```
# Remove all hyphens from ls -l output
ls -l /etc/ | tr -d "-"
# Replace all hyphens from ls -l output with the + char
ls -l /etc/ | tr "-" "+"
```

## Replacing or removing text greater than one character

If you want to replace specific text substrings with other substrings,
the `sed` utility is an extremely versatile tool for doing so. Review the sed
man file for extended details.

```
# Output /etc/passwd and replace the string :x: with ++++++
cat /etc/passwd | sed 's/:x:/++++++/'
# Remove the 3rd line of text (more granular control than head or tail)
cat /etc/passwd | sed '3d'
# Remove all whitespace of any kind
sed -r 's/\s+//g'
```

## Search files for specific content

If you want to search data inside of files, the `grep` utility is an
extremely versatile tool for doing so. Review the grep man file for
extended details.

```
# Search for the phrase "password" in log files.
# Ignore case and search recursively.
grep -inr "password" /var/log
# Search for multiple phrases in log files, and ignore lines with "username"
# Print out the 3 lines after the found phrase
grep -inrE -A 3 "(password|passwd)" /var/log | grep -vE "(username|user)"
# Sometimes it is nice to have the word you search for highlighted
# Piping output to grep -v removes the highlighting
# The 1st solution to this is to use "--color=always"
grep -inrE -A 3 --color=always "(password|passwd)" /var/log | grep -vE "(username|user)"
# The 2nd alternative solution is to pipe to another grep to recreate the color
grep -inrE -A 3 "(password|passwd)" /var/log | grep -vE "(username|user)" | grep -iE -A 3 "(password|passwd)"
```

## Finding specific file name or types to act on

If you want to search data in the name of a file (either a file extension,
substring of the file name, or more), the `find` utility is an
extremely versatile tool for doing so. Review the find man file for
extended details.

```
# Find files under /var/log/ with an extension of .0 or .1
find /var/log/ -name "*.0" -o -name "*.1"
# Find files under /etc/ with name matching *.conf and ignore case
# For each file found, run the ls command on the file and the file command
find /etc/ -iname "*.conf" -ls -exec file {} \;
```

## Piping output to utilities that don't support piped input

If you want to create a bash one-liner and pipe input into a utility that
doesn't support piped input (for example, it requires the input to be specified
after a certain flag), the `xargs` utility is a versatile tool for this case.
Review the xargs man file for extended details, since this tool has
many features.

```
# First, print the full path of all files in /etc/
ls -ldp /etc/* | grep -vE "/$" | awk '{ print $9 }'
# Now use xargs to pipe into the wc utility
ls -ldp /etc/* | grep -vE "/$" | awk '{ print $9 }' | xargs wc
```

## Use shellcheck for linting and static analysis

When you're building a script to be used across many different environments,
it's helpful to use the [shellcheck](https://github.com/koalaman/shellcheck)
tool to verify whether any of your code may encounter issues in other
shell environments. Making a script compatible between bash, sh, and other
shells for certain edge cases can be tough!

```
shellcheck myscript.sh
```

## Accept input flags (-h, -f)

Bash has a built-in `getopts` utility which is useful to working with input flags
when you are writing a bash script.

```
#!/bin/bash

print_help() {
    echo "Here is the help text!"
    echo "Use: $0 [options]"
  echo
  echo " OPTIONS"
  echo "  -e num1 num2   Evaluate the mathematical expression provided"
  echo "  -s num1        Print the square of the number"
  echo "  -c num1        Print the cube of the number"
  echo "  -h             Help information"
}

square_number() {
    echo "$(($1*$1))"
}

cube_number() {
    echo "$(($1*$1*$1))"
}

eval_expr() {
        echo "Received expression $1. Result is:"
    echo "$(($1))"
}

while getopts "hs:c:e:" option; do
  case "${option}" in
    s) square_number;;
    c) cube_number;;
    e) eval_expr "${OPTARG}";;
    h) print_help; exit 0;;
    *) print_help; exit 1;;
  esac
done
```
