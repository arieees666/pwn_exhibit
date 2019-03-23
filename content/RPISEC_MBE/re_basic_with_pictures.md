### Tools and Basic Reverse Engineering pt. 1

#### The first part of the course starts with a basic introduction of what reverse engineering is and what are the common tools used to reverse engineer (or RE) a binary. As discussed in the lecture slides, below are the tools that can be used for RE along with a short description of what it does which will be essential in solving the crackme challenges provided:
```
Hex Editors/Viewers
  xxd : creates a hex dump of a given file or standard input.
ASCII Readable Hex
  strings : print the strings of printable characters in files.
Files and File formats
  readelf : displays information about ELF files.
  file : tests each argument in an attempt to classify it.
Disassembly
  objdump :  displays information from object files.
```
#### I'll also be using NSA's software RE tool, GHIDRA, in some problems, just to familiarize myself with the tool and be able to use it for ctfs in the upcoming days. Now that we know some of the basic tools and their functionalities, let's head on to the crackmes:

### crackme0x00a
#### As listed on the slides, the challenge can be solvable using ```xxd``` and ```strings```, so we'll do that. First, we use the xxd utility to print a hex dump of the binary and search for interesting things:
```
$ xxd crackme0x00a
  [...]
  00000640: 456e 7465 7220 7061 7373 776f 7264 3a20  Enter password: 
  00000650: 0025 7300 436f 6e67 7261 7473 2100 5772  .%s.Congrats!.Wr
  00000660: 6f6e 6721 0000 0000 011b 033b 3000 0000  ong!.......;0...
  [...]
  00001000: c683 0408 d683 0408 e683 0408 f683 0408  ................
  00001010: 0684 0408 1684 0408 2684 0408 0000 0000  ........&.......
  00001020: 0000 0000 6730 3064 4a30 4221 0000 0000  ....g00dJ0B!....
```
#### We see that the binary has a prompt which asks us to enter a password, and the password is this ```g00dJ0B``` string we found. We can also use strings to easily print out the string.
```
$ strings crackme0x00a
  [...]
  Enter password: 
  Congrats!
  Wrong!
  ;*2$"
  g00dJ0B!
```
#### After getting the information we needed, let's run the binary and input what we have:
```
$ ./cracme0x00a
  Enter a password:
  g00dJ0B!
  Congrats!
```

### crackme0x00b
#### This challenge can be solved using strings, but we need to specify some arguments. Let's take a look:
```
$ strings crackme0x00b
  /lib/ld-linux.so.2
  em!!
  __gmon_start__
  libc.so.6
  _IO_stdin_used
  __isoc99_scanf
  puts
  printf
  wcscmp
  __libc_start_main
```
#### The string ```em!!``` is interesting, so we'll try inputting it to the binary but we get nothing out of it.
```
$ ./crackme0x00b
  Enter password:
  em!!
  Wrong!
```
#### Since we need to specify arguments in order to solve the challenge, let's take a look the the options of the strings utility
```
$ strings --help
  Display printable strings in [file(s)] (stdin by default)
   The options are:
   [...]
   -e --encoding={s,S,b,l,B,L} Select character size and endianness:
      s = 7-bit, S = 8-bit, {b,l} = 16-bit, {B,L} = 32-bit
```
#### We can use the ```-e``` then specify the 32-bit ```L``` option, which is 32-bit little endian.
```
$ strings -e L crackme0x00b
  w0wgreat
```
#### And we have the password for the binary:
```
./crackme0x00b
  Enter a password:
  w0wgreat
  Congrats!
```

