# 2019july
### I
```c
void f1(){
     int i;
     for(i=0; i<3; i++) {
         if(fork() == 0) {}
         wait(0);
     }
}

void f2(){
     int i, p = 0;
     for(i=0; i<3; i++) {
         if(p == 0) {
             p = fork();
         }
     wait(0);
     }
}
```

**a) Draw the hierarchy diagram of the processes created by executing function f1 in the parent process.**

```scss
Parent (p0)
├── Child 0 (c0)
│   ├── Child 1 (c1)
│   │   └── Child 2 (c2)
│
├── Child 3 (c3)
│   └── Child 4 (c4)
│
└── Child 5 (c5)
```

- p0 makes c0 and then waits for it. (i=0)
- c0 makes c1 and then waits for it (i=1)
- c1 makes c2 and then waits for it (i=2)
- c2 reaches i=3 in for, so it finishes.
- c1 was waiting for c2, which is now finished. It was at i=2, so now i=3. The for loop ends
- c0 was waiting for c1, which is now finished. It was at i=1, so now i=2. c0 makes c3 and then waits for it (i=1).
- c3 makes c4 and then waits for it (i=2)
- c4 reaches i=3 in for, so it finishes.
- c3 was waiting for c4, which is now finished. It was at i=2, so now i=3. The for loop ends
- c0 was waiting for c3, which is now finished. It was at i=1, so now i=2. c0 makes c5 and then waits for it (i=2).
- c5 reaches i=3 in for, so it finishes.
- c0 was waiting for c5, which is now finished. It was at i=2, so now i=3. The for loop ends
- The program is finished.

**b) Draw the hierarchy diagram of the processes created by executing function f2 in the parent process.**

```scss
Parent (p0)
└── Child 0 (c0)
    └── Child 1 (c1)
        └── Child 2 (c2)
```

Fork returns the pid of the child in the parent process, and 0 in the child process.
- p0 makes c0. Since in p0 now `p=1`, it will never fork again.
- c0 makes c1. Since in c0 now `p=1`, it will never fork again.
- c1 makes c2. Since in c1 now `p=1`, it will never fork again.

