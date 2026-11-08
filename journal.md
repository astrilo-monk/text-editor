# Kilo Text Editor Journal

## 05/11/26

### Starting the Project

-   im starting this project because i am tired of leetcoding and dsa
    and want to do something else.
-   also this project will be coded in C because i saw the GATE syllabus
    has C language in it.
-   i'll be following:
    https://viewsourcecode.org/snaptoken/kilo/index.html
-   i am also doing it on a linux OS, downloading C on windows was a
    pain in the ass.
-   C works perfectly on linux, checked using a new file `hello.c`. i'm
    gonna keep this file just for the love of the game.
-   so startin with the book, it says its an update to antirez's kilo
    editor.
-   name of the editor is kilo.

### Understanding C and Compilation

-   `main()` function in C is special because it is the starting point
    when i run any program. when i return from `main()`, the program
    exits and passes the returned integer back to the OS, ie `0`
    (`return 0`). a return value of 0 indicates success.
-   C is a compiled language so we need to run our program through a C
    compiler to turn it into an executable file. we then run that
    executable like we run any other program on the command line.
-   to compile our `kilo.c` file, we run `cc kilo.c -o kilo` on the
    terminal. `-o` stands for output and specifies that the output
    executable should be named `kilo`.
-   to run kilo we type `./kilo`. the program wont return an output yet
    but i can check the exit status by running `echo $?` and i see the
    perfect `0`.

### Using a Makefile

-   compiling the new code every time using `cc kilo.c -o kilo` is a lil
    tiring so to automate that we use Makefile.
-   what is Makefile? i jus write a make program, it allows me to
    repeatedly run `make` and it will compile the program for me. i just
    have to supply a Makefile to tell it how i wanna compile it.

**Makefile (contents):**

``` makefile
kilo: kilo.c
    $(CC) kilo.c -o kilo -Wall -Wextra -pedantic -std=c99
```

the first line says that kilo is what we want to build and kilo.c is
required to build it. the second line specifies the command to run in
order to actually build kilo out of kilo.c. the indents are very much
needed here.

**(copied from book):**

-   `$(CC)` is a variable that make expands to cc by default.
-   `-Wall` stands for "all Warnings", and gets the compiler to warn you
    when it sees code in your program that might not technically be
    wrong, but is considered bad or questionable usage of the C
    language, like using variables before initializing them.
-   `-Wextra` and `-pedantic` turn on even more warnings. for each step
    in this tutorial, if your program compiles, it shouldn't produce any
    warnings except for "unused variable" warnings in some cases. if you
    get any other warnings, check to make sure your code exactly matches
    the code in that step.
-   `-std=c99` specifies the exact version of the C language standard
    we're using, which is C99. C99 allows us to declare variables
    anywhere within a function, whereas ANSI C requires all variables to
    be declared at the top of a function or block.

### Running Make

-   to run this Makefile i just need to type `make` on terminal and the
    output is:

``` text
make: 'kilo' is up to date.
```

------------------------------------------------------------------------

## 06/11/26

### 1. Reading User Input

-   now working on the actual app, first user inputs ie the things users
    type on their keyboard.
-   `read()` and `STDIN_FILENO` come from `<unistd.h>`. we are asking
    `read()` to read 1 byte from the standard input into the variable
    `c`, and to keep doing it until there are no more bytes to read.
-   `read()` returns the number of bytes it read and will return `0`
    when it reaches the end of a file.
-   now if i run `./kilo`, the terminal gets hooked up to standard input
    and so keyboard inputs get read into `c`.

### 2. Understanding Canonical Mode

-   but the problem is terminal is default on "canonical mode" or
    "cooked mode". this means on the terminal i can write wtv i want and
    when i hit enter only then my program sees it.
-   ie i can write some text like `"hello 123   world"`. i wanna now
    remove the white spaces and the numbers in it. i have the luxury to
    do this, so i can change it to `"hello world"` and press enter and
    then only my program will be able to see this input.
-   we can't do this in a text editor right? so i will need to switch
    the terminal to "raw mode".
-   there is no easy way to turn off canonical mode, so we do this by
    turning off a lot of flags in terminal, which we will be doing in
    future of this project.
-   alr so updating the code and compiling the program, i can see it
    asks for an input infinitely, which is the perfect behavior of the
    code i just wrote.

### 3. Adding a Quit Input

-   now lets add a `"quit"` input which means updating the loop such
    that `c != (quit button wtv i choose)`.

### 4. Disabling Echo

now i need to turn off echo.

1.  **what is echo?** - the text when typed on terminal, what i see on
    the terminal IS what echo shows to user.
2.  **why turn it off?** - if i press `'a'`, terminal does nothing and
    kilo decides where and when to display `'a'`.
3.  **one liner** - a txt editor should control its own screen rendering
    and not let terminal decide what to display, so we disable echo.

### 5. Breakdown of Function `enableRawMode()`

