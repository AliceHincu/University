# 2019 manual
### I
**b. What will print to the screen the program fragment below, considering that the fork system call is successful? Justify your answer.**
```c
int main() {
    int n = 1;
    if(fork() == 0) {
        n = n + 1;
        exit(0);
    }
    n = n + 2;
    printf(“%d: %d\n”, getpid(), n);
    wait(0);
    return 0;
}
```

- The fork() system call creates a new child process. It returns 0 in the child process and returns the child's process ID in the parent process.
- In the child process, n will be incremented by 1, so n becomes 2, and then it calls exit(0). The child process doesn't contain any printf() statement, so nothing is printed from the child process.
- In the parent process, fork() does not return 0, so it skips the if statement. Then n is incremented by 2, so n becomes 3. The printf() function is then called, which prints the process ID of the parent process and the value of n, which is 3.
- Therefore, this program will print something like: 12345: 3\n, where 12345 is the process ID of the parent process (it will be a different number when you run the program).


**c. What will print to the screen the shell script fragment below? Explain the functioning of the first three lines of the fragment.**
``` bash
1 for F in *.txt; do
2   K=`grep abc $F`
3   if [ “$K” != “” ]; then
4       echo $F
5   fi
6 done
```

This Bash shell script scans all .txt files in the current directory for the occurrence of the string "abc". If a file contains "abc", it prints the name of the file.
1. for F in *.txt; do: -> This is the start of a for loop that iterates over every file in the current directory that ends in .txt. F is a variable that holds the name of the current file being processed.
2. K=`grep abc $F` -> This line uses the grep command to search for the string "abc" in the current file ($F). If "abc" is found, grep will return the lines that contain "abc", otherwise, it returns nothing.
3. if [ “$K” != “” ]; then: -> This line starts an if statement that checks if K is not equal to an empty string. If K is not an empty string, it means that grep found "abc" in the current file, and so the body of the if statement will be executed.

### II.
**a. Consider the code fragment below. What lines will it print to the screen and in what order, considering that the fork system call is successful? Justify your answer.**

``` c
int main() {
    int i;
    for(i=0; i<2; i++) {
        printf("%d: %d\n", getpid(), i);
        if(fork() == 0) {
            printf("%d: %d\n", getpid(), i);
            exit(0);
        }
    }
 
    for(i=0; i<2; i++) {
        wait(0);
    }
    return 0;
}
```

First, consider that the parent process has PID 12345. The first iteration (i = 0) of the loop in the parent process prints:
```12345: 0```
Then fork() is called and a child is created, which prints:
```12346: 0```
Then the child process exits.

Now we are back to the parent process, we increase i by 1 and go into the second iteration of the loop (i = 1), and the parent process prints:
```12345: 1```
Then fork() is called and another child is created, which prints:
```12347: 1```
Then the second child process exits. Finally, the parent process waits for its children to finish in the second for loop.

So, the possible output of the program would be:
```
12345: 0
12346: 0
12345: 1
12347: 1
```
This might not be the exact order of the printed lines, because the scheduling of the processes is dependent on the operating system and it might schedule the processes in a different order. But the pattern of having the same i values for each pair of parent-child print statements will remain.


**b. Explain the functioning of the shell script fragment below. What happens if the file report.txt is missing initially? Add the line of code missing for generating the file report.txt**

```bash
more report.txt
rm report.txt
for f in *.sh; do
    if [ ! -x $f ]; then
        chmod 700 $f
    fi
done
mail -s "Affected files report" admin@scs.ubbcluj.ro <report.txt
```

1. more report.txt -> It tries to display the content of report.txt using the more command. If the file report.txt does not exist, this command will show an error message like "report.txt: No such file or directory".
2. rm report.txt -> It tries to remove the file report.txt. If the file doesn't exist, it will just do nothing when -f option is used, otherwise, it would throw an error stating that report.txt doesn't exist.
3. The for loop iterates over all files in the current directory that have the .sh extension. For each of these files, it checks whether the file is executable. If the file is not executable (! -x $f), it changes the file's permissions to 700 (chmod 700 $f). This means the owner of the file will have read, write, and execute permissions, while group and others have no permissions.