**c) Rewrite function f1 so that it creates the same number of processes as function f2, but having a different hierarchy. Explain and draw the process hierarchy diagram.**
``` c
void f1(){
     int i;
     for(i=0; i<3; i++) {
         if(fork() == 0) {
            exit(0) // added this 
         }
         wait(0);
     }
}

In this modified version of function f1, the fork() function is called in the first iteration of the loop, and then immediately the child process terminates, ensuring that only the parent process goes on to the next iteration. This process is repeated in each iteration of the loop, creating a new child process each time. This creates the same number of processes as function f2, but in a different hierarchy. 

The diagram becomes:
```scss
Parent (p0)
├── Child 0 (c0)
├── Child 1 (c1)
└── Child 2 (c2)
```

The difference is that in f2 the child processes are nested (creating a chain), whereas in f1_modified all child processes are siblings.

**d) Explain the role of the system call wait.**
This system call is used by a parent process to pause until one of its child processes terminates. wait also allows the parent to receive the child's exit status. Once the parent has waited on a child process, the child fully terminates and is removed from the system instead of becoming a zombie.

### II
Answer the following questions about the UNIX Shell script below, considering that it is executed in a directory with the structure shown on the right, with the following command line arguments: `a d1 apples b d2 pears`.
```bash
1 #!/bin/bash
2 while [ -n "$1" ]; do
3     if [ -d $1 ]; then
4         for F in `find $1 -type f`; do
5             if file $F | grep -q -v text; then
6                 echo "Error: $F"
7                 continue
8             fi
9             sed "s/^$2//" $F > $F.text
10        done
11        shift
12        shift
13    else
14        shift
15    fi
16 done
```

```scss
.
├─ d1 - directory
│├─ a - text file
│├─ b - text file
│└─ x.txt - PDF file renamed .txt
└─ d2 - directory
└─ b - text file
```

Explanation of whole program:
    - `while [ -n "$1" ]; do` : This line starts a `while` loop that will continue as long as the first argument (`$1`) is non-empty. `-n` checks for a non-empty string.
    - `if [ -d $1 ]; then` "Check if the argument is a directory
    - `for F in `find $1 -type f`; do` : For each directory found, it will find all files under it.
    - `if file $F | grep -q -v text; then` : It checks if the file is a text file or not using the file command. 
        - If it's not a text file, it prints an error message and goes to the next file.
    - `sed "s/^$2//" $F > $F.text`: If it's a text file, it performs a sed operation to remove the string that contains at the beginning the next argument from each line (apples, pears..) and creates a new .text file with the modified content.
    - The two `shift` inside the if condition jumps over the next 2arguments. In this example, `a d1 apples b d2 pears`, when we arrive at d1, it will jump over apples and b and reach d2.
    - The `else` at line 13 means that if the first argument is not a directory, go to the next argument without doing anything.
    

**a) What will the script execution print and what are its effects on the structure of files and directories above?**
After the execution of the script, you will see the output "Error: ./d1/x.txt" because it is not a text file. 

When we run the file command on a file, it will determine the file's type based on the file's content and not its extension. The file command uses a "magic" file database that contains patterns that are checked against the contents of files.

In the given directory structure, a and b under d1 are mentioned as text files. This implies that they are plain text files with readable characters. However, x.txt is mentioned as a "PDF file renamed .txt". This means that although the file extension is .txt, the file content is that of a PDF file, which is binary and not text. So, when the script runs the file command on x.txt, the command will identify it as a PDF file (or at least as binary data), not as a text file, despite the .txt extension. That's why the grep command won't find the "text" pattern in x.txt.

The new structure:
```scss
├─ d1 - directory
│├─ a - text file
│├─ b - text file
│├─ a.text - new text file (created by the script)
│├─ b.text - new text file (created by the script)
│└─ x.txt - PDF file renamed .txt
└─ d2 - directory
│├─ b - text file
│└─ b.text - new text file (created by the script)
```

**b) Explain line 5 in detail.**
It checks if the content of the file is text or not.

`if file $F | grep -q -v text; then`:
    - `file`: Determines the file type of a given file or files. The output of the `file` will be the input of the grep command 
    - `grep -q -v text` is checking if the word "text" is NOT in the output of the file command. If the file is not a text file, this condition will be true.

**c) How are the script execution results affected when the quotes on line 9 are replaced with apostrophes?**
`sed "s/^$2//" $F > $F.text` is replaced by `sed 's/^$2//' $F > $F.text` => it will literally look for "$2" instead of replacing the variable with its value.

This is a key difference between single quotes and double quotes in shell scripting. Inside double quotes, $2 would be replaced with the value of the second positional parameter. Inside single quotes, $2 is treated as literal text. So, replacing the double quotes with single quotes in the script would cause it to function differently.

**d) Explain what happens if lines 13 and 14 are removed**
If lines 13 and 14 are removed, the shift command which moves the positional parameters to the left will not be executed. As a result, the script will get stuck in an infinite loop on the first directory argument given, because the first command line argument $1 will never change and because the first argument is a file, not a directory, so it doesn't enter the if.

# 2018july
### I
**The code fragment below is compiled successfully into executable pr. Argument w of function f specifies which FIFO should be used for writing: 0 for a and 1 for b. Considering that all the instructions are executed successfully, that all necessary FIFOs are deleted and created again before each execution, and that pr is te only program accessing those FIFOs, answer the following questions:**

```c
void f(char* a, char* b, int w, char* s) {
    int f[2], r=1-w; char c;
    int(fork() == 0) {
        f[0] = open(a, w==0 ? O_WRONLY : O_RDONLY);
        f[1] = open(b, w==1 ? O_WRONLY : O_RDONLY);
        write(f[w], s, 1);
        read(f[r], &c, 1);
        print("%c\n", c);
        close(f[0]); close(f[1]);
        exit(0);
    }
}

