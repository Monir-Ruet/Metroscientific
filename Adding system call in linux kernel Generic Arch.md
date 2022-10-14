# Adding system call in linux kernel Generic Machines

### 1. Preparation

Make sure you have installed all the minimal requirement packages.This packages are listed [here](https://www.kernel.org/doc/html/latest/process/changes.html)

For ubuntu / debian based

```bash
sudo apt install build-essential libncurses-dev libssl-dev libelf-dev bison flex -y
```

### 2. Download the kernel source:

```bash
wget https://www.kernel.org/pub/linux/kernel/v4.x/linux-4.19.18.tar.xz
tar -xvf linux-4.19.18.tar.xz
cd linux-4.19.18
```

### 3. Clean the Environment

```bash
make clean
make mrproper
```

### 4. Define a new system call sys_hello( )

Create a directory named `Sayhello` [ Add yours]

```bash
mkdir Sayhello
```

Create a file `hello.c` using your favourite text editor

```bash
vi Sayhello/hello.c
```

Write the following code in the editor

```c
#include<linux/kernel.h>
#include<linux/syscalls.h>

// System call without parameter
SYSCALL_DEFINE0(hello){
    printk("Hello from kernel");
    return 0;
}
// System call with parameter
SYSCALL_DEFINE2(sum,int,a,int ,b){
    printk("%d + %d = %d", a, b, a+b);
    return 0;
}
```

Create  a Makefile in this folder

```bash
vi Sayhello/Makefile
```

Put this in the Makefile & save

```makefile
obj-y := hello.o
```

Open kernel Makefile

```bash
vi Makefile
```

Search for core-y and add Sayhello/ at the end of line

```makefile
core-y    += kernel/ certs/ mm/ fs/ ipc/ security/ crypto/ Sayhello/
```

### 5. Add the new system call to the system call table:

The main entry point for your new hello system call will be called `sys_hello()`.The new entry point also needs a corresponding function prototype, in `include/linux/syscalls.h`

Open `include/linux/syscalls.h`

```bash
vi include/linux/syscalls.h
```

Add the system call defination in this header file. Add these line before #endif

```c
asmlinkage long sys_hello(void);
asmlinkage long sys_sum(int , int);
```

Add your new system call to the generic list by adding an entry to the list in `include/uapi/asm-generic/unistd.h`

Open `include/uapi/asm-generic/unistd.h`
```bash
vi include/uapi/asm-generic/unistd.h
```
Put this lines
```c
#define __NR_hello 294
__SYSCALL(__NR_hello, sys_hello)
#define __NR_sum   295
__SYSCALL(__NR_sum  , sys_sum)

// Add this lines before this following two lines
#undef __NR_syscalls
#define __NR_syscalls 296  // Change this number according to yours
```

Open `kernel/sys_ni.c`

```bash
vi kernel/sys_ni.c
```

Add the following line in `kernel/sys_ni.c`

```bash
COND_SYSCALL(hello);
COND_SYSCALL(sum);
```

### 6. Compilation (Arm64)

Configuration

```bash
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- defconfig
```

Making configuration Compatible

```bash
yes "" | make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- oldconfig -j$(nproc)
```

Try to compile

```bash
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j$(nproc)
```

Now there you will get an error..  If the error like this, then  do the following. This errors occurs in linux kernel 4.19.18. 

```bash
/usr/bin/ld: scripts/dtc/dtc-parser.tab.o:(.bss+0x10): multiple definition of `yylloc'; scrip  
ts/dtc/dtc-lexer.lex.o:(.bss+0x0): first defined here
```

If you try to compile first time then you will get a file `dtc-lexer.lex.c`. To get this file you have to try to compile. Now edit this file and comment out "YYLTYPE yylloc;".

```bash
vim scripts/dtc/dtc-lexer.lex.c
```

Compile again.

```bash
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j$(nproc)
```

Now this will be successful.

We will get an Image. The image is located at `/arch/arm64/boot/`.

### 9. Test system call
Go to your home(~) directory using the following commands and create a `userspace.c` file.
```bash
cd ~
gedit userspace.c
```
Write the following code in this file:
```c
#include <stdio.h>
#include <linux/kernel.h>
#include <sys/syscall.h>
#include <unistd.h>
int main()
{
         syscall(294);
         syscall(295,10,20);
         return 0;
}
```
Now, compile and run the program:
```bash
gcc userspace.c -o userspace
./userspace
```
Now, to check the message of your kernel run the following command:
```bash
sudo dmesg
```
This will show a Hello world message and a sum of two number.

This ends the process of Adding a Hello World System Call to your Linux Kernel.
If there needs modification , then any contribution is appreciated. Thanks.