## Unix Shell Programming Examples
### 1
**Store in file a.txt all the processes in the system with details, replacing all sequences of one or more spaces with a single space.**
``` bash
ps -e -f | sed -E "s/ \+/ /g" > a.txt
```

- `ps -e -f`: display all processes
- `sed -E "s/ \+/ /g"`: -E to enable extended regular expressions, it replaces one or more spaces (` \+`) with just a single space , globally (meaning not only the first apparition on each line)
- `> a.txt` : store the output in a file names a.txt

### 2
**Display the owners of the last 7 processes in file a.txt (created in example E1) sorted alphabetically**
``` bash
tail -n 7 a.txt | cut -d" " -f1 | sort 
```

- `tail -n 7 a.txt` : get the last 7 processes from a.txt
- `cut -d" " -f1` : delimitates by space each line, and takes the first field wich represents the user
- `sort` : it sorts

### 3
**Store in file b.txt all the lines in file a.txt (created in example E1), except for the first one**
```bash
tail -n +2 a.txt > b.txt
```

- `tail -n +2 a.txt` : is used to display the content of a.txt starting from the second line. In other words, it will output all lines of a.txt except the first line
  - Normally, when you use `tail -n 2`, it will print the last 2 lines of the file. The `-n` option here is used to specify the number of lines from the end of the file. However, if you specify `+2` instead, it changes the behavior. `tail -n +2` will start output from line 2 of the file, effectively skipping the first line. The `+` symbol here tells `tail` to count from the beginning of the file instead of from the end.
- `> b.txt` : Redirect the output to b. If b.txt already exists, this command will overwrite it; if b.txt does not exist, the command will create it.

### 4
**Display the owners of the processes in file b.txt (created in example E3) removing the duplicates**

p.s: a.txt has all the processes and b.txt has all except the first.

```bash
cut -d" " -f1 b.txt | sort | uniq
```
- `cut -d" " -f1 b.txt`: extract the first field (or column) from each line in the file b.txt. The -d" " option specifies that the delimiter (the character that separates fields) is a space. The -f1 option means "field 1", i.e., the first field (which is the owner)
- `sort | uniq`: to display the unique owners, sort them initially, so uniq can group them.

### 5
**Display the top 3 usernames with the most number of processes in file b.txt (created in example E3), ordered descending by the number of processes.**

To achieve this, you can combine the use of `cut`, `sort`, `uniq`, `sort` again, and finally `head`. Here is the command:

```bash
cut -d" " -f1 b.txt | sort | uniq -c | sort -nr | head -n 3
```

- `cut -d" " -f1 b.txt`: This part extracts the first field (the username) from each line in the `b.txt` file. 
- `sort`: This sorts the output (the usernames).
- `uniq -c`: This counts the occurrence of each username (hence gives the number of processes each user owns). The `-c` option prefixes each line with the number of consecutive occurrences.
- `sort -nr`: This sorts the lines numerically (`-n` option) and in reverse order (`-r` option), so the usernames with the most processes come first.
- `head -n 3`: This displays only the first 3 lines of the output, hence showing the top 3 usernames with the most number of processes.

### 6
**Display all the lines in file b.txt (created in example E3) that do not start with the word "root"**

``` bash
grep -v "^root\>" b.txt
```

- `grep` is a command-line utility for searching plain-text data sets for lines that match a regular expression.
- `-v` is the "invert match" option. When this option is used, `grep` will select only the lines not matching the pattern.
- `^root\>` is the pattern you are searching for.
  - In regular expressions, the caret (`^`) is used to denote the start of a line. So `^root` matches any line that starts with "root".
  -  the `\>` ensures "root" is a complete word and not part of a larger string. Without `\>`, it would also match lines starting with words like "rooting", "rooted", "rootxyz" etc.

### 7
**E7. Display the number (just the number) of lines in file f.txt**

``` bash
wc -l f.txt | cut -d' ' -f1
```

The `wc` (word count) command in Unix/Linux has several options, one of which is `-l` that counts the number of lines in a file. To display only the number of lines (without the filename), you can use a combination of `wc -l` and `cut -d' ' -f1`. Here is the command:
- `wc -l f.txt`: This counts the number of lines in the file `f.txt`. The output will be the number of lines followed by the filename.
- `|`: This is a pipe, which takes the output of the command on its left (i.e., `wc -l f.txt`) and feeds it as the input to the command on its right.
- `cut -d' ' -f1`: This cuts the output at each space (`-d' '`) and takes the first field (`-f1`). Since `wc -l` outputs the number of lines followed by the filename, `cut -d' ' -f1` will select just the number of lines.