int main(int n, char** a) {
    int i;
    for(i=1; i<n; i+=4) {
        f(a[i], a[i+1], a[i+2][0]-'0', a[i+3];
    }
    for(i=1; i<n; i+=4) {wait(0);}
    return 0;
}
```

Explanation of program:
- It reads 4 arguments at a time. It calls the function f with those arguments, transforming the third argument (which is either 0 or 1) into an int. It also calls correctly wait at the end. Multiple children can be created at the same time.
- Inside the f function:
    - It first creates a child process.
    - In the child process, it opens two FIFOs (named pipes), a and b. If w (write flag) is 0, it opens the first FIFO (a) for writing and the second FIFO (b) for reading, and vice versa if w is 1. r is the read flag , that is why r=w-1
      - w = 1 => f[0] is for reading and f[1] is for writing
      - w = 0 => f[0] is for writing and f[1] is for reading
    - The child process writes a character (s) to the write FIFO, and then reads a character from the read FIFO. This read operation will block until something is written to the FIFO.
    - The character read from the read FIFO is printed to the standard output.

**a) Draw the process hierarchy diagram for an execution where pr receives 4*K command line arguments**

For every four arguments (4*K), a new child process is created, but these child processes are not related in a parent-child manner, they are siblings, all children of the original process.
```scss
Parent (p0)
├── Child 0 (c0)
├── Child 1 (c1)
...
└── Child k (ck)
```

**b) What will the following execution print: `./pr p q 1 x`? Justify your answer.**

- The execution `./pr p q 1 x` will create a single child process.
- It will open the FIFO named p for reading (f[0]) and the FIFO named q for writing (f[1]).
- It will write 'x' to the FIFO q and then wait to read from FIFO p. Because no other process is writing to p, the read will block indefinitely. So, nothing will be printed.

**c) What will the following execution print: `./pr p q 1 x p q 0 y`? Justify your answer.**
- The execution `./pr p q 1 x p q 0 y` will create two child processes.
- The first will open p for reading and q for writing, and the second will open p for writing and q for reading.
- The first child process will write 'x' to q and then wait to read from p. The second child process will write 'y' to p and then wait to read from q.
- Each child process will then read the character written by the other process. So, 'y' will be printed by the first child process, and 'x' will be printed by the second child process.
- The order is not determinable.

**d) Draw a diagram depicting the child processes and their read/write operations on the FIFOs, for the execution in (c).**
![diagrama2019vara](https://github.com/AliceHincu/University/assets/53339016/e0dbc36d-00f0-4589-bc7f-b3ed1596f015)

**e) What will the following execution print: `./pr p q 1 x q p 1 y`? Justify your answer.**
- In the first process: f[0] opens p for reading and f[1] opens q for writing.
- In the second process: f[0] opens q for reading and f[1] opens p for writing.

However, this causes a deadlock. The reason is that both processes are opening their respective read FIFOs and waiting for a process to open these FIFOs in write mode. But, since both processes open the write FIFOs after the read ones, and both are waiting on the read operation, they are stuck in a deadlock, as neither of them can proceed to open the write FIFOs.

When a process opens a FIFO file for reading or writing, it will block until there is another process that opens the file for the opposite operation. 

### II
**Command `sed "s/A/B/"` replaces the first occurrence of a regular expression A on each line with string B, substituting in B any reference \N with the content of the Nth expression between parentheses in A. What are the displayed results of the UNIX Shell script below, when executed in a directory containing C/C++ sources and headers? Explain line 3 in detail: commands, arguments, and pipe.**

```bash
1 for F in *.c *.cpp *.h; do
2     if [ -f $F ]; then
3        grep "#include.*<" $F | sed "s/^.*<\(.*\)>.*$/\1/"
4     fi
5 done | sort
```
(If it used sed -E : s/^.*<(.*)>.*)

This script is meant to find and print all the included system library names in C/C++ source and header files, sorted in lexicographic order
1. The `for` loop goes through each C/C++ source or header file in the current directory (that's what `*.c *.cpp *.h` does).
2. If a file is a regular file (checked by `[ -f $F ]`), the script executes a `grep` command to find lines that contain `#include <library>`. 
3. These lines are then piped (`|`) into the `sed` command, which extracts the library names from the found lines and prints them.
4. This is done for each file and the library names found in each file are printed to the standard output.
5. After all files have been processed by the `for` loop, the entire output (i.e., all the printed library names from all the files) is then piped (`|`) into the `sort` command.
6. The `sort` command sorts this entire output in lexicographic order.

If the `sort` command were inside the `for` loop, it would sort the output for each individual file. But since it's outside the loop, it sorts all the library names together, regardless of which file they came from.

Line 3:
- `grep "#include.*<" $F`: This command searches for lines in the file $F that contain the string "#include" followed by any characters and then a "<" character. This is the common pattern for including system libraries in C/C++ files.
- the lines that have "#include <" string are given as input to the sed command with the help of the pipe.
- inside sed:
    - It matches any line that starts with any characters, followed by a '<', followed by any characters which are captured (due to the parenthesis), followed by a '>', and ending with any characters.
    - The matched line is then replaced by the captured part between '<' and '>', effectively extracting the system library name. For example, for a line like "#include <iostream>", it prints out "iostream"
- The output of each iteration of the loop is a list of system library names included in each C/C++ source or header file.
- The final output of the script is the concatenation of these lists, which is then sorted in lexicographic order due to the sort command at the end of the script.

# 2018 sept
### I
Raspundeti la urmatoarele intrebari, considerand ca toate instructiunile din fragmentul de cod de mai jos se executa cu succes.

```bash
1  int main() {
2      int pfd[2], i;
3      char buffer, c;
4      pipe(pfd);
5      for(i=0; i<3; i++) {
6          if(fork()==0) {
7              while(read(pfd[0], &buffer, 1)>0) {
8                  print("%c\n", buffer);
9              }
10             close(pfd[0]);
11             close(pfd[1]);
12             exit(0);
13         }
14     }
15     close(pfd[0]);
16     for(i=0; i<10; i++) {
17         c = 'a' + i;
18         write(pfd[1], &c, 1);
19     }
20     close(pfd[1]);
21     while(wait(NULL)>0) {}
22     exit(0);
23 }
```

**a) Desenati ierarhia proceselor create, incluzand si procesul parinte**

