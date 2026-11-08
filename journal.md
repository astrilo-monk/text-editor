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


## 08/11/26

### 1. Understanding Ctrl-J and Ctrl-M

- working with other Ctrl-(key), we see Ctrl-J and Ctrl-M shows us the same number, 10, when it is expected to give 13, since it's the 13th letter of the alphabet. the same can be seen for "ENTER". this is because of output processing.
- **why is this happening?** - by default the terminal has a flag called `ICRNL` enabled, so when the terminal receives a carriage return (takes the cursor to the beginning of the line) `\r`, it should've returned 13 but `ICRNL` translates it into a new line, 10. so instead of 13 i get a 10.
- **how do we fix it?** - disable `ICRNL`, which comes from `<termios.h>`.
- now fix it for new line `\n` as well, using `OPOST`, which also comes from `<termios.h>`.
- **how this acts** - `\r\n`. after this a new line can be used. `\n` is for new line, `\r` moves the cursor to the start of the line.
- now lets disable some more of these miscellaneous flags.

### 2. Understanding Terminal Flags

**NOTE:** LMFAO I WAS TRYING TO FIND THE BUG WHERE PRESSING COMBINATIONS LIKE Ctrl-Q, Ctrl-S AND STUFF WERE STILL SHOWING BLANK SPACE, BUT THEN I REALIZED I WAS USING `raw.c_lflag` FOR EACH FLAG.

there exist 5 terminal flag categories:

- `iflag` = input flags
- `oflag` = output flags
- `cflag` = control flags
- `lflag` = local flags

### 3. Disabling Miscellaneous Flags

**(copied from book):**

- When `BRKINT` is turned on, a break condition will cause a `SIGINT` signal to be sent to the program, like pressing Ctrl-C.
- `INPCK` enables parity checking, which doesn't seem to apply to modern terminal emulators.
- `ISTRIP` causes the 8th bit of each input byte to be stripped, meaning it will set it to 0. This is probably already turned off.
- `CS8` is not a flag, it is a bit mask with multiple bits, which we set using the bitwise-OR (`|`) operator unlike all the flags we are turning off. It sets the character size (`CS`) to 8 bits per byte. On my system, it's already set that way.

### 4. Adding a Timeout to read()

- a timeout for `read()`. right now it waits indefinitely for input from the keyboard before it returns.
- `VMIN` and `VTIME` comes from `<termios.h>`. they are indexes into the `c_cc` field, which stands for "control characters", an array of bytes that control various terminal settings.
- the `VMIN` value sets the minimum number of bytes of input needed before `read()`. we set that to `0` so `read()` returns as soon as there is any input to be read.
- `VTIME` value sets the maximum amount of time to wait before `read()` returns. it is in tenths of seconds, so we set it to `1`, meaning 1/10 of a second.
- if `read()` times out, it will return `0` because its usual return value is the number of bytes read.

### 5. Error Handling

- error handling now. `enableRawMode()` now gets fully into raw mode.
- now cleaning up the code by adding some error handling.
- first we'll add a `die()` function that prints an error message and exits the program.


## COMPLETED WITH UNIT 2 - Entering raw mode
# NOW STARTING UNIT 3 - Raw input and input 