Alternative solution:
``` bash
cat f.txt | wc –l
```

### 8
**E8. Display the lines in file f.txt removing those that contain the word "bash"**

To display lines in a file excluding those that contain a specific word, you can use the `grep` command with the `-v` option. The `-v` option inverts the search, meaning `grep` will only display lines that do not match the pattern.
```bash
grep -v 'bash' f.txt
```

Alternative:
``` bash
sed "/bash/d" f.txt
```

To delete lines that contain a certain pattern using `sed`, you would typically use the `/pattern/d` command. The `d` command in `sed` deletes the current pattern space (i.e., the current line if no other sub-commands like `p` have been used), and starts the cycle with the next line. 

Note that this command only affects the output printed to the terminal, and does not actually modify the original file.

### 9
**Display the lines of file f.txt replacing all lowercase vowels with uppercase vowels**

```bash
sed "y/aeiou/AEIOU/" f.txt
```

- `sed`: This is the stream editor command. It can perform a lot of functions on file data, such as appending, inserting, replacing and deleting.
- `'y/aeiou/AEIOU/'`: This tells `sed` to perform a transliteration operation. It replaces all instances of 'a', 'e', 'i', 'o', and 'u' (the characters before the second slash) with 'A', 'E', 'I', 'O', and 'U' respectively (the characters after the second slash). The 'y' stands for "transliterate".
- `f.txt`: This is the file that `sed` operates on.

### 10
**Display the lines of file f.txt duplicating all sequences of two or more vowels**

```bash
sed -E 's/([aeiou]{2,})/\1\1/g' f.txt
```

To achieve this, you can use the `sed` command in combination with Extended Regular Expressions (ERE). The `sed -E` command allows the use of EREs.

Here's a command that duplicates all sequences of two or more vowels in the `f.txt` file:

```bash
sed -E 's/([aeiou]{2,})/\1\1/g' f.txt
```

Here's a breakdown of the command:
- `sed -E`: This invokes the `sed` command with the `-E` flag, which enables the use of Extended Regular Expressions. This allows the use of more powerful pattern-matching capabilities compared to basic regular expressions.
- `'s/([aeiou]{2,})/\1\1/g'`: This is the `sed` command that is being applied to each line of the input file. 
  - `s`: This is the substitute command.
  - `/([aeiou]{2,})/`: This is the pattern to match. It matches a sequence of two or more vowels. The parentheses `()` create a group that can be referred back to in the replacement section.
  - `/\1\1/`: This is the replacement. The `\1` refers back to the group created by the parentheses in the pattern. So, this replacement duplicates the matched sequence of two or more vowels.
  - `g`: This flag applies the substitution globally to each line, replacing all non-overlapping matches in a line, not just the first one.
- `f.txt`: This is the file that `sed` operates on.

### 11
**Display the lines of file f.txt inverting all pairs of letters**

```bash
sed -E "s/([a-zA-Z])([a-zA-Z])/\2\1/g" f.txt
```

Here's the breakdown:
- `sed -E`: As before, this command invokes `sed` with the `-E` flag, which enables Extended Regular Expressions.
- `'s/([a-zA-Z])([a-zA-Z])/\2\1/g'`: This command pattern matches two adjacent alphabetic characters (either lowercase or uppercase). 
  - `s`: This is the substitute command.
  - `/([a-zA-Z])([a-zA-Z])/`: This pattern will match any pair of consecutive letters. `[a-zA-Z]` matches any single alphabetic character, either lowercase or uppercase.
  - `/\2\1/`: This is the replacement. The \2 refers to the second group and \1 refers to the first group created by the parentheses in the pattern. So, this replacement inverts the matched pair of characters.
  - `g`: This flag applies the substitution globally to each line, replacing all non-overlapping matches in a line, not just the first one.
- `f.txt`: This is the input file that `sed` will operate on.

### 12
**Display all the lines in file f.txt that contain sequences of three to five even digits**

``` bash
grep -E "[02468]{3,5}" f.txt
```