The hierarchy of the processes created would look like this:
``` scss
Parent
 ├─ Child1
 ├─ Child2
 └─ Child3
```

This is because the parent process creates three child processes in a for loop. These do not create further processes themselves, so we only have a single branch of children.

**b) Ce afiseaza executia programului ?**

When processes are reading from the same pipe, then they're holding file descriptors pointing to the same struct file in the kernel. This means the kernel will determine who gets the data. Only one process will read any given byte.

The program will print the first ten letters of the alphabet in order, from 'a' to 'j'. These are written into the pipe by the parent process and each child will read and print some of the letters, the order is not determined.

**c) Explicati de ce procesele fiu nu se termina**

When processes are reading from the same pipe, then they're holding file descriptors pointing to the same struct file in the kernel. This means the kernel will determine who gets the data. Only one process will read any given byte.

The program will print the first ten letters of the alphabet in order, from 'a' to 'j'. These are written into the pipe by the parent process and each child will read and print some of the letters, the order is not determined.

So in the case of the child processes in your program, the read call in line 7 is waiting for data to become available on the read end of the pipe. But because the write end of the pipe (pfd[1]) is still open in the child processes, read never encounters EOF and therefore never returns 0, so the child processes never exit their reading loop.

**d) Intre ce linii ati muta linia 11 astfel incat procesele fiu sa se termine?**
By moving the close(pfd[1]) call to before the read loop (e.g., right after the fork call), the write end of the pipe would be closed in the child processes, and read would return 0 once all the data has been read from the pipe, allowing the child processes to exit normally.

Line 11 should be moved between lines 4 and 5. This way, the child processes will not have the file descriptor for the writing end of the pipe, and they will exit from the `read` call when the parent process closes the writing end.

(in manual scrie intre 6 si 7)

**e) Explicati linia 21.**
Line 21 is a loop that waits for all child processes to terminate. The `wait(NULL)` function blocks until a child process terminates, returning its PID or -1 if no child processes remain. In this case, the returned value is checked whether it is greater than 0, so the loop will continue to run until all child processes have terminated.

### II 
Scriptul Shell UNIX de mai jos este salvat intr-un fisier numit a.sh . Raspundeti la urmatoarele int5rebari considerand executia comenzii `cat test | ./a.sh`

```bash
#!/bin/bash
read a
x=0
while [ "$a" != "" ]; do
    if echo "$a" | grep -q "^[0-9].*$"
    then
        x=`expr $a + $x`
    fi
    read a
done
echo $x
```

The script reads in lines from the input (which is the content of "test" file), checks if the line starts with a number, and if so, adds it to a running sum stored in the variable x. At the end of the script, it prints the total sum.

**a) Ce se va afisa daca fisierul test contine numere de la 0 la 5 pe linii consecutive**
In this case, the script would read each number one by one and add it to the total sum. The final sum would be 0+1+2+3+4+5 = 15, so the script would output 15.

**b) Ce se va afisa daca fisierul test contine doar o linie cu numere de la 0 la 5, separate prin spatiu?**
The script reads input line by line, so it would read the entire line (including all numbers) as a single input. The script would then try to add this entire line (as a string) to the variable x:
- 0 + "0 1 2 3 4 5"

