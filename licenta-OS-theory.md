# UNIX processes
In UNIX, processes form the basis of any activity. A process, in the simplest terms, is an executing program. Each process provides the resources needed to execute a program, such as an address space, a set of data structures within the kernel, and one or more threads that execute the program code. 

Let's delve deeper into each concept:

1. **Creation of processes**: In Unix, every process (except the very first process, called the "init" process) is created because an existing process makes an exact copy of itself. This child process has the same environment as the parent, including the same open files, signal handling, user and group ids, and so on. 

2. **fork**: This is the system call that a process uses to replicate itself. `fork` creates a new process by duplicating the existing one. The new process, called the child, is an exact copy of the calling process, called the parent, except for a few values that get changed, such as the process ID. After the `fork`, both processes run independently of each other.

3. **exec**: This system call is used to run a new program in the process. It replaces the current process image with a new process image, resulting in the complete replacement of the process's code and data segment. It's typically used after a `fork` to replace the child process's image with the one we want to run.

4. **exit**: This is the system call a process uses to terminate itself. When a process calls `exit`, it ends its execution and its exit status is returned to the parent process. After `exit` has been called, the process is in the "zombie" state until its parent calls `wait`.

5. **wait**: This system call is used by a parent process to pause until one of its child processes terminates. `wait` also allows the parent to receive the child's exit status. Once the parent has waited on a child process, the child fully terminates and is removed from the system.

6. **Pipe**: A pipe provides a mechanism for interprocess communication. Pipes allow the output of one process to be used as the input to another. For example, in a pipeline like `ls | sort`, `ls` writes to the pipe and `sort` reads from the same pipe.

7. **FIFO (named pipe)**: While anonymous pipes exist only as long as the process that created them keeps running, named pipes or FIFOs are a way of creating a pipe that exists independently of the process that created it. They exist in the file system and can be created using the `mkfifo` command. Multiple processes can read and write to the pipe, but it's primarily used for one-way communication between a writer and a reader.

# Basic concepts
## Variables
Variables in shell programming are a means of storing data and making it available for processing. You can assign a value to a variable with `var=value` (no spaces around `=`) and reference the value with `$var`.

`$F` and `F` are different in shell scripting:
  - F is a variable.
  - $F is the value stored in the variable F.
The $ symbol is used to fetch the value of the variable in shell scripting.

In shell scripts, you often see variables being quoted ("$K") in conditional expressions. The reason for this is to prevent word splitting and pathname expansion.
  - If K is not quoted, and K is empty or unset, the if statement will cause a syntax error. Let's assume K is empty, then the if statement `[ $K != "" ]` will be evaluated as `[ != "" ]` which is a syntax error.
  - But if K is quoted `("$K")`, even if it's empty, it's still considered a single empty string, and no syntax error will occur. The statement `[ "$K" != "" ]` will be evaluated as `[ "" != "" ]` which is syntactically correct.

## Control structures
- `if/then/elif/else/fi`: This is used for conditional statements. `if` starts the condition, `then` starts the block of code that will run if the condition is true, `elif` introduces another condition if the first was false, `else` starts a block of code that will run if all conditions are false, and `fi` ends the conditional statement.
- `for/done`: This is used for loops that run a block of code for each item in a list.
- `while/do/done`: This is used for loops that run a block of code as long as a condition is true.
- `shift`: This command shifts the positional parameters to the left, discarding the first parameter and reducing the total count by one. Useful in scripts that take multiple arguments.
- `break`: This command breaks out of a loop.
- `continue`: This command skips the rest of the current loop iteration and continues with the next iteration.
  
```bash
# if/then/elif/else/fi
if [ $name == "Alice" ]
then
  echo "Hello, Alice!"
elif [ $name == "Bob" ]
then
  echo "Hello, Bob!"
else
  echo "Hello, stranger!"
fi

# for/done
for i in 1 2 3
do
  echo $i  # This will print: 1, then 2, then 3
done

# while/do/done
i=1
while [ $i -le 3 ]
do
  echo $i  # This will print: 1, then 2, then 3
  i=$((i + 1))
done

# shift
set -- "arg1" "arg2" "arg3"  # Set script arguments
echo $1  # This will print: arg1
shift
echo $1  # This will print: arg2

# break
for i in 1 2 3
do
  if [ $i -gt 2 ]
  then
    break
  fi
  echo $i  # This will print: 1, then 2
done

# continue
for i in 1 2 3
do
  if [ $i -eq 2 ]
  then
    continue
  fi
  echo $i  # This will print: 1, then 3
done
```