The command works as follows:
- `grep`: This is the command-line utility used for searching plain-text data sets for lines that match a regular expression.
- `-E`: This flag interprets the pattern as an Extended Regular Expression (ERE), which allows more advanced pattern matching.
- `[02468]{3,5}`: This is the regular expression that matches sequences of three to five even digits. 
  - `[02468]` matches any single even digit (0, 2, 4, 6, 8).  
  - `{3,5}` specifies that the preceding element must occur at least 3 times but not more than 5 times. 
- `f.txt`: This is the file that `grep` searches in.

### 13
**Delete all files with the txt extension in the current directory, hiding the standard output and standard error of the command.**

```bash
rm *.txt > /dev/null 2>&1
```

This command works as follows:
- `rm *.txt`: This deletes all files with the `.txt` extension in the current directory. `*` is a wildcard character that matches any string, and `.txt` specifies files with the `.txt` extension.
- `> /dev/null`: This redirects the standard output (STDOUT) to `/dev/null`, effectively hiding any messages that would normally be printed to the screen. `/dev/null` is a special file that discards all data written to it.
- `2>&1`: This redirects the standard error (STDERR) to wherever the standard output (STDOUT) is currently going. In this case, because STDOUT is being redirected to `/dev/null`, STDERR will also be redirected to `/dev/null`, hiding any error messages.

### 14
**Display the names of the file and the subdirectory with the longest names in a directory given as command line argument. Consider only files and directories that are not hidden (their name does not start with dot).**

```bash
#!/bin/bash
D=$1
if [ ! -d "$D" ]; then
  exit 1
fi

MAX_FILE_NAME=""
MAX_FILE_LEN=0
MAX_DIR_NAME=""
MAX_DIR_LEN=0

for F in $D/*; do
   L=`echo $F|wc -c`
   if [ -f $F ]; then
     if [ $L -gt $MAX_FILE_LEN ]; then
       MAX_FILE_LEN=$L
       MAX_FILE_NAME=$F
     fi
   elif [ -d $F ]; then
     if [ $L -gt $MAX_DIR_LEN ]; then
       MAX_DIR_LEN=$L
       MAX_DIR_NAME=$F
     fi
   else
     echo Ignoring file $F as it is neither a directory nor a file
   fi
done

echo "File with longest name: $MAX_FILE_NAME"
echo "Directory with longest name: $MAX_DIR_NAME"
```

This script checks all files and subdirectories within a given directory and prints the names of the file and the subdirectory with the longest names.

Here's how it works:
1. The script expects a directory path as a command line argument, which it stores in `D`. It then checks if `D` is a directory with `[ ! -d "$D" ]`. If `D` is not a directory, the script exits with status code 1.
2. It initializes two variables `MAX_FILE_NAME` and `MAX_DIR_NAME` to keep track of the file and directory with the longest names, and `MAX_FILE_LEN` and `MAX_DIR_LEN` to store their lengths. 
3. The `for` loop iterates over all the items in the directory `D`.
4. For each item `F` in the directory, it calculates the length `L` of the item's name using `echo $F|wc -c`.
5. The `if`-`elif`-`else` structure checks if `F` is a file or a directory.
    - If `F` is a file (`[ -f $F ]`) and its name length `L` is greater than the current maximum file name length `MAX_FILE_LEN`, it updates `MAX_FILE_LEN` and `MAX_FILE_NAME` to `L` and `F`, respectively.

    - If `F` is a directory (`[ -d $F ]`) and its name length `L` is greater than the current maximum directory name length `MAX_DIR_LEN`, it updates `MAX_DIR_LEN` and `MAX_DIR_NAME` to `L` and `F`, respectively.

    - If `F` is neither a file nor a directory, it prints a message saying so.
6. After the loop finishes, it prints the file and directory with the longest names.

Remember that this script will only check the direct files and subdirectories of `D`, not any sub-subdirectories or files within those subdirectories. It also ignores hidden files and directories (those starting with a dot).


### 15
**Display the first command line argument that is a positive even number.**

``` bash
#!/bin/bash

while [ -n “$1” ]; do
   if echo $1 | grep -q '^[0-9]*[02468]$'; then
     echo $1
     break
   fi
   shift
done
```

Here is a breakdown of what each line does:

- `while [ -n "$1" ]; do`: This line starts a `while` loop that will continue as long as the first argument (`$1`) is non-empty. `-n` checks for a non-empty string.