1. Using Ctrl Key Combinations
- last chapter we saw that Ctrl combined with alphabetic keys maps to bytes 1-26.
- we can use this to detect Ctrl key combinations and map them to different operations in our editor.
- first, we're mapping Ctrl-Q to quit the editor.
Understanding CTRL_KEY(k)
#define CTRL_KEY(k) ((k) & 0x1f)
- the CTRL_KEY macro bitwise-ANDs a character with 0x1f (00011111 in binary).
- this sets the upper 3 bits of the character to 0.
- basically, Ctrl + a key works through bit manipulation. the terminal strips bits 5 and 6 from the key and sends the resulting value.
- we use hexadecimal for bitmasks in C because it is shorter and easier to read than binary.
- the ASCII character set was designed this way, and bit 5 can also be used to switch between lowercase and uppercase.
2. Refactoring Keyboard Input
- now lets clean up the input handling code.
- editorReadKey() is responsible for reading one keypress and returning it.
- editorProcessKeypress() takes that keypress and decides what operation to perform.
- for now, Ctrl-Q is handled inside editorProcessKeypress() and exits the program using exit(0).
- later, editorReadKey() will also handle escape sequences, since keys like the arrow keys send multiple bytes.
- editorProcessKeypress() will eventually handle Ctrl combinations, special keys and printable characters.
- editorReadKey() belongs in the terminal section because it handles low-level terminal input, while editorProcessKeypress() belongs in the new input section because it handles what the keypress actually means.
- this also makes main() much cleaner. the idea is to keep it that way.
3. Clearing the Screen
- now we're going to start rendering the editor's UI after every keypress.
- first, lets just clear the entire screen.
write(STDOUT_FILENO, "\\x1b[2J", 4);
- write() and STDOUT_FILENO come from <unistd.h>.
- we're writing 4 bytes to the terminal. \\x1b is the ESC character (27 in decimal), followed by [2J.
- escape sequences start with ESC followed by [. they tell the terminal to perform different actions like clearing the screen, moving the cursor or changing text formatting.
- J is the Erase In Display command. the argument 2 tells it to clear the entire screen.
- ESC[1J clears from the beginning of the screen up to the cursor, while ESC[0J clears from the cursor to the end. ESC[J is the same as ESC[0J because 0 is the default.
- we'll mostly be using VT100 escape sequences because modern terminal emulators widely support them. another option is the ncurses library, which uses the terminfo database to figure out terminal capabilities and which escape sequences to use.
4. Repositioning the Cursor
- after clearing the screen, the cursor is left at the bottom.
- we need to move it back to the top-left corner before drawing the editor interface.
write(STDOUT_FILENO, "\\x1b[H", 3);
- this uses the H command (Cursor Position).
- H can take two arguments: row and column. for example, ESC[12;40H moves the cursor to row 12 and column 40.
- if we leave the arguments out, both default to 1, so ESC[H moves the cursor to row 1, column 1.
- rows and columns start from 1, not 0.
5. Clearing the Screen on Exit
- now lets clear the screen and reposition the cursor when the program exits.
- we have two places where this needs to happen: inside die() when an error occurs, and when the user presses Ctrl-Q.
- we could use atexit() for this, but then the error message printed by die() would get erased immediately.
- so we're clearing the screen explicitly at both exit points.
6. Drawing Tildes
- now lets draw something on the screen.
- we're going to draw a column of tildes (~) on the left side of the screen, just like Vim.
- every line after the end of the file being edited will have a tilde at the beginning.
void editorDrawRows() {
    int y;
    for (y = 0; y < 24; y++) {
        write(STDOUT_FILENO, "~\\r\\n", 3);
    }
}
- editorDrawRows() will eventually handle drawing each row of the text buffer.
- for now, it just prints a tilde on every row to show that the row is not part of the file.
- we don't know the terminal size yet, so we're hardcoding 24 rows for now.
- after drawing the rows, we move the cursor back to the top-left corner using ESC[H.
7. Introducing Global Editor State
- next goal is to get the terminal size so we know how many rows to draw.
- before that, we're creating a global struct to store the editor's state.
struct editorConfig {
    struct termios orig_termios;
};

struct editorConfig E;
- for now, we're just moving orig_termios into this struct.
- E is our global variable containing the editor state.
- so every occurrence of orig_termios now becomes E.orig_termios.
- we'll keep adding more information to this struct as the project grows.
8. Getting the Terminal Size Using ioctl()
- on most systems, we can get the terminal size using ioctl() with the TIOCGWINSZ request.
- ioctl() stands for Input/Output Control, and TIOCGWINSZ is used to get the terminal window size.
- these come from <sys/ioctl.h>.
int getWindowSize(int *rows, int *cols) {
    struct winsize ws;

    if (ioctl(STDOUT_FILENO, TIOCGWINSZ, &ws) == -1 || ws.ws_col == 0) {
        return -1;
    } else {
        *cols = ws.ws_col;
        *rows = ws.ws_row;
        return 0;
    }
}
- struct winsize stores the terminal dimensions.
- ws.ws_col gives the number of columns and ws.ws_row gives the number of rows.
- if ioctl() fails, it returns -1. we also check if the column count is 0 because that can also be an error.
- we're passing pointers to rows and cols so the function can modify the original variables.
- this is a common way in C to return multiple values while using the normal return value to indicate success or failure.
9. Initializing the Editor
- now lets add screenrows and screencols to our global editor state.
- we'll create an initEditor() function to initialize the fields inside E.
- it calls getWindowSize() and stores the terminal dimensions in E.screenrows and E.screencols.
- if getting the window size fails, we call die("getWindowSize").
- now editorDrawRows() can use E.screenrows instead of the hardcoded value 24.
- the number of tildes displayed now depends on the actual height of the terminal.
10. Window Size: The Fallback Method
- ioctl() isn't guaranteed to work on every system, so we need another way to get the terminal dimensions.
- the fallback approach is to move the cursor to the bottom-right corner and then ask the terminal for its current cursor position.
- from that position, we can figure out the number of rows and columns.
write(STDOUT_FILENO, "\\x1b[999C\\x1b[999B", 12);
- C means Cursor Forward and B means Cursor Down.
- 999 is a large value that should move the cursor to the right and bottom edges of the terminal.
- the terminal prevents the cursor from moving beyond the screen.
- we don't use ESC[999;999H because the documentation doesn't specify what happens when we try to move the cursor off-screen.
- the book temporarily adds 1 || to the if condition to force the fallback branch and test it.
11. Requesting and Reading the Cursor Position
- now that the cursor is at the bottom-right, we need to ask the terminal for its position.
- the n command (Device Status Report) can query the terminal for information. using the argument 6, we ask for the cursor position.
write(STDOUT_FILENO, "\\x1b[6n", 4);
- the terminal responds with another escape sequence, something like ESC[24;80R. this means the cursor is at row 24 and column 80.
- the response comes through standard input, so we read it using read().
- we store the response in a 32-byte buffer and keep reading one character at a time until we get the R character, which marks the end of the response.
- buf[i] = '\\0' adds the null terminator because printf() expects strings to end with one.
- when printing the response, we skip the first character using &buf[1] because that character is ESC and the terminal would interpret it as an escape sequence.
12. Parsing the Cursor Position
- now we have the response in the buffer, so we need to extract the row and column numbers.
- the response looks like ESC[24;80R.
- we'll use sscanf() from <stdio.h> to parse the two integers.
if (buf[0] != '\\x1b' || buf[1] != '[')
    return -1;

if (sscanf(&buf[2], "%d;%d", rows, cols) != 2)
    return -1;

return 0;
- first we check whether the response starts with ESC followed by [.
- then we pass &buf[2] to sscanf(), skipping those first two characters.
- "%d;%d" tells sscanf() to read two integers separated by a semicolon.
- if it successfully reads both values, it returns 2.
- our fallback method for getting the terminal size is now complete.
- after testing it, we remove the temporary 1 || so the program tries ioctl() first and only uses the fallback if ioctl() fails.
13. Back to ioctl()
- the final logic is simple:
  1. first try getting the terminal dimensions using ioctl().
  2. if it fails, move the cursor to the bottom-right corner.
  3. send ESC[6n to request the cursor position.
  4. read the terminal's response.
  5. parse the row and column values using sscanf().
- now the editor can detect the terminal dimensions instead of relying on a hardcoded value of 24.