**1. `struct termios raw;`**

-   creates a structure that stores terminal settings.

**2. `tcgetattr(STDIN_FILENO, &raw);`**

-   fetches the terminal's current settings and stores them inside raw.
-   `STDIN_FILENO` - input from keyboard.
-   `&raw` - where to store the input.

**3. `raw.c_lflag &= ~(ECHO);`**

-   `c_lflag`: contains terminal's local flags.
-   `ECHO`: controls automatic display of typed characters.
-   `~ECHO`: inverts the ECHO bit.
-   `&=`: performs bitwise AND and updates `c_lflag`.

this basically turns OFF echo, while preserving the other local flags.

**4. `tcsetattr(STDIN_FILENO, TCSAFLUSH, &raw);`**

-   applies the modified terminal settings.

arguments:

1.  `STDIN_FILENO`: which terminal to modify.
2.  `TCSAFLUSH`: apply changes after pending output is transmitted and
    discard unread input.
3.  `&raw`: the modified terminal config.

**NOTE:** THIS FUNCTION CURRENTLY DISABLES ONLY ECHO. it does not enable
complete raw mode yet. we need to disable a lot more flags.

### 6. Disabling Raw Mode at Exit

-   now disabling raw mode at exit.
-   this means basically after the user is done with the writing on the
    notepad, the user still wont be able to see the things he's writing
    on the terminal after the program is ran, because the flags get
    changed. so we must restore it so the user can type shit again.
-   we are saving a copy of termios struct in its original state and use
    `tcsetattr()` to apply it to the terminal when the program exits.
-   `atexit()` comes from `<stdlib.h>`, we use it to register our
    `disableRawMode()`.

### 7. Understanding `ICANON`

-   again `ICANON` comes from `<termios.h>`. input flags (the ones in
    the `c_iflag` field) generally start with I like `ICANON` does.
    however, `ICANON` is not an input flag, it's a local flag, in the
    `c_lflag` field, so that's confusing.
-   now the program will exit as soon as i press `q`.

------------------------------------------------------------------------

## 7/6/26

### 1. Reading ASCII Values

-   now lets add a way to see what we're typing using ASCII numbers,
    lets print out each byte that we `read()`.
-   `iscntrl()` comes from `<ctype.h>` and `printf()` comes from
    `<stdio.h>`.
-   `iscntrl()` checks if the typed character is a control character or
    not, ie ASCII 0-31 and 127 are control characters, rest are all
    printable ie 32-126.
-   `printf()` can print multiple representations of a byte. `%d` tells
    it to format the byte as a decimal number (its ASCII code) and `%c`
    tells it to write the letter we typed only.

### 2. Program Outputs

**Output 1: Printable characters**

only printable letters ie 32-126.

**Output 2: Unprintable characters**

unprintable letters = ESC, TAB and ENTER respectively.

**Output 3: Ctrl + Letter Combinations**

combination of Ctrl with other letters A, B, L, W respectively.

**Output 4: Multibyte Unprintable Keys**

1.  arrow keys.
2.  End, Home, Page Down, Page Up.

### 3. Ctrl-S and Ctrl-Q

-   if i press Ctrl-S, program seems to be frozen. i've asked my program
    to stop sending you output, pressing Ctrl-Q to tell it to resume
    sending you output.

### 4. Disabling Ctrl-C and Ctrl-Z Signals

-   also Ctrl-Z suspends the program to the background, and need to run
    the `fg` command to bring it back to the foreground.
-   now lets turn off Ctrl-C and Ctrl-Z signals.
-   by default Ctrl-C is `SIGINT` signal which causes the program to
    terminate and Ctrl-Z sends `SIGTSTP` signal to the current process
    which causes it to suspend.
-   `ISIG` again comes from `<termios.h>`. like `ICANON` it starts with
    I but isn't an input flag.
-   now Ctrl-C can read as 3 byte and Ctrl-Z can be read as 26 byte
    respectively.

### 5. Disabling Ctrl-S and Ctrl-Q

-   now lets disable Ctrl-S and Ctrl-Q. by default these are used for
    software flow control. Ctrl-S stops data from being transmitted to
    the terminal until i press Ctrl-Q. this originates in the days when
    i might want to pause the transmission of data to let a device like
    a printer catch up.
-   `IXON` comes from `<termios.h>`. the I stands for "input flag",
    which it is unlike the other I flags we've seen so far, and XON
    comes from the names of the two control characters that Ctrl-S and
    Ctrl-Q produce: XOFF to pause the transmission and XON to resume
    transmission.
-   now Ctrl-S can read 19 and Ctrl-Q can read as 17 byte.

### 6. Understanding `IEXTEN`

-   again `IEXTEN` comes from `<termios.h>`. it is another flag that
    starts with I but belongs in the `c_lflag` field.
-   Ctrl-V can now read as 22 byte and Ctrl-O as 15 bytes respectively.