- `if echo $1 | grep -q '^[0-9]*[02468]$'; then`: This line checks if the first argument is a non-negative even number. It does this by piping (`|`) the argument to the `grep` command, which searches it for a pattern. The `-q` option tells `grep` to not output anything. The pattern `'^[0-9]*[02468]$'` matches a string if it starts (`^`) with zero or more (`*`) digits (`[0-9]`), and ends (`$`) with an even digit (`[02468]`).

- `echo $1`: If the argument matches the pattern (i.e., it is a non-negative even number), this line prints it.

- `break`: This line immediately exits the `while` loop.

- `fi`: This line ends the `if` statement.

- `shift`: This line "shifts" the arguments, removing the first argument and making the second argument become the first, the third become the second, and so on.

- `done`: This line ends the `while` loop. If the loop hasn't been exited by the `break` statement, it goes back to the start and continues with the new first argument (previously the second argument).

So, this script effectively finds and prints the first non-negative even number from the arguments and stops. If no such number is found, the script doesn't output anything.

### 16
**Read values from the standard input until the sum of all the given natural numbers is strictly greater than 10**

``` bash
#!/bin/bash
SUM=0
while [ $SUM -le 10 ]; do
  read -p "Value: " K
  if echo $K | grep -q "^[0-9]\+$"; then
    SUM=`expr $SUM + $K`
  fi
done
```

The provided script is a simple bash script that reads numbers from the standard input and accumulates their sum until the sum is strictly greater than 10. Here is how it works:

- `SUM=0` initializes the variable `SUM` to 0. This variable is used to keep track of the cumulative sum of the numbers entered.

- `while [ $SUM -le 10 ]; do` starts a loop that continues as long as the value of `SUM` is less than or equal to 10.

- `read -p "Value: " K` prompts the user to input a number. The `-p` option allows a prompt string to be specified, and `K` is the variable that will store the user's input.

- `if echo $K | grep -q "^[0-9]\+$"; then` checks if the user's input (`$K`) is a positive integer. It does this by using `grep` to match `$K` against the regular expression `"^[0-9]\+$"`, which represents one or more (`\+`) digits (`[0-9]`). The `^` and `$` are start and end-of-line anchors respectively, ensuring that the entire input must match the pattern. The `-q` option tells `grep` to not output anything.

- `SUM=`expr $SUM + $K`` adds the user's input to `SUM` if it was a positive integer. `expr` is a command that evaluates its arguments as an expression, in this case, adding `$SUM` and `$K`.

- `fi` ends the `if` statement.

- `done` ends the `while` loop. If `SUM` is still less than or equal to 10, the script goes back to the `read` command and asks the user for another number.

### 17
**Display the names of all the files that contain ASCII text in a directory given as command line argument and all the hierarchy of subdirectories in it**

```bash
D=$1
find $D -type f | while read F; do
  if file $F | grep -q "\<ASCII\>"; then
    echo $F
  fi
done
```

This script is designed to search a directory and its subdirectories for all files containing ASCII text. Here's a breakdown of the script:
- `D=$1`: This line assigns the first command-line argument to the variable `D`. This argument is expected to be the directory you want to search.

- `find $D -type f`: This command is used to find all files (`-type f`) in the directory and its subdirectories specified by `$D`.

- `| while read F; do`: The results from the find command (i.e., the paths of the files) are piped into a `while` loop. For each line of output from `find`, the loop assigns the line to the variable `F` and executes the body of the loop.

- `if file $F | grep -q "\<ASCII\>"; then`: This line checks if the file `$F` is an ASCII text file. The `file` command is used to determine the type of the file. Its output is piped into `grep`, which searches for the term "ASCII". The `-q` option suppresses `grep`'s output; it's used here because we're not interested in the output itself but in `grep`'s return value. The return value of `grep` is 0 (success) if it found a match and 1 (failure) if it didn't. The `\<` and `\>` around "ASCII" are word boundary markers, ensuring that "ASCII" is matched as a whole word and not as part of a larger string.

- `echo $F`: If the `if` condition is true, i.e., the file is an ASCII text file, then the script will print the name of the file to the standard output.

- `fi` and `done` end the `if` statement and `while` loop, respectively.

