# **this is just a makeover of [journal.txt](./journal.txt) using chatgpt**

# Kilo Text Editor Journal

**Date:** 05/11/26

## Why I'm Starting This Project

- im starting this project because i am tired of leetcoding and dsa and want to do something else
- also this project will be coded in c because i saw the gate syllabus has c language in it
- i'll be following -- https://viewsourcecode.org/snaptoken/kilo/index.html
- i am also doing it on a linux os, downloading c on windows was a pain in the ass
- c works perfectly on linux checked using a new file "hello.c" i'm gonna keep this file just for the love of the game

## Getting Started with Kilo

- so startin with the book, it says its an update to antirez's kilo editor
- name of the editor is kilo
- make function in C is speacial because its the starting point when i run any program when i return from main, the program exits and passes the returned integer back to the os, ie 0 (return 0) a return value of 0 indicates success
- c is a compiled language so we need to run our program throught a C compiler to turn it into an executeable file, we then run that executeable like we run any other program on the command line
- to compiler our kilo.c file, we run `cc kilo.c -o kilo` on the terminal, -o stands for output and specefies that the output executeable should be named kilo
- to run kilo we type `./kilo` the program wont return an output yet but i can check the exit status by running `echo $?` and i see the perfect 0

## Using a Makefile

- compiling the new code everytime using `cc kilo.c -o kilo` is a lil tiring so to autmate that we use Makefile
- what is Makefile? i jus write a make program, it allows me to ruply run make and it will complile the program for me, i just have to sypply a Makefile to tell it how i wanna compile it

### Makefile (contents)

```makefile
kilo: kilo.c
	$(CC) kilo.c -o kilo -Wall -Wextra -pedantic -std=c99