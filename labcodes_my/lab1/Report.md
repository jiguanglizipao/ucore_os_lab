# Lab1 erport
### 练习一
1. 操作系统镜像文件ucore.img是如何一步一步生成的？(需要比较详细地解释Makefile中每一条相关命令和命令参数的含义，以及说明命令导致的结果) 
    1. gcc 编译参数为 `CFCFLAGS  := -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc` `CFLAGS  += $(shell $(CC) -fno-stack-protector -E -x c /dev/null >/dev/null 2>&1 && echo -fno-stack-protector)`，其中有几条参数比较特殊:
        - `-fno-builtin` 表示只将带 `__builtin_` 前缀的函数视为内建函数并进行优化。
        - `-ggdb` 生成 gdb 调试信息，与平常所用的 `-g` 不同的是 `-g` 是 "OS native format" 的，可供其他调试器使用; 而 `-ggdb` 生成的是供 gdb 使用的，与平台无关，可跨平台调试，因此这里使用 `-ggdb` 。
        - `-gstabs` 生成 stabs 调试信息，在 gdb 调试信息之上可以附加 "自动变量、全局变量、寄存器变量、静态变量和函数参数" 方便调试。
        - `-m32` 生成 32 位代码，以便在 i386 CPU 上运行。
        - `-nostdinc` 不在标准系统目录中搜索头文件，可以使用自定义的头文件，避免调用标准库。
        - `-fno-stack-protector`  不使用编译器堆栈保护技术。
    2. 调用 ```$(call add_files_cc,$(call listf_cc,$(KSRCDIR)),kernel,$(KCFLAGS)) ``` 执行编译 ```KSRCDIR += kern/init kern/libs kern/debug kern/driver kern/trap kern/mm``` 中的.c文件并生成.o文件
    3. 调用 ```$(V)$(LD) $(LDFLAGS) -T tools/kernel.ld -o $@ $(KOBJS)``` 使用 `tools/kernel.ld` 作为链接脚本链接 `步骤2` 中生成的 `.o` 文件，并输出至 `bin/kernel`。其中 `-T` 参数的作用是"读取链接脚本"，`-m elf_i386` 为在 i386 平台下的链接。
    4. 调用 ```bootfiles = $(call listf_cc,boot)
$(foreach f,$(bootfiles),$(call cc_compile,$(f),$(CC),$(CFLAGS) -Os -nostdinc))``` 编译出 `obj/boot/bootasm.o`和 `obj/boot/bootmain.o`。
    5. 调用 ```$(call add_files_host,tools/sign.c,sign,sign)
$(call create_target_host,sign,sign)``` 生成 `bin/sign`
    6. 调用 ```$(V)$(LD) $(LDFLAGS) -N -e start -Ttext 0x7C00 $^ -o $(call toobj,bootblock)``` 链接 `步骤4` 中生成的 `.o` 文件，并输出到 `obj/bootblock.o`。 `-Ttext 0x7C00` 表示将初始地址重定向为0x7c00。
    7. 调用 ```	@$(OBJCOPY) -S -O binary $(call objfile,bootblock) $(call outfile,bootblock)
	@$(call totarget,sign) $(call outfile,bootblock) $(bootblock)``` 使用 sign 工具对 `obj/bootblock.o` 签名，输出至 `bin/bookblock` 。
    8. `dd if=/dev/zero of=bin/ucore.img count=10000` 创建镜像文件 `bin/ucore.img` 大小10000个512字节的块。
    9. `dd if=bin/bootblock of=bin/ucore.img conv=notrunc` 将 `bootblock` 写入镜像的第一个块。
    10. `dd if=bin/kernel of=bin/ucore.img seek=1 conv=notrunc` 将 `kernel` 写入镜像第二个块起始的空间。

2. 一个被系统认为是符合规范的硬盘主引导扇区的特征是什么?
   - 共 512 字节长，第 511 字节是 `0x55` ，第 512 个字节是 `0xAA` ，即最后两个字节是 `0x55AA` 。