`shift` is a bash built-in which kind of removes arguments from the beginning of the argument list. Given that the 3 arguments provided to the script are available in $1, $2, $3, then a call to shift will make $2 the new $1. A shift 2 will shift by two making new $1 the old $3.

## Predefined variables
  - `$0`, `$1`, ..., `$9`: These represent the arguments passed to a script. `$0` is the name of the script itself, `$1` is the first argument, `$2` is the second argument, and so on.
  - `$*` and `$@`: Both of these represent all arguments passed to a script. They behave slightly differently when quoted: `$*` gives a single string joined with spaces, while `$@` gives a list of separate strings.
  - `$?`: This is the exit status of the most recently completed command. An exit status of 0 usually indicates success, while anything else indicates an error.
    
``` bash
# For this example, imagine you've run a script like this:
# ./myscript arg1 arg2 arg3

# Inside the script, you could do:
echo $0  # This will print: ./myscript
echo $1  # This will print: arg1
echo $*  # This will print: arg1 arg2 arg3
echo $?  # This will print: 0 (if the previous command succeeded)

# The behavior of $* and $@ is more apparent in a loop:
for arg in "$*"; do echo $arg; done  # This prints: arg1 arg2 arg3
for arg in "$@"; do echo $arg; done  # This prints: arg1, then arg2, then arg3
```

## I/O Redirections
https://www.digitalocean.com/community/tutorials/an-introduction-to-linux-i-o-redirection

- `|`: This is a pipe, which sends the output of the command on its left as input to the command on its right.
- `>, >>`: These redirect output to a file. `>` overwrites the file, while `>>` appends to the file.
- `<`: This redirects the input from a file.
- `2>, 2>>`: These redirect error output to a file. `2>` overwrites the file, while `2>>` appends to the file.
- `2>&1`: This redirects error output to the same place as normal output.
- `/dev/null`: This is a special file that discards all data written to it. It's often used to discard unwanted output.
- Back-quotes " ` ": Anything between back-quotes is treated as a command to be executed, and the back-quotes are replaced with the command's output. This is also known as command substitution. In modern scripts, `$(command)` is preferred because it is easier to nest.

#  Extended regular expressions (POSIX ERE, as supported by "grep -E" and "sed -E")
In Unix-like operating systems, regular expressions (regex) are a way to specify complex patterns of characters. They're used in many command line tools, like `grep`, `sed`, `awk`, and many others, to match lines of text.

There are several different "flavors" of regular expressions. The two most common in Unix-like systems are basic regular expressions (BRE) and extended regular expressions (ERE).

## Basic regular expressions (BRE)
Basic regular expressions are the original and simpler form. They support a limited set of special characters and metacharacters like `*`, `.` and `[]`.
- `*`: This is called the "asterisk" or "star". In basic regular expressions, it matches zero or more of the preceding character or group. For example, the regex `a*` could match 'a', 'aa', 'aaa', etc., as well as an empty string.
- `.`: This is the "dot" or "period". It matches any single character. For example, `a.b` could match 'aab', 'acb', 'a5b', etc.
- `[]`: These are "character classes" or "character sets". They match any one character that's inside the brackets. For example, `[abc]` could match 'a', 'b', or 'c'.

You can also use a hyphen to specify a range of characters, like `[a-z]` for any lowercase letter, `[A-Z]` for any uppercase letter, or `[0-9]` for any digit. 

If the first character after the opening bracket is a caret (`^`), it inverts the character class, matching any character that's not inside the brackets. For example, `[^abc]` matches any character except 'a', 'b', or 'c'.

## Extended regular expressions
Extended regular expressions, on the other hand, introduce additional features like `+`, `?`, `{}`, `|` and `()` that make it easier to specify complex patterns. 