In Bash and in many other programming languages, trying to perform arithmetic operations on a string that isn't a number will generally lead to an error. If $a is not a number, expr will not be able to perform the addition and will raise an error.

So, the output in this case would be an error message, followed by "0", which is the initial value of the x variable that gets printed at the end of the script

**c) Ce se va afisa, daca in fisierul test de la punctul (a) se adauga o linie cu secventa abc?**
Since "abc" doesn't start with a number, it wouldn't affect the sum. The script would still output 15, same as in point (a). (It doesn't enter the if condition)

**d) Ce se va afisa, daca in fisierul test de la punctul (a) se adauga o linie cu secventa 6bc?**
In this case, since the line starts with a number, it will enter the if condition. since 6bc is a string, it will throw an error 

# 2017 july
### I
Answer the following questions, considering that all the instructions in the code fragment below are executed successfully.
```bash
1  int main() {
2      int p[2], i=0;
3      char c, s[20];
4      pipe(p);
5      if(fork()==0) {
6          close(p[1]);
7          while(read(p[0], &c, sizeof(char))) {
8              if( i < 5 || i > 8) {     
9                  print("%c\n", buffer);
10             }
11             i++;
12         }
13         printf("\n"); close(p[0]);
14         exit(0);
15     }
16     printf("Result: \n");
17     strcpy(s, "exam not passed");
18     close(p[0]);
19     write(p[1], s, strlen(s)*sizeof(char));
20     close(p[1]);
21     wait(NULL);
22     return 0;
23 }
```

**a) Sketch the hierarchy of the created processes, including the parent process.**
```scss
Parent Process
  | 
  Child Process
```

**b) Give each line displayed by the program, along with the process that prints it.**
- The parent process prints "Result: ".
- The child process prints all characters of "exam not passed" except for the characters at the 5th to 8th indices (0-based indexing). These characters correspond to " not". Therefore, the child process will print "exam" and "passed", each character on a separate line.
  
**c) How many characters are read from the pipe?**
The child process reads the entire string "exam not passed" from the pipe, which is 15 characters long (including the space and null terminator).

**d) How will the processes' termination be affected by the removal of line 20?**
The removal of line 20 (close(p[1]);) in the parent process would mean that the write end of the pipe is not closed. Normally, when the write end of the pipe is closed, it signals EOF (end of file) to the reader. If it is not closed, the child process might hang indefinitely on the read call, as it would be waiting for more data to be written to the pipe.

So the child process is blocked at read, ant the parent process is blocked at wait.

**e) How will the processes' termination be affected by the removal of lines 20 and 21?**
When a process exits, all of its file descriptors are closed, including those for pipes. So when the parent process exits, the write end of the pipe (p[1]) will be closed, even if line 20 (which explicitly closes this end of the pipe) is removed.

Because the parent doesn't wait for the child to finish (because line 21 is removed), the parent process could finish and close its end of the pipe before the child process has finished reading from the pipe. However, because the child process is using the read function in a while loop to read from the pipe, once the write end of the pipe is closed (when the parent exits), read will return 0 and the while loop will end, allowing the child process to finish and exit as well.

### II
```bash
f=`find . -type f`
d=`find . -type d`

for x in $f; do
    for y in $d; do
        if [ $x = $y ]; then
            echo "OK"
        fi
    done
done
```

This script searches the current directory and its subdirectories for files and directories, storing the results in variables f and d respectively. Then it checks if any file has the same name as a directory.

**a) How many times will "OK" be displayed? Justify your answer.**
"OK" will be displayed 0 times. This is because a directory (d) and a file (f) cannot have the same full pathname in a Unix-like filesystem. find . -type f and find . -type d return the full pathnames of files and directories respectively, relative to the current directory. So, it's impossible for a file and a directory to have the same full pathname.

**b) What is the value of variable f?**
The variable f will contain a space-separated list of the pathnames of all regular files in the current directory and its subdirectories, relative to the current directory.

**c) What is the value of variable d?**
 The variable d will contain a space-separated list of the pathnames of all directories in the current directory and its subdirectories, relative to the current directory.
 
**d) What is the value of variable x?**
 The value of variable x will be the pathname of the last regular file found by the find command in the current directory and its subdirectories, relative to the current directory. It is assigned during the last iteration of the outer loop.
 
**e) What is the value of variable y?**
The value of variable y will be the pathname of the last directory found by the find command in the current directory and its subdirectories, relative to the current directory. It is assigned during the last iteration of the inner loop.

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

