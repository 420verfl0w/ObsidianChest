# how do work memset ?
During my 42 cursus i'm very interesting to understand how realy works memset functions

I decided to start by searching on github the source code is directly available you can also have fun to create a small program and compile it statically as below to free yourself from how the memset function is called, this allows you to learn more about the calling conventions already and thus see how the program handles the memset function because YES it's not just a small loop ... we'll see all that keep in mind that the mem functions are very powerful and can be used everywhere 🤓

 ```c
// Compile : gcc -m64 -static main.c -o main
#include <stdio.h>
#include <string.h>

int main(void)
{
	char buf[0x100]; // 256
	memset(buf, 'a', sizeof(buf));
	return (0);
}
```
For can Reverse it im using cutter but we can used any tools to disassembly our code write here.

Again if you don't want to do it with a "Reverse Engineering" method the source code is public and can be found on the internet just here => 
https://github.com/torvalds/linux/blob/master/tools/arch/x86/lib/memset_64.S

I also advise you to install gdb peda or gef for debugging it allows you to understand what each instruction does

when you launch gdb with gef and do the **start** command, you can get something like this :