- `+` matches one or more of the preceding character or group. For example, `a+` matches 'a', 'aa', 'aaa', etc.
- `?` matches zero or one of the preceding character or group. For example, `a?` matches '', 'a'.
- `{}` is a quantifier that acts a bit like `*` and `+`, but lets you specify exactly how many times you want the thing to repeat. `{3}` repeats exactly 3 times, `{3,}` repeats 3 or more times, `{,3}` repeats up to 3 times, and `{3,5}` repeats 3, 4, or 5 times.
- `|` acts like a logical OR. Matches the pattern before or the pattern after the operator.
- `()` are used for grouping expressions, allowing you to apply operators to groups of characters rather than just single characters.

For example, let's say you want to match lines that contain either 'apple' or 'banana'. With extended regular expressions, you could write this as `apple|banana`. This wouldn't be possible with basic regular expressions.

The `grep -E` command tells `grep` to use extended regular expressions, and `sed -E` tells `sed` to do the same. For example, you could use `grep -E 'apple|banana' file.txt` to print lines in `file.txt` that contain either 'apple' or 'banana'.

# Basic commands
- `cat`: Concatenates and prints files. Given one or more filenames, it prints the contents of those files to the standard output.

- `chmod (-R)`: Changes file permissions. `-R` stands for recursive, changes files and directories recursively.

- `cp (-r)`: Copies files or directories. `-r` stands for recursive, copies directories recursively.

- `cut (-d, -f)`: Cuts out selected portions of each line of a file. `-d` specifies a delimiter and `-f` specifies the fields to be cut.
  - cut OPTION... [FILE]...
  - With no FILE, or when FILE is -, read standard input.
- `echo`: Prints its arguments to standard output.

- `expr`: Evaluates expressions. It's often used in scripts for arithmetic, comparison of numbers, and string operations.

- `file`: Determines the file type of a given file or files.

- `find (-name, -type)`: Searches directories for matching files. `-name` specifies the filename to search for, and `-type` specifies the type of file (such as 'd' for directory, 'f' for regular file).

- `grep (-E, -i, -q, -v)`: Searches files for a matching pattern. `-E` enables extended regular expressions, `-i` makes the search case insensitive, `-q` makes grep operate quietly (not outputting anything), and `-v` inverts the match, finding lines that do not match the pattern.

- `head (-n)`: Prints the first part (the "head") of files. `-n` specifies the number of lines to print.
  - `head -n 5 myfile.txt` : print the first 5 lines from the file
    
- `ls (-l)`: Lists directory contents. `-l` uses a long listing format, showing file permissions, number of links, owner, group, size, and time of last modification.

