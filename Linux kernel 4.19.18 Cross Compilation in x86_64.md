# Linux kernel 4.19.18 Cross Compilation in x86_64

Download Kernel

```bash
wget https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.19.18.tar.xz
tar xvf linux-4.19.18.tar.xz
```

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

Now there you will get an error. And now you will get the following file. If you don't run the above command you won't find dtc-lexer.lex.c file. This is must. Edit this file and comment out "YYLTYPE yylloc;"

```bash
vim scripts/dtc/dtc-lexer.lex.c
```

Compile again.

```bash
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j$(nproc)
```

Now this will be successful.
