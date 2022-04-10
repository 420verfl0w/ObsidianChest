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

The main function of code in assembly :
![[Pasted image 20220410164321.png]]

you can see the call function
![[Pasted image 20220410164546.png]]

In fcn.00401110 it represent the stub PLT of the real function
if we click on it, you entered in STUB PLT

The stub PLT is used for procedure linkage table
when binary is dynamicaly linked, it load libX.so in the memory
and jump over PLT stub, The plt stub contains an offset of real function but our binary is staticaly linked so offset is not apear, all function of libc is write directly in the binary, so if you click on it you get layout like this:

![[Pasted image 20220410164943.png]]

When you jump over the function 0x00422270

we can see the other memset function also called "memset_ifunc"

So in my head at this moment
![Horoscope wtf du 2 au 8 décembre | TMV Tours](https://tmv.tmvtours.fr/wp-content/uploads/sites/tours/ca284414d283bd5dcd3a616c3100d357150e8780159dd7771e36051d6baebcf3.jpg)

After fews research i found this header present in glibc library "ifunc-memset.h" => https://github.com/lattera/glibc/blob/master/sysdeps/x86_64/multiarch/ifunc-memset.h

I found this article which explain ifunc (Ifunc is for Indirect function)
Explaination of GNU Indirect Function : https://jasoncc.github.io/gnu_gcc_glibc/gnu-ifunc.html

if you are lazy i resume the Ifunc function family for you but i encourage to read the previous article which explain it better than me

So the Ifunc function, Each “IFUNC” symbol has an associated resolver function. This function is responsible for returning the correct version of the function to utilize. Similar to lazy symbol binding, the resolver is called at the first function invocation in order to determine what symbol should be loaded. Once the resolver has been run, all future function calls will invoke the given function.

The “GNU indirect function” feature requires support in the assembler, linker and loader, which is found in Binutils version 2.20 and later.

The Gnu standard library glibc uses this feature to implement multiple versions of a few memory and string functions, including `memmove`, `memset`, `memcpy`, `strcmp`, `strstr`, etc.

![[Pasted image 20220410170756.png]]

For resume the ifunc function is call at run time to determine which function the programs use, generaly it used for optimization
For example in my IT School memset used __memset_avx2_unaligned_erms

So in my computer memset use __memset_sse2_unaligned

So i explain the CPU x86 extension, we have all differents cpu, each have differents extensions to identify cpu extension you can use assembly instruction cpuid or use builtins in GCC to determine your cpu extension

We use cutter to decompile and etablish an pseudo code of assembly code, we also use debugger GEF to examine this function

![[Pasted image 20220410171751.png]]

If you start gef on the binary, and you enter start command the binary has been launch a this time

When i continue after the stub PLT my memset function is it

![[Pasted image 20220410172154.png]]

So we know which sub-function to use

i place breakpoints to memset_ifunc and observe it 
I get this value, this value represent a 32bits which contains extension list of your cpu (dl_x86_cpu_features) can be obtained via cpuid

cpuid : https://en.wikipedia.org/wiki/CPUID
![[Pasted image 20220410182308.png]]

After this instruction our value is stored in the EDX register

![[Pasted image 20220410182829.png]]

the next instruction set function __memset_erms in rax register

What is ERMS ?

![[Pasted image 20220410183140.png]]

ERMS is for Enhanced rep movsb and stosb