### crackme0x01
#### At this point, we'll be getting our hands dirty with a little bit of disassembly. Reverse engineering deals with breaking down a binary to figure out how it works on a low level. And to be able to do this, we need to 'disassemble' the binary in order to know its inner components. For this challenge, we'll be using objdump along with the intel syntax for asm. 
```
$ objdump -M intel -d crackme0x01
  crackme0x01:     file format elf32-i386
  [...]
  080483e4 <main>:
   80483e4:	55                   	push   ebp
   80483e5:	89 e5                	mov    ebp,esp
   80483e7:	83 ec 18             	sub    esp,0x18
   80483ea:	83 e4 f0             	and    esp,0xfffffff0
   80483ed:	b8 00 00 00 00       	mov    eax,0x0
   80483f2:	83 c0 0f             	add    eax,0xf
   80483f5:	83 c0 0f             	add    eax,0xf
   80483f8:	c1 e8 04             	shr    eax,0x4
   80483fb:	c1 e0 04             	shl    eax,0x4
   80483fe:	29 c4                	sub    esp,eax
   8048400:	c7 04 24 28 85 04 08 	mov    DWORD PTR [esp],0x8048528
   8048407:	e8 10 ff ff ff       	call   804831c <printf@plt>
   804840c:	c7 04 24 41 85 04 08 	mov    DWORD PTR [esp],0x8048541
   8048413:	e8 04 ff ff ff       	call   804831c <printf@plt>
   8048418:	8d 45 fc             	lea    eax,[ebp-0x4]
   804841b:	89 44 24 04          	mov    DWORD PTR [esp+0x4],eax
   804841f:	c7 04 24 4c 85 04 08 	mov    DWORD PTR [esp],0x804854c
   8048426:	e8 e1 fe ff ff       	call   804830c <scanf@plt>
   804842b:	81 7d fc 9a 14 00 00 	cmp    DWORD PTR [ebp-0x4],0x149a
   8048432:	74 0e                	je     8048442 <main+0x5e>
   8048434:	c7 04 24 4f 85 04 08 	mov    DWORD PTR [esp],0x804854f
   804843b:	e8 dc fe ff ff       	call   804831c <printf@plt>
   8048440:	eb 0c                	jmp    804844e <main+0x6a>
   8048442:	c7 04 24 62 85 04 08 	mov    DWORD PTR [esp],0x8048562
   8048449:	e8 ce fe ff ff       	call   804831c <printf@plt>
   804844e:	b8 00 00 00 00       	mov    eax,0x0
   8048453:	c9                   	leave  
  [...]
```
#### objdump prints out the disassembly of every function in the binary, but I'll only be showing the disassembly of the main function since it's where the main execution of the binary starts. Given the disassembly, we'll analyze what happens with the instructions. The first two instructions are known as the prologue of the function. It stores the previous base pointer (ebp) and set the base pointer as it was the top of the stack. This means that all the stack contents is saved down the stack, so the function can push/pop in the stack. The third instruction creates space in the stack for the local variables. 
#### Scrolling further down, a buffer of size 0x4 (4 bytes) is initialized and moved into the eax register with the instruction ```lea    eax,[ebp-0x4]```. After which, the scanf function is called which is gets the user input. When we have already inputted something into stdin, it is compared if it is equal to 0x149a (5274) and if it is equal, it jumps to the next instruction at address ```<main+0x6a>```. Now that we have disassembled the binary, it's time we input what we got. 
```
$ ./crackme0x01
  IOLI Crackme Level 0x01
  Password: 5274
  Password OK :)
```
### crackme0x04
#### In the lecture material, the instructors use and explain how to use IDA for disassembly. Sadly, I don't have IDA, but I do have GHIDRA. And this is what I'll be using to disassemble the binary. First we import the binary we'll be disassembling into the program. After it is imported, GHIDRA shows us the import results of the binary. Then we'll examine the functions of the binary using the code browser. 
![import_results](/assets/RPISEC_PWN/RE_basic/crackme_0x04/1_import_result.jpg)
#### The code browser presents us with the decompiled code of the binary. We quickly search for the main function and examine the decompilation of it. 
![decompiled_main](/assets/RPISEC_PWN/RE_basic/crackme_0x04/2_decompiled_main.jpg)
#### As shown in the picture, the two main panes which we need to focus on is the decomiler pane which houses the decompiled code, and the listing pane, which shows the assembly instructions of the code. GHIDRA gives us the option to rename the variables in order for us to understand the program flow a little bit more, so let's rename some of the variables. 
![labeled_main](/assets/RPISEC_PWN/RE_basic/crackme_0x04/3_edited.jpg)
#### We have labeled the buffer variable so we can be able to easily recognize it. Now analyzing the code, we see that the buffer variable is a string buffer of size 120 is initialized, then the function proceeds prints out some stuff and get user input via scanf and stores it into the buffer variable. After which, a call to a ```check``` function is performed where the buffer is supplied in as an argument. Next step we need to do is to analyze and rename the variables in this check function.
![check_decompiled](/assets/RPISEC_PWN/RE_basic/crackme_0x04/4_decompile_check.jpg)
#### Examining the code, we see that some variables are declared and initialized such as the ones we renamed goal and increment which are set to 0. What follows is a while loop which runs forever until stopped, and inside the loop there is a conditional statement that checks wheter the buffer_length variable is less than or equal the increment variable - if it is, then the password we input is wrong. From this alone, we can be able to deduce that the password must be long/greater than the the value of increment.
#### When the code passes the check, the char_from_index is given the value of the character at index increment of the buffer argument. After the initialization of the char_from_index variable, a ```sscanf()``` function is called. sscanf reads formatted data from a string and stores them according to parameter format into the locations given by the additional arguments. On success, the function returns the number of items in the argument list successfully filled. In the code, we see that the arguments supplied into the function call are the address of char_from_index, a %d format specifier which specifies that the argument is an integer, and the address of add_to_goal which the result of the function will be stored. Whatever the function returns, it will be added to the goal variable, and it will be checked if it is equal to 0xf (15). If it is, then we have the correct password, else the increment variable will be incremented and the loop will run again. So what we have to do now is to supply a string, which contains numeric characters, that when run against the loop will add the number at the current increment index to the goal variable and we need to make the value of the goal variable to 0xf (15).
#### We can do this by supplying the number 1 with a string length of fifteen to achieve the results we want. Remember that sscanf reads a character at the current increment index, converts it into an int using the %d specifier and stores it to add_to_goal? We can continuously add 1 to the goal variable and achieve a sum of 15 just by doing this. So let's have a try at it:
```
$ ./crackme0x04
IOLI Crackme Level 0x04
Password: 111111111111111
Password OK!
```
#### And there we have it! The rest of the slides cover the stack, but that is a whole new topic and we'll need to be able to understand it more and in-depth. 
