# System call testing in busybox based minimal Linux.

### 1. Busybox

##### Configuring Busybox

```bash
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- menuconfig
```

Changes

```bash
Settings -> [*] Build static binary( No shared libs)
Shells   -> [*] 'source' and '.' builtins search current directory after $PATH
```

This changes is ok for our purpose. Now save and exit .

##### Compiling Busybox

```bash
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j$(nproc)
```

### 2. C code to test tystem call

Create a file `A.c` and write the following code.

```c
#include<stdio.h>
#include<unistd.h>
int main(){
    syscall(294); // Use user system call number
}
```

Compiling the source

```bash
gcc A.c -o A -static    # For x86_64
aarch64-linux-gnu-gcc A.c -o A -static  # For arm64
```

Now you will get a object named `A`  .

## 3. Building the root filesystem

This section is same as [this](https://github.com/Monir-Ruet/Metroscientific/blob/master/Minimal%20Linux%20With%20Linux%20kernel%20%2C%20Busybox%20%2C%20Qemu%20Arm64%20with%20different%20features..md).

Here just need a little bit modification. We have to just copy the object `A` in this disk image. If everything ok then you are ready to test the system call.

### 4. Testing

Move to the directory where the `A` object is placed in minimal linux. Now run the object.

```bash
./A
```

##### Output

```bash
dmesg
```