- `mkdir (-p)`: Makes directories. `-p` makes parent directories as needed, and does not return an error if the directory already exists.
  - `mkdir -p dir1/dir2/dir3` : For example, to create a directory dir1, inside it another directory dir2, and within dir2 a directory dir3 (even if dir1 and dir2 don't exist yet), you would use thic command
    
- `mv`: Moves or renames files.
  - `mv oldname.txt newname.txt` : For example, to rename a file from oldname.txt to newname.txt, you would use this command.
  - `mv myfile.txt dir1/` : If you want to move a file myfile.txt to another directory dir1, you would use:
  - `mv *.txt dir1/` : If you want to move all text files from the current directory to dir1, you would use:

- `ps (-e, -f)`: Reports a snapshot of the current processes. `-e` selects all processes, and `-f` uses a full-format listing.

- `pwd`: Prints the name of the current working directory.

- `read (-p)`: Reads a line from standard input. `-p` prompts with a string before trying to read.
  - For example, to prompt the user for their name and store it in a variable, you could use:
  - ```bash
    read -p "Please enter your name: " name
    echo "Hello, $name!"
    ```
- `rm (-f, -r)`: Removes files or directories. `-f` forces removal without confirmation, and `-r` removes directories and their contents recursively.
  - `rm -f myfile.txt` : to delete a file without confirmation.
  - `rm -r mydir` : to delete a directory. If you attempt to use the rm command on a directory without the -r or -R option (which stands for "recursive"), you will get an error message indicating that it's a directory and cannot be removed using rm alone.
    
- `sed (-E and only the commands d, s, y)`: A stream editor for filtering and transforming text. `-E` enables extended regular expressions. The `d` command deletes lines that match the pattern, the `s` command replaces text, and the `y` command transforms characters.

- `sleep`: Delays for a specified amount of time.

- `sort (-n, -r)`: Sorts lines in text files. `-n` sorts numerically, and `-r` sorts in reverse order.

- `tail (-n)`: Prints the last part (the "tail") of files. `-n` specifies the number of lines to print.
  -  For example, to display the last 5 lines of a file named `myfile.txt`, you could use:
   ```bash
   tail -n 5 myfile.txt
   ``` 

- `test (numerical, string and file operators)`: Evaluates an expression and returns a status code. It can evaluate numerical expressions, string expressions, and file conditions.

- `true`: Returns a status code indicating success.

- `uniq (-c)`: Filters out repeated lines in a file. `-c` prefixes lines by the number of occurrences.

- `wc (-c, -l, -w)`: Prints newline, word, and byte counts for files. `-c` prints the byte counts, `-l

## grep
```grep <PATTERN> <FILME_NAME> <ANOTHER_FILE_NAME> ..```

grep searches for PATTERNS in each FILE.  
- PATTERNS is one or more patterns separated by newline characters, and grep prints each line that matches a pattern.  Typically PATTERNS should be quoted when grep is used in a shell command.
- A FILE of “-” stands for standard input.  If no FILE is given, recursive searches examine the working directory, and nonrecursive searches read standard input.

- `-v, --invert-match` : Invert the sense of matching, to select non-matching lines.
- `E, --extended-regexp` : Interpret PATTERNS as extended regular expressions (EREs)
- `-i, --ignore-case` : Ignore case distinctions in patterns and input data, so that characters that differ only in case match each other.
- `q, --quiet, --silent` : Quiet; do not write anything to standard output.  Exit immediately with zero status if any match is found, even if an error was detected.

## sed 
Sure, `sed` stands for "stream editor". It's a utility used for parsing and transforming text. Here are examples of the `d`, `s`, and `y` commands:

1. `d` (delete):

    The `d` command is used to delete lines that match a certain pattern. For example, to delete all lines containing the word "delete" in a file named `myfile.txt`, you could use:

    ```bash
    sed '/delete/d' myfile.txt
    ```

    Note that this command doesn't modify `myfile.txt` -- it just outputs the modified text. If you want to save the modified text back to `myfile.txt`, you could use:

    ```bash
    sed -i '/delete/d' myfile.txt
    ```

2. `s` (substitute):

    The `s` command is used to replace occurrences of a regular expression with something else. For example, to replace all occurrences of "old" with "new" in `myfile.txt`, you could use:

    ```bash
    sed 's/old/new/g' myfile.txt
    ```

    This command replaces "old" with "new" on each line of `myfile.txt`. The `g` after the final delimiter is an option meaning "global". Without it, `sed` would only replace the first occurrence of "old" on each line.

3. `y` (transliterate):

    The `y` command is used to replace all occurrences of certain characters with other characters. For example, to replace all 'a's with 'b's and all 'c's with 'd's in `myfile.txt`, you could use:

    ```bash
    sed 'y/ac/bd/' myfile.txt
    ```

    Note that unlike `s`, `y` does not recognize regular expressions. It literally replaces each character listed before the `/` with the character at the same position in the list after the `/`.

Finally, here's how you could use the `-E` option with the `s` command:

The `-E` option tells `sed` to use extended regular expressions. For example, to replace any sequence of one or more spaces in `myfile.txt` with a single space, you could use:

```bash
sed -E 's/ +/ /g' myfile.txt
```

In this command, ` +` is an extended regular expression that matches one or more spaces. Without the `-E` option, `sed` would interpret it as a basic regular expression that matches a space followed by a plus sign.

## chmod
`chmod` is a command in Linux and other Unix-like operating systems that allows you to change the access permissions of file system objects. It stands for "change mode", and it's used to define the way a file can be accessed.

In Unix-like operating systems, each file and directory has a set of permissions associated with it. These permissions control who can read, write, or execute the file. The permissions are divided into three groups:
1. User: The owner of the file.
2. Group: Other files which are in the same folder or directory.
3. Others: Everyone else.

Each group has three permissions that can be either allowed or denied:
1. Read (r): Permission to read the file.
2. Write (w): Permission to modify the file.
3. Execute (x): Permission to execute the file.

For example, in the command `chmod 700 $f`, `700` is an octal number that represents the permissions. The first digit `7` is for the user, the second digit `0` is for the group, and the third digit `0` is for others. The number `7` (in binary `111`) means read, write, and execute permissions are all set. The number `0` (in binary `000`) means no permissions are set. So `chmod 700 $f` means the user can read, write, and execute `$f`, but nobody else can do anything with it.

`! -x $f` is a condition used in the script's `if` statement. Here `-x` is a test that checks if the execute permission is set on the file `$f`. If `-x $f` is true, then `$f` is executable. But `! -x $f` negates this condition. So `! -x $f` is true if `$f` is NOT executable. In the context of the script, `if [ ! -x $f ]` checks if the file `$f` is not executable, and if it's not, the script changes its permissions to make it executable for the user.

``` bash
if [ -x "$f" ]; then
    echo "$f is EXECUTABLE"
fi

if [ -r "$f" ]; then
    echo "$f is readable"
fi

if [ -w "$f" ]; then
    echo "$f is writable"
fi
```

## cut
The `cut` command in Unix and Linux is used to remove sections from each line of files. Here are some examples:

1. Suppose you have a file `test.txt` with the following contents:

```
John,Doe,30,New York
Jane,Doe,25,Los Angeles
```

Now, if you want to cut out the first field of each line in the file (i.e., the first names), you can use `-d` to specify the delimiter (comma in this case) and `-f` to specify the field number:

```bash
cut -d',' -f1 test.txt
```

This would output:

```
John
Jane
```

2. You can also specify multiple fields. For instance, to get the first names and the ages:

```bash
cut -d',' -f1,3 test.txt
```

This would output:

```
John,30
Jane,25
```

3. If you want to cut a range of fields, you can use a hyphen. For example, to get the first through third fields:

```bash
cut -d',' -f1-3 test.txt
```

This would output:

```
John,Doe,30
Jane,Doe,25
```

4. If the file is tab-delimited or uses space as a delimiter, you don't need the `-d` option. For example, if you have a space-separated file and want to cut the first field, you could just use:

```bash
cut -f1 test.txt
```

Remember that `cut` does not modify the original file; it just sends the result to the standard output. If you want to store the result in another file, you can use a redirection operator. For example:

```bash
cut -d',' -f1 test.txt > first_names.txt
```

This would put the first names in a new file called `first_names.txt`.

## cp
The `cp` command in Unix and Linux is used to copy files and directories. Here are some examples:

1. To copy a file:
    ```bash
    cp source.txt destination.txt
    ```
    This command copies the contents of `source.txt` into `destination.txt`. If `destination.txt` doesn't exist, it will be created. If it does exist, it will be overwritten.

2. To copy a file to a directory, you can specify the directory as the destination:
    ```bash
    cp source.txt /path/to/destination/directory
    ```
    This command copies `source.txt` to the specified directory.

3. To copy multiple files to a directory, you can specify the files followed by the directory:
    ```bash
    cp source1.txt source2.txt /path/to/destination/directory
    ```
    This command copies both `source1.txt` and `source2.txt` to the specified directory.

4. To copy a directory and its contents, you need to use the `-r` (or `-R`) option for recursive copy:
    ```bash
    cp -r /path/to/source/directory /path/to/destination/directory
    ```
    This command copies the source directory and all its contents (including subdirectories and their contents, and so on) to the destination. If the destination directory doesn't exist, it will be created. If it does exist, the contents of the source directory will be merged with those in the destination directory, with matching files being overwritten.

Remember that these commands do not delete or modify the source files/directories; they create copies in the destination location.

## expr
The `expr` command in Unix or Linux is used to evaluate expressions, including arithmetic, comparison, and string operations. Here are some examples:

1. Arithmetic Operations:

    ```bash
    expr 5 + 3
    ```
    This command will output `8`. It calculates the sum of `5` and `3`.

    Similarly, you can perform subtraction, multiplication, and division:

    ```bash
    expr 5 - 3     # Outputs: 2
    expr 5 \* 3    # Outputs: 15 (note the backslash before the asterisk)
    expr 10 / 2    # Outputs: 5
    ```

    Note that `expr` performs integer division, so `expr 10 / 3` would output `3`, not `3.333`.

2. Comparison Operations:

    ```bash
    expr 5 \> 3    # Outputs: 1 (true)
    expr 5 \< 3    # Outputs: 0 (false)
    expr 5 = 3    # Outputs: 0 (false)
    expr 5 != 3   # Outputs: 1 (true)
    ```

    Note the backslashes before the `<` and `>` symbols; these are necessary because these symbols have special meanings in the shell.

3. String Operations:

    ```bash
    expr length "Hello World"     # Outputs: 11 (length of the string)
    expr substr "Hello World" 1 5    # Outputs: Hello (substring starting at position 1 and 5 characters long)
    expr index "Hello World" o    # Outputs: 5 (position of the first 'o' in the string)
    ```

    Note that string positions start from 1 in `expr`, not 0.

    The `expr` command is often used in shell scripts. For example, to increment a variable `i`:

    ```bash
    i=5
    i=$(expr $i + 1)
    echo $i    # Outputs: 6
    ```

    Note the `$(...)` syntax, which runs a command and substitutes its output. So `$(expr $i + 1)` runs `expr`, adding 1 to the value of `i`, and then this result is assigned back to `i`.

## file
The `file` command in Unix or Linux is used to determine the type of a given file or files. It makes an educated guess about the file's type by reading its contents and possibly checking other attributes. Here are some examples:

1. To determine the type of a single file:
    ```bash
    file myfile.txt
    ```
    If `myfile.txt` is a plain text file, this might output `myfile.txt: ASCII text`.

2. The `file` command can also determine types for several types of binary files. For example, if you have an executable file `myprogram`:
    ```bash
    file myprogram
    ```
    This might output something like `myprogram: ELF 64-bit LSB executable`, indicating that `myprogram` is a 64-bit Linux executable.

3. To determine the type of all files in the current directory, you can use the `*` wildcard:
    ```bash
    file *
    ```
    This will print the file types of all files in the current directory.

4. `file` can also follow symbolic links to report on the file the link points to, using the `-L` option:
    ```bash
    file -L mylink
    ```
    If `mylink` is a symbolic link to `myfile.txt`, this will report the type of `myfile.txt`, not the link itself.

5. To display mime type of a file, you can use the `-i` option:
    ```bash
    file -i myfile.txt
    ```
    This might output something like `myfile.txt: text/plain; charset=us-ascii`, indicating that `myfile.txt` is a plain text file using US ASCII character encoding.

The `file` command is handy when you have a file, but you're not sure what type of file it is.

## find
The `find` command in Unix or Linux is a powerful utility used to search for files in a directory hierarchy based on specified criteria such as filename, last modification time, file size, etc. Here are some examples:

1. To find files by name:
    ```bash
    find /path/to/directory -name "myfile.txt"
    ```
    This command will search the directory at `/path/to/directory` and all of its subdirectories for a file named `myfile.txt`.

    The `-name` option is case-sensitive. If you want to ignore case, use `-iname` instead.

2. You can use wildcards with `-name` to match multiple files. For example, to find all `.txt` files:
    ```bash
    find /path/to/directory -name "*.txt"
    ```
    This command will find all files ending with `.txt`.

3. To find files by type:
    ```bash
    find /path/to/directory -type f
    ```
    This command will find all regular files in `/path/to/directory` and its subdirectories. The `-type f` option specifies that you're looking for regular files.

    To find directories instead, use `-type d`.

4. You can combine `-name` and `-type`. For example, to find all directories named `mydir`:
    ```bash
    find /path/to/directory -type d -name "mydir"
    ```
    This command will find all directories named `mydir`.

Remember that if you're running these commands from the shell, and your file names might contain spaces or special characters, it's a good idea to quote your patterns (like `"myfile.txt"` and `"*.txt"` in the examples above). This prevents the shell from interpreting these special characters before `find` sees them.

## sort
The `sort` command in Unix or Linux is used to sort the lines in text files. The `-n` option is used to sort numerically, and the `-r` option is used to reverse the result of comparisons, which effectively sorts in descending order.

Suppose you have a file `numbers.txt` with the following content:
```
10
2
5
1
```

To sort these numbers in descending order, you could use:
```bash
sort -n -r numbers.txt
```

This will output:
```
10
5
2
1
```

## test

The `test` command is used to evaluate conditional expressions. 

Here are some examples:

- Numerical operators: 

 To check if a variable `$num` is equal to 10, you could use:
 ```bash
 test $num -eq 10
 ```
 If the `$num` is equal to 10, the `test` command will return a zero (true) status. Otherwise, it will return a non-zero (false) status.

- String operators:

 To check if a variable `$str` is empty, you could use:
 ```bash
 test -z $str
 ```
 If the `$str` is empty, the `test` command will return a zero (true) status. Otherwise, it will return a non-zero (false) status.

- File operators:

 To check if a file `myfile.txt` exists, you could use:
 ```bash
 test -f myfile.txt
 ```
 If the `myfile.txt` exists and is a regular file, the `test` command will return a zero (true) status. Otherwise, it will return a non-zero (false) status.

The `test` command is often used in conditional statements, like `if` statements. For example:
```bash
if test -f myfile.txt
then
   echo "myfile.txt exists."
else
   echo "myfile.txt does not exist."
fi
```

Note: In shell scripts, `[ expression ]` is equivalent to `test expression`. For example, `if [ -f myfile.txt ]` is the same as `if test -f myfile.txt`.

Sure, here are examples for each command:

## uniq

The `uniq` command in Unix or Linux is used to report or filter out repeated lines in a file. The `-c` option is used to prefix lines by the number of occurrences.

Suppose you have a file `names.txt` with the following content:
```
Alice
Bob
Alice
Alice
Bob
Charlie
Charlie
```
To count the number of occurrences of each name, you could use:

```bash
sort names.txt | uniq -c
```

This will output:
```
3 Alice
2 Bob
2 Charlie
```

## wc

The `wc` command is used to count newline counts, word counts, and byte counts. The `-c` option is used to count bytes, `-l` is used to count lines, and `-w` is used to count words.

For example, to count the number of bytes, lines, and words in a file named `myfile.txt`, you could use:

```bash
wc -c -l -w myfile.txt
```

This will output the number of bytes, lines, and words in `myfile.txt`, in that order.

## who

The `who` command is used to show who is logged on to the system.

For example, to see a list of users currently logged in to the system, you could simply use:

```bash
who
```

This will output a list of usernames, the terminals they're logged in from, and the times they logged in. The exact format may vary between systems.

## ps
If you do `ps -ef`, a list with these columns will be returned:
```
UID          PID    PPID  C STIME TTY          TIME CMD
```

- UID: User Identifier. This is the user ID of the user who owns the process. For example, processes started by the root user will have a UID of 0.
- PID: Process Identifier. This is a unique number that identifies the process. PIDs are reused as processes terminate and new ones are created.
- PPID: Parent Process Identifier. This is the PID of the parent of this process. For example, if you start a process in the background from your shell, the PPID of that process will be the PID of the shell.
- C: CPU utilization of the process. This field represents the amount of CPU time used by the process.
- STIME: Start Time. This is the time when the process was started.
- TTY: Terminal Type. This represents the terminal that the process is running on. If the process is not associated with a terminal, this field may be '?'.
- TIME: Cumulative CPU Time. This is the total CPU time the task has taken to execute. Note that this is different from real elapsed time, as a process can be in the background and not using any CPU, but still be "running".
- CMD: Command. This is the command with all its arguments as it was typed and run by the user. If the process is a kernel thread or a process without an associated terminal, this field may not be meaningful.

