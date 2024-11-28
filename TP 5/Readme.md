# TP 5: Le compilateur ton pire meilleur ami

## I. Programme minimal

🌞 Créer un répertoire de travail dans votre répertoire personnel ( work)
```sh
nathan@ServeurDebian:~$ mkdir work
nathan@ServeurDebian:~$ cd work
```
### 1. Compilation

```sh

nathan@ServeurDebian:~/work$ cat > hello1.c
void _start() {
    const char message[] = "Hello, World!\n";
    asm volatile (
        "mov $4, %%eax\n\t"          // Syscall number for write (4 in 32-bit)
        "mov $1, %%ebx\n\t"          // File descriptor 1 (stdout)
        "mov %[msg], %%ecx\n\t"      // Pointer to the message
        "mov %[len], %%edx\n\t"      // Length of the message
        "int $0x80"                  // Make the syscall (32-bit interrupt)
        :
        : [msg] "r" (message), [len] "r" (14)
        : "eax", "ebx", "ecx", "edx"
    );

    asm volatile (
        "mov $1, %%eax\n\t"          // Syscall number for exit (1 in 32-bit)
        "xor %%ebx, %%ebx\n\t"       // Exit code 0
        "int $0x80"                  // Make the syscall
        :
        :
        : "eax", "ebx"
    );
}
```
```sh
nathan@ServeurDebian:~/work$ gcc -nostdlib -fno-stack-protector -g -m32 -o hello1 hello1.c
```

🌞 Retrouvez à l'aide de readelf l'architecture pour laquelle le programme est compilé

```sh
nathan@ServeurDebian:~/work$ readelf -h hello1
ELF Header:
  Magic:   7f 45 4c 46 01 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF32
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              DYN (Position-Independent Executable file)
  Machine:                           Intel 80386
  Version:                           0x1
  Entry point address:               0x1000
  Start of program headers:          52 (bytes into file)
  Start of section headers:          13304 (bytes into file)
  Flags:                             0x0
  Size of this header:               52 (bytes)
  Size of program headers:           32 (bytes)
  Number of program headers:         11
  Size of section headers:           40 (bytes)
  Number of section headers:         21
  Section header string table index: 20
```

🌞 Afficher la liste des sections contenues dans le programme

```sh
nathan@ServeurDebian:~/work$ readelf -S hello1
There are 21 section headers, starting at offset 0x33f8:

Section Headers:
  [Nr] Name              Type            Addr     Off    Size   ES Flg Lk Inf Al
  [ 0]                   NULL            00000000 000000 000000 00      0   0  0
  [ 1] .interp           PROGBITS        00000194 000194 000013 00   A  0   0  1
  [ 2] .note.gnu.bu[...] NOTE            000001a8 0001a8 000024 00   A  0   0  4
  [ 3] .gnu.hash         GNU_HASH        000001cc 0001cc 000018 04   A  4   0  4
  [ 4] .dynsym           DYNSYM          000001e4 0001e4 000010 10   A  5   1  4
  [ 5] .dynstr           STRTAB          000001f4 0001f4 000001 00   A  0   0  1
  [ 6] .text             PROGBITS        00001000 001000 00005d 00  AX  0   0  1
  [ 7] .eh_frame_hdr     PROGBITS        00002000 002000 00001c 00   A  0   0  4
  [ 8] .eh_frame         PROGBITS        0000201c 00201c 000058 00   A  0   0  4
  [ 9] .dynamic          DYNAMIC         00003f8c 002f8c 000068 08  WA  5   0  4
  [10] .got.plt          PROGBITS        00003ff4 002ff4 00000c 04  WA  0   0  4
  [11] .comment          PROGBITS        00000000 003000 00001e 01  MS  0   0  1
  [12] .debug_aranges    PROGBITS        00000000 00301e 000020 00      0   0  1
  [13] .debug_info       PROGBITS        00000000 00303e 000070 00      0   0  1
  [14] .debug_abbrev     PROGBITS        00000000 0030ae 000062 00      0   0  1
  [15] .debug_line       PROGBITS        00000000 003110 000054 00      0   0  1
  [16] .debug_str        PROGBITS        00000000 003164 000085 01  MS  0   0  1
  [17] .debug_line_str   PROGBITS        00000000 0031e9 00001b 01  MS  0   0  1
  [18] .symtab           SYMTAB          00000000 003204 0000b0 10     19   6  4
  [19] .strtab           STRTAB          00000000 0032b4 00006a 00      0   0  1
  [20] .shstrtab         STRTAB          00000000 00331e 0000d9 00      0   0  1
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), I (info),
  L (link order), O (extra OS processing required), G (group), T (TLS),
  C (compressed), x (unknown), o (OS specific), E (exclude),
  D (mbind), p (processor specific)
```

🌞 Affichez le code assembleur de la section .text à l'aide d'objdump
```sh
nathan@ServeurDebian:~/work$ objdump -M intel -j .text -d hello1

hello1:     file format elf32-i386


Disassembly of section .text:

00001000 <_start>:
    1000:       55                      push   ebp
    1001:       89 e5                   mov    ebp,esp
    1003:       57                      push   edi
    1004:       56                      push   esi
    1005:       53                      push   ebx
    1006:       83 ec 10                sub    esp,0x10
    1009:       e8 4b 00 00 00          call   1059 <__x86.get_pc_thunk.ax>
    100e:       05 e6 2f 00 00          add    eax,0x2fe6
    1013:       c7 45 e5 48 65 6c 6c    mov    DWORD PTR [ebp-0x1b],0x6c6c6548
    101a:       c7 45 e9 6f 2c 20 57    mov    DWORD PTR [ebp-0x17],0x57202c6f
    1021:       c7 45 ed 6f 72 6c 64    mov    DWORD PTR [ebp-0x13],0x646c726f
    1028:       c7 45 f0 64 21 0a 00    mov    DWORD PTR [ebp-0x10],0xa2164
    102f:       8d 75 e5                lea    esi,[ebp-0x1b]
    1032:       bf 0e 00 00 00          mov    edi,0xe
    1037:       b8 04 00 00 00          mov    eax,0x4
    103c:       bb 01 00 00 00          mov    ebx,0x1
    1041:       89 f1                   mov    ecx,esi
    1043:       89 fa                   mov    edx,edi
    1045:       cd 80                   int    0x80
    1047:       b8 01 00 00 00          mov    eax,0x1
    104c:       31 db                   xor    ebx,ebx
    104e:       cd 80                   int    0x80
    1050:       90                      nop
    1051:       83 c4 10                add    esp,0x10
    1054:       5b                      pop    ebx
    1055:       5e                      pop    esi
    1056:       5f                      pop    edi
    1057:       5d                      pop    ebp
    1058:       c3                      ret

00001059 <__x86.get_pc_thunk.ax>:
    1059:       8b 04 24                mov    eax,DWORD PTR [esp]
    105c:       c3                      ret
```
## 2. Librairie et compilation dynamique
### A. Intro lib
```sh
nathan@ServeurDebian:~/work$ cat > hello2.c
#include <stdio.h>

int main()
{
    printf("Hello, World!\n");
}
```

# B. Intro compilation dynamique
➜ On peut maintenant compiler le code
```sh
nathan@ServeurDebian:~/work$ gcc -fno-stack-protector -g -m32 -o hello2 hell
o2.c
```

### C. Tracer les appels à des librairies
🌞 Tracez à l'aide de la commande ldd les librairies appelées par...
- Hello2
```sh
nathan@ServeurDebian:~/work$ ldd hello2
        linux-gate.so.1 (0xf7f05000)
        libc.so.6 => /lib32/libc.so.6 (0xf7ca8000)
        /lib/ld-linux.so.2 (0xf7f07000)
```
- Hello1
```sh
nathan@ServeurDebian:~/work$ ldd hello1
        statically linked
```
- ls
```sh
nathan@ServeurDebian:~/work$ which ls
/usr/bin/ls
nathan@ServeurDebian:~/work$ ldd /bin/ls
        linux-vdso.so.1 (0x00007ffed1be6000)
        libselinux.so.1 => /lib/x86_64-linux-gnu/libselinux.so.1 (0x00007f6829916000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f6829720000)
        libpcre2-8.so.0 => /lib/x86_64-linux-gnu/libpcre2-8.so.0 (0x00007f6829686000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f682997d000)
```
- Firefox
```sh
nathan@ServeurDebian:~/work$ which firefox
/usr/bin/firefox
nathan@ServeurDebian:~/work$ ldd /bin/firefox
        linux-vdso.so.1 (0x00007ffec5fc1000)
        libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007f3b5c54b000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f3b5c355000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f3b5c563000)
```

🌞 Parmi les librairies appelées par hello2, déterminer le type du fichier nommé libc.so.X

```sh
nathan@ServeurDebian:~/work$ file /lib32/libc.so.6
/lib32/libc.so.6: ELF 32-bit LSB shared object, Intel 80386, version 1 (GNU/Linux), dynamically linked, interpreter /lib/ld-linux.so.2, BuildID[sha1]=392018ad420c5b4da4da8c6f4de4014860a15ce8, for GNU/Linux 3.2.0, stripped
```
# 3. Compilation statique
➜ Copiez le fichier hello2.c en un fichier hello3.c, puis compilez-le avec la commande suivante :
```sh
nathan@ServeurDebian:~/work$ cp hello2.c hello3.c
nathan@ServeurDebian:~/work$ gcc -static -fno-stack-protector -g -m32 -o hello3 hello3.c
```
🌞 Affichez le type des fichiers hello2 et hello3
```sh
nathan@ServeurDebian:~/work$ file hello2
hello2: ELF 32-bit LSB pie executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, BuildID[sha1]=b5a74f59d10f4c41d68544ff9266dd3947b20c0e, for GNU/Linux 3.2.0, with debug_info, not stripped
nathan@ServeurDebian:~/work$ file hello3
hello3: ELF 32-bit LSB executable, Intel 80386, version 1 (GNU/Linux), statically linked, BuildID[sha1]=92d994511a13b81120d2f24ae33de51a2af17a61, for GNU/Linux 3.2.0, with debug_info, not stripped
```
🌞 Affichez leurs tailles
```sh
nathan@ServeurDebian:~/work$ du -h hello2
16K     hello2
nathan@ServeurDebian:~/work$ du -h hello3
712K    hello3
```

# 4. Compilation cross-platform

🌞 Affichez l'architecture de votre CPU
```sh
nathan@ServeurDebian:~/work$ cat /proc/cpuinfo
processor       : 0
vendor_id       : GenuineIntel
cpu family      : 6
model           : 154
model name      : 12th Gen Intel(R) Core(TM) i7-12650H
stepping        : 3
microcode       : 0xffffffff
cpu MHz         : 2688.000
cache size      : 24576 KB
physical id     : 0
siblings        : 8
core id         : 0
cpu cores       : 8
apicid          : 0
initial apicid  : 0
fpu             : yes
fpu_exception   : yes
cpuid level     : 22
wp              : yes
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx rdtscp lm constant_tsc rep_good nopl xtopology nonstop_tsc cpuid tsc_known_freq pni pclmulqdq ssse3 cx16 sse4_1 sse4_2 movbe popcnt aes rdrand hypervisor lahf_lm abm 3dnowprefetch ibrs_enhanced fsgsbase bmi1 bmi2 invpcid rdseed adx clflushopt sha_ni arat md_clear flush_l1d arch_capabilities
bugs            : spectre_v1 spectre_v2 spec_store_bypass swapgs retbleed eibrs_pbrsb rfds bhi
bogomips        : 5376.00
clflush size    : 64
cache_alignment : 64
address sizes   : 39 bits physical, 48 bits virtual
power management:

processor       : 1
vendor_id       : GenuineIntel
cpu family      : 6
model           : 154
model name      : 12th Gen Intel(R) Core(TM) i7-12650H
stepping        : 3
microcode       : 0xffffffff
cpu MHz         : 2688.000
cache size      : 24576 KB
physical id     : 0
siblings        : 8
core id         : 1
cpu cores       : 8
apicid          : 1
initial apicid  : 1
fpu             : yes
fpu_exception   : yes
cpuid level     : 22
wp              : yes
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx rdtscp lm constant_tsc rep_good nopl xtopology nonstop_tsc cpuid tsc_known_freq pni pclmulqdq ssse3 cx16 sse4_1 sse4_2 movbe popcnt aes rdrand hypervisor lahf_lm abm 3dnowprefetch ibrs_enhanced fsgsbase bmi1 bmi2 invpcid rdseed adx clflushopt sha_ni arat md_clear flush_l1d arch_capabilities
bugs            : spectre_v1 spectre_v2 spec_store_bypass swapgs retbleed eibrs_pbrsb rfds bhi
bogomips        : 5376.00
clflush size    : 64
cache_alignment : 64
address sizes   : 39 bits physical, 48 bits virtual
power management:

processor       : 2
vendor_id       : GenuineIntel
cpu family      : 6
model           : 154
model name      : 12th Gen Intel(R) Core(TM) i7-12650H
stepping        : 3
microcode       : 0xffffffff
cpu MHz         : 2688.000
cache size      : 24576 KB
physical id     : 0
siblings        : 8
core id         : 2
cpu cores       : 8
apicid          : 2
initial apicid  : 2
fpu             : yes
fpu_exception   : yes
cpuid level     : 22
wp              : yes
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx rdtscp lm constant_tsc rep_good nopl xtopology nonstop_tsc cpuid tsc_known_freq pni pclmulqdq ssse3 cx16 sse4_1 sse4_2 movbe popcnt aes rdrand hypervisor lahf_lm abm 3dnowprefetch ibrs_enhanced fsgsbase bmi1 bmi2 invpcid rdseed adx clflushopt sha_ni arat md_clear flush_l1d arch_capabilities
bugs            : spectre_v1 spectre_v2 spec_store_bypass swapgs retbleed eibrs_pbrsb rfds bhi
bogomips        : 5376.00
clflush size    : 64
cache_alignment : 64
address sizes   : 39 bits physical, 48 bits virtual
power management:

processor       : 3
vendor_id       : GenuineIntel
cpu family      : 6
model           : 154
model name      : 12th Gen Intel(R) Core(TM) i7-12650H
stepping        : 3
microcode       : 0xffffffff
cpu MHz         : 2688.000
cache size      : 24576 KB
physical id     : 0
siblings        : 8
core id         : 3
cpu cores       : 8
apicid          : 3
initial apicid  : 3
fpu             : yes
fpu_exception   : yes
cpuid level     : 22
wp              : yes
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx rdtscp lm constant_tsc rep_good nopl xtopology nonstop_tsc cpuid tsc_known_freq pni pclmulqdq ssse3 cx16 sse4_1 sse4_2 movbe popcnt aes rdrand hypervisor lahf_lm abm 3dnowprefetch ibrs_enhanced fsgsbase bmi1 bmi2 invpcid rdseed adx clflushopt sha_ni arat md_clear flush_l1d arch_capabilities
bugs            : spectre_v1 spectre_v2 spec_store_bypass swapgs retbleed eibrs_pbrsb rfds bhi
bogomips        : 5376.00
clflush size    : 64
cache_alignment : 64
address sizes   : 39 bits physical, 48 bits virtual
power management:

processor       : 4
vendor_id       : GenuineIntel
cpu family      : 6
model           : 154
model name      : 12th Gen Intel(R) Core(TM) i7-12650H
stepping        : 3
microcode       : 0xffffffff
cpu MHz         : 2688.000
cache size      : 24576 KB
physical id     : 0
siblings        : 8
core id         : 4
cpu cores       : 8
apicid          : 4
initial apicid  : 4
fpu             : yes
fpu_exception   : yes
cpuid level     : 22
wp              : yes
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx rdtscp lm constant_tsc rep_good nopl xtopology nonstop_tsc cpuid tsc_known_freq pni pclmulqdq ssse3 cx16 sse4_1 sse4_2 movbe popcnt aes rdrand hypervisor lahf_lm abm 3dnowprefetch ibrs_enhanced fsgsbase bmi1 bmi2 invpcid rdseed adx clflushopt sha_ni arat md_clear flush_l1d arch_capabilities
bugs            : spectre_v1 spectre_v2 spec_store_bypass swapgs retbleed eibrs_pbrsb rfds bhi
bogomips        : 5376.00
clflush size    : 64
cache_alignment : 64
address sizes   : 39 bits physical, 48 bits virtual
power management:

processor       : 5
vendor_id       : GenuineIntel
cpu family      : 6
model           : 154
model name      : 12th Gen Intel(R) Core(TM) i7-12650H
stepping        : 3
microcode       : 0xffffffff
cpu MHz         : 2688.000
cache size      : 24576 KB
physical id     : 0
siblings        : 8
core id         : 5
cpu cores       : 8
apicid          : 5
initial apicid  : 5
fpu             : yes
fpu_exception   : yes
cpuid level     : 22
wp              : yes
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx rdtscp lm constant_tsc rep_good nopl xtopology nonstop_tsc cpuid tsc_known_freq pni pclmulqdq ssse3 cx16 sse4_1 sse4_2 movbe popcnt aes rdrand hypervisor lahf_lm abm 3dnowprefetch ibrs_enhanced fsgsbase bmi1 bmi2 invpcid rdseed adx clflushopt sha_ni arat md_clear flush_l1d arch_capabilities
bugs            : spectre_v1 spectre_v2 spec_store_bypass swapgs retbleed eibrs_pbrsb rfds bhi
bogomips        : 5376.00
clflush size    : 64
cache_alignment : 64
address sizes   : 39 bits physical, 48 bits virtual
power management:

processor       : 6
vendor_id       : GenuineIntel
cpu family      : 6
model           : 154
model name      : 12th Gen Intel(R) Core(TM) i7-12650H
stepping        : 3
microcode       : 0xffffffff
cpu MHz         : 2688.000
cache size      : 24576 KB
physical id     : 0
siblings        : 8
core id         : 6
cpu cores       : 8
apicid          : 6
initial apicid  : 6
fpu             : yes
fpu_exception   : yes
cpuid level     : 22
wp              : yes
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx rdtscp lm constant_tsc rep_good nopl xtopology nonstop_tsc cpuid tsc_known_freq pni pclmulqdq ssse3 cx16 sse4_1 sse4_2 movbe popcnt aes rdrand hypervisor lahf_lm abm 3dnowprefetch ibrs_enhanced fsgsbase bmi1 bmi2 invpcid rdseed adx clflushopt sha_ni arat md_clear flush_l1d arch_capabilities
bugs            : spectre_v1 spectre_v2 spec_store_bypass swapgs retbleed eibrs_pbrsb rfds bhi
bogomips        : 5376.00
clflush size    : 64
cache_alignment : 64
address sizes   : 39 bits physical, 48 bits virtual
power management:

processor       : 7
vendor_id       : GenuineIntel
cpu family      : 6
model           : 154
model name      : 12th Gen Intel(R) Core(TM) i7-12650H
stepping        : 3
microcode       : 0xffffffff
cpu MHz         : 2688.000
cache size      : 24576 KB
physical id     : 0
siblings        : 8
core id         : 7
cpu cores       : 8
apicid          : 7
initial apicid  : 7
fpu             : yes
fpu_exception   : yes
cpuid level     : 22
wp              : yes
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx rdtscp lm constant_tsc rep_good nopl xtopology nonstop_tsc cpuid tsc_known_freq pni pclmulqdq ssse3 cx16 sse4_1 sse4_2 movbe popcnt aes rdrand hypervisor lahf_lm abm 3dnowprefetch ibrs_enhanced fsgsbase bmi1 bmi2 invpcid rdseed adx clflushopt sha_ni arat md_clear flush_l1d arch_capabilities
bugs            : spectre_v1 spectre_v2 spec_store_bypass swapgs retbleed eibrs_pbrsb rfds bhi
bogomips        : 5376.00
clflush size    : 64
cache_alignment : 64
address sizes   : 39 bits physical, 48 bits virtual
power management:
```

🌞 Vérifiez que vous avez la commande x86_64-linux-gnu-gcc
```sh
nathan@ServeurDebian:~/work$ which x86_64-linux-gnu-gcc
/usr/bin/x86_64-linux-gnu-gcc
```

🌞 Compilez votre fichier hello3.c dans un fichier cible nommé hello4 vers une autre architecture que la vôtre
```sh
nathan@ServeurDebian:~/work$ sudo apt-get install gcc-aarch64-linux-gnu
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following packages were automatically installed and are no longer required:
  binutils-arm-linux-gnueabi binutils-i686-linux-gnu
  binutils-mips-linux-gnu binutils-mipsel-linux-gnu
  binutils-powerpc-linux-gnu binutils-powerpc64-linux-gnu
  binutils-riscv64-linux-gnu binutils-sparc64-linux-gnu
  cpp-14-arm-linux-gnueabi cpp-14-i686-linux-gnu cpp-14-mips-linux-gnu
  cpp-14-mipsel-linux-gnu cpp-14-powerpc-linux-gnu
  cpp-14-powerpc64-linux-gnu cpp-14-riscv64-linux-gnu
  cpp-14-sparc64-linux-gnu cpp-arm-linux-gnueabi cpp-i686-linux-gnu
  cpp-mips-linux-gnu cpp-mipsel-linux-gnu cpp-powerpc-linux-gnu
  cpp-powerpc64-linux-gnu cpp-riscv64-linux-gnu cpp-sparc64-linux-gnu
  gcc-14-arm-linux-gnueabi-base gcc-14-cross-base-mipsen
  gcc-14-cross-base-ports gcc-14-i686-linux-gnu-base
  gcc-14-mips-linux-gnu-base gcc-14-mipsel-linux-gnu-base
  gcc-14-powerpc-linux-gnu-base gcc-14-powerpc64-linux-gnu-base
  gcc-14-riscv64-linux-gnu-base gcc-14-sparc64-linux-gnu-base
  libasan8-armel-cross libasan8-i386-cross libasan8-powerpc-cross
  libasan8-ppc64-cross libasan8-riscv64-cross libasan8-sparc64-cross
  libatomic1-armel-cross libatomic1-i386-cross libatomic1-mips-cross
  libatomic1-mipsel-cross libatomic1-powerpc-cross libatomic1-ppc64-cross
  libatomic1-riscv64-cross libatomic1-sparc64-cross libc6-armel-cross
  libc6-dev-armel-cross libc6-dev-i386-cross libc6-dev-mips-cross
  libc6-dev-mipsel-cross libc6-dev-powerpc-cross libc6-dev-ppc64-cross
  libc6-dev-riscv64-cross libc6-dev-sparc64-cross libc6-i386-cross
  libc6-mips-cross libc6-mipsel-cross libc6-powerpc-cross
  libc6-ppc64-cross libc6-riscv64-cross libc6-sparc64-cross
  libgcc-14-dev-armel-cross libgcc-14-dev-i386-cross
  libgcc-14-dev-mips-cross libgcc-14-dev-mipsel-cross
  libgcc-14-dev-powerpc-cross libgcc-14-dev-ppc64-cross
  libgcc-14-dev-riscv64-cross libgcc-14-dev-sparc64-cross
  libgcc-s1-armel-cross libgcc-s1-i386-cross libgcc-s1-mips-cross
  libgcc-s1-mipsel-cross libgcc-s1-powerpc-cross libgcc-s1-ppc64-cross
  libgcc-s1-riscv64-cross libgcc-s1-sparc64-cross libgomp1-armel-cross
  libgomp1-i386-cross libgomp1-mips-cross libgomp1-mipsel-cross
  libgomp1-powerpc-cross libgomp1-ppc64-cross libgomp1-riscv64-cross
  libgomp1-sparc64-cross libitm1-i386-cross libitm1-ppc64-cross
  libitm1-riscv64-cross libitm1-sparc64-cross liblsan0-ppc64-cross
  liblsan0-riscv64-cross libquadmath0-i386-cross libstdc++6-armel-cross
  libstdc++6-i386-cross libstdc++6-powerpc-cross libstdc++6-ppc64-cross
  libstdc++6-riscv64-cross libstdc++6-sparc64-cross libtsan2-ppc64-cross
  libtsan2-riscv64-cross libubsan1-armel-cross libubsan1-i386-cross
  libubsan1-powerpc-cross libubsan1-ppc64-cross libubsan1-riscv64-cross
  libubsan1-sparc64-cross linux-libc-dev-armel-cross
  linux-libc-dev-i386-cross linux-libc-dev-mips-cross
  linux-libc-dev-mipsel-cross linux-libc-dev-powerpc-cross
  linux-libc-dev-ppc64-cross linux-libc-dev-riscv64-cross
  linux-libc-dev-sparc64-cross
Use 'sudo apt autoremove' to remove them.
The following additional packages will be installed:
  gcc-14-aarch64-linux-gnu
Suggested packages:
  gcc-14-doc gcc-14-locales autoconf automake libtool flex bison
  gdb-aarch64-linux-gnu gcc-doc
The following packages will be REMOVED:
  gcc-multilib
The following NEW packages will be installed:
  gcc-14-aarch64-linux-gnu gcc-aarch64-linux-gnu
0 upgraded, 2 newly installed, 1 to remove and 962 not upgraded.
Need to get 0 B/21.6 MB of archives.
After this operation, 78.0 MB of additional disk space will be used.
Do you want to continue? [Y/n] Y
(Reading database ... 150698 files and directories currently installed.)
Removing gcc-multilib (4:14.2.0-1) ...
Selecting previously unselected package gcc-14-aarch64-linux-gnu.
(Reading database ... 150696 files and directories currently installed.)
Preparing to unpack .../gcc-14-aarch64-linux-gnu_14.2.0-6cross1_amd64.deb ...
Unpacking gcc-14-aarch64-linux-gnu (14.2.0-6cross1) ...
Selecting previously unselected package gcc-aarch64-linux-gnu.
Preparing to unpack .../gcc-aarch64-linux-gnu_4%3a14.2.0-1_amd64.deb ...
Unpacking gcc-aarch64-linux-gnu (4:14.2.0-1) ...
Setting up gcc-14-aarch64-linux-gnu (14.2.0-6cross1) ...
Setting up gcc-aarch64-linux-gnu (4:14.2.0-1) ...
Processing triggers for man-db (2.11.2-2) ...
```

```sh
nathan@ServeurDebian:~/work$ aarch64-linux-gnu-gcc -o hello4 hello3.c
```

🌞 Désassemblez hello3 et hello4 à l'aide d'objdump
```sh
nathan@ServeurDebian:~/work$ objdump -d hello3
```sh
 80aac90:       09 d6                   or     %edx,%esi
 80aac92:       84 db                   test   %bl,%bl
 80aac94:       78 ea                   js     80aac80 <.L524+0x4b>
 80aac96:       8d 1c 30                lea    (%eax,%esi,1),%ebx
 80aac99:       e9 2e fd ff ff          jmp    80aa9cc <.L457+0x2>

080aac9e <.L525>:
 80aac9e:       31 ff                   xor    %edi,%edi
 80aaca0:       31 c9                   xor    %ecx,%ecx
 80aaca2:       eb 1c                   jmp    80aacc0 <.L525+0x22>
 80aaca4:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80aacab:       00
 80aacac:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80aacb3:       00
 80aacb4:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80aacbb:       00
 80aacbc:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi
 80aacc0:       0f b6 18                movzbl (%eax),%ebx
 80aacc3:       83 c0 01                add    $0x1,%eax
 80aacc6:       89 da                   mov    %ebx,%edx
 80aacc8:       83 e2 7f                and    $0x7f,%edx
 80aaccb:       d3 e2                   shl    %cl,%edx
 80aaccd:       83 c1 07                add    $0x7,%ecx
 80aacd0:       09 d7                   or     %edx,%edi
 80aacd2:       84 db                   test   %bl,%bl
 80aacd4:       78 ea                   js     80aacc0 <.L525+0x22>
 80aacd6:       31 f6                   xor    %esi,%esi
 80aacd8:       31 c9                   xor    %ecx,%ecx
 80aacda:       8d b6 00 00 00 00       lea    0x0(%esi),%esi
 80aace0:       0f b6 18                movzbl (%eax),%ebx
 80aace3:       83 c0 01                add    $0x1,%eax
 80aace6:       89 da                   mov    %ebx,%edx
 80aace8:       83 e2 7f                and    $0x7f,%edx
 80aaceb:       d3 e2                   shl    %cl,%edx
 80aaced:       83 c1 07                add    $0x7,%ecx
 80aacf0:       09 d6                   or     %edx,%esi
 80aacf2:       84 db                   test   %bl,%bl
 80aacf4:       78 ea                   js     80aace0 <.L525+0x42>
 80aacf6:       83 f9 1f                cmp    $0x1f,%ecx
 80aacf9:       77 09                   ja     80aad04 <.L525+0x66>
 80aacfb:       83 e3 40                and    $0x40,%ebx
 80aacfe:       0f 85 79 04 00 00       jne    80ab17d <.L520+0x62>
 80aad04:       8b 4d 08                mov    0x8(%ebp),%ecx
 80aad07:       0f af 71 74             imul   0x74(%ecx),%esi
 80aad0b:       83 ff 11                cmp    $0x11,%edi
 80aad0e:       0f 87 b6 fc ff ff       ja     80aa9ca <.L457>
 80aad14:       c6 44 39 48 01          movb   $0x1,0x48(%ecx,%edi,1)
 80aad19:       89 34 b9                mov    %esi,(%ecx,%edi,4)
 80aad1c:       e9 a9 fc ff ff          jmp    80aa9ca <.L457>

080aad21 <.L526>:
 80aad21:       31 f6                   xor    %esi,%esi
 80aad23:       31 c9                   xor    %ecx,%ecx
 80aad25:       eb 19                   jmp    80aad40 <.L526+0x1f>
 80aad27:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80aad2e:       00
 80aad2f:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80aad36:       00
 80aad37:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80aad3e:       00
 80aad3f:       90                      nop
 80aad40:       0f b6 18                movzbl (%eax),%ebx
 80aad43:       83 c0 01                add    $0x1,%eax
 80aad46:       89 da                   mov    %ebx,%edx
 80aad48:       83 e2 7f                and    $0x7f,%edx
 80aad4b:       d3 e2                   shl    %cl,%edx
 80aad4d:       83 c1 07                add    $0x7,%ecx
 80aad50:       09 d6                   or     %edx,%esi
 80aad52:       84 db                   test   %bl,%bl
 80aad54:       78 ea                   js     80aad40 <.L526+0x1f>
 80aad56:       8b 7d 08                mov    0x8(%ebp),%edi
 80aad59:       31 c9                   xor    %ecx,%ecx
 80aad5b:       89 77 64                mov    %esi,0x64(%edi)
 80aad5e:       31 f6                   xor    %esi,%esi
 80aad60:       0f b6 18                movzbl (%eax),%ebx
 80aad63:       83 c0 01                add    $0x1,%eax
 80aad66:       89 da                   mov    %ebx,%edx
 80aad68:       83 e2 7f                and    $0x7f,%edx
 80aad6b:       d3 e2                   shl    %cl,%edx
 80aad6d:       83 c1 07                add    $0x7,%ecx
 80aad70:       09 d6                   or     %edx,%esi
 80aad72:       84 db                   test   %bl,%bl
 80aad74:       78 ea                   js     80aad60 <.L526+0x3f>
 80aad76:       83 f9 1f                cmp    $0x1f,%ecx
 80aad79:       77 0e                   ja     80aad89 <.L526+0x68>
 80aad7b:       83 e3 40                and    $0x40,%ebx
 80aad7e:       74 09                   je     80aad89 <.L526+0x68>
 80aad80:       ba ff ff ff ff          mov    $0xffffffff,%edx
 80aad85:       d3 e2                   shl    %cl,%edx
 80aad87:       09 d6                   or     %edx,%esi
 80aad89:       8b 7d 08                mov    0x8(%ebp),%edi
 80aad8c:       89 c3                   mov    %eax,%ebx
 80aad8e:       0f af 77 74             imul   0x74(%edi),%esi
 80aad92:       c6 47 5a 01             movb   $0x1,0x5a(%edi)
 80aad96:       89 77 60                mov    %esi,0x60(%edi)
 80aad99:       e9 2e fc ff ff          jmp    80aa9cc <.L457+0x2>

080aad9e <.L527>:
 80aad9e:       31 f6                   xor    %esi,%esi
 80aada0:       31 c9                   xor    %ecx,%ecx
 80aada2:       eb 1c                   jmp    80aadc0 <.L527+0x22>
 80aada4:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80aadab:       00
 80aadac:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80aadb3:       00
 80aadb4:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80aadbb:       00
 80aadbc:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi
 80aadc0:       0f b6 18                movzbl (%eax),%ebx
 80aadc3:       83 c0 01                add    $0x1,%eax
 80aadc6:       89 da                   mov    %ebx,%edx
 80aadc8:       83 e2 7f                and    $0x7f,%edx
 80aadcb:       d3 e2                   shl    %cl,%edx
 80aadcd:       83 c1 07                add    $0x7,%ecx
 80aadd0:       09 d6                   or     %edx,%esi
 80aadd2:       84 db                   test   %bl,%bl
 80aadd4:       78 ea                   js     80aadc0 <.L527+0x22>
 80aadd6:       83 f9 1f                cmp    $0x1f,%ecx
 80aadd9:       77 0e                   ja     80aade9 <.L527+0x4b>
 80aaddb:       83 e3 40                and    $0x40,%ebx
 80aadde:       74 09                   je     80aade9 <.L527+0x4b>
 80aade0:       ba ff ff ff ff          mov    $0xffffffff,%edx
 80aade5:       d3 e2                   shl    %cl,%edx
 80aade7:       09 d6                   or     %edx,%esi
 80aade9:       8b 7d 08                mov    0x8(%ebp),%edi
 80aadec:       89 c3                   mov    %eax,%ebx
 80aadee:       0f af 77 74             imul   0x74(%edi),%esi
 80aadf2:       89 77 60                mov    %esi,0x60(%edi)
 80aadf5:       e9 d2 fb ff ff          jmp    80aa9cc <.L457+0x2>

080aadfa <.L529>:
 80aadfa:       31 ff                   xor    %edi,%edi
 80aadfc:       31 c9                   xor    %ecx,%ecx
 80aadfe:       66 90                   xchg   %ax,%ax
 80aae00:       0f b6 18                movzbl (%eax),%ebx
 80aae03:       83 c0 01                add    $0x1,%eax
 80aae06:       89 da                   mov    %ebx,%edx
 80aae08:       83 e2 7f                and    $0x7f,%edx
 80aae0b:       d3 e2                   shl    %cl,%edx
 80aae0d:       83 c1 07                add    $0x7,%ecx
 80aae10:       09 d7                   or     %edx,%edi
 80aae12:       84 db                   test   %bl,%bl
 80aae14:       78 ea                   js     80aae00 <.L529+0x6>
 80aae16:       31 f6                   xor    %esi,%esi
 80aae18:       31 c9                   xor    %ecx,%ecx
 80aae1a:       8d b6 00 00 00 00       lea    0x0(%esi),%esi
 80aae20:       0f b6 18                movzbl (%eax),%ebx
 80aae23:       83 c0 01                add    $0x1,%eax
 80aae26:       89 da                   mov    %ebx,%edx
 80aae28:       83 e2 7f                and    $0x7f,%edx
 80aae2b:       d3 e2                   shl    %cl,%edx
 80aae2d:       83 c1 07                add    $0x7,%ecx
 80aae30:       09 d6                   or     %edx,%esi
 80aae32:       84 db                   test   %bl,%bl
 80aae34:       78 ea                   js     80aae20 <.L529+0x26>
 80aae36:       83 f9 1f                cmp    $0x1f,%ecx
 80aae39:       77 09                   ja     80aae44 <.L529+0x4a>
 80aae3b:       83 e3 40                and    $0x40,%ebx
 80aae3e:       0f 85 2b 03 00 00       jne    80ab16f <.L520+0x54>
 80aae44:       8b 4d 08                mov    0x8(%ebp),%ecx
 80aae47:       0f af 71 74             imul   0x74(%ecx),%esi
 80aae4b:       83 ff 11                cmp    $0x11,%edi
 80aae4e:       0f 87 76 fb ff ff       ja     80aa9ca <.L457>
 80aae54:       c6 44 39 48 04          movb   $0x4,0x48(%ecx,%edi,1)
 80aae59:       89 34 b9                mov    %esi,(%ecx,%edi,4)
 80aae5c:       e9 69 fb ff ff          jmp    80aa9ca <.L457>

080aae61 <.L528>:
 80aae61:       31 f6                   xor    %esi,%esi
 80aae63:       31 c9                   xor    %ecx,%ecx
 80aae65:       eb 19                   jmp    80aae80 <.L528+0x1f>
 80aae67:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80aae6e:       00
 80aae6f:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80aae76:       00
 80aae77:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80aae7e:       00
 80aae7f:       90                      nop
 80aae80:       0f b6 18                movzbl (%eax),%ebx
 80aae83:       83 c0 01                add    $0x1,%eax
 80aae86:       89 da                   mov    %ebx,%edx
 80aae88:       83 e2 7f                and    $0x7f,%edx
 80aae8b:       d3 e2                   shl    %cl,%edx
 80aae8d:       83 c1 07                add    $0x7,%ecx
 80aae90:       09 d6                   or     %edx,%esi
 80aae92:       84 db                   test   %bl,%bl
 80aae94:       78 ea                   js     80aae80 <.L528+0x1f>
 80aae96:       31 ff                   xor    %edi,%edi
 80aae98:       31 c9                   xor    %ecx,%ecx
 80aae9a:       8d b6 00 00 00 00       lea    0x0(%esi),%esi
 80aaea0:       0f b6 18                movzbl (%eax),%ebx
 80aaea3:       83 c0 01                add    $0x1,%eax
 80aaea6:       89 da                   mov    %ebx,%edx
 80aaea8:       83 e2 7f                and    $0x7f,%edx
 80aaeab:       d3 e2                   shl    %cl,%edx
 80aaead:       83 c1 07                add    $0x7,%ecx
 80aaeb0:       09 d7                   or     %edx,%edi
 80aaeb2:       84 db                   test   %bl,%bl
 80aaeb4:       78 ea                   js     80aaea0 <.L528+0x3f>
 80aaeb6:       8b 4d 08                mov    0x8(%ebp),%ecx
 80aaeb9:       0f af 79 74             imul   0x74(%ecx),%edi
 80aaebd:       83 fe 11                cmp    $0x11,%esi
 80aaec0:       0f 87 04 fb ff ff       ja     80aa9ca <.L457>
 80aaec6:       c6 44 31 48 04          movb   $0x4,0x48(%ecx,%esi,1)
 80aaecb:       89 3c b1                mov    %edi,(%ecx,%esi,4)
 80aaece:       e9 f7 fa ff ff          jmp    80aa9ca <.L457>

080aaed3 <.L530>:
 80aaed3:       31 f6                   xor    %esi,%esi
 80aaed5:       31 c9                   xor    %ecx,%ecx
 80aaed7:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80aaede:       00
 80aaedf:       90                      nop
 80aaee0:       0f b6 18                movzbl (%eax),%ebx
 80aaee3:       83 c0 01                add    $0x1,%eax
 80aaee6:       89 da                   mov    %ebx,%edx
 80aaee8:       83 e2 7f                and    $0x7f,%edx
 80aaeeb:       d3 e2                   shl    %cl,%edx
 80aaeed:       83 c1 07                add    $0x7,%ecx
 80aaef0:       09 d6                   or     %edx,%esi
 80aaef2:       84 db                   test   %bl,%bl
 80aaef4:       78 ea                   js     80aaee0 <.L530+0xd>
 80aaef6:       83 fe 11                cmp    $0x11,%esi
 80aaef9:       77 0b                   ja     80aaf06 <.L530+0x33>
 80aaefb:       8b 7d 08                mov    0x8(%ebp),%edi
 80aaefe:       c6 44 37 48 05          movb   $0x5,0x48(%edi,%esi,1)
 80aaf03:       89 04 b7                mov    %eax,(%edi,%esi,4)
 80aaf06:       31 f6                   xor    %esi,%esi
 80aaf08:       31 c9                   xor    %ecx,%ecx
 80aaf0a:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80aaf11:       00
 80aaf12:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80aaf19:       00
 80aaf1a:       8d b6 00 00 00 00       lea    0x0(%esi),%esi
 80aaf20:       0f b6 18                movzbl (%eax),%ebx
 80aaf23:       83 c0 01                add    $0x1,%eax
 80aaf26:       89 da                   mov    %ebx,%edx
 80aaf28:       83 e2 7f                and    $0x7f,%edx
 80aaf2b:       d3 e2                   shl    %cl,%edx
 80aaf2d:       83 c1 07                add    $0x7,%ecx
 80aaf30:       09 d6                   or     %edx,%esi
 80aaf32:       84 db                   test   %bl,%bl
 80aaf34:       78 ea                   js     80aaf20 <.L530+0x4d>
 80aaf36:       8d 1c 30                lea    (%eax,%esi,1),%ebx
 80aaf39:       e9 8e fa ff ff          jmp    80aa9cc <.L457+0x2>

080aaf3e <.L531>:
 80aaf3e:       31 f6                   xor    %esi,%esi
 80aaf40:       31 c9                   xor    %ecx,%ecx
 80aaf42:       eb 1c                   jmp    80aaf60 <.L531+0x22>
 80aaf44:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80aaf4b:       00
 80aaf4c:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80aaf53:       00
 80aaf54:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80aaf5b:       00
 80aaf5c:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi
 80aaf60:       0f b6 18                movzbl (%eax),%ebx
 80aaf63:       83 c0 01                add    $0x1,%eax
 80aaf66:       89 da                   mov    %ebx,%edx
 80aaf68:       83 e2 7f                and    $0x7f,%edx
 80aaf6b:       d3 e2                   shl    %cl,%edx
 80aaf6d:       83 c1 07                add    $0x7,%ecx
 80aaf70:       09 d6                   or     %edx,%esi
 80aaf72:       84 db                   test   %bl,%bl
 80aaf74:       78 ea                   js     80aaf60 <.L531+0x22>
 80aaf76:       8b 7d c0                mov    -0x40(%ebp),%edi
 80aaf79:       89 c3                   mov    %eax,%ebx
 80aaf7b:       89 77 68                mov    %esi,0x68(%edi)
 80aaf7e:       e9 49 fa ff ff          jmp    80aa9cc <.L457+0x2>

080aaf83 <.L516>:
 80aaf83:       31 f6                   xor    %esi,%esi
 80aaf85:       31 c9                   xor    %ecx,%ecx
 80aaf87:       eb 17                   jmp    80aafa0 <.L516+0x1d>
 80aaf89:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80aaf90:       00
 80aaf91:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80aaf98:       00
 80aaf99:       8d b4 26 00 00 00 00    lea    0x0(%esi,%eiz,1),%esi
 80aafa0:       0f b6 18                movzbl (%eax),%ebx
 80aafa3:       83 c0 01                add    $0x1,%eax
 80aafa6:       89 da                   mov    %ebx,%edx
 80aafa8:       83 e2 7f                and    $0x7f,%edx
 80aafab:       d3 e2                   shl    %cl,%edx
 80aafad:       83 c1 07                add    $0x7,%ecx
 80aafb0:       09 d6                   or     %edx,%esi
 80aafb2:       84 db                   test   %bl,%bl
 80aafb4:       78 ea                   js     80aafa0 <.L516+0x1d>
 80aafb6:       31 ff                   xor    %edi,%edi
 80aafb8:       31 c9                   xor    %ecx,%ecx
 80aafba:       8d b6 00 00 00 00       lea    0x0(%esi),%esi
 80aafc0:       0f b6 18                movzbl (%eax),%ebx
 80aafc3:       83 c0 01                add    $0x1,%eax
 80aafc6:       89 da                   mov    %ebx,%edx
 80aafc8:       83 e2 7f                and    $0x7f,%edx
 80aafcb:       d3 e2                   shl    %cl,%edx
 80aafcd:       83 c1 07                add    $0x7,%ecx
 80aafd0:       09 d7                   or     %edx,%edi
 80aafd2:       84 db                   test   %bl,%bl
 80aafd4:       78 ea                   js     80aafc0 <.L516+0x3d>
 80aafd6:       8b 4d 08                mov    0x8(%ebp),%ecx
 80aafd9:       0f af 79 74             imul   0x74(%ecx),%edi
 80aafdd:       83 fe 11                cmp    $0x11,%esi
 80aafe0:       0f 87 e4 f9 ff ff       ja     80aa9ca <.L457>
 80aafe6:       c6 44 31 48 01          movb   $0x1,0x48(%ecx,%esi,1)
 80aafeb:       89 3c b1                mov    %edi,(%ecx,%esi,4)
 80aafee:       e9 d7 f9 ff ff          jmp    80aa9ca <.L457>

080aaff3 <.L517>:
 80aaff3:       31 f6                   xor    %esi,%esi
 80aaff5:       31 c9                   xor    %ecx,%ecx
 80aaff7:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80aaffe:       00
 80aafff:       90                      nop
 80ab000:       0f b6 18                movzbl (%eax),%ebx
 80ab003:       83 c0 01                add    $0x1,%eax
 80ab006:       89 da                   mov    %ebx,%edx
 80ab008:       83 e2 7f                and    $0x7f,%edx
 80ab00b:       d3 e2                   shl    %cl,%edx
 80ab00d:       83 c1 07                add    $0x7,%ecx
 80ab010:       09 d6                   or     %edx,%esi
 80ab012:       84 db                   test   %bl,%bl
 80ab014:       78 ea                   js     80ab000 <.L517+0xd>
 80ab016:       83 fe 11                cmp    $0x11,%esi
 80ab019:       0f 87 ab f9 ff ff       ja     80aa9ca <.L457>
 80ab01f:       8b 7d 08                mov    0x8(%ebp),%edi
 80ab022:       c6 44 37 48 00          movb   $0x0,0x48(%edi,%esi,1)
 80ab027:       e9 9e f9 ff ff          jmp    80aa9ca <.L457>

080ab02c <.L522>:
 80ab02c:       31 f6                   xor    %esi,%esi
 80ab02e:       31 c9                   xor    %ecx,%ecx
 80ab030:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ab037:       00
 80ab038:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ab03f:       00
 80ab040:       0f b6 18                movzbl (%eax),%ebx
 80ab043:       83 c0 01                add    $0x1,%eax
 80ab046:       89 da                   mov    %ebx,%edx
 80ab048:       83 e2 7f                and    $0x7f,%edx
 80ab04b:       d3 e2                   shl    %cl,%edx
 80ab04d:       83 c1 07                add    $0x7,%ecx
 80ab050:       09 d6                   or     %edx,%esi
 80ab052:       84 db                   test   %bl,%bl
 80ab054:       78 ea                   js     80ab040 <.L522+0x14>
 80ab056:       8b 7d 08                mov    0x8(%ebp),%edi
 80ab059:       89 c3                   mov    %eax,%ebx
 80ab05b:       89 77 64                mov    %esi,0x64(%edi)
 80ab05e:       c6 47 5a 01             movb   $0x1,0x5a(%edi)
 80ab062:       e9 65 f9 ff ff          jmp    80aa9cc <.L457+0x2>

080ab067 <.L523>:
 80ab067:       31 f6                   xor    %esi,%esi
 80ab069:       31 c9                   xor    %ecx,%ecx
 80ab06b:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ab072:       00
 80ab073:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ab07a:       00
 80ab07b:       2e 8d 74 26 00          lea    %cs:0x0(%esi,%eiz,1),%esi
 80ab080:       0f b6 18                movzbl (%eax),%ebx
 80ab083:       83 c0 01                add    $0x1,%eax
 80ab086:       89 da                   mov    %ebx,%edx
 80ab088:       83 e2 7f                and    $0x7f,%edx
 80ab08b:       d3 e2                   shl    %cl,%edx
 80ab08d:       83 c1 07                add    $0x7,%ecx
 80ab090:       09 d6                   or     %edx,%esi
 80ab092:       84 db                   test   %bl,%bl
 80ab094:       78 ea                   js     80ab080 <.L523+0x19>
 80ab096:       8b 7d 08                mov    0x8(%ebp),%edi
 80ab099:       89 c3                   mov    %eax,%ebx
 80ab09b:       89 77 60                mov    %esi,0x60(%edi)
 80ab09e:       e9 29 f9 ff ff          jmp    80aa9cc <.L457+0x2>

080ab0a3 <.L518>:
 80ab0a3:       31 f6                   xor    %esi,%esi
 80ab0a5:       31 c9                   xor    %ecx,%ecx
 80ab0a7:       eb 17                   jmp    80ab0c0 <.L518+0x1d>
 80ab0a9:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ab0b0:       00
 80ab0b1:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ab0b8:       00
 80ab0b9:       8d b4 26 00 00 00 00    lea    0x0(%esi,%eiz,1),%esi
 80ab0c0:       0f b6 18                movzbl (%eax),%ebx
 80ab0c3:       83 c0 01                add    $0x1,%eax
 80ab0c6:       89 da                   mov    %ebx,%edx
 80ab0c8:       83 e2 7f                and    $0x7f,%edx
 80ab0cb:       d3 e2                   shl    %cl,%edx
 80ab0cd:       83 c1 07                add    $0x7,%ecx
 80ab0d0:       09 d6                   or     %edx,%esi
 80ab0d2:       84 db                   test   %bl,%bl
 80ab0d4:       78 ea                   js     80ab0c0 <.L518+0x1d>
 80ab0d6:       83 fe 11                cmp    $0x11,%esi
 80ab0d9:       0f 87 eb f8 ff ff       ja     80aa9ca <.L457>
 80ab0df:       8b 7d 08                mov    0x8(%ebp),%edi
 80ab0e2:       c6 44 37 48 07          movb   $0x7,0x48(%edi,%esi,1)
 80ab0e7:       e9 de f8 ff ff          jmp    80aa9ca <.L457>

080ab0ec <.L519>:
 80ab0ec:       31 f6                   xor    %esi,%esi
 80ab0ee:       31 c9                   xor    %ecx,%ecx
 80ab0f0:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ab0f7:       00
 80ab0f8:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ab0ff:       00
 80ab100:       0f b6 18                movzbl (%eax),%ebx
 80ab103:       83 c0 01                add    $0x1,%eax
 80ab106:       89 da                   mov    %ebx,%edx
 80ab108:       83 e2 7f                and    $0x7f,%edx
 80ab10b:       d3 e2                   shl    %cl,%edx
 80ab10d:       83 c1 07                add    $0x7,%ecx
 80ab110:       09 d6                   or     %edx,%esi
 80ab112:       84 db                   test   %bl,%bl
 80ab114:       78 ea                   js     80ab100 <.L519+0x14>
 80ab116:       e9 fb fe ff ff          jmp    80ab016 <.L517+0x23>

080ab11b <.L520>:
 80ab11b:       31 f6                   xor    %esi,%esi
 80ab11d:       31 c9                   xor    %ecx,%ecx
 80ab11f:       90                      nop
 80ab120:       0f b6 18                movzbl (%eax),%ebx
 80ab123:       83 c0 01                add    $0x1,%eax
 80ab126:       89 da                   mov    %ebx,%edx
 80ab128:       83 e2 7f                and    $0x7f,%edx
 80ab12b:       d3 e2                   shl    %cl,%edx
 80ab12d:       83 c1 07                add    $0x7,%ecx
 80ab130:       09 d6                   or     %edx,%esi
 80ab132:       84 db                   test   %bl,%bl
 80ab134:       78 ea                   js     80ab120 <.L520+0x5>
 80ab136:       31 ff                   xor    %edi,%edi
 80ab138:       31 c9                   xor    %ecx,%ecx
 80ab13a:       8d b6 00 00 00 00       lea    0x0(%esi),%esi
 80ab140:       0f b6 18                movzbl (%eax),%ebx
 80ab143:       83 c0 01                add    $0x1,%eax
 80ab146:       89 da                   mov    %ebx,%edx
 80ab148:       83 e2 7f                and    $0x7f,%edx
 80ab14b:       d3 e2                   shl    %cl,%edx
 80ab14d:       83 c1 07                add    $0x7,%ecx
 80ab150:       09 d7                   or     %edx,%edi
 80ab152:       84 db                   test   %bl,%bl
 80ab154:       78 ea                   js     80ab140 <.L520+0x25>
 80ab156:       83 fe 11                cmp    $0x11,%esi
 80ab159:       0f 87 6b f8 ff ff       ja     80aa9ca <.L457>
 80ab15f:       8b 4d 08                mov    0x8(%ebp),%ecx
 80ab162:       c6 44 31 48 02          movb   $0x2,0x48(%ecx,%esi,1)
 80ab167:       89 3c b1                mov    %edi,(%ecx,%esi,4)
 80ab16a:       e9 5b f8 ff ff          jmp    80aa9ca <.L457>
 80ab16f:       ba ff ff ff ff          mov    $0xffffffff,%edx
 80ab174:       d3 e2                   shl    %cl,%edx
 80ab176:       09 d6                   or     %edx,%esi
 80ab178:       e9 c7 fc ff ff          jmp    80aae44 <.L529+0x4a>
 80ab17d:       ba ff ff ff ff          mov    $0xffffffff,%edx
 80ab182:       d3 e2                   shl    %cl,%edx
 80ab184:       09 d6                   or     %edx,%esi
 80ab186:       e9 79 fb ff ff          jmp    80aad04 <.L525+0x66>
 80ab18b:       83 c4 80                add    $0xffffff80,%esp
 80ab18e:       8d 54 24 0f             lea    0xf(%esp),%edx
 80ab192:       83 e2 f0                and    $0xfffffff0,%edx
 80ab195:       e9 98 f9 ff ff          jmp    80aab32 <.L476+0x13>
 80ab19a:       8d b6 00 00 00 00       lea    0x0(%esi),%esi

080ab1a0 <uw_frame_state_for>:
 80ab1a0:       55                      push   %ebp
 80ab1a1:       b9 10 00 00 00          mov    $0x10,%ecx
 80ab1a6:       57                      push   %edi
 80ab1a7:       56                      push   %esi
 80ab1a8:       e8 82 0c fa ff          call   804be2f <__x86.get_pc_thunk.si>
 80ab1ad:       81 c6 47 be 03 00       add    $0x3be47,%esi
 80ab1b3:       53                      push   %ebx
 80ab1b4:       83 ec 3c                sub    $0x3c,%esp
 80ab1b7:       89 54 24 04             mov    %edx,0x4(%esp)
 80ab1bb:       83 c2 48                add    $0x48,%edx
 80ab1be:       89 d7                   mov    %edx,%edi
 80ab1c0:       89 44 24 08             mov    %eax,0x8(%esp)
 80ab1c4:       89 74 24 0c             mov    %esi,0xc(%esp)
 80ab1c8:       89 c6                   mov    %eax,%esi
 80ab1ca:       31 c0                   xor    %eax,%eax
 80ab1cc:       f3 ab                   rep stos %eax,%es:(%edi)
 80ab1ce:       c7 46 68 00 00 00 00    movl   $0x0,0x68(%esi)
 80ab1d5:       c7 46 50 00 00 00 00    movl   $0x0,0x50(%esi)
 80ab1dc:       8b 56 4c                mov    0x4c(%esi),%edx
 80ab1df:       85 d2                   test   %edx,%edx
 80ab1e1:       0f 84 e4 03 00 00       je     80ab5cb <.L604+0x4b>
 80ab1e7:       83 ec 08                sub    $0x8,%esp
 80ab1ea:       8b 74 24 10             mov    0x10(%esp),%esi
 80ab1ee:       8d 46 54                lea    0x54(%esi),%eax
 80ab1f1:       50                      push   %eax
 80ab1f2:       8b 46 60                mov    0x60(%esi),%eax
 80ab1f5:       89 44 24 1c             mov    %eax,0x1c(%esp)
 80ab1f9:       c1 e8 1f                shr    $0x1f,%eax
 80ab1fc:       8d 44 02 ff             lea    -0x1(%edx,%eax,1),%eax
 80ab200:       50                      push   %eax
 80ab201:       8b 5c 24 1c             mov    0x1c(%esp),%ebx
 80ab205:       e8 66 3a 00 00          call   80aec70 <_Unwind_Find_FDE>
 80ab20a:       89 44 24 24             mov    %eax,0x24(%esp)
 80ab20e:       83 c4 10                add    $0x10,%esp
 80ab211:       85 c0                   test   %eax,%eax
 80ab213:       8b 44 24 08             mov    0x8(%esp),%eax
 80ab217:       0f 84 83 03 00 00       je     80ab5a0 <.L604+0x20>
 80ab21d:       8b 74 24 04             mov    0x4(%esp),%esi
 80ab221:       8b 40 5c                mov    0x5c(%eax),%eax
 80ab224:       83 ec 0c                sub    $0xc,%esp
 80ab227:       89 46 6c                mov    %eax,0x6c(%esi)
 80ab22a:       8b 74 24 20             mov    0x20(%esp),%esi
 80ab22e:       8d 46 04                lea    0x4(%esi),%eax
 80ab231:       2b 46 04                sub    0x4(%esi),%eax
 80ab234:       89 44 24 24             mov    %eax,0x24(%esp)
 80ab238:       89 c6                   mov    %eax,%esi
 80ab23a:       8d 40 09                lea    0x9(%eax),%eax
 80ab23d:       89 44 24 1c             mov    %eax,0x1c(%esp)
 80ab241:       89 c7                   mov    %eax,%edi
 80ab243:       50                      push   %eax
 80ab244:       8b 5c 24 1c             mov    0x1c(%esp),%ebx
 80ab248:       e8 33 f5 fa ff          call   805a780 <strlen>
 80ab24d:       83 c4 10                add    $0x10,%esp
 80ab250:       80 7e 09 65             cmpb   $0x65,0x9(%esi)
 80ab254:       8d 44 07 01             lea    0x1(%edi,%eax,1),%eax
 80ab258:       0f 84 42 02 00 00       je     80ab4a0 <.L651+0x110>
 80ab25e:       8b 74 24 18             mov    0x18(%esp),%esi
 80ab262:       0f b6 4e 08             movzbl 0x8(%esi),%ecx
 80ab266:       88 4c 24 1f             mov    %cl,0x1f(%esp)
 80ab26a:       80 f9 03                cmp    $0x3,%cl
 80ab26d:       0f 87 5d 04 00 00       ja     80ab6d0 <.L604+0x150>
 80ab273:       31 f6                   xor    %esi,%esi
 80ab275:       31 c9                   xor    %ecx,%ecx
 80ab277:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ab27e:       00
 80ab27f:       90                      nop
 80ab280:       0f b6 18                movzbl (%eax),%ebx
 80ab283:       83 c0 01                add    $0x1,%eax
 80ab286:       89 da                   mov    %ebx,%edx
 80ab288:       83 e2 7f                and    $0x7f,%edx
 80ab28b:       d3 e2                   shl    %cl,%edx
 80ab28d:       83 c1 07                add    $0x7,%ecx
 80ab290:       09 d6                   or     %edx,%esi
 80ab292:       84 db                   test   %bl,%bl
 80ab294:       78 ea                   js     80ab280 <uw_frame_state_for+0xe0>
 80ab296:       8b 5c 24 04             mov    0x4(%esp),%ebx
 80ab29a:       31 c9                   xor    %ecx,%ecx
 80ab29c:       89 73 78                mov    %esi,0x78(%ebx)
 80ab29f:       31 db                   xor    %ebx,%ebx
 80ab2a1:       eb 1d                   jmp    80ab2c0 <uw_frame_state_for+0x120>
 80ab2a3:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ab2aa:       00
 80ab2ab:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ab2b2:       00
 80ab2b3:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ab2ba:       00
 80ab2bb:       2e 8d 74 26 00          lea    %cs:0x0(%esi,%eiz,1),%esi
 80ab2c0:       89 c5                   mov    %eax,%ebp
 80ab2c2:       0f b6 10                movzbl (%eax),%edx
 80ab2c5:       83 c0 01                add    $0x1,%eax
 80ab2c8:       89 d7                   mov    %edx,%edi
 80ab2ca:       83 e7 7f                and    $0x7f,%edi
 80ab2cd:       d3 e7                   shl    %cl,%edi
 80ab2cf:       83 c1 07                add    $0x7,%ecx
 80ab2d2:       09 fb                   or     %edi,%ebx
 80ab2d4:       84 d2                   test   %dl,%dl
 80ab2d6:       78 e8                   js     80ab2c0 <uw_frame_state_for+0x120>
 80ab2d8:       89 ef                   mov    %ebp,%edi
 80ab2da:       89 d5                   mov    %edx,%ebp
 80ab2dc:       89 fa                   mov    %edi,%edx
 80ab2de:       83 f9 1f                cmp    $0x1f,%ecx
 80ab2e1:       77 0e                   ja     80ab2f1 <uw_frame_state_for+0x151>
 80ab2e3:       83 e5 40                and    $0x40,%ebp
 80ab2e6:       74 09                   je     80ab2f1 <uw_frame_state_for+0x151>
 80ab2e8:       bf ff ff ff ff          mov    $0xffffffff,%edi
 80ab2ed:       d3 e7                   shl    %cl,%edi
 80ab2ef:       09 fb                   or     %edi,%ebx
 80ab2f1:       8b 7c 24 04             mov    0x4(%esp),%edi
 80ab2f5:       31 c9                   xor    %ecx,%ecx
 80ab2f7:       89 dd                   mov    %ebx,%ebp
 80ab2f9:       89 5f 74                mov    %ebx,0x74(%edi)
 80ab2fc:       31 ff                   xor    %edi,%edi
 80ab2fe:       80 7c 24 1f 01          cmpb   $0x1,0x1f(%esp)
 80ab303:       0f 84 c7 01 00 00       je     80ab4d0 <.L651+0x140>
 80ab309:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ab310:       00
 80ab311:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ab318:       00
 80ab319:       8d b4 26 00 00 00 00    lea    0x0(%esi,%eiz,1),%esi
 80ab320:       0f b6 18                movzbl (%eax),%ebx
 80ab323:       83 c0 01                add    $0x1,%eax
 80ab326:       89 da                   mov    %ebx,%edx
 80ab328:       83 e2 7f                and    $0x7f,%edx
 80ab32b:       d3 e2                   shl    %cl,%edx
 80ab32d:       83 c1 07                add    $0x7,%ecx
 80ab330:       09 d7                   or     %edx,%edi
 80ab332:       84 db                   test   %bl,%bl
 80ab334:       78 ea                   js     80ab320 <uw_frame_state_for+0x180>
 80ab336:       89 eb                   mov    %ebp,%ebx
 80ab338:       8b 4c 24 04             mov    0x4(%esp),%ecx
 80ab33c:       31 ed                   xor    %ebp,%ebp
 80ab33e:       89 79 7c                mov    %edi,0x7c(%ecx)
 80ab341:       8b 7c 24 10             mov    0x10(%esp),%edi
 80ab345:       c6 81 81 00 00 00 ff    movb   $0xff,0x81(%ecx)
 80ab34c:       0f b6 17                movzbl (%edi),%edx
 80ab34f:       80 fa 7a                cmp    $0x7a,%dl
 80ab352:       0f 84 88 02 00 00       je     80ab5e0 <.L604+0x60>
 80ab358:       84 d2                   test   %dl,%dl
 80ab35a:       0f 84 72 04 00 00       je     80ab7d2 <.L604+0x252>
 80ab360:       8b 7c 24 10             mov    0x10(%esp),%edi
 80ab364:       89 5c 24 10             mov    %ebx,0x10(%esp)
 80ab368:       83 c7 01                add    $0x1,%edi
 80ab36b:       83 ea 42                sub    $0x42,%edx
 80ab36e:       80 fa 11                cmp    $0x11,%dl
 80ab371:       77 1d                   ja     80ab390 <.L651>
 80ab373:       8b 4c 24 0c             mov    0xc(%esp),%ecx
 80ab377:       0f b6 d2                movzbl %dl,%edx
 80ab37a:       8b 9c 91 c8 6c fe ff    mov    -0x19338(%ecx,%edx,4),%ebx
 80ab381:       01 cb                   add    %ecx,%ebx
 80ab383:       3e ff e3                notrack jmp *%ebx
 80ab386:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ab38d:       00
 80ab38e:       66 90                   xchg   %ax,%ax

080ab390 <.L651>:
 80ab390:       8b 5c 24 10             mov    0x10(%esp),%ebx
 80ab394:       85 ed                   test   %ebp,%ebp
 80ab396:       0f 84 39 03 00 00       je     80ab6d5 <.L604+0x155>
 80ab39c:       8b 7c 24 18             mov    0x18(%esp),%edi
 80ab3a0:       8b 07                   mov    (%edi),%eax
 80ab3a2:       8d 54 07 04             lea    0x4(%edi,%eax,1),%edx
 80ab3a6:       83 fb fc                cmp    $0xfffffffc,%ebx
 80ab3a9:       75 09                   jne    80ab3b4 <.L651+0x24>
 80ab3ab:       83 fe 01                cmp    $0x1,%esi
 80ab3ae:       0f 84 57 03 00 00       je     80ab70b <.L604+0x18b>
 80ab3b4:       83 ec 0c                sub    $0xc,%esp
 80ab3b7:       89 e8                   mov    %ebp,%eax
 80ab3b9:       ff 74 24 10             push   0x10(%esp)
 80ab3bd:       8b 4c 24 18             mov    0x18(%esp),%ecx
 80ab3c1:       e8 5a f5 ff ff          call   80aa920 <execute_cfa_program_generic>
 80ab3c6:       83 c4 10                add    $0x10,%esp
 80ab3c9:       8b 44 24 04             mov    0x4(%esp),%eax
 80ab3cd:       0f b6 80 80 00 00 00    movzbl 0x80(%eax),%eax
 80ab3d4:       3c ff                   cmp    $0xff,%al
 80ab3d6:       0f 84 0c 03 00 00       je     80ab6e8 <.L604+0x168>
 80ab3dc:       83 e0 07                and    $0x7,%eax
 80ab3df:       3c 02                   cmp    $0x2,%al
 80ab3e1:       0f 84 d9 02 00 00       je     80ab6c0 <.L604+0x140>
 80ab3e7:       0f 86 83 02 00 00       jbe    80ab670 <.L604+0xf0>
 80ab3ed:       3c 03                   cmp    $0x3,%al
 80ab3ef:       0f 84 83 02 00 00       je     80ab678 <.L604+0xf8>
 80ab3f5:       3c 04                   cmp    $0x4,%al
 80ab3f7:       0f 85 c2 e1 f9 ff       jne    80495bf <uw_frame_state_for.cold>
 80ab3fd:       bb 18 00 00 00          mov    $0x18,%ebx
 80ab402:       8b 44 24 14             mov    0x14(%esp),%eax
 80ab406:       01 c3                   add    %eax,%ebx
 80ab408:       8b 44 24 04             mov    0x4(%esp),%eax
 80ab40c:       80 b8 82 00 00 00 00    cmpb   $0x0,0x82(%eax)
 80ab413:       0f b6 90 81 00 00 00    movzbl 0x81(%eax),%edx
 80ab41a:       0f 84 68 02 00 00       je     80ab688 <.L604+0x108>
 80ab420:       31 f6                   xor    %esi,%esi
 80ab422:       31 c9                   xor    %ecx,%ecx
 80ab424:       89 d7                   mov    %edx,%edi
 80ab426:       eb 18                   jmp    80ab440 <.L651+0xb0>
 80ab428:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ab42f:       00
 80ab430:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ab437:       00
 80ab438:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ab43f:       00
 80ab440:       0f b6 13                movzbl (%ebx),%edx
 80ab443:       83 c3 01                add    $0x1,%ebx
 80ab446:       89 d0                   mov    %edx,%eax
 80ab448:       83 e0 7f                and    $0x7f,%eax
 80ab44b:       d3 e0                   shl    %cl,%eax
 80ab44d:       83 c1 07                add    $0x7,%ecx
 80ab450:       09 c6                   or     %eax,%esi
 80ab452:       84 d2                   test   %dl,%dl
 80ab454:       78 ea                   js     80ab440 <.L651+0xb0>
 80ab456:       89 fa                   mov    %edi,%edx
 80ab458:       80 fa ff                cmp    $0xff,%dl
 80ab45b:       0f 85 e7 01 00 00       jne    80ab648 <.L604+0xc8>
 80ab461:       01 f3                   add    %esi,%ebx
 80ab463:       8b 74 24 14             mov    0x14(%esp),%esi
 80ab467:       8b 06                   mov    (%esi),%eax
 80ab469:       8d 54 06 04             lea    0x4(%esi,%eax,1),%edx
 80ab46d:       8b 44 24 04             mov    0x4(%esp),%eax
 80ab471:       83 78 74 fc             cmpl   $0xfffffffc,0x74(%eax)
 80ab475:       75 0a                   jne    80ab481 <.L651+0xf1>
 80ab477:       83 78 78 01             cmpl   $0x1,0x78(%eax)
 80ab47b:       0f 84 71 02 00 00       je     80ab6f2 <.L604+0x172>
 80ab481:       83 ec 0c                sub    $0xc,%esp
 80ab484:       89 d8                   mov    %ebx,%eax
 80ab486:       ff 74 24 10             push   0x10(%esp)
 80ab48a:       8b 4c 24 18             mov    0x18(%esp),%ecx
 80ab48e:       e8 8d f4 ff ff          call   80aa920 <execute_cfa_program_generic>
 80ab493:       83 c4 10                add    $0x10,%esp
 80ab496:       31 c0                   xor    %eax,%eax
 80ab498:       83 c4 3c                add    $0x3c,%esp
 80ab49b:       5b                      pop    %ebx
 80ab49c:       5e                      pop    %esi
 80ab49d:       5f                      pop    %edi
 80ab49e:       5d                      pop    %ebp
 80ab49f:       c3                      ret
 80ab4a0:       8b 74 24 18             mov    0x18(%esp),%esi
 80ab4a4:       80 7e 0a 68             cmpb   $0x68,0xa(%esi)
 80ab4a8:       0f 85 b0 fd ff ff       jne    80ab25e <uw_frame_state_for+0xbe>
 80ab4ae:       8b 10                   mov    (%eax),%edx
 80ab4b0:       8b 74 24 04             mov    0x4(%esp),%esi
 80ab4b4:       83 c0 04                add    $0x4,%eax
 80ab4b7:       89 96 84 00 00 00       mov    %edx,0x84(%esi)
 80ab4bd:       8b 74 24 18             mov    0x18(%esp),%esi
 80ab4c1:       83 c6 0b                add    $0xb,%esi
 80ab4c4:       89 74 24 10             mov    %esi,0x10(%esp)
 80ab4c8:       e9 91 fd ff ff          jmp    80ab25e <uw_frame_state_for+0xbe>
 80ab4cd:       8d 76 00                lea    0x0(%esi),%esi
 80ab4d0:       0f b6 38                movzbl (%eax),%edi
 80ab4d3:       8d 42 02                lea    0x2(%edx),%eax
 80ab4d6:       e9 5d fe ff ff          jmp    80ab338 <uw_frame_state_for+0x198>
 80ab4db:       2e 8d 74 26 00          lea    %cs:0x0(%esi,%eiz,1),%esi

080ab4e0 <.L608>:
 80ab4e0:       0f b6 10                movzbl (%eax),%edx
 80ab4e3:       8b 5c 24 04             mov    0x4(%esp),%ebx
 80ab4e7:       83 c0 01                add    $0x1,%eax
 80ab4ea:       88 93 81 00 00 00       mov    %dl,0x81(%ebx)

080ab4f0 <.L610>:
 80ab4f0:       0f b6 17                movzbl (%edi),%edx
 80ab4f3:       83 c7 01                add    $0x1,%edi
 80ab4f6:       84 d2                   test   %dl,%dl
 80ab4f8:       0f 85 6d fe ff ff       jne    80ab36b <uw_frame_state_for+0x1cb>
 80ab4fe:       8b 5c 24 10             mov    0x10(%esp),%ebx
 80ab502:       85 ed                   test   %ebp,%ebp
 80ab504:       0f 85 92 fe ff ff       jne    80ab39c <.L651+0xc>
 80ab50a:       89 c5                   mov    %eax,%ebp
 80ab50c:       e9 83 fe ff ff          jmp    80ab394 <.L651+0x4>
 80ab511:       8d b4 26 00 00 00 00    lea    0x0(%esi,%eiz,1),%esi

080ab518 <.L606>:
 80ab518:       0f b6 10                movzbl (%eax),%edx
 80ab51b:       8b 5c 24 04             mov    0x4(%esp),%ebx
 80ab51f:       83 c0 01                add    $0x1,%eax
 80ab522:       83 c7 01                add    $0x1,%edi
 80ab525:       88 93 80 00 00 00       mov    %dl,0x80(%ebx)
 80ab52b:       0f b6 57 ff             movzbl -0x1(%edi),%edx
 80ab52f:       84 d2                   test   %dl,%dl
 80ab531:       0f 85 34 fe ff ff       jne    80ab36b <uw_frame_state_for+0x1cb>
 80ab537:       eb c5                   jmp    80ab4fe <.L610+0xe>
 80ab539:       8d b4 26 00 00 00 00    lea    0x0(%esi,%eiz,1),%esi

080ab540 <.L607>:
 80ab540:       83 ec 0c                sub    $0xc,%esp
 80ab543:       0f b6 10                movzbl (%eax),%edx
 80ab546:       8d 48 01                lea    0x1(%eax),%ecx
 80ab549:       83 c7 01                add    $0x1,%edi
 80ab54c:       8d 44 24 38             lea    0x38(%esp),%eax
 80ab550:       50                      push   %eax
 80ab551:       8b 44 24 18             mov    0x18(%esp),%eax
 80ab555:       e8 86 de ff ff          call   80a93e0 <read_encoded_value>
 80ab55a:       8b 54 24 3c             mov    0x3c(%esp),%edx
 80ab55e:       8b 5c 24 14             mov    0x14(%esp),%ebx
 80ab562:       83 c4 10                add    $0x10,%esp
 80ab565:       89 53 70                mov    %edx,0x70(%ebx)
 80ab568:       0f b6 57 ff             movzbl -0x1(%edi),%edx
 80ab56c:       84 d2                   test   %dl,%dl
 80ab56e:       0f 85 f7 fd ff ff       jne    80ab36b <uw_frame_state_for+0x1cb>
 80ab574:       eb 88                   jmp    80ab4fe <.L610+0xe>
 80ab576:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ab57d:       00
 80ab57e:       66 90                   xchg   %ax,%ax

080ab580 <.L604>:
 80ab580:       8b 5c 24 04             mov    0x4(%esp),%ebx
 80ab584:       83 c7 01                add    $0x1,%edi
 80ab587:       c6 83 83 00 00 00 01    movb   $0x1,0x83(%ebx)
 80ab58e:       0f b6 57 ff             movzbl -0x1(%edi),%edx
 80ab592:       84 d2                   test   %dl,%dl
 80ab594:       0f 85 d1 fd ff ff       jne    80ab36b <uw_frame_state_for+0x1cb>
 80ab59a:       e9 5f ff ff ff          jmp    80ab4fe <.L610+0xe>
 80ab59f:       90                      nop
 80ab5a0:       8b 48 48                mov    0x48(%eax),%ecx
 80ab5a3:       8b 40 4c                mov    0x4c(%eax),%eax
 80ab5a6:       66 81 38 58 b8          cmpw   $0xb858,(%eax)
 80ab5ab:       0f 84 03 02 00 00       je     80ab7b4 <.L604+0x234>
 80ab5b1:       80 38 b8                cmpb   $0xb8,(%eax)
 80ab5b4:       75 15                   jne    80ab5cb <.L604+0x4b>
 80ab5b6:       81 78 01 ad 00 00 00    cmpl   $0xad,0x1(%eax)
 80ab5bd:       75 0c                   jne    80ab5cb <.L604+0x4b>
 80ab5bf:       66 81 78 05 cd 80       cmpw   $0x80cd,0x5(%eax)
 80ab5c5:       0f 84 68 01 00 00       je     80ab733 <.L604+0x1b3>
 80ab5cb:       83 c4 3c                add    $0x3c,%esp
 80ab5ce:       b8 05 00 00 00          mov    $0x5,%eax
 80ab5d3:       5b                      pop    %ebx
 80ab5d4:       5e                      pop    %esi
 80ab5d5:       5f                      pop    %edi
 80ab5d6:       5d                      pop    %ebp
 80ab5d7:       c3                      ret
 80ab5d8:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ab5df:       00
 80ab5e0:       31 c9                   xor    %ecx,%ecx
 80ab5e2:       89 df                   mov    %ebx,%edi
 80ab5e4:       eb 1a                   jmp    80ab600 <.L604+0x80>
 80ab5e6:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ab5ed:       00
 80ab5ee:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ab5f5:       00
 80ab5f6:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ab5fd:       00
 80ab5fe:       66 90                   xchg   %ax,%ax
 80ab600:       0f b6 18                movzbl (%eax),%ebx
 80ab603:       83 c0 01                add    $0x1,%eax
 80ab606:       89 da                   mov    %ebx,%edx
 80ab608:       83 e2 7f                and    $0x7f,%edx
 80ab60b:       d3 e2                   shl    %cl,%edx
 80ab60d:       83 c1 07                add    $0x7,%ecx
 80ab610:       09 d5                   or     %edx,%ebp
 80ab612:       84 db                   test   %bl,%bl
 80ab614:       78 ea                   js     80ab600 <.L604+0x80>
 80ab616:       8b 4c 24 04             mov    0x4(%esp),%ecx
 80ab61a:       89 fb                   mov    %edi,%ebx
 80ab61c:       8b 7c 24 10             mov    0x10(%esp),%edi
 80ab620:       01 c5                   add    %eax,%ebp
 80ab622:       c6 81 82 00 00 00 01    movb   $0x1,0x82(%ecx)
 80ab629:       0f b6 57 01             movzbl 0x1(%edi),%edx
 80ab62d:       8d 4f 01                lea    0x1(%edi),%ecx
 80ab630:       84 d2                   test   %dl,%dl
 80ab632:       0f 84 64 fd ff ff       je     80ab39c <.L651+0xc>
 80ab638:       89 4c 24 10             mov    %ecx,0x10(%esp)
 80ab63c:       e9 1f fd ff ff          jmp    80ab360 <uw_frame_state_for+0x1c0>
 80ab641:       8d b4 26 00 00 00 00    lea    0x0(%esi,%eiz,1),%esi
 80ab648:       83 ec 0c                sub    $0xc,%esp
 80ab64b:       0f b6 d2                movzbl %dl,%edx
 80ab64e:       89 d9                   mov    %ebx,%ecx
 80ab650:       8d 44 24 38             lea    0x38(%esp),%eax
 80ab654:       50                      push   %eax
 80ab655:       8b 7c 24 18             mov    0x18(%esp),%edi
 80ab659:       89 f8                   mov    %edi,%eax
 80ab65b:       e8 80 dd ff ff          call   80a93e0 <read_encoded_value>
 80ab660:       8b 44 24 3c             mov    0x3c(%esp),%eax
 80ab664:       83 c4 10                add    $0x10,%esp
 80ab667:       89 47 50                mov    %eax,0x50(%edi)
 80ab66a:       e9 f2 fd ff ff          jmp    80ab461 <.L651+0xd1>
 80ab66f:       90                      nop
 80ab670:       84 c0                   test   %al,%al
 80ab672:       0f 85 61 01 00 00       jne    80ab7d9 <.L604+0x259>
 80ab678:       bb 10 00 00 00          mov    $0x10,%ebx
 80ab67d:       e9 80 fd ff ff          jmp    80ab402 <.L651+0x72>
 80ab682:       8d b6 00 00 00 00       lea    0x0(%esi),%esi
 80ab688:       80 fa ff                cmp    $0xff,%dl
 80ab68b:       0f 84 d2 fd ff ff       je     80ab463 <.L651+0xd3>
 80ab691:       83 ec 0c                sub    $0xc,%esp
 80ab694:       89 d9                   mov    %ebx,%ecx
 80ab696:       8d 44 24 38             lea    0x38(%esp),%eax
 80ab69a:       50                      push   %eax
 80ab69b:       8b 74 24 18             mov    0x18(%esp),%esi
 80ab69f:       89 f0                   mov    %esi,%eax
 80ab6a1:       e8 3a dd ff ff          call   80a93e0 <read_encoded_value>
 80ab6a6:       89 c3                   mov    %eax,%ebx
 80ab6a8:       8b 44 24 3c             mov    0x3c(%esp),%eax
 80ab6ac:       83 c4 10                add    $0x10,%esp
 80ab6af:       89 46 50                mov    %eax,0x50(%esi)
 80ab6b2:       e9 ac fd ff ff          jmp    80ab463 <.L651+0xd3>
 80ab6b7:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ab6be:       00
 80ab6bf:       90                      nop
 80ab6c0:       bb 0c 00 00 00          mov    $0xc,%ebx
 80ab6c5:       e9 38 fd ff ff          jmp    80ab402 <.L651+0x72>
 80ab6ca:       8d b6 00 00 00 00       lea    0x0(%esi),%esi
 80ab6d0:       80 38 04                cmpb   $0x4,(%eax)
 80ab6d3:       74 50                   je     80ab725 <.L604+0x1a5>
 80ab6d5:       83 c4 3c                add    $0x3c,%esp
 80ab6d8:       b8 03 00 00 00          mov    $0x3,%eax
 80ab6dd:       5b                      pop    %ebx
 80ab6de:       5e                      pop    %esi
 80ab6df:       5f                      pop    %edi
 80ab6e0:       5d                      pop    %ebp
 80ab6e1:       c3                      ret
 80ab6e2:       8d b6 00 00 00 00       lea    0x0(%esi),%esi
 80ab6e8:       bb 08 00 00 00          mov    $0x8,%ebx
 80ab6ed:       e9 10 fd ff ff          jmp    80ab402 <.L651+0x72>
 80ab6f2:       83 ec 0c                sub    $0xc,%esp
 80ab6f5:       50                      push   %eax
 80ab6f6:       8b 4c 24 18             mov    0x18(%esp),%ecx
 80ab6fa:       89 d8                   mov    %ebx,%eax
 80ab6fc:       e8 df e9 ff ff          call   80aa0e0 <execute_cfa_program_specialized>
 80ab701:       83 c4 10                add    $0x10,%esp
 80ab704:       31 c0                   xor    %eax,%eax
 80ab706:       e9 8d fd ff ff          jmp    80ab498 <.L651+0x108>
 80ab70b:       83 ec 0c                sub    $0xc,%esp
 80ab70e:       89 e8                   mov    %ebp,%eax
 80ab710:       ff 74 24 10             push   0x10(%esp)
 80ab714:       8b 4c 24 18             mov    0x18(%esp),%ecx
 80ab718:       e8 c3 e9 ff ff          call   80aa0e0 <execute_cfa_program_specialized>
 80ab71d:       83 c4 10                add    $0x10,%esp
 80ab720:       e9 a4 fc ff ff          jmp    80ab3c9 <.L651+0x39>
 80ab725:       80 78 01 00             cmpb   $0x0,0x1(%eax)
 80ab729:       75 aa                   jne    80ab6d5 <.L604+0x155>
 80ab72b:       83 c0 02                add    $0x2,%eax
 80ab72e:       e9 40 fb ff ff          jmp    80ab273 <uw_frame_state_for+0xd3>
 80ab733:       8d 81 a0 00 00 00       lea    0xa0(%ecx),%eax
 80ab739:       8b 50 1c                mov    0x1c(%eax),%edx
 80ab73c:       8b 74 24 04             mov    0x4(%esp),%esi
 80ab740:       89 d3                   mov    %edx,%ebx
 80ab742:       c6 46 5a 01             movb   $0x1,0x5a(%esi)
 80ab746:       29 cb                   sub    %ecx,%ebx
 80ab748:       8d 48 2c                lea    0x2c(%eax),%ecx
 80ab74b:       c7 46 64 04 00 00 00    movl   $0x4,0x64(%esi)
 80ab752:       29 d1                   sub    %edx,%ecx
 80ab754:       89 5e 60                mov    %ebx,0x60(%esi)
 80ab757:       89 0e                   mov    %ecx,(%esi)
 80ab759:       8d 48 20                lea    0x20(%eax),%ecx
 80ab75c:       29 d1                   sub    %edx,%ecx
 80ab75e:       c7 46 48 01 01 01 01    movl   $0x1010101,0x48(%esi)
 80ab765:       89 4e 0c                mov    %ecx,0xc(%esi)
 80ab768:       8d 48 28                lea    0x28(%eax),%ecx
 80ab76b:       29 d1                   sub    %edx,%ecx
 80ab76d:       c7 46 4d 01 01 01 01    movl   $0x1010101,0x4d(%esi)
 80ab774:       89 4e 04                mov    %ecx,0x4(%esi)
 80ab777:       8d 48 24                lea    0x24(%eax),%ecx
 80ab77a:       29 d1                   sub    %edx,%ecx
 80ab77c:       c7 46 7c 08 00 00 00    movl   $0x8,0x7c(%esi)
 80ab783:       89 4e 08                mov    %ecx,0x8(%esi)
 80ab786:       8d 48 14                lea    0x14(%eax),%ecx
 80ab789:       29 d1                   sub    %edx,%ecx
 80ab78b:       c6 86 83 00 00 00 01    movb   $0x1,0x83(%esi)
 80ab792:       89 4e 18                mov    %ecx,0x18(%esi)
 80ab795:       8d 48 10                lea    0x10(%eax),%ecx
 80ab798:       29 d1                   sub    %edx,%ecx
 80ab79a:       89 4e 1c                mov    %ecx,0x1c(%esi)
 80ab79d:       8d 48 18                lea    0x18(%eax),%ecx
 80ab7a0:       83 c0 38                add    $0x38,%eax
 80ab7a3:       29 d0                   sub    %edx,%eax
 80ab7a5:       29 d1                   sub    %edx,%ecx
 80ab7a7:       89 46 20                mov    %eax,0x20(%esi)
 80ab7aa:       31 c0                   xor    %eax,%eax
 80ab7ac:       89 4e 14                mov    %ecx,0x14(%esi)
 80ab7af:       e9 e4 fc ff ff          jmp    80ab498 <.L651+0x108>
 80ab7b4:       83 78 02 77             cmpl   $0x77,0x2(%eax)
 80ab7b8:       0f 85 f3 fd ff ff       jne    80ab5b1 <.L604+0x31>
 80ab7be:       66 81 78 06 cd 80       cmpw   $0x80cd,0x6(%eax)
 80ab7c4:       0f 85 e7 fd ff ff       jne    80ab5b1 <.L604+0x31>
 80ab7ca:       8d 41 04                lea    0x4(%ecx),%eax
 80ab7cd:       e9 67 ff ff ff          jmp    80ab739 <.L604+0x1b9>
 80ab7d2:       89 c5                   mov    %eax,%ebp
 80ab7d4:       e9 c3 fb ff ff          jmp    80ab39c <.L651+0xc>
 80ab7d9:       e9 e1 dd f9 ff          jmp    80495bf <uw_frame_state_for.cold>
 80ab7de:       66 90                   xchg   %ax,%ax

080ab7e0 <uw_init_context_1>:
 80ab7e0:       55                      push   %ebp
 80ab7e1:       89 d5                   mov    %edx,%ebp
 80ab7e3:       57                      push   %edi
 80ab7e4:       56                      push   %esi
 80ab7e5:       89 c6                   mov    %eax,%esi
 80ab7e7:       31 c0                   xor    %eax,%eax
 80ab7e9:       53                      push   %ebx
 80ab7ea:       89 f7                   mov    %esi,%edi
 80ab7ec:       e8 ef df f9 ff          call   80497e0 <__x86.get_pc_thunk.bx>
 80ab7f1:       81 c3 03 b8 03 00       add    $0x3b803,%ebx
 80ab7f7:       81 ec ac 00 00 00       sub    $0xac,%esp
 80ab7fd:       89 4c 24 0c             mov    %ecx,0xc(%esp)
 80ab801:       b9 20 00 00 00          mov    $0x20,%ecx
 80ab806:       f3 ab                   rep stos %eax,%es:(%edi)
 80ab808:       8d 7c 24 18             lea    0x18(%esp),%edi
 80ab80c:       89 fa                   mov    %edi,%edx
 80ab80e:       8b 84 24 bc 00 00 00    mov    0xbc(%esp),%eax
 80ab815:       c7 46 60 00 00 00 40    movl   $0x40000000,0x60(%esi)
 80ab81c:       89 46 4c                mov    %eax,0x4c(%esi)
 80ab81f:       89 f0                   mov    %esi,%eax
 80ab821:       e8 7a f9 ff ff          call   80ab1a0 <uw_frame_state_for>
 80ab826:       85 c0                   test   %eax,%eax
 80ab828:       0f 85 9a dd f9 ff       jne    80495c8 <uw_init_context_1.cold>
 80ab82e:       83 ec 08                sub    $0x8,%esp
 80ab831:       8d 83 ec 21 fc ff       lea    -0x3de14(%ebx),%eax
 80ab837:       50                      push   %eax
 80ab838:       8d 83 e0 3b 00 00       lea    0x3be0(%ebx),%eax
 80ab83e:       50                      push   %eax
 80ab83f:       e8 2c e8 fc ff          call   807a070 <___pthread_once>
 80ab844:       83 c4 10                add    $0x10,%esp
 80ab847:       85 c0                   test   %eax,%eax
 80ab849:       74 09                   je     80ab854 <uw_init_context_1+0x74>
 80ab84b:       80 bb e4 3b 00 00 00    cmpb   $0x0,0x3be4(%ebx)
 80ab852:       74 5c                   je     80ab8b0 <uw_init_context_1+0xd0>
 80ab854:       80 bb e8 3b 00 00 04    cmpb   $0x4,0x3be8(%ebx)
 80ab85b:       0f 85 67 dd f9 ff       jne    80495c8 <uw_init_context_1.cold>
 80ab861:       89 6c 24 14             mov    %ebp,0x14(%esp)
 80ab865:       f6 46 63 40             testb  $0x40,0x63(%esi)
 80ab869:       74 04                   je     80ab86f <uw_init_context_1+0x8f>
 80ab86b:       c6 46 70 00             movb   $0x0,0x70(%esi)
 80ab86f:       8d 44 24 14             lea    0x14(%esp),%eax
 80ab873:       89 fa                   mov    %edi,%edx
 80ab875:       c6 44 24 72 01          movb   $0x1,0x72(%esp)
 80ab87a:       89 46 10                mov    %eax,0x10(%esi)
 80ab87d:       89 f0                   mov    %esi,%eax
 80ab87f:       c7 44 24 7c 04 00 00    movl   $0x4,0x7c(%esp)
 80ab886:       00
 80ab887:       c7 44 24 78 00 00 00    movl   $0x0,0x78(%esp)
 80ab88e:       00
 80ab88f:       e8 8c e4 ff ff          call   80a9d20 <uw_update_context_1>
 80ab894:       8b 44 24 0c             mov    0xc(%esp),%eax
 80ab898:       89 46 4c                mov    %eax,0x4c(%esi)
 80ab89b:       81 c4 ac 00 00 00       add    $0xac,%esp
 80ab8a1:       5b                      pop    %ebx
 80ab8a2:       5e                      pop    %esi
 80ab8a3:       5f                      pop    %edi
 80ab8a4:       5d                      pop    %ebp
 80ab8a5:       c3                      ret
 80ab8a6:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ab8ad:       00
 80ab8ae:       66 90                   xchg   %ax,%ax
 80ab8b0:       c6 83 e4 3b 00 00 04    movb   $0x4,0x3be4(%ebx)
 80ab8b7:       c6 83 e6 3b 00 00 04    movb   $0x4,0x3be6(%ebx)
 80ab8be:       c6 83 e5 3b 00 00 04    movb   $0x4,0x3be5(%ebx)
 80ab8c5:       c6 83 e7 3b 00 00 04    movb   $0x4,0x3be7(%ebx)
 80ab8cc:       c6 83 ea 3b 00 00 04    movb   $0x4,0x3bea(%ebx)
 80ab8d3:       c6 83 eb 3b 00 00 04    movb   $0x4,0x3beb(%ebx)
 80ab8da:       c6 83 e9 3b 00 00 04    movb   $0x4,0x3be9(%ebx)
 80ab8e1:       c6 83 e8 3b 00 00 04    movb   $0x4,0x3be8(%ebx)
 80ab8e8:       c6 83 ef 3b 00 00 0c    movb   $0xc,0x3bef(%ebx)
 80ab8ef:       c6 83 f0 3b 00 00 0c    movb   $0xc,0x3bf0(%ebx)
 80ab8f6:       c6 83 f1 3b 00 00 0c    movb   $0xc,0x3bf1(%ebx)
 80ab8fd:       c6 83 f2 3b 00 00 0c    movb   $0xc,0x3bf2(%ebx)
 80ab904:       c6 83 f3 3b 00 00 0c    movb   $0xc,0x3bf3(%ebx)
 80ab90b:       c6 83 f4 3b 00 00 0c    movb   $0xc,0x3bf4(%ebx)
 80ab912:       c6 83 ed 3b 00 00 04    movb   $0x4,0x3bed(%ebx)
 80ab919:       c6 83 ec 3b 00 00 04    movb   $0x4,0x3bec(%ebx)
 80ab920:       e9 2f ff ff ff          jmp    80ab854 <uw_init_context_1+0x74>
 80ab925:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ab92c:       00
 80ab92d:       8d 76 00                lea    0x0(%esi),%esi

080ab930 <_Unwind_RaiseException_Phase2>:
 80ab930:       55                      push   %ebp
 80ab931:       e8 65 3f fa ff          call   804f89b <__x86.get_pc_thunk.bp>
 80ab936:       81 c5 be b6 03 00       add    $0x3b6be,%ebp
 80ab93c:       57                      push   %edi
 80ab93d:       56                      push   %esi
 80ab93e:       89 c6                   mov    %eax,%esi
 80ab940:       53                      push   %ebx
 80ab941:       89 d3                   mov    %edx,%ebx
 80ab943:       81 ec ac 00 00 00       sub    $0xac,%esp
 80ab949:       8d 85 e4 3b 00 00       lea    0x3be4(%ebp),%eax
 80ab94f:       89 6c 24 0c             mov    %ebp,0xc(%esp)
 80ab953:       8d 7c 24 18             lea    0x18(%esp),%edi
 80ab957:       89 4c 24 08             mov    %ecx,0x8(%esp)
 80ab95b:       c7 04 24 01 00 00 00    movl   $0x1,(%esp)
 80ab962:       89 44 24 04             mov    %eax,0x4(%esp)
 80ab966:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ab96d:       00
 80ab96e:       66 90                   xchg   %ax,%ax
 80ab970:       89 fa                   mov    %edi,%edx
 80ab972:       89 d8                   mov    %ebx,%eax
 80ab974:       e8 27 f8 ff ff          call   80ab1a0 <uw_frame_state_for>
 80ab979:       8b 4b 60                mov    0x60(%ebx),%ecx
 80ab97c:       8b 53 48                mov    0x48(%ebx),%edx
 80ab97f:       c1 e9 1f                shr    $0x1f,%ecx
 80ab982:       29 ca                   sub    %ecx,%edx
 80ab984:       39 56 10                cmp    %edx,0x10(%esi)
 80ab987:       0f 84 3b 01 00 00       je     80abac8 <_Unwind_RaiseException_Phase2+0x198>
 80ab98d:       85 c0                   test   %eax,%eax
 80ab98f:       0f 85 1f 01 00 00       jne    80abab4 <_Unwind_RaiseException_Phase2+0x184>
 80ab995:       8b 84 24 88 00 00 00    mov    0x88(%esp),%eax
 80ab99c:       b9 02 00 00 00          mov    $0x2,%ecx
 80ab9a1:       31 ed                   xor    %ebp,%ebp
 80ab9a3:       85 c0                   test   %eax,%eax
 80ab9a5:       74 2c                   je     80ab9d3 <_Unwind_RaiseException_Phase2+0xa3>
 80ab9a7:       83 ec 08                sub    $0x8,%esp
 80ab9aa:       53                      push   %ebx
 80ab9ab:       56                      push   %esi
 80ab9ac:       ff 76 04                push   0x4(%esi)
 80ab9af:       ff 36                   push   (%esi)
 80ab9b1:       51                      push   %ecx
 80ab9b2:       6a 01                   push   $0x1
 80ab9b4:       ff d0                   call   *%eax
 80ab9b6:       83 c4 20                add    $0x20,%esp
 80ab9b9:       83 f8 07                cmp    $0x7,%eax
 80ab9bc:       0f 84 2e 01 00 00       je     80abaf0 <_Unwind_RaiseException_Phase2+0x1c0>
 80ab9c2:       83 f8 08                cmp    $0x8,%eax
 80ab9c5:       0f 85 e9 00 00 00       jne    80abab4 <_Unwind_RaiseException_Phase2+0x184>
 80ab9cb:       85 ed                   test   %ebp,%ebp
 80ab9cd:       0f 85 fa db f9 ff       jne    80495cd <_Unwind_RaiseException_Phase2.cold>
 80ab9d3:       89 fa                   mov    %edi,%edx
 80ab9d5:       89 d8                   mov    %ebx,%eax
 80ab9d7:       e8 44 e3 ff ff          call   80a9d20 <uw_update_context_1>
 80ab9dc:       8b 94 24 94 00 00 00    mov    0x94(%esp),%edx
 80ab9e3:       80 7c 14 60 07          cmpb   $0x7,0x60(%esp,%edx,1)
 80ab9e8:       0f 84 8a 00 00 00       je     80aba78 <_Unwind_RaiseException_Phase2+0x148>
 80ab9ee:       83 fa 11                cmp    $0x11,%edx
 80ab9f1:       0f 8f d6 db f9 ff       jg     80495cd <_Unwind_RaiseException_Phase2.cold>
 80ab9f7:       8b 43 60                mov    0x60(%ebx),%eax
 80ab9fa:       8b 0c 93                mov    (%ebx,%edx,4),%ecx
 80ab9fd:       a9 00 00 00 40          test   $0x40000000,%eax
 80aba02:       74 07                   je     80aba0b <_Unwind_RaiseException_Phase2+0xdb>
 80aba04:       80 7c 13 6c 00          cmpb   $0x0,0x6c(%ebx,%edx,1)
 80aba09:       75 11                   jne    80aba1c <_Unwind_RaiseException_Phase2+0xec>
 80aba0b:       8b 6c 24 04             mov    0x4(%esp),%ebp
 80aba0f:       80 7c 15 00 04          cmpb   $0x4,0x0(%ebp,%edx,1)
 80aba14:       0f 85 b3 db f9 ff       jne    80495cd <_Unwind_RaiseException_Phase2.cold>
 80aba1a:       8b 09                   mov    (%ecx),%ecx
 80aba1c:       89 ca                   mov    %ecx,%edx
 80aba1e:       c1 e8 1f                shr    $0x1f,%eax
 80aba21:       89 53 4c                mov    %edx,0x4c(%ebx)
 80aba24:       74 5f                   je     80aba85 <_Unwind_RaiseException_Phase2+0x155>
 80aba26:       31 c0                   xor    %eax,%eax
 80aba28:       f3 0f 1e c8             rdsspd %eax
 80aba2c:       85 c0                   test   %eax,%eax
 80aba2e:       0f 84 3c ff ff ff       je     80ab970 <_Unwind_RaiseException_Phase2+0x40>
 80aba34:       83 c0 04                add    $0x4,%eax
 80aba37:       83 e0 f8                and    $0xfffffff8,%eax
 80aba3a:       8b 50 f8                mov    -0x8(%eax),%edx
 80aba3d:       89 d1                   mov    %edx,%ecx
 80aba3f:       83 e1 f8                and    $0xfffffff8,%ecx
 80aba42:       39 c8                   cmp    %ecx,%eax
 80aba44:       74 18                   je     80aba5e <_Unwind_RaiseException_Phase2+0x12e>
 80aba46:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80aba4d:       00
 80aba4e:       66 90                   xchg   %ax,%ax
 80aba50:       8b 10                   mov    (%eax),%edx
 80aba52:       83 c0 08                add    $0x8,%eax
 80aba55:       89 d1                   mov    %edx,%ecx
 80aba57:       83 e1 f8                and    $0xfffffff8,%ecx
 80aba5a:       39 c1                   cmp    %eax,%ecx
 80aba5c:       75 f2                   jne    80aba50 <_Unwind_RaiseException_Phase2+0x120>
 80aba5e:       8b 0c 24                mov    (%esp),%ecx
 80aba61:       31 c0                   xor    %eax,%eax
 80aba63:       83 e2 04                and    $0x4,%edx
 80aba66:       0f 95 c0                setne  %al
 80aba69:       8d 44 01 02             lea    0x2(%ecx,%eax,1),%eax
 80aba6d:       89 04 24                mov    %eax,(%esp)
 80aba70:       e9 fb fe ff ff          jmp    80ab970 <_Unwind_RaiseException_Phase2+0x40>
 80aba75:       8d 76 00                lea    0x0(%esi),%esi
 80aba78:       8b 43 60                mov    0x60(%ebx),%eax
 80aba7b:       31 d2                   xor    %edx,%edx
 80aba7d:       89 53 4c                mov    %edx,0x4c(%ebx)
 80aba80:       c1 e8 1f                shr    $0x1f,%eax
 80aba83:       75 a1                   jne    80aba26 <_Unwind_RaiseException_Phase2+0xf6>
 80aba85:       83 04 24 01             addl   $0x1,(%esp)
 80aba89:       8b 0e                   mov    (%esi),%ecx
 80aba8b:       0b 4e 04                or     0x4(%esi),%ecx
 80aba8e:       8b 2c 24                mov    (%esp),%ebp
 80aba91:       0f 84 d9 fe ff ff       je     80ab970 <_Unwind_RaiseException_Phase2+0x40>
 80aba97:       85 d2                   test   %edx,%edx
 80aba99:       0f 84 d1 fe ff ff       je     80ab970 <_Unwind_RaiseException_Phase2+0x40>
 80aba9f:       f3 0f 1e c8             rdsspd %eax
 80abaa3:       85 c0                   test   %eax,%eax
 80abaa5:       0f 84 c5 fe ff ff       je     80ab970 <_Unwind_RaiseException_Phase2+0x40>
 80abaab:       3b 14 a8                cmp    (%eax,%ebp,4),%edx
 80abaae:       0f 84 bc fe ff ff       je     80ab970 <_Unwind_RaiseException_Phase2+0x40>
 80abab4:       81 c4 ac 00 00 00       add    $0xac,%esp
 80ababa:       b8 02 00 00 00          mov    $0x2,%eax
 80ababf:       5b                      pop    %ebx
 80abac0:       5e                      pop    %esi
 80abac1:       5f                      pop    %edi
 80abac2:       5d                      pop    %ebp
 80abac3:       c3                      ret
 80abac4:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi
 80abac8:       85 c0                   test   %eax,%eax
 80abaca:       75 e8                   jne    80abab4 <_Unwind_RaiseException_Phase2+0x184>
 80abacc:       8b 84 24 88 00 00 00    mov    0x88(%esp),%eax
 80abad3:       85 c0                   test   %eax,%eax
 80abad5:       0f 84 f2 da f9 ff       je     80495cd <_Unwind_RaiseException_Phase2.cold>
 80abadb:       b9 06 00 00 00          mov    $0x6,%ecx
 80abae0:       bd 04 00 00 00          mov    $0x4,%ebp
 80abae5:       e9 bd fe ff ff          jmp    80ab9a7 <_Unwind_RaiseException_Phase2+0x77>
 80abaea:       8d b6 00 00 00 00       lea    0x0(%esi),%esi
 80abaf0:       8b 7c 24 08             mov    0x8(%esp),%edi
 80abaf4:       8b 34 24                mov    (%esp),%esi
 80abaf7:       89 37                   mov    %esi,(%edi)
 80abaf9:       81 c4 ac 00 00 00       add    $0xac,%esp
 80abaff:       5b                      pop    %ebx
 80abb00:       5e                      pop    %esi
 80abb01:       5f                      pop    %edi
 80abb02:       5d                      pop    %ebp
 80abb03:       c3                      ret
 80abb04:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80abb0b:       00
 80abb0c:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi

080abb10 <_Unwind_ForcedUnwind_Phase2>:
 80abb10:       55                      push   %ebp
 80abb11:       bd 01 00 00 00          mov    $0x1,%ebp
 80abb16:       57                      push   %edi
 80abb17:       e8 17 03 fa ff          call   804be33 <__x86.get_pc_thunk.di>
 80abb1c:       81 c7 d8 b4 03 00       add    $0x3b4d8,%edi
 80abb22:       56                      push   %esi
 80abb23:       89 c6                   mov    %eax,%esi
 80abb25:       53                      push   %ebx
 80abb26:       89 d3                   mov    %edx,%ebx
 80abb28:       81 ec bc 00 00 00       sub    $0xbc,%esp
 80abb2e:       8b 50 0c                mov    0xc(%eax),%edx
 80abb31:       89 4c 24 1c             mov    %ecx,0x1c(%esp)
 80abb35:       8b 48 10                mov    0x10(%eax),%ecx
 80abb38:       89 7c 24 18             mov    %edi,0x18(%esp)
 80abb3c:       8b 44 24 18             mov    0x18(%esp),%eax
 80abb40:       8d 7c 24 28             lea    0x28(%esp),%edi
 80abb44:       89 54 24 0c             mov    %edx,0xc(%esp)
 80abb48:       8d 80 e4 3b 00 00       lea    0x3be4(%eax),%eax
 80abb4e:       89 4c 24 10             mov    %ecx,0x10(%esp)
 80abb52:       89 44 24 14             mov    %eax,0x14(%esp)
 80abb56:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80abb5d:       00
 80abb5e:       66 90                   xchg   %ax,%ax
 80abb60:       89 fa                   mov    %edi,%edx
 80abb62:       89 d8                   mov    %ebx,%eax
 80abb64:       e8 37 f6 ff ff          call   80ab1a0 <uw_frame_state_for>
 80abb69:       89 c1                   mov    %eax,%ecx
 80abb6b:       b8 21 00 00 00          mov    $0x21,%eax
 80abb70:       0f a3 c8                bt     %ecx,%eax
 80abb73:       0f 83 f2 00 00 00       jae    80abc6b <_Unwind_ForcedUnwind_Phase2+0x15b>
 80abb79:       8b 06                   mov    (%esi),%eax
 80abb7b:       8b 56 04                mov    0x4(%esi),%edx
 80abb7e:       85 c9                   test   %ecx,%ecx
 80abb80:       0f 85 42 01 00 00       jne    80abcc8 <_Unwind_ForcedUnwind_Phase2+0x1b8>
 80abb86:       83 ec 04                sub    $0x4,%esp
 80abb89:       ff 74 24 14             push   0x14(%esp)
 80abb8d:       53                      push   %ebx
 80abb8e:       56                      push   %esi
 80abb8f:       52                      push   %edx
 80abb90:       50                      push   %eax
 80abb91:       6a 0a                   push   $0xa
 80abb93:       6a 01                   push   $0x1
 80abb95:       8b 44 24 2c             mov    0x2c(%esp),%eax
 80abb99:       ff d0                   call   *%eax
 80abb9b:       83 c4 20                add    $0x20,%esp
 80abb9e:       85 c0                   test   %eax,%eax
 80abba0:       0f 85 c5 00 00 00       jne    80abc6b <_Unwind_ForcedUnwind_Phase2+0x15b>
 80abba6:       8b 84 24 98 00 00 00    mov    0x98(%esp),%eax
 80abbad:       85 c0                   test   %eax,%eax
 80abbaf:       74 25                   je     80abbd6 <_Unwind_ForcedUnwind_Phase2+0xc6>
 80abbb1:       83 ec 08                sub    $0x8,%esp
 80abbb4:       53                      push   %ebx
 80abbb5:       56                      push   %esi
 80abbb6:       ff 76 04                push   0x4(%esi)
 80abbb9:       ff 36                   push   (%esi)
 80abbbb:       6a 0a                   push   $0xa
 80abbbd:       6a 01                   push   $0x1
 80abbbf:       ff d0                   call   *%eax
 80abbc1:       83 c4 20                add    $0x20,%esp
 80abbc4:       83 f8 07                cmp    $0x7,%eax
 80abbc7:       0f 84 1c 01 00 00       je     80abce9 <_Unwind_ForcedUnwind_Phase2+0x1d9>
 80abbcd:       83 f8 08                cmp    $0x8,%eax
 80abbd0:       0f 85 95 00 00 00       jne    80abc6b <_Unwind_ForcedUnwind_Phase2+0x15b>
 80abbd6:       89 fa                   mov    %edi,%edx
 80abbd8:       89 d8                   mov    %ebx,%eax
 80abbda:       e8 41 e1 ff ff          call   80a9d20 <uw_update_context_1>
 80abbdf:       8b 94 24 a4 00 00 00    mov    0xa4(%esp),%edx
 80abbe6:       80 7c 14 70 07          cmpb   $0x7,0x70(%esp,%edx,1)
 80abbeb:       0f 85 8f 00 00 00       jne    80abc80 <_Unwind_ForcedUnwind_Phase2+0x170>
 80abbf1:       8b 43 60                mov    0x60(%ebx),%eax
 80abbf4:       31 d2                   xor    %edx,%edx
 80abbf6:       c1 e8 1f                shr    $0x1f,%eax
 80abbf9:       89 53 4c                mov    %edx,0x4c(%ebx)
 80abbfc:       74 42                   je     80abc40 <_Unwind_ForcedUnwind_Phase2+0x130>
 80abbfe:       31 c0                   xor    %eax,%eax
 80abc00:       f3 0f 1e c8             rdsspd %eax
 80abc04:       85 c0                   test   %eax,%eax
 80abc06:       0f 84 54 ff ff ff       je     80abb60 <_Unwind_ForcedUnwind_Phase2+0x50>
 80abc0c:       83 c0 04                add    $0x4,%eax
 80abc0f:       83 e0 f8                and    $0xfffffff8,%eax
 80abc12:       8b 50 f8                mov    -0x8(%eax),%edx
 80abc15:       89 d1                   mov    %edx,%ecx
 80abc17:       83 e1 f8                and    $0xfffffff8,%ecx
 80abc1a:       39 c8                   cmp    %ecx,%eax
 80abc1c:       74 10                   je     80abc2e <_Unwind_ForcedUnwind_Phase2+0x11e>
 80abc1e:       66 90                   xchg   %ax,%ax
 80abc20:       8b 10                   mov    (%eax),%edx
 80abc22:       83 c0 08                add    $0x8,%eax
 80abc25:       89 d1                   mov    %edx,%ecx
 80abc27:       83 e1 f8                and    $0xfffffff8,%ecx
 80abc2a:       39 c8                   cmp    %ecx,%eax
 80abc2c:       75 f2                   jne    80abc20 <_Unwind_ForcedUnwind_Phase2+0x110>
 80abc2e:       31 c0                   xor    %eax,%eax
 80abc30:       83 e2 04                and    $0x4,%edx
 80abc33:       0f 95 c0                setne  %al
 80abc36:       8d 6c 05 02             lea    0x2(%ebp,%eax,1),%ebp
 80abc3a:       e9 21 ff ff ff          jmp    80abb60 <_Unwind_ForcedUnwind_Phase2+0x50>
 80abc3f:       90                      nop
 80abc40:       8b 0e                   mov    (%esi),%ecx
 80abc42:       83 c5 01                add    $0x1,%ebp
 80abc45:       0b 4e 04                or     0x4(%esi),%ecx
 80abc48:       0f 84 12 ff ff ff       je     80abb60 <_Unwind_ForcedUnwind_Phase2+0x50>
 80abc4e:       85 d2                   test   %edx,%edx
 80abc50:       0f 84 0a ff ff ff       je     80abb60 <_Unwind_ForcedUnwind_Phase2+0x50>
 80abc56:       f3 0f 1e c8             rdsspd %eax
 80abc5a:       85 c0                   test   %eax,%eax
 80abc5c:       0f 84 fe fe ff ff       je     80abb60 <_Unwind_ForcedUnwind_Phase2+0x50>
 80abc62:       3b 14 a8                cmp    (%eax,%ebp,4),%edx
 80abc65:       0f 84 f5 fe ff ff       je     80abb60 <_Unwind_ForcedUnwind_Phase2+0x50>
 80abc6b:       81 c4 bc 00 00 00       add    $0xbc,%esp
 80abc71:       b8 02 00 00 00          mov    $0x2,%eax
 80abc76:       5b                      pop    %ebx
 80abc77:       5e                      pop    %esi
 80abc78:       5f                      pop    %edi
 80abc79:       5d                      pop    %ebp
 80abc7a:       c3                      ret
 80abc7b:       2e 8d 74 26 00          lea    %cs:0x0(%esi,%eiz,1),%esi
 80abc80:       83 fa 11                cmp    $0x11,%edx
 80abc83:       0f 8f 4d d9 f9 ff       jg     80495d6 <_Unwind_ForcedUnwind_Phase2.cold>
 80abc89:       8b 04 93                mov    (%ebx,%edx,4),%eax
 80abc8c:       89 44 24 08             mov    %eax,0x8(%esp)
 80abc90:       8b 43 60                mov    0x60(%ebx),%eax
 80abc93:       a9 00 00 00 40          test   $0x40000000,%eax
 80abc98:       74 07                   je     80abca1 <_Unwind_ForcedUnwind_Phase2+0x191>
 80abc9a:       80 7c 13 6c 00          cmpb   $0x0,0x6c(%ebx,%edx,1)
 80abc9f:       75 18                   jne    80abcb9 <_Unwind_ForcedUnwind_Phase2+0x1a9>
 80abca1:       8b 4c 24 14             mov    0x14(%esp),%ecx
 80abca5:       80 3c 11 04             cmpb   $0x4,(%ecx,%edx,1)
 80abca9:       0f 85 27 d9 f9 ff       jne    80495d6 <_Unwind_ForcedUnwind_Phase2.cold>
 80abcaf:       8b 54 24 08             mov    0x8(%esp),%edx
 80abcb3:       8b 0a                   mov    (%edx),%ecx
 80abcb5:       89 4c 24 08             mov    %ecx,0x8(%esp)
 80abcb9:       8b 54 24 08             mov    0x8(%esp),%edx
 80abcbd:       e9 34 ff ff ff          jmp    80abbf6 <_Unwind_ForcedUnwind_Phase2+0xe6>
 80abcc2:       8d b6 00 00 00 00       lea    0x0(%esi),%esi
 80abcc8:       83 ec 04                sub    $0x4,%esp
 80abccb:       ff 74 24 14             push   0x14(%esp)
 80abccf:       53                      push   %ebx
 80abcd0:       56                      push   %esi
 80abcd1:       52                      push   %edx
 80abcd2:       50                      push   %eax
 80abcd3:       6a 1a                   push   $0x1a
 80abcd5:       6a 01                   push   $0x1
 80abcd7:       8b 44 24 2c             mov    0x2c(%esp),%eax
 80abcdb:       ff d0                   call   *%eax
 80abcdd:       83 c4 20                add    $0x20,%esp
 80abce0:       85 c0                   test   %eax,%eax
 80abce2:       75 87                   jne    80abc6b <_Unwind_ForcedUnwind_Phase2+0x15b>
 80abce4:       b8 05 00 00 00          mov    $0x5,%eax
 80abce9:       8b 7c 24 1c             mov    0x1c(%esp),%edi
 80abced:       89 2f                   mov    %ebp,(%edi)
 80abcef:       81 c4 bc 00 00 00       add    $0xbc,%esp
 80abcf5:       5b                      pop    %ebx
 80abcf6:       5e                      pop    %esi
 80abcf7:       5f                      pop    %edi
 80abcf8:       5d                      pop    %ebp
 80abcf9:       c3                      ret
 80abcfa:       8d b6 00 00 00 00       lea    0x0(%esi),%esi

080abd00 <_Unwind_GetGR>:
 80abd00:       f3 0f 1e fb             endbr32
 80abd04:       53                      push   %ebx
 80abd05:       e8 d6 da f9 ff          call   80497e0 <__x86.get_pc_thunk.bx>
 80abd0a:       81 c3 ea b2 03 00       add    $0x3b2ea,%ebx
 80abd10:       83 ec 08                sub    $0x8,%esp
 80abd13:       8b 54 24 14             mov    0x14(%esp),%edx
 80abd17:       8b 4c 24 10             mov    0x10(%esp),%ecx
 80abd1b:       83 fa 11                cmp    $0x11,%edx
 80abd1e:       0f 8f bb d8 f9 ff       jg     80495df <_Unwind_GetGR.cold>
 80abd24:       8b 04 91                mov    (%ecx,%edx,4),%eax
 80abd27:       f6 41 63 40             testb  $0x40,0x63(%ecx)
 80abd2b:       74 07                   je     80abd34 <_Unwind_GetGR+0x34>
 80abd2d:       80 7c 11 6c 00          cmpb   $0x0,0x6c(%ecx,%edx,1)
 80abd32:       75 10                   jne    80abd44 <_Unwind_GetGR+0x44>
 80abd34:       80 bc 13 e4 3b 00 00    cmpb   $0x4,0x3be4(%ebx,%edx,1)
 80abd3b:       04
 80abd3c:       0f 85 9d d8 f9 ff       jne    80495df <_Unwind_GetGR.cold>
 80abd42:       8b 00                   mov    (%eax),%eax
 80abd44:       83 c4 08                add    $0x8,%esp
 80abd47:       5b                      pop    %ebx
 80abd48:       c3                      ret
 80abd49:       8d b4 26 00 00 00 00    lea    0x0(%esi,%eiz,1),%esi

080abd50 <_Unwind_GetCFA>:
 80abd50:       f3 0f 1e fb             endbr32
 80abd54:       8b 44 24 04             mov    0x4(%esp),%eax
 80abd58:       8b 40 48                mov    0x48(%eax),%eax
 80abd5b:       c3                      ret
 80abd5c:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi

080abd60 <_Unwind_SetGR>:
 80abd60:       f3 0f 1e fb             endbr32
 80abd64:       e8 2e 3b fa ff          call   804f897 <__x86.get_pc_thunk.cx>
 80abd69:       81 c1 8b b2 03 00       add    $0x3b28b,%ecx
 80abd6f:       53                      push   %ebx
 80abd70:       83 ec 08                sub    $0x8,%esp
 80abd73:       8b 44 24 14             mov    0x14(%esp),%eax
 80abd77:       8b 54 24 10             mov    0x10(%esp),%edx
 80abd7b:       83 f8 11                cmp    $0x11,%eax
 80abd7e:       0f 8f 60 d8 f9 ff       jg     80495e4 <_Unwind_SetGR.cold>
 80abd84:       0f b6 9c 01 e4 3b 00    movzbl 0x3be4(%ecx,%eax,1),%ebx
 80abd8b:       00
 80abd8c:       f6 42 63 40             testb  $0x40,0x63(%edx)
 80abd90:       74 07                   je     80abd99 <_Unwind_SetGR+0x39>
 80abd92:       80 7c 02 6c 00          cmpb   $0x0,0x6c(%edx,%eax,1)
 80abd97:       75 17                   jne    80abdb0 <_Unwind_SetGR+0x50>
 80abd99:       8b 04 82                mov    (%edx,%eax,4),%eax
 80abd9c:       80 fb 04                cmp    $0x4,%bl
 80abd9f:       0f 85 3f d8 f9 ff       jne    80495e4 <_Unwind_SetGR.cold>
 80abda5:       8b 5c 24 18             mov    0x18(%esp),%ebx
 80abda9:       89 18                   mov    %ebx,(%eax)
 80abdab:       83 c4 08                add    $0x8,%esp
 80abdae:       5b                      pop    %ebx
 80abdaf:       c3                      ret
 80abdb0:       8b 4c 24 18             mov    0x18(%esp),%ecx
 80abdb4:       89 0c 82                mov    %ecx,(%edx,%eax,4)
 80abdb7:       83 c4 08                add    $0x8,%esp
 80abdba:       5b                      pop    %ebx
 80abdbb:       c3                      ret
 80abdbc:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi

080abdc0 <_Unwind_GetIP>:
 80abdc0:       f3 0f 1e fb             endbr32
 80abdc4:       8b 44 24 04             mov    0x4(%esp),%eax
 80abdc8:       8b 40 4c                mov    0x4c(%eax),%eax
 80abdcb:       c3                      ret
 80abdcc:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi

080abdd0 <_Unwind_GetIPInfo>:
 80abdd0:       f3 0f 1e fb             endbr32
 80abdd4:       8b 54 24 04             mov    0x4(%esp),%edx
 80abdd8:       8b 4c 24 08             mov    0x8(%esp),%ecx
 80abddc:       8b 42 60                mov    0x60(%edx),%eax
 80abddf:       c1 e8 1f                shr    $0x1f,%eax
 80abde2:       89 01                   mov    %eax,(%ecx)
 80abde4:       8b 42 4c                mov    0x4c(%edx),%eax
 80abde7:       c3                      ret
 80abde8:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80abdef:       00

080abdf0 <_Unwind_SetIP>:
 80abdf0:       f3 0f 1e fb             endbr32
 80abdf4:       8b 44 24 04             mov    0x4(%esp),%eax
 80abdf8:       8b 54 24 08             mov    0x8(%esp),%edx
 80abdfc:       89 50 4c                mov    %edx,0x4c(%eax)
 80abdff:       c3                      ret

080abe00 <_Unwind_GetLanguageSpecificData>:
 80abe00:       f3 0f 1e fb             endbr32
 80abe04:       8b 44 24 04             mov    0x4(%esp),%eax
 80abe08:       8b 40 50                mov    0x50(%eax),%eax
 80abe0b:       c3                      ret
 80abe0c:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi

080abe10 <_Unwind_GetRegionStart>:
 80abe10:       f3 0f 1e fb             endbr32
 80abe14:       8b 44 24 04             mov    0x4(%esp),%eax
 80abe18:       8b 40 5c                mov    0x5c(%eax),%eax
 80abe1b:       c3                      ret
 80abe1c:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi

080abe20 <_Unwind_FindEnclosingFunction>:
 80abe20:       f3 0f 1e fb             endbr32
 80abe24:       53                      push   %ebx
 80abe25:       e8 b6 d9 f9 ff          call   80497e0 <__x86.get_pc_thunk.bx>
 80abe2a:       81 c3 ca b1 03 00       add    $0x3b1ca,%ebx
 80abe30:       83 ec 20                sub    $0x20,%esp
 80abe33:       8d 44 24 0c             lea    0xc(%esp),%eax
 80abe37:       50                      push   %eax
 80abe38:       8b 44 24 2c             mov    0x2c(%esp),%eax
 80abe3c:       83 e8 01                sub    $0x1,%eax
 80abe3f:       50                      push   %eax
 80abe40:       e8 2b 2e 00 00          call   80aec70 <_Unwind_Find_FDE>
 80abe45:       83 c4 10                add    $0x10,%esp
 80abe48:       85 c0                   test   %eax,%eax
 80abe4a:       74 04                   je     80abe50 <_Unwind_FindEnclosingFunction+0x30>
 80abe4c:       8b 44 24 0c             mov    0xc(%esp),%eax
 80abe50:       83 c4 18                add    $0x18,%esp
 80abe53:       5b                      pop    %ebx
 80abe54:       c3                      ret
 80abe55:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80abe5c:       00
 80abe5d:       8d 76 00                lea    0x0(%esi),%esi

080abe60 <_Unwind_GetDataRelBase>:
 80abe60:       f3 0f 1e fb             endbr32
 80abe64:       8b 44 24 04             mov    0x4(%esp),%eax
 80abe68:       8b 40 58                mov    0x58(%eax),%eax
 80abe6b:       c3                      ret
 80abe6c:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi

080abe70 <_Unwind_GetTextRelBase>:
 80abe70:       f3 0f 1e fb             endbr32
 80abe74:       8b 44 24 04             mov    0x4(%esp),%eax
 80abe78:       8b 40 54                mov    0x54(%eax),%eax
 80abe7b:       c3                      ret
 80abe7c:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi

080abe80 <__frame_state_for>:
 80abe80:       f3 0f 1e fb             endbr32
 80abe84:       57                      push   %edi
 80abe85:       31 c0                   xor    %eax,%eax
 80abe87:       b9 20 00 00 00          mov    $0x20,%ecx
 80abe8c:       56                      push   %esi
 80abe8d:       53                      push   %ebx
 80abe8e:       81 ec 10 01 00 00       sub    $0x110,%esp
 80abe94:       8d 74 24 08             lea    0x8(%esp),%esi
 80abe98:       8b 9c 24 24 01 00 00    mov    0x124(%esp),%ebx
 80abe9f:       89 f7                   mov    %esi,%edi
 80abea1:       f3 ab                   rep stos %eax,%es:(%edi)
 80abea3:       8b 84 24 20 01 00 00    mov    0x120(%esp),%eax
 80abeaa:       8d bc 24 88 00 00 00    lea    0x88(%esp),%edi
 80abeb1:       c7 44 24 68 00 00 00    movl   $0x40000000,0x68(%esp)
 80abeb8:       40
 80abeb9:       89 fa                   mov    %edi,%edx
 80abebb:       83 c0 01                add    $0x1,%eax
 80abebe:       89 44 24 54             mov    %eax,0x54(%esp)
 80abec2:       89 f0                   mov    %esi,%eax
 80abec4:       e8 d7 f2 ff ff          call   80ab1a0 <uw_frame_state_for>
 80abec9:       85 c0                   test   %eax,%eax
 80abecb:       75 73                   jne    80abf40 <__frame_state_for+0xc0>
 80abecd:       80 bc 24 e2 00 00 00    cmpb   $0x2,0xe2(%esp)
 80abed4:       02
 80abed5:       74 69                   je     80abf40 <__frame_state_for+0xc0>
 80abed7:       8d b4 24 d0 00 00 00    lea    0xd0(%esp),%esi
 80abede:       66 90                   xchg   %ax,%ax
 80abee0:       0f b6 14 06             movzbl (%esi,%eax,1),%edx
 80abee4:       88 54 03 5c             mov    %dl,0x5c(%ebx,%eax,1)
 80abee8:       80 fa 01                cmp    $0x1,%dl
 80abeeb:       74 07                   je     80abef4 <__frame_state_for+0x74>
 80abeed:       31 c9                   xor    %ecx,%ecx
 80abeef:       80 fa 02                cmp    $0x2,%dl
 80abef2:       75 03                   jne    80abef7 <__frame_state_for+0x77>
 80abef4:       8b 0c 87                mov    (%edi,%eax,4),%ecx
 80abef7:       89 4c 83 10             mov    %ecx,0x10(%ebx,%eax,4)
 80abefb:       83 c0 01                add    $0x1,%eax
 80abefe:       83 f8 12                cmp    $0x12,%eax
 80abf01:       75 dd                   jne    80abee0 <__frame_state_for+0x60>
 80abf03:       8b 84 24 e8 00 00 00    mov    0xe8(%esp),%eax
 80abf0a:       89 43 08                mov    %eax,0x8(%ebx)
 80abf0d:       8b 84 24 ec 00 00 00    mov    0xec(%esp),%eax
 80abf14:       66 89 43 58             mov    %ax,0x58(%ebx)
 80abf18:       8b 84 24 04 01 00 00    mov    0x104(%esp),%eax
 80abf1f:       66 89 43 5a             mov    %ax,0x5a(%ebx)
 80abf23:       8b 44 24 70             mov    0x70(%esp),%eax
 80abf27:       89 43 0c                mov    %eax,0xc(%ebx)
 80abf2a:       8b 84 24 0c 01 00 00    mov    0x10c(%esp),%eax
 80abf31:       89 43 04                mov    %eax,0x4(%ebx)
 80abf34:       81 c4 10 01 00 00       add    $0x110,%esp
 80abf3a:       89 d8                   mov    %ebx,%eax
 80abf3c:       5b                      pop    %ebx
 80abf3d:       5e                      pop    %esi
 80abf3e:       5f                      pop    %edi
 80abf3f:       c3                      ret
 80abf40:       81 c4 10 01 00 00       add    $0x110,%esp
 80abf46:       31 c0                   xor    %eax,%eax
 80abf48:       5b                      pop    %ebx
 80abf49:       5e                      pop    %esi
 80abf4a:       5f                      pop    %edi
 80abf4b:       c3                      ret
 80abf4c:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi

080abf50 <_Unwind_DebugHook>:
 80abf50:       f3 0f 1e fb             endbr32
 80abf54:       c3                      ret
 80abf55:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80abf5c:       00
 80abf5d:       8d 76 00                lea    0x0(%esi),%esi

080abf60 <_Unwind_RaiseException>:
 80abf60:       f3 0f 1e fb             endbr32
 80abf64:       55                      push   %ebp
 80abf65:       89 e5                   mov    %esp,%ebp
 80abf67:       57                      push   %edi
 80abf68:       56                      push   %esi
 80abf69:       8d b5 60 fe ff ff       lea    -0x1a0(%ebp),%esi
 80abf6f:       53                      push   %ebx
 80abf70:       e8 6b d8 f9 ff          call   80497e0 <__x86.get_pc_thunk.bx>
 80abf75:       81 c3 7f b0 03 00       add    $0x3b07f,%ebx
 80abf7b:       52                      push   %edx
 80abf7c:       8d 55 08                lea    0x8(%ebp),%edx
 80abf7f:       50                      push   %eax
 80abf80:       89 f0                   mov    %esi,%eax
 80abf82:       81 ec a4 01 00 00       sub    $0x1a4,%esp
 80abf88:       8b 4d 04                mov    0x4(%ebp),%ecx
 80abf8b:       89 b5 4c fe ff ff       mov    %esi,-0x1b4(%ebp)
 80abf91:       89 9d 48 fe ff ff       mov    %ebx,-0x1b8(%ebp)
 80abf97:       e8 44 f8 ff ff          call   80ab7e0 <uw_init_context_1>
 80abf9c:       8d 95 e0 fe ff ff       lea    -0x120(%ebp),%edx
 80abfa2:       b9 20 00 00 00          mov    $0x20,%ecx
 80abfa7:       8d 85 60 ff ff ff       lea    -0xa0(%ebp),%eax
 80abfad:       89 d7                   mov    %edx,%edi
 80abfaf:       89 85 54 fe ff ff       mov    %eax,-0x1ac(%ebp)
 80abfb5:       f3 a5                   rep movsl %ds:(%esi),%es:(%edi)
 80abfb7:       8d b3 e4 3b 00 00       lea    0x3be4(%ebx),%esi
 80abfbd:       8b 5d 08                mov    0x8(%ebp),%ebx
 80abfc0:       89 b5 50 fe ff ff       mov    %esi,-0x1b0(%ebp)
 80abfc6:       89 d6                   mov    %edx,%esi
 80abfc8:       e9 8b 00 00 00          jmp    80ac058 <_Unwind_RaiseException+0xf8>
 80abfcd:       8d 76 00                lea    0x0(%esi),%esi
 80abfd0:       85 c0                   test   %eax,%eax
 80abfd2:       0f 85 a8 00 00 00       jne    80ac080 <_Unwind_RaiseException+0x120>
 80abfd8:       8b 45 d0                mov    -0x30(%ebp),%eax
 80abfdb:       85 c0                   test   %eax,%eax
 80abfdd:       74 25                   je     80ac004 <_Unwind_RaiseException+0xa4>
 80abfdf:       83 ec 08                sub    $0x8,%esp
 80abfe2:       56                      push   %esi
 80abfe3:       53                      push   %ebx
 80abfe4:       ff 73 04                push   0x4(%ebx)
 80abfe7:       ff 33                   push   (%ebx)
 80abfe9:       6a 01                   push   $0x1
 80abfeb:       6a 01                   push   $0x1
 80abfed:       ff d0                   call   *%eax
 80abfef:       83 c4 20                add    $0x20,%esp
 80abff2:       83 f8 06                cmp    $0x6,%eax
 80abff5:       0f 84 9d 00 00 00       je     80ac098 <_Unwind_RaiseException+0x138>
 80abffb:       83 f8 08                cmp    $0x8,%eax
 80abffe:       0f 85 7c 00 00 00       jne    80ac080 <_Unwind_RaiseException+0x120>
 80ac004:       8b 95 54 fe ff ff       mov    -0x1ac(%ebp),%edx
 80ac00a:       89 f0                   mov    %esi,%eax
 80ac00c:       e8 0f dd ff ff          call   80a9d20 <uw_update_context_1>
 80ac011:       8b 55 dc                mov    -0x24(%ebp),%edx
 80ac014:       31 c0                   xor    %eax,%eax
 80ac016:       80 7c 15 a8 07          cmpb   $0x7,-0x58(%ebp,%edx,1)
 80ac01b:       74 35                   je     80ac052 <_Unwind_RaiseException+0xf2>
 80ac01d:       83 fa 11                cmp    $0x11,%edx
 80ac020:       0f 8f c5 d5 f9 ff       jg     80495eb <_Unwind_RaiseException.cold>
 80ac026:       8b 84 95 e0 fe ff ff    mov    -0x120(%ebp,%edx,4),%eax
 80ac02d:       f6 85 43 ff ff ff 40    testb  $0x40,-0xbd(%ebp)
 80ac034:       74 0a                   je     80ac040 <_Unwind_RaiseException+0xe0>
 80ac036:       80 bc 15 4c ff ff ff    cmpb   $0x0,-0xb4(%ebp,%edx,1)
 80ac03d:       00
 80ac03e:       75 12                   jne    80ac052 <_Unwind_RaiseException+0xf2>
 80ac040:       8b 8d 50 fe ff ff       mov    -0x1b0(%ebp),%ecx
 80ac046:       80 3c 11 04             cmpb   $0x4,(%ecx,%edx,1)
 80ac04a:       0f 85 9b d5 f9 ff       jne    80495eb <_Unwind_RaiseException.cold>
 80ac050:       8b 00                   mov    (%eax),%eax
 80ac052:       89 85 2c ff ff ff       mov    %eax,-0xd4(%ebp)
 80ac058:       8b 95 54 fe ff ff       mov    -0x1ac(%ebp),%edx
 80ac05e:       89 f0                   mov    %esi,%eax
 80ac060:       e8 3b f1 ff ff          call   80ab1a0 <uw_frame_state_for>
 80ac065:       89 c7                   mov    %eax,%edi
 80ac067:       83 f8 05                cmp    $0x5,%eax
 80ac06a:       0f 85 60 ff ff ff       jne    80abfd0 <_Unwind_RaiseException+0x70>
 80ac070:       89 c3                   mov    %eax,%ebx
 80ac072:       8b 75 f8                mov    -0x8(%ebp),%esi
 80ac075:       8b 7d fc                mov    -0x4(%ebp),%edi
 80ac078:       89 d8                   mov    %ebx,%eax
 80ac07a:       8b 5d f4                mov    -0xc(%ebp),%ebx
 80ac07d:       c9                      leave
 80ac07e:       c3                      ret
 80ac07f:       90                      nop
 80ac080:       bb 03 00 00 00          mov    $0x3,%ebx
 80ac085:       8b 75 f8                mov    -0x8(%ebp),%esi
 80ac088:       8b 7d fc                mov    -0x4(%ebp),%edi
 80ac08b:       89 d8                   mov    %ebx,%eax
 80ac08d:       8b 5d f4                mov    -0xc(%ebp),%ebx
 80ac090:       c9                      leave
 80ac091:       c3                      ret
 80ac092:       8d b6 00 00 00 00       lea    0x0(%esi),%esi
 80ac098:       8b 45 08                mov    0x8(%ebp),%eax
 80ac09b:       8b 8d 40 ff ff ff       mov    -0xc0(%ebp),%ecx
 80ac0a1:       89 f2                   mov    %esi,%edx
 80ac0a3:       89 fb                   mov    %edi,%ebx
 80ac0a5:       89 f7                   mov    %esi,%edi
 80ac0a7:       8b b5 4c fe ff ff       mov    -0x1b4(%ebp),%esi
 80ac0ad:       c7 40 0c 00 00 00 00    movl   $0x0,0xc(%eax)
 80ac0b4:       8b 85 28 ff ff ff       mov    -0xd8(%ebp),%eax
 80ac0ba:       c1 e9 1f                shr    $0x1f,%ecx
 80ac0bd:       29 c8                   sub    %ecx,%eax
 80ac0bf:       8b 4d 08                mov    0x8(%ebp),%ecx
 80ac0c2:       89 41 10                mov    %eax,0x10(%ecx)
 80ac0c5:       8b 45 08                mov    0x8(%ebp),%eax
 80ac0c8:       b9 20 00 00 00          mov    $0x20,%ecx
 80ac0cd:       f3 a5                   rep movsl %ds:(%esi),%es:(%edi)
 80ac0cf:       8b 8d 54 fe ff ff       mov    -0x1ac(%ebp),%ecx
 80ac0d5:       89 95 54 fe ff ff       mov    %edx,-0x1ac(%ebp)
 80ac0db:       e8 50 f8 ff ff          call   80ab930 <_Unwind_RaiseException_Phase2>
 80ac0e0:       8b 95 54 fe ff ff       mov    -0x1ac(%ebp),%edx
 80ac0e6:       83 f8 07                cmp    $0x7,%eax
 80ac0e9:       74 12                   je     80ac0fd <_Unwind_RaiseException+0x19d>
 80ac0eb:       bb 02 00 00 00          mov    $0x2,%ebx
 80ac0f0:       8b 75 f8                mov    -0x8(%ebp),%esi
 80ac0f3:       8b 7d fc                mov    -0x4(%ebp),%edi
 80ac0f6:       89 d8                   mov    %ebx,%eax
 80ac0f8:       8b 5d f4                mov    -0xc(%ebp),%ebx
 80ac0fb:       c9                      leave
 80ac0fc:       c3                      ret
 80ac0fd:       8b 85 4c fe ff ff       mov    -0x1b4(%ebp),%eax
 80ac103:       e8 58 d1 ff ff          call   80a9260 <uw_install_context_1>
 80ac108:       8b b5 2c ff ff ff       mov    -0xd4(%ebp),%esi
 80ac10e:       89 c2                   mov    %eax,%edx
 80ac110:       50                      push   %eax
 80ac111:       50                      push   %eax
 80ac112:       56                      push   %esi
 80ac113:       ff b5 28 ff ff ff       push   -0xd8(%ebp)
 80ac119:       e8 32 fe ff ff          call   80abf50 <_Unwind_DebugHook>
 80ac11e:       f3 0f 1e cb             rdsspd %ebx
 80ac122:       83 c4 10                add    $0x10,%esp
 80ac125:       85 db                   test   %ebx,%ebx
 80ac127:       74 21                   je     80ac14a <_Unwind_RaiseException+0x1ea>
 80ac129:       8b 85 60 ff ff ff       mov    -0xa0(%ebp),%eax
 80ac12f:       b9 ff 00 00 00          mov    $0xff,%ecx
 80ac134:       eb 09                   jmp    80ac13f <_Unwind_RaiseException+0x1df>
 80ac136:       f3 0f ae e9             incsspd %ecx
 80ac13a:       2d ff 00 00 00          sub    $0xff,%eax
 80ac13f:       3d ff 00 00 00          cmp    $0xff,%eax
 80ac144:       77 f0                   ja     80ac136 <_Unwind_RaiseException+0x1d6>
 80ac146:       f3 0f ae e8             incsspd %eax
 80ac14a:       89 d1                   mov    %edx,%ecx
 80ac14c:       89 74 15 04             mov    %esi,0x4(%ebp,%edx,1)
 80ac150:       8b 45 ec                mov    -0x14(%ebp),%eax
 80ac153:       8d 4c 0d 04             lea    0x4(%ebp,%ecx,1),%ecx
 80ac157:       8b 55 f0                mov    -0x10(%ebp),%edx
 80ac15a:       8b 5d f4                mov    -0xc(%ebp),%ebx
 80ac15d:       8b 75 f8                mov    -0x8(%ebp),%esi
 80ac160:       8b 7d fc                mov    -0x4(%ebp),%edi
 80ac163:       8b 6d 00                mov    0x0(%ebp),%ebp
 80ac166:       89 cc                   mov    %ecx,%esp
 80ac168:       59                      pop    %ecx
 80ac169:       ff e1                   jmp    *%ecx
 80ac16b:       2e 8d 74 26 00          lea    %cs:0x0(%esi,%eiz,1),%esi

080ac170 <_Unwind_ForcedUnwind>:
 80ac170:       f3 0f 1e fb             endbr32
 80ac174:       55                      push   %ebp
 80ac175:       89 e5                   mov    %esp,%ebp
 80ac177:       57                      push   %edi
 80ac178:       56                      push   %esi
 80ac179:       8d b5 e8 fe ff ff       lea    -0x118(%ebp),%esi
 80ac17f:       8d bd 68 ff ff ff       lea    -0x98(%ebp),%edi
 80ac185:       53                      push   %ebx
 80ac186:       52                      push   %edx
 80ac187:       8d 55 08                lea    0x8(%ebp),%edx
 80ac18a:       50                      push   %eax
 80ac18b:       89 f0                   mov    %esi,%eax
 80ac18d:       81 ec 24 01 00 00       sub    $0x124,%esp
 80ac193:       8b 4d 04                mov    0x4(%ebp),%ecx
 80ac196:       89 b5 d4 fe ff ff       mov    %esi,-0x12c(%ebp)
 80ac19c:       e8 3f f6 ff ff          call   80ab7e0 <uw_init_context_1>
 80ac1a1:       8b 45 0c                mov    0xc(%ebp),%eax
 80ac1a4:       b9 20 00 00 00          mov    $0x20,%ecx
 80ac1a9:       f3 a5                   rep movsl %ds:(%esi),%es:(%edi)
 80ac1ab:       8b 7d 08                mov    0x8(%ebp),%edi
 80ac1ae:       8d b5 68 ff ff ff       lea    -0x98(%ebp),%esi
 80ac1b4:       89 f2                   mov    %esi,%edx
 80ac1b6:       89 47 0c                mov    %eax,0xc(%edi)
 80ac1b9:       8b 45 10                mov    0x10(%ebp),%eax
 80ac1bc:       89 47 10                mov    %eax,0x10(%edi)
 80ac1bf:       89 f8                   mov    %edi,%eax
 80ac1c1:       89 cb                   mov    %ecx,%ebx
 80ac1c3:       8d 8d e4 fe ff ff       lea    -0x11c(%ebp),%ecx
 80ac1c9:       e8 42 f9 ff ff          call   80abb10 <_Unwind_ForcedUnwind_Phase2>
 80ac1ce:       83 f8 07                cmp    $0x7,%eax
 80ac1d1:       74 0b                   je     80ac1de <_Unwind_ForcedUnwind+0x6e>
 80ac1d3:       8b 5d f4                mov    -0xc(%ebp),%ebx
 80ac1d6:       8b 75 f8                mov    -0x8(%ebp),%esi
 80ac1d9:       8b 7d fc                mov    -0x4(%ebp),%edi
 80ac1dc:       c9                      leave
 80ac1dd:       c3                      ret
 80ac1de:       8b 85 d4 fe ff ff       mov    -0x12c(%ebp),%eax
 80ac1e4:       89 f2                   mov    %esi,%edx
 80ac1e6:       e8 75 d0 ff ff          call   80a9260 <uw_install_context_1>
 80ac1eb:       8b 75 b4                mov    -0x4c(%ebp),%esi
 80ac1ee:       89 c7                   mov    %eax,%edi
 80ac1f0:       50                      push   %eax
 80ac1f1:       50                      push   %eax
 80ac1f2:       56                      push   %esi
 80ac1f3:       ff 75 b0                push   -0x50(%ebp)
 80ac1f6:       e8 55 fd ff ff          call   80abf50 <_Unwind_DebugHook>
 80ac1fb:       f3 0f 1e cb             rdsspd %ebx
 80ac1ff:       83 c4 10                add    $0x10,%esp
 80ac202:       85 db                   test   %ebx,%ebx
 80ac204:       74 51                   je     80ac257 <_Unwind_ForcedUnwind+0xe7>
 80ac206:       8b 8d e4 fe ff ff       mov    -0x11c(%ebp),%ecx
 80ac20c:       81 f9 ff 00 00 00       cmp    $0xff,%ecx
 80ac212:       76 3f                   jbe    80ac253 <_Unwind_ForcedUnwind+0xe3>
 80ac214:       8d 81 00 ff ff ff       lea    -0x100(%ecx),%eax
 80ac21a:       ba 81 80 80 80          mov    $0x80808081,%edx
 80ac21f:       bb ff 00 00 00          mov    $0xff,%ebx
 80ac224:       f7 e2                   mul    %edx
 80ac226:       80 e2 80                and    $0x80,%dl
 80ac229:       75 12                   jne    80ac23d <_Unwind_ForcedUnwind+0xcd>
 80ac22b:       f3 0f ae eb             incsspd %ebx
 80ac22f:       81 e9 ff 00 00 00       sub    $0xff,%ecx
 80ac235:       81 f9 ff 00 00 00       cmp    $0xff,%ecx
 80ac23b:       76 16                   jbe    80ac253 <_Unwind_ForcedUnwind+0xe3>
 80ac23d:       f3 0f ae eb             incsspd %ebx
 80ac241:       f3 0f ae eb             incsspd %ebx
 80ac245:       81 e9 fe 01 00 00       sub    $0x1fe,%ecx
 80ac24b:       81 f9 ff 00 00 00       cmp    $0xff,%ecx
 80ac251:       77 ea                   ja     80ac23d <_Unwind_ForcedUnwind+0xcd>
 80ac253:       f3 0f ae e9             incsspd %ecx
 80ac257:       89 f9                   mov    %edi,%ecx
 80ac259:       89 74 3d 04             mov    %esi,0x4(%ebp,%edi,1)
 80ac25d:       8b 45 ec                mov    -0x14(%ebp),%eax
 80ac260:       8d 4c 0d 04             lea    0x4(%ebp,%ecx,1),%ecx
 80ac264:       8b 55 f0                mov    -0x10(%ebp),%edx
 80ac267:       8b 5d f4                mov    -0xc(%ebp),%ebx
 80ac26a:       8b 75 f8                mov    -0x8(%ebp),%esi
 80ac26d:       8b 7d fc                mov    -0x4(%ebp),%edi
 80ac270:       8b 6d 00                mov    0x0(%ebp),%ebp
 80ac273:       89 cc                   mov    %ecx,%esp
 80ac275:       59                      pop    %ecx
 80ac276:       ff e1                   jmp    *%ecx
 80ac278:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ac27f:       00

080ac280 <_Unwind_Resume>:
 80ac280:       f3 0f 1e fb             endbr32
 80ac284:       55                      push   %ebp
 80ac285:       89 e5                   mov    %esp,%ebp
 80ac287:       57                      push   %edi
 80ac288:       56                      push   %esi
 80ac289:       8d b5 e8 fe ff ff       lea    -0x118(%ebp),%esi
 80ac28f:       53                      push   %ebx
 80ac290:       8d 9d 68 ff ff ff       lea    -0x98(%ebp),%ebx
 80ac296:       52                      push   %edx
 80ac297:       8d 55 08                lea    0x8(%ebp),%edx
 80ac29a:       89 df                   mov    %ebx,%edi
 80ac29c:       50                      push   %eax
 80ac29d:       e8 9f d6 f9 ff          call   8049941 <__x86.get_pc_thunk.ax>
 80ac2a2:       05 52 ad 03 00          add    $0x3ad52,%eax
 80ac2a7:       81 ec 24 01 00 00       sub    $0x124,%esp
 80ac2ad:       8b 4d 04                mov    0x4(%ebp),%ecx
 80ac2b0:       89 b5 d4 fe ff ff       mov    %esi,-0x12c(%ebp)
 80ac2b6:       89 85 d0 fe ff ff       mov    %eax,-0x130(%ebp)
 80ac2bc:       89 f0                   mov    %esi,%eax
 80ac2be:       e8 1d f5 ff ff          call   80ab7e0 <uw_init_context_1>
 80ac2c3:       8b 45 08                mov    0x8(%ebp),%eax
 80ac2c6:       b9 20 00 00 00          mov    $0x20,%ecx
 80ac2cb:       89 da                   mov    %ebx,%edx
 80ac2cd:       f3 a5                   rep movsl %ds:(%esi),%es:(%edi)
 80ac2cf:       8d 8d e4 fe ff ff       lea    -0x11c(%ebp),%ecx
 80ac2d5:       8b 70 0c                mov    0xc(%eax),%esi
 80ac2d8:       85 f6                   test   %esi,%esi
 80ac2da:       0f 85 bb 00 00 00       jne    80ac39b <_Unwind_Resume+0x11b>
 80ac2e0:       e8 4b f6 ff ff          call   80ab930 <_Unwind_RaiseException_Phase2>
 80ac2e5:       83 f8 07                cmp    $0x7,%eax
 80ac2e8:       0f 85 08 d3 f9 ff       jne    80495f6 <_Unwind_Resume.cold>
 80ac2ee:       8b 85 d4 fe ff ff       mov    -0x12c(%ebp),%eax
 80ac2f4:       89 da                   mov    %ebx,%edx
 80ac2f6:       e8 65 cf ff ff          call   80a9260 <uw_install_context_1>
 80ac2fb:       8b 75 b4                mov    -0x4c(%ebp),%esi
 80ac2fe:       83 ec 08                sub    $0x8,%esp
 80ac301:       89 c7                   mov    %eax,%edi
 80ac303:       31 c0                   xor    %eax,%eax
 80ac305:       56                      push   %esi
 80ac306:       ff 75 b0                push   -0x50(%ebp)
 80ac309:       e8 42 fc ff ff          call   80abf50 <_Unwind_DebugHook>
 80ac30e:       f3 0f 1e c8             rdsspd %eax
 80ac312:       83 c4 10                add    $0x10,%esp
 80ac315:       85 c0                   test   %eax,%eax
 80ac317:       74 61                   je     80ac37a <_Unwind_Resume+0xfa>
 80ac319:       8b 8d e4 fe ff ff       mov    -0x11c(%ebp),%ecx
 80ac31f:       81 f9 ff 00 00 00       cmp    $0xff,%ecx
 80ac325:       76 4f                   jbe    80ac376 <_Unwind_Resume+0xf6>
 80ac327:       8d 81 00 ff ff ff       lea    -0x100(%ecx),%eax
 80ac32d:       ba 81 80 80 80          mov    $0x80808081,%edx
 80ac332:       bb ff 00 00 00          mov    $0xff,%ebx
 80ac337:       f7 e2                   mul    %edx
 80ac339:       80 e2 80                and    $0x80,%dl
 80ac33c:       75 22                   jne    80ac360 <_Unwind_Resume+0xe0>
 80ac33e:       f3 0f ae eb             incsspd %ebx
 80ac342:       81 e9 ff 00 00 00       sub    $0xff,%ecx
 80ac348:       81 f9 ff 00 00 00       cmp    $0xff,%ecx
 80ac34e:       76 26                   jbe    80ac376 <_Unwind_Resume+0xf6>
 80ac350:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ac357:       00
 80ac358:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ac35f:       00
 80ac360:       f3 0f ae eb             incsspd %ebx
 80ac364:       f3 0f ae eb             incsspd %ebx
 80ac368:       81 e9 fe 01 00 00       sub    $0x1fe,%ecx
 80ac36e:       81 f9 ff 00 00 00       cmp    $0xff,%ecx
 80ac374:       77 ea                   ja     80ac360 <_Unwind_Resume+0xe0>
 80ac376:       f3 0f ae e9             incsspd %ecx
 80ac37a:       89 f9                   mov    %edi,%ecx
 80ac37c:       89 74 3d 04             mov    %esi,0x4(%ebp,%edi,1)
 80ac380:       8b 45 ec                mov    -0x14(%ebp),%eax
 80ac383:       8d 4c 0d 04             lea    0x4(%ebp,%ecx,1),%ecx
 80ac387:       8b 55 f0                mov    -0x10(%ebp),%edx
 80ac38a:       8b 5d f4                mov    -0xc(%ebp),%ebx
 80ac38d:       8b 75 f8                mov    -0x8(%ebp),%esi
 80ac390:       8b 7d fc                mov    -0x4(%ebp),%edi
 80ac393:       8b 6d 00                mov    0x0(%ebp),%ebp
 80ac396:       89 cc                   mov    %ecx,%esp
 80ac398:       59                      pop    %ecx
 80ac399:       ff e1                   jmp    *%ecx
 80ac39b:       8b 45 08                mov    0x8(%ebp),%eax
 80ac39e:       e8 6d f7 ff ff          call   80abb10 <_Unwind_ForcedUnwind_Phase2>
 80ac3a3:       e9 3d ff ff ff          jmp    80ac2e5 <_Unwind_Resume+0x65>
 80ac3a8:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ac3af:       00

080ac3b0 <_Unwind_Resume_or_Rethrow>:
 80ac3b0:       f3 0f 1e fb             endbr32
 80ac3b4:       55                      push   %ebp
 80ac3b5:       89 e5                   mov    %esp,%ebp
 80ac3b7:       57                      push   %edi
 80ac3b8:       56                      push   %esi
 80ac3b9:       53                      push   %ebx
 80ac3ba:       e8 21 d4 f9 ff          call   80497e0 <__x86.get_pc_thunk.bx>
 80ac3bf:       81 c3 35 ac 03 00       add    $0x3ac35,%ebx
 80ac3c5:       52                      push   %edx
 80ac3c6:       50                      push   %eax
 80ac3c7:       81 ec 24 01 00 00       sub    $0x124,%esp
 80ac3cd:       8b 45 08                mov    0x8(%ebp),%eax
 80ac3d0:       8b 50 0c                mov    0xc(%eax),%edx
 80ac3d3:       85 d2                   test   %edx,%edx
 80ac3d5:       75 19                   jne    80ac3f0 <_Unwind_Resume_or_Rethrow+0x40>
 80ac3d7:       83 ec 0c                sub    $0xc,%esp
 80ac3da:       ff 75 08                push   0x8(%ebp)
 80ac3dd:       e8 7e fb ff ff          call   80abf60 <_Unwind_RaiseException>
 80ac3e2:       8b 5d f4                mov    -0xc(%ebp),%ebx
 80ac3e5:       8b 75 f8                mov    -0x8(%ebp),%esi
 80ac3e8:       83 c4 10                add    $0x10,%esp
 80ac3eb:       8b 7d fc                mov    -0x4(%ebp),%edi
 80ac3ee:       c9                      leave
 80ac3ef:       c3                      ret
 80ac3f0:       8d b5 e8 fe ff ff       lea    -0x118(%ebp),%esi
 80ac3f6:       8b 4d 04                mov    0x4(%ebp),%ecx
 80ac3f9:       8d 55 08                lea    0x8(%ebp),%edx
 80ac3fc:       89 f0                   mov    %esi,%eax
 80ac3fe:       89 b5 d4 fe ff ff       mov    %esi,-0x12c(%ebp)
 80ac404:       8d bd 68 ff ff ff       lea    -0x98(%ebp),%edi
 80ac40a:       e8 d1 f3 ff ff          call   80ab7e0 <uw_init_context_1>
 80ac40f:       b9 20 00 00 00          mov    $0x20,%ecx
 80ac414:       8b 45 08                mov    0x8(%ebp),%eax
 80ac417:       f3 a5                   rep movsl %ds:(%esi),%es:(%edi)
 80ac419:       8d b5 68 ff ff ff       lea    -0x98(%ebp),%esi
 80ac41f:       89 f2                   mov    %esi,%edx
 80ac421:       89 8d d0 fe ff ff       mov    %ecx,-0x130(%ebp)
 80ac427:       8d 8d e4 fe ff ff       lea    -0x11c(%ebp),%ecx
 80ac42d:       e8 de f6 ff ff          call   80abb10 <_Unwind_ForcedUnwind_Phase2>
 80ac432:       83 f8 07                cmp    $0x7,%eax
 80ac435:       0f 85 c6 d1 f9 ff       jne    8049601 <_Unwind_Resume_or_Rethrow.cold>
 80ac43b:       8b 85 d4 fe ff ff       mov    -0x12c(%ebp),%eax
 80ac441:       89 f2                   mov    %esi,%edx
 80ac443:       e8 18 ce ff ff          call   80a9260 <uw_install_context_1>
 80ac448:       8b 5d b4                mov    -0x4c(%ebp),%ebx
 80ac44b:       89 c7                   mov    %eax,%edi
 80ac44d:       50                      push   %eax
 80ac44e:       50                      push   %eax
 80ac44f:       53                      push   %ebx
 80ac450:       ff 75 b0                push   -0x50(%ebp)
 80ac453:       e8 f8 fa ff ff          call   80abf50 <_Unwind_DebugHook>
 80ac458:       8b 85 d0 fe ff ff       mov    -0x130(%ebp),%eax
 80ac45e:       f3 0f 1e c8             rdsspd %eax
 80ac462:       83 c4 10                add    $0x10,%esp
 80ac465:       85 c0                   test   %eax,%eax
 80ac467:       74 51                   je     80ac4ba <_Unwind_Resume_or_Rethrow+0x10a>
 80ac469:       8b 8d e4 fe ff ff       mov    -0x11c(%ebp),%ecx
 80ac46f:       81 f9 ff 00 00 00       cmp    $0xff,%ecx
 80ac475:       76 3f                   jbe    80ac4b6 <_Unwind_Resume_or_Rethrow+0x106>
 80ac477:       8d 81 00 ff ff ff       lea    -0x100(%ecx),%eax
 80ac47d:       ba 81 80 80 80          mov    $0x80808081,%edx
 80ac482:       be ff 00 00 00          mov    $0xff,%esi
 80ac487:       f7 e2                   mul    %edx
 80ac489:       80 e2 80                and    $0x80,%dl
 80ac48c:       75 12                   jne    80ac4a0 <_Unwind_Resume_or_Rethrow+0xf0>
 80ac48e:       f3 0f ae ee             incsspd %esi
 80ac492:       81 e9 ff 00 00 00       sub    $0xff,%ecx
 80ac498:       81 f9 ff 00 00 00       cmp    $0xff,%ecx
 80ac49e:       76 16                   jbe    80ac4b6 <_Unwind_Resume_or_Rethrow+0x106>
 80ac4a0:       f3 0f ae ee             incsspd %esi
 80ac4a4:       f3 0f ae ee             incsspd %esi
 80ac4a8:       81 e9 fe 01 00 00       sub    $0x1fe,%ecx
 80ac4ae:       81 f9 ff 00 00 00       cmp    $0xff,%ecx
 80ac4b4:       77 ea                   ja     80ac4a0 <_Unwind_Resume_or_Rethrow+0xf0>
 80ac4b6:       f3 0f ae e9             incsspd %ecx
 80ac4ba:       89 f9                   mov    %edi,%ecx
 80ac4bc:       89 5c 3d 04             mov    %ebx,0x4(%ebp,%edi,1)
 80ac4c0:       8b 45 ec                mov    -0x14(%ebp),%eax
 80ac4c3:       8d 4c 0d 04             lea    0x4(%ebp,%ecx,1),%ecx
 80ac4c7:       8b 55 f0                mov    -0x10(%ebp),%edx
 80ac4ca:       8b 5d f4                mov    -0xc(%ebp),%ebx
 80ac4cd:       8b 75 f8                mov    -0x8(%ebp),%esi
 80ac4d0:       8b 7d fc                mov    -0x4(%ebp),%edi
 80ac4d3:       8b 6d 00                mov    0x0(%ebp),%ebp
 80ac4d6:       89 cc                   mov    %ecx,%esp
 80ac4d8:       59                      pop    %ecx
 80ac4d9:       ff e1                   jmp    *%ecx
 80ac4db:       2e 8d 74 26 00          lea    %cs:0x0(%esi,%eiz,1),%esi

080ac4e0 <_Unwind_DeleteException>:
 80ac4e0:       f3 0f 1e fb             endbr32
 80ac4e4:       83 ec 0c                sub    $0xc,%esp
 80ac4e7:       8b 54 24 10             mov    0x10(%esp),%edx
 80ac4eb:       8b 42 08                mov    0x8(%edx),%eax
 80ac4ee:       85 c0                   test   %eax,%eax
 80ac4f0:       74 0b                   je     80ac4fd <_Unwind_DeleteException+0x1d>
 80ac4f2:       83 ec 08                sub    $0x8,%esp
 80ac4f5:       52                      push   %edx
 80ac4f6:       6a 01                   push   $0x1
 80ac4f8:       ff d0                   call   *%eax
 80ac4fa:       83 c4 10                add    $0x10,%esp
 80ac4fd:       83 c4 0c                add    $0xc,%esp
 80ac500:       c3                      ret
 80ac501:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ac508:       00
 80ac509:       8d b4 26 00 00 00 00    lea    0x0(%esi,%eiz,1),%esi

080ac510 <_Unwind_Backtrace>:
 80ac510:       f3 0f 1e fb             endbr32
 80ac514:       55                      push   %ebp
 80ac515:       89 e5                   mov    %esp,%ebp
 80ac517:       57                      push   %edi
 80ac518:       e8 16 f9 f9 ff          call   804be33 <__x86.get_pc_thunk.di>
 80ac51d:       81 c7 d7 aa 03 00       add    $0x3aad7,%edi
 80ac523:       56                      push   %esi
 80ac524:       8d 55 08                lea    0x8(%ebp),%edx
 80ac527:       8d b5 60 ff ff ff       lea    -0xa0(%ebp),%esi
 80ac52d:       53                      push   %ebx
 80ac52e:       8d 9d e0 fe ff ff       lea    -0x120(%ebp),%ebx
 80ac534:       89 d8                   mov    %ebx,%eax
 80ac536:       81 ec 2c 01 00 00       sub    $0x12c,%esp
 80ac53c:       8b 4d 04                mov    0x4(%ebp),%ecx
 80ac53f:       89 bd d0 fe ff ff       mov    %edi,-0x130(%ebp)
 80ac545:       e8 96 f2 ff ff          call   80ab7e0 <uw_init_context_1>
 80ac54a:       8d 87 e4 3b 00 00       lea    0x3be4(%edi),%eax
 80ac550:       89 85 d4 fe ff ff       mov    %eax,-0x12c(%ebp)
 80ac556:       eb 6e                   jmp    80ac5c6 <_Unwind_Backtrace+0xb6>
 80ac558:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ac55f:       00
 80ac560:       83 ec 08                sub    $0x8,%esp
 80ac563:       ff 75 0c                push   0xc(%ebp)
 80ac566:       53                      push   %ebx
 80ac567:       ff 55 08                call   *0x8(%ebp)
 80ac56a:       83 c4 10                add    $0x10,%esp
 80ac56d:       85 c0                   test   %eax,%eax
 80ac56f:       75 6c                   jne    80ac5dd <_Unwind_Backtrace+0xcd>
 80ac571:       83 ff 05                cmp    $0x5,%edi
 80ac574:       74 6c                   je     80ac5e2 <_Unwind_Backtrace+0xd2>
 80ac576:       89 f2                   mov    %esi,%edx
 80ac578:       89 d8                   mov    %ebx,%eax
 80ac57a:       e8 a1 d7 ff ff          call   80a9d20 <uw_update_context_1>
 80ac57f:       8b 55 dc                mov    -0x24(%ebp),%edx
 80ac582:       31 c0                   xor    %eax,%eax
 80ac584:       80 7c 15 a8 07          cmpb   $0x7,-0x58(%ebp,%edx,1)
 80ac589:       74 35                   je     80ac5c0 <_Unwind_Backtrace+0xb0>
 80ac58b:       83 fa 11                cmp    $0x11,%edx
 80ac58e:       0f 8f 72 d0 f9 ff       jg     8049606 <_Unwind_Backtrace.cold>
 80ac594:       8b 84 95 e0 fe ff ff    mov    -0x120(%ebp,%edx,4),%eax
 80ac59b:       f6 85 43 ff ff ff 40    testb  $0x40,-0xbd(%ebp)
 80ac5a2:       74 0a                   je     80ac5ae <_Unwind_Backtrace+0x9e>
 80ac5a4:       80 bc 15 4c ff ff ff    cmpb   $0x0,-0xb4(%ebp,%edx,1)
 80ac5ab:       00
 80ac5ac:       75 12                   jne    80ac5c0 <_Unwind_Backtrace+0xb0>
 80ac5ae:       8b 8d d4 fe ff ff       mov    -0x12c(%ebp),%ecx
 80ac5b4:       80 3c 11 04             cmpb   $0x4,(%ecx,%edx,1)
 80ac5b8:       0f 85 48 d0 f9 ff       jne    8049606 <_Unwind_Backtrace.cold>
 80ac5be:       8b 00                   mov    (%eax),%eax
 80ac5c0:       89 85 2c ff ff ff       mov    %eax,-0xd4(%ebp)
 80ac5c6:       89 f2                   mov    %esi,%edx
 80ac5c8:       89 d8                   mov    %ebx,%eax
 80ac5ca:       e8 d1 eb ff ff          call   80ab1a0 <uw_frame_state_for>
 80ac5cf:       89 c7                   mov    %eax,%edi
 80ac5d1:       b8 21 00 00 00          mov    $0x21,%eax
 80ac5d6:       89 f9                   mov    %edi,%ecx
 80ac5d8:       0f a3 c8                bt     %ecx,%eax
 80ac5db:       72 83                   jb     80ac560 <_Unwind_Backtrace+0x50>
 80ac5dd:       bf 03 00 00 00          mov    $0x3,%edi
 80ac5e2:       8d 65 f4                lea    -0xc(%ebp),%esp
 80ac5e5:       89 f8                   mov    %edi,%eax
 80ac5e7:       5b                      pop    %ebx
 80ac5e8:       5e                      pop    %esi
 80ac5e9:       5f                      pop    %edi
 80ac5ea:       5d                      pop    %ebp
 80ac5eb:       c3                      ret
 80ac5ec:       66 90                   xchg   %ax,%ax
 80ac5ee:       66 90                   xchg   %ax,%ax
 80ac5f0:       66 90                   xchg   %ax,%ax
 80ac5f2:       66 90                   xchg   %ax,%ax
 80ac5f4:       66 90                   xchg   %ax,%ax
 80ac5f6:       66 90                   xchg   %ax,%ax
 80ac5f8:       66 90                   xchg   %ax,%ax
 80ac5fa:       66 90                   xchg   %ax,%ax
 80ac5fc:       66 90                   xchg   %ax,%ax
 80ac5fe:       66 90                   xchg   %ax,%ax

080ac600 <btree_node_update_separator_after_split>:
 80ac600:       55                      push   %ebp
 80ac601:       57                      push   %edi
 80ac602:       89 c7                   mov    %eax,%edi
 80ac604:       56                      push   %esi
 80ac605:       53                      push   %ebx
 80ac606:       e8 d5 d1 f9 ff          call   80497e0 <__x86.get_pc_thunk.bx>
 80ac60b:       81 c3 e9 a9 03 00       add    $0x3a9e9,%ebx
 80ac611:       83 ec 1c                sub    $0x1c,%esp
 80ac614:       8b 68 04                mov    0x4(%eax),%ebp
 80ac617:       89 4c 24 08             mov    %ecx,0x8(%esp)
 80ac61b:       85 ed                   test   %ebp,%ebp
 80ac61d:       0f 84 7e 00 00 00       je     80ac6a1 <btree_node_update_separator_after_split+0xa1>
 80ac623:       31 c0                   xor    %eax,%eax
 80ac625:       eb 1d                   jmp    80ac644 <btree_node_update_separator_after_split+0x44>
 80ac627:       eb 17                   jmp    80ac640 <btree_node_update_separator_after_split+0x40>
 80ac629:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ac630:       00
 80ac631:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ac638:       00
 80ac639:       8d b4 26 00 00 00 00    lea    0x0(%esi,%eiz,1),%esi
 80ac640:       39 c5                   cmp    %eax,%ebp
 80ac642:       74 6c                   je     80ac6b0 <btree_node_update_separator_after_split+0xb0>
 80ac644:       8b 4c c7 0c             mov    0xc(%edi,%eax,8),%ecx
 80ac648:       89 c6                   mov    %eax,%esi
 80ac64a:       8d 40 01                lea    0x1(%eax),%eax
 80ac64d:       39 d1                   cmp    %edx,%ecx
 80ac64f:       72 ef                   jb     80ac640 <btree_node_update_separator_after_split+0x40>
 80ac651:       89 c2                   mov    %eax,%edx
 80ac653:       39 ee                   cmp    %ebp,%esi
 80ac655:       73 2c                   jae    80ac683 <btree_node_update_separator_after_split+0x83>
 80ac657:       89 44 24 0c             mov    %eax,0xc(%esp)
 80ac65b:       89 e8                   mov    %ebp,%eax
 80ac65d:       83 ec 04                sub    $0x4,%esp
 80ac660:       8d 0c f5 00 00 00 00    lea    0x0(,%esi,8),%ecx
 80ac667:       29 f0                   sub    %esi,%eax
 80ac669:       c1 e0 03                shl    $0x3,%eax
 80ac66c:       50                      push   %eax
 80ac66d:       8d 44 0f 0c             lea    0xc(%edi,%ecx,1),%eax
 80ac671:       50                      push   %eax
 80ac672:       8d 44 0f 14             lea    0x14(%edi,%ecx,1),%eax
 80ac676:       50                      push   %eax
 80ac677:       e8 24 e0 fa ff          call   805a6a0 <memmove>
 80ac67c:       8b 54 24 1c             mov    0x1c(%esp),%edx
 80ac680:       83 c4 10                add    $0x10,%esp
 80ac683:       8b 44 24 08             mov    0x8(%esp),%eax
 80ac687:       83 c5 01                add    $0x1,%ebp
 80ac68a:       89 44 f7 0c             mov    %eax,0xc(%edi,%esi,8)
 80ac68e:       8b 44 24 30             mov    0x30(%esp),%eax
 80ac692:       89 44 d7 10             mov    %eax,0x10(%edi,%edx,8)
 80ac696:       89 6f 04                mov    %ebp,0x4(%edi)
 80ac699:       83 c4 1c                add    $0x1c,%esp
 80ac69c:       5b                      pop    %ebx
 80ac69d:       5e                      pop    %esi
 80ac69e:       5f                      pop    %edi
 80ac69f:       5d                      pop    %ebp
 80ac6a0:       c3                      ret
 80ac6a1:       31 f6                   xor    %esi,%esi
 80ac6a3:       ba 01 00 00 00          mov    $0x1,%edx
 80ac6a8:       eb d9                   jmp    80ac683 <btree_node_update_separator_after_split+0x83>
 80ac6aa:       8d b6 00 00 00 00       lea    0x0(%esi),%esi
 80ac6b0:       8d 56 02                lea    0x2(%esi),%edx
 80ac6b3:       89 ee                   mov    %ebp,%esi
 80ac6b5:       eb cc                   jmp    80ac683 <btree_node_update_separator_after_split+0x83>
 80ac6b7:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ac6be:       00
 80ac6bf:       90                      nop

080ac6c0 <fde_unencoded_compare>:
 80ac6c0:       f3 0f 1e fb             endbr32
 80ac6c4:       8b 44 24 08             mov    0x8(%esp),%eax
 80ac6c8:       8b 48 08                mov    0x8(%eax),%ecx
 80ac6cb:       8b 44 24 0c             mov    0xc(%esp),%eax
 80ac6cf:       8b 50 08                mov    0x8(%eax),%edx
 80ac6d2:       b8 01 00 00 00          mov    $0x1,%eax
 80ac6d7:       39 ca                   cmp    %ecx,%edx
 80ac6d9:       72 04                   jb     80ac6df <fde_unencoded_compare+0x1f>
 80ac6db:       39 d1                   cmp    %edx,%ecx
 80ac6dd:       19 c0                   sbb    %eax,%eax
 80ac6df:       c3                      ret

080ac6e0 <fde_unencoded_extract>:
 80ac6e0:       f3 0f 1e fb             endbr32
 80ac6e4:       53                      push   %ebx
 80ac6e5:       8b 4c 24 14             mov    0x14(%esp),%ecx
 80ac6e9:       85 c9                   test   %ecx,%ecx
 80ac6eb:       7e 25                   jle    80ac712 <fde_unencoded_extract+0x32>
 80ac6ed:       8b 44 24 10             mov    0x10(%esp),%eax
 80ac6f1:       8b 54 24 0c             mov    0xc(%esp),%edx
 80ac6f5:       8d 1c 88                lea    (%eax,%ecx,4),%ebx
 80ac6f8:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ac6ff:       00
 80ac700:       8b 08                   mov    (%eax),%ecx
 80ac702:       83 c0 04                add    $0x4,%eax
 80ac705:       83 c2 04                add    $0x4,%edx
 80ac708:       8b 49 08                mov    0x8(%ecx),%ecx
 80ac70b:       89 4a fc                mov    %ecx,-0x4(%edx)
 80ac70e:       39 d8                   cmp    %ebx,%eax
 80ac710:       75 ee                   jne    80ac700 <fde_unencoded_extract+0x20>
 80ac712:       5b                      pop    %ebx
 80ac713:       c3                      ret
 80ac714:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ac71b:       00
 80ac71c:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi

080ac720 <frame_downheap>:
 80ac720:       55                      push   %ebp
 80ac721:       89 cd                   mov    %ecx,%ebp
 80ac723:       57                      push   %edi
 80ac724:       56                      push   %esi
 80ac725:       53                      push   %ebx
 80ac726:       83 ec 1c                sub    $0x1c,%esp
 80ac729:       89 54 24 08             mov    %edx,0x8(%esp)
 80ac72d:       8b 54 24 30             mov    0x30(%esp),%edx
 80ac731:       89 44 24 04             mov    %eax,0x4(%esp)
 80ac735:       8d 5c 12 01             lea    0x1(%edx,%edx,1),%ebx
 80ac739:       3b 5c 24 34             cmp    0x34(%esp),%ebx
 80ac73d:       0f 8d 8d 00 00 00       jge    80ac7d0 <frame_downheap+0xb0>
 80ac743:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ac74a:       00
 80ac74b:       2e 8d 74 26 00          lea    %cs:0x0(%esi,%eiz,1),%esi
 80ac750:       8d 04 9d 00 00 00 00    lea    0x0(,%ebx,4),%eax
 80ac757:       8d 7b 01                lea    0x1(%ebx),%edi
 80ac75a:       8d 74 05 00             lea    0x0(%ebp,%eax,1),%esi
 80ac75e:       3b 7c 24 34             cmp    0x34(%esp),%edi
 80ac762:       7d 2c                   jge    80ac790 <frame_downheap+0x70>
 80ac764:       8d 4c 05 04             lea    0x4(%ebp,%eax,1),%ecx
 80ac768:       89 54 24 30             mov    %edx,0x30(%esp)
 80ac76c:       83 ec 04                sub    $0x4,%esp
 80ac76f:       ff 31                   push   (%ecx)
 80ac771:       89 4c 24 14             mov    %ecx,0x14(%esp)
 80ac775:       ff 36                   push   (%esi)
 80ac777:       ff 74 24 10             push   0x10(%esp)
 80ac77b:       8b 44 24 18             mov    0x18(%esp),%eax
 80ac77f:       ff d0                   call   *%eax
 80ac781:       8b 4c 24 1c             mov    0x1c(%esp),%ecx
 80ac785:       8b 54 24 40             mov    0x40(%esp),%edx
 80ac789:       83 c4 10                add    $0x10,%esp
 80ac78c:       85 c0                   test   %eax,%eax
 80ac78e:       78 38                   js     80ac7c8 <frame_downheap+0xa8>
 80ac790:       89 df                   mov    %ebx,%edi
 80ac792:       83 ec 04                sub    $0x4,%esp
 80ac795:       8d 5c 95 00             lea    0x0(%ebp,%edx,4),%ebx
 80ac799:       ff 36                   push   (%esi)
 80ac79b:       ff 33                   push   (%ebx)
 80ac79d:       ff 74 24 10             push   0x10(%esp)
 80ac7a1:       8b 44 24 18             mov    0x18(%esp),%eax
 80ac7a5:       ff d0                   call   *%eax
 80ac7a7:       83 c4 10                add    $0x10,%esp
 80ac7aa:       85 c0                   test   %eax,%eax
 80ac7ac:       79 22                   jns    80ac7d0 <frame_downheap+0xb0>
 80ac7ae:       8b 03                   mov    (%ebx),%eax
 80ac7b0:       8b 16                   mov    (%esi),%edx
 80ac7b2:       89 13                   mov    %edx,(%ebx)
 80ac7b4:       8d 5c 3f 01             lea    0x1(%edi,%edi,1),%ebx
 80ac7b8:       89 06                   mov    %eax,(%esi)
 80ac7ba:       39 5c 24 34             cmp    %ebx,0x34(%esp)
 80ac7be:       7e 10                   jle    80ac7d0 <frame_downheap+0xb0>
 80ac7c0:       89 fa                   mov    %edi,%edx
 80ac7c2:       eb 8c                   jmp    80ac750 <frame_downheap+0x30>
 80ac7c4:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi
 80ac7c8:       89 ce                   mov    %ecx,%esi
 80ac7ca:       eb c6                   jmp    80ac792 <frame_downheap+0x72>
 80ac7cc:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi
 80ac7d0:       83 c4 1c                add    $0x1c,%esp
 80ac7d3:       5b                      pop    %ebx
 80ac7d4:       5e                      pop    %esi
 80ac7d5:       5f                      pop    %edi
 80ac7d6:       5d                      pop    %ebp
 80ac7d7:       c3                      ret
 80ac7d8:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ac7df:       00

080ac7e0 <version_lock_lock_exclusive>:
 80ac7e0:       55                      push   %ebp
 80ac7e1:       57                      push   %edi
 80ac7e2:       56                      push   %esi
 80ac7e3:       89 c6                   mov    %eax,%esi
 80ac7e5:       53                      push   %ebx
 80ac7e6:       e8 f5 cf f9 ff          call   80497e0 <__x86.get_pc_thunk.bx>
 80ac7eb:       81 c3 09 a8 03 00       add    $0x3a809,%ebx
 80ac7f1:       83 ec 0c                sub    $0xc,%esp
 80ac7f4:       8b 00                   mov    (%eax),%eax
 80ac7f6:       a8 01                   test   $0x1,%al
 80ac7f8:       75 16                   jne    80ac810 <version_lock_lock_exclusive+0x30>
 80ac7fa:       89 c2                   mov    %eax,%edx
 80ac7fc:       83 ca 01                or     $0x1,%edx
 80ac7ff:       f0 0f b1 16             lock cmpxchg %edx,(%esi)
 80ac803:       75 0b                   jne    80ac810 <version_lock_lock_exclusive+0x30>
 80ac805:       83 c4 0c                add    $0xc,%esp
 80ac808:       5b                      pop    %ebx
 80ac809:       5e                      pop    %esi
 80ac80a:       5f                      pop    %edi
 80ac80b:       5d                      pop    %ebp
 80ac80c:       c3                      ret
 80ac80d:       8d 76 00                lea    0x0(%esi),%esi
 80ac810:       83 ec 0c                sub    $0xc,%esp
 80ac813:       8d bb 7c 3c 00 00       lea    0x3c7c(%ebx),%edi
 80ac819:       8d ab 4c 3c 00 00       lea    0x3c4c(%ebx),%ebp
 80ac81f:       57                      push   %edi
 80ac820:       e8 5b cd fc ff          call   8079580 <___pthread_mutex_lock>
 80ac825:       83 c4 10                add    $0x10,%esp
 80ac828:       8b 06                   mov    (%esi),%eax
 80ac82a:       a8 01                   test   $0x1,%al
 80ac82c:       75 22                   jne    80ac850 <version_lock_lock_exclusive+0x70>
 80ac82e:       89 c2                   mov    %eax,%edx
 80ac830:       83 ca 01                or     $0x1,%edx
 80ac833:       f0 0f b1 16             lock cmpxchg %edx,(%esi)
 80ac837:       75 f1                   jne    80ac82a <version_lock_lock_exclusive+0x4a>
 80ac839:       83 ec 0c                sub    $0xc,%esp
 80ac83c:       57                      push   %edi
 80ac83d:       e8 1e d6 fc ff          call   8079e60 <___pthread_mutex_unlock>
 80ac842:       83 c4 10                add    $0x10,%esp
 80ac845:       83 c4 0c                add    $0xc,%esp
 80ac848:       5b                      pop    %ebx
 80ac849:       5e                      pop    %esi
 80ac84a:       5f                      pop    %edi
 80ac84b:       5d                      pop    %ebp
 80ac84c:       c3                      ret
 80ac84d:       8d 76 00                lea    0x0(%esi),%esi
 80ac850:       a8 02                   test   $0x2,%al
 80ac852:       75 0b                   jne    80ac85f <version_lock_lock_exclusive+0x7f>
 80ac854:       89 c2                   mov    %eax,%edx
 80ac856:       83 ca 02                or     $0x2,%edx
 80ac859:       f0 0f b1 16             lock cmpxchg %edx,(%esi)
 80ac85d:       75 cb                   jne    80ac82a <version_lock_lock_exclusive+0x4a>
 80ac85f:       83 ec 08                sub    $0x8,%esp
 80ac862:       57                      push   %edi
 80ac863:       55                      push   %ebp
 80ac864:       e8 f7 40 00 00          call   80b0960 <___pthread_cond_wait>
 80ac869:       83 c4 10                add    $0x10,%esp
 80ac86c:       8b 06                   mov    (%esi),%eax
 80ac86e:       eb ba                   jmp    80ac82a <version_lock_lock_exclusive+0x4a>

080ac870 <read_encoded_value_with_base>:
 80ac870:       55                      push   %ebp
 80ac871:       e8 25 30 fa ff          call   804f89b <__x86.get_pc_thunk.bp>
 80ac876:       81 c5 7e a7 03 00       add    $0x3a77e,%ebp
 80ac87c:       57                      push   %edi
 80ac87d:       56                      push   %esi
 80ac87e:       89 ce                   mov    %ecx,%esi
 80ac880:       53                      push   %ebx
 80ac881:       83 ec 1c                sub    $0x1c,%esp
 80ac884:       3c 50                   cmp    $0x50,%al
 80ac886:       74 58                   je     80ac8e0 <.L52+0x30>
 80ac888:       89 c3                   mov    %eax,%ebx
 80ac88a:       83 e0 0f                and    $0xf,%eax
 80ac88d:       3c 0c                   cmp    $0xc,%al
 80ac88f:       0f 87 7c cd f9 ff       ja     8049611 <read_encoded_value_with_base.cold>
 80ac895:       0f b6 c0                movzbl %al,%eax
 80ac898:       89 d7                   mov    %edx,%edi
 80ac89a:       8b 8c 85 10 6d fe ff    mov    -0x192f0(%ebp,%eax,4),%ecx
 80ac8a1:       01 e9                   add    %ebp,%ecx
 80ac8a3:       3e ff e1                notrack jmp *%ecx
 80ac8a6:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ac8ad:       00
 80ac8ae:       66 90                   xchg   %ax,%ax

080ac8b0 <.L52>:
 80ac8b0:       8b 16                   mov    (%esi),%edx
 80ac8b2:       8d 46 04                lea    0x4(%esi),%eax
 80ac8b5:       85 d2                   test   %edx,%edx
 80ac8b7:       74 11                   je     80ac8ca <.L52+0x1a>
 80ac8b9:       89 d9                   mov    %ebx,%ecx
 80ac8bb:       83 e1 70                and    $0x70,%ecx
 80ac8be:       80 f9 10                cmp    $0x10,%cl
 80ac8c1:       0f 44 fe                cmove  %esi,%edi
 80ac8c4:       01 fa                   add    %edi,%edx
 80ac8c6:       84 db                   test   %bl,%bl
 80ac8c8:       78 36                   js     80ac900 <.L52+0x50>
 80ac8ca:       8b 4c 24 30             mov    0x30(%esp),%ecx
 80ac8ce:       89 11                   mov    %edx,(%ecx)
 80ac8d0:       83 c4 1c                add    $0x1c,%esp
 80ac8d3:       5b                      pop    %ebx
 80ac8d4:       5e                      pop    %esi
 80ac8d5:       5f                      pop    %edi
 80ac8d6:       5d                      pop    %ebp
 80ac8d7:       c3                      ret
 80ac8d8:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ac8df:       00
 80ac8e0:       8d 41 03                lea    0x3(%ecx),%eax
 80ac8e3:       8b 4c 24 30             mov    0x30(%esp),%ecx
 80ac8e7:       83 e0 fc                and    $0xfffffffc,%eax
 80ac8ea:       8b 10                   mov    (%eax),%edx
 80ac8ec:       83 c0 04                add    $0x4,%eax
 80ac8ef:       89 11                   mov    %edx,(%ecx)
 80ac8f1:       83 c4 1c                add    $0x1c,%esp
 80ac8f4:       5b                      pop    %ebx
 80ac8f5:       5e                      pop    %esi
 80ac8f6:       5f                      pop    %edi
 80ac8f7:       5d                      pop    %ebp
 80ac8f8:       c3                      ret
 80ac8f9:       8d b4 26 00 00 00 00    lea    0x0(%esi,%eiz,1),%esi
 80ac900:       8b 12                   mov    (%edx),%edx
 80ac902:       8b 4c 24 30             mov    0x30(%esp),%ecx
 80ac906:       89 11                   mov    %edx,(%ecx)
 80ac908:       83 c4 1c                add    $0x1c,%esp
 80ac90b:       5b                      pop    %ebx
 80ac90c:       5e                      pop    %esi
 80ac90d:       5f                      pop    %edi
 80ac90e:       5d                      pop    %ebp
 80ac90f:       c3                      ret

080ac910 <.L50>:
 80ac910:       8b 16                   mov    (%esi),%edx
 80ac912:       8d 46 08                lea    0x8(%esi),%eax
 80ac915:       eb 9e                   jmp    80ac8b5 <.L52+0x5>
 80ac917:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ac91e:       00
 80ac91f:       90                      nop

080ac920 <.L57>:
 80ac920:       0f b7 16                movzwl (%esi),%edx
 80ac923:       8d 46 02                lea    0x2(%esi),%eax
 80ac926:       eb 8d                   jmp    80ac8b5 <.L52+0x5>
 80ac928:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ac92f:       00

080ac930 <.L53>:
 80ac930:       0f bf 16                movswl (%esi),%edx
 80ac933:       8d 46 02                lea    0x2(%esi),%eax
 80ac936:       e9 7a ff ff ff          jmp    80ac8b5 <.L52+0x5>
 80ac93b:       2e 8d 74 26 00          lea    %cs:0x0(%esi,%eiz,1),%esi

080ac940 <.L63>:
 80ac940:       89 5c 24 08             mov    %ebx,0x8(%esp)
 80ac944:       31 d2                   xor    %edx,%edx
 80ac946:       89 f0                   mov    %esi,%eax
 80ac948:       31 c9                   xor    %ecx,%ecx
 80ac94a:       89 d5                   mov    %edx,%ebp
 80ac94c:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ac953:       00
 80ac954:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ac95b:       00
 80ac95c:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi
 80ac960:       0f b6 10                movzbl (%eax),%edx
 80ac963:       83 c0 01                add    $0x1,%eax
 80ac966:       89 d3                   mov    %edx,%ebx
 80ac968:       83 e3 7f                and    $0x7f,%ebx
 80ac96b:       d3 e3                   shl    %cl,%ebx
 80ac96d:       83 c1 07                add    $0x7,%ecx
 80ac970:       09 dd                   or     %ebx,%ebp
 80ac972:       84 d2                   test   %dl,%dl
 80ac974:       78 ea                   js     80ac960 <.L63+0x20>
 80ac976:       8b 5c 24 08             mov    0x8(%esp),%ebx
 80ac97a:       89 ea                   mov    %ebp,%edx
 80ac97c:       e9 34 ff ff ff          jmp    80ac8b5 <.L52+0x5>
 80ac981:       8d b4 26 00 00 00 00    lea    0x0(%esi,%eiz,1),%esi

080ac988 <.L64>:
 80ac988:       89 5c 24 0c             mov    %ebx,0xc(%esp)
 80ac98c:       31 d2                   xor    %edx,%edx
 80ac98e:       89 f0                   mov    %esi,%eax
 80ac990:       31 c9                   xor    %ecx,%ecx
 80ac992:       89 d5                   mov    %edx,%ebp
 80ac994:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ac99b:       00
 80ac99c:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi
 80ac9a0:       0f b6 10                movzbl (%eax),%edx
 80ac9a3:       83 c0 01                add    $0x1,%eax
 80ac9a6:       89 d3                   mov    %edx,%ebx
 80ac9a8:       83 e3 7f                and    $0x7f,%ebx
 80ac9ab:       d3 e3                   shl    %cl,%ebx
 80ac9ad:       83 c1 07                add    $0x7,%ecx
 80ac9b0:       09 dd                   or     %ebx,%ebp
 80ac9b2:       84 d2                   test   %dl,%dl
 80ac9b4:       78 ea                   js     80ac9a0 <.L64+0x18>
 80ac9b6:       88 54 24 08             mov    %dl,0x8(%esp)
 80ac9ba:       8b 5c 24 0c             mov    0xc(%esp),%ebx
 80ac9be:       89 ea                   mov    %ebp,%edx
 80ac9c0:       0f b6 6c 24 08          movzbl 0x8(%esp),%ebp
 80ac9c5:       83 f9 1f                cmp    $0x1f,%ecx
 80ac9c8:       0f 87 e7 fe ff ff       ja     80ac8b5 <.L52+0x5>
 80ac9ce:       83 e5 40                and    $0x40,%ebp
 80ac9d1:       0f 84 de fe ff ff       je     80ac8b5 <.L52+0x5>
 80ac9d7:       bd ff ff ff ff          mov    $0xffffffff,%ebp
 80ac9dc:       d3 e5                   shl    %cl,%ebp
 80ac9de:       09 ea                   or     %ebp,%edx
 80ac9e0:       e9 d4 fe ff ff          jmp    80ac8b9 <.L52+0x9>
 80ac9e5:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ac9ec:       00
 80ac9ed:       8d 76 00                lea    0x0(%esi),%esi

080ac9f0 <fde_radixsort>:
 80ac9f0:       55                      push   %ebp
 80ac9f1:       57                      push   %edi
 80ac9f2:       56                      push   %esi
 80ac9f3:       e8 37 f4 f9 ff          call   804be2f <__x86.get_pc_thunk.si>
 80ac9f8:       81 c6 fc a5 03 00       add    $0x3a5fc,%esi
 80ac9fe:       53                      push   %ebx
 80ac9ff:       81 ec 5c 06 00 00       sub    $0x65c,%esp
 80aca05:       8b 59 04                mov    0x4(%ecx),%ebx
 80aca08:       8d bc 24 50 02 00 00    lea    0x250(%esp),%edi
 80aca0f:       89 44 24 1c             mov    %eax,0x1c(%esp)
 80aca13:       31 c0                   xor    %eax,%eax
 80aca15:       89 7c 24 30             mov    %edi,0x30(%esp)
 80aca19:       89 74 24 3c             mov    %esi,0x3c(%esp)
 80aca1d:       8d 71 08                lea    0x8(%ecx),%esi
 80aca20:       b9 00 01 00 00          mov    $0x100,%ecx
 80aca25:       f3 ab                   rep stos %eax,%es:(%edi)
 80aca27:       89 54 24 20             mov    %edx,0x20(%esp)
 80aca2b:       89 5c 24 10             mov    %ebx,0x10(%esp)
 80aca2f:       89 74 24 0c             mov    %esi,0xc(%esp)
 80aca33:       85 db                   test   %ebx,%ebx
 80aca35:       0f 84 fb 01 00 00       je     80acc36 <fde_radixsort+0x246>
 80aca3b:       8b 84 24 70 06 00 00    mov    0x670(%esp),%eax
 80aca42:       89 74 24 14             mov    %esi,0x14(%esp)
 80aca46:       8d 6c 24 4c             lea    0x4c(%esp),%ebp
 80aca4a:       c7 44 24 08 00 00 00    movl   $0x0,0x8(%esp)
 80aca51:       00
 80aca52:       83 c0 08                add    $0x8,%eax
 80aca55:       89 74 24 38             mov    %esi,0x38(%esp)
 80aca59:       89 44 24 0c             mov    %eax,0xc(%esp)
 80aca5d:       8d 44 24 50             lea    0x50(%esp),%eax
 80aca61:       89 44 24 2c             mov    %eax,0x2c(%esp)
 80aca65:       8d 84 24 50 06 00 00    lea    0x650(%esp),%eax
 80aca6c:       89 44 24 34             mov    %eax,0x34(%esp)
 80aca70:       89 6c 24 28             mov    %ebp,0x28(%esp)
 80aca74:       31 c9                   xor    %ecx,%ecx
 80aca76:       31 db                   xor    %ebx,%ebx
 80aca78:       31 f6                   xor    %esi,%esi
 80aca7a:       89 c8                   mov    %ecx,%eax
 80aca7c:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi
 80aca80:       8b 6c 24 10             mov    0x10(%esp),%ebp
 80aca84:       ba 80 00 00 00          mov    $0x80,%edx
 80aca89:       89 ef                   mov    %ebp,%edi
 80aca8b:       29 c7                   sub    %eax,%edi
 80aca8d:       39 d7                   cmp    %edx,%edi
 80aca8f:       0f 47 fa                cmova  %edx,%edi
 80aca92:       57                      push   %edi
 80aca93:       8b 4c 24 18             mov    0x18(%esp),%ecx
 80aca97:       8d 14 81                lea    (%ecx,%eax,4),%edx
 80aca9a:       89 44 24 1c             mov    %eax,0x1c(%esp)
 80aca9e:       52                      push   %edx
 80aca9f:       ff 74 24 34             push   0x34(%esp)
 80acaa3:       ff 74 24 28             push   0x28(%esp)
 80acaa7:       8b 44 24 30             mov    0x30(%esp),%eax
 80acaab:       ff d0                   call   *%eax
 80acaad:       89 5c 24 5c             mov    %ebx,0x5c(%esp)
 80acab1:       8b 44 24 28             mov    0x28(%esp),%eax
 80acab5:       83 c4 10                add    $0x10,%esp
 80acab8:       39 c5                   cmp    %eax,%ebp
 80acaba:       74 4c                   je     80acb08 <fde_radixsort+0x118>
 80acabc:       8b 4c 24 28             mov    0x28(%esp),%ecx
 80acac0:       89 7c 24 18             mov    %edi,0x18(%esp)
 80acac4:       89 44 24 24             mov    %eax,0x24(%esp)
 80acac8:       89 ca                   mov    %ecx,%edx
 80acaca:       8d 2c b9                lea    (%ecx,%edi,4),%ebp
 80acacd:       8b 4c 24 08             mov    0x8(%esp),%ecx
 80acad1:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80acad8:       00
 80acad9:       8d b4 26 00 00 00 00    lea    0x0(%esi,%eiz,1),%esi
 80acae0:       89 df                   mov    %ebx,%edi
 80acae2:       8b 5a 04                mov    0x4(%edx),%ebx
 80acae5:       89 d8                   mov    %ebx,%eax
 80acae7:       d3 e8                   shr    %cl,%eax
 80acae9:       0f b6 c0                movzbl %al,%eax
 80acaec:       83 84 84 50 02 00 00    addl   $0x1,0x250(%esp,%eax,4)
 80acaf3:       01
 80acaf4:       39 fb                   cmp    %edi,%ebx
 80acaf6:       83 d6 00                adc    $0x0,%esi
 80acaf9:       83 c2 04                add    $0x4,%edx
 80acafc:       39 d5                   cmp    %edx,%ebp
 80acafe:       75 e0                   jne    80acae0 <fde_radixsort+0xf0>
 80acb00:       8b 7c 24 18             mov    0x18(%esp),%edi
 80acb04:       8b 44 24 24             mov    0x24(%esp),%eax
 80acb08:       8b 5c bc 4c             mov    0x4c(%esp,%edi,4),%ebx
 80acb0c:       01 f8                   add    %edi,%eax
 80acb0e:       8b 7c 24 10             mov    0x10(%esp),%edi
 80acb12:       39 f8                   cmp    %edi,%eax
 80acb14:       0f 82 66 ff ff ff       jb     80aca80 <fde_radixsort+0x90>
 80acb1a:       8b 6c 24 28             mov    0x28(%esp),%ebp
 80acb1e:       85 f6                   test   %esi,%esi
 80acb20:       0f 84 3c 01 00 00       je     80acc62 <fde_radixsort+0x272>
 80acb26:       8b 44 24 30             mov    0x30(%esp),%eax
 80acb2a:       8b 5c 24 34             mov    0x34(%esp),%ebx
 80acb2e:       31 d2                   xor    %edx,%edx
 80acb30:       89 d1                   mov    %edx,%ecx
 80acb32:       83 c0 04                add    $0x4,%eax
 80acb35:       03 50 fc                add    -0x4(%eax),%edx
 80acb38:       89 48 fc                mov    %ecx,-0x4(%eax)
 80acb3b:       39 c3                   cmp    %eax,%ebx
 80acb3d:       75 f1                   jne    80acb30 <fde_radixsort+0x140>
 80acb3f:       31 ff                   xor    %edi,%edi
 80acb41:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80acb48:       00
 80acb49:       8d b4 26 00 00 00 00    lea    0x0(%esi,%eiz,1),%esi
 80acb50:       8b 74 24 10             mov    0x10(%esp),%esi
 80acb54:       ba 80 00 00 00          mov    $0x80,%edx
 80acb59:       8b 5c 24 14             mov    0x14(%esp),%ebx
 80acb5d:       29 fe                   sub    %edi,%esi
 80acb5f:       8d 1c bb                lea    (%ebx,%edi,4),%ebx
 80acb62:       39 d6                   cmp    %edx,%esi
 80acb64:       0f 47 f2                cmova  %edx,%esi
 80acb67:       56                      push   %esi
 80acb68:       53                      push   %ebx
 80acb69:       55                      push   %ebp
 80acb6a:       ff 74 24 28             push   0x28(%esp)
 80acb6e:       8b 4c 24 30             mov    0x30(%esp),%ecx
 80acb72:       ff d1                   call   *%ecx
 80acb74:       31 d2                   xor    %edx,%edx
 80acb76:       83 c4 10                add    $0x10,%esp
 80acb79:       39 7c 24 10             cmp    %edi,0x10(%esp)
 80acb7d:       74 75                   je     80acbf4 <fde_radixsort+0x204>
 80acb7f:       89 7c 24 18             mov    %edi,0x18(%esp)
 80acb83:       eb 3b                   jmp    80acbc0 <fde_radixsort+0x1d0>
 80acb85:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80acb8c:       00
 80acb8d:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80acb94:       00
 80acb95:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80acb9c:       00
 80acb9d:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80acba4:       00
 80acba5:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80acbac:       00
 80acbad:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80acbb4:       00
 80acbb5:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80acbbc:       00
 80acbbd:       8d 76 00                lea    0x0(%esi),%esi
 80acbc0:       8b 44 95 00             mov    0x0(%ebp,%edx,4),%eax
 80acbc4:       0f b6 4c 24 08          movzbl 0x8(%esp),%ecx
 80acbc9:       d3 e8                   shr    %cl,%eax
 80acbcb:       0f b6 c0                movzbl %al,%eax
 80acbce:       8b 8c 84 50 02 00 00    mov    0x250(%esp,%eax,4),%ecx
 80acbd5:       8d 79 01                lea    0x1(%ecx),%edi
 80acbd8:       89 bc 84 50 02 00 00    mov    %edi,0x250(%esp,%eax,4)
 80acbdf:       8b 04 93                mov    (%ebx,%edx,4),%eax
 80acbe2:       83 c2 01                add    $0x1,%edx
 80acbe5:       8b 7c 24 0c             mov    0xc(%esp),%edi
 80acbe9:       89 04 8f                mov    %eax,(%edi,%ecx,4)
 80acbec:       39 d6                   cmp    %edx,%esi
 80acbee:       75 d0                   jne    80acbc0 <fde_radixsort+0x1d0>
 80acbf0:       8b 7c 24 18             mov    0x18(%esp),%edi
 80acbf4:       8b 44 24 10             mov    0x10(%esp),%eax
 80acbf8:       01 f7                   add    %esi,%edi
 80acbfa:       39 c7                   cmp    %eax,%edi
 80acbfc:       0f 82 4e ff ff ff       jb     80acb50 <fde_radixsort+0x160>
 80acc02:       83 44 24 08 08          addl   $0x8,0x8(%esp)
 80acc07:       8b 44 24 08             mov    0x8(%esp),%eax
 80acc0b:       83 f8 20                cmp    $0x20,%eax
 80acc0e:       74 22                   je     80acc32 <fde_radixsort+0x242>
 80acc10:       8b 7c 24 30             mov    0x30(%esp),%edi
 80acc14:       31 c0                   xor    %eax,%eax
 80acc16:       b9 00 01 00 00          mov    $0x100,%ecx
 80acc1b:       8b 74 24 0c             mov    0xc(%esp),%esi
 80acc1f:       f3 ab                   rep stos %eax,%es:(%edi)
 80acc21:       8b 44 24 14             mov    0x14(%esp),%eax
 80acc25:       89 74 24 14             mov    %esi,0x14(%esp)
 80acc29:       89 44 24 0c             mov    %eax,0xc(%esp)
 80acc2d:       e9 3e fe ff ff          jmp    80aca70 <fde_radixsort+0x80>
 80acc32:       8b 74 24 38             mov    0x38(%esp),%esi
 80acc36:       8b 5c 24 0c             mov    0xc(%esp),%ebx
 80acc3a:       39 de                   cmp    %ebx,%esi
 80acc3c:       74 19                   je     80acc57 <fde_radixsort+0x267>
 80acc3e:       8b 44 24 10             mov    0x10(%esp),%eax
 80acc42:       83 ec 04                sub    $0x4,%esp
 80acc45:       c1 e0 02                shl    $0x2,%eax
 80acc48:       50                      push   %eax
 80acc49:       53                      push   %ebx
 80acc4a:       56                      push   %esi
 80acc4b:       8b 5c 24 4c             mov    0x4c(%esp),%ebx
 80acc4f:       e8 dc d9 fa ff          call   805a630 <memcpy>
 80acc54:       83 c4 10                add    $0x10,%esp
 80acc57:       81 c4 5c 06 00 00       add    $0x65c,%esp
 80acc5d:       5b                      pop    %ebx
 80acc5e:       5e                      pop    %esi
 80acc5f:       5f                      pop    %edi
 80acc60:       5d                      pop    %ebp
 80acc61:       c3                      ret
 80acc62:       8b 44 24 14             mov    0x14(%esp),%eax
 80acc66:       8b 74 24 38             mov    0x38(%esp),%esi
 80acc6a:       89 44 24 0c             mov    %eax,0xc(%esp)
 80acc6e:       eb c6                   jmp    80acc36 <fde_radixsort+0x246>

080acc70 <version_lock_unlock_exclusive>:
 80acc70:       56                      push   %esi
 80acc71:       53                      push   %ebx
 80acc72:       e8 69 cb f9 ff          call   80497e0 <__x86.get_pc_thunk.bx>
 80acc77:       81 c3 7d a3 03 00       add    $0x3a37d,%ebx
 80acc7d:       83 ec 04                sub    $0x4,%esp
 80acc80:       8b 10                   mov    (%eax),%edx
 80acc82:       83 c2 04                add    $0x4,%edx
 80acc85:       83 e2 fc                and    $0xfffffffc,%edx
 80acc88:       87 10                   xchg   %edx,(%eax)
 80acc8a:       83 e2 02                and    $0x2,%edx
 80acc8d:       75 09                   jne    80acc98 <version_lock_unlock_exclusive+0x28>
 80acc8f:       83 c4 04                add    $0x4,%esp
 80acc92:       5b                      pop    %ebx
 80acc93:       5e                      pop    %esi
 80acc94:       c3                      ret
 80acc95:       8d 76 00                lea    0x0(%esi),%esi
 80acc98:       83 ec 0c                sub    $0xc,%esp
 80acc9b:       8d b3 7c 3c 00 00       lea    0x3c7c(%ebx),%esi
 80acca1:       56                      push   %esi
 80acca2:       e8 d9 c8 fc ff          call   8079580 <___pthread_mutex_lock>
 80acca7:       8d 83 4c 3c 00 00       lea    0x3c4c(%ebx),%eax
 80accad:       89 04 24                mov    %eax,(%esp)
 80accb0:       e8 4b 31 00 00          call   80afe00 <___pthread_cond_broadcast>
 80accb5:       89 34 24                mov    %esi,(%esp)
 80accb8:       e8 a3 d1 fc ff          call   8079e60 <___pthread_mutex_unlock>
 80accbd:       83 c4 10                add    $0x10,%esp
 80accc0:       83 c4 04                add    $0x4,%esp
 80accc3:       5b                      pop    %ebx
 80accc4:       5e                      pop    %esi
 80accc5:       c3                      ret
 80accc6:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80acccd:       00
 80accce:       66 90                   xchg   %ax,%ax

080accd0 <btree_allocate_node>:
 80accd0:       55                      push   %ebp
 80accd1:       57                      push   %edi
 80accd2:       e8 5c f1 f9 ff          call   804be33 <__x86.get_pc_thunk.di>
 80accd7:       81 c7 1d a3 03 00       add    $0x3a31d,%edi
 80accdd:       56                      push   %esi
 80accde:       89 d6                   mov    %edx,%esi
 80acce0:       53                      push   %ebx
 80acce1:       8d 58 04                lea    0x4(%eax),%ebx
 80acce4:       83 ec 0c                sub    $0xc,%esp
 80acce7:       8b 0b                   mov    (%ebx),%ecx
 80acce9:       89 ca                   mov    %ecx,%edx
 80acceb:       85 c9                   test   %ecx,%ecx
 80acced:       74 50                   je     80acd3f <btree_allocate_node+0x6f>
 80accef:       8b 01                   mov    (%ecx),%eax
 80accf1:       a8 01                   test   $0x1,%al
 80accf3:       75 f2                   jne    80acce7 <btree_allocate_node+0x17>
 80accf5:       89 c5                   mov    %eax,%ebp
 80accf7:       83 cd 01                or     $0x1,%ebp
 80accfa:       f0 0f b1 29             lock cmpxchg %ebp,(%ecx)
 80accfe:       75 e7                   jne    80acce7 <btree_allocate_node+0x17>
 80acd00:       83 79 08 02             cmpl   $0x2,0x8(%ecx)
 80acd04:       75 2a                   jne    80acd30 <btree_allocate_node+0x60>
 80acd06:       8b 69 10                mov    0x10(%ecx),%ebp
 80acd09:       89 c8                   mov    %ecx,%eax
 80acd0b:       f0 0f b1 2b             lock cmpxchg %ebp,(%ebx)
 80acd0f:       75 1f                   jne    80acd30 <btree_allocate_node+0x60>
 80acd11:       83 f6 01                xor    $0x1,%esi
 80acd14:       c7 41 04 00 00 00 00    movl   $0x0,0x4(%ecx)
 80acd1b:       89 f0                   mov    %esi,%eax
 80acd1d:       0f b6 f0                movzbl %al,%esi
 80acd20:       89 d0                   mov    %edx,%eax
 80acd22:       89 71 08                mov    %esi,0x8(%ecx)
 80acd25:       83 c4 0c                add    $0xc,%esp
 80acd28:       5b                      pop    %ebx
 80acd29:       5e                      pop    %esi
 80acd2a:       5f                      pop    %edi
 80acd2b:       5d                      pop    %ebp
 80acd2c:       c3                      ret
 80acd2d:       8d 76 00                lea    0x0(%esi),%esi
 80acd30:       89 c8                   mov    %ecx,%eax
 80acd32:       e8 39 ff ff ff          call   80acc70 <version_lock_unlock_exclusive>
 80acd37:       8b 0b                   mov    (%ebx),%ecx
 80acd39:       89 ca                   mov    %ecx,%edx
 80acd3b:       85 c9                   test   %ecx,%ecx
 80acd3d:       75 b0                   jne    80accef <btree_allocate_node+0x1f>
 80acd3f:       83 ec 0c                sub    $0xc,%esp
 80acd42:       89 fb                   mov    %edi,%ebx
 80acd44:       83 f6 01                xor    $0x1,%esi
 80acd47:       68 84 00 00 00          push   $0x84
 80acd4c:       e8 bf a5 fa ff          call   8057310 <__libc_malloc>
 80acd51:       83 c4 10                add    $0x10,%esp
 80acd54:       c7 00 01 00 00 00       movl   $0x1,(%eax)
 80acd5a:       89 c2                   mov    %eax,%edx
 80acd5c:       c7 40 04 00 00 00 00    movl   $0x0,0x4(%eax)
 80acd63:       89 f0                   mov    %esi,%eax
 80acd65:       0f b6 f0                movzbl %al,%esi
 80acd68:       89 d0                   mov    %edx,%eax
 80acd6a:       89 72 08                mov    %esi,0x8(%edx)
 80acd6d:       83 c4 0c                add    $0xc,%esp
 80acd70:       5b                      pop    %ebx
 80acd71:       5e                      pop    %esi
 80acd72:       5f                      pop    %edi
 80acd73:       5d                      pop    %ebp
 80acd74:       c3                      ret
 80acd75:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80acd7c:       00
 80acd7d:       8d 76 00                lea    0x0(%esi),%esi

080acd80 <btree_handle_root_split.part.0>:
 80acd80:       55                      push   %ebp
 80acd81:       89 cd                   mov    %ecx,%ebp
 80acd83:       57                      push   %edi
 80acd84:       56                      push   %esi
 80acd85:       53                      push   %ebx
 80acd86:       89 d3                   mov    %edx,%ebx
 80acd88:       83 ec 0c                sub    $0xc,%esp
 80acd8b:       8b 12                   mov    (%edx),%edx
 80acd8d:       8b 52 08                mov    0x8(%edx),%edx
 80acd90:       85 d2                   test   %edx,%edx
 80acd92:       0f 94 c2                sete   %dl
 80acd95:       0f b6 d2                movzbl %dl,%edx
 80acd98:       e8 33 ff ff ff          call   80accd0 <btree_allocate_node>
 80acd9d:       89 c2                   mov    %eax,%edx
 80acd9f:       8b 03                   mov    (%ebx),%eax
 80acda1:       8d 7a 0c                lea    0xc(%edx),%edi
 80acda4:       8b 48 04                mov    0x4(%eax),%ecx
 80acda7:       8d 70 0c                lea    0xc(%eax),%esi
 80acdaa:       89 4a 04                mov    %ecx,0x4(%edx)
 80acdad:       b9 1e 00 00 00          mov    $0x1e,%ecx
 80acdb2:       f3 a5                   rep movsl %ds:(%esi),%es:(%edi)
 80acdb4:       c7 40 0c ff ff ff ff    movl   $0xffffffff,0xc(%eax)
 80acdbb:       89 50 10                mov    %edx,0x10(%eax)
 80acdbe:       c7 40 04 01 00 00 00    movl   $0x1,0x4(%eax)
 80acdc5:       c7 40 08 00 00 00 00    movl   $0x0,0x8(%eax)
 80acdcc:       89 45 00                mov    %eax,0x0(%ebp)
 80acdcf:       89 13                   mov    %edx,(%ebx)
 80acdd1:       83 c4 0c                add    $0xc,%esp
 80acdd4:       5b                      pop    %ebx
 80acdd5:       5e                      pop    %esi
 80acdd6:       5f                      pop    %edi
 80acdd7:       5d                      pop    %ebp
 80acdd8:       c3                      ret
 80acdd9:       8d b4 26 00 00 00 00    lea    0x0(%esi,%eiz,1),%esi

080acde0 <btree_insert.isra.0>:
 80acde0:       55                      push   %ebp
 80acde1:       57                      push   %edi
 80acde2:       56                      push   %esi
 80acde3:       53                      push   %ebx
 80acde4:       e8 f7 c9 f9 ff          call   80497e0 <__x86.get_pc_thunk.bx>
 80acde9:       81 c3 0b a2 03 00       add    $0x3a20b,%ebx
 80acdef:       83 ec 3c                sub    $0x3c,%esp
 80acdf2:       89 44 24 04             mov    %eax,0x4(%esp)
 80acdf6:       89 4c 24 0c             mov    %ecx,0xc(%esp)
 80acdfa:       89 5c 24 14             mov    %ebx,0x14(%esp)
 80acdfe:       8b 5c 24 50             mov    0x50(%esp),%ebx
 80ace02:       89 5c 24 10             mov    %ebx,0x10(%esp)
 80ace06:       85 c9                   test   %ecx,%ecx
 80ace08:       75 0e                   jne    80ace18 <btree_insert.isra.0+0x38>
 80ace0a:       83 c4 3c                add    $0x3c,%esp
 80ace0d:       5b                      pop    %ebx
 80ace0e:       5e                      pop    %esi
 80ace0f:       5f                      pop    %edi
 80ace10:       5d                      pop    %ebp
 80ace11:       c3                      ret
 80ace12:       8d b6 00 00 00 00       lea    0x0(%esi),%esi
 80ace18:       8d 58 08                lea    0x8(%eax),%ebx
 80ace1b:       89 c6                   mov    %eax,%esi
 80ace1d:       c7 44 24 2c 00 00 00    movl   $0x0,0x2c(%esp)
 80ace24:       00
 80ace25:       89 d7                   mov    %edx,%edi
 80ace27:       89 d8                   mov    %ebx,%eax
 80ace29:       e8 b2 f9 ff ff          call   80ac7e0 <version_lock_lock_exclusive>
 80ace2e:       8b 36                   mov    (%esi),%esi
 80ace30:       89 74 24 28             mov    %esi,0x28(%esp)
 80ace34:       85 f6                   test   %esi,%esi
 80ace36:       0f 84 6f 02 00 00       je     80ad0ab <btree_insert.isra.0+0x2cb>
 80ace3c:       89 f0                   mov    %esi,%eax
 80ace3e:       e8 9d f9 ff ff          call   80ac7e0 <version_lock_lock_exclusive>
 80ace43:       89 d8                   mov    %ebx,%eax
 80ace45:       e8 26 fe ff ff          call   80acc70 <version_lock_unlock_exclusive>
 80ace4a:       8b 5e 08                mov    0x8(%esi),%ebx
 80ace4d:       85 db                   test   %ebx,%ebx
 80ace4f:       0f 85 71 02 00 00       jne    80ad0c6 <btree_insert.isra.0+0x2e6>
 80ace55:       83 7e 04 0f             cmpl   $0xf,0x4(%esi)
 80ace59:       0f 84 8e 00 00 00       je     80aceed <btree_insert.isra.0+0x10d>
 80ace5f:       8b 74 24 28             mov    0x28(%esp),%esi
 80ace63:       c7 04 24 00 00 00 00    movl   $0x0,(%esp)
 80ace6a:       8b 46 04                mov    0x4(%esi),%eax
 80ace6d:       85 c0                   test   %eax,%eax
 80ace6f:       0f 84 46 03 00 00       je     80ad1bb <btree_insert.isra.0+0x3db>
 80ace75:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ace7c:       00
 80ace7d:       8d 76 00                lea    0x0(%esi),%esi
 80ace80:       31 ed                   xor    %ebp,%ebp
 80ace82:       eb 13                   jmp    80ace97 <btree_insert.isra.0+0xb7>
 80ace84:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ace8b:       00
 80ace8c:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi
 80ace90:       83 c5 01                add    $0x1,%ebp
 80ace93:       39 c5                   cmp    %eax,%ebp
 80ace95:       74 06                   je     80ace9d <btree_insert.isra.0+0xbd>
 80ace97:       39 7c ee 0c             cmp    %edi,0xc(%esi,%ebp,8)
 80ace9b:       72 f3                   jb     80ace90 <btree_insert.isra.0+0xb0>
 80ace9d:       8b 0c 24                mov    (%esp),%ecx
 80acea0:       85 c9                   test   %ecx,%ecx
 80acea2:       75 3c                   jne    80acee0 <btree_insert.isra.0+0x100>
 80acea4:       89 34 24                mov    %esi,(%esp)
 80acea7:       89 74 24 2c             mov    %esi,0x2c(%esp)
 80aceab:       8b 5c ee 0c             mov    0xc(%esi,%ebp,8),%ebx
 80aceaf:       8b 74 ee 10             mov    0x10(%esi,%ebp,8),%esi
 80aceb3:       89 f0                   mov    %esi,%eax
 80aceb5:       89 74 24 28             mov    %esi,0x28(%esp)
 80aceb9:       e8 22 f9 ff ff          call   80ac7e0 <version_lock_lock_exclusive>
 80acebe:       8b 56 08                mov    0x8(%esi),%edx
 80acec1:       85 d2                   test   %edx,%edx
 80acec3:       0f 85 f7 00 00 00       jne    80acfc0 <btree_insert.isra.0+0x1e0>
 80acec9:       8b 46 04                mov    0x4(%esi),%eax
 80acecc:       83 f8 0f                cmp    $0xf,%eax
 80acecf:       74 3f                   je     80acf10 <btree_insert.isra.0+0x130>
 80aced1:       31 ed                   xor    %ebp,%ebp
 80aced3:       85 c0                   test   %eax,%eax
 80aced5:       75 a9                   jne    80ace80 <btree_insert.isra.0+0xa0>
 80aced7:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80acede:       00
 80acedf:       90                      nop
 80acee0:       8b 04 24                mov    (%esp),%eax
 80acee3:       e8 88 fd ff ff          call   80acc70 <version_lock_unlock_exclusive>
 80acee8:       89 34 24                mov    %esi,(%esp)
 80aceeb:       eb ba                   jmp    80acea7 <btree_insert.isra.0+0xc7>
 80aceed:       8b 44 24 04             mov    0x4(%esp),%eax
 80acef1:       8d 4c 24 2c             lea    0x2c(%esp),%ecx
 80acef5:       8d 54 24 28             lea    0x28(%esp),%edx
 80acef9:       e8 82 fe ff ff          call   80acd80 <btree_handle_root_split.part.0>
 80acefe:       8b 44 24 2c             mov    0x2c(%esp),%eax
 80acf02:       89 04 24                mov    %eax,(%esp)
 80acf05:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80acf0c:       00
 80acf0d:       8d 76 00                lea    0x0(%esi),%esi
 80acf10:       8b 74 24 28             mov    0x28(%esp),%esi
 80acf14:       ba 01 00 00 00          mov    $0x1,%edx
 80acf19:       8b 46 04                mov    0x4(%esi),%eax
 80acf1c:       8b 44 c6 04             mov    0x4(%esi,%eax,8),%eax
 80acf20:       89 44 24 08             mov    %eax,0x8(%esp)
 80acf24:       8b 44 24 04             mov    0x4(%esp),%eax
 80acf28:       e8 a3 fd ff ff          call   80accd0 <btree_allocate_node>
 80acf2d:       8b 56 04                mov    0x4(%esi),%edx
 80acf30:       89 c3                   mov    %eax,%ebx
 80acf32:       89 d5                   mov    %edx,%ebp
 80acf34:       d1 ed                   shr    $1,%ebp
 80acf36:       29 ea                   sub    %ebp,%edx
 80acf38:       8d 0c ee                lea    (%esi,%ebp,8),%ecx
 80acf3b:       89 50 04                mov    %edx,0x4(%eax)
 80acf3e:       b8 00 00 00 00          mov    $0x0,%eax
 80acf43:       74 3a                   je     80acf7f <btree_insert.isra.0+0x19f>
 80acf45:       89 74 24 18             mov    %esi,0x18(%esp)
 80acf49:       89 7c 24 1c             mov    %edi,0x1c(%esp)
 80acf4d:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80acf54:       00
 80acf55:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80acf5c:       00
 80acf5d:       8d 76 00                lea    0x0(%esi),%esi
 80acf60:       8b 74 c1 0c             mov    0xc(%ecx,%eax,8),%esi
 80acf64:       8b 7c c1 10             mov    0x10(%ecx,%eax,8),%edi
 80acf68:       89 74 c3 0c             mov    %esi,0xc(%ebx,%eax,8)
 80acf6c:       89 7c c3 10             mov    %edi,0x10(%ebx,%eax,8)
 80acf70:       83 c0 01                add    $0x1,%eax
 80acf73:       39 c2                   cmp    %eax,%edx
 80acf75:       75 e9                   jne    80acf60 <btree_insert.isra.0+0x180>
 80acf77:       8b 74 24 18             mov    0x18(%esp),%esi
 80acf7b:       8b 7c 24 1c             mov    0x1c(%esp),%edi
 80acf7f:       83 ec 0c                sub    $0xc,%esp
 80acf82:       89 6e 04                mov    %ebp,0x4(%esi)
 80acf85:       8b 6c ee 04             mov    0x4(%esi,%ebp,8),%ebp
 80acf89:       53                      push   %ebx
 80acf8a:       8b 54 24 18             mov    0x18(%esp),%edx
 80acf8e:       8b 44 24 10             mov    0x10(%esp),%eax
 80acf92:       89 e9                   mov    %ebp,%ecx
 80acf94:       e8 67 f6 ff ff          call   80ac600 <btree_node_update_separator_after_split>
 80acf99:       83 c4 10                add    $0x10,%esp
 80acf9c:       39 fd                   cmp    %edi,%ebp
 80acf9e:       0f 82 f4 00 00 00       jb     80ad098 <btree_insert.isra.0+0x2b8>
 80acfa4:       89 d8                   mov    %ebx,%eax
 80acfa6:       e8 c5 fc ff ff          call   80acc70 <version_lock_unlock_exclusive>
 80acfab:       8b 46 04                mov    0x4(%esi),%eax
 80acfae:       85 c0                   test   %eax,%eax
 80acfb0:       0f 85 ca fe ff ff       jne    80ace80 <btree_insert.isra.0+0xa0>
 80acfb6:       31 ed                   xor    %ebp,%ebp
 80acfb8:       e9 e0 fe ff ff          jmp    80ace9d <btree_insert.isra.0+0xbd>
 80acfbd:       8d 76 00                lea    0x0(%esi),%esi
 80acfc0:       83 7e 04 0a             cmpl   $0xa,0x4(%esi)
 80acfc4:       0f 84 27 01 00 00       je     80ad0f1 <btree_insert.isra.0+0x311>
 80acfca:       8b 04 24                mov    (%esp),%eax
 80acfcd:       e8 9e fc ff ff          call   80acc70 <version_lock_unlock_exclusive>
 80acfd2:       8b 56 04                mov    0x4(%esi),%edx
 80acfd5:       85 d2                   test   %edx,%edx
 80acfd7:       0f 84 e8 01 00 00       je     80ad1c5 <btree_insert.isra.0+0x3e5>
 80acfdd:       8d 46 0c                lea    0xc(%esi),%eax
 80acfe0:       31 ed                   xor    %ebp,%ebp
 80acfe2:       eb 2a                   jmp    80ad00e <btree_insert.isra.0+0x22e>
 80acfe4:       eb 1a                   jmp    80ad000 <btree_insert.isra.0+0x220>
 80acfe6:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80acfed:       00
 80acfee:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80acff5:       00
 80acff6:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80acffd:       00
 80acffe:       66 90                   xchg   %ax,%ax
 80ad000:       83 c5 01                add    $0x1,%ebp
 80ad003:       83 c0 0c                add    $0xc,%eax
 80ad006:       39 ea                   cmp    %ebp,%edx
 80ad008:       0f 84 98 00 00 00       je     80ad0a6 <btree_insert.isra.0+0x2c6>
 80ad00e:       8b 48 04                mov    0x4(%eax),%ecx
 80ad011:       03 08                   add    (%eax),%ecx
 80ad013:       39 cf                   cmp    %ecx,%edi
 80ad015:       73 e9                   jae    80ad000 <btree_insert.isra.0+0x220>
 80ad017:       39 d5                   cmp    %edx,%ebp
 80ad019:       0f 83 af 01 00 00       jae    80ad1ce <btree_insert.isra.0+0x3ee>
 80ad01f:       8d 4c 2d 00             lea    0x0(%ebp,%ebp,1),%ecx
 80ad023:       8d 04 29                lea    (%ecx,%ebp,1),%eax
 80ad026:       8d 1c 85 00 00 00 00    lea    0x0(,%eax,4),%ebx
 80ad02d:       3b 7c 86 0c             cmp    0xc(%esi,%eax,4),%edi
 80ad031:       74 53                   je     80ad086 <btree_insert.isra.0+0x2a6>
 80ad033:       89 d0                   mov    %edx,%eax
 80ad035:       89 4c 24 04             mov    %ecx,0x4(%esp)
 80ad039:       83 ec 04                sub    $0x4,%esp
 80ad03c:       29 e8                   sub    %ebp,%eax
 80ad03e:       89 54 24 04             mov    %edx,0x4(%esp)
 80ad042:       8d 04 40                lea    (%eax,%eax,2),%eax
 80ad045:       c1 e0 02                shl    $0x2,%eax
 80ad048:       50                      push   %eax
 80ad049:       8d 44 6d 03             lea    0x3(%ebp,%ebp,2),%eax
 80ad04d:       8d 04 86                lea    (%esi,%eax,4),%eax
 80ad050:       50                      push   %eax
 80ad051:       8d 44 1e 18             lea    0x18(%esi,%ebx,1),%eax
 80ad055:       50                      push   %eax
 80ad056:       8b 5c 24 24             mov    0x24(%esp),%ebx
 80ad05a:       e8 41 d6 fa ff          call   805a6a0 <memmove>
 80ad05f:       8b 4c 24 14             mov    0x14(%esp),%ecx
 80ad063:       8b 54 24 10             mov    0x10(%esp),%edx
 80ad067:       83 c4 10                add    $0x10,%esp
 80ad06a:       8b 5c 24 0c             mov    0xc(%esp),%ebx
 80ad06e:       01 e9                   add    %ebp,%ecx
 80ad070:       83 c2 01                add    $0x1,%edx
 80ad073:       8d 04 8e                lea    (%esi,%ecx,4),%eax
 80ad076:       89 58 10                mov    %ebx,0x10(%eax)
 80ad079:       8b 5c 24 10             mov    0x10(%esp),%ebx
 80ad07d:       89 78 0c                mov    %edi,0xc(%eax)
 80ad080:       89 58 14                mov    %ebx,0x14(%eax)
 80ad083:       89 56 04                mov    %edx,0x4(%esi)
 80ad086:       83 c4 3c                add    $0x3c,%esp
 80ad089:       89 f0                   mov    %esi,%eax
 80ad08b:       5b                      pop    %ebx
 80ad08c:       5e                      pop    %esi
 80ad08d:       5f                      pop    %edi
 80ad08e:       5d                      pop    %ebp
 80ad08f:       e9 dc fb ff ff          jmp    80acc70 <version_lock_unlock_exclusive>
 80ad094:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi
 80ad098:       89 f0                   mov    %esi,%eax
 80ad09a:       89 de                   mov    %ebx,%esi
 80ad09c:       e8 cf fb ff ff          call   80acc70 <version_lock_unlock_exclusive>
 80ad0a1:       e9 05 ff ff ff          jmp    80acfab <btree_insert.isra.0+0x1cb>
 80ad0a6:       8d 0c 12                lea    (%edx,%edx,1),%ecx
 80ad0a9:       eb bf                   jmp    80ad06a <btree_insert.isra.0+0x28a>
 80ad0ab:       8b 6c 24 04             mov    0x4(%esp),%ebp
 80ad0af:       31 d2                   xor    %edx,%edx
 80ad0b1:       89 e8                   mov    %ebp,%eax
 80ad0b3:       e8 18 fc ff ff          call   80accd0 <btree_allocate_node>
 80ad0b8:       89 44 24 28             mov    %eax,0x28(%esp)
 80ad0bc:       89 c6                   mov    %eax,%esi
 80ad0be:       89 45 00                mov    %eax,0x0(%ebp)
 80ad0c1:       e9 7d fd ff ff          jmp    80ace43 <btree_insert.isra.0+0x63>
 80ad0c6:       83 7e 04 0a             cmpl   $0xa,0x4(%esi)
 80ad0ca:       0f 85 02 ff ff ff       jne    80acfd2 <btree_insert.isra.0+0x1f2>
 80ad0d0:       8b 44 24 04             mov    0x4(%esp),%eax
 80ad0d4:       8d 4c 24 2c             lea    0x2c(%esp),%ecx
 80ad0d8:       8d 54 24 28             lea    0x28(%esp),%edx
 80ad0dc:       bb ff ff ff ff          mov    $0xffffffff,%ebx
 80ad0e1:       e8 9a fc ff ff          call   80acd80 <btree_handle_root_split.part.0>
 80ad0e6:       8b 44 24 2c             mov    0x2c(%esp),%eax
 80ad0ea:       8b 74 24 28             mov    0x28(%esp),%esi
 80ad0ee:       89 04 24                mov    %eax,(%esp)
 80ad0f1:       8b 44 24 04             mov    0x4(%esp),%eax
 80ad0f5:       31 d2                   xor    %edx,%edx
 80ad0f7:       e8 d4 fb ff ff          call   80accd0 <btree_allocate_node>
 80ad0fc:       89 c1                   mov    %eax,%ecx
 80ad0fe:       89 44 24 04             mov    %eax,0x4(%esp)
 80ad102:       8b 46 04                mov    0x4(%esi),%eax
 80ad105:       89 c2                   mov    %eax,%edx
 80ad107:       89 c5                   mov    %eax,%ebp
 80ad109:       d1 ea                   shr    $1,%edx
 80ad10b:       29 d5                   sub    %edx,%ebp
 80ad10d:       89 54 24 08             mov    %edx,0x8(%esp)
 80ad111:       89 69 04                mov    %ebp,0x4(%ecx)
 80ad114:       74 4c                   je     80ad162 <btree_insert.isra.0+0x382>
 80ad116:       8b 44 24 08             mov    0x8(%esp),%eax
 80ad11a:       89 5c 24 18             mov    %ebx,0x18(%esp)
 80ad11e:       31 c9                   xor    %ecx,%ecx
 80ad120:       8d 44 40 03             lea    0x3(%eax,%eax,2),%eax
 80ad124:       8d 14 86                lea    (%esi,%eax,4),%edx
 80ad127:       8b 44 24 04             mov    0x4(%esp),%eax
 80ad12b:       83 c0 0c                add    $0xc,%eax
 80ad12e:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ad135:       00
 80ad136:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ad13d:       00
 80ad13e:       66 90                   xchg   %ax,%ax
 80ad140:       8b 1a                   mov    (%edx),%ebx
 80ad142:       83 c1 01                add    $0x1,%ecx
 80ad145:       83 c2 0c                add    $0xc,%edx
 80ad148:       83 c0 0c                add    $0xc,%eax
 80ad14b:       89 58 f4                mov    %ebx,-0xc(%eax)
 80ad14e:       8b 5a f8                mov    -0x8(%edx),%ebx
 80ad151:       89 58 f8                mov    %ebx,-0x8(%eax)
 80ad154:       8b 5a fc                mov    -0x4(%edx),%ebx
 80ad157:       89 58 fc                mov    %ebx,-0x4(%eax)
 80ad15a:       39 cd                   cmp    %ecx,%ebp
 80ad15c:       75 e2                   jne    80ad140 <btree_insert.isra.0+0x360>
 80ad15e:       8b 5c 24 18             mov    0x18(%esp),%ebx
 80ad162:       8b 44 24 08             mov    0x8(%esp),%eax
 80ad166:       83 ec 0c                sub    $0xc,%esp
 80ad169:       89 46 04                mov    %eax,0x4(%esi)
 80ad16c:       8b 44 24 10             mov    0x10(%esp),%eax
 80ad170:       8b 50 0c                mov    0xc(%eax),%edx
 80ad173:       89 54 24 14             mov    %edx,0x14(%esp)
 80ad177:       8d 6a ff                lea    -0x1(%edx),%ebp
 80ad17a:       89 da                   mov    %ebx,%edx
 80ad17c:       50                      push   %eax
 80ad17d:       8b 44 24 10             mov    0x10(%esp),%eax
 80ad181:       89 e9                   mov    %ebp,%ecx
 80ad183:       e8 78 f4 ff ff          call   80ac600 <btree_node_update_separator_after_split>
 80ad188:       83 c4 10                add    $0x10,%esp
 80ad18b:       39 fd                   cmp    %edi,%ebp
 80ad18d:       73 21                   jae    80ad1b0 <btree_insert.isra.0+0x3d0>
 80ad18f:       8b 5c 24 04             mov    0x4(%esp),%ebx
 80ad193:       89 f0                   mov    %esi,%eax
 80ad195:       89 5c 24 28             mov    %ebx,0x28(%esp)
 80ad199:       89 de                   mov    %ebx,%esi
 80ad19b:       e8 d0 fa ff ff          call   80acc70 <version_lock_unlock_exclusive>
 80ad1a0:       8b 04 24                mov    (%esp),%eax
 80ad1a3:       85 c0                   test   %eax,%eax
 80ad1a5:       0f 84 27 fe ff ff       je     80acfd2 <btree_insert.isra.0+0x1f2>
 80ad1ab:       e9 1a fe ff ff          jmp    80acfca <btree_insert.isra.0+0x1ea>
 80ad1b0:       8b 44 24 04             mov    0x4(%esp),%eax
 80ad1b4:       e8 b7 fa ff ff          call   80acc70 <version_lock_unlock_exclusive>
 80ad1b9:       eb e5                   jmp    80ad1a0 <btree_insert.isra.0+0x3c0>
 80ad1bb:       89 34 24                mov    %esi,(%esp)
 80ad1be:       31 ed                   xor    %ebp,%ebp
 80ad1c0:       e9 e2 fc ff ff          jmp    80acea7 <btree_insert.isra.0+0xc7>
 80ad1c5:       31 ed                   xor    %ebp,%ebp
 80ad1c7:       31 c9                   xor    %ecx,%ecx
 80ad1c9:       e9 9c fe ff ff          jmp    80ad06a <btree_insert.isra.0+0x28a>
 80ad1ce:       8d 4c 2d 00             lea    0x0(%ebp,%ebp,1),%ecx
 80ad1d2:       e9 93 fe ff ff          jmp    80ad06a <btree_insert.isra.0+0x28a>
 80ad1d7:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ad1de:       00
 80ad1df:       90                      nop

080ad1e0 <get_cie_encoding>:
 80ad1e0:       57                      push   %edi
 80ad1e1:       8d 78 09                lea    0x9(%eax),%edi
 80ad1e4:       56                      push   %esi
 80ad1e5:       89 c6                   mov    %eax,%esi
 80ad1e7:       53                      push   %ebx
 80ad1e8:       e8 f3 c5 f9 ff          call   80497e0 <__x86.get_pc_thunk.bx>
 80ad1ed:       81 c3 07 9e 03 00       add    $0x39e07,%ebx
 80ad1f3:       83 ec 1c                sub    $0x1c,%esp
 80ad1f6:       57                      push   %edi
 80ad1f7:       e8 84 d5 fa ff          call   805a780 <strlen>
 80ad1fc:       0f b6 4e 08             movzbl 0x8(%esi),%ecx
 80ad200:       83 c4 10                add    $0x10,%esp
 80ad203:       8d 44 07 01             lea    0x1(%edi,%eax,1),%eax
 80ad207:       80 f9 03                cmp    $0x3,%cl
 80ad20a:       0f 87 d8 00 00 00       ja     80ad2e8 <get_cie_encoding+0x108>
 80ad210:       80 7e 09 7a             cmpb   $0x7a,0x9(%esi)
 80ad214:       74 1a                   je     80ad230 <get_cie_encoding+0x50>
 80ad216:       31 d2                   xor    %edx,%edx
 80ad218:       83 c4 10                add    $0x10,%esp
 80ad21b:       89 d0                   mov    %edx,%eax
 80ad21d:       5b                      pop    %ebx
 80ad21e:       5e                      pop    %esi
 80ad21f:       5f                      pop    %edi
 80ad220:       c3                      ret
 80ad221:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ad228:       00
 80ad229:       8d b4 26 00 00 00 00    lea    0x0(%esi,%eiz,1),%esi
 80ad230:       83 c0 01                add    $0x1,%eax
 80ad233:       80 78 ff 00             cmpb   $0x0,-0x1(%eax)
 80ad237:       78 f7                   js     80ad230 <get_cie_encoding+0x50>
 80ad239:       8d b4 26 00 00 00 00    lea    0x0(%esi,%eiz,1),%esi
 80ad240:       89 c2                   mov    %eax,%edx
 80ad242:       83 c0 01                add    $0x1,%eax
 80ad245:       80 78 ff 00             cmpb   $0x0,-0x1(%eax)
 80ad249:       78 f5                   js     80ad240 <get_cie_encoding+0x60>
 80ad24b:       80 f9 01                cmp    $0x1,%cl
 80ad24e:       0f 84 8c 00 00 00       je     80ad2e0 <get_cie_encoding+0x100>
 80ad254:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ad25b:       00
 80ad25c:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi
 80ad260:       83 c0 01                add    $0x1,%eax
 80ad263:       80 78 ff 00             cmpb   $0x0,-0x1(%eax)
 80ad267:       78 f7                   js     80ad260 <get_cie_encoding+0x80>
 80ad269:       8d 5e 0a                lea    0xa(%esi),%ebx
 80ad26c:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi
 80ad270:       83 c0 01                add    $0x1,%eax
 80ad273:       80 78 ff 00             cmpb   $0x0,-0x1(%eax)
 80ad277:       78 f7                   js     80ad270 <get_cie_encoding+0x90>
 80ad279:       0f b6 56 0a             movzbl 0xa(%esi),%edx
 80ad27d:       8d 74 24 0c             lea    0xc(%esp),%esi
 80ad281:       80 fa 52                cmp    $0x52,%dl
 80ad284:       75 27                   jne    80ad2ad <get_cie_encoding+0xcd>
 80ad286:       eb 4c                   jmp    80ad2d4 <get_cie_encoding+0xf4>
 80ad288:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ad28f:       00
 80ad290:       80 fa 4c                cmp    $0x4c,%dl
 80ad293:       74 09                   je     80ad29e <get_cie_encoding+0xbe>
 80ad295:       80 fa 42                cmp    $0x42,%dl
 80ad298:       0f 85 78 ff ff ff       jne    80ad216 <get_cie_encoding+0x36>
 80ad29e:       0f b6 53 01             movzbl 0x1(%ebx),%edx
 80ad2a2:       83 c3 01                add    $0x1,%ebx
 80ad2a5:       83 c0 01                add    $0x1,%eax
 80ad2a8:       80 fa 52                cmp    $0x52,%dl
 80ad2ab:       74 27                   je     80ad2d4 <get_cie_encoding+0xf4>
 80ad2ad:       80 fa 50                cmp    $0x50,%dl
 80ad2b0:       75 de                   jne    80ad290 <get_cie_encoding+0xb0>
 80ad2b2:       83 ec 0c                sub    $0xc,%esp
 80ad2b5:       8d 48 01                lea    0x1(%eax),%ecx
 80ad2b8:       0f b6 00                movzbl (%eax),%eax
 80ad2bb:       31 d2                   xor    %edx,%edx
 80ad2bd:       56                      push   %esi
 80ad2be:       83 c3 01                add    $0x1,%ebx
 80ad2c1:       83 e0 7f                and    $0x7f,%eax
 80ad2c4:       e8 a7 f5 ff ff          call   80ac870 <read_encoded_value_with_base>
 80ad2c9:       0f b6 13                movzbl (%ebx),%edx
 80ad2cc:       83 c4 10                add    $0x10,%esp
 80ad2cf:       80 fa 52                cmp    $0x52,%dl
 80ad2d2:       75 d9                   jne    80ad2ad <get_cie_encoding+0xcd>
 80ad2d4:       0f b6 10                movzbl (%eax),%edx
 80ad2d7:       83 c4 10                add    $0x10,%esp
 80ad2da:       5b                      pop    %ebx
 80ad2db:       5e                      pop    %esi
 80ad2dc:       89 d0                   mov    %edx,%eax
 80ad2de:       5f                      pop    %edi
 80ad2df:       c3                      ret
 80ad2e0:       8d 42 02                lea    0x2(%edx),%eax
 80ad2e3:       eb 84                   jmp    80ad269 <get_cie_encoding+0x89>
 80ad2e5:       8d 76 00                lea    0x0(%esi),%esi
 80ad2e8:       80 38 04                cmpb   $0x4,(%eax)
 80ad2eb:       ba ff 00 00 00          mov    $0xff,%edx
 80ad2f0:       0f 85 22 ff ff ff       jne    80ad218 <get_cie_encoding+0x38>
 80ad2f6:       80 78 01 00             cmpb   $0x0,0x1(%eax)
 80ad2fa:       0f 85 18 ff ff ff       jne    80ad218 <get_cie_encoding+0x38>
 80ad300:       83 c0 02                add    $0x2,%eax
 80ad303:       e9 08 ff ff ff          jmp    80ad210 <get_cie_encoding+0x30>
 80ad308:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ad30f:       00

080ad310 <btree_remove>:
 80ad310:       55                      push   %ebp
 80ad311:       57                      push   %edi
 80ad312:       56                      push   %esi
 80ad313:       89 c6                   mov    %eax,%esi
 80ad315:       53                      push   %ebx
 80ad316:       8d 58 08                lea    0x8(%eax),%ebx
 80ad319:       83 ec 3c                sub    $0x3c,%esp
 80ad31c:       89 44 24 18             mov    %eax,0x18(%esp)
 80ad320:       89 d8                   mov    %ebx,%eax
 80ad322:       89 54 24 14             mov    %edx,0x14(%esp)
 80ad326:       e8 b5 f4 ff ff          call   80ac7e0 <version_lock_lock_exclusive>
 80ad32b:       8b 3e                   mov    (%esi),%edi
 80ad32d:       85 ff                   test   %edi,%edi
 80ad32f:       0f 84 13 07 00 00       je     80ada48 <btree_remove+0x738>
 80ad335:       89 f8                   mov    %edi,%eax
 80ad337:       e8 a4 f4 ff ff          call   80ac7e0 <version_lock_lock_exclusive>
 80ad33c:       89 d8                   mov    %ebx,%eax
 80ad33e:       e8 2d f9 ff ff          call   80acc70 <version_lock_unlock_exclusive>
 80ad343:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ad34a:       00
 80ad34b:       2e 8d 74 26 00          lea    %cs:0x0(%esi,%eiz,1),%esi
 80ad350:       8b 5f 08                mov    0x8(%edi),%ebx
 80ad353:       85 db                   test   %ebx,%ebx
 80ad355:       8b 5f 04                mov    0x4(%edi),%ebx
 80ad358:       0f 85 e2 01 00 00       jne    80ad540 <btree_remove+0x230>
 80ad35e:       85 db                   test   %ebx,%ebx
 80ad360:       74 1d                   je     80ad37f <btree_remove+0x6f>
 80ad362:       8b 54 24 14             mov    0x14(%esp),%edx
 80ad366:       31 c0                   xor    %eax,%eax
 80ad368:       eb 0d                   jmp    80ad377 <btree_remove+0x67>
 80ad36a:       8d b6 00 00 00 00       lea    0x0(%esi),%esi
 80ad370:       83 c0 01                add    $0x1,%eax
 80ad373:       39 c3                   cmp    %eax,%ebx
 80ad375:       74 08                   je     80ad37f <btree_remove+0x6f>
 80ad377:       39 54 c7 0c             cmp    %edx,0xc(%edi,%eax,8)
 80ad37b:       72 f3                   jb     80ad370 <btree_remove+0x60>
 80ad37d:       89 c3                   mov    %eax,%ebx
 80ad37f:       8b 74 df 10             mov    0x10(%edi,%ebx,8),%esi
 80ad383:       89 f0                   mov    %esi,%eax
 80ad385:       e8 56 f4 ff ff          call   80ac7e0 <version_lock_lock_exclusive>
 80ad38a:       8b 4e 08                mov    0x8(%esi),%ecx
 80ad38d:       8b 56 04                mov    0x4(%esi),%edx
 80ad390:       b8 05 00 00 00          mov    $0x5,%eax
 80ad395:       85 c9                   test   %ecx,%ecx
 80ad397:       75 05                   jne    80ad39e <btree_remove+0x8e>
 80ad399:       b8 07 00 00 00          mov    $0x7,%eax
 80ad39e:       39 c2                   cmp    %eax,%edx
 80ad3a0:       0f 83 52 02 00 00       jae    80ad5f8 <btree_remove+0x2e8>
 80ad3a6:       85 db                   test   %ebx,%ebx
 80ad3a8:       0f 84 62 02 00 00       je     80ad610 <btree_remove+0x300>
 80ad3ae:       8d 73 ff                lea    -0x1(%ebx),%esi
 80ad3b1:       8d 43 01                lea    0x1(%ebx),%eax
 80ad3b4:       8b 4c f7 10             mov    0x10(%edi,%esi,8),%ecx
 80ad3b8:       89 4c 24 08             mov    %ecx,0x8(%esp)
 80ad3bc:       3b 47 04                cmp    0x4(%edi),%eax
 80ad3bf:       73 10                   jae    80ad3d1 <btree_remove+0xc1>
 80ad3c1:       8b 6c c7 10             mov    0x10(%edi,%eax,8),%ebp
 80ad3c5:       8b 41 04                mov    0x4(%ecx),%eax
 80ad3c8:       39 45 04                cmp    %eax,0x4(%ebp)
 80ad3cb:       0f 82 42 02 00 00       jb     80ad613 <btree_remove+0x303>
 80ad3d1:       8b 44 24 08             mov    0x8(%esp),%eax
 80ad3d5:       8b 6c df 10             mov    0x10(%edi,%ebx,8),%ebp
 80ad3d9:       89 f3                   mov    %esi,%ebx
 80ad3db:       e8 00 f4 ff ff          call   80ac7e0 <version_lock_lock_exclusive>
 80ad3e0:       8b 44 24 08             mov    0x8(%esp),%eax
 80ad3e4:       8b 55 04                mov    0x4(%ebp),%edx
 80ad3e7:       8b 70 04                mov    0x4(%eax),%esi
 80ad3ea:       8b 40 08                mov    0x8(%eax),%eax
 80ad3ed:       89 54 24 0c             mov    %edx,0xc(%esp)
 80ad3f1:       89 74 24 10             mov    %esi,0x10(%esp)
 80ad3f5:       01 d6                   add    %edx,%esi
 80ad3f7:       85 c0                   test   %eax,%eax
 80ad3f9:       0f 84 31 02 00 00       je     80ad630 <btree_remove+0x320>
 80ad3ff:       83 fe 0a                cmp    $0xa,%esi
 80ad402:       0f 87 58 03 00 00       ja     80ad760 <btree_remove+0x450>
 80ad408:       8b 4f 04                mov    0x4(%edi),%ecx
 80ad40b:       83 f9 02                cmp    $0x2,%ecx
 80ad40e:       0f 84 34 07 00 00       je     80adb48 <btree_remove+0x838>
 80ad414:       85 d2                   test   %edx,%edx
 80ad416:       0f 84 98 00 00 00       je     80ad4b4 <btree_remove+0x1a4>
 80ad41c:       8b 74 24 10             mov    0x10(%esp),%esi
 80ad420:       8b 4c 24 08             mov    0x8(%esp),%ecx
 80ad424:       89 5c 24 10             mov    %ebx,0x10(%esp)
 80ad428:       31 d2                   xor    %edx,%edx
 80ad42a:       89 7c 24 1c             mov    %edi,0x1c(%esp)
 80ad42e:       8d 04 76                lea    (%esi,%esi,2),%eax
 80ad431:       83 c6 01                add    $0x1,%esi
 80ad434:       89 74 24 0c             mov    %esi,0xc(%esp)
 80ad438:       8b 74 24 08             mov    0x8(%esp),%esi
 80ad43c:       8d 0c 81                lea    (%ecx,%eax,4),%ecx
 80ad43f:       31 c0                   xor    %eax,%eax
 80ad441:       8b 7c 24 0c             mov    0xc(%esp),%edi
 80ad445:       eb 39                   jmp    80ad480 <btree_remove+0x170>
 80ad447:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ad44e:       00
 80ad44f:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ad456:       00
 80ad457:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ad45e:       00
 80ad45f:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ad466:       00
 80ad467:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ad46e:       00
 80ad46f:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ad476:       00
 80ad477:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ad47e:       00
 80ad47f:       90                      nop
 80ad480:       8d 1c 17                lea    (%edi,%edx,1),%ebx
 80ad483:       83 c2 01                add    $0x1,%edx
 80ad486:       89 5e 04                mov    %ebx,0x4(%esi)
 80ad489:       8b 5c 05 0c             mov    0xc(%ebp,%eax,1),%ebx
 80ad48d:       89 5c 01 0c             mov    %ebx,0xc(%ecx,%eax,1)
 80ad491:       8b 5c 05 10             mov    0x10(%ebp,%eax,1),%ebx
 80ad495:       89 5c 01 10             mov    %ebx,0x10(%ecx,%eax,1)
 80ad499:       8b 5c 05 14             mov    0x14(%ebp,%eax,1),%ebx
 80ad49d:       89 5c 01 14             mov    %ebx,0x14(%ecx,%eax,1)
 80ad4a1:       83 c0 0c                add    $0xc,%eax
 80ad4a4:       3b 55 04                cmp    0x4(%ebp),%edx
 80ad4a7:       75 d7                   jne    80ad480 <btree_remove+0x170>
 80ad4a9:       8b 7c 24 1c             mov    0x1c(%esp),%edi
 80ad4ad:       8b 5c 24 10             mov    0x10(%esp),%ebx
 80ad4b1:       8b 4f 04                mov    0x4(%edi),%ecx
 80ad4b4:       8b 44 df 14             mov    0x14(%edi,%ebx,8),%eax
 80ad4b8:       89 44 df 0c             mov    %eax,0xc(%edi,%ebx,8)
 80ad4bc:       8d 43 02                lea    0x2(%ebx),%eax
 80ad4bf:       83 c3 03                add    $0x3,%ebx
 80ad4c2:       39 c8                   cmp    %ecx,%eax
 80ad4c4:       73 32                   jae    80ad4f8 <btree_remove+0x1e8>
 80ad4c6:       eb 18                   jmp    80ad4e0 <btree_remove+0x1d0>
 80ad4c8:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ad4cf:       00
 80ad4d0:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ad4d7:       00
 80ad4d8:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ad4df:       00
 80ad4e0:       8b 44 df 04             mov    0x4(%edi,%ebx,8),%eax
 80ad4e4:       8b 54 df 08             mov    0x8(%edi,%ebx,8),%edx
 80ad4e8:       89 44 df fc             mov    %eax,-0x4(%edi,%ebx,8)
 80ad4ec:       89 d8                   mov    %ebx,%eax
 80ad4ee:       89 14 df                mov    %edx,(%edi,%ebx,8)
 80ad4f1:       83 c3 01                add    $0x1,%ebx
 80ad4f4:       39 c8                   cmp    %ecx,%eax
 80ad4f6:       75 e8                   jne    80ad4e0 <btree_remove+0x1d0>
 80ad4f8:       83 e9 01                sub    $0x1,%ecx
 80ad4fb:       8b 44 24 18             mov    0x18(%esp),%eax
 80ad4ff:       89 4f 04                mov    %ecx,0x4(%edi)
 80ad502:       c7 45 08 02 00 00 00    movl   $0x2,0x8(%ebp)
 80ad509:       8d 50 04                lea    0x4(%eax),%edx
 80ad50c:       8b 40 04                mov    0x4(%eax),%eax
 80ad50f:       89 45 10                mov    %eax,0x10(%ebp)
 80ad512:       f0 0f b1 2a             lock cmpxchg %ebp,(%edx)
 80ad516:       75 f7                   jne    80ad50f <btree_remove+0x1ff>
 80ad518:       89 e8                   mov    %ebp,%eax
 80ad51a:       e8 51 f7 ff ff          call   80acc70 <version_lock_unlock_exclusive>
 80ad51f:       89 f8                   mov    %edi,%eax
 80ad521:       e8 4a f7 ff ff          call   80acc70 <version_lock_unlock_exclusive>
 80ad526:       8b 7c 24 08             mov    0x8(%esp),%edi
 80ad52a:       8b 5f 08                mov    0x8(%edi),%ebx
 80ad52d:       85 db                   test   %ebx,%ebx
 80ad52f:       8b 5f 04                mov    0x4(%edi),%ebx
 80ad532:       0f 84 26 fe ff ff       je     80ad35e <btree_remove+0x4e>
 80ad538:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ad53f:       00
 80ad540:       85 db                   test   %ebx,%ebx
 80ad542:       0f 84 18 05 00 00       je     80ada60 <btree_remove+0x750>
 80ad548:       8b 74 24 14             mov    0x14(%esp),%esi
 80ad54c:       8d 57 0c                lea    0xc(%edi),%edx
 80ad54f:       31 c0                   xor    %eax,%eax
 80ad551:       eb 1b                   jmp    80ad56e <btree_remove+0x25e>
 80ad553:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ad55a:       00
 80ad55b:       2e 8d 74 26 00          lea    %cs:0x0(%esi,%eiz,1),%esi
 80ad560:       83 c0 01                add    $0x1,%eax
 80ad563:       83 c2 0c                add    $0xc,%edx
 80ad566:       39 c3                   cmp    %eax,%ebx
 80ad568:       0f 84 f2 04 00 00       je     80ada60 <btree_remove+0x750>
 80ad56e:       8b 4a 04                mov    0x4(%edx),%ecx
 80ad571:       03 0a                   add    (%edx),%ecx
 80ad573:       39 ce                   cmp    %ecx,%esi
 80ad575:       73 e9                   jae    80ad560 <btree_remove+0x250>
 80ad577:       39 d8                   cmp    %ebx,%eax
 80ad579:       0f 83 e1 04 00 00       jae    80ada60 <btree_remove+0x750>
 80ad57f:       8d 14 40                lea    (%eax,%eax,2),%edx
 80ad582:       8b 74 24 14             mov    0x14(%esp),%esi
 80ad586:       8d 14 97                lea    (%edi,%edx,4),%edx
 80ad589:       39 72 0c                cmp    %esi,0xc(%edx)
 80ad58c:       0f 85 ce 04 00 00       jne    80ada60 <btree_remove+0x750>
 80ad592:       83 c0 01                add    $0x1,%eax
 80ad595:       8b 72 14                mov    0x14(%edx),%esi
 80ad598:       39 d8                   cmp    %ebx,%eax
 80ad59a:       73 40                   jae    80ad5dc <btree_remove+0x2cc>
 80ad59c:       8d 14 40                lea    (%eax,%eax,2),%edx
 80ad59f:       8d 14 97                lea    (%edi,%edx,4),%edx
 80ad5a2:       eb 1c                   jmp    80ad5c0 <btree_remove+0x2b0>
 80ad5a4:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ad5ab:       00
 80ad5ac:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ad5b3:       00
 80ad5b4:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ad5bb:       00
 80ad5bc:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi
 80ad5c0:       8b 4a 0c                mov    0xc(%edx),%ecx
 80ad5c3:       83 c0 01                add    $0x1,%eax
 80ad5c6:       83 c2 0c                add    $0xc,%edx
 80ad5c9:       89 4a f4                mov    %ecx,-0xc(%edx)
 80ad5cc:       8b 4a 04                mov    0x4(%edx),%ecx
 80ad5cf:       89 4a f8                mov    %ecx,-0x8(%edx)
 80ad5d2:       8b 4a 08                mov    0x8(%edx),%ecx
 80ad5d5:       89 4a fc                mov    %ecx,-0x4(%edx)
 80ad5d8:       39 c3                   cmp    %eax,%ebx
 80ad5da:       75 e4                   jne    80ad5c0 <btree_remove+0x2b0>
 80ad5dc:       83 eb 01                sub    $0x1,%ebx
 80ad5df:       89 f8                   mov    %edi,%eax
 80ad5e1:       89 5f 04                mov    %ebx,0x4(%edi)
 80ad5e4:       e8 87 f6 ff ff          call   80acc70 <version_lock_unlock_exclusive>
 80ad5e9:       83 c4 3c                add    $0x3c,%esp
 80ad5ec:       89 f0                   mov    %esi,%eax
 80ad5ee:       5b                      pop    %ebx
 80ad5ef:       5e                      pop    %esi
 80ad5f0:       5f                      pop    %edi
 80ad5f1:       5d                      pop    %ebp
 80ad5f2:       c3                      ret
 80ad5f3:       2e 8d 74 26 00          lea    %cs:0x0(%esi,%eiz,1),%esi
 80ad5f8:       89 f8                   mov    %edi,%eax
 80ad5fa:       89 f7                   mov    %esi,%edi
 80ad5fc:       e8 6f f6 ff ff          call   80acc70 <version_lock_unlock_exclusive>
 80ad601:       e9 4a fd ff ff          jmp    80ad350 <btree_remove+0x40>
 80ad606:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ad60d:       00
 80ad60e:       66 90                   xchg   %ax,%ax
 80ad610:       8b 6f 18                mov    0x18(%edi),%ebp
 80ad613:       8b 44 df 10             mov    0x10(%edi,%ebx,8),%eax
 80ad617:       89 44 24 08             mov    %eax,0x8(%esp)
 80ad61b:       89 e8                   mov    %ebp,%eax
 80ad61d:       e8 be f1 ff ff          call   80ac7e0 <version_lock_lock_exclusive>
 80ad622:       e9 b9 fd ff ff          jmp    80ad3e0 <btree_remove+0xd0>
 80ad627:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ad62e:       00
 80ad62f:       90                      nop
 80ad630:       83 fe 0f                cmp    $0xf,%esi
 80ad633:       77 5b                   ja     80ad690 <btree_remove+0x380>
 80ad635:       8b 4f 04                mov    0x4(%edi),%ecx
 80ad638:       83 f9 02                cmp    $0x2,%ecx
 80ad63b:       0f 84 2a 04 00 00       je     80ada6b <btree_remove+0x75b>
 80ad641:       8b 54 24 10             mov    0x10(%esp),%edx
 80ad645:       8b 74 24 0c             mov    0xc(%esp),%esi
 80ad649:       83 c2 01                add    $0x1,%edx
 80ad64c:       85 f6                   test   %esi,%esi
 80ad64e:       0f 84 60 fe ff ff       je     80ad4b4 <btree_remove+0x1a4>
 80ad654:       89 5c 24 0c             mov    %ebx,0xc(%esp)
 80ad658:       8b 4c 24 08             mov    0x8(%esp),%ecx
 80ad65c:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi
 80ad660:       89 51 04                mov    %edx,0x4(%ecx)
 80ad663:       8b 5c c5 0c             mov    0xc(%ebp,%eax,8),%ebx
 80ad667:       8b 74 c5 10             mov    0x10(%ebp,%eax,8),%esi
 80ad66b:       83 c0 01                add    $0x1,%eax
 80ad66e:       89 5c d1 04             mov    %ebx,0x4(%ecx,%edx,8)
 80ad672:       89 74 d1 08             mov    %esi,0x8(%ecx,%edx,8)
 80ad676:       83 c2 01                add    $0x1,%edx
 80ad679:       3b 45 04                cmp    0x4(%ebp),%eax
 80ad67c:       75 e2                   jne    80ad660 <btree_remove+0x350>
 80ad67e:       8b 5c 24 0c             mov    0xc(%esp),%ebx
 80ad682:       8b 4f 04                mov    0x4(%edi),%ecx
 80ad685:       e9 2a fe ff ff          jmp    80ad4b4 <btree_remove+0x1a4>
 80ad68a:       8d b6 00 00 00 00       lea    0x0(%esi),%esi
 80ad690:       8b 4c 24 0c             mov    0xc(%esp),%ecx
 80ad694:       8b 54 24 10             mov    0x10(%esp),%edx
 80ad698:       39 d1                   cmp    %edx,%ecx
 80ad69a:       0f 83 b1 02 00 00       jae    80ad951 <btree_remove+0x641>
 80ad6a0:       89 d6                   mov    %edx,%esi
 80ad6a2:       29 ce                   sub    %ecx,%esi
 80ad6a4:       d1 ee                   shr    $1,%esi
 80ad6a6:       85 c9                   test   %ecx,%ecx
 80ad6a8:       74 36                   je     80ad6e0 <btree_remove+0x3d0>
 80ad6aa:       8b 4c 24 0c             mov    0xc(%esp),%ecx
 80ad6ae:       89 6c 24 1c             mov    %ebp,0x1c(%esp)
 80ad6b2:       89 7c 24 20             mov    %edi,0x20(%esp)
 80ad6b6:       8d 54 cd 04             lea    0x4(%ebp,%ecx,8),%edx
 80ad6ba:       31 c9                   xor    %ecx,%ecx
 80ad6bc:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi
 80ad6c0:       8b 3a                   mov    (%edx),%edi
 80ad6c2:       8b 6a 04                mov    0x4(%edx),%ebp
 80ad6c5:       83 c1 01                add    $0x1,%ecx
 80ad6c8:       89 3c f2                mov    %edi,(%edx,%esi,8)
 80ad6cb:       89 6c f2 04             mov    %ebp,0x4(%edx,%esi,8)
 80ad6cf:       83 ea 08                sub    $0x8,%edx
 80ad6d2:       39 4c 24 0c             cmp    %ecx,0xc(%esp)
 80ad6d6:       75 e8                   jne    80ad6c0 <btree_remove+0x3b0>
 80ad6d8:       8b 6c 24 1c             mov    0x1c(%esp),%ebp
 80ad6dc:       8b 7c 24 20             mov    0x20(%esp),%edi
 80ad6e0:       8b 4c 24 10             mov    0x10(%esp),%ecx
 80ad6e4:       8b 54 24 08             mov    0x8(%esp),%edx
 80ad6e8:       29 f1                   sub    %esi,%ecx
 80ad6ea:       8d 14 ca                lea    (%edx,%ecx,8),%edx
 80ad6ed:       85 f6                   test   %esi,%esi
 80ad6ef:       0f 84 0e 05 00 00       je     80adc03 <btree_remove+0x8f3>
 80ad6f5:       89 5c 24 0c             mov    %ebx,0xc(%esp)
 80ad6f9:       89 4c 24 10             mov    %ecx,0x10(%esp)
 80ad6fd:       8d 76 00                lea    0x0(%esi),%esi
 80ad700:       8b 4c c2 0c             mov    0xc(%edx,%eax,8),%ecx
 80ad704:       8b 5c c2 10             mov    0x10(%edx,%eax,8),%ebx
 80ad708:       89 4c c5 0c             mov    %ecx,0xc(%ebp,%eax,8)
 80ad70c:       89 5c c5 10             mov    %ebx,0x10(%ebp,%eax,8)
 80ad710:       83 c0 01                add    $0x1,%eax
 80ad713:       39 f0                   cmp    %esi,%eax
 80ad715:       75 e9                   jne    80ad700 <btree_remove+0x3f0>
 80ad717:       8b 4c 24 10             mov    0x10(%esp),%ecx
 80ad71b:       8b 74 24 08             mov    0x8(%esp),%esi
 80ad71f:       8b 5c 24 0c             mov    0xc(%esp),%ebx
 80ad723:       89 4e 04                mov    %ecx,0x4(%esi)
 80ad726:       01 45 04                add    %eax,0x4(%ebp)
 80ad729:       8b 74 24 08             mov    0x8(%esp),%esi
 80ad72d:       8b 46 04                mov    0x4(%esi),%eax
 80ad730:       8b 74 c6 04             mov    0x4(%esi,%eax,8),%esi
 80ad734:       89 74 df 0c             mov    %esi,0xc(%edi,%ebx,8)
 80ad738:       89 f8                   mov    %edi,%eax
 80ad73a:       e8 31 f5 ff ff          call   80acc70 <version_lock_unlock_exclusive>
 80ad73f:       8b 44 24 14             mov    0x14(%esp),%eax
 80ad743:       39 c6                   cmp    %eax,%esi
 80ad745:       0f 82 ed 02 00 00       jb     80ada38 <btree_remove+0x728>
 80ad74b:       89 e8                   mov    %ebp,%eax
 80ad74d:       e8 1e f5 ff ff          call   80acc70 <version_lock_unlock_exclusive>
 80ad752:       e9 cf fd ff ff          jmp    80ad526 <btree_remove+0x216>
 80ad757:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ad75e:       00
 80ad75f:       90                      nop
 80ad760:       8b 74 24 0c             mov    0xc(%esp),%esi
 80ad764:       8b 54 24 10             mov    0x10(%esp),%edx
 80ad768:       39 d6                   cmp    %edx,%esi
 80ad76a:       0f 83 cd 00 00 00       jae    80ad83d <btree_remove+0x52d>
 80ad770:       29 f2                   sub    %esi,%edx
 80ad772:       d1 ea                   shr    $1,%edx
 80ad774:       89 54 24 1c             mov    %edx,0x1c(%esp)
 80ad778:       85 f6                   test   %esi,%esi
 80ad77a:       74 4a                   je     80ad7c6 <btree_remove+0x4b6>
 80ad77c:       8b 74 24 0c             mov    0xc(%esp),%esi
 80ad780:       8b 4c 24 1c             mov    0x1c(%esp),%ecx
 80ad784:       89 6c 24 20             mov    %ebp,0x20(%esp)
 80ad788:       8d 14 76                lea    (%esi,%esi,2),%edx
 80ad78b:       8d 0c 49                lea    (%ecx,%ecx,2),%ecx
 80ad78e:       31 f6                   xor    %esi,%esi
 80ad790:       8d 54 95 00             lea    0x0(%ebp,%edx,4),%edx
 80ad794:       8b 6c 24 0c             mov    0xc(%esp),%ebp
 80ad798:       89 44 24 0c             mov    %eax,0xc(%esp)
 80ad79c:       c1 e1 02                shl    $0x2,%ecx
 80ad79f:       90                      nop
 80ad7a0:       8b 02                   mov    (%edx),%eax
 80ad7a2:       83 c6 01                add    $0x1,%esi
 80ad7a5:       83 ea 0c                sub    $0xc,%edx
 80ad7a8:       89 44 0a 0c             mov    %eax,0xc(%edx,%ecx,1)
 80ad7ac:       8b 42 10                mov    0x10(%edx),%eax
 80ad7af:       89 44 0a 10             mov    %eax,0x10(%edx,%ecx,1)
 80ad7b3:       8b 42 14                mov    0x14(%edx),%eax
 80ad7b6:       89 44 0a 14             mov    %eax,0x14(%edx,%ecx,1)
 80ad7ba:       39 f5                   cmp    %esi,%ebp
 80ad7bc:       75 e2                   jne    80ad7a0 <btree_remove+0x490>
 80ad7be:       8b 6c 24 20             mov    0x20(%esp),%ebp
 80ad7c2:       8b 44 24 0c             mov    0xc(%esp),%eax
 80ad7c6:       8b 54 24 1c             mov    0x1c(%esp),%edx
 80ad7ca:       85 d2                   test   %edx,%edx
 80ad7cc:       74 58                   je     80ad826 <btree_remove+0x516>
 80ad7ce:       8b 4c 24 1c             mov    0x1c(%esp),%ecx
 80ad7d2:       29 4c 24 10             sub    %ecx,0x10(%esp)
 80ad7d6:       8b 74 24 10             mov    0x10(%esp),%esi
 80ad7da:       89 44 24 20             mov    %eax,0x20(%esp)
 80ad7de:       89 6c 24 0c             mov    %ebp,0xc(%esp)
 80ad7e2:       8d 54 76 03             lea    0x3(%esi,%esi,2),%edx
 80ad7e6:       8b 74 24 08             mov    0x8(%esp),%esi
 80ad7ea:       8d 0c 96                lea    (%esi,%edx,4),%ecx
 80ad7ed:       8d 55 0c                lea    0xc(%ebp),%edx
 80ad7f0:       8b 6c 24 1c             mov    0x1c(%esp),%ebp
 80ad7f4:       31 f6                   xor    %esi,%esi
 80ad7f6:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ad7fd:       00
 80ad7fe:       66 90                   xchg   %ax,%ax
 80ad800:       8b 01                   mov    (%ecx),%eax
 80ad802:       83 c6 01                add    $0x1,%esi
 80ad805:       83 c1 0c                add    $0xc,%ecx
 80ad808:       83 c2 0c                add    $0xc,%edx
 80ad80b:       89 42 f4                mov    %eax,-0xc(%edx)
 80ad80e:       8b 41 f8                mov    -0x8(%ecx),%eax
 80ad811:       89 42 f8                mov    %eax,-0x8(%edx)
 80ad814:       8b 41 fc                mov    -0x4(%ecx),%eax
 80ad817:       89 42 fc                mov    %eax,-0x4(%edx)
 80ad81a:       39 ee                   cmp    %ebp,%esi
 80ad81c:       75 e2                   jne    80ad800 <btree_remove+0x4f0>
 80ad81e:       8b 6c 24 0c             mov    0xc(%esp),%ebp
 80ad822:       8b 44 24 20             mov    0x20(%esp),%eax
 80ad826:       8b 4c 24 08             mov    0x8(%esp),%ecx
 80ad82a:       8b 74 24 10             mov    0x10(%esp),%esi
 80ad82e:       8b 54 24 1c             mov    0x1c(%esp),%edx
 80ad832:       89 71 04                mov    %esi,0x4(%ecx)
 80ad835:       03 55 04                add    0x4(%ebp),%edx
 80ad838:       e9 fd 00 00 00          jmp    80ad93a <btree_remove+0x62a>
 80ad83d:       8b 54 24 0c             mov    0xc(%esp),%edx
 80ad841:       8b 4c 24 10             mov    0x10(%esp),%ecx
 80ad845:       29 ca                   sub    %ecx,%edx
 80ad847:       d1 ea                   shr    $1,%edx
 80ad849:       89 54 24 1c             mov    %edx,0x1c(%esp)
 80ad84d:       74 57                   je     80ad8a6 <btree_remove+0x596>
 80ad84f:       8b 74 24 10             mov    0x10(%esp),%esi
 80ad853:       89 44 24 24             mov    %eax,0x24(%esp)
 80ad857:       8d 4d 0c                lea    0xc(%ebp),%ecx
 80ad85a:       89 6c 24 20             mov    %ebp,0x20(%esp)
 80ad85e:       8b 6c 24 1c             mov    0x1c(%esp),%ebp
 80ad862:       8d 54 76 03             lea    0x3(%esi,%esi,2),%edx
 80ad866:       8b 74 24 08             mov    0x8(%esp),%esi
 80ad86a:       8d 14 96                lea    (%esi,%edx,4),%edx
 80ad86d:       31 f6                   xor    %esi,%esi
 80ad86f:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ad876:       00
 80ad877:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ad87e:       00
 80ad87f:       90                      nop
 80ad880:       8b 01                   mov    (%ecx),%eax
 80ad882:       83 c6 01                add    $0x1,%esi
 80ad885:       83 c1 0c                add    $0xc,%ecx
 80ad888:       83 c2 0c                add    $0xc,%edx
 80ad88b:       89 42 f4                mov    %eax,-0xc(%edx)
 80ad88e:       8b 41 f8                mov    -0x8(%ecx),%eax
 80ad891:       89 42 f8                mov    %eax,-0x8(%edx)
 80ad894:       8b 41 fc                mov    -0x4(%ecx),%eax
 80ad897:       89 42 fc                mov    %eax,-0x4(%edx)
 80ad89a:       39 ee                   cmp    %ebp,%esi
 80ad89c:       75 e2                   jne    80ad880 <btree_remove+0x570>
 80ad89e:       8b 6c 24 20             mov    0x20(%esp),%ebp
 80ad8a2:       8b 44 24 24             mov    0x24(%esp),%eax
 80ad8a6:       8b 74 24 0c             mov    0xc(%esp),%esi
 80ad8aa:       8b 54 24 1c             mov    0x1c(%esp),%edx
 80ad8ae:       89 f1                   mov    %esi,%ecx
 80ad8b0:       29 d1                   sub    %edx,%ecx
 80ad8b2:       8b 54 24 10             mov    0x10(%esp),%edx
 80ad8b6:       89 4c 24 20             mov    %ecx,0x20(%esp)
 80ad8ba:       8b 4c 24 1c             mov    0x1c(%esp),%ecx
 80ad8be:       01 ca                   add    %ecx,%edx
 80ad8c0:       89 54 24 0c             mov    %edx,0xc(%esp)
 80ad8c4:       39 ce                   cmp    %ecx,%esi
 80ad8c6:       74 5e                   je     80ad926 <btree_remove+0x616>
 80ad8c8:       8b 4c 24 1c             mov    0x1c(%esp),%ecx
 80ad8cc:       89 44 24 24             mov    %eax,0x24(%esp)
 80ad8d0:       8d 55 0c                lea    0xc(%ebp),%edx
 80ad8d3:       31 f6                   xor    %esi,%esi
 80ad8d5:       89 6c 24 10             mov    %ebp,0x10(%esp)
 80ad8d9:       8b 6c 24 20             mov    0x20(%esp),%ebp
 80ad8dd:       8d 0c 49                lea    (%ecx,%ecx,2),%ecx
 80ad8e0:       c1 e1 02                shl    $0x2,%ecx
 80ad8e3:       eb 1b                   jmp    80ad900 <btree_remove+0x5f0>
 80ad8e5:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ad8ec:       00
 80ad8ed:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ad8f4:       00
 80ad8f5:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ad8fc:       00
 80ad8fd:       8d 76 00                lea    0x0(%esi),%esi
 80ad900:       8b 04 0a                mov    (%edx,%ecx,1),%eax
 80ad903:       83 c6 01                add    $0x1,%esi
 80ad906:       83 c2 0c                add    $0xc,%edx
 80ad909:       89 42 f4                mov    %eax,-0xc(%edx)
 80ad90c:       8b 44 0a f8             mov    -0x8(%edx,%ecx,1),%eax
 80ad910:       89 42 f8                mov    %eax,-0x8(%edx)
 80ad913:       8b 44 0a fc             mov    -0x4(%edx,%ecx,1),%eax
 80ad917:       89 42 fc                mov    %eax,-0x4(%edx)
 80ad91a:       39 ee                   cmp    %ebp,%esi
 80ad91c:       75 e2                   jne    80ad900 <btree_remove+0x5f0>
 80ad91e:       8b 6c 24 10             mov    0x10(%esp),%ebp
 80ad922:       8b 44 24 24             mov    0x24(%esp),%eax
 80ad926:       8b 4c 24 08             mov    0x8(%esp),%ecx
 80ad92a:       8b 74 24 0c             mov    0xc(%esp),%esi
 80ad92e:       89 71 04                mov    %esi,0x4(%ecx)
 80ad931:       8b 4c 24 1c             mov    0x1c(%esp),%ecx
 80ad935:       8b 55 04                mov    0x4(%ebp),%edx
 80ad938:       29 ca                   sub    %ecx,%edx
 80ad93a:       89 55 04                mov    %edx,0x4(%ebp)
 80ad93d:       83 f8 01                cmp    $0x1,%eax
 80ad940:       0f 85 e3 fd ff ff       jne    80ad729 <btree_remove+0x419>
 80ad946:       8b 45 0c                mov    0xc(%ebp),%eax
 80ad949:       8d 70 ff                lea    -0x1(%eax),%esi
 80ad94c:       e9 e3 fd ff ff          jmp    80ad734 <btree_remove+0x424>
 80ad951:       8b 54 24 10             mov    0x10(%esp),%edx
 80ad955:       8b 4c 24 0c             mov    0xc(%esp),%ecx
 80ad959:       29 d1                   sub    %edx,%ecx
 80ad95b:       89 4c 24 1c             mov    %ecx,0x1c(%esp)
 80ad95f:       8b 4c 24 08             mov    0x8(%esp),%ecx
 80ad963:       8d 0c d1                lea    (%ecx,%edx,8),%ecx
 80ad966:       31 d2                   xor    %edx,%edx
 80ad968:       89 4c 24 20             mov    %ecx,0x20(%esp)
 80ad96c:       8b 4c 24 1c             mov    0x1c(%esp),%ecx
 80ad970:       d1 e9                   shr    $1,%ecx
 80ad972:       89 4c 24 1c             mov    %ecx,0x1c(%esp)
 80ad976:       74 4b                   je     80ad9c3 <btree_remove+0x6b3>
 80ad978:       89 5c 24 24             mov    %ebx,0x24(%esp)
 80ad97c:       8b 4c 24 20             mov    0x20(%esp),%ecx
 80ad980:       89 74 24 28             mov    %esi,0x28(%esp)
 80ad984:       89 44 24 2c             mov    %eax,0x2c(%esp)
 80ad988:       8b 44 24 1c             mov    0x1c(%esp),%eax
 80ad98c:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ad993:       00
 80ad994:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ad99b:       00
 80ad99c:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi
 80ad9a0:       8b 5c d5 0c             mov    0xc(%ebp,%edx,8),%ebx
 80ad9a4:       8b 74 d5 10             mov    0x10(%ebp,%edx,8),%esi
 80ad9a8:       89 5c d1 0c             mov    %ebx,0xc(%ecx,%edx,8)
 80ad9ac:       89 74 d1 10             mov    %esi,0x10(%ecx,%edx,8)
 80ad9b0:       83 c2 01                add    $0x1,%edx
 80ad9b3:       39 d0                   cmp    %edx,%eax
 80ad9b5:       75 e9                   jne    80ad9a0 <btree_remove+0x690>
 80ad9b7:       8b 5c 24 24             mov    0x24(%esp),%ebx
 80ad9bb:       8b 74 24 28             mov    0x28(%esp),%esi
 80ad9bf:       8b 44 24 2c             mov    0x2c(%esp),%eax
 80ad9c3:       8b 54 24 1c             mov    0x1c(%esp),%edx
 80ad9c7:       8b 4c 24 0c             mov    0xc(%esp),%ecx
 80ad9cb:       29 d1                   sub    %edx,%ecx
 80ad9cd:       89 4c 24 20             mov    %ecx,0x20(%esp)
 80ad9d1:       8d 4c d5 00             lea    0x0(%ebp,%edx,8),%ecx
 80ad9d5:       89 4c 24 24             mov    %ecx,0x24(%esp)
 80ad9d9:       89 d1                   mov    %edx,%ecx
 80ad9db:       8b 54 24 0c             mov    0xc(%esp),%edx
 80ad9df:       39 d1                   cmp    %edx,%ecx
 80ad9e1:       0f 84 2c 02 00 00       je     80adc13 <btree_remove+0x903>
 80ad9e7:       8b 4c 24 20             mov    0x20(%esp),%ecx
 80ad9eb:       8b 54 24 24             mov    0x24(%esp),%edx
 80ad9ef:       89 5c 24 0c             mov    %ebx,0xc(%esp)
 80ad9f3:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ad9fa:       00
 80ad9fb:       2e 8d 74 26 00          lea    %cs:0x0(%esi,%eiz,1),%esi
 80ada00:       8b 5c c2 0c             mov    0xc(%edx,%eax,8),%ebx
 80ada04:       8b 74 c2 10             mov    0x10(%edx,%eax,8),%esi
 80ada08:       89 5c c5 0c             mov    %ebx,0xc(%ebp,%eax,8)
 80ada0c:       89 74 c5 10             mov    %esi,0x10(%ebp,%eax,8)
 80ada10:       83 c0 01                add    $0x1,%eax
 80ada13:       39 c8                   cmp    %ecx,%eax
 80ada15:       75 e9                   jne    80ada00 <btree_remove+0x6f0>
 80ada17:       8b 74 24 1c             mov    0x1c(%esp),%esi
 80ada1b:       8b 44 24 10             mov    0x10(%esp),%eax
 80ada1f:       8b 4c 24 08             mov    0x8(%esp),%ecx
 80ada23:       8b 5c 24 0c             mov    0xc(%esp),%ebx
 80ada27:       01 f0                   add    %esi,%eax
 80ada29:       89 41 04                mov    %eax,0x4(%ecx)
 80ada2c:       29 75 04                sub    %esi,0x4(%ebp)
 80ada2f:       e9 f5 fc ff ff          jmp    80ad729 <btree_remove+0x419>
 80ada34:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi
 80ada38:       8b 44 24 08             mov    0x8(%esp),%eax
 80ada3c:       89 ef                   mov    %ebp,%edi
 80ada3e:       e8 2d f2 ff ff          call   80acc70 <version_lock_unlock_exclusive>
 80ada43:       e9 08 f9 ff ff          jmp    80ad350 <btree_remove+0x40>
 80ada48:       89 d8                   mov    %ebx,%eax
 80ada4a:       31 f6                   xor    %esi,%esi
 80ada4c:       e8 1f f2 ff ff          call   80acc70 <version_lock_unlock_exclusive>
 80ada51:       83 c4 3c                add    $0x3c,%esp
 80ada54:       89 f0                   mov    %esi,%eax
 80ada56:       5b                      pop    %ebx
 80ada57:       5e                      pop    %esi
 80ada58:       5f                      pop    %edi
 80ada59:       5d                      pop    %ebp
 80ada5a:       c3                      ret
 80ada5b:       2e 8d 74 26 00          lea    %cs:0x0(%esi,%eiz,1),%esi
 80ada60:       89 f8                   mov    %edi,%eax
 80ada62:       31 f6                   xor    %esi,%esi
 80ada64:       e8 07 f2 ff ff          call   80acc70 <version_lock_unlock_exclusive>
 80ada69:       eb e6                   jmp    80ada51 <btree_remove+0x741>
 80ada6b:       8b 5c 24 10             mov    0x10(%esp),%ebx
 80ada6f:       31 d2                   xor    %edx,%edx
 80ada71:       85 db                   test   %ebx,%ebx
 80ada73:       74 48                   je     80adabd <btree_remove+0x7ad>
 80ada75:       89 74 24 20             mov    %esi,0x20(%esp)
 80ada79:       8b 74 24 10             mov    0x10(%esp),%esi
 80ada7d:       8b 4c 24 08             mov    0x8(%esp),%ecx
 80ada81:       89 74 24 1c             mov    %esi,0x1c(%esp)
 80ada85:       eb 19                   jmp    80adaa0 <btree_remove+0x790>
 80ada87:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ada8e:       00
 80ada8f:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ada96:       00
 80ada97:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ada9e:       00
 80ada9f:       90                      nop
 80adaa0:       8b 5c d1 0c             mov    0xc(%ecx,%edx,8),%ebx
 80adaa4:       8b 74 d1 10             mov    0x10(%ecx,%edx,8),%esi
 80adaa8:       89 5c d7 0c             mov    %ebx,0xc(%edi,%edx,8)
 80adaac:       89 74 d7 10             mov    %esi,0x10(%edi,%edx,8)
 80adab0:       83 c2 01                add    $0x1,%edx
 80adab3:       39 54 24 1c             cmp    %edx,0x1c(%esp)
 80adab7:       75 e7                   jne    80adaa0 <btree_remove+0x790>
 80adab9:       8b 74 24 20             mov    0x20(%esp),%esi
 80adabd:       8b 4c 24 10             mov    0x10(%esp),%ecx
 80adac1:       8d 14 cf                lea    (%edi,%ecx,8),%edx
 80adac4:       8b 4c 24 0c             mov    0xc(%esp),%ecx
 80adac8:       85 c9                   test   %ecx,%ecx
 80adaca:       74 2f                   je     80adafb <btree_remove+0x7eb>
 80adacc:       8b 4c 24 0c             mov    0xc(%esp),%ecx
 80adad0:       89 74 24 0c             mov    %esi,0xc(%esp)
 80adad4:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80adadb:       00
 80adadc:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi
 80adae0:       8b 5c c5 0c             mov    0xc(%ebp,%eax,8),%ebx
 80adae4:       8b 74 c5 10             mov    0x10(%ebp,%eax,8),%esi
 80adae8:       89 5c c2 0c             mov    %ebx,0xc(%edx,%eax,8)
 80adaec:       89 74 c2 10             mov    %esi,0x10(%edx,%eax,8)
 80adaf0:       83 c0 01                add    $0x1,%eax
 80adaf3:       39 c1                   cmp    %eax,%ecx
 80adaf5:       75 e9                   jne    80adae0 <btree_remove+0x7d0>
 80adaf7:       8b 74 24 0c             mov    0xc(%esp),%esi
 80adafb:       8b 54 24 08             mov    0x8(%esp),%edx
 80adaff:       8b 44 24 18             mov    0x18(%esp),%eax
 80adb03:       89 77 04                mov    %esi,0x4(%edi)
 80adb06:       c7 42 08 02 00 00 00    movl   $0x2,0x8(%edx)
 80adb0d:       8d 58 04                lea    0x4(%eax),%ebx
 80adb10:       8b 40 04                mov    0x4(%eax),%eax
 80adb13:       89 42 10                mov    %eax,0x10(%edx)
 80adb16:       f0 0f b1 13             lock cmpxchg %edx,(%ebx)
 80adb1a:       75 f7                   jne    80adb13 <btree_remove+0x803>
 80adb1c:       8b 44 24 08             mov    0x8(%esp),%eax
 80adb20:       e8 4b f1 ff ff          call   80acc70 <version_lock_unlock_exclusive>
 80adb25:       c7 45 08 02 00 00 00    movl   $0x2,0x8(%ebp)
 80adb2c:       8b 44 24 18             mov    0x18(%esp),%eax
 80adb30:       8b 40 04                mov    0x4(%eax),%eax
 80adb33:       89 45 10                mov    %eax,0x10(%ebp)
 80adb36:       f0 0f b1 2b             lock cmpxchg %ebp,(%ebx)
 80adb3a:       75 f7                   jne    80adb33 <btree_remove+0x823>
 80adb3c:       89 e8                   mov    %ebp,%eax
 80adb3e:       e8 2d f1 ff ff          call   80acc70 <version_lock_unlock_exclusive>
 80adb43:       e9 08 f8 ff ff          jmp    80ad350 <btree_remove+0x40>
 80adb48:       8b 44 24 08             mov    0x8(%esp),%eax
 80adb4c:       8b 5c 24 10             mov    0x10(%esp),%ebx
 80adb50:       c7 47 08 01 00 00 00    movl   $0x1,0x8(%edi)
 80adb57:       31 c9                   xor    %ecx,%ecx
 80adb59:       8d 50 0c                lea    0xc(%eax),%edx
 80adb5c:       8d 47 0c                lea    0xc(%edi),%eax
 80adb5f:       85 db                   test   %ebx,%ebx
 80adb61:       74 3f                   je     80adba2 <btree_remove+0x892>
 80adb63:       89 74 24 1c             mov    %esi,0x1c(%esp)
 80adb67:       8b 5c 24 10             mov    0x10(%esp),%ebx
 80adb6b:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80adb72:       00
 80adb73:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80adb7a:       00
 80adb7b:       2e 8d 74 26 00          lea    %cs:0x0(%esi,%eiz,1),%esi
 80adb80:       8b 32                   mov    (%edx),%esi
 80adb82:       83 c1 01                add    $0x1,%ecx
 80adb85:       83 c2 0c                add    $0xc,%edx
 80adb88:       83 c0 0c                add    $0xc,%eax
 80adb8b:       89 70 f4                mov    %esi,-0xc(%eax)
 80adb8e:       8b 72 f8                mov    -0x8(%edx),%esi
 80adb91:       89 70 f8                mov    %esi,-0x8(%eax)
 80adb94:       8b 72 fc                mov    -0x4(%edx),%esi
 80adb97:       89 70 fc                mov    %esi,-0x4(%eax)
 80adb9a:       39 cb                   cmp    %ecx,%ebx
 80adb9c:       75 e2                   jne    80adb80 <btree_remove+0x870>
 80adb9e:       8b 74 24 1c             mov    0x1c(%esp),%esi
 80adba2:       8b 44 24 0c             mov    0xc(%esp),%eax
 80adba6:       85 c0                   test   %eax,%eax
 80adba8:       0f 84 4d ff ff ff       je     80adafb <btree_remove+0x7eb>
 80adbae:       8b 44 24 10             mov    0x10(%esp),%eax
 80adbb2:       8b 5c 24 0c             mov    0xc(%esp),%ebx
 80adbb6:       89 74 24 0c             mov    %esi,0xc(%esp)
 80adbba:       8d 55 0c                lea    0xc(%ebp),%edx
 80adbbd:       31 c9                   xor    %ecx,%ecx
 80adbbf:       8d 44 40 03             lea    0x3(%eax,%eax,2),%eax
 80adbc3:       8d 04 87                lea    (%edi,%eax,4),%eax
 80adbc6:       eb 18                   jmp    80adbe0 <btree_remove+0x8d0>
 80adbc8:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80adbcf:       00
 80adbd0:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80adbd7:       00
 80adbd8:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80adbdf:       00
 80adbe0:       8b 32                   mov    (%edx),%esi
 80adbe2:       83 c1 01                add    $0x1,%ecx
 80adbe5:       83 c2 0c                add    $0xc,%edx
 80adbe8:       83 c0 0c                add    $0xc,%eax
 80adbeb:       89 70 f4                mov    %esi,-0xc(%eax)
 80adbee:       8b 72 f8                mov    -0x8(%edx),%esi
 80adbf1:       89 70 f8                mov    %esi,-0x8(%eax)
 80adbf4:       8b 72 fc                mov    -0x4(%edx),%esi
 80adbf7:       89 70 fc                mov    %esi,-0x4(%eax)
 80adbfa:       39 cb                   cmp    %ecx,%ebx
 80adbfc:       75 e2                   jne    80adbe0 <btree_remove+0x8d0>
 80adbfe:       e9 f4 fe ff ff          jmp    80adaf7 <btree_remove+0x7e7>
 80adc03:       8b 44 24 08             mov    0x8(%esp),%eax
 80adc07:       8b 74 24 10             mov    0x10(%esp),%esi
 80adc0b:       89 70 04                mov    %esi,0x4(%eax)
 80adc0e:       e9 16 fb ff ff          jmp    80ad729 <btree_remove+0x419>
 80adc13:       8b 44 24 08             mov    0x8(%esp),%eax
 80adc17:       89 70 04                mov    %esi,0x4(%eax)
 80adc1a:       8b 44 24 1c             mov    0x1c(%esp),%eax
 80adc1e:       29 45 04                sub    %eax,0x4(%ebp)
 80adc21:       e9 03 fb ff ff          jmp    80ad729 <btree_remove+0x419>
 80adc26:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80adc2d:       00
 80adc2e:       66 90                   xchg   %ax,%ax

080adc30 <fde_mixed_encoding_extract>:
 80adc30:       f3 0f 1e fb             endbr32
 80adc34:       e8 08 bd f9 ff          call   8049941 <__x86.get_pc_thunk.ax>
 80adc39:       05 bb 93 03 00          add    $0x393bb,%eax
 80adc3e:       55                      push   %ebp
 80adc3f:       57                      push   %edi
 80adc40:       56                      push   %esi
 80adc41:       53                      push   %ebx
 80adc42:       83 ec 1c                sub    $0x1c,%esp
 80adc45:       89 44 24 0c             mov    %eax,0xc(%esp)
 80adc49:       8b 44 24 3c             mov    0x3c(%esp),%eax
 80adc4d:       8b 6c 24 38             mov    0x38(%esp),%ebp
 80adc51:       85 c0                   test   %eax,%eax
 80adc53:       7e 73                   jle    80adcc8 <fde_mixed_encoding_extract+0x98>
 80adc55:       8b 74 24 34             mov    0x34(%esp),%esi
 80adc59:       31 db                   xor    %ebx,%ebx
 80adc5b:       eb 29                   jmp    80adc86 <fde_mixed_encoding_extract+0x56>
 80adc5d:       8d 76 00                lea    0x0(%esi),%esi
 80adc60:       76 5e                   jbe    80adcc0 <fde_mixed_encoding_extract+0x90>
 80adc62:       80 fa 30                cmp    $0x30,%dl
 80adc65:       75 51                   jne    80adcb8 <fde_mixed_encoding_extract+0x88>
 80adc67:       8b 7c 24 30             mov    0x30(%esp),%edi
 80adc6b:       8b 57 08                mov    0x8(%edi),%edx
 80adc6e:       83 ec 0c                sub    $0xc,%esp
 80adc71:       83 c3 01                add    $0x1,%ebx
 80adc74:       56                      push   %esi
 80adc75:       83 c6 04                add    $0x4,%esi
 80adc78:       e8 f3 eb ff ff          call   80ac870 <read_encoded_value_with_base>
 80adc7d:       83 c4 10                add    $0x10,%esp
 80adc80:       39 5c 24 3c             cmp    %ebx,0x3c(%esp)
 80adc84:       74 42                   je     80adcc8 <fde_mixed_encoding_extract+0x98>
 80adc86:       8b 7c 9d 00             mov    0x0(%ebp,%ebx,4),%edi
 80adc8a:       8d 47 04                lea    0x4(%edi),%eax
 80adc8d:       2b 47 04                sub    0x4(%edi),%eax
 80adc90:       e8 4b f5 ff ff          call   80ad1e0 <get_cie_encoding>
 80adc95:       8d 4f 08                lea    0x8(%edi),%ecx
 80adc98:       3d ff 00 00 00          cmp    $0xff,%eax
 80adc9d:       74 21                   je     80adcc0 <fde_mixed_encoding_extract+0x90>
 80adc9f:       89 c2                   mov    %eax,%edx
 80adca1:       83 e2 70                and    $0x70,%edx
 80adca4:       80 fa 20                cmp    $0x20,%dl
 80adca7:       75 b7                   jne    80adc60 <fde_mixed_encoding_extract+0x30>
 80adca9:       8b 7c 24 30             mov    0x30(%esp),%edi
 80adcad:       8b 57 04                mov    0x4(%edi),%edx
 80adcb0:       eb bc                   jmp    80adc6e <fde_mixed_encoding_extract+0x3e>
 80adcb2:       8d b6 00 00 00 00       lea    0x0(%esi),%esi
 80adcb8:       80 fa 50                cmp    $0x50,%dl
 80adcbb:       75 13                   jne    80adcd0 <fde_mixed_encoding_extract+0xa0>
 80adcbd:       8d 76 00                lea    0x0(%esi),%esi
 80adcc0:       31 d2                   xor    %edx,%edx
 80adcc2:       eb aa                   jmp    80adc6e <fde_mixed_encoding_extract+0x3e>
 80adcc4:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi
 80adcc8:       83 c4 1c                add    $0x1c,%esp
 80adccb:       5b                      pop    %ebx
 80adccc:       5e                      pop    %esi
 80adccd:       5f                      pop    %edi
 80adcce:       5d                      pop    %ebp
 80adccf:       c3                      ret
 80adcd0:       e9 43 b9 f9 ff          jmp    8049618 <fde_mixed_encoding_extract.cold>
 80adcd5:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80adcdc:       00
 80adcdd:       8d 76 00                lea    0x0(%esi),%esi

080adce0 <classify_object_over_fdes>:
 80adce0:       55                      push   %ebp
 80adce1:       57                      push   %edi
 80adce2:       56                      push   %esi
 80adce3:       e8 47 e1 f9 ff          call   804be2f <__x86.get_pc_thunk.si>
 80adce8:       81 c6 0c 93 03 00       add    $0x3930c,%esi
 80adcee:       53                      push   %ebx
 80adcef:       83 ec 3c                sub    $0x3c,%esp
 80adcf2:       89 44 24 10             mov    %eax,0x10(%esp)
 80adcf6:       8b 02                   mov    (%edx),%eax
 80adcf8:       89 4c 24 04             mov    %ecx,0x4(%esp)
 80adcfc:       89 74 24 1c             mov    %esi,0x1c(%esp)
 80add00:       89 44 24 14             mov    %eax,0x14(%esp)
 80add04:       85 c0                   test   %eax,%eax
 80add06:       0f 84 74 01 00 00       je     80ade80 <classify_object_over_fdes+0x1a0>
 80add0c:       8d 44 24 2c             lea    0x2c(%esp),%eax
 80add10:       c7 44 24 0c 00 00 00    movl   $0x0,0xc(%esp)
 80add17:       00
 80add18:       89 d3                   mov    %edx,%ebx
 80add1a:       31 ed                   xor    %ebp,%ebp
 80add1c:       c7 44 24 14 00 00 00    movl   $0x0,0x14(%esp)
 80add23:       00
 80add24:       c7 44 24 08 00 00 00    movl   $0x0,0x8(%esp)
 80add2b:       00
 80add2c:       89 44 24 18             mov    %eax,0x18(%esp)
 80add30:       e9 fb 00 00 00          jmp    80ade30 <classify_object_over_fdes+0x150>
 80add35:       8d 76 00                lea    0x0(%esi),%esi
 80add38:       89 f8                   mov    %edi,%eax
 80add3a:       e8 a1 f4 ff ff          call   80ad1e0 <get_cie_encoding>
 80add3f:       89 c5                   mov    %eax,%ebp
 80add41:       3d ff 00 00 00          cmp    $0xff,%eax
 80add46:       0f 84 14 02 00 00       je     80adf60 <classify_object_over_fdes+0x280>
 80add4c:       89 c6                   mov    %eax,%esi
 80add4e:       83 e0 70                and    $0x70,%eax
 80add51:       3c 20                   cmp    $0x20,%al
 80add53:       0f 84 d7 01 00 00       je     80adf30 <classify_object_over_fdes+0x250>
 80add59:       0f 86 89 01 00 00       jbe    80adee8 <classify_object_over_fdes+0x208>
 80add5f:       3c 30                   cmp    $0x30,%al
 80add61:       0f 85 79 01 00 00       jne    80adee0 <classify_object_over_fdes+0x200>
 80add67:       8b 44 24 10             mov    0x10(%esp),%eax
 80add6b:       8b 40 08                mov    0x8(%eax),%eax
 80add6e:       89 44 24 0c             mov    %eax,0xc(%esp)
 80add72:       8b 4c 24 04             mov    0x4(%esp),%ecx
 80add76:       85 c9                   test   %ecx,%ecx
 80add78:       0f 84 7e 01 00 00       je     80adefc <classify_object_over_fdes+0x21c>
 80add7e:       83 ec 0c                sub    $0xc,%esp
 80add81:       8d 4b 08                lea    0x8(%ebx),%ecx
 80add84:       8d 44 24 34             lea    0x34(%esp),%eax
 80add88:       50                      push   %eax
 80add89:       8b 54 24 1c             mov    0x1c(%esp),%edx
 80add8d:       89 e8                   mov    %ebp,%eax
 80add8f:       e8 dc ea ff ff          call   80ac870 <read_encoded_value_with_base>
 80add94:       83 c4 10                add    $0x10,%esp
 80add97:       89 c1                   mov    %eax,%ecx
 80add99:       89 f0                   mov    %esi,%eax
 80add9b:       83 e0 07                and    $0x7,%eax
 80add9e:       3c 02                   cmp    $0x2,%al
 80adda0:       0f 84 2a 01 00 00       je     80aded0 <classify_object_over_fdes+0x1f0>
 80adda6:       0f 87 14 01 00 00       ja     80adec0 <classify_object_over_fdes+0x1e0>
 80addac:       84 c0                   test   %al,%al
 80addae:       0f 85 c5 01 00 00       jne    80adf79 <classify_object_over_fdes+0x299>
 80addb4:       b8 ff ff ff ff          mov    $0xffffffff,%eax
 80addb9:       8b 74 24 28             mov    0x28(%esp),%esi
 80addbd:       89 7c 24 08             mov    %edi,0x8(%esp)
 80addc1:       85 c6                   test   %eax,%esi
 80addc3:       74 5f                   je     80ade24 <classify_object_over_fdes+0x144>
 80addc5:       8b 54 24 04             mov    0x4(%esp),%edx
 80addc9:       83 44 24 14 01          addl   $0x1,0x14(%esp)
 80addce:       85 d2                   test   %edx,%edx
 80addd0:       0f 84 ba 00 00 00       je     80ade90 <classify_object_over_fdes+0x1b0>
 80addd6:       83 ec 0c                sub    $0xc,%esp
 80addd9:       89 ea                   mov    %ebp,%edx
 80adddb:       ff 74 24 24             push   0x24(%esp)
 80adddf:       83 e2 0f                and    $0xf,%edx
 80adde2:       89 d0                   mov    %edx,%eax
 80adde4:       31 d2                   xor    %edx,%edx
 80adde6:       e8 85 ea ff ff          call   80ac870 <read_encoded_value_with_base>
 80addeb:       8b 4c 24 14             mov    0x14(%esp),%ecx
 80addef:       8b 54 24 3c             mov    0x3c(%esp),%edx
 80addf3:       83 c4 10                add    $0x10,%esp
 80addf6:       8b 01                   mov    (%ecx),%eax
 80addf8:       01 f2                   add    %esi,%edx
 80addfa:       8b 49 04                mov    0x4(%ecx),%ecx
 80addfd:       85 c0                   test   %eax,%eax
 80addff:       0f 85 9b 00 00 00       jne    80adea0 <classify_object_over_fdes+0x1c0>
 80ade05:       85 c9                   test   %ecx,%ecx
 80ade07:       0f 85 9d 00 00 00       jne    80adeaa <classify_object_over_fdes+0x1ca>
 80ade0d:       8b 4c 24 04             mov    0x4(%esp),%ecx
 80ade11:       89 31                   mov    %esi,(%ecx)
 80ade13:       89 51 04                mov    %edx,0x4(%ecx)
 80ade16:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ade1d:       00
 80ade1e:       66 90                   xchg   %ax,%ax
 80ade20:       89 7c 24 08             mov    %edi,0x8(%esp)
 80ade24:       8b 03                   mov    (%ebx),%eax
 80ade26:       8d 5c 03 04             lea    0x4(%ebx,%eax,1),%ebx
 80ade2a:       8b 03                   mov    (%ebx),%eax
 80ade2c:       85 c0                   test   %eax,%eax
 80ade2e:       74 50                   je     80ade80 <classify_object_over_fdes+0x1a0>
 80ade30:       8b 43 04                mov    0x4(%ebx),%eax
 80ade33:       85 c0                   test   %eax,%eax
 80ade35:       74 ed                   je     80ade24 <classify_object_over_fdes+0x144>
 80ade37:       8d 53 04                lea    0x4(%ebx),%edx
 80ade3a:       89 d7                   mov    %edx,%edi
 80ade3c:       29 c7                   sub    %eax,%edi
 80ade3e:       8b 44 24 08             mov    0x8(%esp),%eax
 80ade42:       39 c7                   cmp    %eax,%edi
 80ade44:       0f 85 ee fe ff ff       jne    80add38 <classify_object_over_fdes+0x58>
 80ade4a:       83 ec 0c                sub    $0xc,%esp
 80ade4d:       89 e8                   mov    %ebp,%eax
 80ade4f:       8d 4b 08                lea    0x8(%ebx),%ecx
 80ade52:       89 ee                   mov    %ebp,%esi
 80ade54:       8d 54 24 34             lea    0x34(%esp),%edx
 80ade58:       0f b6 c0                movzbl %al,%eax
 80ade5b:       52                      push   %edx
 80ade5c:       8b 54 24 1c             mov    0x1c(%esp),%edx
 80ade60:       e8 0b ea ff ff          call   80ac870 <read_encoded_value_with_base>
 80ade65:       83 c4 10                add    $0x10,%esp
 80ade68:       89 c1                   mov    %eax,%ecx
 80ade6a:       89 e8                   mov    %ebp,%eax
 80ade6c:       3c ff                   cmp    $0xff,%al
 80ade6e:       0f 85 25 ff ff ff       jne    80add99 <classify_object_over_fdes+0xb9>
 80ade74:       8b 03                   mov    (%ebx),%eax
 80ade76:       8d 5c 03 04             lea    0x4(%ebx,%eax,1),%ebx
 80ade7a:       8b 03                   mov    (%ebx),%eax
 80ade7c:       85 c0                   test   %eax,%eax
 80ade7e:       75 b0                   jne    80ade30 <classify_object_over_fdes+0x150>
 80ade80:       8b 44 24 14             mov    0x14(%esp),%eax
 80ade84:       83 c4 3c                add    $0x3c,%esp
 80ade87:       5b                      pop    %ebx
 80ade88:       5e                      pop    %esi
 80ade89:       5f                      pop    %edi
 80ade8a:       5d                      pop    %ebp
 80ade8b:       c3                      ret
 80ade8c:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi
 80ade90:       8b 44 24 10             mov    0x10(%esp),%eax
 80ade94:       3b 30                   cmp    (%eax),%esi
 80ade96:       73 88                   jae    80ade20 <classify_object_over_fdes+0x140>
 80ade98:       89 30                   mov    %esi,(%eax)
 80ade9a:       eb 84                   jmp    80ade20 <classify_object_over_fdes+0x140>
 80ade9c:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi
 80adea0:       39 c6                   cmp    %eax,%esi
 80adea2:       73 06                   jae    80adeaa <classify_object_over_fdes+0x1ca>
 80adea4:       8b 44 24 04             mov    0x4(%esp),%eax
 80adea8:       89 30                   mov    %esi,(%eax)
 80adeaa:       39 d1                   cmp    %edx,%ecx
 80adeac:       0f 83 6e ff ff ff       jae    80ade20 <classify_object_over_fdes+0x140>
 80adeb2:       8b 44 24 04             mov    0x4(%esp),%eax
 80adeb6:       89 50 04                mov    %edx,0x4(%eax)
 80adeb9:       e9 62 ff ff ff          jmp    80ade20 <classify_object_over_fdes+0x140>
 80adebe:       66 90                   xchg   %ax,%ax
 80adec0:       83 e8 03                sub    $0x3,%eax
 80adec3:       3c 01                   cmp    $0x1,%al
 80adec5:       0f 86 e9 fe ff ff       jbe    80addb4 <classify_object_over_fdes+0xd4>
 80adecb:       e9 51 b7 f9 ff          jmp    8049621 <classify_object_over_fdes.cold>
 80aded0:       b8 ff ff 00 00          mov    $0xffff,%eax
 80aded5:       e9 df fe ff ff          jmp    80addb9 <classify_object_over_fdes+0xd9>
 80adeda:       8d b6 00 00 00 00       lea    0x0(%esi),%esi
 80adee0:       3c 50                   cmp    $0x50,%al
 80adee2:       0f 85 8c 00 00 00       jne    80adf74 <classify_object_over_fdes+0x294>
 80adee8:       8b 4c 24 04             mov    0x4(%esp),%ecx
 80adeec:       c7 44 24 0c 00 00 00    movl   $0x0,0xc(%esp)
 80adef3:       00
 80adef4:       85 c9                   test   %ecx,%ecx
 80adef6:       0f 85 82 fe ff ff       jne    80add7e <classify_object_over_fdes+0x9e>
 80adefc:       8b 44 24 10             mov    0x10(%esp),%eax
 80adf00:       0f b7 40 10             movzwl 0x10(%eax),%eax
 80adf04:       89 c2                   mov    %eax,%edx
 80adf06:       f7 d2                   not    %edx
 80adf08:       66 f7 c2 f8 07          test   $0x7f8,%dx
 80adf0d:       74 31                   je     80adf40 <classify_object_over_fdes+0x260>
 80adf0f:       66 c1 e8 03             shr    $0x3,%ax
 80adf13:       0f b6 c0                movzbl %al,%eax
 80adf16:       39 e8                   cmp    %ebp,%eax
 80adf18:       0f 84 60 fe ff ff       je     80add7e <classify_object_over_fdes+0x9e>
 80adf1e:       8b 44 24 10             mov    0x10(%esp),%eax
 80adf22:       80 48 10 04             orb    $0x4,0x10(%eax)
 80adf26:       e9 53 fe ff ff          jmp    80add7e <classify_object_over_fdes+0x9e>
 80adf2b:       2e 8d 74 26 00          lea    %cs:0x0(%esi,%eiz,1),%esi
 80adf30:       8b 44 24 10             mov    0x10(%esp),%eax
 80adf34:       8b 40 04                mov    0x4(%eax),%eax
 80adf37:       89 44 24 0c             mov    %eax,0xc(%esp)
 80adf3b:       e9 32 fe ff ff          jmp    80add72 <classify_object_over_fdes+0x92>
 80adf40:       89 f1                   mov    %esi,%ecx
 80adf42:       66 25 07 f8             and    $0xf807,%ax
 80adf46:       0f b6 d1                movzbl %cl,%edx
 80adf49:       c1 e2 03                shl    $0x3,%edx
 80adf4c:       09 d0                   or     %edx,%eax
 80adf4e:       8b 54 24 10             mov    0x10(%esp),%edx
 80adf52:       66 89 42 10             mov    %ax,0x10(%edx)
 80adf56:       e9 23 fe ff ff          jmp    80add7e <classify_object_over_fdes+0x9e>
 80adf5b:       2e 8d 74 26 00          lea    %cs:0x0(%esi,%eiz,1),%esi
 80adf60:       c7 44 24 14 ff ff ff    movl   $0xffffffff,0x14(%esp)
 80adf67:       ff
 80adf68:       8b 44 24 14             mov    0x14(%esp),%eax
 80adf6c:       83 c4 3c                add    $0x3c,%esp
 80adf6f:       5b                      pop    %ebx
 80adf70:       5e                      pop    %esi
 80adf71:       5f                      pop    %edi
 80adf72:       5d                      pop    %ebp
 80adf73:       c3                      ret
 80adf74:       e9 a8 b6 f9 ff          jmp    8049621 <classify_object_over_fdes.cold>
 80adf79:       e9 a3 b6 f9 ff          jmp    8049621 <classify_object_over_fdes.cold>
 80adf7e:       66 90                   xchg   %ax,%ax

080adf80 <get_pc_range>:
 80adf80:       57                      push   %edi
 80adf81:       89 d7                   mov    %edx,%edi
 80adf83:       56                      push   %esi
 80adf84:       89 c6                   mov    %eax,%esi
 80adf86:       53                      push   %ebx
 80adf87:       8b 5e 0c                mov    0xc(%esi),%ebx
 80adf8a:       c7 42 04 00 00 00 00    movl   $0x0,0x4(%edx)
 80adf91:       c7 02 00 00 00 00       movl   $0x0,(%edx)
 80adf97:       0f b6 40 10             movzbl 0x10(%eax),%eax
 80adf9b:       a8 01                   test   $0x1,%al
 80adf9d:       75 41                   jne    80adfe0 <get_pc_range+0x60>
 80adf9f:       a8 02                   test   $0x2,%al
 80adfa1:       74 2d                   je     80adfd0 <get_pc_range+0x50>
 80adfa3:       8b 13                   mov    (%ebx),%edx
 80adfa5:       85 d2                   test   %edx,%edx
 80adfa7:       74 19                   je     80adfc2 <get_pc_range+0x42>
 80adfa9:       8d b4 26 00 00 00 00    lea    0x0(%esi,%eiz,1),%esi
 80adfb0:       89 f9                   mov    %edi,%ecx
 80adfb2:       89 f0                   mov    %esi,%eax
 80adfb4:       83 c3 04                add    $0x4,%ebx
 80adfb7:       e8 24 fd ff ff          call   80adce0 <classify_object_over_fdes>
 80adfbc:       8b 13                   mov    (%ebx),%edx
 80adfbe:       85 d2                   test   %edx,%edx
 80adfc0:       75 ee                   jne    80adfb0 <get_pc_range+0x30>
 80adfc2:       5b                      pop    %ebx
 80adfc3:       5e                      pop    %esi
 80adfc4:       5f                      pop    %edi
 80adfc5:       c3                      ret
 80adfc6:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80adfcd:       00
 80adfce:       66 90                   xchg   %ax,%ax
 80adfd0:       89 d1                   mov    %edx,%ecx
 80adfd2:       89 f0                   mov    %esi,%eax
 80adfd4:       89 da                   mov    %ebx,%edx
 80adfd6:       5b                      pop    %ebx
 80adfd7:       5e                      pop    %esi
 80adfd8:       5f                      pop    %edi
 80adfd9:       e9 02 fd ff ff          jmp    80adce0 <classify_object_over_fdes>
 80adfde:       66 90                   xchg   %ax,%ax
 80adfe0:       8b 13                   mov    (%ebx),%edx
 80adfe2:       89 f9                   mov    %edi,%ecx
 80adfe4:       5b                      pop    %ebx
 80adfe5:       89 f0                   mov    %esi,%eax
 80adfe7:       5e                      pop    %esi
 80adfe8:       5f                      pop    %edi
 80adfe9:       e9 f2 fc ff ff          jmp    80adce0 <classify_object_over_fdes>
 80adfee:       66 90                   xchg   %ax,%ax

080adff0 <__deregister_frame_info_bases.part.0>:
 80adff0:       56                      push   %esi
 80adff1:       89 c2                   mov    %eax,%edx
 80adff3:       53                      push   %ebx
 80adff4:       e8 e7 b7 f9 ff          call   80497e0 <__x86.get_pc_thunk.bx>
 80adff9:       81 c3 fb 8f 03 00       add    $0x38ffb,%ebx
 80adfff:       83 ec 14                sub    $0x14,%esp
 80ae002:       8d 83 28 3c 00 00       lea    0x3c28(%ebx),%eax
 80ae008:       e8 03 f3 ff ff          call   80ad310 <btree_remove>
 80ae00d:       89 c6                   mov    %eax,%esi
 80ae00f:       85 c0                   test   %eax,%eax
 80ae011:       74 3d                   je     80ae050 <__deregister_frame_info_bases.part.0+0x60>
 80ae013:       8d 54 24 08             lea    0x8(%esp),%edx
 80ae017:       e8 64 ff ff ff          call   80adf80 <get_pc_range>
 80ae01c:       8b 54 24 08             mov    0x8(%esp),%edx
 80ae020:       3b 54 24 0c             cmp    0xc(%esp),%edx
 80ae024:       75 12                   jne    80ae038 <__deregister_frame_info_bases.part.0+0x48>
 80ae026:       f6 46 10 01             testb  $0x1,0x10(%esi)
 80ae02a:       75 34                   jne    80ae060 <__deregister_frame_info_bases.part.0+0x70>
 80ae02c:       83 c4 14                add    $0x14,%esp
 80ae02f:       89 f0                   mov    %esi,%eax
 80ae031:       5b                      pop    %ebx
 80ae032:       5e                      pop    %esi
 80ae033:       c3                      ret
 80ae034:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi
 80ae038:       8d 83 34 3c 00 00       lea    0x3c34(%ebx),%eax
 80ae03e:       e8 cd f2 ff ff          call   80ad310 <btree_remove>
 80ae043:       f6 46 10 01             testb  $0x1,0x10(%esi)
 80ae047:       74 e3                   je     80ae02c <__deregister_frame_info_bases.part.0+0x3c>
 80ae049:       eb 15                   jmp    80ae060 <__deregister_frame_info_bases.part.0+0x70>
 80ae04b:       2e 8d 74 26 00          lea    %cs:0x0(%esi,%eiz,1),%esi
 80ae050:       80 bb 24 3c 00 00 00    cmpb   $0x0,0x3c24(%ebx)
 80ae057:       0f 84 cd b5 f9 ff       je     804962a <__deregister_frame_info_bases.part.0.cold>
 80ae05d:       eb cd                   jmp    80ae02c <__deregister_frame_info_bases.part.0+0x3c>
 80ae05f:       90                      nop
 80ae060:       83 ec 0c                sub    $0xc,%esp
 80ae063:       ff 76 0c                push   0xc(%esi)
 80ae066:       e8 f5 98 fa ff          call   8057960 <__free>
 80ae06b:       83 c4 10                add    $0x10,%esp
 80ae06e:       89 f0                   mov    %esi,%eax
 80ae070:       83 c4 14                add    $0x14,%esp
 80ae073:       5b                      pop    %ebx
 80ae074:       5e                      pop    %esi
 80ae075:       c3                      ret
 80ae076:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ae07d:       00
 80ae07e:       66 90                   xchg   %ax,%ax

080ae080 <fde_single_encoding_extract>:
 80ae080:       f3 0f 1e fb             endbr32
 80ae084:       55                      push   %ebp
 80ae085:       57                      push   %edi
 80ae086:       56                      push   %esi
 80ae087:       53                      push   %ebx
 80ae088:       e8 53 b7 f9 ff          call   80497e0 <__x86.get_pc_thunk.bx>
 80ae08d:       81 c3 67 8f 03 00       add    $0x38f67,%ebx
 80ae093:       83 ec 1c                sub    $0x1c,%esp
 80ae096:       8b 6c 24 30             mov    0x30(%esp),%ebp
 80ae09a:       8b 7c 24 38             mov    0x38(%esp),%edi
 80ae09e:       0f b7 55 10             movzwl 0x10(%ebp),%edx
 80ae0a2:       66 c1 ea 03             shr    $0x3,%dx
 80ae0a6:       89 d0                   mov    %edx,%eax
 80ae0a8:       80 fa ff                cmp    $0xff,%dl
 80ae0ab:       74 73                   je     80ae120 <fde_single_encoding_extract+0xa0>
 80ae0ad:       83 e2 70                and    $0x70,%edx
 80ae0b0:       80 fa 20                cmp    $0x20,%dl
 80ae0b3:       74 73                   je     80ae128 <fde_single_encoding_extract+0xa8>
 80ae0b5:       76 69                   jbe    80ae120 <fde_single_encoding_extract+0xa0>
 80ae0b7:       80 fa 30                cmp    $0x30,%dl
 80ae0ba:       75 51                   jne    80ae10d <fde_single_encoding_extract+0x8d>
 80ae0bc:       8b 55 08                mov    0x8(%ebp),%edx
 80ae0bf:       8b 4c 24 3c             mov    0x3c(%esp),%ecx
 80ae0c3:       85 c9                   test   %ecx,%ecx
 80ae0c5:       7e 3e                   jle    80ae105 <fde_single_encoding_extract+0x85>
 80ae0c7:       89 54 24 0c             mov    %edx,0xc(%esp)
 80ae0cb:       8b 74 24 34             mov    0x34(%esp),%esi
 80ae0cf:       31 db                   xor    %ebx,%ebx
 80ae0d1:       eb 0d                   jmp    80ae0e0 <fde_single_encoding_extract+0x60>
 80ae0d3:       2e 8d 74 26 00          lea    %cs:0x0(%esi,%eiz,1),%esi
 80ae0d8:       0f b7 45 10             movzwl 0x10(%ebp),%eax
 80ae0dc:       66 c1 e8 03             shr    $0x3,%ax
 80ae0e0:       8b 14 9f                mov    (%edi,%ebx,4),%edx
 80ae0e3:       83 ec 0c                sub    $0xc,%esp
 80ae0e6:       0f b6 c0                movzbl %al,%eax
 80ae0e9:       83 c3 01                add    $0x1,%ebx
 80ae0ec:       56                      push   %esi
 80ae0ed:       83 c6 04                add    $0x4,%esi
 80ae0f0:       8d 4a 08                lea    0x8(%edx),%ecx
 80ae0f3:       8b 54 24 1c             mov    0x1c(%esp),%edx
 80ae0f7:       e8 74 e7 ff ff          call   80ac870 <read_encoded_value_with_base>
 80ae0fc:       83 c4 10                add    $0x10,%esp
 80ae0ff:       39 5c 24 3c             cmp    %ebx,0x3c(%esp)
 80ae103:       75 d3                   jne    80ae0d8 <fde_single_encoding_extract+0x58>
 80ae105:       83 c4 1c                add    $0x1c,%esp
 80ae108:       5b                      pop    %ebx
 80ae109:       5e                      pop    %esi
 80ae10a:       5f                      pop    %edi
 80ae10b:       5d                      pop    %ebp
 80ae10c:       c3                      ret
 80ae10d:       80 fa 50                cmp    $0x50,%dl
 80ae110:       75 1b                   jne    80ae12d <fde_single_encoding_extract+0xad>
 80ae112:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ae119:       00
 80ae11a:       8d b6 00 00 00 00       lea    0x0(%esi),%esi
 80ae120:       31 d2                   xor    %edx,%edx
 80ae122:       eb 9b                   jmp    80ae0bf <fde_single_encoding_extract+0x3f>
 80ae124:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi
 80ae128:       8b 55 04                mov    0x4(%ebp),%edx
 80ae12b:       eb 92                   jmp    80ae0bf <fde_single_encoding_extract+0x3f>
 80ae12d:       e9 fd b4 f9 ff          jmp    804962f <fde_single_encoding_extract.cold>
 80ae132:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ae139:       00
 80ae13a:       8d b6 00 00 00 00       lea    0x0(%esi),%esi

080ae140 <fde_single_encoding_compare>:
 80ae140:       f3 0f 1e fb             endbr32
 80ae144:       e8 4e 17 fa ff          call   804f897 <__x86.get_pc_thunk.cx>
 80ae149:       81 c1 ab 8e 03 00       add    $0x38eab,%ecx
 80ae14f:       56                      push   %esi
 80ae150:       53                      push   %ebx
 80ae151:       83 ec 14                sub    $0x14,%esp
 80ae154:       8b 54 24 20             mov    0x20(%esp),%edx
 80ae158:       0f b7 42 10             movzwl 0x10(%edx),%eax
 80ae15c:       66 c1 e8 03             shr    $0x3,%ax
 80ae160:       0f b6 d8                movzbl %al,%ebx
 80ae163:       3c ff                   cmp    $0xff,%al
 80ae165:       74 69                   je     80ae1d0 <fde_single_encoding_compare+0x90>
 80ae167:       83 e0 70                and    $0x70,%eax
 80ae16a:       3c 20                   cmp    $0x20,%al
 80ae16c:       74 6a                   je     80ae1d8 <fde_single_encoding_compare+0x98>
 80ae16e:       76 60                   jbe    80ae1d0 <fde_single_encoding_compare+0x90>
 80ae170:       3c 30                   cmp    $0x30,%al
 80ae172:       75 54                   jne    80ae1c8 <fde_single_encoding_compare+0x88>
 80ae174:       8b 72 08                mov    0x8(%edx),%esi
 80ae177:       8b 44 24 24             mov    0x24(%esp),%eax
 80ae17b:       83 ec 0c                sub    $0xc,%esp
 80ae17e:       89 f2                   mov    %esi,%edx
 80ae180:       8d 48 08                lea    0x8(%eax),%ecx
 80ae183:       8d 44 24 14             lea    0x14(%esp),%eax
 80ae187:       50                      push   %eax
 80ae188:       89 d8                   mov    %ebx,%eax
 80ae18a:       e8 e1 e6 ff ff          call   80ac870 <read_encoded_value_with_base>
 80ae18f:       8b 44 24 38             mov    0x38(%esp),%eax
 80ae193:       89 f2                   mov    %esi,%edx
 80ae195:       8d 48 08                lea    0x8(%eax),%ecx
 80ae198:       58                      pop    %eax
 80ae199:       8d 44 24 18             lea    0x18(%esp),%eax
 80ae19d:       50                      push   %eax
 80ae19e:       89 d8                   mov    %ebx,%eax
 80ae1a0:       e8 cb e6 ff ff          call   80ac870 <read_encoded_value_with_base>
 80ae1a5:       8b 4c 24 18             mov    0x18(%esp),%ecx
 80ae1a9:       8b 54 24 1c             mov    0x1c(%esp),%edx
 80ae1ad:       b8 01 00 00 00          mov    $0x1,%eax
 80ae1b2:       83 c4 10                add    $0x10,%esp
 80ae1b5:       39 ca                   cmp    %ecx,%edx
 80ae1b7:       72 04                   jb     80ae1bd <fde_single_encoding_compare+0x7d>
 80ae1b9:       39 d1                   cmp    %edx,%ecx
 80ae1bb:       19 c0                   sbb    %eax,%eax
 80ae1bd:       83 c4 14                add    $0x14,%esp
 80ae1c0:       5b                      pop    %ebx
 80ae1c1:       5e                      pop    %esi
 80ae1c2:       c3                      ret
 80ae1c3:       2e 8d 74 26 00          lea    %cs:0x0(%esi,%eiz,1),%esi
 80ae1c8:       3c 50                   cmp    $0x50,%al
 80ae1ca:       75 11                   jne    80ae1dd <fde_single_encoding_compare+0x9d>
 80ae1cc:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi
 80ae1d0:       31 f6                   xor    %esi,%esi
 80ae1d2:       eb a3                   jmp    80ae177 <fde_single_encoding_compare+0x37>
 80ae1d4:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi
 80ae1d8:       8b 72 04                mov    0x4(%edx),%esi
 80ae1db:       eb 9a                   jmp    80ae177 <fde_single_encoding_compare+0x37>
 80ae1dd:       e9 52 b4 f9 ff          jmp    8049634 <fde_single_encoding_compare.cold>
 80ae1e2:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ae1e9:       00
 80ae1ea:       8d b6 00 00 00 00       lea    0x0(%esi),%esi

080ae1f0 <fde_mixed_encoding_compare>:
 80ae1f0:       f3 0f 1e fb             endbr32
 80ae1f4:       55                      push   %ebp
 80ae1f5:       57                      push   %edi
 80ae1f6:       e8 38 dc f9 ff          call   804be33 <__x86.get_pc_thunk.di>
 80ae1fb:       81 c7 f9 8d 03 00       add    $0x38df9,%edi
 80ae201:       56                      push   %esi
 80ae202:       53                      push   %ebx
 80ae203:       83 ec 1c                sub    $0x1c,%esp
 80ae206:       8b 6c 24 34             mov    0x34(%esp),%ebp
 80ae20a:       8b 74 24 30             mov    0x30(%esp),%esi
 80ae20e:       8b 5c 24 38             mov    0x38(%esp),%ebx
 80ae212:       8d 45 04                lea    0x4(%ebp),%eax
 80ae215:       2b 45 04                sub    0x4(%ebp),%eax
 80ae218:       e8 c3 ef ff ff          call   80ad1e0 <get_cie_encoding>
 80ae21d:       8d 4d 08                lea    0x8(%ebp),%ecx
 80ae220:       3d ff 00 00 00          cmp    $0xff,%eax
 80ae225:       0f 84 95 00 00 00       je     80ae2c0 <fde_mixed_encoding_compare+0xd0>
 80ae22b:       89 c2                   mov    %eax,%edx
 80ae22d:       83 e2 70                and    $0x70,%edx
 80ae230:       80 fa 20                cmp    $0x20,%dl
 80ae233:       0f 84 af 00 00 00       je     80ae2e8 <fde_mixed_encoding_compare+0xf8>
 80ae239:       0f 86 81 00 00 00       jbe    80ae2c0 <fde_mixed_encoding_compare+0xd0>
 80ae23f:       80 fa 30                cmp    $0x30,%dl
 80ae242:       75 6c                   jne    80ae2b0 <fde_mixed_encoding_compare+0xc0>
 80ae244:       8b 56 08                mov    0x8(%esi),%edx
 80ae247:       83 ec 0c                sub    $0xc,%esp
 80ae24a:       8d 6c 24 14             lea    0x14(%esp),%ebp
 80ae24e:       55                      push   %ebp
 80ae24f:       e8 1c e6 ff ff          call   80ac870 <read_encoded_value_with_base>
 80ae254:       8d 43 04                lea    0x4(%ebx),%eax
 80ae257:       2b 43 04                sub    0x4(%ebx),%eax
 80ae25a:       e8 81 ef ff ff          call   80ad1e0 <get_cie_encoding>
 80ae25f:       8d 4b 08                lea    0x8(%ebx),%ecx
 80ae262:       83 c4 10                add    $0x10,%esp
 80ae265:       3d ff 00 00 00          cmp    $0xff,%eax
 80ae26a:       74 74                   je     80ae2e0 <fde_mixed_encoding_compare+0xf0>
 80ae26c:       89 c2                   mov    %eax,%edx
 80ae26e:       83 e2 70                and    $0x70,%edx
 80ae271:       80 fa 20                cmp    $0x20,%dl
 80ae274:       74 7a                   je     80ae2f0 <fde_mixed_encoding_compare+0x100>
 80ae276:       76 68                   jbe    80ae2e0 <fde_mixed_encoding_compare+0xf0>
 80ae278:       80 fa 30                cmp    $0x30,%dl
 80ae27b:       75 53                   jne    80ae2d0 <fde_mixed_encoding_compare+0xe0>
 80ae27d:       8b 5e 08                mov    0x8(%esi),%ebx
 80ae280:       83 ec 0c                sub    $0xc,%esp
 80ae283:       8d 54 24 18             lea    0x18(%esp),%edx
 80ae287:       52                      push   %edx
 80ae288:       89 da                   mov    %ebx,%edx
 80ae28a:       e8 e1 e5 ff ff          call   80ac870 <read_encoded_value_with_base>
 80ae28f:       8b 4c 24 18             mov    0x18(%esp),%ecx
 80ae293:       8b 54 24 1c             mov    0x1c(%esp),%edx
 80ae297:       b8 01 00 00 00          mov    $0x1,%eax
 80ae29c:       83 c4 10                add    $0x10,%esp
 80ae29f:       39 ca                   cmp    %ecx,%edx
 80ae2a1:       72 04                   jb     80ae2a7 <fde_mixed_encoding_compare+0xb7>
 80ae2a3:       39 d1                   cmp    %edx,%ecx
 80ae2a5:       19 c0                   sbb    %eax,%eax
 80ae2a7:       83 c4 1c                add    $0x1c,%esp
 80ae2aa:       5b                      pop    %ebx
 80ae2ab:       5e                      pop    %esi
 80ae2ac:       5f                      pop    %edi
 80ae2ad:       5d                      pop    %ebp
 80ae2ae:       c3                      ret
 80ae2af:       90                      nop
 80ae2b0:       80 fa 50                cmp    $0x50,%dl
 80ae2b3:       75 40                   jne    80ae2f5 <fde_mixed_encoding_compare+0x105>
 80ae2b5:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ae2bc:       00
 80ae2bd:       8d 76 00                lea    0x0(%esi),%esi
 80ae2c0:       31 d2                   xor    %edx,%edx
 80ae2c2:       e9 80 ff ff ff          jmp    80ae247 <fde_mixed_encoding_compare+0x57>
 80ae2c7:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ae2ce:       00
 80ae2cf:       90                      nop
 80ae2d0:       80 fa 50                cmp    $0x50,%dl
 80ae2d3:       75 25                   jne    80ae2fa <fde_mixed_encoding_compare+0x10a>
 80ae2d5:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ae2dc:       00
 80ae2dd:       8d 76 00                lea    0x0(%esi),%esi
 80ae2e0:       31 db                   xor    %ebx,%ebx
 80ae2e2:       eb 9c                   jmp    80ae280 <fde_mixed_encoding_compare+0x90>
 80ae2e4:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi
 80ae2e8:       8b 56 04                mov    0x4(%esi),%edx
 80ae2eb:       e9 57 ff ff ff          jmp    80ae247 <fde_mixed_encoding_compare+0x57>
 80ae2f0:       8b 5e 04                mov    0x4(%esi),%ebx
 80ae2f3:       eb 8b                   jmp    80ae280 <fde_mixed_encoding_compare+0x90>
 80ae2f5:       e9 41 b3 f9 ff          jmp    804963b <fde_mixed_encoding_compare.cold>
 80ae2fa:       e9 3c b3 f9 ff          jmp    804963b <fde_mixed_encoding_compare.cold>
 80ae2ff:       90                      nop

080ae300 <add_fdes.isra.0>:
 80ae300:       55                      push   %ebp
 80ae301:       57                      push   %edi
 80ae302:       e8 2c db f9 ff          call   804be33 <__x86.get_pc_thunk.di>
 80ae307:       81 c7 ed 8c 03 00       add    $0x38ced,%edi
 80ae30d:       56                      push   %esi
 80ae30e:       53                      push   %ebx
 80ae30f:       89 cb                   mov    %ecx,%ebx
 80ae311:       83 ec 3c                sub    $0x3c,%esp
 80ae314:       89 44 24 04             mov    %eax,0x4(%esp)
 80ae318:       0f b7 40 10             movzwl 0x10(%eax),%eax
 80ae31c:       89 7c 24 1c             mov    %edi,0x1c(%esp)
 80ae320:       66 89 44 24 08          mov    %ax,0x8(%esp)
 80ae325:       66 c1 e8 03             shr    $0x3,%ax
 80ae329:       89 54 24 0c             mov    %edx,0xc(%esp)
 80ae32d:       3c ff                   cmp    $0xff,%al
 80ae32f:       0f 84 3b 02 00 00       je     80ae570 <add_fdes.isra.0+0x270>
 80ae335:       0f b6 e8                movzbl %al,%ebp
 80ae338:       83 e0 70                and    $0x70,%eax
 80ae33b:       3c 20                   cmp    $0x20,%al
 80ae33d:       0f 84 15 02 00 00       je     80ae558 <add_fdes.isra.0+0x258>
 80ae343:       0f 86 38 01 00 00       jbe    80ae481 <add_fdes.isra.0+0x181>
 80ae349:       3c 30                   cmp    $0x30,%al
 80ae34b:       0f 85 28 01 00 00       jne    80ae479 <add_fdes.isra.0+0x179>
 80ae351:       8b 44 24 04             mov    0x4(%esp),%eax
 80ae355:       8b 40 08                mov    0x8(%eax),%eax
 80ae358:       89 44 24 14             mov    %eax,0x14(%esp)
 80ae35c:       8b 33                   mov    (%ebx),%esi
 80ae35e:       c7 44 24 08 00 00 00    movl   $0x0,0x8(%esp)
 80ae365:       00
 80ae366:       8d 54 24 2c             lea    0x2c(%esp),%edx
 80ae36a:       85 f6                   test   %esi,%esi
 80ae36c:       0f 84 ff 00 00 00       je     80ae471 <add_fdes.isra.0+0x171>
 80ae372:       89 54 24 18             mov    %edx,0x18(%esp)
 80ae376:       e9 8d 00 00 00          jmp    80ae408 <add_fdes.isra.0+0x108>
 80ae37b:       2e 8d 74 26 00          lea    %cs:0x0(%esi,%eiz,1),%esi
 80ae380:       8d 7b 04                lea    0x4(%ebx),%edi
 80ae383:       29 c7                   sub    %eax,%edi
 80ae385:       8b 44 24 08             mov    0x8(%esp),%eax
 80ae389:       39 c7                   cmp    %eax,%edi
 80ae38b:       0f 84 8c 00 00 00       je     80ae41d <add_fdes.isra.0+0x11d>
 80ae391:       89 f8                   mov    %edi,%eax
 80ae393:       e8 48 ee ff ff          call   80ad1e0 <get_cie_encoding>
 80ae398:       88 44 24 13             mov    %al,0x13(%esp)
 80ae39c:       89 c5                   mov    %eax,%ebp
 80ae39e:       3d ff 00 00 00          cmp    $0xff,%eax
 80ae3a3:       0f 84 7f 01 00 00       je     80ae528 <add_fdes.isra.0+0x228>
 80ae3a9:       83 e0 70                and    $0x70,%eax
 80ae3ac:       3c 20                   cmp    $0x20,%al
 80ae3ae:       0f 84 3c 01 00 00       je     80ae4f0 <add_fdes.isra.0+0x1f0>
 80ae3b4:       0f 86 fe 00 00 00       jbe    80ae4b8 <add_fdes.isra.0+0x1b8>
 80ae3ba:       3c 30                   cmp    $0x30,%al
 80ae3bc:       0f 85 ee 00 00 00       jne    80ae4b0 <add_fdes.isra.0+0x1b0>
 80ae3c2:       8b 44 24 04             mov    0x4(%esp),%eax
 80ae3c6:       89 7c 24 08             mov    %edi,0x8(%esp)
 80ae3ca:       8b 40 08                mov    0x8(%eax),%eax
 80ae3cd:       89 44 24 14             mov    %eax,0x14(%esp)
 80ae3d1:       85 ed                   test   %ebp,%ebp
 80ae3d3:       0f 85 2e 01 00 00       jne    80ae507 <add_fdes.isra.0+0x207>
 80ae3d9:       8d b4 26 00 00 00 00    lea    0x0(%esi,%eiz,1),%esi
 80ae3e0:       8b 43 08                mov    0x8(%ebx),%eax
 80ae3e3:       31 ed                   xor    %ebp,%ebp
 80ae3e5:       85 c0                   test   %eax,%eax
 80ae3e7:       74 15                   je     80ae3fe <add_fdes.isra.0+0xfe>
 80ae3e9:       8b 4c 24 0c             mov    0xc(%esp),%ecx
 80ae3ed:       85 c9                   test   %ecx,%ecx
 80ae3ef:       74 0d                   je     80ae3fe <add_fdes.isra.0+0xfe>
 80ae3f1:       8b 41 04                mov    0x4(%ecx),%eax
 80ae3f4:       8d 50 01                lea    0x1(%eax),%edx
 80ae3f7:       89 51 04                mov    %edx,0x4(%ecx)
 80ae3fa:       89 5c 81 08             mov    %ebx,0x8(%ecx,%eax,4)
 80ae3fe:       8d 5c 33 04             lea    0x4(%ebx,%esi,1),%ebx
 80ae402:       8b 33                   mov    (%ebx),%esi
 80ae404:       85 f6                   test   %esi,%esi
 80ae406:       74 69                   je     80ae471 <add_fdes.isra.0+0x171>
 80ae408:       8b 43 04                mov    0x4(%ebx),%eax
 80ae40b:       85 c0                   test   %eax,%eax
 80ae40d:       74 ef                   je     80ae3fe <add_fdes.isra.0+0xfe>
 80ae40f:       8b 54 24 04             mov    0x4(%esp),%edx
 80ae413:       f6 42 10 04             testb  $0x4,0x10(%edx)
 80ae417:       0f 85 63 ff ff ff       jne    80ae380 <add_fdes.isra.0+0x80>
 80ae41d:       85 ed                   test   %ebp,%ebp
 80ae41f:       74 bf                   je     80ae3e0 <add_fdes.isra.0+0xe0>
 80ae421:       83 ec 0c                sub    $0xc,%esp
 80ae424:       89 e8                   mov    %ebp,%eax
 80ae426:       8d 4b 08                lea    0x8(%ebx),%ecx
 80ae429:       ff 74 24 24             push   0x24(%esp)
 80ae42d:       8b 54 24 24             mov    0x24(%esp),%edx
 80ae431:       0f b6 c0                movzbl %al,%eax
 80ae434:       e8 37 e4 ff ff          call   80ac870 <read_encoded_value_with_base>
 80ae439:       89 e8                   mov    %ebp,%eax
 80ae43b:       88 44 24 23             mov    %al,0x23(%esp)
 80ae43f:       83 c4 10                add    $0x10,%esp
 80ae442:       3c ff                   cmp    $0xff,%al
 80ae444:       74 b8                   je     80ae3fe <add_fdes.isra.0+0xfe>
 80ae446:       0f b6 44 24 13          movzbl 0x13(%esp),%eax
 80ae44b:       83 e0 07                and    $0x7,%eax
 80ae44e:       3c 02                   cmp    $0x2,%al
 80ae450:       74 4e                   je     80ae4a0 <add_fdes.isra.0+0x1a0>
 80ae452:       77 3c                   ja     80ae490 <add_fdes.isra.0+0x190>
 80ae454:       84 c0                   test   %al,%al
 80ae456:       0f 85 2b 01 00 00       jne    80ae587 <add_fdes.isra.0+0x287>
 80ae45c:       b8 ff ff ff ff          mov    $0xffffffff,%eax
 80ae461:       23 44 24 2c             and    0x2c(%esp),%eax
 80ae465:       75 82                   jne    80ae3e9 <add_fdes.isra.0+0xe9>
 80ae467:       8d 5c 33 04             lea    0x4(%ebx,%esi,1),%ebx
 80ae46b:       8b 33                   mov    (%ebx),%esi
 80ae46d:       85 f6                   test   %esi,%esi
 80ae46f:       75 97                   jne    80ae408 <add_fdes.isra.0+0x108>
 80ae471:       83 c4 3c                add    $0x3c,%esp
 80ae474:       5b                      pop    %ebx
 80ae475:       5e                      pop    %esi
 80ae476:       5f                      pop    %edi
 80ae477:       5d                      pop    %ebp
 80ae478:       c3                      ret
 80ae479:       3c 50                   cmp    $0x50,%al
 80ae47b:       0f 85 0b 01 00 00       jne    80ae58c <add_fdes.isra.0+0x28c>
 80ae481:       c7 44 24 14 00 00 00    movl   $0x0,0x14(%esp)
 80ae488:       00
 80ae489:       e9 ce fe ff ff          jmp    80ae35c <add_fdes.isra.0+0x5c>
 80ae48e:       66 90                   xchg   %ax,%ax
 80ae490:       83 e8 03                sub    $0x3,%eax
 80ae493:       3c 01                   cmp    $0x1,%al
 80ae495:       76 c5                   jbe    80ae45c <add_fdes.isra.0+0x15c>
 80ae497:       e9 a6 b1 f9 ff          jmp    8049642 <add_fdes.isra.0.cold>
 80ae49c:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi
 80ae4a0:       b8 ff ff 00 00          mov    $0xffff,%eax
 80ae4a5:       eb ba                   jmp    80ae461 <add_fdes.isra.0+0x161>
 80ae4a7:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ae4ae:       00
 80ae4af:       90                      nop
 80ae4b0:       3c 50                   cmp    $0x50,%al
 80ae4b2:       0f 85 ca 00 00 00       jne    80ae582 <add_fdes.isra.0+0x282>
 80ae4b8:       89 7c 24 08             mov    %edi,0x8(%esp)
 80ae4bc:       c7 44 24 14 00 00 00    movl   $0x0,0x14(%esp)
 80ae4c3:       00
 80ae4c4:       85 ed                   test   %ebp,%ebp
 80ae4c6:       0f 84 14 ff ff ff       je     80ae3e0 <add_fdes.isra.0+0xe0>
 80ae4cc:       83 ec 0c                sub    $0xc,%esp
 80ae4cf:       8d 4b 08                lea    0x8(%ebx),%ecx
 80ae4d2:       31 d2                   xor    %edx,%edx
 80ae4d4:       8d 44 24 38             lea    0x38(%esp),%eax
 80ae4d8:       50                      push   %eax
 80ae4d9:       89 e8                   mov    %ebp,%eax
 80ae4db:       e8 90 e3 ff ff          call   80ac870 <read_encoded_value_with_base>
 80ae4e0:       89 e8                   mov    %ebp,%eax
 80ae4e2:       88 44 24 23             mov    %al,0x23(%esp)
 80ae4e6:       83 c4 10                add    $0x10,%esp
 80ae4e9:       e9 58 ff ff ff          jmp    80ae446 <add_fdes.isra.0+0x146>
 80ae4ee:       66 90                   xchg   %ax,%ax
 80ae4f0:       8b 44 24 04             mov    0x4(%esp),%eax
 80ae4f4:       89 7c 24 08             mov    %edi,0x8(%esp)
 80ae4f8:       8b 40 04                mov    0x4(%eax),%eax
 80ae4fb:       89 44 24 14             mov    %eax,0x14(%esp)
 80ae4ff:       85 ed                   test   %ebp,%ebp
 80ae501:       0f 84 d9 fe ff ff       je     80ae3e0 <add_fdes.isra.0+0xe0>
 80ae507:       83 ec 0c                sub    $0xc,%esp
 80ae50a:       8d 4b 08                lea    0x8(%ebx),%ecx
 80ae50d:       8d 44 24 38             lea    0x38(%esp),%eax
 80ae511:       50                      push   %eax
 80ae512:       8b 54 24 24             mov    0x24(%esp),%edx
 80ae516:       89 e8                   mov    %ebp,%eax
 80ae518:       e8 53 e3 ff ff          call   80ac870 <read_encoded_value_with_base>
 80ae51d:       83 c4 10                add    $0x10,%esp
 80ae520:       e9 21 ff ff ff          jmp    80ae446 <add_fdes.isra.0+0x146>
 80ae525:       8d 76 00                lea    0x0(%esi),%esi
 80ae528:       83 ec 0c                sub    $0xc,%esp
 80ae52b:       8d 4b 08                lea    0x8(%ebx),%ecx
 80ae52e:       31 d2                   xor    %edx,%edx
 80ae530:       8d 44 24 38             lea    0x38(%esp),%eax
 80ae534:       50                      push   %eax
 80ae535:       b8 ff 00 00 00          mov    $0xff,%eax
 80ae53a:       e8 31 e3 ff ff          call   80ac870 <read_encoded_value_with_base>
 80ae53f:       83 c4 10                add    $0x10,%esp
 80ae542:       89 7c 24 08             mov    %edi,0x8(%esp)
 80ae546:       c7 44 24 14 00 00 00    movl   $0x0,0x14(%esp)
 80ae54d:       00
 80ae54e:       e9 ab fe ff ff          jmp    80ae3fe <add_fdes.isra.0+0xfe>
 80ae553:       2e 8d 74 26 00          lea    %cs:0x0(%esi,%eiz,1),%esi
 80ae558:       8b 44 24 04             mov    0x4(%esp),%eax
 80ae55c:       8b 40 04                mov    0x4(%eax),%eax
 80ae55f:       89 44 24 14             mov    %eax,0x14(%esp)
 80ae563:       e9 f4 fd ff ff          jmp    80ae35c <add_fdes.isra.0+0x5c>
 80ae568:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ae56f:       00
 80ae570:       c7 44 24 14 00 00 00    movl   $0x0,0x14(%esp)
 80ae577:       00
 80ae578:       bd ff 00 00 00          mov    $0xff,%ebp
 80ae57d:       e9 da fd ff ff          jmp    80ae35c <add_fdes.isra.0+0x5c>
 80ae582:       e9 bb b0 f9 ff          jmp    8049642 <add_fdes.isra.0.cold>
 80ae587:       e9 b6 b0 f9 ff          jmp    8049642 <add_fdes.isra.0.cold>
 80ae58c:       e9 b1 b0 f9 ff          jmp    8049642 <add_fdes.isra.0.cold>
 80ae591:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ae598:       00
 80ae599:       8d b4 26 00 00 00 00    lea    0x0(%esi,%eiz,1),%esi

080ae5a0 <linear_search_fdes>:
 80ae5a0:       55                      push   %ebp
 80ae5a1:       57                      push   %edi
 80ae5a2:       e8 8c d8 f9 ff          call   804be33 <__x86.get_pc_thunk.di>
 80ae5a7:       81 c7 4d 8a 03 00       add    $0x38a4d,%edi
 80ae5ad:       56                      push   %esi
 80ae5ae:       53                      push   %ebx
 80ae5af:       89 d3                   mov    %edx,%ebx
 80ae5b1:       83 ec 3c                sub    $0x3c,%esp
 80ae5b4:       89 44 24 04             mov    %eax,0x4(%esp)
 80ae5b8:       0f b7 40 10             movzwl 0x10(%eax),%eax
 80ae5bc:       89 7c 24 1c             mov    %edi,0x1c(%esp)
 80ae5c0:       66 89 44 24 08          mov    %ax,0x8(%esp)
 80ae5c5:       66 c1 e8 03             shr    $0x3,%ax
 80ae5c9:       89 4c 24 0c             mov    %ecx,0xc(%esp)
 80ae5cd:       3c ff                   cmp    $0xff,%al
 80ae5cf:       0f 84 7b 02 00 00       je     80ae850 <linear_search_fdes+0x2b0>
 80ae5d5:       0f b6 e8                movzbl %al,%ebp
 80ae5d8:       83 e0 70                and    $0x70,%eax
 80ae5db:       3c 20                   cmp    $0x20,%al
 80ae5dd:       0f 84 5d 02 00 00       je     80ae840 <linear_search_fdes+0x2a0>
 80ae5e3:       0f 86 64 01 00 00       jbe    80ae74d <linear_search_fdes+0x1ad>
 80ae5e9:       3c 30                   cmp    $0x30,%al
 80ae5eb:       0f 85 54 01 00 00       jne    80ae745 <linear_search_fdes+0x1a5>
 80ae5f1:       8b 44 24 04             mov    0x4(%esp),%eax
 80ae5f5:       8b 40 08                mov    0x8(%eax),%eax
 80ae5f8:       89 44 24 14             mov    %eax,0x14(%esp)
 80ae5fc:       8b 33                   mov    (%ebx),%esi
 80ae5fe:       8d 44 24 28             lea    0x28(%esp),%eax
 80ae602:       c7 44 24 08 00 00 00    movl   $0x0,0x8(%esp)
 80ae609:       00
 80ae60a:       89 44 24 18             mov    %eax,0x18(%esp)
 80ae60e:       85 f6                   test   %esi,%esi
 80ae610:       0f 85 9a 00 00 00       jne    80ae6b0 <linear_search_fdes+0x110>
 80ae616:       e9 45 01 00 00          jmp    80ae760 <linear_search_fdes+0x1c0>
 80ae61b:       2e 8d 74 26 00          lea    %cs:0x0(%esi,%eiz,1),%esi
 80ae620:       8d 7b 04                lea    0x4(%ebx),%edi
 80ae623:       29 c7                   sub    %eax,%edi
 80ae625:       8b 44 24 08             mov    0x8(%esp),%eax
 80ae629:       39 c7                   cmp    %eax,%edi
 80ae62b:       0f 84 94 00 00 00       je     80ae6c5 <linear_search_fdes+0x125>
 80ae631:       89 f8                   mov    %edi,%eax
 80ae633:       e8 a8 eb ff ff          call   80ad1e0 <get_cie_encoding>
 80ae638:       88 44 24 13             mov    %al,0x13(%esp)
 80ae63c:       89 c5                   mov    %eax,%ebp
 80ae63e:       3d ff 00 00 00          cmp    $0xff,%eax
 80ae643:       0f 84 b7 01 00 00       je     80ae800 <linear_search_fdes+0x260>
 80ae649:       83 e0 70                and    $0x70,%eax
 80ae64c:       3c 20                   cmp    $0x20,%al
 80ae64e:       0f 84 5c 01 00 00       je     80ae7b0 <linear_search_fdes+0x210>
 80ae654:       0f 86 3e 01 00 00       jbe    80ae798 <linear_search_fdes+0x1f8>
 80ae65a:       3c 30                   cmp    $0x30,%al
 80ae65c:       0f 85 2e 01 00 00       jne    80ae790 <linear_search_fdes+0x1f0>
 80ae662:       8b 44 24 04             mov    0x4(%esp),%eax
 80ae666:       89 7c 24 08             mov    %edi,0x8(%esp)
 80ae66a:       8b 40 08                mov    0x8(%eax),%eax
 80ae66d:       89 44 24 14             mov    %eax,0x14(%esp)
 80ae671:       85 ed                   test   %ebp,%ebp
 80ae673:       0f 85 4e 01 00 00       jne    80ae7c7 <linear_search_fdes+0x227>
 80ae679:       8d b4 26 00 00 00 00    lea    0x0(%esi,%eiz,1),%esi
 80ae680:       8b 43 08                mov    0x8(%ebx),%eax
 80ae683:       8b 53 0c                mov    0xc(%ebx),%edx
 80ae686:       31 ed                   xor    %ebp,%ebp
 80ae688:       89 44 24 28             mov    %eax,0x28(%esp)
 80ae68c:       89 54 24 2c             mov    %edx,0x2c(%esp)
 80ae690:       85 c0                   test   %eax,%eax
 80ae692:       74 0e                   je     80ae6a2 <linear_search_fdes+0x102>
 80ae694:       8b 4c 24 0c             mov    0xc(%esp),%ecx
 80ae698:       29 c1                   sub    %eax,%ecx
 80ae69a:       39 d1                   cmp    %edx,%ecx
 80ae69c:       0f 82 99 00 00 00       jb     80ae73b <linear_search_fdes+0x19b>
 80ae6a2:       8d 5c 33 04             lea    0x4(%ebx,%esi,1),%ebx
 80ae6a6:       8b 33                   mov    (%ebx),%esi
 80ae6a8:       85 f6                   test   %esi,%esi
 80ae6aa:       0f 84 b0 00 00 00       je     80ae760 <linear_search_fdes+0x1c0>
 80ae6b0:       8b 43 04                mov    0x4(%ebx),%eax
 80ae6b3:       85 c0                   test   %eax,%eax
 80ae6b5:       74 eb                   je     80ae6a2 <linear_search_fdes+0x102>
 80ae6b7:       8b 7c 24 04             mov    0x4(%esp),%edi
 80ae6bb:       f6 47 10 04             testb  $0x4,0x10(%edi)
 80ae6bf:       0f 85 5b ff ff ff       jne    80ae620 <linear_search_fdes+0x80>
 80ae6c5:       85 ed                   test   %ebp,%ebp
 80ae6c7:       74 b7                   je     80ae680 <linear_search_fdes+0xe0>
 80ae6c9:       83 ec 0c                sub    $0xc,%esp
 80ae6cc:       89 e8                   mov    %ebp,%eax
 80ae6ce:       8d 4b 08                lea    0x8(%ebx),%ecx
 80ae6d1:       ff 74 24 24             push   0x24(%esp)
 80ae6d5:       8b 54 24 24             mov    0x24(%esp),%edx
 80ae6d9:       0f b6 c0                movzbl %al,%eax
 80ae6dc:       e8 8f e1 ff ff          call   80ac870 <read_encoded_value_with_base>
 80ae6e1:       5a                      pop    %edx
 80ae6e2:       8d 54 24 38             lea    0x38(%esp),%edx
 80ae6e6:       89 c1                   mov    %eax,%ecx
 80ae6e8:       89 e8                   mov    %ebp,%eax
 80ae6ea:       52                      push   %edx
 80ae6eb:       83 e0 0f                and    $0xf,%eax
 80ae6ee:       31 d2                   xor    %edx,%edx
 80ae6f0:       e8 7b e1 ff ff          call   80ac870 <read_encoded_value_with_base>
 80ae6f5:       89 e8                   mov    %ebp,%eax
 80ae6f7:       88 44 24 23             mov    %al,0x23(%esp)
 80ae6fb:       83 c4 10                add    $0x10,%esp
 80ae6fe:       3c ff                   cmp    $0xff,%al
 80ae700:       74 a0                   je     80ae6a2 <linear_search_fdes+0x102>
 80ae702:       0f b6 44 24 13          movzbl 0x13(%esp),%eax
 80ae707:       83 e0 07                and    $0x7,%eax
 80ae70a:       3c 02                   cmp    $0x2,%al
 80ae70c:       74 72                   je     80ae780 <linear_search_fdes+0x1e0>
 80ae70e:       77 60                   ja     80ae770 <linear_search_fdes+0x1d0>
 80ae710:       84 c0                   test   %al,%al
 80ae712:       0f 85 4f 01 00 00       jne    80ae867 <linear_search_fdes+0x2c7>
 80ae718:       ba ff ff ff ff          mov    $0xffffffff,%edx
 80ae71d:       8b 44 24 28             mov    0x28(%esp),%eax
 80ae721:       85 d0                   test   %edx,%eax
 80ae723:       0f 84 79 ff ff ff       je     80ae6a2 <linear_search_fdes+0x102>
 80ae729:       8b 4c 24 0c             mov    0xc(%esp),%ecx
 80ae72d:       8b 54 24 2c             mov    0x2c(%esp),%edx
 80ae731:       29 c1                   sub    %eax,%ecx
 80ae733:       39 d1                   cmp    %edx,%ecx
 80ae735:       0f 83 67 ff ff ff       jae    80ae6a2 <linear_search_fdes+0x102>
 80ae73b:       83 c4 3c                add    $0x3c,%esp
 80ae73e:       89 d8                   mov    %ebx,%eax
 80ae740:       5b                      pop    %ebx
 80ae741:       5e                      pop    %esi
 80ae742:       5f                      pop    %edi
 80ae743:       5d                      pop    %ebp
 80ae744:       c3                      ret
 80ae745:       3c 50                   cmp    $0x50,%al
 80ae747:       0f 85 1f 01 00 00       jne    80ae86c <linear_search_fdes+0x2cc>
 80ae74d:       c7 44 24 14 00 00 00    movl   $0x0,0x14(%esp)
 80ae754:       00
 80ae755:       e9 a2 fe ff ff          jmp    80ae5fc <linear_search_fdes+0x5c>
 80ae75a:       8d b6 00 00 00 00       lea    0x0(%esi),%esi
 80ae760:       83 c4 3c                add    $0x3c,%esp
 80ae763:       31 c0                   xor    %eax,%eax
 80ae765:       5b                      pop    %ebx
 80ae766:       5e                      pop    %esi
 80ae767:       5f                      pop    %edi
 80ae768:       5d                      pop    %ebp
 80ae769:       c3                      ret
 80ae76a:       8d b6 00 00 00 00       lea    0x0(%esi),%esi
 80ae770:       83 e8 03                sub    $0x3,%eax
 80ae773:       3c 01                   cmp    $0x1,%al
 80ae775:       76 a1                   jbe    80ae718 <linear_search_fdes+0x178>
 80ae777:       e9 cf ae f9 ff          jmp    804964b <linear_search_fdes.cold>
 80ae77c:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi
 80ae780:       ba ff ff 00 00          mov    $0xffff,%edx
 80ae785:       eb 96                   jmp    80ae71d <linear_search_fdes+0x17d>
 80ae787:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ae78e:       00
 80ae78f:       90                      nop
 80ae790:       3c 50                   cmp    $0x50,%al
 80ae792:       0f 85 ca 00 00 00       jne    80ae862 <linear_search_fdes+0x2c2>
 80ae798:       89 7c 24 08             mov    %edi,0x8(%esp)
 80ae79c:       c7 44 24 14 00 00 00    movl   $0x0,0x14(%esp)
 80ae7a3:       00
 80ae7a4:       e9 1c ff ff ff          jmp    80ae6c5 <linear_search_fdes+0x125>
 80ae7a9:       8d b4 26 00 00 00 00    lea    0x0(%esi,%eiz,1),%esi
 80ae7b0:       8b 44 24 04             mov    0x4(%esp),%eax
 80ae7b4:       89 7c 24 08             mov    %edi,0x8(%esp)
 80ae7b8:       8b 40 04                mov    0x4(%eax),%eax
 80ae7bb:       89 44 24 14             mov    %eax,0x14(%esp)
 80ae7bf:       85 ed                   test   %ebp,%ebp
 80ae7c1:       0f 84 b9 fe ff ff       je     80ae680 <linear_search_fdes+0xe0>
 80ae7c7:       83 ec 0c                sub    $0xc,%esp
 80ae7ca:       8d 4b 08                lea    0x8(%ebx),%ecx
 80ae7cd:       8d 44 24 34             lea    0x34(%esp),%eax
 80ae7d1:       50                      push   %eax
 80ae7d2:       8b 54 24 24             mov    0x24(%esp),%edx
 80ae7d6:       89 e8                   mov    %ebp,%eax
 80ae7d8:       e8 93 e0 ff ff          call   80ac870 <read_encoded_value_with_base>
 80ae7dd:       5a                      pop    %edx
 80ae7de:       8d 54 24 38             lea    0x38(%esp),%edx
 80ae7e2:       89 c1                   mov    %eax,%ecx
 80ae7e4:       89 e8                   mov    %ebp,%eax
 80ae7e6:       52                      push   %edx
 80ae7e7:       83 e0 0f                and    $0xf,%eax
 80ae7ea:       31 d2                   xor    %edx,%edx
 80ae7ec:       e8 7f e0 ff ff          call   80ac870 <read_encoded_value_with_base>
 80ae7f1:       83 c4 10                add    $0x10,%esp
 80ae7f4:       e9 09 ff ff ff          jmp    80ae702 <linear_search_fdes+0x162>
 80ae7f9:       8d b4 26 00 00 00 00    lea    0x0(%esi,%eiz,1),%esi
 80ae800:       83 ec 0c                sub    $0xc,%esp
 80ae803:       8d 4b 08                lea    0x8(%ebx),%ecx
 80ae806:       31 d2                   xor    %edx,%edx
 80ae808:       8d 44 24 34             lea    0x34(%esp),%eax
 80ae80c:       50                      push   %eax
 80ae80d:       b8 ff 00 00 00          mov    $0xff,%eax
 80ae812:       e8 59 e0 ff ff          call   80ac870 <read_encoded_value_with_base>
 80ae817:       31 d2                   xor    %edx,%edx
 80ae819:       89 c1                   mov    %eax,%ecx
 80ae81b:       58                      pop    %eax
 80ae81c:       8d 44 24 38             lea    0x38(%esp),%eax
 80ae820:       50                      push   %eax
 80ae821:       b8 0f 00 00 00          mov    $0xf,%eax
 80ae826:       e8 45 e0 ff ff          call   80ac870 <read_encoded_value_with_base>
 80ae82b:       83 c4 10                add    $0x10,%esp
 80ae82e:       89 7c 24 08             mov    %edi,0x8(%esp)
 80ae832:       c7 44 24 14 00 00 00    movl   $0x0,0x14(%esp)
 80ae839:       00
 80ae83a:       e9 63 fe ff ff          jmp    80ae6a2 <linear_search_fdes+0x102>
 80ae83f:       90                      nop
 80ae840:       8b 44 24 04             mov    0x4(%esp),%eax
 80ae844:       8b 40 04                mov    0x4(%eax),%eax
 80ae847:       89 44 24 14             mov    %eax,0x14(%esp)
 80ae84b:       e9 ac fd ff ff          jmp    80ae5fc <linear_search_fdes+0x5c>
 80ae850:       c7 44 24 14 00 00 00    movl   $0x0,0x14(%esp)
 80ae857:       00
 80ae858:       bd ff 00 00 00          mov    $0xff,%ebp
 80ae85d:       e9 9a fd ff ff          jmp    80ae5fc <linear_search_fdes+0x5c>
 80ae862:       e9 e4 ad f9 ff          jmp    804964b <linear_search_fdes.cold>
 80ae867:       e9 df ad f9 ff          jmp    804964b <linear_search_fdes.cold>
 80ae86c:       e9 da ad f9 ff          jmp    804964b <linear_search_fdes.cold>
 80ae871:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ae878:       00
 80ae879:       8d b4 26 00 00 00 00    lea    0x0(%esi,%eiz,1),%esi

080ae880 <__register_frame_info_bases>:
 80ae880:       f3 0f 1e fb             endbr32
 80ae884:       56                      push   %esi
 80ae885:       e8 a5 d5 f9 ff          call   804be2f <__x86.get_pc_thunk.si>
 80ae88a:       81 c6 6a 87 03 00       add    $0x3876a,%esi
 80ae890:       53                      push   %ebx
 80ae891:       83 ec 14                sub    $0x14,%esp
 80ae894:       8b 54 24 20             mov    0x20(%esp),%edx
 80ae898:       8b 5c 24 24             mov    0x24(%esp),%ebx
 80ae89c:       8b 4c 24 28             mov    0x28(%esp),%ecx
 80ae8a0:       8b 44 24 2c             mov    0x2c(%esp),%eax
 80ae8a4:       85 d2                   test   %edx,%edx
 80ae8a6:       74 05                   je     80ae8ad <__register_frame_info_bases+0x2d>
 80ae8a8:       83 3a 00                cmpl   $0x0,(%edx)
 80ae8ab:       75 0b                   jne    80ae8b8 <__register_frame_info_bases+0x38>
 80ae8ad:       83 c4 14                add    $0x14,%esp
 80ae8b0:       5b                      pop    %ebx
 80ae8b1:       5e                      pop    %esi
 80ae8b2:       c3                      ret
 80ae8b3:       2e 8d 74 26 00          lea    %cs:0x0(%esi,%eiz,1),%esi
 80ae8b8:       83 ec 0c                sub    $0xc,%esp
 80ae8bb:       89 4b 04                mov    %ecx,0x4(%ebx)
 80ae8be:       b9 01 00 00 00          mov    $0x1,%ecx
 80ae8c3:       89 43 08                mov    %eax,0x8(%ebx)
 80ae8c6:       8d 86 28 3c 00 00       lea    0x3c28(%esi),%eax
 80ae8cc:       89 53 0c                mov    %edx,0xc(%ebx)
 80ae8cf:       c7 03 ff ff ff ff       movl   $0xffffffff,(%ebx)
 80ae8d5:       c7 43 10 f8 07 00 00    movl   $0x7f8,0x10(%ebx)
 80ae8dc:       53                      push   %ebx
 80ae8dd:       e8 fe e4 ff ff          call   80acde0 <btree_insert.isra.0>
 80ae8e2:       8d 54 24 18             lea    0x18(%esp),%edx
 80ae8e6:       89 d8                   mov    %ebx,%eax
 80ae8e8:       e8 93 f6 ff ff          call   80adf80 <get_pc_range>
 80ae8ed:       8b 54 24 18             mov    0x18(%esp),%edx
 80ae8f1:       8b 4c 24 1c             mov    0x1c(%esp),%ecx
 80ae8f5:       89 5c 24 30             mov    %ebx,0x30(%esp)
 80ae8f9:       83 c4 24                add    $0x24,%esp
 80ae8fc:       8d 86 34 3c 00 00       lea    0x3c34(%esi),%eax
 80ae902:       5b                      pop    %ebx
 80ae903:       29 d1                   sub    %edx,%ecx
 80ae905:       5e                      pop    %esi
 80ae906:       e9 d5 e4 ff ff          jmp    80acde0 <btree_insert.isra.0>
 80ae90b:       2e 8d 74 26 00          lea    %cs:0x0(%esi,%eiz,1),%esi

080ae910 <__register_frame_info>:
 80ae910:       f3 0f 1e fb             endbr32
 80ae914:       56                      push   %esi
 80ae915:       e8 15 d5 f9 ff          call   804be2f <__x86.get_pc_thunk.si>
 80ae91a:       81 c6 da 86 03 00       add    $0x386da,%esi
 80ae920:       53                      push   %ebx
 80ae921:       83 ec 14                sub    $0x14,%esp
 80ae924:       8b 54 24 20             mov    0x20(%esp),%edx
 80ae928:       8b 5c 24 24             mov    0x24(%esp),%ebx
 80ae92c:       85 d2                   test   %edx,%edx
 80ae92e:       74 06                   je     80ae936 <__register_frame_info+0x26>
 80ae930:       8b 02                   mov    (%edx),%eax
 80ae932:       85 c0                   test   %eax,%eax
 80ae934:       75 0a                   jne    80ae940 <__register_frame_info+0x30>
 80ae936:       83 c4 14                add    $0x14,%esp
 80ae939:       5b                      pop    %ebx
 80ae93a:       5e                      pop    %esi
 80ae93b:       c3                      ret
 80ae93c:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi
 80ae940:       83 ec 0c                sub    $0xc,%esp
 80ae943:       89 53 0c                mov    %edx,0xc(%ebx)
 80ae946:       b9 01 00 00 00          mov    $0x1,%ecx
 80ae94b:       8d 86 28 3c 00 00       lea    0x3c28(%esi),%eax
 80ae951:       c7 03 ff ff ff ff       movl   $0xffffffff,(%ebx)
 80ae957:       c7 43 04 00 00 00 00    movl   $0x0,0x4(%ebx)
 80ae95e:       c7 43 08 00 00 00 00    movl   $0x0,0x8(%ebx)
 80ae965:       c7 43 10 f8 07 00 00    movl   $0x7f8,0x10(%ebx)
 80ae96c:       53                      push   %ebx
 80ae96d:       e8 6e e4 ff ff          call   80acde0 <btree_insert.isra.0>
 80ae972:       8d 54 24 18             lea    0x18(%esp),%edx
 80ae976:       89 d8                   mov    %ebx,%eax
 80ae978:       e8 03 f6 ff ff          call   80adf80 <get_pc_range>
 80ae97d:       8b 54 24 18             mov    0x18(%esp),%edx
 80ae981:       8b 4c 24 1c             mov    0x1c(%esp),%ecx
 80ae985:       89 5c 24 30             mov    %ebx,0x30(%esp)
 80ae989:       83 c4 24                add    $0x24,%esp
 80ae98c:       8d 86 34 3c 00 00       lea    0x3c34(%esi),%eax
 80ae992:       5b                      pop    %ebx
 80ae993:       29 d1                   sub    %edx,%ecx
 80ae995:       5e                      pop    %esi
 80ae996:       e9 45 e4 ff ff          jmp    80acde0 <btree_insert.isra.0>
 80ae99b:       2e 8d 74 26 00          lea    %cs:0x0(%esi,%eiz,1),%esi

080ae9a0 <__register_frame>:
 80ae9a0:       f3 0f 1e fb             endbr32
 80ae9a4:       57                      push   %edi
 80ae9a5:       e8 89 d4 f9 ff          call   804be33 <__x86.get_pc_thunk.di>
 80ae9aa:       81 c7 4a 86 03 00       add    $0x3864a,%edi
 80ae9b0:       56                      push   %esi
 80ae9b1:       53                      push   %ebx
 80ae9b2:       83 ec 10                sub    $0x10,%esp
 80ae9b5:       8b 74 24 20             mov    0x20(%esp),%esi
 80ae9b9:       8b 06                   mov    (%esi),%eax
 80ae9bb:       85 c0                   test   %eax,%eax
 80ae9bd:       75 11                   jne    80ae9d0 <__register_frame+0x30>
 80ae9bf:       83 c4 10                add    $0x10,%esp
 80ae9c2:       5b                      pop    %ebx
 80ae9c3:       5e                      pop    %esi
 80ae9c4:       5f                      pop    %edi
 80ae9c5:       c3                      ret
 80ae9c6:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80ae9cd:       00
 80ae9ce:       66 90                   xchg   %ax,%ax
 80ae9d0:       83 ec 0c                sub    $0xc,%esp
 80ae9d3:       89 fb                   mov    %edi,%ebx
 80ae9d5:       6a 18                   push   $0x18
 80ae9d7:       e8 34 89 fa ff          call   8057310 <__libc_malloc>
 80ae9dc:       b9 01 00 00 00          mov    $0x1,%ecx
 80ae9e1:       89 f2                   mov    %esi,%edx
 80ae9e3:       89 70 0c                mov    %esi,0xc(%eax)
 80ae9e6:       89 c3                   mov    %eax,%ebx
 80ae9e8:       c7 00 ff ff ff ff       movl   $0xffffffff,(%eax)
 80ae9ee:       c7 40 04 00 00 00 00    movl   $0x0,0x4(%eax)
 80ae9f5:       c7 40 08 00 00 00 00    movl   $0x0,0x8(%eax)
 80ae9fc:       c7 40 10 f8 07 00 00    movl   $0x7f8,0x10(%eax)
 80aea03:       89 04 24                mov    %eax,(%esp)
 80aea06:       8d 87 28 3c 00 00       lea    0x3c28(%edi),%eax
 80aea0c:       e8 cf e3 ff ff          call   80acde0 <btree_insert.isra.0>
 80aea11:       8d 54 24 18             lea    0x18(%esp),%edx
 80aea15:       89 d8                   mov    %ebx,%eax
 80aea17:       e8 64 f5 ff ff          call   80adf80 <get_pc_range>
 80aea1c:       8b 54 24 18             mov    0x18(%esp),%edx
 80aea20:       8b 4c 24 1c             mov    0x1c(%esp),%ecx
 80aea24:       89 5c 24 30             mov    %ebx,0x30(%esp)
 80aea28:       83 c4 20                add    $0x20,%esp
 80aea2b:       8d 87 34 3c 00 00       lea    0x3c34(%edi),%eax
 80aea31:       5b                      pop    %ebx
 80aea32:       29 d1                   sub    %edx,%ecx
 80aea34:       5e                      pop    %esi
 80aea35:       5f                      pop    %edi
 80aea36:       e9 a5 e3 ff ff          jmp    80acde0 <btree_insert.isra.0>
 80aea3b:       2e 8d 74 26 00          lea    %cs:0x0(%esi,%eiz,1),%esi

080aea40 <__register_frame_info_table_bases>:
 80aea40:       f3 0f 1e fb             endbr32
 80aea44:       56                      push   %esi
 80aea45:       b9 01 00 00 00          mov    $0x1,%ecx
 80aea4a:       e8 e0 d3 f9 ff          call   804be2f <__x86.get_pc_thunk.si>
 80aea4f:       81 c6 a5 85 03 00       add    $0x385a5,%esi
 80aea55:       53                      push   %ebx
 80aea56:       83 ec 20                sub    $0x20,%esp
 80aea59:       8b 5c 24 30             mov    0x30(%esp),%ebx
 80aea5d:       8b 44 24 34             mov    0x34(%esp),%eax
 80aea61:       8b 54 24 2c             mov    0x2c(%esp),%edx
 80aea65:       89 43 04                mov    %eax,0x4(%ebx)
 80aea68:       8b 44 24 38             mov    0x38(%esp),%eax
 80aea6c:       89 53 0c                mov    %edx,0xc(%ebx)
 80aea6f:       89 43 08                mov    %eax,0x8(%ebx)
 80aea72:       8d 86 28 3c 00 00       lea    0x3c28(%esi),%eax
 80aea78:       c7 03 ff ff ff ff       movl   $0xffffffff,(%ebx)
 80aea7e:       c7 43 10 fa 07 00 00    movl   $0x7fa,0x10(%ebx)
 80aea85:       53                      push   %ebx
 80aea86:       e8 55 e3 ff ff          call   80acde0 <btree_insert.isra.0>
 80aea8b:       8d 54 24 18             lea    0x18(%esp),%edx
 80aea8f:       89 d8                   mov    %ebx,%eax
 80aea91:       e8 ea f4 ff ff          call   80adf80 <get_pc_range>
 80aea96:       8b 54 24 18             mov    0x18(%esp),%edx
 80aea9a:       8b 4c 24 1c             mov    0x1c(%esp),%ecx
 80aea9e:       89 5c 24 30             mov    %ebx,0x30(%esp)
 80aeaa2:       83 c4 24                add    $0x24,%esp
 80aeaa5:       8d 86 34 3c 00 00       lea    0x3c34(%esi),%eax
 80aeaab:       5b                      pop    %ebx
 80aeaac:       29 d1                   sub    %edx,%ecx
 80aeaae:       5e                      pop    %esi
 80aeaaf:       e9 2c e3 ff ff          jmp    80acde0 <btree_insert.isra.0>
 80aeab4:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80aeabb:       00
 80aeabc:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi

080aeac0 <__register_frame_info_table>:
 80aeac0:       f3 0f 1e fb             endbr32
 80aeac4:       56                      push   %esi
 80aeac5:       b9 01 00 00 00          mov    $0x1,%ecx
 80aeaca:       e8 60 d3 f9 ff          call   804be2f <__x86.get_pc_thunk.si>
 80aeacf:       81 c6 25 85 03 00       add    $0x38525,%esi
 80aead5:       53                      push   %ebx
 80aead6:       83 ec 20                sub    $0x20,%esp
 80aead9:       8b 5c 24 30             mov    0x30(%esp),%ebx
 80aeadd:       8b 54 24 2c             mov    0x2c(%esp),%edx
 80aeae1:       c7 03 ff ff ff ff       movl   $0xffffffff,(%ebx)
 80aeae7:       8d 86 28 3c 00 00       lea    0x3c28(%esi),%eax
 80aeaed:       89 53 0c                mov    %edx,0xc(%ebx)
 80aeaf0:       c7 43 04 00 00 00 00    movl   $0x0,0x4(%ebx)
 80aeaf7:       c7 43 08 00 00 00 00    movl   $0x0,0x8(%ebx)
 80aeafe:       c7 43 10 fa 07 00 00    movl   $0x7fa,0x10(%ebx)
 80aeb05:       53                      push   %ebx
 80aeb06:       e8 d5 e2 ff ff          call   80acde0 <btree_insert.isra.0>
 80aeb0b:       8d 54 24 18             lea    0x18(%esp),%edx
 80aeb0f:       89 d8                   mov    %ebx,%eax
 80aeb11:       e8 6a f4 ff ff          call   80adf80 <get_pc_range>
 80aeb16:       8b 54 24 18             mov    0x18(%esp),%edx
 80aeb1a:       8b 4c 24 1c             mov    0x1c(%esp),%ecx
 80aeb1e:       89 5c 24 30             mov    %ebx,0x30(%esp)
 80aeb22:       83 c4 24                add    $0x24,%esp
 80aeb25:       8d 86 34 3c 00 00       lea    0x3c34(%esi),%eax
 80aeb2b:       5b                      pop    %ebx
 80aeb2c:       29 d1                   sub    %edx,%ecx
 80aeb2e:       5e                      pop    %esi
 80aeb2f:       e9 ac e2 ff ff          jmp    80acde0 <btree_insert.isra.0>
 80aeb34:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80aeb3b:       00
 80aeb3c:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi

080aeb40 <__register_frame_table>:
 80aeb40:       f3 0f 1e fb             endbr32
 80aeb44:       57                      push   %edi
 80aeb45:       56                      push   %esi
 80aeb46:       e8 e4 d2 f9 ff          call   804be2f <__x86.get_pc_thunk.si>
 80aeb4b:       81 c6 a9 84 03 00       add    $0x384a9,%esi
 80aeb51:       53                      push   %ebx
 80aeb52:       83 ec 1c                sub    $0x1c,%esp
 80aeb55:       8b 7c 24 2c             mov    0x2c(%esp),%edi
 80aeb59:       6a 18                   push   $0x18
 80aeb5b:       89 f3                   mov    %esi,%ebx
 80aeb5d:       e8 ae 87 fa ff          call   8057310 <__libc_malloc>
 80aeb62:       b9 01 00 00 00          mov    $0x1,%ecx
 80aeb67:       89 fa                   mov    %edi,%edx
 80aeb69:       89 78 0c                mov    %edi,0xc(%eax)
 80aeb6c:       89 c3                   mov    %eax,%ebx
 80aeb6e:       c7 00 ff ff ff ff       movl   $0xffffffff,(%eax)
 80aeb74:       c7 40 04 00 00 00 00    movl   $0x0,0x4(%eax)
 80aeb7b:       c7 40 08 00 00 00 00    movl   $0x0,0x8(%eax)
 80aeb82:       c7 40 10 fa 07 00 00    movl   $0x7fa,0x10(%eax)
 80aeb89:       89 04 24                mov    %eax,(%esp)
 80aeb8c:       8d 86 28 3c 00 00       lea    0x3c28(%esi),%eax
 80aeb92:       e8 49 e2 ff ff          call   80acde0 <btree_insert.isra.0>
 80aeb97:       8d 54 24 18             lea    0x18(%esp),%edx
 80aeb9b:       89 d8                   mov    %ebx,%eax
 80aeb9d:       e8 de f3 ff ff          call   80adf80 <get_pc_range>
 80aeba2:       8b 54 24 18             mov    0x18(%esp),%edx
 80aeba6:       8b 4c 24 1c             mov    0x1c(%esp),%ecx
 80aebaa:       89 5c 24 30             mov    %ebx,0x30(%esp)
 80aebae:       83 c4 20                add    $0x20,%esp
 80aebb1:       8d 86 34 3c 00 00       lea    0x3c34(%esi),%eax
 80aebb7:       5b                      pop    %ebx
 80aebb8:       29 d1                   sub    %edx,%ecx
 80aebba:       5e                      pop    %esi
 80aebbb:       5f                      pop    %edi
 80aebbc:       e9 1f e2 ff ff          jmp    80acde0 <btree_insert.isra.0>
 80aebc1:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80aebc8:       00
 80aebc9:       8d b4 26 00 00 00 00    lea    0x0(%esi,%eiz,1),%esi

080aebd0 <__deregister_frame_info_bases>:
 80aebd0:       f3 0f 1e fb             endbr32
 80aebd4:       8b 44 24 04             mov    0x4(%esp),%eax
 80aebd8:       85 c0                   test   %eax,%eax
 80aebda:       74 14                   je     80aebf0 <__deregister_frame_info_bases+0x20>
 80aebdc:       8b 10                   mov    (%eax),%edx
 80aebde:       85 d2                   test   %edx,%edx
 80aebe0:       74 0e                   je     80aebf0 <__deregister_frame_info_bases+0x20>
 80aebe2:       e9 09 f4 ff ff          jmp    80adff0 <__deregister_frame_info_bases.part.0>
 80aebe7:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80aebee:       00
 80aebef:       90                      nop
 80aebf0:       31 c0                   xor    %eax,%eax
 80aebf2:       c3                      ret
 80aebf3:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80aebfa:       00
 80aebfb:       2e 8d 74 26 00          lea    %cs:0x0(%esi,%eiz,1),%esi

080aec00 <__deregister_frame_info>:
 80aec00:       f3 0f 1e fb             endbr32
 80aec04:       8b 44 24 04             mov    0x4(%esp),%eax
 80aec08:       85 c0                   test   %eax,%eax
 80aec0a:       74 14                   je     80aec20 <__deregister_frame_info+0x20>
 80aec0c:       8b 10                   mov    (%eax),%edx
 80aec0e:       85 d2                   test   %edx,%edx
 80aec10:       74 0e                   je     80aec20 <__deregister_frame_info+0x20>
 80aec12:       e9 d9 f3 ff ff          jmp    80adff0 <__deregister_frame_info_bases.part.0>
 80aec17:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80aec1e:       00
 80aec1f:       90                      nop
 80aec20:       31 c0                   xor    %eax,%eax
 80aec22:       c3                      ret
 80aec23:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80aec2a:       00
 80aec2b:       2e 8d 74 26 00          lea    %cs:0x0(%esi,%eiz,1),%esi

080aec30 <__deregister_frame>:
 80aec30:       f3 0f 1e fb             endbr32
 80aec34:       53                      push   %ebx
 80aec35:       e8 a6 ab f9 ff          call   80497e0 <__x86.get_pc_thunk.bx>
 80aec3a:       81 c3 ba 83 03 00       add    $0x383ba,%ebx
 80aec40:       83 ec 08                sub    $0x8,%esp
 80aec43:       8b 44 24 10             mov    0x10(%esp),%eax
 80aec47:       8b 10                   mov    (%eax),%edx
 80aec49:       85 d2                   test   %edx,%edx
 80aec4b:       75 0b                   jne    80aec58 <__deregister_frame+0x28>
 80aec4d:       83 c4 08                add    $0x8,%esp
 80aec50:       5b                      pop    %ebx
 80aec51:       c3                      ret
 80aec52:       8d b6 00 00 00 00       lea    0x0(%esi),%esi
 80aec58:       e8 93 f3 ff ff          call   80adff0 <__deregister_frame_info_bases.part.0>
 80aec5d:       83 ec 0c                sub    $0xc,%esp
 80aec60:       50                      push   %eax
 80aec61:       e8 fa 8c fa ff          call   8057960 <__free>
 80aec66:       83 c4 10                add    $0x10,%esp
 80aec69:       83 c4 08                add    $0x8,%esp
 80aec6c:       5b                      pop    %ebx
 80aec6d:       c3                      ret
 80aec6e:       66 90                   xchg   %ax,%ax

080aec70 <_Unwind_Find_FDE>:
 80aec70:       f3 0f 1e fb             endbr32
 80aec74:       e8 c8 ac f9 ff          call   8049941 <__x86.get_pc_thunk.ax>
 80aec79:       05 7b 83 03 00          add    $0x3837b,%eax
 80aec7e:       55                      push   %ebp
 80aec7f:       57                      push   %edi
 80aec80:       56                      push   %esi
 80aec81:       53                      push   %ebx
 80aec82:       81 ec bc 00 00 00       sub    $0xbc,%esp
 80aec88:       89 44 24 0c             mov    %eax,0xc(%esp)
 80aec8c:       8b ac 24 d0 00 00 00    mov    0xd0(%esp),%ebp
 80aec93:       8d 54 24 58             lea    0x58(%esp),%edx
 80aec97:       8b 80 34 3c 00 00       mov    0x3c34(%eax),%eax
 80aec9d:       85 c0                   test   %eax,%eax
 80aec9f:       0f 85 0b 01 00 00       jne    80aedb0 <_Unwind_Find_FDE+0x140>
 80aeca5:       83 ec 08                sub    $0x8,%esp
 80aeca8:       52                      push   %edx
 80aeca9:       55                      push   %ebp
 80aecaa:       8b 5c 24 1c             mov    0x1c(%esp),%ebx
 80aecae:       e8 4d 3b ff ff          call   80a2800 <__dl_find_object>
 80aecb3:       83 c4 10                add    $0x10,%esp
 80aecb6:       85 c0                   test   %eax,%eax
 80aecb8:       0f 85 ca 02 00 00       jne    80aef88 <_Unwind_Find_FDE+0x318>
 80aecbe:       8b 5c 24 6c             mov    0x6c(%esp),%ebx
 80aecc2:       85 db                   test   %ebx,%ebx
 80aecc4:       0f 84 d9 00 00 00       je     80aeda3 <_Unwind_Find_FDE+0x133>
 80aecca:       80 3b 01                cmpb   $0x1,(%ebx)
 80aeccd:       0f 85 b5 02 00 00       jne    80aef88 <_Unwind_Find_FDE+0x318>
 80aecd3:       8b 74 24 70             mov    0x70(%esp),%esi
 80aecd7:       0f b6 43 01             movzbl 0x1(%ebx),%eax
 80aecdb:       8d 4b 04                lea    0x4(%ebx),%ecx
 80aecde:       89 f7                   mov    %esi,%edi
 80aece0:       3c 1b                   cmp    $0x1b,%al
 80aece2:       0f 85 5b 0a 00 00       jne    80af743 <_Unwind_Find_FDE+0xad3>
 80aece8:       03 4b 04                add    0x4(%ebx),%ecx
 80aeceb:       89 4c 24 38             mov    %ecx,0x38(%esp)
 80aecef:       8d 4b 08                lea    0x8(%ebx),%ecx
 80aecf2:       0f b6 43 02             movzbl 0x2(%ebx),%eax
 80aecf6:       3c ff                   cmp    $0xff,%al
 80aecf8:       74 0a                   je     80aed04 <_Unwind_Find_FDE+0x94>
 80aecfa:       80 7b 03 3b             cmpb   $0x3b,0x3(%ebx)
 80aecfe:       0f 84 93 02 00 00       je     80aef97 <_Unwind_Find_FDE+0x327>
 80aed04:       8b 54 24 38             mov    0x38(%esp),%edx
 80aed08:       8d 44 24 40             lea    0x40(%esp),%eax
 80aed0c:       89 e9                   mov    %ebp,%ecx
 80aed0e:       89 74 24 48             mov    %esi,0x48(%esp)
 80aed12:       c7 44 24 40 00 00 00    movl   $0x0,0x40(%esp)
 80aed19:       00
 80aed1a:       c7 44 24 44 00 00 00    movl   $0x0,0x44(%esp)
 80aed21:       00
 80aed22:       89 54 24 4c             mov    %edx,0x4c(%esp)
 80aed26:       c7 44 24 50 04 00 00    movl   $0x4,0x50(%esp)
 80aed2d:       00
 80aed2e:       e8 6d f8 ff ff          call   80ae5a0 <linear_search_fdes>
 80aed33:       89 c3                   mov    %eax,%ebx
 80aed35:       85 c0                   test   %eax,%eax
 80aed37:       0f 84 4b 02 00 00       je     80aef88 <_Unwind_Find_FDE+0x318>
 80aed3d:       8d 40 04                lea    0x4(%eax),%eax
 80aed40:       2b 43 04                sub    0x4(%ebx),%eax
 80aed43:       e8 98 e4 ff ff          call   80ad1e0 <get_cie_encoding>
 80aed48:       8d 4b 08                lea    0x8(%ebx),%ecx
 80aed4b:       3d ff 00 00 00          cmp    $0xff,%eax
 80aed50:       0f 84 f2 08 00 00       je     80af648 <_Unwind_Find_FDE+0x9d8>
 80aed56:       89 c2                   mov    %eax,%edx
 80aed58:       83 e2 70                and    $0x70,%edx
 80aed5b:       80 fa 20                cmp    $0x20,%dl
 80aed5e:       0f 84 e4 08 00 00       je     80af648 <_Unwind_Find_FDE+0x9d8>
 80aed64:       0f 86 de 08 00 00       jbe    80af648 <_Unwind_Find_FDE+0x9d8>
 80aed6a:       80 fa 30                cmp    $0x30,%dl
 80aed6d:       0f 85 cc 08 00 00       jne    80af63f <_Unwind_Find_FDE+0x9cf>
 80aed73:       83 ec 0c                sub    $0xc,%esp
 80aed76:       8d 54 24 48             lea    0x48(%esp),%edx
 80aed7a:       52                      push   %edx
 80aed7b:       89 fa                   mov    %edi,%edx
 80aed7d:       e8 ee da ff ff          call   80ac870 <read_encoded_value_with_base>
 80aed82:       8b 84 24 e4 00 00 00    mov    0xe4(%esp),%eax
 80aed89:       89 70 04                mov    %esi,0x4(%eax)
 80aed8c:       8b b4 24 e4 00 00 00    mov    0xe4(%esp),%esi
 80aed93:       c7 00 00 00 00 00       movl   $0x0,(%eax)
 80aed99:       8b 44 24 4c             mov    0x4c(%esp),%eax
 80aed9d:       83 c4 10                add    $0x10,%esp
 80aeda0:       89 46 08                mov    %eax,0x8(%esi)
 80aeda3:       81 c4 bc 00 00 00       add    $0xbc,%esp
 80aeda9:       89 d8                   mov    %ebx,%eax
 80aedab:       5b                      pop    %ebx
 80aedac:       5e                      pop    %esi
 80aedad:       5f                      pop    %edi
 80aedae:       5d                      pop    %ebp
 80aedaf:       c3                      ret
 80aedb0:       8b 7c 24 0c             mov    0xc(%esp),%edi
 80aedb4:       8b 87 3c 3c 00 00       mov    0x3c3c(%edi),%eax
 80aedba:       a8 01                   test   $0x1,%al
 80aedbc:       75 f2                   jne    80aedb0 <_Unwind_Find_FDE+0x140>
 80aedbe:       8b 97 34 3c 00 00       mov    0x3c34(%edi),%edx
 80aedc4:       8b 8f 3c 3c 00 00       mov    0x3c3c(%edi),%ecx
 80aedca:       39 c8                   cmp    %ecx,%eax
 80aedcc:       75 e2                   jne    80aedb0 <_Unwind_Find_FDE+0x140>
 80aedce:       85 d2                   test   %edx,%edx
 80aedd0:       0f 84 a4 01 00 00       je     80aef7a <_Unwind_Find_FDE+0x30a>
 80aedd6:       89 d3                   mov    %edx,%ebx
 80aedd8:       8b 3a                   mov    (%edx),%edi
 80aedda:       f7 c7 01 00 00 00       test   $0x1,%edi
 80aede0:       75 ce                   jne    80aedb0 <_Unwind_Find_FDE+0x140>
 80aede2:       89 f9                   mov    %edi,%ecx
 80aede4:       89 df                   mov    %ebx,%edi
 80aede6:       8b 74 24 0c             mov    0xc(%esp),%esi
 80aedea:       8b 96 3c 3c 00 00       mov    0x3c3c(%esi),%edx
 80aedf0:       39 d0                   cmp    %edx,%eax
 80aedf2:       75 bc                   jne    80aedb0 <_Unwind_Find_FDE+0x140>
 80aedf4:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80aedfb:       00
 80aedfc:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi
 80aee00:       8b 47 08                mov    0x8(%edi),%eax
 80aee03:       8b 77 04                mov    0x4(%edi),%esi
 80aee06:       89 7c 24 08             mov    %edi,0x8(%esp)
 80aee0a:       8b 17                   mov    (%edi),%edx
 80aee0c:       39 d1                   cmp    %edx,%ecx
 80aee0e:       75 a0                   jne    80aedb0 <_Unwind_Find_FDE+0x140>
 80aee10:       85 f6                   test   %esi,%esi
 80aee12:       0f 84 62 01 00 00       je     80aef7a <_Unwind_Find_FDE+0x30a>
 80aee18:       85 c0                   test   %eax,%eax
 80aee1a:       75 6d                   jne    80aee89 <_Unwind_Find_FDE+0x219>
 80aee1c:       89 4c 24 10             mov    %ecx,0x10(%esp)
 80aee20:       8d 57 0c                lea    0xc(%edi),%edx
 80aee23:       eb 24                   jmp    80aee49 <_Unwind_Find_FDE+0x1d9>
 80aee25:       eb 19                   jmp    80aee40 <_Unwind_Find_FDE+0x1d0>
 80aee27:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80aee2e:       00
 80aee2f:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80aee36:       00
 80aee37:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80aee3e:       00
 80aee3f:       90                      nop
 80aee40:       8b 0a                   mov    (%edx),%ecx
 80aee42:       83 c2 08                add    $0x8,%edx
 80aee45:       39 e9                   cmp    %ebp,%ecx
 80aee47:       73 09                   jae    80aee52 <_Unwind_Find_FDE+0x1e2>
 80aee49:       89 c3                   mov    %eax,%ebx
 80aee4b:       83 c0 01                add    $0x1,%eax
 80aee4e:       39 c6                   cmp    %eax,%esi
 80aee50:       75 ee                   jne    80aee40 <_Unwind_Find_FDE+0x1d0>
 80aee52:       83 c3 02                add    $0x2,%ebx
 80aee55:       8b 4c 24 10             mov    0x10(%esp),%ecx
 80aee59:       8b 3c df                mov    (%edi,%ebx,8),%edi
 80aee5c:       8b 44 24 08             mov    0x8(%esp),%eax
 80aee60:       8b 00                   mov    (%eax),%eax
 80aee62:       39 c1                   cmp    %eax,%ecx
 80aee64:       0f 85 46 ff ff ff       jne    80aedb0 <_Unwind_Find_FDE+0x140>
 80aee6a:       8b 07                   mov    (%edi),%eax
 80aee6c:       a8 01                   test   $0x1,%al
 80aee6e:       0f 85 3c ff ff ff       jne    80aedb0 <_Unwind_Find_FDE+0x140>
 80aee74:       8b 5c 24 08             mov    0x8(%esp),%ebx
 80aee78:       8b 13                   mov    (%ebx),%edx
 80aee7a:       39 d1                   cmp    %edx,%ecx
 80aee7c:       0f 85 2e ff ff ff       jne    80aedb0 <_Unwind_Find_FDE+0x140>
 80aee82:       89 c1                   mov    %eax,%ecx
 80aee84:       e9 77 ff ff ff          jmp    80aee00 <_Unwind_Find_FDE+0x190>
 80aee89:       89 f8                   mov    %edi,%eax
 80aee8b:       89 7c 24 10             mov    %edi,0x10(%esp)
 80aee8f:       31 db                   xor    %ebx,%ebx
 80aee91:       89 4c 24 14             mov    %ecx,0x14(%esp)
 80aee95:       83 c0 0c                add    $0xc,%eax
 80aee98:       eb 14                   jmp    80aeeae <_Unwind_Find_FDE+0x23e>
 80aee9a:       8d b6 00 00 00 00       lea    0x0(%esi),%esi
 80aeea0:       8b 10                   mov    (%eax),%edx
 80aeea2:       8b 48 04                mov    0x4(%eax),%ecx
 80aeea5:       83 c0 0c                add    $0xc,%eax
 80aeea8:       01 ca                   add    %ecx,%edx
 80aeeaa:       39 d5                   cmp    %edx,%ebp
 80aeeac:       72 0e                   jb     80aeebc <_Unwind_Find_FDE+0x24c>
 80aeeae:       83 c3 01                add    $0x1,%ebx
 80aeeb1:       89 44 24 08             mov    %eax,0x8(%esp)
 80aeeb5:       8d 78 04                lea    0x4(%eax),%edi
 80aeeb8:       39 f3                   cmp    %esi,%ebx
 80aeeba:       72 e4                   jb     80aeea0 <_Unwind_Find_FDE+0x230>
 80aeebc:       8b 44 24 08             mov    0x8(%esp),%eax
 80aeec0:       89 fe                   mov    %edi,%esi
 80aeec2:       8b 7c 24 14             mov    0x14(%esp),%edi
 80aeec6:       8d 1c 5b                lea    (%ebx,%ebx,2),%ebx
 80aeec9:       8b 00                   mov    (%eax),%eax
 80aeecb:       8b 16                   mov    (%esi),%edx
 80aeecd:       8b 74 24 10             mov    0x10(%esp),%esi
 80aeed1:       8d 1c 9e                lea    (%esi,%ebx,4),%ebx
 80aeed4:       8b 5b 08                mov    0x8(%ebx),%ebx
 80aeed7:       8b 0e                   mov    (%esi),%ecx
 80aeed9:       39 cf                   cmp    %ecx,%edi
 80aeedb:       0f 85 cf fe ff ff       jne    80aedb0 <_Unwind_Find_FDE+0x140>
 80aeee1:       89 5c 24 08             mov    %ebx,0x8(%esp)
 80aeee5:       39 c5                   cmp    %eax,%ebp
 80aeee7:       0f 82 8d 00 00 00       jb     80aef7a <_Unwind_Find_FDE+0x30a>
 80aeeed:       01 d0                   add    %edx,%eax
 80aeeef:       39 c5                   cmp    %eax,%ebp
 80aeef1:       0f 83 83 00 00 00       jae    80aef7a <_Unwind_Find_FDE+0x30a>
 80aeef7:       83 7c 24 08 00          cmpl   $0x0,0x8(%esp)
 80aeefc:       74 7c                   je     80aef7a <_Unwind_Find_FDE+0x30a>
 80aeefe:       8b 44 24 08             mov    0x8(%esp),%eax
 80aef02:       8b 40 10                mov    0x10(%eax),%eax
 80aef05:       89 44 24 68             mov    %eax,0x68(%esp)
 80aef09:       a8 01                   test   $0x1,%al
 80aef0b:       0f 84 ff 01 00 00       je     80af110 <_Unwind_Find_FDE+0x4a0>
 80aef11:       8b 44 24 08             mov    0x8(%esp),%eax
 80aef15:       0f b6 70 10             movzbl 0x10(%eax),%esi
 80aef19:       f7 c6 01 00 00 00       test   $0x1,%esi
 80aef1f:       0f 84 13 01 00 00       je     80af038 <_Unwind_Find_FDE+0x3c8>
 80aef25:       83 e6 04                and    $0x4,%esi
 80aef28:       0f 85 ed 03 00 00       jne    80af31b <_Unwind_Find_FDE+0x6ab>
 80aef2e:       8b 44 24 08             mov    0x8(%esp),%eax
 80aef32:       0f b7 70 10             movzwl 0x10(%eax),%esi
 80aef36:       66 f7 c6 f8 07          test   $0x7f8,%si
 80aef3b:       0f 85 53 02 00 00       jne    80af194 <_Unwind_Find_FDE+0x524>
 80aef41:       8b 78 0c                mov    0xc(%eax),%edi
 80aef44:       66 89 74 24 10          mov    %si,0x10(%esp)
 80aef49:       31 c9                   xor    %ecx,%ecx
 80aef4b:       8b 57 04                mov    0x4(%edi),%edx
 80aef4e:       eb 26                   jmp    80aef76 <_Unwind_Find_FDE+0x306>
 80aef50:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80aef57:       00
 80aef58:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80aef5f:       00
 80aef60:       8d 04 11                lea    (%ecx,%edx,1),%eax
 80aef63:       d1 e8                   shr    $1,%eax
 80aef65:       8b 5c 87 08             mov    0x8(%edi,%eax,4),%ebx
 80aef69:       8b 73 08                mov    0x8(%ebx),%esi
 80aef6c:       39 f5                   cmp    %esi,%ebp
 80aef6e:       0f 83 2d 03 00 00       jae    80af2a1 <_Unwind_Find_FDE+0x631>
 80aef74:       89 c2                   mov    %eax,%edx
 80aef76:       39 d1                   cmp    %edx,%ecx
 80aef78:       72 e6                   jb     80aef60 <_Unwind_Find_FDE+0x2f0>
 80aef7a:       8d 54 24 58             lea    0x58(%esp),%edx
 80aef7e:       e9 22 fd ff ff          jmp    80aeca5 <_Unwind_Find_FDE+0x35>
 80aef83:       2e 8d 74 26 00          lea    %cs:0x0(%esi,%eiz,1),%esi
 80aef88:       81 c4 bc 00 00 00       add    $0xbc,%esp
 80aef8e:       31 db                   xor    %ebx,%ebx
 80aef90:       89 d8                   mov    %ebx,%eax
 80aef92:       5b                      pop    %ebx
 80aef93:       5e                      pop    %esi
 80aef94:       5f                      pop    %edi
 80aef95:       5d                      pop    %ebp
 80aef96:       c3                      ret
 80aef97:       3c 03                   cmp    $0x3,%al
 80aef99:       0f 85 75 08 00 00       jne    80af814 <_Unwind_Find_FDE+0xba4>
 80aef9f:       8b 01                   mov    (%ecx),%eax
 80aefa1:       83 c1 04                add    $0x4,%ecx
 80aefa4:       89 44 24 3c             mov    %eax,0x3c(%esp)
 80aefa8:       85 c0                   test   %eax,%eax
 80aefaa:       74 dc                   je     80aef88 <_Unwind_Find_FDE+0x318>
 80aefac:       89 ca                   mov    %ecx,%edx
 80aefae:       83 e2 03                and    $0x3,%edx
 80aefb1:       89 54 24 08             mov    %edx,0x8(%esp)
 80aefb5:       0f 85 49 fd ff ff       jne    80aed04 <_Unwind_Find_FDE+0x94>
 80aefbb:       8b 11                   mov    (%ecx),%edx
 80aefbd:       89 5c 24 18             mov    %ebx,0x18(%esp)
 80aefc1:       01 da                   add    %ebx,%edx
 80aefc3:       39 d5                   cmp    %edx,%ebp
 80aefc5:       72 c1                   jb     80aef88 <_Unwind_Find_FDE+0x318>
 80aefc7:       8d 78 ff                lea    -0x1(%eax),%edi
 80aefca:       8d 04 f9                lea    (%ecx,%edi,8),%eax
 80aefcd:       8b 10                   mov    (%eax),%edx
 80aefcf:       89 44 24 10             mov    %eax,0x10(%esp)
 80aefd3:       01 da                   add    %ebx,%edx
 80aefd5:       39 d5                   cmp    %edx,%ebp
 80aefd7:       0f 83 b5 07 00 00       jae    80af792 <_Unwind_Find_FDE+0xb22>
 80aefdd:       85 ff                   test   %edi,%edi
 80aefdf:       0f 84 b4 08 00 00       je     80af899 <_Unwind_Find_FDE+0xc29>
 80aefe5:       89 74 24 1c             mov    %esi,0x1c(%esp)
 80aefe9:       eb 23                   jmp    80af00e <_Unwind_Find_FDE+0x39e>
 80aefeb:       8b 74 24 14             mov    0x14(%esp),%esi
 80aefef:       89 da                   mov    %ebx,%edx
 80aeff1:       83 c0 01                add    $0x1,%eax
 80aeff4:       03 54 31 08             add    0x8(%ecx,%esi,1),%edx
 80aeff8:       39 d5                   cmp    %edx,%ebp
 80aeffa:       0f 82 84 07 00 00       jb     80af784 <_Unwind_Find_FDE+0xb14>
 80af000:       89 44 24 08             mov    %eax,0x8(%esp)
 80af004:       39 7c 24 08             cmp    %edi,0x8(%esp)
 80af008:       0f 83 8b 08 00 00       jae    80af899 <_Unwind_Find_FDE+0xc29>
 80af00e:       8b 44 24 08             mov    0x8(%esp),%eax
 80af012:       01 f8                   add    %edi,%eax
 80af014:       d1 e8                   shr    $1,%eax
 80af016:       8d 34 c5 00 00 00 00    lea    0x0(,%eax,8),%esi
 80af01d:       89 74 24 14             mov    %esi,0x14(%esp)
 80af021:       01 ce                   add    %ecx,%esi
 80af023:       8b 16                   mov    (%esi),%edx
 80af025:       89 74 24 10             mov    %esi,0x10(%esp)
 80af029:       01 da                   add    %ebx,%edx
 80af02b:       39 d5                   cmp    %edx,%ebp
 80af02d:       73 bc                   jae    80aefeb <_Unwind_Find_FDE+0x37b>
 80af02f:       89 c7                   mov    %eax,%edi
 80af031:       eb d1                   jmp    80af004 <_Unwind_Find_FDE+0x394>
 80af033:       2e 8d 74 26 00          lea    %cs:0x0(%esi,%eiz,1),%esi
 80af038:       8b 44 24 08             mov    0x8(%esp),%eax
 80af03c:       8b 78 0c                mov    0xc(%eax),%edi
 80af03f:       f7 c6 02 00 00 00       test   $0x2,%esi
 80af045:       0f 84 e5 03 00 00       je     80af430 <_Unwind_Find_FDE+0x7c0>
 80af04b:       8b 17                   mov    (%edi),%edx
 80af04d:       8b 5c 24 08             mov    0x8(%esp),%ebx
 80af051:       85 d2                   test   %edx,%edx
 80af053:       75 19                   jne    80af06e <_Unwind_Find_FDE+0x3fe>
 80af055:       e9 20 ff ff ff          jmp    80aef7a <_Unwind_Find_FDE+0x30a>
 80af05a:       8d b6 00 00 00 00       lea    0x0(%esi),%esi
 80af060:       8b 57 04                mov    0x4(%edi),%edx
 80af063:       83 c7 04                add    $0x4,%edi
 80af066:       85 d2                   test   %edx,%edx
 80af068:       0f 84 0c ff ff ff       je     80aef7a <_Unwind_Find_FDE+0x30a>
 80af06e:       89 e9                   mov    %ebp,%ecx
 80af070:       89 d8                   mov    %ebx,%eax
 80af072:       e8 29 f5 ff ff          call   80ae5a0 <linear_search_fdes>
 80af077:       85 c0                   test   %eax,%eax
 80af079:       74 e5                   je     80af060 <_Unwind_Find_FDE+0x3f0>
 80af07b:       89 c3                   mov    %eax,%ebx
 80af07d:       8b 44 24 08             mov    0x8(%esp),%eax
 80af081:       8b bc 24 d4 00 00 00    mov    0xd4(%esp),%edi
 80af088:       83 e6 04                and    $0x4,%esi
 80af08b:       8b 68 04                mov    0x4(%eax),%ebp
 80af08e:       8b 50 08                mov    0x8(%eax),%edx
 80af091:       89 2f                   mov    %ebp,(%edi)
 80af093:       89 57 04                mov    %edx,0x4(%edi)
 80af096:       0f 85 c3 00 00 00       jne    80af15f <_Unwind_Find_FDE+0x4ef>
 80af09c:       0f b7 70 10             movzwl 0x10(%eax),%esi
 80af0a0:       8d 7b 08                lea    0x8(%ebx),%edi
 80af0a3:       89 f0                   mov    %esi,%eax
 80af0a5:       89 7c 24 10             mov    %edi,0x10(%esp)
 80af0a9:       66 c1 e8 03             shr    $0x3,%ax
 80af0ad:       89 c6                   mov    %eax,%esi
 80af0af:       0f b6 c0                movzbl %al,%eax
 80af0b2:       3d ff 00 00 00          cmp    $0xff,%eax
 80af0b7:       0f 84 9b 00 00 00       je     80af158 <_Unwind_Find_FDE+0x4e8>
 80af0bd:       89 f1                   mov    %esi,%ecx
 80af0bf:       83 e1 70                and    $0x70,%ecx
 80af0c2:       80 f9 20                cmp    $0x20,%cl
 80af0c5:       0f 84 cf 01 00 00       je     80af29a <_Unwind_Find_FDE+0x62a>
 80af0cb:       0f 86 87 00 00 00       jbe    80af158 <_Unwind_Find_FDE+0x4e8>
 80af0d1:       80 f9 30                cmp    $0x30,%cl
 80af0d4:       0f 85 c4 03 00 00       jne    80af49e <_Unwind_Find_FDE+0x82e>
 80af0da:       83 ec 0c                sub    $0xc,%esp
 80af0dd:       8d 4c 24 64             lea    0x64(%esp),%ecx
 80af0e1:       51                      push   %ecx
 80af0e2:       8b 4c 24 20             mov    0x20(%esp),%ecx
 80af0e6:       e8 85 d7 ff ff          call   80ac870 <read_encoded_value_with_base>
 80af0eb:       8b 44 24 68             mov    0x68(%esp),%eax
 80af0ef:       8b b4 24 e4 00 00 00    mov    0xe4(%esp),%esi
 80af0f6:       83 c4 10                add    $0x10,%esp
 80af0f9:       89 46 08                mov    %eax,0x8(%esi)
 80af0fc:       81 c4 bc 00 00 00       add    $0xbc,%esp
 80af102:       89 d8                   mov    %ebx,%eax
 80af104:       5b                      pop    %ebx
 80af105:       5e                      pop    %esi
 80af106:       5f                      pop    %edi
 80af107:       5d                      pop    %ebp
 80af108:       c3                      ret
 80af109:       8d b4 26 00 00 00 00    lea    0x0(%esi,%eiz,1),%esi
 80af110:       83 ec 0c                sub    $0xc,%esp
 80af113:       8b 5c 24 18             mov    0x18(%esp),%ebx
 80af117:       8d 83 0c 3c 00 00       lea    0x3c0c(%ebx),%eax
 80af11d:       89 44 24 1c             mov    %eax,0x1c(%esp)
 80af121:       50                      push   %eax
 80af122:       e8 59 a4 fc ff          call   8079580 <___pthread_mutex_lock>
 80af127:       8b 44 24 18             mov    0x18(%esp),%eax
 80af12b:       83 c4 10                add    $0x10,%esp
 80af12e:       0f b6 40 10             movzbl 0x10(%eax),%eax
 80af132:       a8 01                   test   $0x1,%al
 80af134:       0f 84 7a 01 00 00       je     80af2b4 <_Unwind_Find_FDE+0x644>
 80af13a:       83 ec 0c                sub    $0xc,%esp
 80af13d:       ff 74 24 1c             push   0x1c(%esp)
 80af141:       8b 5c 24 1c             mov    0x1c(%esp),%ebx
 80af145:       e8 16 ad fc ff          call   8079e60 <___pthread_mutex_unlock>
 80af14a:       83 c4 10                add    $0x10,%esp
 80af14d:       e9 bf fd ff ff          jmp    80aef11 <_Unwind_Find_FDE+0x2a1>
 80af152:       8d b6 00 00 00 00       lea    0x0(%esi),%esi
 80af158:       31 d2                   xor    %edx,%edx
 80af15a:       e9 7b ff ff ff          jmp    80af0da <_Unwind_Find_FDE+0x46a>
 80af15f:       8b 43 04                mov    0x4(%ebx),%eax
 80af162:       f7 d8                   neg    %eax
 80af164:       89 44 24 18             mov    %eax,0x18(%esp)
 80af168:       8d 43 04                lea    0x4(%ebx),%eax
 80af16b:       89 44 24 1c             mov    %eax,0x1c(%esp)
 80af16f:       8d 43 08                lea    0x8(%ebx),%eax
 80af172:       89 44 24 10             mov    %eax,0x10(%esp)
 80af176:       8b 74 24 18             mov    0x18(%esp),%esi
 80af17a:       8b 44 24 1c             mov    0x1c(%esp),%eax
 80af17e:       89 54 24 08             mov    %edx,0x8(%esp)
 80af182:       01 f0                   add    %esi,%eax
 80af184:       e8 57 e0 ff ff          call   80ad1e0 <get_cie_encoding>
 80af189:       8b 54 24 08             mov    0x8(%esp),%edx
 80af18d:       89 c6                   mov    %eax,%esi
 80af18f:       e9 1e ff ff ff          jmp    80af0b2 <_Unwind_Find_FDE+0x442>
 80af194:       8b 44 24 08             mov    0x8(%esp),%eax
 80af198:       66 c1 ee 03             shr    $0x3,%si
 80af19c:       89 f2                   mov    %esi,%edx
 80af19e:       8b 40 0c                mov    0xc(%eax),%eax
 80af1a1:       89 44 24 14             mov    %eax,0x14(%esp)
 80af1a5:       0f b6 c2                movzbl %dl,%eax
 80af1a8:       80 fa ff                cmp    $0xff,%dl
 80af1ab:       0f 84 9b 02 00 00       je     80af44c <_Unwind_Find_FDE+0x7dc>
 80af1b1:       89 f1                   mov    %esi,%ecx
 80af1b3:       83 e1 70                and    $0x70,%ecx
 80af1b6:       80 f9 20                cmp    $0x20,%cl
 80af1b9:       0f 84 f1 02 00 00       je     80af4b0 <_Unwind_Find_FDE+0x840>
 80af1bf:       0f 86 87 02 00 00       jbe    80af44c <_Unwind_Find_FDE+0x7dc>
 80af1c5:       80 f9 30                cmp    $0x30,%cl
 80af1c8:       0f 85 a3 04 00 00       jne    80af671 <_Unwind_Find_FDE+0xa01>
 80af1ce:       8b 5c 24 08             mov    0x8(%esp),%ebx
 80af1d2:       8b 5b 08                mov    0x8(%ebx),%ebx
 80af1d5:       89 5c 24 20             mov    %ebx,0x20(%esp)
 80af1d9:       8b 5c 24 14             mov    0x14(%esp),%ebx
 80af1dd:       8b 7b 04                mov    0x4(%ebx),%edi
 80af1e0:       85 ff                   test   %edi,%edi
 80af1e2:       0f 84 92 fd ff ff       je     80aef7a <_Unwind_Find_FDE+0x30a>
 80af1e8:       89 d3                   mov    %edx,%ebx
 80af1ea:       31 c9                   xor    %ecx,%ecx
 80af1ec:       8d 54 24 58             lea    0x58(%esp),%edx
 80af1f0:       89 44 24 1c             mov    %eax,0x1c(%esp)
 80af1f4:       83 e3 0f                and    $0xf,%ebx
 80af1f7:       89 ac 24 d0 00 00 00    mov    %ebp,0xd0(%esp)
 80af1fe:       89 cd                   mov    %ecx,%ebp
 80af200:       89 5c 24 24             mov    %ebx,0x24(%esp)
 80af204:       8d 5c 24 40             lea    0x40(%esp),%ebx
 80af208:       89 5c 24 28             mov    %ebx,0x28(%esp)
 80af20c:       89 f3                   mov    %esi,%ebx
 80af20e:       88 5c 24 2f             mov    %bl,0x2f(%esp)
 80af212:       89 54 24 18             mov    %edx,0x18(%esp)
 80af216:       eb 20                   jmp    80af238 <_Unwind_Find_FDE+0x5c8>
 80af218:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80af21f:       00
 80af220:       03 44 24 58             add    0x58(%esp),%eax
 80af224:       39 84 24 d0 00 00 00    cmp    %eax,0xd0(%esp)
 80af22b:       0f 82 28 02 00 00       jb     80af459 <_Unwind_Find_FDE+0x7e9>
 80af231:       8d 6e 01                lea    0x1(%esi),%ebp
 80af234:       39 fd                   cmp    %edi,%ebp
 80af236:       73 52                   jae    80af28a <_Unwind_Find_FDE+0x61a>
 80af238:       8b 44 24 14             mov    0x14(%esp),%eax
 80af23c:       8d 34 2f                lea    (%edi,%ebp,1),%esi
 80af23f:       83 ec 0c                sub    $0xc,%esp
 80af242:       d1 ee                   shr    $1,%esi
 80af244:       8b 5c b0 08             mov    0x8(%eax,%esi,4),%ebx
 80af248:       8d 43 08                lea    0x8(%ebx),%eax
 80af24b:       89 44 24 1c             mov    %eax,0x1c(%esp)
 80af24f:       89 c1                   mov    %eax,%ecx
 80af251:       ff 74 24 34             push   0x34(%esp)
 80af255:       8b 54 24 30             mov    0x30(%esp),%edx
 80af259:       8b 44 24 2c             mov    0x2c(%esp),%eax
 80af25d:       e8 0e d6 ff ff          call   80ac870 <read_encoded_value_with_base>
 80af262:       31 d2                   xor    %edx,%edx
 80af264:       89 c1                   mov    %eax,%ecx
 80af266:       58                      pop    %eax
 80af267:       ff 74 24 24             push   0x24(%esp)
 80af26b:       8b 44 24 34             mov    0x34(%esp),%eax
 80af26f:       e8 fc d5 ff ff          call   80ac870 <read_encoded_value_with_base>
 80af274:       8b 44 24 50             mov    0x50(%esp),%eax
 80af278:       83 c4 10                add    $0x10,%esp
 80af27b:       39 84 24 d0 00 00 00    cmp    %eax,0xd0(%esp)
 80af282:       73 9c                   jae    80af220 <_Unwind_Find_FDE+0x5b0>
 80af284:       89 f7                   mov    %esi,%edi
 80af286:       39 fd                   cmp    %edi,%ebp
 80af288:       72 ae                   jb     80af238 <_Unwind_Find_FDE+0x5c8>
 80af28a:       8b ac 24 d0 00 00 00    mov    0xd0(%esp),%ebp
 80af291:       8b 54 24 18             mov    0x18(%esp),%edx
 80af295:       e9 0b fa ff ff          jmp    80aeca5 <_Unwind_Find_FDE+0x35>
 80af29a:       89 ea                   mov    %ebp,%edx
 80af29c:       e9 39 fe ff ff          jmp    80af0da <_Unwind_Find_FDE+0x46a>
 80af2a1:       03 73 0c                add    0xc(%ebx),%esi
 80af2a4:       39 f5                   cmp    %esi,%ebp
 80af2a6:       0f 82 77 04 00 00       jb     80af723 <_Unwind_Find_FDE+0xab3>
 80af2ac:       8d 48 01                lea    0x1(%eax),%ecx
 80af2af:       e9 c2 fc ff ff          jmp    80aef76 <_Unwind_Find_FDE+0x306>
 80af2b4:       8b 7c 24 08             mov    0x8(%esp),%edi
 80af2b8:       8b 77 10                mov    0x10(%edi),%esi
 80af2bb:       89 74 24 14             mov    %esi,0x14(%esp)
 80af2bf:       c1 ee 0b                shr    $0xb,%esi
 80af2c2:       0f 85 3e 02 00 00       jne    80af506 <_Unwind_Find_FDE+0x896>
 80af2c8:       8b 5f 0c                mov    0xc(%edi),%ebx
 80af2cb:       a8 02                   test   $0x2,%al
 80af2cd:       0f 84 ed 01 00 00       je     80af4c0 <_Unwind_Find_FDE+0x850>
 80af2d3:       8b 13                   mov    (%ebx),%edx
 80af2d5:       85 d2                   test   %edx,%edx
 80af2d7:       75 17                   jne    80af2f0 <_Unwind_Find_FDE+0x680>
 80af2d9:       e9 5c fe ff ff          jmp    80af13a <_Unwind_Find_FDE+0x4ca>
 80af2de:       66 90                   xchg   %ax,%ax
 80af2e0:       8b 53 04                mov    0x4(%ebx),%edx
 80af2e3:       83 c3 04                add    $0x4,%ebx
 80af2e6:       01 c6                   add    %eax,%esi
 80af2e8:       85 d2                   test   %edx,%edx
 80af2ea:       0f 84 e8 01 00 00       je     80af4d8 <_Unwind_Find_FDE+0x868>
 80af2f0:       31 c9                   xor    %ecx,%ecx
 80af2f2:       89 f8                   mov    %edi,%eax
 80af2f4:       e8 e7 e9 ff ff          call   80adce0 <classify_object_over_fdes>
 80af2f9:       83 f8 ff                cmp    $0xffffffff,%eax
 80af2fc:       75 e2                   jne    80af2e0 <_Unwind_Find_FDE+0x670>
 80af2fe:       8b 44 24 0c             mov    0xc(%esp),%eax
 80af302:       8b 5c 24 08             mov    0x8(%esp),%ebx
 80af306:       8d 80 44 6d fe ff       lea    -0x192bc(%eax),%eax
 80af30c:       c7 43 10 f8 07 00 00    movl   $0x7f8,0x10(%ebx)
 80af313:       89 43 0c                mov    %eax,0xc(%ebx)
 80af316:       e9 1f fe ff ff          jmp    80af13a <_Unwind_Find_FDE+0x4ca>
 80af31b:       8b 40 0c                mov    0xc(%eax),%eax
 80af31e:       89 44 24 20             mov    %eax,0x20(%esp)
 80af322:       8b 40 04                mov    0x4(%eax),%eax
 80af325:       89 c3                   mov    %eax,%ebx
 80af327:       85 c0                   test   %eax,%eax
 80af329:       0f 84 4b fc ff ff       je     80aef7a <_Unwind_Find_FDE+0x30a>
 80af32f:       8d 44 24 40             lea    0x40(%esp),%eax
 80af333:       89 ac 24 d0 00 00 00    mov    %ebp,0xd0(%esp)
 80af33a:       89 dd                   mov    %ebx,%ebp
 80af33c:       c7 44 24 14 00 00 00    movl   $0x0,0x14(%esp)
 80af343:       00
 80af344:       89 44 24 24             mov    %eax,0x24(%esp)
 80af348:       eb 28                   jmp    80af372 <_Unwind_Find_FDE+0x702>
 80af34a:       8d b6 00 00 00 00       lea    0x0(%esi),%esi
 80af350:       03 44 24 58             add    0x58(%esp),%eax
 80af354:       39 84 24 d0 00 00 00    cmp    %eax,0xd0(%esp)
 80af35b:       0f 82 c3 02 00 00       jb     80af624 <_Unwind_Find_FDE+0x9b4>
 80af361:       8d 47 01                lea    0x1(%edi),%eax
 80af364:       89 44 24 14             mov    %eax,0x14(%esp)
 80af368:       39 6c 24 14             cmp    %ebp,0x14(%esp)
 80af36c:       0f 83 1e 01 00 00       jae    80af490 <_Unwind_Find_FDE+0x820>
 80af372:       8b 44 24 14             mov    0x14(%esp),%eax
 80af376:       8d 3c 28                lea    (%eax,%ebp,1),%edi
 80af379:       8b 44 24 20             mov    0x20(%esp),%eax
 80af37d:       d1 ef                   shr    $1,%edi
 80af37f:       8b 5c b8 08             mov    0x8(%eax,%edi,4),%ebx
 80af383:       8b 43 04                mov    0x4(%ebx),%eax
 80af386:       89 c6                   mov    %eax,%esi
 80af388:       f7 de                   neg    %esi
 80af38a:       89 74 24 18             mov    %esi,0x18(%esp)
 80af38e:       8d 73 04                lea    0x4(%ebx),%esi
 80af391:       89 f2                   mov    %esi,%edx
 80af393:       89 74 24 1c             mov    %esi,0x1c(%esp)
 80af397:       29 c2                   sub    %eax,%edx
 80af399:       89 d0                   mov    %edx,%eax
 80af39b:       e8 40 de ff ff          call   80ad1e0 <get_cie_encoding>
 80af3a0:       89 c6                   mov    %eax,%esi
 80af3a2:       8d 43 08                lea    0x8(%ebx),%eax
 80af3a5:       89 44 24 10             mov    %eax,0x10(%esp)
 80af3a9:       81 fe ff 00 00 00       cmp    $0xff,%esi
 80af3af:       74 5f                   je     80af410 <_Unwind_Find_FDE+0x7a0>
 80af3b1:       89 f0                   mov    %esi,%eax
 80af3b3:       83 e0 70                and    $0x70,%eax
 80af3b6:       3c 20                   cmp    $0x20,%al
 80af3b8:       74 5e                   je     80af418 <_Unwind_Find_FDE+0x7a8>
 80af3ba:       76 54                   jbe    80af410 <_Unwind_Find_FDE+0x7a0>
 80af3bc:       3c 30                   cmp    $0x30,%al
 80af3be:       75 61                   jne    80af421 <_Unwind_Find_FDE+0x7b1>
 80af3c0:       8b 44 24 08             mov    0x8(%esp),%eax
 80af3c4:       8b 50 08                mov    0x8(%eax),%edx
 80af3c7:       83 ec 0c                sub    $0xc,%esp
 80af3ca:       89 f0                   mov    %esi,%eax
 80af3cc:       ff 74 24 30             push   0x30(%esp)
 80af3d0:       8b 4c 24 20             mov    0x20(%esp),%ecx
 80af3d4:       e8 97 d4 ff ff          call   80ac870 <read_encoded_value_with_base>
 80af3d9:       5a                      pop    %edx
 80af3da:       31 d2                   xor    %edx,%edx
 80af3dc:       89 c1                   mov    %eax,%ecx
 80af3de:       89 f0                   mov    %esi,%eax
 80af3e0:       8d 74 24 64             lea    0x64(%esp),%esi
 80af3e4:       56                      push   %esi
 80af3e5:       83 e0 0f                and    $0xf,%eax
 80af3e8:       e8 83 d4 ff ff          call   80ac870 <read_encoded_value_with_base>
 80af3ed:       8b 44 24 50             mov    0x50(%esp),%eax
 80af3f1:       83 c4 10                add    $0x10,%esp
 80af3f4:       39 84 24 d0 00 00 00    cmp    %eax,0xd0(%esp)
 80af3fb:       0f 83 4f ff ff ff       jae    80af350 <_Unwind_Find_FDE+0x6e0>
 80af401:       89 fd                   mov    %edi,%ebp
 80af403:       e9 60 ff ff ff          jmp    80af368 <_Unwind_Find_FDE+0x6f8>
 80af408:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80af40f:       00
 80af410:       31 d2                   xor    %edx,%edx
 80af412:       eb b3                   jmp    80af3c7 <_Unwind_Find_FDE+0x757>
 80af414:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi
 80af418:       8b 44 24 08             mov    0x8(%esp),%eax
 80af41c:       8b 50 04                mov    0x4(%eax),%edx
 80af41f:       eb a6                   jmp    80af3c7 <_Unwind_Find_FDE+0x757>
 80af421:       3c 50                   cmp    $0x50,%al
 80af423:       74 eb                   je     80af410 <_Unwind_Find_FDE+0x7a0>
 80af425:       e9 2a a2 f9 ff          jmp    8049654 <_Unwind_Find_FDE.cold>
 80af42a:       8d b6 00 00 00 00       lea    0x0(%esi),%esi
 80af430:       8b 44 24 08             mov    0x8(%esp),%eax
 80af434:       89 e9                   mov    %ebp,%ecx
 80af436:       89 fa                   mov    %edi,%edx
 80af438:       e8 63 f1 ff ff          call   80ae5a0 <linear_search_fdes>
 80af43d:       89 c3                   mov    %eax,%ebx
 80af43f:       85 c0                   test   %eax,%eax
 80af441:       0f 85 36 fc ff ff       jne    80af07d <_Unwind_Find_FDE+0x40d>
 80af447:       e9 2e fb ff ff          jmp    80aef7a <_Unwind_Find_FDE+0x30a>
 80af44c:       c7 44 24 20 00 00 00    movl   $0x0,0x20(%esp)
 80af453:       00
 80af454:       e9 80 fd ff ff          jmp    80af1d9 <_Unwind_Find_FDE+0x569>
 80af459:       0f b6 74 24 2f          movzbl 0x2f(%esp),%esi
 80af45e:       8b 44 24 1c             mov    0x1c(%esp),%eax
 80af462:       8b ac 24 d0 00 00 00    mov    0xd0(%esp),%ebp
 80af469:       8b 54 24 18             mov    0x18(%esp),%edx
 80af46d:       85 db                   test   %ebx,%ebx
 80af46f:       0f 84 30 f8 ff ff       je     80aeca5 <_Unwind_Find_FDE+0x35>
 80af475:       8b 7c 24 08             mov    0x8(%esp),%edi
 80af479:       8b 8c 24 d4 00 00 00    mov    0xd4(%esp),%ecx
 80af480:       8b 6f 04                mov    0x4(%edi),%ebp
 80af483:       8b 57 08                mov    0x8(%edi),%edx
 80af486:       89 29                   mov    %ebp,(%ecx)
 80af488:       89 51 04                mov    %edx,0x4(%ecx)
 80af48b:       e9 22 fc ff ff          jmp    80af0b2 <_Unwind_Find_FDE+0x442>
 80af490:       8b ac 24 d0 00 00 00    mov    0xd0(%esp),%ebp
 80af497:       89 f2                   mov    %esi,%edx
 80af499:       e9 07 f8 ff ff          jmp    80aeca5 <_Unwind_Find_FDE+0x35>
 80af49e:       80 f9 50                cmp    $0x50,%cl
 80af4a1:       0f 84 b1 fc ff ff       je     80af158 <_Unwind_Find_FDE+0x4e8>
 80af4a7:       e9 a8 a1 f9 ff          jmp    8049654 <_Unwind_Find_FDE.cold>
 80af4ac:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi
 80af4b0:       8b 5c 24 08             mov    0x8(%esp),%ebx
 80af4b4:       8b 5b 04                mov    0x4(%ebx),%ebx
 80af4b7:       89 5c 24 20             mov    %ebx,0x20(%esp)
 80af4bb:       e9 19 fd ff ff          jmp    80af1d9 <_Unwind_Find_FDE+0x569>
 80af4c0:       8b 44 24 08             mov    0x8(%esp),%eax
 80af4c4:       31 c9                   xor    %ecx,%ecx
 80af4c6:       89 da                   mov    %ebx,%edx
 80af4c8:       e8 13 e8 ff ff          call   80adce0 <classify_object_over_fdes>
 80af4cd:       89 c6                   mov    %eax,%esi
 80af4cf:       83 f8 ff                cmp    $0xffffffff,%eax
 80af4d2:       0f 84 26 fe ff ff       je     80af2fe <_Unwind_Find_FDE+0x68e>
 80af4d8:       8b 5c 24 08             mov    0x8(%esp),%ebx
 80af4dc:       89 f2                   mov    %esi,%edx
 80af4de:       c1 e2 0b                shl    $0xb,%edx
 80af4e1:       8b 43 10                mov    0x10(%ebx),%eax
 80af4e4:       89 44 24 14             mov    %eax,0x14(%esp)
 80af4e8:       25 ff 07 00 00          and    $0x7ff,%eax
 80af4ed:       09 d0                   or     %edx,%eax
 80af4ef:       89 43 10                mov    %eax,0x10(%ebx)
 80af4f2:       81 fe ff ff 1f 00       cmp    $0x1fffff,%esi
 80af4f8:       0f 86 51 01 00 00       jbe    80af64f <_Unwind_Find_FDE+0x9df>
 80af4fe:       25 ff 07 00 00          and    $0x7ff,%eax
 80af503:       89 43 10                mov    %eax,0x10(%ebx)
 80af506:       83 ec 0c                sub    $0xc,%esp
 80af509:       8d 14 b5 08 00 00 00    lea    0x8(,%esi,4),%edx
 80af510:       52                      push   %edx
 80af511:       8b 5c 24 1c             mov    0x1c(%esp),%ebx
 80af515:       89 54 24 24             mov    %edx,0x24(%esp)
 80af519:       e8 f2 7d fa ff          call   8057310 <__libc_malloc>
 80af51e:       83 c4 10                add    $0x10,%esp
 80af521:       89 c7                   mov    %eax,%edi
 80af523:       85 c0                   test   %eax,%eax
 80af525:       0f 84 0f fc ff ff       je     80af13a <_Unwind_Find_FDE+0x4ca>
 80af52b:       83 ec 0c                sub    $0xc,%esp
 80af52e:       c7 40 04 00 00 00 00    movl   $0x0,0x4(%eax)
 80af535:       8b 54 24 20             mov    0x20(%esp),%edx
 80af539:       52                      push   %edx
 80af53a:       e8 d1 7d fa ff          call   8057310 <__libc_malloc>
 80af53f:       89 44 24 28             mov    %eax,0x28(%esp)
 80af543:       83 c4 10                add    $0x10,%esp
 80af546:       85 c0                   test   %eax,%eax
 80af548:       74 07                   je     80af551 <_Unwind_Find_FDE+0x8e1>
 80af54a:       c7 40 04 00 00 00 00    movl   $0x0,0x4(%eax)
 80af551:       8b 54 24 08             mov    0x8(%esp),%edx
 80af555:       0f b6 42 10             movzbl 0x10(%edx),%eax
 80af559:       8b 5a 0c                mov    0xc(%edx),%ebx
 80af55c:       88 44 24 14             mov    %al,0x14(%esp)
 80af560:       a8 02                   test   $0x2,%al
 80af562:       0f 84 f4 00 00 00       je     80af65c <_Unwind_Find_FDE+0x9ec>
 80af568:       8b 0b                   mov    (%ebx),%ecx
 80af56a:       85 c9                   test   %ecx,%ecx
 80af56c:       0f 84 27 03 00 00       je     80af899 <_Unwind_Find_FDE+0xc29>
 80af572:       89 74 24 1c             mov    %esi,0x1c(%esp)
 80af576:       89 d6                   mov    %edx,%esi
 80af578:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80af57f:       00
 80af580:       89 fa                   mov    %edi,%edx
 80af582:       89 f0                   mov    %esi,%eax
 80af584:       83 c3 04                add    $0x4,%ebx
 80af587:       e8 74 ed ff ff          call   80ae300 <add_fdes.isra.0>
 80af58c:       8b 0b                   mov    (%ebx),%ecx
 80af58e:       85 c9                   test   %ecx,%ecx
 80af590:       75 ee                   jne    80af580 <_Unwind_Find_FDE+0x910>
 80af592:       8b 74 24 1c             mov    0x1c(%esp),%esi
 80af596:       8b 5f 04                mov    0x4(%edi),%ebx
 80af599:       39 de                   cmp    %ebx,%esi
 80af59b:       0f 85 f8 02 00 00       jne    80af899 <_Unwind_Find_FDE+0xc29>
 80af5a1:       0f b6 44 24 14          movzbl 0x14(%esp),%eax
 80af5a6:       8b 4c 24 18             mov    0x18(%esp),%ecx
 80af5aa:       83 e0 04                and    $0x4,%eax
 80af5ad:       85 c9                   test   %ecx,%ecx
 80af5af:       0f 84 cb 00 00 00       je     80af680 <_Unwind_Find_FDE+0xa10>
 80af5b5:       8b 5c 24 0c             mov    0xc(%esp),%ebx
 80af5b9:       8d 93 3c 6c fc ff       lea    -0x393c4(%ebx),%edx
 80af5bf:       84 c0                   test   %al,%al
 80af5c1:       75 19                   jne    80af5dc <_Unwind_Find_FDE+0x96c>
 80af5c3:       8b 44 24 08             mov    0x8(%esp),%eax
 80af5c7:       8d 93 8c 70 fc ff       lea    -0x38f74(%ebx),%edx
 80af5cd:       66 f7 40 10 f8 07       testw  $0x7f8,0x10(%eax)
 80af5d3:       8d 83 ec 56 fc ff       lea    -0x3a914(%ebx),%eax
 80af5d9:       0f 44 d0                cmove  %eax,%edx
 80af5dc:       83 ec 0c                sub    $0xc,%esp
 80af5df:       89 f9                   mov    %edi,%ecx
 80af5e1:       8b 74 24 24             mov    0x24(%esp),%esi
 80af5e5:       56                      push   %esi
 80af5e6:       8b 44 24 18             mov    0x18(%esp),%eax
 80af5ea:       e8 01 d4 ff ff          call   80ac9f0 <fde_radixsort>
 80af5ef:       8b 5c 24 1c             mov    0x1c(%esp),%ebx
 80af5f3:       89 34 24                mov    %esi,(%esp)
 80af5f6:       e8 65 83 fa ff          call   8057960 <__free>
 80af5fb:       83 c4 10                add    $0x10,%esp
 80af5fe:       8b 74 24 08             mov    0x8(%esp),%esi
 80af602:       8b 46 0c                mov    0xc(%esi),%eax
 80af605:       89 07                   mov    %eax,(%edi)
 80af607:       8b 46 10                mov    0x10(%esi),%eax
 80af60a:       89 7e 0c                mov    %edi,0xc(%esi)
 80af60d:       89 44 24 68             mov    %eax,0x68(%esp)
 80af611:       83 c8 01                or     $0x1,%eax
 80af614:       88 44 24 68             mov    %al,0x68(%esp)
 80af618:       8b 44 24 68             mov    0x68(%esp),%eax
 80af61c:       89 46 10                mov    %eax,0x10(%esi)
 80af61f:       e9 16 fb ff ff          jmp    80af13a <_Unwind_Find_FDE+0x4ca>
 80af624:       8b 44 24 08             mov    0x8(%esp),%eax
 80af628:       8b b4 24 d4 00 00 00    mov    0xd4(%esp),%esi
 80af62f:       8b 68 04                mov    0x4(%eax),%ebp
 80af632:       8b 50 08                mov    0x8(%eax),%edx
 80af635:       89 2e                   mov    %ebp,(%esi)
 80af637:       89 56 04                mov    %edx,0x4(%esi)
 80af63a:       e9 37 fb ff ff          jmp    80af176 <_Unwind_Find_FDE+0x506>
 80af63f:       80 fa 50                cmp    $0x50,%dl
 80af642:       0f 85 0c a0 f9 ff       jne    8049654 <_Unwind_Find_FDE.cold>
 80af648:       31 ff                   xor    %edi,%edi
 80af64a:       e9 24 f7 ff ff          jmp    80aed73 <_Unwind_Find_FDE+0x103>
 80af64f:       85 f6                   test   %esi,%esi
 80af651:       0f 84 e3 fa ff ff       je     80af13a <_Unwind_Find_FDE+0x4ca>
 80af657:       e9 aa fe ff ff          jmp    80af506 <_Unwind_Find_FDE+0x896>
 80af65c:       8b 44 24 08             mov    0x8(%esp),%eax
 80af660:       89 d9                   mov    %ebx,%ecx
 80af662:       89 fa                   mov    %edi,%edx
 80af664:       e8 97 ec ff ff          call   80ae300 <add_fdes.isra.0>
 80af669:       8b 5f 04                mov    0x4(%edi),%ebx
 80af66c:       e9 28 ff ff ff          jmp    80af599 <_Unwind_Find_FDE+0x929>
 80af671:       80 f9 50                cmp    $0x50,%cl
 80af674:       0f 84 d2 fd ff ff       je     80af44c <_Unwind_Find_FDE+0x7dc>
 80af67a:       e9 d5 9f f9 ff          jmp    8049654 <_Unwind_Find_FDE.cold>
 80af67f:       90                      nop
 80af680:       8b 74 24 0c             mov    0xc(%esp),%esi
 80af684:       8d 8e fc 71 fc ff       lea    -0x38e04(%esi),%ecx
 80af68a:       89 4c 24 14             mov    %ecx,0x14(%esp)
 80af68e:       84 c0                   test   %al,%al
 80af690:       75 1d                   jne    80af6af <_Unwind_Find_FDE+0xa3f>
 80af692:       8b 44 24 08             mov    0x8(%esp),%eax
 80af696:       8d 96 cc 56 fc ff       lea    -0x3a934(%esi),%edx
 80af69c:       66 f7 40 10 f8 07       testw  $0x7f8,0x10(%eax)
 80af6a2:       8d 86 4c 71 fc ff       lea    -0x38eb4(%esi),%eax
 80af6a8:       0f 45 d0                cmovne %eax,%edx
 80af6ab:       89 54 24 14             mov    %edx,0x14(%esp)
 80af6af:       8d 47 08                lea    0x8(%edi),%eax
 80af6b2:       89 44 24 18             mov    %eax,0x18(%esp)
 80af6b6:       89 d8                   mov    %ebx,%eax
 80af6b8:       d1 e8                   shr    $1,%eax
 80af6ba:       8d 70 ff                lea    -0x1(%eax),%esi
 80af6bd:       0f 84 3b ff ff ff       je     80af5fe <_Unwind_Find_FDE+0x98e>
 80af6c3:       83 ec 08                sub    $0x8,%esp
 80af6c6:       53                      push   %ebx
 80af6c7:       56                      push   %esi
 80af6c8:       8b 4c 24 28             mov    0x28(%esp),%ecx
 80af6cc:       83 ee 01                sub    $0x1,%esi
 80af6cf:       8b 54 24 24             mov    0x24(%esp),%edx
 80af6d3:       8b 44 24 18             mov    0x18(%esp),%eax
 80af6d7:       e8 44 d0 ff ff          call   80ac720 <frame_downheap>
 80af6dc:       83 c4 10                add    $0x10,%esp
 80af6df:       83 fe ff                cmp    $0xffffffff,%esi
 80af6e2:       75 df                   jne    80af6c3 <_Unwind_Find_FDE+0xa53>
 80af6e4:       83 eb 01                sub    $0x1,%ebx
 80af6e7:       85 db                   test   %ebx,%ebx
 80af6e9:       0f 8e 0f ff ff ff       jle    80af5fe <_Unwind_Find_FDE+0x98e>
 80af6ef:       8b 74 24 18             mov    0x18(%esp),%esi
 80af6f3:       8b 47 08                mov    0x8(%edi),%eax
 80af6f6:       8b 54 9f 08             mov    0x8(%edi,%ebx,4),%edx
 80af6fa:       83 ec 08                sub    $0x8,%esp
 80af6fd:       89 f1                   mov    %esi,%ecx
 80af6ff:       89 57 08                mov    %edx,0x8(%edi)
 80af702:       89 44 9f 08             mov    %eax,0x8(%edi,%ebx,4)
 80af706:       53                      push   %ebx
 80af707:       6a 00                   push   $0x0
 80af709:       8b 54 24 24             mov    0x24(%esp),%edx
 80af70d:       8b 44 24 18             mov    0x18(%esp),%eax
 80af711:       e8 0a d0 ff ff          call   80ac720 <frame_downheap>
 80af716:       83 c4 10                add    $0x10,%esp
 80af719:       83 eb 01                sub    $0x1,%ebx
 80af71c:       75 d5                   jne    80af6f3 <_Unwind_Find_FDE+0xa83>
 80af71e:       e9 db fe ff ff          jmp    80af5fe <_Unwind_Find_FDE+0x98e>
 80af723:       8b 44 24 08             mov    0x8(%esp),%eax
 80af727:       8b bc 24 d4 00 00 00    mov    0xd4(%esp),%edi
 80af72e:       0f b7 74 24 10          movzwl 0x10(%esp),%esi
 80af733:       8b 68 04                mov    0x4(%eax),%ebp
 80af736:       8b 50 08                mov    0x8(%eax),%edx
 80af739:       89 2f                   mov    %ebp,(%edi)
 80af73b:       89 57 04                mov    %edx,0x4(%edi)
 80af73e:       e9 5d f9 ff ff          jmp    80af0a0 <_Unwind_Find_FDE+0x430>
 80af743:       0f b6 d0                movzbl %al,%edx
 80af746:       89 54 24 08             mov    %edx,0x8(%esp)
 80af74a:       3c ff                   cmp    $0xff,%al
 80af74c:       74 32                   je     80af780 <_Unwind_Find_FDE+0xb10>
 80af74e:       83 e0 70                and    $0x70,%eax
 80af751:       3c 20                   cmp    $0x20,%al
 80af753:       74 2b                   je     80af780 <_Unwind_Find_FDE+0xb10>
 80af755:       76 29                   jbe    80af780 <_Unwind_Find_FDE+0xb10>
 80af757:       89 f2                   mov    %esi,%edx
 80af759:       3c 30                   cmp    $0x30,%al
 80af75b:       75 1b                   jne    80af778 <_Unwind_Find_FDE+0xb08>
 80af75d:       83 ec 0c                sub    $0xc,%esp
 80af760:       8d 44 24 44             lea    0x44(%esp),%eax
 80af764:       50                      push   %eax
 80af765:       8b 44 24 18             mov    0x18(%esp),%eax
 80af769:       e8 02 d1 ff ff          call   80ac870 <read_encoded_value_with_base>
 80af76e:       83 c4 10                add    $0x10,%esp
 80af771:       89 c1                   mov    %eax,%ecx
 80af773:       e9 7a f5 ff ff          jmp    80aecf2 <_Unwind_Find_FDE+0x82>
 80af778:       3c 50                   cmp    $0x50,%al
 80af77a:       0f 85 d4 9e f9 ff       jne    8049654 <_Unwind_Find_FDE.cold>
 80af780:       31 d2                   xor    %edx,%edx
 80af782:       eb d9                   jmp    80af75d <_Unwind_Find_FDE+0xaed>
 80af784:       8b 74 24 1c             mov    0x1c(%esp),%esi
 80af788:       39 7c 24 08             cmp    %edi,0x8(%esp)
 80af78c:       0f 83 07 01 00 00       jae    80af899 <_Unwind_Find_FDE+0xc29>
 80af792:       8b 44 24 10             mov    0x10(%esp),%eax
 80af796:       03 58 04                add    0x4(%eax),%ebx
 80af799:       8d 43 04                lea    0x4(%ebx),%eax
 80af79c:       2b 43 04                sub    0x4(%ebx),%eax
 80af79f:       e8 3c da ff ff          call   80ad1e0 <get_cie_encoding>
 80af7a4:       3d ff 00 00 00          cmp    $0xff,%eax
 80af7a9:       0f 84 ad 00 00 00       je     80af85c <_Unwind_Find_FDE+0xbec>
 80af7af:       89 c1                   mov    %eax,%ecx
 80af7b1:       83 e1 07                and    $0x7,%ecx
 80af7b4:       80 f9 02                cmp    $0x2,%cl
 80af7b7:       0f 84 8c 00 00 00       je     80af849 <_Unwind_Find_FDE+0xbd9>
 80af7bd:       0f 86 c4 00 00 00       jbe    80af887 <_Unwind_Find_FDE+0xc17>
 80af7c3:       80 f9 03                cmp    $0x3,%cl
 80af7c6:       0f 84 b1 00 00 00       je     80af87d <_Unwind_Find_FDE+0xc0d>
 80af7cc:       80 f9 04                cmp    $0x4,%cl
 80af7cf:       0f 85 7f 9e f9 ff       jne    8049654 <_Unwind_Find_FDE.cold>
 80af7d5:       ba 10 00 00 00          mov    $0x10,%edx
 80af7da:       8d 0c 13                lea    (%ebx,%edx,1),%ecx
 80af7dd:       83 f8 1b                cmp    $0x1b,%eax
 80af7e0:       75 7d                   jne    80af85f <_Unwind_Find_FDE+0xbef>
 80af7e2:       8b 01                   mov    (%ecx),%eax
 80af7e4:       89 44 24 40             mov    %eax,0x40(%esp)
 80af7e8:       8b 7c 24 10             mov    0x10(%esp),%edi
 80af7ec:       8b 54 24 18             mov    0x18(%esp),%edx
 80af7f0:       03 17                   add    (%edi),%edx
 80af7f2:       01 d0                   add    %edx,%eax
 80af7f4:       39 c5                   cmp    %eax,%ebp
 80af7f6:       0f 83 8c f7 ff ff       jae    80aef88 <_Unwind_Find_FDE+0x318>
 80af7fc:       8b 84 24 d4 00 00 00    mov    0xd4(%esp),%eax
 80af803:       c7 00 00 00 00 00       movl   $0x0,(%eax)
 80af809:       89 70 04                mov    %esi,0x4(%eax)
 80af80c:       89 50 08                mov    %edx,0x8(%eax)
 80af80f:       e9 8f f5 ff ff          jmp    80aeda3 <_Unwind_Find_FDE+0x133>
 80af814:       0f b6 d0                movzbl %al,%edx
 80af817:       83 e0 70                and    $0x70,%eax
 80af81a:       89 54 24 08             mov    %edx,0x8(%esp)
 80af81e:       3c 20                   cmp    $0x20,%al
 80af820:       74 36                   je     80af858 <_Unwind_Find_FDE+0xbe8>
 80af822:       76 34                   jbe    80af858 <_Unwind_Find_FDE+0xbe8>
 80af824:       89 f2                   mov    %esi,%edx
 80af826:       3c 30                   cmp    $0x30,%al
 80af828:       75 26                   jne    80af850 <_Unwind_Find_FDE+0xbe0>
 80af82a:       83 ec 0c                sub    $0xc,%esp
 80af82d:       8d 44 24 48             lea    0x48(%esp),%eax
 80af831:       50                      push   %eax
 80af832:       8b 44 24 18             mov    0x18(%esp),%eax
 80af836:       e8 35 d0 ff ff          call   80ac870 <read_encoded_value_with_base>
 80af83b:       89 c1                   mov    %eax,%ecx
 80af83d:       8b 44 24 4c             mov    0x4c(%esp),%eax
 80af841:       83 c4 10                add    $0x10,%esp
 80af844:       e9 5f f7 ff ff          jmp    80aefa8 <_Unwind_Find_FDE+0x338>
 80af849:       ba 0a 00 00 00          mov    $0xa,%edx
 80af84e:       eb 8a                   jmp    80af7da <_Unwind_Find_FDE+0xb6a>
 80af850:       3c 50                   cmp    $0x50,%al
 80af852:       0f 85 fc 9d f9 ff       jne    8049654 <_Unwind_Find_FDE.cold>
 80af858:       31 d2                   xor    %edx,%edx
 80af85a:       eb ce                   jmp    80af82a <_Unwind_Find_FDE+0xbba>
 80af85c:       8d 4b 08                lea    0x8(%ebx),%ecx
 80af85f:       83 ec 0c                sub    $0xc,%esp
 80af862:       83 e0 0f                and    $0xf,%eax
 80af865:       8d 54 24 4c             lea    0x4c(%esp),%edx
 80af869:       52                      push   %edx
 80af86a:       31 d2                   xor    %edx,%edx
 80af86c:       e8 ff cf ff ff          call   80ac870 <read_encoded_value_with_base>
 80af871:       8b 44 24 50             mov    0x50(%esp),%eax
 80af875:       83 c4 10                add    $0x10,%esp
 80af878:       e9 6b ff ff ff          jmp    80af7e8 <_Unwind_Find_FDE+0xb78>
 80af87d:       ba 0c 00 00 00          mov    $0xc,%edx
 80af882:       e9 53 ff ff ff          jmp    80af7da <_Unwind_Find_FDE+0xb6a>
 80af887:       ba 0c 00 00 00          mov    $0xc,%edx
 80af88c:       84 c9                   test   %cl,%cl
 80af88e:       0f 84 46 ff ff ff       je     80af7da <_Unwind_Find_FDE+0xb6a>
 80af894:       e9 bb 9d f9 ff          jmp    8049654 <_Unwind_Find_FDE.cold>
 80af899:       8b 5c 24 0c             mov    0xc(%esp),%ebx
 80af89d:       e8 ec 98 f9 ff          call   804918e <abort>
 80af8a2:       66 90                   xchg   %ax,%ax
 80af8a4:       66 90                   xchg   %ax,%ax
 80af8a6:       66 90                   xchg   %ax,%ax
 80af8a8:       66 90                   xchg   %ax,%ax
 80af8aa:       66 90                   xchg   %ax,%ax
 80af8ac:       66 90                   xchg   %ax,%ax
 80af8ae:       66 90                   xchg   %ax,%ax
 80af8b0:       66 90                   xchg   %ax,%ax
 80af8b2:       66 90                   xchg   %ax,%ax
 80af8b4:       66 90                   xchg   %ax,%ax
 80af8b6:       66 90                   xchg   %ax,%ax
 80af8b8:       66 90                   xchg   %ax,%ax
 80af8ba:       66 90                   xchg   %ax,%ax
 80af8bc:       66 90                   xchg   %ax,%ax
 80af8be:       66 90                   xchg   %ax,%ax

080af8c0 <base_of_encoded_value>:
 80af8c0:       53                      push   %ebx
 80af8c1:       e8 1a 9f f9 ff          call   80497e0 <__x86.get_pc_thunk.bx>
 80af8c6:       81 c3 2e 77 03 00       add    $0x3772e,%ebx
 80af8cc:       83 ec 08                sub    $0x8,%esp
 80af8cf:       3c ff                   cmp    $0xff,%al
 80af8d1:       74 43                   je     80af916 <base_of_encoded_value+0x56>
 80af8d3:       83 e0 70                and    $0x70,%eax
 80af8d6:       3c 30                   cmp    $0x30,%al
 80af8d8:       74 56                   je     80af930 <base_of_encoded_value+0x70>
 80af8da:       77 1c                   ja     80af8f8 <base_of_encoded_value+0x38>
 80af8dc:       3c 20                   cmp    $0x20,%al
 80af8de:       75 30                   jne    80af910 <base_of_encoded_value+0x50>
 80af8e0:       83 ec 0c                sub    $0xc,%esp
 80af8e3:       52                      push   %edx
 80af8e4:       e8 87 c5 ff ff          call   80abe70 <_Unwind_GetTextRelBase>
 80af8e9:       83 c4 10                add    $0x10,%esp
 80af8ec:       83 c4 08                add    $0x8,%esp
 80af8ef:       5b                      pop    %ebx
 80af8f0:       c3                      ret
 80af8f1:       8d b4 26 00 00 00 00    lea    0x0(%esi,%eiz,1),%esi
 80af8f8:       3c 40                   cmp    $0x40,%al
 80af8fa:       75 24                   jne    80af920 <base_of_encoded_value+0x60>
 80af8fc:       83 ec 0c                sub    $0xc,%esp
 80af8ff:       52                      push   %edx
 80af900:       e8 0b c5 ff ff          call   80abe10 <_Unwind_GetRegionStart>
 80af905:       83 c4 10                add    $0x10,%esp
 80af908:       83 c4 08                add    $0x8,%esp
 80af90b:       5b                      pop    %ebx
 80af90c:       c3                      ret
 80af90d:       8d 76 00                lea    0x0(%esi),%esi
 80af910:       0f 87 47 9d f9 ff       ja     804965d <base_of_encoded_value.cold>
 80af916:       83 c4 08                add    $0x8,%esp
 80af919:       31 c0                   xor    %eax,%eax
 80af91b:       5b                      pop    %ebx
 80af91c:       c3                      ret
 80af91d:       8d 76 00                lea    0x0(%esi),%esi
 80af920:       3c 50                   cmp    $0x50,%al
 80af922:       74 f2                   je     80af916 <base_of_encoded_value+0x56>
 80af924:       e9 34 9d f9 ff          jmp    804965d <base_of_encoded_value.cold>
 80af929:       8d b4 26 00 00 00 00    lea    0x0(%esi,%eiz,1),%esi
 80af930:       83 ec 0c                sub    $0xc,%esp
 80af933:       52                      push   %edx
 80af934:       e8 27 c5 ff ff          call   80abe60 <_Unwind_GetDataRelBase>
 80af939:       83 c4 10                add    $0x10,%esp
 80af93c:       83 c4 08                add    $0x8,%esp
 80af93f:       5b                      pop    %ebx
 80af940:       c3                      ret
 80af941:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80af948:       00
 80af949:       8d b4 26 00 00 00 00    lea    0x0(%esi,%eiz,1),%esi

080af950 <read_encoded_value_with_base>:
 80af950:       55                      push   %ebp
 80af951:       e8 45 ff f9 ff          call   804f89b <__x86.get_pc_thunk.bp>
 80af956:       81 c5 9e 76 03 00       add    $0x3769e,%ebp
 80af95c:       57                      push   %edi
 80af95d:       56                      push   %esi
 80af95e:       89 ce                   mov    %ecx,%esi
 80af960:       53                      push   %ebx
 80af961:       83 ec 1c                sub    $0x1c,%esp
 80af964:       3c 50                   cmp    $0x50,%al
 80af966:       74 58                   je     80af9c0 <.L20+0x30>
 80af968:       89 c3                   mov    %eax,%ebx
 80af96a:       83 e0 0f                and    $0xf,%eax
 80af96d:       3c 0c                   cmp    $0xc,%al
 80af96f:       0f 87 ed 9c f9 ff       ja     8049662 <read_encoded_value_with_base.cold>
 80af975:       0f b6 c0                movzbl %al,%eax
 80af978:       89 d7                   mov    %edx,%edi
 80af97a:       8b 8c 85 4c 6d fe ff    mov    -0x192b4(%ebp,%eax,4),%ecx
 80af981:       01 e9                   add    %ebp,%ecx
 80af983:       3e ff e1                notrack jmp *%ecx
 80af986:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80af98d:       00
 80af98e:       66 90                   xchg   %ax,%ax

080af990 <.L20>:
 80af990:       8b 16                   mov    (%esi),%edx
 80af992:       8d 46 04                lea    0x4(%esi),%eax
 80af995:       85 d2                   test   %edx,%edx
 80af997:       74 11                   je     80af9aa <.L20+0x1a>
 80af999:       89 d9                   mov    %ebx,%ecx
 80af99b:       83 e1 70                and    $0x70,%ecx
 80af99e:       80 f9 10                cmp    $0x10,%cl
 80af9a1:       0f 44 fe                cmove  %esi,%edi
 80af9a4:       01 fa                   add    %edi,%edx
 80af9a6:       84 db                   test   %bl,%bl
 80af9a8:       78 36                   js     80af9e0 <.L20+0x50>
 80af9aa:       8b 4c 24 30             mov    0x30(%esp),%ecx
 80af9ae:       89 11                   mov    %edx,(%ecx)
 80af9b0:       83 c4 1c                add    $0x1c,%esp
 80af9b3:       5b                      pop    %ebx
 80af9b4:       5e                      pop    %esi
 80af9b5:       5f                      pop    %edi
 80af9b6:       5d                      pop    %ebp
 80af9b7:       c3                      ret
 80af9b8:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80af9bf:       00
 80af9c0:       8d 41 03                lea    0x3(%ecx),%eax
 80af9c3:       8b 4c 24 30             mov    0x30(%esp),%ecx
 80af9c7:       83 e0 fc                and    $0xfffffffc,%eax
 80af9ca:       8b 10                   mov    (%eax),%edx
 80af9cc:       83 c0 04                add    $0x4,%eax
 80af9cf:       89 11                   mov    %edx,(%ecx)
 80af9d1:       83 c4 1c                add    $0x1c,%esp
 80af9d4:       5b                      pop    %ebx
 80af9d5:       5e                      pop    %esi
 80af9d6:       5f                      pop    %edi
 80af9d7:       5d                      pop    %ebp
 80af9d8:       c3                      ret
 80af9d9:       8d b4 26 00 00 00 00    lea    0x0(%esi,%eiz,1),%esi
 80af9e0:       8b 12                   mov    (%edx),%edx
 80af9e2:       8b 4c 24 30             mov    0x30(%esp),%ecx
 80af9e6:       89 11                   mov    %edx,(%ecx)
 80af9e8:       83 c4 1c                add    $0x1c,%esp
 80af9eb:       5b                      pop    %ebx
 80af9ec:       5e                      pop    %esi
 80af9ed:       5f                      pop    %edi
 80af9ee:       5d                      pop    %ebp
 80af9ef:       c3                      ret

080af9f0 <.L18>:
 80af9f0:       8b 16                   mov    (%esi),%edx
 80af9f2:       8d 46 08                lea    0x8(%esi),%eax
 80af9f5:       eb 9e                   jmp    80af995 <.L20+0x5>
 80af9f7:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80af9fe:       00
 80af9ff:       90                      nop

080afa00 <.L25>:
 80afa00:       0f b7 16                movzwl (%esi),%edx
 80afa03:       8d 46 02                lea    0x2(%esi),%eax
 80afa06:       eb 8d                   jmp    80af995 <.L20+0x5>
 80afa08:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80afa0f:       00

080afa10 <.L21>:
 80afa10:       0f bf 16                movswl (%esi),%edx
 80afa13:       8d 46 02                lea    0x2(%esi),%eax
 80afa16:       e9 7a ff ff ff          jmp    80af995 <.L20+0x5>
 80afa1b:       2e 8d 74 26 00          lea    %cs:0x0(%esi,%eiz,1),%esi

080afa20 <.L31>:
 80afa20:       89 5c 24 08             mov    %ebx,0x8(%esp)
 80afa24:       31 d2                   xor    %edx,%edx
 80afa26:       89 f0                   mov    %esi,%eax
 80afa28:       31 c9                   xor    %ecx,%ecx
 80afa2a:       89 d5                   mov    %edx,%ebp
 80afa2c:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80afa33:       00
 80afa34:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80afa3b:       00
 80afa3c:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi
 80afa40:       0f b6 10                movzbl (%eax),%edx
 80afa43:       83 c0 01                add    $0x1,%eax
 80afa46:       89 d3                   mov    %edx,%ebx
 80afa48:       83 e3 7f                and    $0x7f,%ebx
 80afa4b:       d3 e3                   shl    %cl,%ebx
 80afa4d:       83 c1 07                add    $0x7,%ecx
 80afa50:       09 dd                   or     %ebx,%ebp
 80afa52:       84 d2                   test   %dl,%dl
 80afa54:       78 ea                   js     80afa40 <.L31+0x20>
 80afa56:       8b 5c 24 08             mov    0x8(%esp),%ebx
 80afa5a:       89 ea                   mov    %ebp,%edx
 80afa5c:       e9 34 ff ff ff          jmp    80af995 <.L20+0x5>
 80afa61:       8d b4 26 00 00 00 00    lea    0x0(%esi,%eiz,1),%esi

080afa68 <.L32>:
 80afa68:       89 5c 24 0c             mov    %ebx,0xc(%esp)
 80afa6c:       31 d2                   xor    %edx,%edx
 80afa6e:       89 f0                   mov    %esi,%eax
 80afa70:       31 c9                   xor    %ecx,%ecx
 80afa72:       89 d5                   mov    %edx,%ebp
 80afa74:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80afa7b:       00
 80afa7c:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi
 80afa80:       0f b6 10                movzbl (%eax),%edx
 80afa83:       83 c0 01                add    $0x1,%eax
 80afa86:       89 d3                   mov    %edx,%ebx
 80afa88:       83 e3 7f                and    $0x7f,%ebx
 80afa8b:       d3 e3                   shl    %cl,%ebx
 80afa8d:       83 c1 07                add    $0x7,%ecx
 80afa90:       09 dd                   or     %ebx,%ebp
 80afa92:       84 d2                   test   %dl,%dl
 80afa94:       78 ea                   js     80afa80 <.L32+0x18>
 80afa96:       88 54 24 08             mov    %dl,0x8(%esp)
 80afa9a:       8b 5c 24 0c             mov    0xc(%esp),%ebx
 80afa9e:       89 ea                   mov    %ebp,%edx
 80afaa0:       0f b6 6c 24 08          movzbl 0x8(%esp),%ebp
 80afaa5:       83 f9 1f                cmp    $0x1f,%ecx
 80afaa8:       0f 87 e7 fe ff ff       ja     80af995 <.L20+0x5>
 80afaae:       83 e5 40                and    $0x40,%ebp
 80afab1:       0f 84 de fe ff ff       je     80af995 <.L20+0x5>
 80afab7:       bd ff ff ff ff          mov    $0xffffffff,%ebp
 80afabc:       d3 e5                   shl    %cl,%ebp
 80afabe:       09 ea                   or     %ebp,%edx
 80afac0:       e9 d4 fe ff ff          jmp    80af999 <.L20+0x9>
 80afac5:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80afacc:       00
 80afacd:       8d 76 00                lea    0x0(%esi),%esi

080afad0 <__gcc_personality_v0>:
 80afad0:       f3 0f 1e fb             endbr32
 80afad4:       55                      push   %ebp
 80afad5:       57                      push   %edi
 80afad6:       56                      push   %esi
 80afad7:       53                      push   %ebx
 80afad8:       e8 03 9d f9 ff          call   80497e0 <__x86.get_pc_thunk.bx>
 80afadd:       81 c3 17 75 03 00       add    $0x37517,%ebx
 80afae3:       83 ec 5c                sub    $0x5c,%esp
 80afae6:       83 7c 24 70 01          cmpl   $0x1,0x70(%esp)
 80afaeb:       c7 44 24 28 00 00 00    movl   $0x0,0x28(%esp)
 80afaf2:       00
 80afaf3:       89 5c 24 18             mov    %ebx,0x18(%esp)
 80afaf7:       75 17                   jne    80afb10 <__gcc_personality_v0+0x40>
 80afaf9:       f6 44 24 74 02          testb  $0x2,0x74(%esp)
 80afafe:       75 20                   jne    80afb20 <__gcc_personality_v0+0x50>
 80afb00:       83 c4 5c                add    $0x5c,%esp
 80afb03:       b8 08 00 00 00          mov    $0x8,%eax
 80afb08:       5b                      pop    %ebx
 80afb09:       5e                      pop    %esi
 80afb0a:       5f                      pop    %edi
 80afb0b:       5d                      pop    %ebp
 80afb0c:       c3                      ret
 80afb0d:       8d 76 00                lea    0x0(%esi),%esi
 80afb10:       b8 03 00 00 00          mov    $0x3,%eax
 80afb15:       83 c4 5c                add    $0x5c,%esp
 80afb18:       5b                      pop    %ebx
 80afb19:       5e                      pop    %esi
 80afb1a:       5f                      pop    %edi
 80afb1b:       5d                      pop    %ebp
 80afb1c:       c3                      ret
 80afb1d:       8d 76 00                lea    0x0(%esi),%esi
 80afb20:       83 ec 0c                sub    $0xc,%esp
 80afb23:       ff b4 24 90 00 00 00    push   0x90(%esp)
 80afb2a:       e8 d1 c2 ff ff          call   80abe00 <_Unwind_GetLanguageSpecificData>
 80afb2f:       83 c4 10                add    $0x10,%esp
 80afb32:       89 c6                   mov    %eax,%esi
 80afb34:       85 c0                   test   %eax,%eax
 80afb36:       74 c8                   je     80afb00 <__gcc_personality_v0+0x30>
 80afb38:       8b 8c 24 84 00 00 00    mov    0x84(%esp),%ecx
 80afb3f:       31 ff                   xor    %edi,%edi
 80afb41:       85 c9                   test   %ecx,%ecx
 80afb43:       74 14                   je     80afb59 <__gcc_personality_v0+0x89>
 80afb45:       83 ec 0c                sub    $0xc,%esp
 80afb48:       ff b4 24 90 00 00 00    push   0x90(%esp)
 80afb4f:       e8 bc c2 ff ff          call   80abe10 <_Unwind_GetRegionStart>
 80afb54:       83 c4 10                add    $0x10,%esp
 80afb57:       89 c7                   mov    %eax,%edi
 80afb59:       0f b6 06                movzbl (%esi),%eax
 80afb5c:       89 7c 24 38             mov    %edi,0x38(%esp)
 80afb60:       8d 6e 01                lea    0x1(%esi),%ebp
 80afb63:       89 7c 24 1c             mov    %edi,0x1c(%esp)
 80afb67:       3c ff                   cmp    $0xff,%al
 80afb69:       74 52                   je     80afbbd <__gcc_personality_v0+0xed>
 80afb6b:       0f b6 f0                movzbl %al,%esi
 80afb6e:       83 e0 70                and    $0x70,%eax
 80afb71:       3c 30                   cmp    $0x30,%al
 80afb73:       0f 84 3b 02 00 00       je     80afdb4 <__gcc_personality_v0+0x2e4>
 80afb79:       0f 87 14 02 00 00       ja     80afd93 <__gcc_personality_v0+0x2c3>
 80afb7f:       3c 20                   cmp    $0x20,%al
 80afb81:       0f 85 4a 02 00 00       jne    80afdd1 <__gcc_personality_v0+0x301>
 80afb87:       83 ec 0c                sub    $0xc,%esp
 80afb8a:       ff b4 24 90 00 00 00    push   0x90(%esp)
 80afb91:       8b 5c 24 28             mov    0x28(%esp),%ebx
 80afb95:       e8 d6 c2 ff ff          call   80abe70 <_Unwind_GetTextRelBase>
 80afb9a:       83 c4 10                add    $0x10,%esp
 80afb9d:       89 c2                   mov    %eax,%edx
 80afb9f:       83 ec 0c                sub    $0xc,%esp
 80afba2:       89 e9                   mov    %ebp,%ecx
 80afba4:       8d 44 24 48             lea    0x48(%esp),%eax
 80afba8:       50                      push   %eax
 80afba9:       89 f0                   mov    %esi,%eax
 80afbab:       e8 a0 fd ff ff          call   80af950 <read_encoded_value_with_base>
 80afbb0:       89 c5                   mov    %eax,%ebp
 80afbb2:       8b 44 24 4c             mov    0x4c(%esp),%eax
 80afbb6:       89 44 24 2c             mov    %eax,0x2c(%esp)
 80afbba:       83 c4 10                add    $0x10,%esp
 80afbbd:       0f b6 45 00             movzbl 0x0(%ebp),%eax
 80afbc1:       8d 55 01                lea    0x1(%ebp),%edx
 80afbc4:       31 f6                   xor    %esi,%esi
 80afbc6:       88 44 24 4c             mov    %al,0x4c(%esp)
 80afbca:       3c ff                   cmp    $0xff,%al
 80afbcc:       74 2a                   je     80afbf8 <__gcc_personality_v0+0x128>
 80afbce:       31 c9                   xor    %ecx,%ecx
 80afbd0:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80afbd7:       00
 80afbd8:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80afbdf:       00
 80afbe0:       0f b6 1a                movzbl (%edx),%ebx
 80afbe3:       83 c2 01                add    $0x1,%edx
 80afbe6:       89 d8                   mov    %ebx,%eax
 80afbe8:       83 e0 7f                and    $0x7f,%eax
 80afbeb:       d3 e0                   shl    %cl,%eax
 80afbed:       83 c1 07                add    $0x7,%ecx
 80afbf0:       09 c6                   or     %eax,%esi
 80afbf2:       84 db                   test   %bl,%bl
 80afbf4:       78 ea                   js     80afbe0 <__gcc_personality_v0+0x110>
 80afbf6:       01 d6                   add    %edx,%esi
 80afbf8:       89 74 24 44             mov    %esi,0x44(%esp)
 80afbfc:       8d 72 01                lea    0x1(%edx),%esi
 80afbff:       0f b6 12                movzbl (%edx),%edx
 80afc02:       31 ed                   xor    %ebp,%ebp
 80afc04:       31 c9                   xor    %ecx,%ecx
 80afc06:       88 54 24 4d             mov    %dl,0x4d(%esp)
 80afc0a:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80afc11:       00
 80afc12:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80afc19:       00
 80afc1a:       8d b6 00 00 00 00       lea    0x0(%esi),%esi
 80afc20:       0f b6 1e                movzbl (%esi),%ebx
 80afc23:       83 c6 01                add    $0x1,%esi
 80afc26:       89 d8                   mov    %ebx,%eax
 80afc28:       83 e0 7f                and    $0x7f,%eax
 80afc2b:       d3 e0                   shl    %cl,%eax
 80afc2d:       83 c1 07                add    $0x7,%ecx
 80afc30:       09 c5                   or     %eax,%ebp
 80afc32:       84 db                   test   %bl,%bl
 80afc34:       78 ea                   js     80afc20 <__gcc_personality_v0+0x150>
 80afc36:       8d 04 2e                lea    (%esi,%ebp,1),%eax
 80afc39:       88 54 24 0c             mov    %dl,0xc(%esp)
 80afc3d:       83 ec 08                sub    $0x8,%esp
 80afc40:       89 44 24 10             mov    %eax,0x10(%esp)
 80afc44:       89 44 24 50             mov    %eax,0x50(%esp)
 80afc48:       8d 44 24 30             lea    0x30(%esp),%eax
 80afc4c:       50                      push   %eax
 80afc4d:       ff b4 24 90 00 00 00    push   0x90(%esp)
 80afc54:       8b 5c 24 28             mov    0x28(%esp),%ebx
 80afc58:       e8 73 c1 ff ff          call   80abdd0 <_Unwind_GetIPInfo>
 80afc5d:       83 c4 10                add    $0x10,%esp
 80afc60:       89 c5                   mov    %eax,%ebp
 80afc62:       8b 44 24 08             mov    0x8(%esp),%eax
 80afc66:       83 7c 24 28 01          cmpl   $0x1,0x28(%esp)
 80afc6b:       83 dd 00                sbb    $0x0,%ebp
 80afc6e:       39 c6                   cmp    %eax,%esi
 80afc70:       0f 83 8a fe ff ff       jae    80afb00 <__gcc_personality_v0+0x30>
 80afc76:       8d 44 24 2c             lea    0x2c(%esp),%eax
 80afc7a:       0f b6 5c 24 0c          movzbl 0xc(%esp),%ebx
 80afc7f:       89 44 24 0c             mov    %eax,0xc(%esp)
 80afc83:       8d 44 24 30             lea    0x30(%esp),%eax
 80afc87:       89 44 24 10             mov    %eax,0x10(%esp)
 80afc8b:       8d 44 24 34             lea    0x34(%esp),%eax
 80afc8f:       89 44 24 14             mov    %eax,0x14(%esp)
 80afc93:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80afc9a:       00
 80afc9b:       2e 8d 74 26 00          lea    %cs:0x0(%esi,%eiz,1),%esi
 80afca0:       31 d2                   xor    %edx,%edx
 80afca2:       89 d8                   mov    %ebx,%eax
 80afca4:       e8 17 fc ff ff          call   80af8c0 <base_of_encoded_value>
 80afca9:       83 ec 0c                sub    $0xc,%esp
 80afcac:       89 f1                   mov    %esi,%ecx
 80afcae:       ff 74 24 18             push   0x18(%esp)
 80afcb2:       89 c2                   mov    %eax,%edx
 80afcb4:       89 d8                   mov    %ebx,%eax
 80afcb6:       e8 95 fc ff ff          call   80af950 <read_encoded_value_with_base>
 80afcbb:       83 c4 10                add    $0x10,%esp
 80afcbe:       31 d2                   xor    %edx,%edx
 80afcc0:       89 c6                   mov    %eax,%esi
 80afcc2:       89 d8                   mov    %ebx,%eax
 80afcc4:       e8 f7 fb ff ff          call   80af8c0 <base_of_encoded_value>
 80afcc9:       83 ec 0c                sub    $0xc,%esp
 80afccc:       89 f1                   mov    %esi,%ecx
 80afcce:       ff 74 24 1c             push   0x1c(%esp)
 80afcd2:       89 c2                   mov    %eax,%edx
 80afcd4:       89 d8                   mov    %ebx,%eax
 80afcd6:       e8 75 fc ff ff          call   80af950 <read_encoded_value_with_base>
 80afcdb:       83 c4 10                add    $0x10,%esp
 80afcde:       31 d2                   xor    %edx,%edx
 80afce0:       89 c6                   mov    %eax,%esi
 80afce2:       89 d8                   mov    %ebx,%eax
 80afce4:       e8 d7 fb ff ff          call   80af8c0 <base_of_encoded_value>
 80afce9:       83 ec 0c                sub    $0xc,%esp
 80afcec:       89 f1                   mov    %esi,%ecx
 80afcee:       ff 74 24 20             push   0x20(%esp)
 80afcf2:       89 c2                   mov    %eax,%edx
 80afcf4:       89 d8                   mov    %ebx,%eax
 80afcf6:       e8 55 fc ff ff          call   80af950 <read_encoded_value_with_base>
 80afcfb:       83 c4 10                add    $0x10,%esp
 80afcfe:       89 c6                   mov    %eax,%esi
 80afd00:       83 c6 01                add    $0x1,%esi
 80afd03:       80 7e ff 00             cmpb   $0x0,-0x1(%esi)
 80afd07:       78 f7                   js     80afd00 <__gcc_personality_v0+0x230>
 80afd09:       8b 44 24 2c             mov    0x2c(%esp),%eax
 80afd0d:       01 f8                   add    %edi,%eax
 80afd0f:       39 c5                   cmp    %eax,%ebp
 80afd11:       0f 82 e9 fd ff ff       jb     80afb00 <__gcc_personality_v0+0x30>
 80afd17:       03 44 24 30             add    0x30(%esp),%eax
 80afd1b:       39 c5                   cmp    %eax,%ebp
 80afd1d:       72 11                   jb     80afd30 <__gcc_personality_v0+0x260>
 80afd1f:       8b 44 24 08             mov    0x8(%esp),%eax
 80afd23:       39 c6                   cmp    %eax,%esi
 80afd25:       0f 82 75 ff ff ff       jb     80afca0 <__gcc_personality_v0+0x1d0>
 80afd2b:       e9 d0 fd ff ff          jmp    80afb00 <__gcc_personality_v0+0x30>
 80afd30:       8b 44 24 34             mov    0x34(%esp),%eax
 80afd34:       85 c0                   test   %eax,%eax
 80afd36:       0f 84 c4 fd ff ff       je     80afb00 <__gcc_personality_v0+0x30>
 80afd3c:       8b 74 24 1c             mov    0x1c(%esp),%esi
 80afd40:       01 c6                   add    %eax,%esi
 80afd42:       0f 84 b8 fd ff ff       je     80afb00 <__gcc_personality_v0+0x30>
 80afd48:       83 ec 04                sub    $0x4,%esp
 80afd4b:       ff b4 24 84 00 00 00    push   0x84(%esp)
 80afd52:       6a 00                   push   $0x0
 80afd54:       ff b4 24 90 00 00 00    push   0x90(%esp)
 80afd5b:       8b 5c 24 28             mov    0x28(%esp),%ebx
 80afd5f:       e8 fc bf ff ff          call   80abd60 <_Unwind_SetGR>
 80afd64:       83 c4 0c                add    $0xc,%esp
 80afd67:       6a 00                   push   $0x0
 80afd69:       6a 02                   push   $0x2
 80afd6b:       ff b4 24 90 00 00 00    push   0x90(%esp)
 80afd72:       e8 e9 bf ff ff          call   80abd60 <_Unwind_SetGR>
 80afd77:       58                      pop    %eax
 80afd78:       5a                      pop    %edx
 80afd79:       56                      push   %esi
 80afd7a:       ff b4 24 90 00 00 00    push   0x90(%esp)
 80afd81:       e8 6a c0 ff ff          call   80abdf0 <_Unwind_SetIP>
 80afd86:       83 c4 10                add    $0x10,%esp
 80afd89:       b8 07 00 00 00          mov    $0x7,%eax
 80afd8e:       e9 82 fd ff ff          jmp    80afb15 <__gcc_personality_v0+0x45>
 80afd93:       3c 40                   cmp    $0x40,%al
 80afd95:       75 47                   jne    80afdde <__gcc_personality_v0+0x30e>
 80afd97:       83 ec 0c                sub    $0xc,%esp
 80afd9a:       ff b4 24 90 00 00 00    push   0x90(%esp)
 80afda1:       8b 5c 24 28             mov    0x28(%esp),%ebx
 80afda5:       e8 66 c0 ff ff          call   80abe10 <_Unwind_GetRegionStart>
 80afdaa:       83 c4 10                add    $0x10,%esp
 80afdad:       89 c2                   mov    %eax,%edx
 80afdaf:       e9 eb fd ff ff          jmp    80afb9f <__gcc_personality_v0+0xcf>
 80afdb4:       83 ec 0c                sub    $0xc,%esp
 80afdb7:       ff b4 24 90 00 00 00    push   0x90(%esp)
 80afdbe:       8b 5c 24 28             mov    0x28(%esp),%ebx
 80afdc2:       e8 99 c0 ff ff          call   80abe60 <_Unwind_GetDataRelBase>
 80afdc7:       83 c4 10                add    $0x10,%esp
 80afdca:       89 c2                   mov    %eax,%edx
 80afdcc:       e9 ce fd ff ff          jmp    80afb9f <__gcc_personality_v0+0xcf>
 80afdd1:       0f 87 92 98 f9 ff       ja     8049669 <__gcc_personality_v0.cold>
 80afdd7:       31 d2                   xor    %edx,%edx
 80afdd9:       e9 c1 fd ff ff          jmp    80afb9f <__gcc_personality_v0+0xcf>
 80afdde:       3c 50                   cmp    $0x50,%al
 80afde0:       74 f5                   je     80afdd7 <__gcc_personality_v0+0x307>
 80afde2:       e9 82 98 f9 ff          jmp    8049669 <__gcc_personality_v0.cold>
 80afde7:       66 90                   xchg   %ax,%ax
 80afde9:       66 90                   xchg   %ax,%ax
 80afdeb:       66 90                   xchg   %ax,%ax
 80afded:       66 90                   xchg   %ax,%ax
 80afdef:       66 90                   xchg   %ax,%ax
 80afdf1:       66 90                   xchg   %ax,%ax
 80afdf3:       66 90                   xchg   %ax,%ax
 80afdf5:       66 90                   xchg   %ax,%ax
 80afdf7:       66 90                   xchg   %ax,%ax
 80afdf9:       66 90                   xchg   %ax,%ax
 80afdfb:       66 90                   xchg   %ax,%ax
 80afdfd:       66 90                   xchg   %ax,%ax
 80afdff:       90                      nop

080afe00 <___pthread_cond_broadcast>:
 80afe00:       e8 3c 9b f9 ff          call   8049941 <__x86.get_pc_thunk.ax>
 80afe05:       05 ef 71 03 00          add    $0x371ef,%eax
 80afe0a:       55                      push   %ebp
 80afe0b:       57                      push   %edi
 80afe0c:       56                      push   %esi
 80afe0d:       53                      push   %ebx
 80afe0e:       83 ec 3c                sub    $0x3c,%esp
 80afe11:       8b 7c 24 50             mov    0x50(%esp),%edi
 80afe15:       89 44 24 18             mov    %eax,0x18(%esp)
 80afe19:       8b 47 24                mov    0x24(%edi),%eax
 80afe1c:       89 c6                   mov    %eax,%esi
 80afe1e:       c1 ee 03                shr    $0x3,%esi
 80afe21:       0f 84 ff 00 00 00       je     80aff26 <___pthread_cond_broadcast+0x126>
 80afe27:       c1 e0 07                shl    $0x7,%eax
 80afe2a:       0f b6 c0                movzbl %al,%eax
 80afe2d:       89 44 24 04             mov    %eax,0x4(%esp)
 80afe31:       8d 47 20                lea    0x20(%edi),%eax
 80afe34:       89 04 24                mov    %eax,(%esp)
 80afe37:       8b 47 20                mov    0x20(%edi),%eax
 80afe3a:       a8 03                   test   $0x3,%al
 80afe3c:       0f 85 42 03 00 00       jne    80b0184 <___pthread_cond_broadcast+0x384>
 80afe42:       89 c2                   mov    %eax,%edx
 80afe44:       8b 34 24                mov    (%esp),%esi
 80afe47:       83 ca 01                or     $0x1,%edx
 80afe4a:       f0 0f b1 16             lock cmpxchg %edx,(%esi)
 80afe4e:       75 ea                   jne    80afe3a <___pthread_cond_broadcast+0x3a>
 80afe50:       83 ec 0c                sub    $0xc,%esp
 80afe53:       57                      push   %edi
 80afe54:       e8 a7 6d ff ff          call   80a6c00 <__atomic_wide_counter_load_relaxed>
 80afe59:       89 c1                   mov    %eax,%ecx
 80afe5b:       89 44 24 20             mov    %eax,0x20(%esp)
 80afe5f:       0f ac d0 01             shrd   $0x1,%edx,%eax
 80afe63:       f7 d1                   not    %ecx
 80afe65:       d1 ea                   shr    $1,%edx
 80afe67:       89 44 24 18             mov    %eax,0x18(%esp)
 80afe6b:       83 e1 01                and    $0x1,%ecx
 80afe6e:       89 54 24 1c             mov    %edx,0x1c(%esp)
 80afe72:       83 c4 10                add    $0x10,%esp
 80afe75:       8d 51 04                lea    0x4(%ecx),%edx
 80afe78:       89 cd                   mov    %ecx,%ebp
 80afe7a:       8b 44 97 08             mov    0x8(%edi,%edx,4),%eax
 80afe7e:       85 c0                   test   %eax,%eax
 80afe80:       74 36                   je     80afeb8 <___pthread_cond_broadcast+0xb8>
 80afe82:       8d 5c 8f 28             lea    0x28(%edi,%ecx,4),%ebx
 80afe86:       01 c0                   add    %eax,%eax
 80afe88:       f0 01 03                lock add %eax,(%ebx)
 80afe8b:       b8 f0 00 00 00          mov    $0xf0,%eax
 80afe90:       31 f6                   xor    %esi,%esi
 80afe92:       8b 4c 24 04             mov    0x4(%esp),%ecx
 80afe96:       80 f1 81                xor    $0x81,%cl
 80afe99:       c7 44 97 08 00 00 00    movl   $0x0,0x8(%edi,%edx,4)
 80afea0:       00
 80afea1:       ba ff ff ff 7f          mov    $0x7fffffff,%edx
 80afea6:       65 ff 15 10 00 00 00    call   *%gs:0x10
 80afead:       3d 00 f0 ff ff          cmp    $0xfffff000,%eax
 80afeb2:       0f 87 bb 02 00 00       ja     80b0173 <___pthread_cond_broadcast+0x373>
 80afeb8:       8b 47 20                mov    0x20(%edi),%eax
 80afebb:       83 ec 0c                sub    $0xc,%esp
 80afebe:       c1 e8 02                shr    $0x2,%eax
 80afec1:       89 44 24 20             mov    %eax,0x20(%esp)
 80afec5:       89 c6                   mov    %eax,%esi
 80afec7:       8d 47 08                lea    0x8(%edi),%eax
 80afeca:       89 44 24 28             mov    %eax,0x28(%esp)
 80afece:       50                      push   %eax
 80afecf:       e8 2c 6d ff ff          call   80a6c00 <__atomic_wide_counter_load_relaxed>
 80afed4:       89 c1                   mov    %eax,%ecx
 80afed6:       0f ac d1 01             shrd   $0x1,%edx,%ecx
 80afeda:       8b 54 24 20             mov    0x20(%esp),%edx
 80afede:       89 4c 24 30             mov    %ecx,0x30(%esp)
 80afee2:       83 e2 01                and    $0x1,%edx
 80afee5:       8d 04 95 00 00 00 00    lea    0x0(,%edx,4),%eax
 80afeec:       8d 1c 07                lea    (%edi,%eax,1),%ebx
 80afeef:       89 44 24 20             mov    %eax,0x20(%esp)
 80afef3:       8b 44 24 18             mov    0x18(%esp),%eax
 80afef7:       03 43 18                add    0x18(%ebx),%eax
 80afefa:       89 5c 24 34             mov    %ebx,0x34(%esp)
 80afefe:       83 c4 10                add    $0x10,%esp
 80aff01:       29 f0                   sub    %esi,%eax
 80aff03:       39 c1                   cmp    %eax,%ecx
 80aff05:       75 29                   jne    80aff30 <___pthread_cond_broadcast+0x130>
 80aff07:       8b 47 20                mov    0x20(%edi),%eax
 80aff0a:       89 c1                   mov    %eax,%ecx
 80aff0c:       8b 34 24                mov    (%esp),%esi
 80aff0f:       89 c2                   mov    %eax,%edx
 80aff11:       83 e1 fc                and    $0xfffffffc,%ecx
 80aff14:       f0 0f b1 0e             lock cmpxchg %ecx,(%esi)
 80aff18:       75 f0                   jne    80aff0a <___pthread_cond_broadcast+0x10a>
 80aff1a:       83 e2 03                and    $0x3,%edx
 80aff1d:       83 fa 02                cmp    $0x2,%edx
 80aff20:       0f 84 d2 01 00 00       je     80b00f8 <___pthread_cond_broadcast+0x2f8>
 80aff26:       83 c4 3c                add    $0x3c,%esp
 80aff29:       31 c0                   xor    %eax,%eax
 80aff2b:       5b                      pop    %ebx
 80aff2c:       5e                      pop    %esi
 80aff2d:       5f                      pop    %edi
 80aff2e:       5d                      pop    %ebp
 80aff2f:       c3                      ret
 80aff30:       8d 77 28                lea    0x28(%edi),%esi
 80aff33:       8d 04 ad 00 00 00 00    lea    0x0(,%ebp,4),%eax
 80aff3a:       89 74 24 08             mov    %esi,0x8(%esp)
 80aff3e:       01 c6                   add    %eax,%esi
 80aff40:       89 74 24 28             mov    %esi,0x28(%esp)
 80aff44:       f0 83 0e 01             lock orl $0x1,(%esi)
 80aff48:       8d 5c 07 10             lea    0x10(%edi,%eax,1),%ebx
 80aff4c:       8b 03                   mov    (%ebx),%eax
 80aff4e:       89 c6                   mov    %eax,%esi
 80aff50:       f0 0f b1 03             lock cmpxchg %eax,(%ebx)
 80aff54:       75 f8                   jne    80aff4e <___pthread_cond_broadcast+0x14e>
 80aff56:       8b 4c 24 04             mov    0x4(%esp),%ecx
 80aff5a:       bd f0 00 00 00          mov    $0xf0,%ebp
 80aff5f:       80 f1 80                xor    $0x80,%cl
 80aff62:       d1 ee                   shr    $1,%esi
 80aff64:       74 3d                   je     80affa3 <___pthread_cond_broadcast+0x1a3>
 80aff66:       89 54 24 2c             mov    %edx,0x2c(%esp)
 80aff6a:       8d b6 00 00 00 00       lea    0x0(%esi),%esi
 80aff70:       8b 03                   mov    (%ebx),%eax
 80aff72:       89 c2                   mov    %eax,%edx
 80aff74:       83 ca 01                or     $0x1,%edx
 80aff77:       f0 0f b1 13             lock cmpxchg %edx,(%ebx)
 80aff7b:       75 f5                   jne    80aff72 <___pthread_cond_broadcast+0x172>
 80aff7d:       89 d0                   mov    %edx,%eax
 80aff7f:       d1 e8                   shr    $1,%eax
 80aff81:       74 16                   je     80aff99 <___pthread_cond_broadcast+0x199>
 80aff83:       31 f6                   xor    %esi,%esi
 80aff85:       89 e8                   mov    %ebp,%eax
 80aff87:       65 ff 15 10 00 00 00    call   *%gs:0x10
 80aff8e:       3d 00 f0 ff ff          cmp    $0xfffff000,%eax
 80aff93:       0f 87 47 01 00 00       ja     80b00e0 <___pthread_cond_broadcast+0x2e0>
 80aff99:       8b 03                   mov    (%ebx),%eax
 80aff9b:       d1 e8                   shr    $1,%eax
 80aff9d:       75 d1                   jne    80aff70 <___pthread_cond_broadcast+0x170>
 80aff9f:       8b 54 24 2c             mov    0x2c(%esp),%edx
 80affa3:       f7 da                   neg    %edx
 80affa5:       19 c0                   sbb    %eax,%eax
 80affa7:       83 ec 08                sub    $0x8,%esp
 80affaa:       8b 74 24 1c             mov    0x1c(%esp),%esi
 80affae:       83 c8 01                or     $0x1,%eax
 80affb1:       8d 04 70                lea    (%eax,%esi,2),%eax
 80affb4:       50                      push   %eax
 80affb5:       ff 74 24 28             push   0x28(%esp)
 80affb9:       e8 f2 6b ff ff          call   80a6bb0 <__atomic_wide_counter_fetch_add_relaxed>
 80affbe:       8b 44 24 38             mov    0x38(%esp),%eax
 80affc2:       83 c4 10                add    $0x10,%esp
 80affc5:       c7 00 00 00 00 00       movl   $0x0,(%eax)
 80affcb:       8d 47 04                lea    0x4(%edi),%eax
 80affce:       66 90                   xchg   %ax,%ax
 80affd0:       8b 08                   mov    (%eax),%ecx
 80affd2:       8b 1f                   mov    (%edi),%ebx
 80affd4:       8b 10                   mov    (%eax),%edx
 80affd6:       39 d1                   cmp    %edx,%ecx
 80affd8:       75 f6                   jne    80affd0 <___pthread_cond_broadcast+0x1d0>
 80affda:       89 c8                   mov    %ecx,%eax
 80affdc:       f7 d0                   not    %eax
 80affde:       21 d8                   and    %ebx,%eax
 80affe0:       c1 e8 1f                shr    $0x1f,%eax
 80affe3:       01 c8                   add    %ecx,%eax
 80affe5:       25 ff ff ff 7f          and    $0x7fffffff,%eax
 80affea:       89 c2                   mov    %eax,%edx
 80affec:       8b 07                   mov    (%edi),%eax
 80affee:       89 c6                   mov    %eax,%esi
 80afff0:       89 c1                   mov    %eax,%ecx
 80afff2:       83 f6 01                xor    $0x1,%esi
 80afff5:       f0 0f b1 37             lock cmpxchg %esi,(%edi)
 80afff9:       75 f3                   jne    80affee <___pthread_cond_broadcast+0x1ee>
 80afffb:       81 e1 ff ff ff 7f       and    $0x7fffffff,%ecx
 80b0001:       81 e3 ff ff ff 7f       and    $0x7fffffff,%ebx
 80b0007:       39 d9                   cmp    %ebx,%ecx
 80b0009:       83 d2 00                adc    $0x0,%edx
 80b000c:       89 d0                   mov    %edx,%eax
 80b000e:       31 d2                   xor    %edx,%edx
 80b0010:       0f a4 c2 1f             shld   $0x1f,%eax,%edx
 80b0014:       c1 e0 1f                shl    $0x1f,%eax
 80b0017:       01 c8                   add    %ecx,%eax
 80b0019:       83 d2 00                adc    $0x0,%edx
 80b001c:       0f ac d0 01             shrd   $0x1,%edx,%eax
 80b0020:       8b 54 24 14             mov    0x14(%esp),%edx
 80b0024:       89 c1                   mov    %eax,%ecx
 80b0026:       8b 44 24 20             mov    0x20(%esp),%eax
 80b002a:       01 c2                   add    %eax,%edx
 80b002c:       89 c8                   mov    %ecx,%eax
 80b002e:       29 d0                   sub    %edx,%eax
 80b0030:       8b 57 20                mov    0x20(%edi),%edx
 80b0033:       8b 34 24                mov    (%esp),%esi
 80b0036:       8d 0c 85 00 00 00 00    lea    0x0(,%eax,4),%ecx
 80b003d:       83 e2 03                and    $0x3,%edx
 80b0040:       09 ca                   or     %ecx,%edx
 80b0042:       89 d3                   mov    %edx,%ebx
 80b0044:       87 1e                   xchg   %ebx,(%esi)
 80b0046:       31 da                   xor    %ebx,%edx
 80b0048:       83 e2 03                and    $0x3,%edx
 80b004b:       0f 85 17 01 00 00       jne    80b0168 <___pthread_cond_broadcast+0x368>
 80b0051:       8b 74 24 24             mov    0x24(%esp),%esi
 80b0055:       03 46 18                add    0x18(%esi),%eax
 80b0058:       89 46 18                mov    %eax,0x18(%esi)
 80b005b:       0f 84 a6 fe ff ff       je     80aff07 <___pthread_cond_broadcast+0x107>
 80b0061:       8b 4c 24 10             mov    0x10(%esp),%ecx
 80b0065:       01 c0                   add    %eax,%eax
 80b0067:       f0 01 44 0f 28          lock add %eax,0x28(%edi,%ecx,1)
 80b006c:       c7 46 18 00 00 00 00    movl   $0x0,0x18(%esi)
 80b0073:       8b 47 20                mov    0x20(%edi),%eax
 80b0076:       89 c1                   mov    %eax,%ecx
 80b0078:       8b 34 24                mov    (%esp),%esi
 80b007b:       89 c2                   mov    %eax,%edx
 80b007d:       83 e1 fc                and    $0xfffffffc,%ecx
 80b0080:       f0 0f b1 0e             lock cmpxchg %ecx,(%esi)
 80b0084:       75 f0                   jne    80b0076 <___pthread_cond_broadcast+0x276>
 80b0086:       8b 4c 24 04             mov    0x4(%esp),%ecx
 80b008a:       83 e2 03                and    $0x3,%edx
 80b008d:       80 f1 81                xor    $0x81,%cl
 80b0090:       83 fa 02                cmp    $0x2,%edx
 80b0093:       0f 84 97 00 00 00       je     80b0130 <___pthread_cond_broadcast+0x330>
 80b0099:       8b 44 24 10             mov    0x10(%esp),%eax
 80b009d:       8b 5c 24 08             mov    0x8(%esp),%ebx
 80b00a1:       ba ff ff ff 7f          mov    $0x7fffffff,%edx
 80b00a6:       31 f6                   xor    %esi,%esi
 80b00a8:       01 c3                   add    %eax,%ebx
 80b00aa:       b8 f0 00 00 00          mov    $0xf0,%eax
 80b00af:       65 ff 15 10 00 00 00    call   *%gs:0x10
 80b00b6:       3d 00 f0 ff ff          cmp    $0xfffff000,%eax
 80b00bb:       0f 86 65 fe ff ff       jbe    80aff26 <___pthread_cond_broadcast+0x126>
 80b00c1:       83 c0 16                add    $0x16,%eax
 80b00c4:       83 e0 f7                and    $0xfffffff7,%eax
 80b00c7:       0f 84 59 fe ff ff       je     80aff26 <___pthread_cond_broadcast+0x126>
 80b00cd:       83 ec 0c                sub    $0xc,%esp
 80b00d0:       8b 5c 24 24             mov    0x24(%esp),%ebx
 80b00d4:       8d 83 ec d6 fc ff       lea    -0x32914(%ebx),%eax
 80b00da:       50                      push   %eax
 80b00db:       e8 80 ce f9 ff          call   804cf60 <__libc_fatal>
 80b00e0:       83 f8 f5                cmp    $0xfffffff5,%eax
 80b00e3:       0f 84 b0 fe ff ff       je     80aff99 <___pthread_cond_broadcast+0x199>
 80b00e9:       83 f8 fc                cmp    $0xfffffffc,%eax
 80b00ec:       0f 84 a7 fe ff ff       je     80aff99 <___pthread_cond_broadcast+0x199>
 80b00f2:       eb d9                   jmp    80b00cd <___pthread_cond_broadcast+0x2cd>
 80b00f4:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi
 80b00f8:       8b 4c 24 04             mov    0x4(%esp),%ecx
 80b00fc:       8b 1c 24                mov    (%esp),%ebx
 80b00ff:       b8 f0 00 00 00          mov    $0xf0,%eax
 80b0104:       31 f6                   xor    %esi,%esi
 80b0106:       ba 01 00 00 00          mov    $0x1,%edx
 80b010b:       80 f1 81                xor    $0x81,%cl
 80b010e:       65 ff 15 10 00 00 00    call   *%gs:0x10
 80b0115:       3d 00 f0 ff ff          cmp    $0xfffff000,%eax
 80b011a:       77 a5                   ja     80b00c1 <___pthread_cond_broadcast+0x2c1>
 80b011c:       83 c4 3c                add    $0x3c,%esp
 80b011f:       31 c0                   xor    %eax,%eax
 80b0121:       5b                      pop    %ebx
 80b0122:       5e                      pop    %esi
 80b0123:       5f                      pop    %edi
 80b0124:       5d                      pop    %ebp
 80b0125:       c3                      ret
 80b0126:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80b012d:       00
 80b012e:       66 90                   xchg   %ax,%ax
 80b0130:       8b 1c 24                mov    (%esp),%ebx
 80b0133:       b8 f0 00 00 00          mov    $0xf0,%eax
 80b0138:       ba 01 00 00 00          mov    $0x1,%edx
 80b013d:       31 f6                   xor    %esi,%esi
 80b013f:       65 ff 15 10 00 00 00    call   *%gs:0x10
 80b0146:       3d 00 f0 ff ff          cmp    $0xfffff000,%eax
 80b014b:       0f 86 48 ff ff ff       jbe    80b0099 <___pthread_cond_broadcast+0x299>
 80b0151:       83 c0 16                add    $0x16,%eax
 80b0154:       83 e0 f7                and    $0xfffffff7,%eax
 80b0157:       0f 84 3c ff ff ff       je     80b0099 <___pthread_cond_broadcast+0x299>
 80b015d:       e9 6b ff ff ff          jmp    80b00cd <___pthread_cond_broadcast+0x2cd>
 80b0162:       8d b6 00 00 00 00       lea    0x0(%esi),%esi
 80b0168:       83 c9 02                or     $0x2,%ecx
 80b016b:       89 4f 20                mov    %ecx,0x20(%edi)
 80b016e:       e9 de fe ff ff          jmp    80b0051 <___pthread_cond_broadcast+0x251>
 80b0173:       83 c0 16                add    $0x16,%eax
 80b0176:       83 e0 f7                and    $0xfffffff7,%eax
 80b0179:       0f 85 4e ff ff ff       jne    80b00cd <___pthread_cond_broadcast+0x2cd>
 80b017f:       e9 34 fd ff ff          jmp    80afeb8 <___pthread_cond_broadcast+0xb8>
 80b0184:       8b 4c 24 04             mov    0x4(%esp),%ecx
 80b0188:       8b 1c 24                mov    (%esp),%ebx
 80b018b:       bd f0 00 00 00          mov    $0xf0,%ebp
 80b0190:       80 f1 80                xor    $0x80,%cl
 80b0193:       eb 2b                   jmp    80b01c0 <___pthread_cond_broadcast+0x3c0>
 80b0195:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80b019c:       00
 80b019d:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80b01a4:       00
 80b01a5:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80b01ac:       00
 80b01ad:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80b01b4:       00
 80b01b5:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80b01bc:       00
 80b01bd:       8d 76 00                lea    0x0(%esi),%esi
 80b01c0:       89 c2                   mov    %eax,%edx
 80b01c2:       83 e2 03                and    $0x3,%edx
 80b01c5:       83 fa 02                cmp    $0x2,%edx
 80b01c8:       74 25                   je     80b01ef <___pthread_cond_broadcast+0x3ef>
 80b01ca:       89 c2                   mov    %eax,%edx
 80b01cc:       83 e2 fc                and    $0xfffffffc,%edx
 80b01cf:       83 ca 02                or     $0x2,%edx
 80b01d2:       f0 0f b1 13             lock cmpxchg %edx,(%ebx)
 80b01d6:       0f 94 c2                sete   %dl
 80b01d9:       0f b6 d2                movzbl %dl,%edx
 80b01dc:       89 d6                   mov    %edx,%esi
 80b01de:       89 c2                   mov    %eax,%edx
 80b01e0:       83 e2 03                and    $0x3,%edx
 80b01e3:       85 f6                   test   %esi,%esi
 80b01e5:       74 d9                   je     80b01c0 <___pthread_cond_broadcast+0x3c0>
 80b01e7:       85 d2                   test   %edx,%edx
 80b01e9:       0f 84 61 fc ff ff       je     80afe50 <___pthread_cond_broadcast+0x50>
 80b01ef:       83 e0 fc                and    $0xfffffffc,%eax
 80b01f2:       31 f6                   xor    %esi,%esi
 80b01f4:       89 c2                   mov    %eax,%edx
 80b01f6:       89 e8                   mov    %ebp,%eax
 80b01f8:       83 ca 02                or     $0x2,%edx
 80b01fb:       65 ff 15 10 00 00 00    call   *%gs:0x10
 80b0202:       3d 00 f0 ff ff          cmp    $0xfffff000,%eax
 80b0207:       77 07                   ja     80b0210 <___pthread_cond_broadcast+0x410>
 80b0209:       8b 03                   mov    (%ebx),%eax
 80b020b:       eb b3                   jmp    80b01c0 <___pthread_cond_broadcast+0x3c0>
 80b020d:       8d 76 00                lea    0x0(%esi),%esi
 80b0210:       83 f8 f5                cmp    $0xfffffff5,%eax
 80b0213:       74 f4                   je     80b0209 <___pthread_cond_broadcast+0x409>
 80b0215:       83 f8 fc                cmp    $0xfffffffc,%eax
 80b0218:       0f 85 af fe ff ff       jne    80b00cd <___pthread_cond_broadcast+0x2cd>
 80b021e:       8b 03                   mov    (%ebx),%eax
 80b0220:       eb 9e                   jmp    80b01c0 <___pthread_cond_broadcast+0x3c0>
 80b0222:       66 90                   xchg   %ax,%ax
 80b0224:       66 90                   xchg   %ax,%ax
 80b0226:       66 90                   xchg   %ax,%ax
 80b0228:       66 90                   xchg   %ax,%ax
 80b022a:       66 90                   xchg   %ax,%ax
 80b022c:       66 90                   xchg   %ax,%ax
 80b022e:       66 90                   xchg   %ax,%ax
 80b0230:       66 90                   xchg   %ax,%ax
 80b0232:       66 90                   xchg   %ax,%ax
 80b0234:       66 90                   xchg   %ax,%ax
 80b0236:       66 90                   xchg   %ax,%ax
 80b0238:       66 90                   xchg   %ax,%ax
 80b023a:       66 90                   xchg   %ax,%ax
 80b023c:       66 90                   xchg   %ax,%ax
 80b023e:       66 90                   xchg   %ax,%ax

080b0240 <__condvar_dec_grefs>:
 80b0240:       57                      push   %edi
 80b0241:       e8 ed bb f9 ff          call   804be33 <__x86.get_pc_thunk.di>
 80b0246:       81 c7 ae 6d 03 00       add    $0x36dae,%edi
 80b024c:       56                      push   %esi
 80b024d:       53                      push   %ebx
 80b024e:       8d 5c 90 10             lea    0x10(%eax,%edx,4),%ebx
 80b0252:       b8 fe ff ff ff          mov    $0xfffffffe,%eax
 80b0257:       f0 0f c1 03             lock xadd %eax,(%ebx)
 80b025b:       83 f8 03                cmp    $0x3,%eax
 80b025e:       74 08                   je     80b0268 <__condvar_dec_grefs+0x28>
 80b0260:       5b                      pop    %ebx
 80b0261:       5e                      pop    %esi
 80b0262:       5f                      pop    %edi
 80b0263:       c3                      ret
 80b0264:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi
 80b0268:       f0 83 23 fe             lock andl $0xfffffffe,(%ebx)
 80b026c:       b8 f0 00 00 00          mov    $0xf0,%eax
 80b0271:       ba ff ff ff 7f          mov    $0x7fffffff,%edx
 80b0276:       31 f6                   xor    %esi,%esi
 80b0278:       80 f1 81                xor    $0x81,%cl
 80b027b:       65 ff 15 10 00 00 00    call   *%gs:0x10
 80b0282:       3d 00 f0 ff ff          cmp    $0xfffff000,%eax
 80b0287:       76 d7                   jbe    80b0260 <__condvar_dec_grefs+0x20>
 80b0289:       83 c0 16                add    $0x16,%eax
 80b028c:       83 e0 f7                and    $0xfffffff7,%eax
 80b028f:       74 cf                   je     80b0260 <__condvar_dec_grefs+0x20>
 80b0291:       83 ec 0c                sub    $0xc,%esp
 80b0294:       8d 87 ec d6 fc ff       lea    -0x32914(%edi),%eax
 80b029a:       89 fb                   mov    %edi,%ebx
 80b029c:       50                      push   %eax
 80b029d:       e8 be cc f9 ff          call   804cf60 <__libc_fatal>
 80b02a2:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80b02a9:       00
 80b02aa:       8d b6 00 00 00 00       lea    0x0(%esi),%esi

080b02b0 <__condvar_confirm_wakeup>:
 80b02b0:       57                      push   %edi
 80b02b1:       e8 7d bb f9 ff          call   804be33 <__x86.get_pc_thunk.di>
 80b02b6:       81 c7 3e 6d 03 00       add    $0x36d3e,%edi
 80b02bc:       56                      push   %esi
 80b02bd:       53                      push   %ebx
 80b02be:       8d 58 24                lea    0x24(%eax),%ebx
 80b02c1:       b8 f8 ff ff ff          mov    $0xfffffff8,%eax
 80b02c6:       f0 0f c1 03             lock xadd %eax,(%ebx)
 80b02ca:       c1 e8 02                shr    $0x2,%eax
 80b02cd:       83 f8 03                cmp    $0x3,%eax
 80b02d0:       74 0e                   je     80b02e0 <__condvar_confirm_wakeup+0x30>
 80b02d2:       5b                      pop    %ebx
 80b02d3:       5e                      pop    %esi
 80b02d4:       5f                      pop    %edi
 80b02d5:       c3                      ret
 80b02d6:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80b02dd:       00
 80b02de:       66 90                   xchg   %ax,%ax
 80b02e0:       89 d1                   mov    %edx,%ecx
 80b02e2:       b8 f0 00 00 00          mov    $0xf0,%eax
 80b02e7:       ba ff ff ff 7f          mov    $0x7fffffff,%edx
 80b02ec:       31 f6                   xor    %esi,%esi
 80b02ee:       80 f1 81                xor    $0x81,%cl
 80b02f1:       65 ff 15 10 00 00 00    call   *%gs:0x10
 80b02f8:       3d 00 f0 ff ff          cmp    $0xfffff000,%eax
 80b02fd:       76 d3                   jbe    80b02d2 <__condvar_confirm_wakeup+0x22>
 80b02ff:       83 c0 16                add    $0x16,%eax
 80b0302:       83 e0 f7                and    $0xfffffff7,%eax
 80b0305:       74 cb                   je     80b02d2 <__condvar_confirm_wakeup+0x22>
 80b0307:       83 ec 0c                sub    $0xc,%esp
 80b030a:       8d 87 ec d6 fc ff       lea    -0x32914(%edi),%eax
 80b0310:       89 fb                   mov    %edi,%ebx
 80b0312:       50                      push   %eax
 80b0313:       e8 48 cc f9 ff          call   804cf60 <__libc_fatal>
 80b0318:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80b031f:       00

080b0320 <__condvar_release_lock>:
 80b0320:       57                      push   %edi
 80b0321:       89 d1                   mov    %edx,%ecx
 80b0323:       e8 0b bb f9 ff          call   804be33 <__x86.get_pc_thunk.di>
 80b0328:       81 c7 cc 6c 03 00       add    $0x36ccc,%edi
 80b032e:       56                      push   %esi
 80b032f:       53                      push   %ebx
 80b0330:       8d 58 20                lea    0x20(%eax),%ebx
 80b0333:       8b 40 20                mov    0x20(%eax),%eax
 80b0336:       89 c6                   mov    %eax,%esi
 80b0338:       89 c2                   mov    %eax,%edx
 80b033a:       83 e6 fc                and    $0xfffffffc,%esi
 80b033d:       f0 0f b1 33             lock cmpxchg %esi,(%ebx)
 80b0341:       75 f3                   jne    80b0336 <__condvar_release_lock+0x16>
 80b0343:       83 e2 03                and    $0x3,%edx
 80b0346:       83 fa 02                cmp    $0x2,%edx
 80b0349:       74 05                   je     80b0350 <__condvar_release_lock+0x30>
 80b034b:       5b                      pop    %ebx
 80b034c:       5e                      pop    %esi
 80b034d:       5f                      pop    %edi
 80b034e:       c3                      ret
 80b034f:       90                      nop
 80b0350:       80 f1 81                xor    $0x81,%cl
 80b0353:       b8 f0 00 00 00          mov    $0xf0,%eax
 80b0358:       ba 01 00 00 00          mov    $0x1,%edx
 80b035d:       31 f6                   xor    %esi,%esi
 80b035f:       65 ff 15 10 00 00 00    call   *%gs:0x10
 80b0366:       3d 00 f0 ff ff          cmp    $0xfffff000,%eax
 80b036b:       76 de                   jbe    80b034b <__condvar_release_lock+0x2b>
 80b036d:       83 c0 16                add    $0x16,%eax
 80b0370:       83 e0 f7                and    $0xfffffff7,%eax
 80b0373:       74 d6                   je     80b034b <__condvar_release_lock+0x2b>
 80b0375:       83 ec 0c                sub    $0xc,%esp
 80b0378:       8d 87 ec d6 fc ff       lea    -0x32914(%edi),%eax
 80b037e:       89 fb                   mov    %edi,%ebx
 80b0380:       50                      push   %eax
 80b0381:       e8 da cb f9 ff          call   804cf60 <__libc_fatal>
 80b0386:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80b038d:       00
 80b038e:       66 90                   xchg   %ax,%ax

080b0390 <__condvar_cancel_waiting>:
 80b0390:       55                      push   %ebp
 80b0391:       89 cd                   mov    %ecx,%ebp
 80b0393:       57                      push   %edi
 80b0394:       89 c7                   mov    %eax,%edi
 80b0396:       56                      push   %esi
 80b0397:       53                      push   %ebx
 80b0398:       e8 43 94 f9 ff          call   80497e0 <__x86.get_pc_thunk.bx>
 80b039d:       81 c3 57 6c 03 00       add    $0x36c57,%ebx
 80b03a3:       83 ec 1c                sub    $0x1c,%esp
 80b03a6:       8b 44 24 30             mov    0x30(%esp),%eax
 80b03aa:       89 14 24                mov    %edx,(%esp)
 80b03ad:       89 44 24 0c             mov    %eax,0xc(%esp)
 80b03b1:       8b 44 24 34             mov    0x34(%esp),%eax
 80b03b5:       89 5c 24 08             mov    %ebx,0x8(%esp)
 80b03b9:       8d 5f 20                lea    0x20(%edi),%ebx
 80b03bc:       89 44 24 04             mov    %eax,0x4(%esp)
 80b03c0:       8b 47 20                mov    0x20(%edi),%eax
 80b03c3:       a8 03                   test   $0x3,%al
 80b03c5:       0f 85 bd 00 00 00       jne    80b0488 <__condvar_cancel_waiting+0xf8>
 80b03cb:       89 c2                   mov    %eax,%edx
 80b03cd:       83 ca 01                or     $0x1,%edx
 80b03d0:       f0 0f b1 13             lock cmpxchg %edx,(%ebx)
 80b03d4:       75 ed                   jne    80b03c3 <__condvar_cancel_waiting+0x33>
 80b03d6:       83 ec 0c                sub    $0xc,%esp
 80b03d9:       8d 47 08                lea    0x8(%edi),%eax
 80b03dc:       50                      push   %eax
 80b03dd:       e8 1e 68 ff ff          call   80a6c00 <__atomic_wide_counter_load_relaxed>
 80b03e2:       83 c4 10                add    $0x10,%esp
 80b03e5:       0f ac d0 01             shrd   $0x1,%edx,%eax
 80b03e9:       d1 ea                   shr    $1,%edx
 80b03eb:       39 04 24                cmp    %eax,(%esp)
 80b03ee:       89 c1                   mov    %eax,%ecx
 80b03f0:       89 e8                   mov    %ebp,%eax
 80b03f2:       89 d3                   mov    %edx,%ebx
 80b03f4:       19 d0                   sbb    %edx,%eax
 80b03f6:       73 28                   jae    80b0420 <__condvar_cancel_waiting+0x90>
 80b03f8:       8b 54 24 04             mov    0x4(%esp),%edx
 80b03fc:       89 f8                   mov    %edi,%eax
 80b03fe:       e8 1d ff ff ff          call   80b0320 <__condvar_release_lock>
 80b0403:       83 ec 0c                sub    $0xc,%esp
 80b0406:       57                      push   %edi
 80b0407:       8b 5c 24 18             mov    0x18(%esp),%ebx
 80b040b:       e8 f0 16 00 00          call   80b1b00 <___pthread_cond_signal>
 80b0410:       83 c4 10                add    $0x10,%esp
 80b0413:       83 c4 1c                add    $0x1c,%esp
 80b0416:       5b                      pop    %ebx
 80b0417:       5e                      pop    %esi
 80b0418:       5f                      pop    %edi
 80b0419:       5d                      pop    %ebp
 80b041a:       c3                      ret
 80b041b:       2e 8d 74 26 00          lea    %cs:0x0(%esi,%eiz,1),%esi
 80b0420:       8b 77 20                mov    0x20(%edi),%esi
 80b0423:       8b 44 24 0c             mov    0xc(%esp),%eax
 80b0427:       c1 ee 02                shr    $0x2,%esi
 80b042a:       83 c0 04                add    $0x4,%eax
 80b042d:       01 f1                   add    %esi,%ecx
 80b042f:       8b 54 87 08             mov    0x8(%edi,%eax,4),%edx
 80b0433:       83 d3 00                adc    $0x0,%ebx
 80b0436:       39 0c 24                cmp    %ecx,(%esp)
 80b0439:       19 dd                   sbb    %ebx,%ebp
 80b043b:       73 23                   jae    80b0460 <__condvar_cancel_waiting+0xd0>
 80b043d:       85 d2                   test   %edx,%edx
 80b043f:       74 b7                   je     80b03f8 <__condvar_cancel_waiting+0x68>
 80b0441:       83 ea 01                sub    $0x1,%edx
 80b0444:       89 54 87 08             mov    %edx,0x8(%edi,%eax,4)
 80b0448:       8b 54 24 04             mov    0x4(%esp),%edx
 80b044c:       83 c4 1c                add    $0x1c,%esp
 80b044f:       89 f8                   mov    %edi,%eax
 80b0451:       5b                      pop    %ebx
 80b0452:       5e                      pop    %esi
 80b0453:       5f                      pop    %edi
 80b0454:       5d                      pop    %ebp
 80b0455:       e9 c6 fe ff ff          jmp    80b0320 <__condvar_release_lock>
 80b045a:       8d b6 00 00 00 00       lea    0x0(%esi),%esi
 80b0460:       81 fa 00 00 00 e0       cmp    $0xe0000000,%edx
 80b0466:       75 d9                   jne    80b0441 <__condvar_cancel_waiting+0xb1>
 80b0468:       8b 54 24 04             mov    0x4(%esp),%edx
 80b046c:       89 f8                   mov    %edi,%eax
 80b046e:       e8 ad fe ff ff          call   80b0320 <__condvar_release_lock>
 80b0473:       83 ec 0c                sub    $0xc,%esp
 80b0476:       57                      push   %edi
 80b0477:       8b 5c 24 18             mov    0x18(%esp),%ebx
 80b047b:       e8 80 f9 ff ff          call   80afe00 <___pthread_cond_broadcast>
 80b0480:       83 c4 10                add    $0x10,%esp
 80b0483:       eb 8e                   jmp    80b0413 <__condvar_cancel_waiting+0x83>
 80b0485:       8d 76 00                lea    0x0(%esi),%esi
 80b0488:       8b 4c 24 04             mov    0x4(%esp),%ecx
 80b048c:       80 f1 80                xor    $0x80,%cl
 80b048f:       eb 2f                   jmp    80b04c0 <__condvar_cancel_waiting+0x130>
 80b0491:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80b0498:       00
 80b0499:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80b04a0:       00
 80b04a1:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80b04a8:       00
 80b04a9:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80b04b0:       00
 80b04b1:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80b04b8:       00
 80b04b9:       8d b4 26 00 00 00 00    lea    0x0(%esi,%eiz,1),%esi
 80b04c0:       89 c2                   mov    %eax,%edx
 80b04c2:       83 e2 03                and    $0x3,%edx
 80b04c5:       83 fa 02                cmp    $0x2,%edx
 80b04c8:       74 23                   je     80b04ed <__condvar_cancel_waiting+0x15d>
 80b04ca:       89 c2                   mov    %eax,%edx
 80b04cc:       83 e2 fc                and    $0xfffffffc,%edx
 80b04cf:       83 ca 02                or     $0x2,%edx
 80b04d2:       f0 0f b1 13             lock cmpxchg %edx,(%ebx)
 80b04d6:       0f 94 c2                sete   %dl
 80b04d9:       89 c6                   mov    %eax,%esi
 80b04db:       0f b6 d2                movzbl %dl,%edx
 80b04de:       83 e6 03                and    $0x3,%esi
 80b04e1:       85 d2                   test   %edx,%edx
 80b04e3:       74 db                   je     80b04c0 <__condvar_cancel_waiting+0x130>
 80b04e5:       85 f6                   test   %esi,%esi
 80b04e7:       0f 84 e9 fe ff ff       je     80b03d6 <__condvar_cancel_waiting+0x46>
 80b04ed:       83 e0 fc                and    $0xfffffffc,%eax
 80b04f0:       31 f6                   xor    %esi,%esi
 80b04f2:       89 c2                   mov    %eax,%edx
 80b04f4:       b8 f0 00 00 00          mov    $0xf0,%eax
 80b04f9:       83 ca 02                or     $0x2,%edx
 80b04fc:       65 ff 15 10 00 00 00    call   *%gs:0x10
 80b0503:       3d 00 f0 ff ff          cmp    $0xfffff000,%eax
 80b0508:       77 06                   ja     80b0510 <__condvar_cancel_waiting+0x180>
 80b050a:       8b 03                   mov    (%ebx),%eax
 80b050c:       eb b2                   jmp    80b04c0 <__condvar_cancel_waiting+0x130>
 80b050e:       66 90                   xchg   %ax,%ax
 80b0510:       83 f8 f5                cmp    $0xfffffff5,%eax
 80b0513:       74 f5                   je     80b050a <__condvar_cancel_waiting+0x17a>
 80b0515:       83 f8 fc                cmp    $0xfffffffc,%eax
 80b0518:       74 f0                   je     80b050a <__condvar_cancel_waiting+0x17a>
 80b051a:       83 ec 0c                sub    $0xc,%esp
 80b051d:       8b 5c 24 14             mov    0x14(%esp),%ebx
 80b0521:       8d 83 ec d6 fc ff       lea    -0x32914(%ebx),%eax
 80b0527:       50                      push   %eax
 80b0528:       e8 33 ca f9 ff          call   804cf60 <__libc_fatal>
 80b052d:       8d 76 00                lea    0x0(%esi),%esi

080b0530 <__condvar_cleanup_waiting>:
 80b0530:       e8 0c 94 f9 ff          call   8049941 <__x86.get_pc_thunk.ax>
 80b0535:       05 bf 6a 03 00          add    $0x36abf,%eax
 80b053a:       55                      push   %ebp
 80b053b:       57                      push   %edi
 80b053c:       56                      push   %esi
 80b053d:       31 f6                   xor    %esi,%esi
 80b053f:       53                      push   %ebx
 80b0540:       83 ec 1c                sub    $0x1c,%esp
 80b0543:       8b 7c 24 30             mov    0x30(%esp),%edi
 80b0547:       89 44 24 0c             mov    %eax,0xc(%esp)
 80b054b:       8b 1f                   mov    (%edi),%ebx
 80b054d:       8b 6f 08                mov    0x8(%edi),%ebp
 80b0550:       8b 4f 10                mov    0x10(%edi),%ecx
 80b0553:       83 e3 01                and    $0x1,%ebx
 80b0556:       89 e8                   mov    %ebp,%eax
 80b0558:       89 da                   mov    %ebx,%edx
 80b055a:       e8 e1 fc ff ff          call   80b0240 <__condvar_dec_grefs>
 80b055f:       8b 57 04                mov    0x4(%edi),%edx
 80b0562:       8b 07                   mov    (%edi),%eax
 80b0564:       83 ec 08                sub    $0x8,%esp
 80b0567:       ff 77 10                push   0x10(%edi)
 80b056a:       53                      push   %ebx
 80b056b:       0f ac d0 01             shrd   $0x1,%edx,%eax
 80b056f:       d1 ea                   shr    $1,%edx
 80b0571:       8d 5c 9d 28             lea    0x28(%ebp,%ebx,4),%ebx
 80b0575:       89 d1                   mov    %edx,%ecx
 80b0577:       89 c2                   mov    %eax,%edx
 80b0579:       89 e8                   mov    %ebp,%eax
 80b057b:       e8 10 fe ff ff          call   80b0390 <__condvar_cancel_waiting>
 80b0580:       8b 4f 10                mov    0x10(%edi),%ecx
 80b0583:       b8 f0 00 00 00          mov    $0xf0,%eax
 80b0588:       ba 01 00 00 00          mov    $0x1,%edx
 80b058d:       80 f1 81                xor    $0x81,%cl
 80b0590:       65 ff 15 10 00 00 00    call   *%gs:0x10
 80b0597:       83 c4 10                add    $0x10,%esp
 80b059a:       3d 00 f0 ff ff          cmp    $0xfffff000,%eax
 80b059f:       77 1f                   ja     80b05c0 <__condvar_cleanup_waiting+0x90>
 80b05a1:       8b 57 10                mov    0x10(%edi),%edx
 80b05a4:       89 e8                   mov    %ebp,%eax
 80b05a6:       e8 05 fd ff ff          call   80b02b0 <__condvar_confirm_wakeup>
 80b05ab:       8b 47 0c                mov    0xc(%edi),%eax
 80b05ae:       89 44 24 30             mov    %eax,0x30(%esp)
 80b05b2:       83 c4 1c                add    $0x1c,%esp
 80b05b5:       5b                      pop    %ebx
 80b05b6:       5e                      pop    %esi
 80b05b7:       5f                      pop    %edi
 80b05b8:       5d                      pop    %ebp
 80b05b9:       e9 32 12 00 00          jmp    80b17f0 <__pthread_mutex_cond_lock>
 80b05be:       66 90                   xchg   %ax,%ax
 80b05c0:       83 c0 16                add    $0x16,%eax
 80b05c3:       83 e0 f7                and    $0xfffffff7,%eax
 80b05c6:       74 d9                   je     80b05a1 <__condvar_cleanup_waiting+0x71>
 80b05c8:       83 ec 0c                sub    $0xc,%esp
 80b05cb:       8b 5c 24 18             mov    0x18(%esp),%ebx
 80b05cf:       8d 83 ec d6 fc ff       lea    -0x32914(%ebx),%eax
 80b05d5:       50                      push   %eax
 80b05d6:       e8 85 c9 f9 ff          call   804cf60 <__libc_fatal>
 80b05db:       2e 8d 74 26 00          lea    %cs:0x0(%esi,%eiz,1),%esi

080b05e0 <___pthread_cond_clockwait64.part.0>:
 80b05e0:       55                      push   %ebp
 80b05e1:       89 d5                   mov    %edx,%ebp
 80b05e3:       57                      push   %edi
 80b05e4:       89 c7                   mov    %eax,%edi
 80b05e6:       56                      push   %esi
 80b05e7:       53                      push   %ebx
 80b05e8:       e8 f3 91 f9 ff          call   80497e0 <__x86.get_pc_thunk.bx>
 80b05ed:       81 c3 07 6a 03 00       add    $0x36a07,%ebx
 80b05f3:       81 ec 84 00 00 00       sub    $0x84,%esp
 80b05f9:       8b 84 24 98 00 00 00    mov    0x98(%esp),%eax
 80b0600:       89 54 24 30             mov    %edx,0x30(%esp)
 80b0604:       89 4c 24 38             mov    %ecx,0x38(%esp)
 80b0608:       89 44 24 3c             mov    %eax,0x3c(%esp)
 80b060c:       89 5c 24 34             mov    %ebx,0x34(%esp)
 80b0610:       65 a1 14 00 00 00       mov    %gs:0x14,%eax
 80b0616:       89 44 24 74             mov    %eax,0x74(%esp)
 80b061a:       31 c0                   xor    %eax,%eax
 80b061c:       6a 02                   push   $0x2
 80b061e:       57                      push   %edi
 80b061f:       e8 8c 65 ff ff          call   80a6bb0 <__atomic_wide_counter_fetch_add_relaxed>
 80b0624:       89 44 24 30             mov    %eax,0x30(%esp)
 80b0628:       89 c6                   mov    %eax,%esi
 80b062a:       0f ac d0 01             shrd   $0x1,%edx,%eax
 80b062e:       89 54 24 34             mov    %edx,0x34(%esp)
 80b0632:       83 e6 01                and    $0x1,%esi
 80b0635:       d1 ea                   shr    $1,%edx
 80b0637:       89 44 24 28             mov    %eax,0x28(%esp)
 80b063b:       b8 08 00 00 00          mov    $0x8,%eax
 80b0640:       89 74 24 20             mov    %esi,0x20(%esp)
 80b0644:       89 54 24 2c             mov    %edx,0x2c(%esp)
 80b0648:       f0 0f c1 47 24          lock xadd %eax,0x24(%edi)
 80b064d:       c1 e0 07                shl    $0x7,%eax
 80b0650:       0f b6 c0                movzbl %al,%eax
 80b0653:       89 44 24 18             mov    %eax,0x18(%esp)
 80b0657:       58                      pop    %eax
 80b0658:       5a                      pop    %edx
 80b0659:       6a 00                   push   $0x0
 80b065b:       55                      push   %ebp
 80b065c:       e8 bf 96 fc ff          call   8079d20 <__pthread_mutex_unlock_usercnt>
 80b0661:       83 c4 10                add    $0x10,%esp
 80b0664:       89 c5                   mov    %eax,%ebp
 80b0666:       85 c0                   test   %eax,%eax
 80b0668:       0f 85 62 02 00 00       jne    80b08d0 <___pthread_cond_clockwait64.part.0+0x2f0>
 80b066e:       8b 44 24 10             mov    0x10(%esp),%eax
 80b0672:       8d 34 85 00 00 00 00    lea    0x0(,%eax,4),%esi
 80b0679:       8d 44 37 28             lea    0x28(%edi,%esi,1),%eax
 80b067d:       89 44 24 0c             mov    %eax,0xc(%esp)
 80b0681:       8b 00                   mov    (%eax),%eax
 80b0683:       a8 01                   test   $0x1,%al
 80b0685:       75 46                   jne    80b06cd <___pthread_cond_clockwait64.part.0+0xed>
 80b0687:       85 c0                   test   %eax,%eax
 80b0689:       0f 84 61 01 00 00       je     80b07f0 <___pthread_cond_clockwait64.part.0+0x210>
 80b068f:       8b 4c 24 0c             mov    0xc(%esp),%ecx
 80b0693:       8d 50 fe                lea    -0x2(%eax),%edx
 80b0696:       f0 0f b1 11             lock cmpxchg %edx,(%ecx)
 80b069a:       0f 85 23 02 00 00       jne    80b08c3 <___pthread_cond_clockwait64.part.0+0x2e3>
 80b06a0:       83 ec 0c                sub    $0xc,%esp
 80b06a3:       8d 77 08                lea    0x8(%edi),%esi
 80b06a6:       56                      push   %esi
 80b06a7:       e8 54 65 ff ff          call   80a6c00 <__atomic_wide_counter_load_relaxed>
 80b06ac:       89 44 24 30             mov    %eax,0x30(%esp)
 80b06b0:       0f ac d0 01             shrd   $0x1,%edx,%eax
 80b06b4:       89 54 24 34             mov    %edx,0x34(%esp)
 80b06b8:       8b 4c 24 28             mov    0x28(%esp),%ecx
 80b06bc:       83 c4 10                add    $0x10,%esp
 80b06bf:       d1 ea                   shr    $1,%edx
 80b06c1:       8b 5c 24 1c             mov    0x1c(%esp),%ebx
 80b06c5:       39 c1                   cmp    %eax,%ecx
 80b06c7:       89 d9                   mov    %ebx,%ecx
 80b06c9:       19 d1                   sbb    %edx,%ecx
 80b06cb:       72 43                   jb     80b0710 <___pthread_cond_clockwait64.part.0+0x130>
 80b06cd:       8b 54 24 08             mov    0x8(%esp),%edx
 80b06d1:       89 f8                   mov    %edi,%eax
 80b06d3:       e8 d8 fb ff ff          call   80b02b0 <__condvar_confirm_wakeup>
 80b06d8:       83 ec 0c                sub    $0xc,%esp
 80b06db:       ff 74 24 34             push   0x34(%esp)
 80b06df:       e8 0c 11 00 00          call   80b17f0 <__pthread_mutex_cond_lock>
 80b06e4:       83 c4 10                add    $0x10,%esp
 80b06e7:       85 c0                   test   %eax,%eax
 80b06e9:       0f 45 e8                cmovne %eax,%ebp
 80b06ec:       8b 44 24 6c             mov    0x6c(%esp),%eax
 80b06f0:       65 2b 05 14 00 00 00    sub    %gs:0x14,%eax
 80b06f7:       0f 85 56 02 00 00       jne    80b0953 <___pthread_cond_clockwait64.part.0+0x373>
 80b06fd:       83 c4 7c                add    $0x7c,%esp
 80b0700:       89 e8                   mov    %ebp,%eax
 80b0702:       5b                      pop    %ebx
 80b0703:       5e                      pop    %esi
 80b0704:       5f                      pop    %edi
 80b0705:       5d                      pop    %ebp
 80b0706:       c3                      ret
 80b0707:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80b070e:       00
 80b070f:       90                      nop
 80b0710:       8b 44 24 20             mov    0x20(%esp),%eax
 80b0714:       8b 5c 24 10             mov    0x10(%esp),%ebx
 80b0718:       31 d2                   xor    %edx,%edx
 80b071a:       c7 44 24 1c 00 00 00    movl   $0x0,0x1c(%esp)
 80b0721:       00
 80b0722:       89 54 24 14             mov    %edx,0x14(%esp)
 80b0726:       8b 54 24 1c             mov    0x1c(%esp),%edx
 80b072a:       f7 d0                   not    %eax
 80b072c:       89 5c 24 18             mov    %ebx,0x18(%esp)
 80b0730:       83 e0 01                and    $0x1,%eax
 80b0733:       89 44 24 10             mov    %eax,0x10(%esp)
 80b0737:       8b 44 24 18             mov    0x18(%esp),%eax
 80b073b:       8b 4c 24 10             mov    0x10(%esp),%ecx
 80b073f:       31 c8                   xor    %ecx,%eax
 80b0741:       8b 4c 24 14             mov    0x14(%esp),%ecx
 80b0745:       31 ca                   xor    %ecx,%edx
 80b0747:       09 d0                   or     %edx,%eax
 80b0749:       75 82                   jne    80b06cd <___pthread_cond_clockwait64.part.0+0xed>
 80b074b:       8b 44 24 0c             mov    0xc(%esp),%eax
 80b074f:       89 fb                   mov    %edi,%ebx
 80b0751:       89 f7                   mov    %esi,%edi
 80b0753:       8b 00                   mov    (%eax),%eax
 80b0755:       89 c6                   mov    %eax,%esi
 80b0757:       83 ec 0c                sub    $0xc,%esp
 80b075a:       57                      push   %edi
 80b075b:       e8 a0 64 ff ff          call   80a6c00 <__atomic_wide_counter_load_relaxed>
 80b0760:       8b 4c 24 30             mov    0x30(%esp),%ecx
 80b0764:       83 c4 10                add    $0x10,%esp
 80b0767:       31 c8                   xor    %ecx,%eax
 80b0769:       8b 4c 24 24             mov    0x24(%esp),%ecx
 80b076d:       31 ca                   xor    %ecx,%edx
 80b076f:       09 d0                   or     %edx,%eax
 80b0771:       0f 85 d5 01 00 00       jne    80b094c <___pthread_cond_clockwait64.part.0+0x36c>
 80b0777:       f7 c6 01 00 00 00       test   $0x1,%esi
 80b077d:       75 11                   jne    80b0790 <___pthread_cond_clockwait64.part.0+0x1b0>
 80b077f:       8b 4c 24 0c             mov    0xc(%esp),%ecx
 80b0783:       8d 56 02                lea    0x2(%esi),%edx
 80b0786:       89 f0                   mov    %esi,%eax
 80b0788:       f0 0f b1 11             lock cmpxchg %edx,(%ecx)
 80b078c:       89 c6                   mov    %eax,%esi
 80b078e:       75 c7                   jne    80b0757 <___pthread_cond_clockwait64.part.0+0x177>
 80b0790:       8b 4c 24 08             mov    0x8(%esp),%ecx
 80b0794:       89 df                   mov    %ebx,%edi
 80b0796:       b8 f0 00 00 00          mov    $0xf0,%eax
 80b079b:       8b 5c 24 0c             mov    0xc(%esp),%ebx
 80b079f:       ba 01 00 00 00          mov    $0x1,%edx
 80b07a4:       31 f6                   xor    %esi,%esi
 80b07a6:       80 f1 81                xor    $0x81,%cl
 80b07a9:       65 ff 15 10 00 00 00    call   *%gs:0x10
 80b07b0:       3d 00 f0 ff ff          cmp    $0xfffff000,%eax
 80b07b5:       0f 86 12 ff ff ff       jbe    80b06cd <___pthread_cond_clockwait64.part.0+0xed>
 80b07bb:       83 c0 16                add    $0x16,%eax
 80b07be:       83 e0 f7                and    $0xfffffff7,%eax
 80b07c1:       0f 84 06 ff ff ff       je     80b06cd <___pthread_cond_clockwait64.part.0+0xed>
 80b07c7:       8b 44 24 6c             mov    0x6c(%esp),%eax
 80b07cb:       65 2b 05 14 00 00 00    sub    %gs:0x14,%eax
 80b07d2:       0f 85 7b 01 00 00       jne    80b0953 <___pthread_cond_clockwait64.part.0+0x373>
 80b07d8:       83 ec 0c                sub    $0xc,%esp
 80b07db:       8b 5c 24 38             mov    0x38(%esp),%ebx
 80b07df:       8d 83 ec d6 fc ff       lea    -0x32914(%ebx),%eax
 80b07e5:       50                      push   %eax
 80b07e6:       e8 75 c7 f9 ff          call   804cf60 <__libc_fatal>
 80b07eb:       2e 8d 74 26 00          lea    %cs:0x0(%esi,%eiz,1),%esi
 80b07f0:       f0 83 44 37 10 02       lock addl $0x2,0x10(%edi,%esi,1)
 80b07f6:       8b 44 24 0c             mov    0xc(%esp),%eax
 80b07fa:       8b 00                   mov    (%eax),%eax
 80b07fc:       a8 01                   test   $0x1,%al
 80b07fe:       0f 85 34 01 00 00       jne    80b0938 <___pthread_cond_clockwait64.part.0+0x358>
 80b0804:       83 ec 0c                sub    $0xc,%esp
 80b0807:       8d 47 08                lea    0x8(%edi),%eax
 80b080a:       50                      push   %eax
 80b080b:       e8 f0 63 ff ff          call   80a6c00 <__atomic_wide_counter_load_relaxed>
 80b0810:       8b 4c 24 28             mov    0x28(%esp),%ecx
 80b0814:       8b 5c 24 2c             mov    0x2c(%esp),%ebx
 80b0818:       83 c4 10                add    $0x10,%esp
 80b081b:       0f ac d0 01             shrd   $0x1,%edx,%eax
 80b081f:       d1 ea                   shr    $1,%edx
 80b0821:       39 c1                   cmp    %eax,%ecx
 80b0823:       89 d9                   mov    %ebx,%ecx
 80b0825:       19 d1                   sbb    %edx,%ecx
 80b0827:       0f 82 0b 01 00 00       jb     80b0938 <___pthread_cond_clockwait64.part.0+0x358>
 80b082d:       8b 44 24 20             mov    0x20(%esp),%eax
 80b0831:       8b 54 24 24             mov    0x24(%esp),%edx
 80b0835:       89 7c 24 60             mov    %edi,0x60(%esp)
 80b0839:       83 ec 04                sub    $0x4,%esp
 80b083c:       89 44 24 5c             mov    %eax,0x5c(%esp)
 80b0840:       8b 44 24 2c             mov    0x2c(%esp),%eax
 80b0844:       89 54 24 60             mov    %edx,0x60(%esp)
 80b0848:       89 44 24 68             mov    %eax,0x68(%esp)
 80b084c:       8b 44 24 0c             mov    0xc(%esp),%eax
 80b0850:       89 44 24 6c             mov    %eax,0x6c(%esp)
 80b0854:       8d 44 24 5c             lea    0x5c(%esp),%eax
 80b0858:       50                      push   %eax
 80b0859:       8b 5c 24 34             mov    0x34(%esp),%ebx
 80b085d:       8d 83 3c 95 fc ff       lea    -0x36ac4(%ebx),%eax
 80b0863:       50                      push   %eax
 80b0864:       8d 54 24 54             lea    0x54(%esp),%edx
 80b0868:       52                      push   %edx
 80b0869:       89 54 24 4c             mov    %edx,0x4c(%esp)
 80b086d:       e8 9e a2 fe ff          call   809ab10 <__pthread_cleanup_push>
 80b0872:       58                      pop    %eax
 80b0873:       ff 74 24 14             push   0x14(%esp)
 80b0877:       ff 74 24 44             push   0x44(%esp)
 80b087b:       ff 74 24 44             push   0x44(%esp)
 80b087f:       6a 00                   push   $0x0
 80b0881:       ff 74 24 28             push   0x28(%esp)
 80b0885:       e8 e6 a6 fe ff          call   809af70 <__futex_abstimed_wait_cancelable64>
 80b088a:       89 44 24 58             mov    %eax,0x58(%esp)
 80b088e:       83 c4 18                add    $0x18,%esp
 80b0891:       6a 00                   push   $0x0
 80b0893:       8b 54 24 48             mov    0x48(%esp),%edx
 80b0897:       52                      push   %edx
 80b0898:       e8 a3 a2 fe ff          call   809ab40 <__pthread_cleanup_pop>
 80b089d:       8b 44 24 48             mov    0x48(%esp),%eax
 80b08a1:       83 c4 10                add    $0x10,%esp
 80b08a4:       83 f8 6e                cmp    $0x6e,%eax
 80b08a7:       74 57                   je     80b0900 <___pthread_cond_clockwait64.part.0+0x320>
 80b08a9:       83 f8 4b                cmp    $0x4b,%eax
 80b08ac:       74 52                   je     80b0900 <___pthread_cond_clockwait64.part.0+0x320>
 80b08ae:       8b 4c 24 08             mov    0x8(%esp),%ecx
 80b08b2:       8b 54 24 10             mov    0x10(%esp),%edx
 80b08b6:       89 f8                   mov    %edi,%eax
 80b08b8:       e8 83 f9 ff ff          call   80b0240 <__condvar_dec_grefs>
 80b08bd:       8b 44 24 0c             mov    0xc(%esp),%eax
 80b08c1:       8b 00                   mov    (%eax),%eax
 80b08c3:       a8 01                   test   $0x1,%al
 80b08c5:       0f 84 bc fd ff ff       je     80b0687 <___pthread_cond_clockwait64.part.0+0xa7>
 80b08cb:       e9 fd fd ff ff          jmp    80b06cd <___pthread_cond_clockwait64.part.0+0xed>
 80b08d0:       83 ec 08                sub    $0x8,%esp
 80b08d3:       89 f8                   mov    %edi,%eax
 80b08d5:       8b 74 24 10             mov    0x10(%esp),%esi
 80b08d9:       56                      push   %esi
 80b08da:       ff 74 24 1c             push   0x1c(%esp)
 80b08de:       8b 54 24 28             mov    0x28(%esp),%edx
 80b08e2:       8b 4c 24 2c             mov    0x2c(%esp),%ecx
 80b08e6:       e8 a5 fa ff ff          call   80b0390 <__condvar_cancel_waiting>
 80b08eb:       83 c4 10                add    $0x10,%esp
 80b08ee:       89 f2                   mov    %esi,%edx
 80b08f0:       89 f8                   mov    %edi,%eax
 80b08f2:       e8 b9 f9 ff ff          call   80b02b0 <__condvar_confirm_wakeup>
 80b08f7:       e9 f0 fd ff ff          jmp    80b06ec <___pthread_cond_clockwait64.part.0+0x10c>
 80b08fc:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi
 80b0900:       8b 5c 24 08             mov    0x8(%esp),%ebx
 80b0904:       8b 74 24 10             mov    0x10(%esp),%esi
 80b0908:       89 c5                   mov    %eax,%ebp
 80b090a:       89 f8                   mov    %edi,%eax
 80b090c:       89 d9                   mov    %ebx,%ecx
 80b090e:       89 f2                   mov    %esi,%edx
 80b0910:       e8 2b f9 ff ff          call   80b0240 <__condvar_dec_grefs>
 80b0915:       83 ec 08                sub    $0x8,%esp
 80b0918:       89 f8                   mov    %edi,%eax
 80b091a:       53                      push   %ebx
 80b091b:       56                      push   %esi
 80b091c:       8b 54 24 28             mov    0x28(%esp),%edx
 80b0920:       8b 4c 24 2c             mov    0x2c(%esp),%ecx
 80b0924:       e8 67 fa ff ff          call   80b0390 <__condvar_cancel_waiting>
 80b0929:       83 c4 10                add    $0x10,%esp
 80b092c:       e9 9c fd ff ff          jmp    80b06cd <___pthread_cond_clockwait64.part.0+0xed>
 80b0931:       8d b4 26 00 00 00 00    lea    0x0(%esi,%eiz,1),%esi
 80b0938:       8b 4c 24 08             mov    0x8(%esp),%ecx
 80b093c:       8b 54 24 10             mov    0x10(%esp),%edx
 80b0940:       89 f8                   mov    %edi,%eax
 80b0942:       e8 f9 f8 ff ff          call   80b0240 <__condvar_dec_grefs>
 80b0947:       e9 81 fd ff ff          jmp    80b06cd <___pthread_cond_clockwait64.part.0+0xed>
 80b094c:       89 df                   mov    %ebx,%edi
 80b094e:       e9 7a fd ff ff          jmp    80b06cd <___pthread_cond_clockwait64.part.0+0xed>
 80b0953:       e8 b8 c0 fa ff          call   805ca10 <__stack_chk_fail>
 80b0958:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80b095f:       00

080b0960 <___pthread_cond_wait>:
 80b0960:       55                      push   %ebp
 80b0961:       57                      push   %edi
 80b0962:       56                      push   %esi
 80b0963:       53                      push   %ebx
 80b0964:       e8 77 8e f9 ff          call   80497e0 <__x86.get_pc_thunk.bx>
 80b0969:       81 c3 8b 66 03 00       add    $0x3668b,%ebx
 80b096f:       83 ec 74                sub    $0x74,%esp
 80b0972:       8b b4 24 8c 00 00 00    mov    0x8c(%esp),%esi
 80b0979:       8b bc 24 88 00 00 00    mov    0x88(%esp),%edi
 80b0980:       89 74 24 28             mov    %esi,0x28(%esp)
 80b0984:       89 5c 24 2c             mov    %ebx,0x2c(%esp)
 80b0988:       65 a1 14 00 00 00       mov    %gs:0x14,%eax
 80b098e:       89 44 24 64             mov    %eax,0x64(%esp)
 80b0992:       31 c0                   xor    %eax,%eax
 80b0994:       6a 02                   push   $0x2
 80b0996:       57                      push   %edi
 80b0997:       e8 14 62 ff ff          call   80a6bb0 <__atomic_wide_counter_fetch_add_relaxed>
 80b099c:       89 44 24 28             mov    %eax,0x28(%esp)
 80b09a0:       89 c1                   mov    %eax,%ecx
 80b09a2:       0f ac d0 01             shrd   $0x1,%edx,%eax
 80b09a6:       89 54 24 2c             mov    %edx,0x2c(%esp)
 80b09aa:       83 e1 01                and    $0x1,%ecx
 80b09ad:       d1 ea                   shr    $1,%edx
 80b09af:       89 44 24 20             mov    %eax,0x20(%esp)
 80b09b3:       b8 08 00 00 00          mov    $0x8,%eax
 80b09b8:       89 4c 24 18             mov    %ecx,0x18(%esp)
 80b09bc:       89 54 24 24             mov    %edx,0x24(%esp)
 80b09c0:       f0 0f c1 47 24          lock xadd %eax,0x24(%edi)
 80b09c5:       c1 e0 07                shl    $0x7,%eax
 80b09c8:       0f b6 c0                movzbl %al,%eax
 80b09cb:       89 44 24 10             mov    %eax,0x10(%esp)
 80b09cf:       58                      pop    %eax
 80b09d0:       5a                      pop    %edx
 80b09d1:       6a 00                   push   $0x0
 80b09d3:       56                      push   %esi
 80b09d4:       e8 47 93 fc ff          call   8079d20 <__pthread_mutex_unlock_usercnt>
 80b09d9:       83 c4 10                add    $0x10,%esp
 80b09dc:       89 c5                   mov    %eax,%ebp
 80b09de:       85 c0                   test   %eax,%eax
 80b09e0:       0f 85 5a 02 00 00       jne    80b0c40 <___pthread_cond_wait+0x2e0>
 80b09e6:       8b 44 24 08             mov    0x8(%esp),%eax
 80b09ea:       8d 34 85 00 00 00 00    lea    0x0(,%eax,4),%esi
 80b09f1:       8d 44 37 28             lea    0x28(%edi,%esi,1),%eax
 80b09f5:       89 44 24 04             mov    %eax,0x4(%esp)
 80b09f9:       8b 00                   mov    (%eax),%eax
 80b09fb:       a8 01                   test   $0x1,%al
 80b09fd:       75 46                   jne    80b0a45 <___pthread_cond_wait+0xe5>
 80b09ff:       85 c0                   test   %eax,%eax
 80b0a01:       0f 84 59 01 00 00       je     80b0b60 <___pthread_cond_wait+0x200>
 80b0a07:       8b 4c 24 04             mov    0x4(%esp),%ecx
 80b0a0b:       8d 50 fe                lea    -0x2(%eax),%edx
 80b0a0e:       f0 0f b1 11             lock cmpxchg %edx,(%ecx)
 80b0a12:       0f 85 16 02 00 00       jne    80b0c2e <___pthread_cond_wait+0x2ce>
 80b0a18:       83 ec 0c                sub    $0xc,%esp
 80b0a1b:       8d 77 08                lea    0x8(%edi),%esi
 80b0a1e:       56                      push   %esi
 80b0a1f:       e8 dc 61 ff ff          call   80a6c00 <__atomic_wide_counter_load_relaxed>
 80b0a24:       89 44 24 28             mov    %eax,0x28(%esp)
 80b0a28:       0f ac d0 01             shrd   $0x1,%edx,%eax
 80b0a2c:       89 54 24 2c             mov    %edx,0x2c(%esp)
 80b0a30:       8b 4c 24 20             mov    0x20(%esp),%ecx
 80b0a34:       83 c4 10                add    $0x10,%esp
 80b0a37:       d1 ea                   shr    $1,%edx
 80b0a39:       8b 5c 24 14             mov    0x14(%esp),%ebx
 80b0a3d:       39 c1                   cmp    %eax,%ecx
 80b0a3f:       89 d9                   mov    %ebx,%ecx
 80b0a41:       19 d1                   sbb    %edx,%ecx
 80b0a43:       72 3b                   jb     80b0a80 <___pthread_cond_wait+0x120>
 80b0a45:       8b 14 24                mov    (%esp),%edx
 80b0a48:       89 f8                   mov    %edi,%eax
 80b0a4a:       e8 61 f8 ff ff          call   80b02b0 <__condvar_confirm_wakeup>
 80b0a4f:       83 ec 0c                sub    $0xc,%esp
 80b0a52:       ff 74 24 2c             push   0x2c(%esp)
 80b0a56:       e8 95 0d 00 00          call   80b17f0 <__pthread_mutex_cond_lock>
 80b0a5b:       83 c4 10                add    $0x10,%esp
 80b0a5e:       85 c0                   test   %eax,%eax
 80b0a60:       0f 45 e8                cmovne %eax,%ebp
 80b0a63:       8b 44 24 5c             mov    0x5c(%esp),%eax
 80b0a67:       65 2b 05 14 00 00 00    sub    %gs:0x14,%eax
 80b0a6e:       0f 85 46 02 00 00       jne    80b0cba <___pthread_cond_wait+0x35a>
 80b0a74:       83 c4 6c                add    $0x6c,%esp
 80b0a77:       89 e8                   mov    %ebp,%eax
 80b0a79:       5b                      pop    %ebx
 80b0a7a:       5e                      pop    %esi
 80b0a7b:       5f                      pop    %edi
 80b0a7c:       5d                      pop    %ebp
 80b0a7d:       c3                      ret
 80b0a7e:       66 90                   xchg   %ax,%ax
 80b0a80:       8b 44 24 18             mov    0x18(%esp),%eax
 80b0a84:       8b 5c 24 08             mov    0x8(%esp),%ebx
 80b0a88:       31 d2                   xor    %edx,%edx
 80b0a8a:       c7 44 24 14 00 00 00    movl   $0x0,0x14(%esp)
 80b0a91:       00
 80b0a92:       89 54 24 0c             mov    %edx,0xc(%esp)
 80b0a96:       8b 54 24 14             mov    0x14(%esp),%edx
 80b0a9a:       f7 d0                   not    %eax
 80b0a9c:       89 5c 24 10             mov    %ebx,0x10(%esp)
 80b0aa0:       83 e0 01                and    $0x1,%eax
 80b0aa3:       89 44 24 08             mov    %eax,0x8(%esp)
 80b0aa7:       8b 44 24 10             mov    0x10(%esp),%eax
 80b0aab:       8b 4c 24 08             mov    0x8(%esp),%ecx
 80b0aaf:       31 c8                   xor    %ecx,%eax
 80b0ab1:       8b 4c 24 0c             mov    0xc(%esp),%ecx
 80b0ab5:       31 ca                   xor    %ecx,%edx
 80b0ab7:       09 d0                   or     %edx,%eax
 80b0ab9:       75 8a                   jne    80b0a45 <___pthread_cond_wait+0xe5>
 80b0abb:       8b 44 24 04             mov    0x4(%esp),%eax
 80b0abf:       89 fb                   mov    %edi,%ebx
 80b0ac1:       89 f7                   mov    %esi,%edi
 80b0ac3:       8b 00                   mov    (%eax),%eax
 80b0ac5:       89 c6                   mov    %eax,%esi
 80b0ac7:       83 ec 0c                sub    $0xc,%esp
 80b0aca:       57                      push   %edi
 80b0acb:       e8 30 61 ff ff          call   80a6c00 <__atomic_wide_counter_load_relaxed>
 80b0ad0:       8b 4c 24 28             mov    0x28(%esp),%ecx
 80b0ad4:       83 c4 10                add    $0x10,%esp
 80b0ad7:       31 c8                   xor    %ecx,%eax
 80b0ad9:       8b 4c 24 1c             mov    0x1c(%esp),%ecx
 80b0add:       31 ca                   xor    %ecx,%edx
 80b0adf:       09 d0                   or     %edx,%eax
 80b0ae1:       0f 85 cc 01 00 00       jne    80b0cb3 <___pthread_cond_wait+0x353>
 80b0ae7:       f7 c6 01 00 00 00       test   $0x1,%esi
 80b0aed:       75 11                   jne    80b0b00 <___pthread_cond_wait+0x1a0>
 80b0aef:       8b 4c 24 04             mov    0x4(%esp),%ecx
 80b0af3:       8d 56 02                lea    0x2(%esi),%edx
 80b0af6:       89 f0                   mov    %esi,%eax
 80b0af8:       f0 0f b1 11             lock cmpxchg %edx,(%ecx)
 80b0afc:       89 c6                   mov    %eax,%esi
 80b0afe:       75 c7                   jne    80b0ac7 <___pthread_cond_wait+0x167>
 80b0b00:       8b 0c 24                mov    (%esp),%ecx
 80b0b03:       89 df                   mov    %ebx,%edi
 80b0b05:       b8 f0 00 00 00          mov    $0xf0,%eax
 80b0b0a:       8b 5c 24 04             mov    0x4(%esp),%ebx
 80b0b0e:       ba 01 00 00 00          mov    $0x1,%edx
 80b0b13:       31 f6                   xor    %esi,%esi
 80b0b15:       80 f1 81                xor    $0x81,%cl
 80b0b18:       65 ff 15 10 00 00 00    call   *%gs:0x10
 80b0b1f:       3d 00 f0 ff ff          cmp    $0xfffff000,%eax
 80b0b24:       0f 86 1b ff ff ff       jbe    80b0a45 <___pthread_cond_wait+0xe5>
 80b0b2a:       83 c0 16                add    $0x16,%eax
 80b0b2d:       83 e0 f7                and    $0xfffffff7,%eax
 80b0b30:       0f 84 0f ff ff ff       je     80b0a45 <___pthread_cond_wait+0xe5>
 80b0b36:       8b 44 24 5c             mov    0x5c(%esp),%eax
 80b0b3a:       65 2b 05 14 00 00 00    sub    %gs:0x14,%eax
 80b0b41:       0f 85 73 01 00 00       jne    80b0cba <___pthread_cond_wait+0x35a>
 80b0b47:       83 ec 0c                sub    $0xc,%esp
 80b0b4a:       8b 5c 24 30             mov    0x30(%esp),%ebx
 80b0b4e:       8d 83 ec d6 fc ff       lea    -0x32914(%ebx),%eax
 80b0b54:       50                      push   %eax
 80b0b55:       e8 06 c4 f9 ff          call   804cf60 <__libc_fatal>
 80b0b5a:       8d b6 00 00 00 00       lea    0x0(%esi),%esi
 80b0b60:       f0 83 44 37 10 02       lock addl $0x2,0x10(%edi,%esi,1)
 80b0b66:       8b 44 24 04             mov    0x4(%esp),%eax
 80b0b6a:       8b 00                   mov    (%eax),%eax
 80b0b6c:       a8 01                   test   $0x1,%al
 80b0b6e:       0f 85 2c 01 00 00       jne    80b0ca0 <___pthread_cond_wait+0x340>
 80b0b74:       83 ec 0c                sub    $0xc,%esp
 80b0b77:       8d 47 08                lea    0x8(%edi),%eax
 80b0b7a:       50                      push   %eax
 80b0b7b:       e8 80 60 ff ff          call   80a6c00 <__atomic_wide_counter_load_relaxed>
 80b0b80:       8b 4c 24 20             mov    0x20(%esp),%ecx
 80b0b84:       8b 5c 24 24             mov    0x24(%esp),%ebx
 80b0b88:       83 c4 10                add    $0x10,%esp
 80b0b8b:       0f ac d0 01             shrd   $0x1,%edx,%eax
 80b0b8f:       d1 ea                   shr    $1,%edx
 80b0b91:       39 c1                   cmp    %eax,%ecx
 80b0b93:       89 d9                   mov    %ebx,%ecx
 80b0b95:       19 d1                   sbb    %edx,%ecx
 80b0b97:       0f 82 03 01 00 00       jb     80b0ca0 <___pthread_cond_wait+0x340>
 80b0b9d:       8b 44 24 18             mov    0x18(%esp),%eax
 80b0ba1:       8b 54 24 1c             mov    0x1c(%esp),%edx
 80b0ba5:       89 7c 24 50             mov    %edi,0x50(%esp)
 80b0ba9:       83 ec 04                sub    $0x4,%esp
 80b0bac:       89 44 24 4c             mov    %eax,0x4c(%esp)
 80b0bb0:       8b 44 24 24             mov    0x24(%esp),%eax
 80b0bb4:       89 54 24 50             mov    %edx,0x50(%esp)
 80b0bb8:       89 44 24 58             mov    %eax,0x58(%esp)
 80b0bbc:       8b 44 24 04             mov    0x4(%esp),%eax
 80b0bc0:       89 44 24 5c             mov    %eax,0x5c(%esp)
 80b0bc4:       8d 44 24 4c             lea    0x4c(%esp),%eax
 80b0bc8:       50                      push   %eax
 80b0bc9:       8b 5c 24 2c             mov    0x2c(%esp),%ebx
 80b0bcd:       8d 83 3c 95 fc ff       lea    -0x36ac4(%ebx),%eax
 80b0bd3:       50                      push   %eax
 80b0bd4:       8d 54 24 44             lea    0x44(%esp),%edx
 80b0bd8:       52                      push   %edx
 80b0bd9:       89 54 24 3c             mov    %edx,0x3c(%esp)
 80b0bdd:       e8 2e 9f fe ff          call   809ab10 <__pthread_cleanup_push>
 80b0be2:       58                      pop    %eax
 80b0be3:       ff 74 24 0c             push   0xc(%esp)
 80b0be7:       6a 00                   push   $0x0
 80b0be9:       6a 00                   push   $0x0
 80b0beb:       6a 00                   push   $0x0
 80b0bed:       ff 74 24 20             push   0x20(%esp)
 80b0bf1:       e8 7a a3 fe ff          call   809af70 <__futex_abstimed_wait_cancelable64>
 80b0bf6:       89 44 24 48             mov    %eax,0x48(%esp)
 80b0bfa:       83 c4 18                add    $0x18,%esp
 80b0bfd:       6a 00                   push   $0x0
 80b0bff:       8b 54 24 38             mov    0x38(%esp),%edx
 80b0c03:       52                      push   %edx
 80b0c04:       e8 37 9f fe ff          call   809ab40 <__pthread_cleanup_pop>
 80b0c09:       8b 44 24 38             mov    0x38(%esp),%eax
 80b0c0d:       83 c4 10                add    $0x10,%esp
 80b0c10:       83 f8 6e                cmp    $0x6e,%eax
 80b0c13:       74 5b                   je     80b0c70 <___pthread_cond_wait+0x310>
 80b0c15:       83 f8 4b                cmp    $0x4b,%eax
 80b0c18:       74 56                   je     80b0c70 <___pthread_cond_wait+0x310>
 80b0c1a:       8b 0c 24                mov    (%esp),%ecx
 80b0c1d:       8b 54 24 08             mov    0x8(%esp),%edx
 80b0c21:       89 f8                   mov    %edi,%eax
 80b0c23:       e8 18 f6 ff ff          call   80b0240 <__condvar_dec_grefs>
 80b0c28:       8b 44 24 04             mov    0x4(%esp),%eax
 80b0c2c:       8b 00                   mov    (%eax),%eax
 80b0c2e:       a8 01                   test   $0x1,%al
 80b0c30:       0f 84 c9 fd ff ff       je     80b09ff <___pthread_cond_wait+0x9f>
 80b0c36:       e9 0a fe ff ff          jmp    80b0a45 <___pthread_cond_wait+0xe5>
 80b0c3b:       2e 8d 74 26 00          lea    %cs:0x0(%esi,%eiz,1),%esi
 80b0c40:       83 ec 08                sub    $0x8,%esp
 80b0c43:       89 f8                   mov    %edi,%eax
 80b0c45:       8b 5c 24 08             mov    0x8(%esp),%ebx
 80b0c49:       53                      push   %ebx
 80b0c4a:       ff 74 24 14             push   0x14(%esp)
 80b0c4e:       8b 54 24 20             mov    0x20(%esp),%edx
 80b0c52:       8b 4c 24 24             mov    0x24(%esp),%ecx
 80b0c56:       e8 35 f7 ff ff          call   80b0390 <__condvar_cancel_waiting>
 80b0c5b:       83 c4 10                add    $0x10,%esp
 80b0c5e:       89 da                   mov    %ebx,%edx
 80b0c60:       89 f8                   mov    %edi,%eax
 80b0c62:       e8 49 f6 ff ff          call   80b02b0 <__condvar_confirm_wakeup>
 80b0c67:       e9 f7 fd ff ff          jmp    80b0a63 <___pthread_cond_wait+0x103>
 80b0c6c:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi
 80b0c70:       8b 34 24                mov    (%esp),%esi
 80b0c73:       8b 5c 24 08             mov    0x8(%esp),%ebx
 80b0c77:       89 c5                   mov    %eax,%ebp
 80b0c79:       89 f8                   mov    %edi,%eax
 80b0c7b:       89 f1                   mov    %esi,%ecx
 80b0c7d:       89 da                   mov    %ebx,%edx
 80b0c7f:       e8 bc f5 ff ff          call   80b0240 <__condvar_dec_grefs>
 80b0c84:       83 ec 08                sub    $0x8,%esp
 80b0c87:       89 f8                   mov    %edi,%eax
 80b0c89:       56                      push   %esi
 80b0c8a:       53                      push   %ebx
 80b0c8b:       8b 54 24 20             mov    0x20(%esp),%edx
 80b0c8f:       8b 4c 24 24             mov    0x24(%esp),%ecx
 80b0c93:       e8 f8 f6 ff ff          call   80b0390 <__condvar_cancel_waiting>
 80b0c98:       83 c4 10                add    $0x10,%esp
 80b0c9b:       e9 a5 fd ff ff          jmp    80b0a45 <___pthread_cond_wait+0xe5>
 80b0ca0:       8b 0c 24                mov    (%esp),%ecx
 80b0ca3:       8b 54 24 08             mov    0x8(%esp),%edx
 80b0ca7:       89 f8                   mov    %edi,%eax
 80b0ca9:       e8 92 f5 ff ff          call   80b0240 <__condvar_dec_grefs>
 80b0cae:       e9 92 fd ff ff          jmp    80b0a45 <___pthread_cond_wait+0xe5>
 80b0cb3:       89 df                   mov    %ebx,%edi
 80b0cb5:       e9 8b fd ff ff          jmp    80b0a45 <___pthread_cond_wait+0xe5>
 80b0cba:       e8 51 bd fa ff          call   805ca10 <__stack_chk_fail>
 80b0cbf:       90                      nop

080b0cc0 <___pthread_cond_timedwait64>:
 80b0cc0:       55                      push   %ebp
 80b0cc1:       e8 d5 eb f9 ff          call   804f89b <__x86.get_pc_thunk.bp>
 80b0cc6:       81 c5 2e 63 03 00       add    $0x3632e,%ebp
 80b0ccc:       57                      push   %edi
 80b0ccd:       56                      push   %esi
 80b0cce:       53                      push   %ebx
 80b0ccf:       83 ec 7c                sub    $0x7c,%esp
 80b0cd2:       8b 84 24 94 00 00 00    mov    0x94(%esp),%eax
 80b0cd9:       89 6c 24 30             mov    %ebp,0x30(%esp)
 80b0cdd:       8b bc 24 90 00 00 00    mov    0x90(%esp),%edi
 80b0ce4:       89 44 24 04             mov    %eax,0x4(%esp)
 80b0ce8:       65 8b 35 14 00 00 00    mov    %gs:0x14,%esi
 80b0cef:       89 74 24 6c             mov    %esi,0x6c(%esp)
 80b0cf3:       8b b4 24 98 00 00 00    mov    0x98(%esp),%esi
 80b0cfa:       81 7e 08 ff c9 9a 3b    cmpl   $0x3b9ac9ff,0x8(%esi)
 80b0d01:       0f 87 c9 02 00 00       ja     80b0fd0 <___pthread_cond_timedwait64+0x310>
 80b0d07:       8b 47 24                mov    0x24(%edi),%eax
 80b0d0a:       83 ec 08                sub    $0x8,%esp
 80b0d0d:       8d 5f 24                lea    0x24(%edi),%ebx
 80b0d10:       d1 e8                   shr    $1,%eax
 80b0d12:       89 c1                   mov    %eax,%ecx
 80b0d14:       83 e1 01                and    $0x1,%ecx
 80b0d17:       89 4c 24 28             mov    %ecx,0x28(%esp)
 80b0d1b:       6a 02                   push   $0x2
 80b0d1d:       57                      push   %edi
 80b0d1e:       e8 8d 5e ff ff          call   80a6bb0 <__atomic_wide_counter_fetch_add_relaxed>
 80b0d23:       89 44 24 38             mov    %eax,0x38(%esp)
 80b0d27:       89 c1                   mov    %eax,%ecx
 80b0d29:       0f ac d0 01             shrd   $0x1,%edx,%eax
 80b0d2d:       89 54 24 3c             mov    %edx,0x3c(%esp)
 80b0d31:       83 e1 01                and    $0x1,%ecx
 80b0d34:       d1 ea                   shr    $1,%edx
 80b0d36:       89 4c 24 20             mov    %ecx,0x20(%esp)
 80b0d3a:       b9 08 00 00 00          mov    $0x8,%ecx
 80b0d3f:       89 44 24 28             mov    %eax,0x28(%esp)
 80b0d43:       89 54 24 2c             mov    %edx,0x2c(%esp)
 80b0d47:       f0 0f c1 0b             lock xadd %ecx,(%ebx)
 80b0d4b:       89 c8                   mov    %ecx,%eax
 80b0d4d:       c1 e0 07                shl    $0x7,%eax
 80b0d50:       0f b6 d8                movzbl %al,%ebx
 80b0d53:       89 5c 24 18             mov    %ebx,0x18(%esp)
 80b0d57:       5b                      pop    %ebx
 80b0d58:       89 eb                   mov    %ebp,%ebx
 80b0d5a:       58                      pop    %eax
 80b0d5b:       6a 00                   push   $0x0
 80b0d5d:       ff 74 24 10             push   0x10(%esp)
 80b0d61:       e8 ba 8f fc ff          call   8079d20 <__pthread_mutex_unlock_usercnt>
 80b0d66:       83 c4 10                add    $0x10,%esp
 80b0d69:       89 c5                   mov    %eax,%ebp
 80b0d6b:       85 c0                   test   %eax,%eax
 80b0d6d:       0f 85 6d 02 00 00       jne    80b0fe0 <___pthread_cond_timedwait64+0x320>
 80b0d73:       8b 44 24 10             mov    0x10(%esp),%eax
 80b0d77:       8d 0c 85 00 00 00 00    lea    0x0(,%eax,4),%ecx
 80b0d7e:       8d 44 0f 28             lea    0x28(%edi,%ecx,1),%eax
 80b0d82:       89 44 24 0c             mov    %eax,0xc(%esp)
 80b0d86:       8b 00                   mov    (%eax),%eax
 80b0d88:       a8 01                   test   $0x1,%al
 80b0d8a:       75 4a                   jne    80b0dd6 <___pthread_cond_timedwait64+0x116>
 80b0d8c:       89 74 24 34             mov    %esi,0x34(%esp)
 80b0d90:       89 ce                   mov    %ecx,%esi
 80b0d92:       85 c0                   test   %eax,%eax
 80b0d94:       0f 84 56 01 00 00       je     80b0ef0 <___pthread_cond_timedwait64+0x230>
 80b0d9a:       8b 5c 24 0c             mov    0xc(%esp),%ebx
 80b0d9e:       8d 50 fe                lea    -0x2(%eax),%edx
 80b0da1:       f0 0f b1 13             lock cmpxchg %edx,(%ebx)
 80b0da5:       0f 85 18 02 00 00       jne    80b0fc3 <___pthread_cond_timedwait64+0x303>
 80b0dab:       83 ec 0c                sub    $0xc,%esp
 80b0dae:       8d 77 08                lea    0x8(%edi),%esi
 80b0db1:       56                      push   %esi
 80b0db2:       e8 49 5e ff ff          call   80a6c00 <__atomic_wide_counter_load_relaxed>
 80b0db7:       89 44 24 30             mov    %eax,0x30(%esp)
 80b0dbb:       0f ac d0 01             shrd   $0x1,%edx,%eax
 80b0dbf:       89 54 24 34             mov    %edx,0x34(%esp)
 80b0dc3:       8b 4c 24 28             mov    0x28(%esp),%ecx
 80b0dc7:       83 c4 10                add    $0x10,%esp
 80b0dca:       d1 ea                   shr    $1,%edx
 80b0dcc:       8b 5c 24 1c             mov    0x1c(%esp),%ebx
 80b0dd0:       39 c1                   cmp    %eax,%ecx
 80b0dd2:       19 d3                   sbb    %edx,%ebx
 80b0dd4:       72 3a                   jb     80b0e10 <___pthread_cond_timedwait64+0x150>
 80b0dd6:       8b 54 24 08             mov    0x8(%esp),%edx
 80b0dda:       89 f8                   mov    %edi,%eax
 80b0ddc:       e8 cf f4 ff ff          call   80b02b0 <__condvar_confirm_wakeup>
 80b0de1:       83 ec 0c                sub    $0xc,%esp
 80b0de4:       ff 74 24 10             push   0x10(%esp)
 80b0de8:       e8 03 0a 00 00          call   80b17f0 <__pthread_mutex_cond_lock>
 80b0ded:       83 c4 10                add    $0x10,%esp
 80b0df0:       85 c0                   test   %eax,%eax
 80b0df2:       0f 45 e8                cmovne %eax,%ebp
 80b0df5:       8b 44 24 6c             mov    0x6c(%esp),%eax
 80b0df9:       65 2b 05 14 00 00 00    sub    %gs:0x14,%eax
 80b0e00:       0f 85 5d 02 00 00       jne    80b1063 <___pthread_cond_timedwait64+0x3a3>
 80b0e06:       83 c4 7c                add    $0x7c,%esp
 80b0e09:       89 e8                   mov    %ebp,%eax
 80b0e0b:       5b                      pop    %ebx
 80b0e0c:       5e                      pop    %esi
 80b0e0d:       5f                      pop    %edi
 80b0e0e:       5d                      pop    %ebp
 80b0e0f:       c3                      ret
 80b0e10:       8b 44 24 20             mov    0x20(%esp),%eax
 80b0e14:       8b 5c 24 10             mov    0x10(%esp),%ebx
 80b0e18:       31 d2                   xor    %edx,%edx
 80b0e1a:       c7 44 24 1c 00 00 00    movl   $0x0,0x1c(%esp)
 80b0e21:       00
 80b0e22:       89 54 24 14             mov    %edx,0x14(%esp)
 80b0e26:       8b 54 24 1c             mov    0x1c(%esp),%edx
 80b0e2a:       f7 d0                   not    %eax
 80b0e2c:       89 5c 24 18             mov    %ebx,0x18(%esp)
 80b0e30:       8b 5c 24 14             mov    0x14(%esp),%ebx
 80b0e34:       83 e0 01                and    $0x1,%eax
 80b0e37:       89 44 24 10             mov    %eax,0x10(%esp)
 80b0e3b:       8b 44 24 18             mov    0x18(%esp),%eax
 80b0e3f:       31 da                   xor    %ebx,%edx
 80b0e41:       8b 4c 24 10             mov    0x10(%esp),%ecx
 80b0e45:       31 c8                   xor    %ecx,%eax
 80b0e47:       09 d0                   or     %edx,%eax
 80b0e49:       75 8b                   jne    80b0dd6 <___pthread_cond_timedwait64+0x116>
 80b0e4b:       8b 44 24 0c             mov    0xc(%esp),%eax
 80b0e4f:       89 fb                   mov    %edi,%ebx
 80b0e51:       89 f7                   mov    %esi,%edi
 80b0e53:       8b 00                   mov    (%eax),%eax
 80b0e55:       89 c6                   mov    %eax,%esi
 80b0e57:       83 ec 0c                sub    $0xc,%esp
 80b0e5a:       57                      push   %edi
 80b0e5b:       e8 a0 5d ff ff          call   80a6c00 <__atomic_wide_counter_load_relaxed>
 80b0e60:       8b 4c 24 30             mov    0x30(%esp),%ecx
 80b0e64:       83 c4 10                add    $0x10,%esp
 80b0e67:       31 c8                   xor    %ecx,%eax
 80b0e69:       8b 4c 24 24             mov    0x24(%esp),%ecx
 80b0e6d:       31 ca                   xor    %ecx,%edx
 80b0e6f:       09 d0                   or     %edx,%eax
 80b0e71:       0f 85 e5 01 00 00       jne    80b105c <___pthread_cond_timedwait64+0x39c>
 80b0e77:       f7 c6 01 00 00 00       test   $0x1,%esi
 80b0e7d:       75 11                   jne    80b0e90 <___pthread_cond_timedwait64+0x1d0>
 80b0e7f:       8b 4c 24 0c             mov    0xc(%esp),%ecx
 80b0e83:       8d 56 02                lea    0x2(%esi),%edx
 80b0e86:       89 f0                   mov    %esi,%eax
 80b0e88:       f0 0f b1 11             lock cmpxchg %edx,(%ecx)
 80b0e8c:       89 c6                   mov    %eax,%esi
 80b0e8e:       75 c7                   jne    80b0e57 <___pthread_cond_timedwait64+0x197>
 80b0e90:       8b 4c 24 08             mov    0x8(%esp),%ecx
 80b0e94:       89 df                   mov    %ebx,%edi
 80b0e96:       b8 f0 00 00 00          mov    $0xf0,%eax
 80b0e9b:       8b 5c 24 0c             mov    0xc(%esp),%ebx
 80b0e9f:       ba 01 00 00 00          mov    $0x1,%edx
 80b0ea4:       31 f6                   xor    %esi,%esi
 80b0ea6:       80 f1 81                xor    $0x81,%cl
 80b0ea9:       65 ff 15 10 00 00 00    call   *%gs:0x10
 80b0eb0:       3d 00 f0 ff ff          cmp    $0xfffff000,%eax
 80b0eb5:       0f 86 1b ff ff ff       jbe    80b0dd6 <___pthread_cond_timedwait64+0x116>
 80b0ebb:       83 c0 16                add    $0x16,%eax
 80b0ebe:       83 e0 f7                and    $0xfffffff7,%eax
 80b0ec1:       0f 84 0f ff ff ff       je     80b0dd6 <___pthread_cond_timedwait64+0x116>
 80b0ec7:       8b 44 24 6c             mov    0x6c(%esp),%eax
 80b0ecb:       65 2b 05 14 00 00 00    sub    %gs:0x14,%eax
 80b0ed2:       0f 85 8b 01 00 00       jne    80b1063 <___pthread_cond_timedwait64+0x3a3>
 80b0ed8:       83 ec 0c                sub    $0xc,%esp
 80b0edb:       8b 5c 24 3c             mov    0x3c(%esp),%ebx
 80b0edf:       8d 83 ec d6 fc ff       lea    -0x32914(%ebx),%eax
 80b0ee5:       50                      push   %eax
 80b0ee6:       e8 75 c0 f9 ff          call   804cf60 <__libc_fatal>
 80b0eeb:       2e 8d 74 26 00          lea    %cs:0x0(%esi,%eiz,1),%esi
 80b0ef0:       f0 83 44 37 10 02       lock addl $0x2,0x10(%edi,%esi,1)
 80b0ef6:       8b 44 24 0c             mov    0xc(%esp),%eax
 80b0efa:       8b 00                   mov    (%eax),%eax
 80b0efc:       a8 01                   test   $0x1,%al
 80b0efe:       0f 85 44 01 00 00       jne    80b1048 <___pthread_cond_timedwait64+0x388>
 80b0f04:       83 ec 0c                sub    $0xc,%esp
 80b0f07:       8d 47 08                lea    0x8(%edi),%eax
 80b0f0a:       50                      push   %eax
 80b0f0b:       e8 f0 5c ff ff          call   80a6c00 <__atomic_wide_counter_load_relaxed>
 80b0f10:       8b 4c 24 28             mov    0x28(%esp),%ecx
 80b0f14:       8b 5c 24 2c             mov    0x2c(%esp),%ebx
 80b0f18:       83 c4 10                add    $0x10,%esp
 80b0f1b:       0f ac d0 01             shrd   $0x1,%edx,%eax
 80b0f1f:       d1 ea                   shr    $1,%edx
 80b0f21:       39 c1                   cmp    %eax,%ecx
 80b0f23:       89 d9                   mov    %ebx,%ecx
 80b0f25:       19 d1                   sbb    %edx,%ecx
 80b0f27:       0f 82 1b 01 00 00       jb     80b1048 <___pthread_cond_timedwait64+0x388>
 80b0f2d:       8b 44 24 28             mov    0x28(%esp),%eax
 80b0f31:       8b 54 24 2c             mov    0x2c(%esp),%edx
 80b0f35:       89 7c 24 60             mov    %edi,0x60(%esp)
 80b0f39:       83 ec 04                sub    $0x4,%esp
 80b0f3c:       89 44 24 5c             mov    %eax,0x5c(%esp)
 80b0f40:       8b 44 24 08             mov    0x8(%esp),%eax
 80b0f44:       89 54 24 60             mov    %edx,0x60(%esp)
 80b0f48:       89 44 24 68             mov    %eax,0x68(%esp)
 80b0f4c:       8b 44 24 0c             mov    0xc(%esp),%eax
 80b0f50:       89 44 24 6c             mov    %eax,0x6c(%esp)
 80b0f54:       8d 44 24 5c             lea    0x5c(%esp),%eax
 80b0f58:       50                      push   %eax
 80b0f59:       8b 5c 24 38             mov    0x38(%esp),%ebx
 80b0f5d:       8d 83 3c 95 fc ff       lea    -0x36ac4(%ebx),%eax
 80b0f63:       50                      push   %eax
 80b0f64:       8d 54 24 54             lea    0x54(%esp),%edx
 80b0f68:       52                      push   %edx
 80b0f69:       89 54 24 4c             mov    %edx,0x4c(%esp)
 80b0f6d:       e8 9e 9b fe ff          call   809ab10 <__pthread_cleanup_push>
 80b0f72:       58                      pop    %eax
 80b0f73:       ff 74 24 14             push   0x14(%esp)
 80b0f77:       ff 74 24 44             push   0x44(%esp)
 80b0f7b:       ff 74 24 34             push   0x34(%esp)
 80b0f7f:       6a 00                   push   $0x0
 80b0f81:       ff 74 24 28             push   0x28(%esp)
 80b0f85:       e8 e6 9f fe ff          call   809af70 <__futex_abstimed_wait_cancelable64>
 80b0f8a:       89 44 24 58             mov    %eax,0x58(%esp)
 80b0f8e:       83 c4 18                add    $0x18,%esp
 80b0f91:       6a 00                   push   $0x0
 80b0f93:       8b 54 24 48             mov    0x48(%esp),%edx
 80b0f97:       52                      push   %edx
 80b0f98:       e8 a3 9b fe ff          call   809ab40 <__pthread_cleanup_pop>
 80b0f9d:       8b 44 24 48             mov    0x48(%esp),%eax
 80b0fa1:       83 c4 10                add    $0x10,%esp
 80b0fa4:       83 f8 6e                cmp    $0x6e,%eax
 80b0fa7:       74 67                   je     80b1010 <___pthread_cond_timedwait64+0x350>
 80b0fa9:       83 f8 4b                cmp    $0x4b,%eax
 80b0fac:       74 62                   je     80b1010 <___pthread_cond_timedwait64+0x350>
 80b0fae:       8b 4c 24 08             mov    0x8(%esp),%ecx
 80b0fb2:       8b 54 24 10             mov    0x10(%esp),%edx
 80b0fb6:       89 f8                   mov    %edi,%eax
 80b0fb8:       e8 83 f2 ff ff          call   80b0240 <__condvar_dec_grefs>
 80b0fbd:       8b 44 24 0c             mov    0xc(%esp),%eax
 80b0fc1:       8b 00                   mov    (%eax),%eax
 80b0fc3:       a8 01                   test   $0x1,%al
 80b0fc5:       0f 84 c7 fd ff ff       je     80b0d92 <___pthread_cond_timedwait64+0xd2>
 80b0fcb:       e9 06 fe ff ff          jmp    80b0dd6 <___pthread_cond_timedwait64+0x116>
 80b0fd0:       bd 16 00 00 00          mov    $0x16,%ebp
 80b0fd5:       e9 1b fe ff ff          jmp    80b0df5 <___pthread_cond_timedwait64+0x135>
 80b0fda:       8d b6 00 00 00 00       lea    0x0(%esi),%esi
 80b0fe0:       83 ec 08                sub    $0x8,%esp
 80b0fe3:       89 f8                   mov    %edi,%eax
 80b0fe5:       8b 74 24 10             mov    0x10(%esp),%esi
 80b0fe9:       56                      push   %esi
 80b0fea:       ff 74 24 1c             push   0x1c(%esp)
 80b0fee:       8b 54 24 28             mov    0x28(%esp),%edx
 80b0ff2:       8b 4c 24 2c             mov    0x2c(%esp),%ecx
 80b0ff6:       e8 95 f3 ff ff          call   80b0390 <__condvar_cancel_waiting>
 80b0ffb:       83 c4 10                add    $0x10,%esp
 80b0ffe:       89 f2                   mov    %esi,%edx
 80b1000:       89 f8                   mov    %edi,%eax
 80b1002:       e8 a9 f2 ff ff          call   80b02b0 <__condvar_confirm_wakeup>
 80b1007:       e9 e9 fd ff ff          jmp    80b0df5 <___pthread_cond_timedwait64+0x135>
 80b100c:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi
 80b1010:       8b 5c 24 08             mov    0x8(%esp),%ebx
 80b1014:       8b 74 24 10             mov    0x10(%esp),%esi
 80b1018:       89 c5                   mov    %eax,%ebp
 80b101a:       89 f8                   mov    %edi,%eax
 80b101c:       89 d9                   mov    %ebx,%ecx
 80b101e:       89 f2                   mov    %esi,%edx
 80b1020:       e8 1b f2 ff ff          call   80b0240 <__condvar_dec_grefs>
 80b1025:       83 ec 08                sub    $0x8,%esp
 80b1028:       89 f8                   mov    %edi,%eax
 80b102a:       53                      push   %ebx
 80b102b:       56                      push   %esi
 80b102c:       8b 54 24 28             mov    0x28(%esp),%edx
 80b1030:       8b 4c 24 2c             mov    0x2c(%esp),%ecx
 80b1034:       e8 57 f3 ff ff          call   80b0390 <__condvar_cancel_waiting>
 80b1039:       83 c4 10                add    $0x10,%esp
 80b103c:       e9 95 fd ff ff          jmp    80b0dd6 <___pthread_cond_timedwait64+0x116>
 80b1041:       8d b4 26 00 00 00 00    lea    0x0(%esi,%eiz,1),%esi
 80b1048:       8b 4c 24 08             mov    0x8(%esp),%ecx
 80b104c:       8b 54 24 10             mov    0x10(%esp),%edx
 80b1050:       89 f8                   mov    %edi,%eax
 80b1052:       e8 e9 f1 ff ff          call   80b0240 <__condvar_dec_grefs>
 80b1057:       e9 7a fd ff ff          jmp    80b0dd6 <___pthread_cond_timedwait64+0x116>
 80b105c:       89 df                   mov    %ebx,%edi
 80b105e:       e9 73 fd ff ff          jmp    80b0dd6 <___pthread_cond_timedwait64+0x116>
 80b1063:       e8 a8 b9 fa ff          call   805ca10 <__stack_chk_fail>
 80b1068:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80b106f:       00

080b1070 <___pthread_cond_timedwait>:
 80b1070:       83 ec 30                sub    $0x30,%esp
 80b1073:       65 a1 14 00 00 00       mov    %gs:0x14,%eax
 80b1079:       89 44 24 20             mov    %eax,0x20(%esp)
 80b107d:       8b 44 24 3c             mov    0x3c(%esp),%eax
 80b1081:       c7 44 24 1c 00 00 00    movl   $0x0,0x1c(%esp)
 80b1088:       00
 80b1089:       8b 50 04                mov    0x4(%eax),%edx
 80b108c:       8b 00                   mov    (%eax),%eax
 80b108e:       89 44 24 10             mov    %eax,0x10(%esp)
 80b1092:       c1 f8 1f                sar    $0x1f,%eax
 80b1095:       89 44 24 14             mov    %eax,0x14(%esp)
 80b1099:       8d 44 24 10             lea    0x10(%esp),%eax
 80b109d:       89 54 24 18             mov    %edx,0x18(%esp)
 80b10a1:       50                      push   %eax
 80b10a2:       ff 74 24 3c             push   0x3c(%esp)
 80b10a6:       ff 74 24 3c             push   0x3c(%esp)
 80b10aa:       e8 11 fc ff ff          call   80b0cc0 <___pthread_cond_timedwait64>
 80b10af:       83 c4 10                add    $0x10,%esp
 80b10b2:       8b 54 24 1c             mov    0x1c(%esp),%edx
 80b10b6:       65 2b 15 14 00 00 00    sub    %gs:0x14,%edx
 80b10bd:       75 04                   jne    80b10c3 <___pthread_cond_timedwait+0x53>
 80b10bf:       83 c4 2c                add    $0x2c,%esp
 80b10c2:       c3                      ret
 80b10c3:       e8 48 b9 fa ff          call   805ca10 <__stack_chk_fail>
 80b10c8:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80b10cf:       00

080b10d0 <___pthread_cond_clockwait64>:
 80b10d0:       53                      push   %ebx
 80b10d1:       8b 44 24 14             mov    0x14(%esp),%eax
 80b10d5:       8b 5c 24 08             mov    0x8(%esp),%ebx
 80b10d9:       8b 54 24 0c             mov    0xc(%esp),%edx
 80b10dd:       81 78 08 ff c9 9a 3b    cmpl   $0x3b9ac9ff,0x8(%eax)
 80b10e4:       8b 4c 24 10             mov    0x10(%esp),%ecx
 80b10e8:       77 16                   ja     80b1100 <___pthread_cond_clockwait64+0x30>
 80b10ea:       83 f9 01                cmp    $0x1,%ecx
 80b10ed:       77 11                   ja     80b1100 <___pthread_cond_clockwait64+0x30>
 80b10ef:       89 44 24 08             mov    %eax,0x8(%esp)
 80b10f3:       89 d8                   mov    %ebx,%eax
 80b10f5:       5b                      pop    %ebx
 80b10f6:       e9 e5 f4 ff ff          jmp    80b05e0 <___pthread_cond_clockwait64.part.0>
 80b10fb:       2e 8d 74 26 00          lea    %cs:0x0(%esi,%eiz,1),%esi
 80b1100:       b8 16 00 00 00          mov    $0x16,%eax
 80b1105:       5b                      pop    %ebx
 80b1106:       c3                      ret
 80b1107:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80b110e:       00
 80b110f:       90                      nop

080b1110 <___pthread_cond_clockwait>:
 80b1110:       57                      push   %edi
 80b1111:       56                      push   %esi
 80b1112:       53                      push   %ebx
 80b1113:       83 ec 20                sub    $0x20,%esp
 80b1116:       65 8b 15 14 00 00 00    mov    %gs:0x14,%edx
 80b111d:       89 54 24 1c             mov    %edx,0x1c(%esp)
 80b1121:       8b 54 24 3c             mov    0x3c(%esp),%edx
 80b1125:       8b 5c 24 30             mov    0x30(%esp),%ebx
 80b1129:       8b 74 24 34             mov    0x34(%esp),%esi
 80b112d:       c7 44 24 18 00 00 00    movl   $0x0,0x18(%esp)
 80b1134:       00
 80b1135:       8b 4c 24 38             mov    0x38(%esp),%ecx
 80b1139:       8b 42 04                mov    0x4(%edx),%eax
 80b113c:       8b 12                   mov    (%edx),%edx
 80b113e:       89 54 24 0c             mov    %edx,0xc(%esp)
 80b1142:       c1 fa 1f                sar    $0x1f,%edx
 80b1145:       3d ff c9 9a 3b          cmp    $0x3b9ac9ff,%eax
 80b114a:       89 44 24 14             mov    %eax,0x14(%esp)
 80b114e:       b8 16 00 00 00          mov    $0x16,%eax
 80b1153:       89 54 24 10             mov    %edx,0x10(%esp)
 80b1157:       77 19                   ja     80b1172 <___pthread_cond_clockwait+0x62>
 80b1159:       83 f9 01                cmp    $0x1,%ecx
 80b115c:       77 14                   ja     80b1172 <___pthread_cond_clockwait+0x62>
 80b115e:       83 ec 0c                sub    $0xc,%esp
 80b1161:       89 f2                   mov    %esi,%edx
 80b1163:       8d 44 24 18             lea    0x18(%esp),%eax
 80b1167:       50                      push   %eax
 80b1168:       89 d8                   mov    %ebx,%eax
 80b116a:       e8 71 f4 ff ff          call   80b05e0 <___pthread_cond_clockwait64.part.0>
 80b116f:       83 c4 10                add    $0x10,%esp
 80b1172:       8b 54 24 1c             mov    0x1c(%esp),%edx
 80b1176:       65 2b 15 14 00 00 00    sub    %gs:0x14,%edx
 80b117d:       75 07                   jne    80b1186 <___pthread_cond_clockwait+0x76>
 80b117f:       83 c4 20                add    $0x20,%esp
 80b1182:       5b                      pop    %ebx
 80b1183:       5e                      pop    %esi
 80b1184:       5f                      pop    %edi
 80b1185:       c3                      ret
 80b1186:       e8 85 b8 fa ff          call   805ca10 <__stack_chk_fail>
 80b118b:       66 90                   xchg   %ax,%ax
 80b118d:       66 90                   xchg   %ax,%ax
 80b118f:       90                      nop

080b1190 <__pthread_mutex_cond_lock_full>:
 80b1190:       55                      push   %ebp
 80b1191:       57                      push   %edi
 80b1192:       56                      push   %esi
 80b1193:       e8 97 ac f9 ff          call   804be2f <__x86.get_pc_thunk.si>
 80b1198:       81 c6 5c 5e 03 00       add    $0x35e5c,%esi
 80b119e:       53                      push   %ebx
 80b119f:       83 ec 3c                sub    $0x3c,%esp
 80b11a2:       89 74 24 14             mov    %esi,0x14(%esp)
 80b11a6:       65 8b 2d 14 00 00 00    mov    %gs:0x14,%ebp
 80b11ad:       89 6c 24 2c             mov    %ebp,0x2c(%esp)
 80b11b1:       89 c5                   mov    %eax,%ebp
 80b11b3:       65 a1 68 00 00 00       mov    %gs:0x68,%eax
 80b11b9:       89 44 24 1c             mov    %eax,0x1c(%esp)
 80b11bd:       8d 45 0c                lea    0xc(%ebp),%eax
 80b11c0:       89 44 24 10             mov    %eax,0x10(%esp)
 80b11c4:       8b 55 0c                mov    0xc(%ebp),%edx
 80b11c7:       89 d0                   mov    %edx,%eax
 80b11c9:       83 e0 7f                and    $0x7f,%eax
 80b11cc:       83 f8 33                cmp    $0x33,%eax
 80b11cf:       0f 87 ab 00 00 00       ja     80b1280 <__pthread_mutex_cond_lock_full+0xf0>
 80b11d5:       83 f8 2f                cmp    $0x2f,%eax
 80b11d8:       0f 87 bd 01 00 00       ja     80b139b <__pthread_mutex_cond_lock_full+0x20b>
 80b11de:       83 f8 13                cmp    $0x13,%eax
 80b11e1:       0f 87 5a 02 00 00       ja     80b1441 <__pthread_mutex_cond_lock_full+0x2b1>
 80b11e7:       83 e2 70                and    $0x70,%edx
 80b11ea:       0f 84 5d 02 00 00       je     80b144d <__pthread_mutex_cond_lock_full+0x2bd>
 80b11f0:       8d 7d 14                lea    0x14(%ebp),%edi
 80b11f3:       65 89 3d 74 00 00 00    mov    %edi,%gs:0x74
 80b11fa:       8b 44 24 1c             mov    0x1c(%esp),%eax
 80b11fe:       8b 55 00                mov    0x0(%ebp),%edx
 80b1201:       89 eb                   mov    %ebp,%ebx
 80b1203:       0d 00 00 00 80          or     $0x80000000,%eax
 80b1208:       89 44 24 0c             mov    %eax,0xc(%esp)
 80b120c:       85 d2                   test   %edx,%edx
 80b120e:       0f 85 c1 02 00 00       jne    80b14d5 <__pthread_mutex_cond_lock_full+0x345>
 80b1214:       8b 74 24 0c             mov    0xc(%esp),%esi
 80b1218:       89 d0                   mov    %edx,%eax
 80b121a:       f0 0f b1 33             lock cmpxchg %esi,(%ebx)
 80b121e:       89 c2                   mov    %eax,%edx
 80b1220:       85 c0                   test   %eax,%eax
 80b1222:       0f 85 ad 02 00 00       jne    80b14d5 <__pthread_mutex_cond_lock_full+0x345>
 80b1228:       81 7b 08 fe ff ff 7f    cmpl   $0x7ffffffe,0x8(%ebx)
 80b122f:       89 dd                   mov    %ebx,%ebp
 80b1231:       0f 84 36 04 00 00       je     80b166d <__pthread_mutex_cond_lock_full+0x4dd>
 80b1237:       c7 43 04 01 00 00 00    movl   $0x1,0x4(%ebx)
 80b123e:       65 a1 6c 00 00 00       mov    %gs:0x6c,%eax
 80b1244:       89 43 14                mov    %eax,0x14(%ebx)
 80b1247:       65 89 3d 6c 00 00 00    mov    %edi,%gs:0x6c
 80b124e:       65 c7 05 74 00 00 00    movl   $0x0,%gs:0x74
 80b1255:       00 00 00 00
 80b1259:       8b 44 24 1c             mov    0x1c(%esp),%eax
 80b125d:       89 45 08                mov    %eax,0x8(%ebp)
 80b1260:       31 c0                   xor    %eax,%eax
 80b1262:       8b 54 24 2c             mov    0x2c(%esp),%edx
 80b1266:       65 2b 15 14 00 00 00    sub    %gs:0x14,%edx
 80b126d:       0f 85 3d 05 00 00       jne    80b17b0 <__pthread_mutex_cond_lock_full+0x620>
 80b1273:       83 c4 3c                add    $0x3c,%esp
 80b1276:       5b                      pop    %ebx
 80b1277:       5e                      pop    %esi
 80b1278:       5f                      pop    %edi
 80b1279:       5d                      pop    %ebp
 80b127a:       c3                      ret
 80b127b:       2e 8d 74 26 00          lea    %cs:0x0(%esi,%eiz,1),%esi
 80b1280:       83 e8 40                sub    $0x40,%eax
 80b1283:       83 f8 03                cmp    $0x3,%eax
 80b1286:       0f 87 c1 01 00 00       ja     80b144d <__pthread_mutex_cond_lock_full+0x2bd>
 80b128c:       8b 45 0c                mov    0xc(%ebp),%eax
 80b128f:       be ff ff ff ff          mov    $0xffffffff,%esi
 80b1294:       8b 4d 00                mov    0x0(%ebp),%ecx
 80b1297:       8b 5c 24 1c             mov    0x1c(%esp),%ebx
 80b129b:       3b 5d 08                cmp    0x8(%ebp),%ebx
 80b129e:       0f 84 05 02 00 00       je     80b14a9 <__pthread_mutex_cond_lock_full+0x319>
 80b12a4:       89 f7                   mov    %esi,%edi
 80b12a6:       89 ce                   mov    %ecx,%esi
 80b12a8:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80b12af:       00
 80b12b0:       8b 5c 24 14             mov    0x14(%esp),%ebx
 80b12b4:       89 f0                   mov    %esi,%eax
 80b12b6:       c1 e8 13                shr    $0x13,%eax
 80b12b9:       89 44 24 18             mov    %eax,0x18(%esp)
 80b12bd:       e8 7e 9a fc ff          call   807ad40 <__pthread_current_priority>
 80b12c2:       8b 5c 24 18             mov    0x18(%esp),%ebx
 80b12c6:       39 d8                   cmp    %ebx,%eax
 80b12c8:       0f 8f 48 02 00 00       jg     80b1516 <__pthread_mutex_cond_lock_full+0x386>
 80b12ce:       83 ec 08                sub    $0x8,%esp
 80b12d1:       ff 74 24 20             push   0x20(%esp)
 80b12d5:       57                      push   %edi
 80b12d6:       8b 5c 24 24             mov    0x24(%esp),%ebx
 80b12da:       e8 61 96 fc ff          call   807a940 <__pthread_tpp_change_priority>
 80b12df:       83 c4 10                add    $0x10,%esp
 80b12e2:       85 c0                   test   %eax,%eax
 80b12e4:       0f 85 78 ff ff ff       jne    80b1262 <__pthread_mutex_cond_lock_full+0xd2>
 80b12ea:       81 e6 00 00 f8 ff       and    $0xfff80000,%esi
 80b12f0:       89 f2                   mov    %esi,%edx
 80b12f2:       89 f0                   mov    %esi,%eax
 80b12f4:       83 ca 02                or     $0x2,%edx
 80b12f7:       f0 0f b1 55 00          lock cmpxchg %edx,0x0(%ebp)
 80b12fc:       74 74                   je     80b1372 <__pthread_mutex_cond_lock_full+0x1e2>
 80b12fe:       89 f0                   mov    %esi,%eax
 80b1300:       89 eb                   mov    %ebp,%ebx
 80b1302:       89 f5                   mov    %esi,%ebp
 80b1304:       83 c8 01                or     $0x1,%eax
 80b1307:       89 44 24 0c             mov    %eax,0xc(%esp)
 80b130b:       eb 0d                   jmp    80b131a <__pthread_mutex_cond_lock_full+0x18a>
 80b130d:       8d 76 00                lea    0x0(%esi),%esi
 80b1310:       89 f8                   mov    %edi,%eax
 80b1312:       f0 0f b1 13             lock cmpxchg %edx,(%ebx)
 80b1316:       39 e8                   cmp    %ebp,%eax
 80b1318:       74 56                   je     80b1370 <__pthread_mutex_cond_lock_full+0x1e0>
 80b131a:       8b 44 24 0c             mov    0xc(%esp),%eax
 80b131e:       f0 0f b1 13             lock cmpxchg %edx,(%ebx)
 80b1322:       89 c7                   mov    %eax,%edi
 80b1324:       89 c6                   mov    %eax,%esi
 80b1326:       81 e7 00 00 f8 ff       and    $0xfff80000,%edi
 80b132c:       39 fd                   cmp    %edi,%ebp
 80b132e:       75 60                   jne    80b1390 <__pthread_mutex_cond_lock_full+0x200>
 80b1330:       39 c5                   cmp    %eax,%ebp
 80b1332:       74 dc                   je     80b1310 <__pthread_mutex_cond_lock_full+0x180>
 80b1334:       8b 44 24 10             mov    0x10(%esp),%eax
 80b1338:       31 f6                   xor    %esi,%esi
 80b133a:       8b 08                   mov    (%eax),%ecx
 80b133c:       b8 f0 00 00 00          mov    $0xf0,%eax
 80b1341:       f7 d1                   not    %ecx
 80b1343:       81 e1 80 00 00 00       and    $0x80,%ecx
 80b1349:       65 ff 15 10 00 00 00    call   *%gs:0x10
 80b1350:       3d 00 f0 ff ff          cmp    $0xfffff000,%eax
 80b1355:       76 b9                   jbe    80b1310 <__pthread_mutex_cond_lock_full+0x180>
 80b1357:       83 f8 f5                cmp    $0xfffffff5,%eax
 80b135a:       74 b4                   je     80b1310 <__pthread_mutex_cond_lock_full+0x180>
 80b135c:       83 f8 fc                cmp    $0xfffffffc,%eax
 80b135f:       74 af                   je     80b1310 <__pthread_mutex_cond_lock_full+0x180>
 80b1361:       e9 f3 01 00 00          jmp    80b1559 <__pthread_mutex_cond_lock_full+0x3c9>
 80b1366:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80b136d:       00
 80b136e:       66 90                   xchg   %ax,%ax
 80b1370:       89 dd                   mov    %ebx,%ebp
 80b1372:       8b 45 08                mov    0x8(%ebp),%eax
 80b1375:       85 c0                   test   %eax,%eax
 80b1377:       0f 85 ed 03 00 00       jne    80b176a <__pthread_mutex_cond_lock_full+0x5da>
 80b137d:       c7 45 04 01 00 00 00    movl   $0x1,0x4(%ebp)
 80b1384:       e9 d0 fe ff ff          jmp    80b1259 <__pthread_mutex_cond_lock_full+0xc9>
 80b1389:       8d b4 26 00 00 00 00    lea    0x0(%esi,%eiz,1),%esi
 80b1390:       8b 7c 24 18             mov    0x18(%esp),%edi
 80b1394:       89 dd                   mov    %ebx,%ebp
 80b1396:       e9 15 ff ff ff          jmp    80b12b0 <__pthread_mutex_cond_lock_full+0x120>
 80b139b:       8b 55 0c                mov    0xc(%ebp),%edx
 80b139e:       89 d7                   mov    %edx,%edi
 80b13a0:       83 e7 03                and    $0x3,%edi
 80b13a3:       83 e2 10                and    $0x10,%edx
 80b13a6:       0f 85 ab 00 00 00       jne    80b1457 <__pthread_mutex_cond_lock_full+0x2c7>
 80b13ac:       8b 45 00                mov    0x0(%ebp),%eax
 80b13af:       25 ff ff ff 3f          and    $0x3fffffff,%eax
 80b13b4:       39 44 24 1c             cmp    %eax,0x1c(%esp)
 80b13b8:       0f 84 8d 02 00 00       je     80b164b <__pthread_mutex_cond_lock_full+0x4bb>
 80b13be:       8b 4c 24 1c             mov    0x1c(%esp),%ecx
 80b13c2:       31 c0                   xor    %eax,%eax
 80b13c4:       81 c9 00 00 00 80       or     $0x80000000,%ecx
 80b13ca:       f0 0f b1 4d 00          lock cmpxchg %ecx,0x0(%ebp)
 80b13cf:       0f 84 93 00 00 00       je     80b1468 <__pthread_mutex_cond_lock_full+0x2d8>
 80b13d5:       85 d2                   test   %edx,%edx
 80b13d7:       0f 84 cd 01 00 00       je     80b15aa <__pthread_mutex_cond_lock_full+0x41a>
 80b13dd:       68 80 00 00 00          push   $0x80
 80b13e2:       6a 00                   push   $0x0
 80b13e4:       6a 00                   push   $0x0
 80b13e6:       55                      push   %ebp
 80b13e7:       8b 5c 24 24             mov    0x24(%esp),%ebx
 80b13eb:       e8 b0 9b fe ff          call   809afa0 <__futex_lock_pi64>
 80b13f0:       83 c4 10                add    $0x10,%esp
 80b13f3:       89 c2                   mov    %eax,%edx
 80b13f5:       83 e2 df                and    $0xffffffdf,%edx
 80b13f8:       83 fa 03                cmp    $0x3,%edx
 80b13fb:       0f 85 ff 01 00 00       jne    80b1600 <__pthread_mutex_cond_lock_full+0x470>
 80b1401:       83 f8 23                cmp    $0x23,%eax
 80b1404:       0f 85 83 03 00 00       jne    80b178d <__pthread_mutex_cond_lock_full+0x5fd>
 80b140a:       be 80 00 00 00          mov    $0x80,%esi
 80b140f:       83 ef 01                sub    $0x1,%edi
 80b1412:       83 ff 01                cmp    $0x1,%edi
 80b1415:       0f 86 9a 03 00 00       jbe    80b17b5 <__pthread_mutex_cond_lock_full+0x625>
 80b141b:       8d 7c 24 28             lea    0x28(%esp),%edi
 80b141f:       90                      nop
 80b1420:       c7 44 24 28 00 00 00    movl   $0x0,0x28(%esp)
 80b1427:       00
 80b1428:       83 ec 0c                sub    $0xc,%esp
 80b142b:       56                      push   %esi
 80b142c:       6a 00                   push   $0x0
 80b142e:       6a 00                   push   $0x0
 80b1430:       6a 00                   push   $0x0
 80b1432:       57                      push   %edi
 80b1433:       8b 5c 24 34             mov    0x34(%esp),%ebx
 80b1437:       e8 04 9b fe ff          call   809af40 <__futex_abstimed_wait64>
 80b143c:       83 c4 20                add    $0x20,%esp
 80b143f:       eb df                   jmp    80b1420 <__pthread_mutex_cond_lock_full+0x290>
 80b1441:       83 e8 20                sub    $0x20,%eax
 80b1444:       83 f8 03                cmp    $0x3,%eax
 80b1447:       0f 86 4e ff ff ff       jbe    80b139b <__pthread_mutex_cond_lock_full+0x20b>
 80b144d:       b8 16 00 00 00          mov    $0x16,%eax
 80b1452:       e9 0b fe ff ff          jmp    80b1262 <__pthread_mutex_cond_lock_full+0xd2>
 80b1457:       8d 45 14                lea    0x14(%ebp),%eax
 80b145a:       83 c8 01                or     $0x1,%eax
 80b145d:       65 a3 74 00 00 00       mov    %eax,%gs:0x74
 80b1463:       e9 44 ff ff ff          jmp    80b13ac <__pthread_mutex_cond_lock_full+0x21c>
 80b1468:       85 d2                   test   %edx,%edx
 80b146a:       0f 84 0d ff ff ff       je     80b137d <__pthread_mutex_cond_lock_full+0x1ed>
 80b1470:       81 7d 08 fe ff ff 7f    cmpl   $0x7ffffffe,0x8(%ebp)
 80b1477:       0f 84 32 02 00 00       je     80b16af <__pthread_mutex_cond_lock_full+0x51f>
 80b147d:       c7 45 04 01 00 00 00    movl   $0x1,0x4(%ebp)
 80b1484:       65 a1 6c 00 00 00       mov    %gs:0x6c,%eax
 80b148a:       89 45 14                mov    %eax,0x14(%ebp)
 80b148d:       8d 45 14                lea    0x14(%ebp),%eax
 80b1490:       83 c8 01                or     $0x1,%eax
 80b1493:       65 a3 6c 00 00 00       mov    %eax,%gs:0x6c
 80b1499:       65 c7 05 74 00 00 00    movl   $0x0,%gs:0x74
 80b14a0:       00 00 00 00
 80b14a4:       e9 b0 fd ff ff          jmp    80b1259 <__pthread_mutex_cond_lock_full+0xc9>
 80b14a9:       83 e0 03                and    $0x3,%eax
 80b14ac:       83 f8 02                cmp    $0x2,%eax
 80b14af:       0f 84 4e 02 00 00       je     80b1703 <__pthread_mutex_cond_lock_full+0x573>
 80b14b5:       83 f8 01                cmp    $0x1,%eax
 80b14b8:       0f 85 e6 fd ff ff       jne    80b12a4 <__pthread_mutex_cond_lock_full+0x114>
 80b14be:       8b 45 04                mov    0x4(%ebp),%eax
 80b14c1:       83 f8 ff                cmp    $0xffffffff,%eax
 80b14c4:       0f 84 56 02 00 00       je     80b1720 <__pthread_mutex_cond_lock_full+0x590>
 80b14ca:       83 c0 01                add    $0x1,%eax
 80b14cd:       89 45 04                mov    %eax,0x4(%ebp)
 80b14d0:       e9 8b fd ff ff          jmp    80b1260 <__pthread_mutex_cond_lock_full+0xd0>
 80b14d5:       f7 c2 00 00 00 40       test   $0x40000000,%edx
 80b14db:       75 59                   jne    80b1536 <__pthread_mutex_cond_lock_full+0x3a6>
 80b14dd:       89 d0                   mov    %edx,%eax
 80b14df:       8b 74 24 1c             mov    0x1c(%esp),%esi
 80b14e3:       25 ff ff ff 3f          and    $0x3fffffff,%eax
 80b14e8:       39 f0                   cmp    %esi,%eax
 80b14ea:       0f 84 3a 02 00 00       je     80b172a <__pthread_mutex_cond_lock_full+0x59a>
 80b14f0:       85 d2                   test   %edx,%edx
 80b14f2:       0f 89 9c 01 00 00       jns    80b1694 <__pthread_mutex_cond_lock_full+0x504>
 80b14f8:       b8 f0 00 00 00          mov    $0xf0,%eax
 80b14fd:       31 c9                   xor    %ecx,%ecx
 80b14ff:       31 f6                   xor    %esi,%esi
 80b1501:       65 ff 15 10 00 00 00    call   *%gs:0x10
 80b1508:       3d 00 f0 ff ff          cmp    $0xfffff000,%eax
 80b150d:       77 40                   ja     80b154f <__pthread_mutex_cond_lock_full+0x3bf>
 80b150f:       8b 13                   mov    (%ebx),%edx
 80b1511:       e9 f6 fc ff ff          jmp    80b120c <__pthread_mutex_cond_lock_full+0x7c>
 80b1516:       83 ff ff                cmp    $0xffffffff,%edi
 80b1519:       0f 84 2e ff ff ff       je     80b144d <__pthread_mutex_cond_lock_full+0x2bd>
 80b151f:       83 ec 08                sub    $0x8,%esp
 80b1522:       6a ff                   push   $0xffffffff
 80b1524:       57                      push   %edi
 80b1525:       8b 5c 24 24             mov    0x24(%esp),%ebx
 80b1529:       e8 12 94 fc ff          call   807a940 <__pthread_tpp_change_priority>
 80b152e:       83 c4 10                add    $0x10,%esp
 80b1531:       e9 17 ff ff ff          jmp    80b144d <__pthread_mutex_cond_lock_full+0x2bd>
 80b1536:       8b 4c 24 1c             mov    0x1c(%esp),%ecx
 80b153a:       89 d0                   mov    %edx,%eax
 80b153c:       81 c9 00 00 00 80       or     $0x80000000,%ecx
 80b1542:       f0 0f b1 0b             lock cmpxchg %ecx,(%ebx)
 80b1546:       74 24                   je     80b156c <__pthread_mutex_cond_lock_full+0x3dc>
 80b1548:       89 c2                   mov    %eax,%edx
 80b154a:       e9 bd fc ff ff          jmp    80b120c <__pthread_mutex_cond_lock_full+0x7c>
 80b154f:       83 f8 f5                cmp    $0xfffffff5,%eax
 80b1552:       74 bb                   je     80b150f <__pthread_mutex_cond_lock_full+0x37f>
 80b1554:       83 f8 fc                cmp    $0xfffffffc,%eax
 80b1557:       74 b6                   je     80b150f <__pthread_mutex_cond_lock_full+0x37f>
 80b1559:       83 ec 0c                sub    $0xc,%esp
 80b155c:       8b 5c 24 20             mov    0x20(%esp),%ebx
 80b1560:       8d 83 ec d6 fc ff       lea    -0x32914(%ebx),%eax
 80b1566:       50                      push   %eax
 80b1567:       e8 f4 b9 f9 ff          call   804cf60 <__libc_fatal>
 80b156c:       89 dd                   mov    %ebx,%ebp
 80b156e:       c7 43 04 01 00 00 00    movl   $0x1,0x4(%ebx)
 80b1575:       c7 43 08 ff ff ff 7f    movl   $0x7fffffff,0x8(%ebx)
 80b157c:       65 a1 6c 00 00 00       mov    %gs:0x6c,%eax
 80b1582:       89 43 14                mov    %eax,0x14(%ebx)
 80b1585:       65 89 3d 6c 00 00 00    mov    %edi,%gs:0x6c
 80b158c:       65 c7 05 74 00 00 00    movl   $0x0,%gs:0x74
 80b1593:       00 00 00 00
 80b1597:       8b 43 10                mov    0x10(%ebx),%eax
 80b159a:       83 e8 01                sub    $0x1,%eax
 80b159d:       89 45 10                mov    %eax,0x10(%ebp)
 80b15a0:       b8 82 00 00 00          mov    $0x82,%eax
 80b15a5:       e9 b8 fc ff ff          jmp    80b1262 <__pthread_mutex_cond_lock_full+0xd2>
 80b15aa:       8b 75 0c                mov    0xc(%ebp),%esi
 80b15ad:       81 e6 80 00 00 00       and    $0x80,%esi
 80b15b3:       56                      push   %esi
 80b15b4:       6a 00                   push   $0x0
 80b15b6:       6a 00                   push   $0x0
 80b15b8:       55                      push   %ebp
 80b15b9:       8b 5c 24 24             mov    0x24(%esp),%ebx
 80b15bd:       e8 de 99 fe ff          call   809afa0 <__futex_lock_pi64>
 80b15c2:       83 c4 10                add    $0x10,%esp
 80b15c5:       89 c2                   mov    %eax,%edx
 80b15c7:       83 e2 df                and    $0xffffffdf,%edx
 80b15ca:       83 fa 03                cmp    $0x3,%edx
 80b15cd:       0f 84 3a 01 00 00       je     80b170d <__pthread_mutex_cond_lock_full+0x57d>
 80b15d3:       f6 45 03 40             testb  $0x40,0x3(%ebp)
 80b15d7:       0f 84 a0 fd ff ff       je     80b137d <__pthread_mutex_cond_lock_full+0x1ed>
 80b15dd:       8b 74 24 14             mov    0x14(%esp),%esi
 80b15e1:       8d 86 a8 11 fd ff       lea    -0x2ee58(%esi),%eax
 80b15e7:       50                      push   %eax
 80b15e8:       8d 86 a7 d5 fc ff       lea    -0x32a59(%esi),%eax
 80b15ee:       68 cc 01 00 00          push   $0x1cc
 80b15f3:       50                      push   %eax
 80b15f4:       8d 86 c8 f0 fc ff       lea    -0x30f38(%esi),%eax
 80b15fa:       50                      push   %eax
 80b15fb:       e8 20 ab f9 ff          call   804c120 <__libc_assert_fail>
 80b1600:       f6 45 03 40             testb  $0x40,0x3(%ebp)
 80b1604:       0f 84 66 fe ff ff       je     80b1470 <__pthread_mutex_cond_lock_full+0x2e0>
 80b160a:       f0 81 65 00 ff ff ff    lock andl $0xbfffffff,0x0(%ebp)
 80b1611:       bf
 80b1612:       c7 45 04 01 00 00 00    movl   $0x1,0x4(%ebp)
 80b1619:       c7 45 08 ff ff ff 7f    movl   $0x7fffffff,0x8(%ebp)
 80b1620:       65 a1 6c 00 00 00       mov    %gs:0x6c,%eax
 80b1626:       89 45 14                mov    %eax,0x14(%ebp)
 80b1629:       8d 45 14                lea    0x14(%ebp),%eax
 80b162c:       83 c8 01                or     $0x1,%eax
 80b162f:       65 a3 6c 00 00 00       mov    %eax,%gs:0x6c
 80b1635:       65 c7 05 74 00 00 00    movl   $0x0,%gs:0x74
 80b163c:       00 00 00 00
 80b1640:       8b 45 10                mov    0x10(%ebp),%eax
 80b1643:       83 e8 01                sub    $0x1,%eax
 80b1646:       e9 52 ff ff ff          jmp    80b159d <__pthread_mutex_cond_lock_full+0x40d>
 80b164b:       83 ff 02                cmp    $0x2,%edi
 80b164e:       0f 84 a4 00 00 00       je     80b16f8 <__pthread_mutex_cond_lock_full+0x568>
 80b1654:       83 ff 01                cmp    $0x1,%edi
 80b1657:       0f 85 61 fd ff ff       jne    80b13be <__pthread_mutex_cond_lock_full+0x22e>
 80b165d:       65 c7 05 74 00 00 00    movl   $0x0,%gs:0x74
 80b1664:       00 00 00 00
 80b1668:       e9 51 fe ff ff          jmp    80b14be <__pthread_mutex_cond_lock_full+0x32e>
 80b166d:       c7 43 04 00 00 00 00    movl   $0x0,0x4(%ebx)
 80b1674:       87 13                   xchg   %edx,(%ebx)
 80b1676:       83 fa 01                cmp    $0x1,%edx
 80b1679:       0f 8f d1 00 00 00       jg     80b1750 <__pthread_mutex_cond_lock_full+0x5c0>
 80b167f:       65 c7 05 74 00 00 00    movl   $0x0,%gs:0x74
 80b1686:       00 00 00 00
 80b168a:       b8 83 00 00 00          mov    $0x83,%eax
 80b168f:       e9 ce fb ff ff          jmp    80b1262 <__pthread_mutex_cond_lock_full+0xd2>
 80b1694:       89 d1                   mov    %edx,%ecx
 80b1696:       89 d0                   mov    %edx,%eax
 80b1698:       81 c9 00 00 00 80       or     $0x80000000,%ecx
 80b169e:       f0 0f b1 0b             lock cmpxchg %ecx,(%ebx)
 80b16a2:       0f 85 a0 fe ff ff       jne    80b1548 <__pthread_mutex_cond_lock_full+0x3b8>
 80b16a8:       89 ca                   mov    %ecx,%edx
 80b16aa:       e9 49 fe ff ff          jmp    80b14f8 <__pthread_mutex_cond_lock_full+0x368>
 80b16af:       c7 45 04 00 00 00 00    movl   $0x0,0x4(%ebp)
 80b16b6:       b8 f0 00 00 00          mov    $0xf0,%eax
 80b16bb:       89 eb                   mov    %ebp,%ebx
 80b16bd:       31 d2                   xor    %edx,%edx
 80b16bf:       b9 07 00 00 00          mov    $0x7,%ecx
 80b16c4:       31 f6                   xor    %esi,%esi
 80b16c6:       65 ff 15 10 00 00 00    call   *%gs:0x10
 80b16cd:       3d 00 f0 ff ff          cmp    $0xfffff000,%eax
 80b16d2:       76 ab                   jbe    80b167f <__pthread_mutex_cond_lock_full+0x4ef>
 80b16d4:       83 f8 f5                cmp    $0xfffffff5,%eax
 80b16d7:       0f 8d fb 00 00 00       jge    80b17d8 <__pthread_mutex_cond_lock_full+0x648>
 80b16dd:       83 f8 da                cmp    $0xffffffda,%eax
 80b16e0:       74 9d                   je     80b167f <__pthread_mutex_cond_lock_full+0x4ef>
 80b16e2:       83 f8 dd                cmp    $0xffffffdd,%eax
 80b16e5:       74 98                   je     80b167f <__pthread_mutex_cond_lock_full+0x4ef>
 80b16e7:       83 f8 92                cmp    $0xffffff92,%eax
 80b16ea:       74 93                   je     80b167f <__pthread_mutex_cond_lock_full+0x4ef>
 80b16ec:       e9 68 fe ff ff          jmp    80b1559 <__pthread_mutex_cond_lock_full+0x3c9>
 80b16f1:       8d b4 26 00 00 00 00    lea    0x0(%esi,%eiz,1),%esi
 80b16f8:       65 c7 05 74 00 00 00    movl   $0x0,%gs:0x74
 80b16ff:       00 00 00 00
 80b1703:       b8 23 00 00 00          mov    $0x23,%eax
 80b1708:       e9 55 fb ff ff          jmp    80b1262 <__pthread_mutex_cond_lock_full+0xd2>
 80b170d:       83 f8 23                cmp    $0x23,%eax
 80b1710:       0f 85 05 fd ff ff       jne    80b141b <__pthread_mutex_cond_lock_full+0x28b>
 80b1716:       e9 f4 fc ff ff          jmp    80b140f <__pthread_mutex_cond_lock_full+0x27f>
 80b171b:       2e 8d 74 26 00          lea    %cs:0x0(%esi,%eiz,1),%esi
 80b1720:       b8 0b 00 00 00          mov    $0xb,%eax
 80b1725:       e9 38 fb ff ff          jmp    80b1262 <__pthread_mutex_cond_lock_full+0xd2>
 80b172a:       8b 44 24 10             mov    0x10(%esp),%eax
 80b172e:       8b 00                   mov    (%eax),%eax
 80b1730:       83 e0 7f                and    $0x7f,%eax
 80b1733:       83 f8 12                cmp    $0x12,%eax
 80b1736:       74 c0                   je     80b16f8 <__pthread_mutex_cond_lock_full+0x568>
 80b1738:       83 f8 11                cmp    $0x11,%eax
 80b173b:       0f 85 af fd ff ff       jne    80b14f0 <__pthread_mutex_cond_lock_full+0x360>
 80b1741:       89 dd                   mov    %ebx,%ebp
 80b1743:       e9 15 ff ff ff          jmp    80b165d <__pthread_mutex_cond_lock_full+0x4cd>
 80b1748:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80b174f:       00
 80b1750:       83 ec 08                sub    $0x8,%esp
 80b1753:       68 80 00 00 00          push   $0x80
 80b1758:       53                      push   %ebx
 80b1759:       8b 5c 24 24             mov    0x24(%esp),%ebx
 80b175d:       e8 7e 15 fa ff          call   8052ce0 <__lll_lock_wake>
 80b1762:       83 c4 10                add    $0x10,%esp
 80b1765:       e9 15 ff ff ff          jmp    80b167f <__pthread_mutex_cond_lock_full+0x4ef>
 80b176a:       8b 74 24 14             mov    0x14(%esp),%esi
 80b176e:       8d 86 a8 11 fd ff       lea    -0x2ee58(%esi),%eax
 80b1774:       50                      push   %eax
 80b1775:       8d 86 a7 d5 fc ff       lea    -0x32a59(%esi),%eax
 80b177b:       68 5f 02 00 00          push   $0x25f
 80b1780:       50                      push   %eax
 80b1781:       8d 86 f8 be fc ff       lea    -0x34108(%esi),%eax
 80b1787:       50                      push   %eax
 80b1788:       e8 93 a9 f9 ff          call   804c120 <__libc_assert_fail>
 80b178d:       8b 74 24 14             mov    0x14(%esp),%esi
 80b1791:       8d 86 a8 11 fd ff       lea    -0x2ee58(%esi),%eax
 80b1797:       50                      push   %eax
 80b1798:       8d 86 a7 d5 fc ff       lea    -0x32a59(%esi),%eax
 80b179e:       68 c2 01 00 00          push   $0x1c2
 80b17a3:       50                      push   %eax
 80b17a4:       8d 86 e2 be fc ff       lea    -0x3411e(%esi),%eax
 80b17aa:       50                      push   %eax
 80b17ab:       e8 70 a9 f9 ff          call   804c120 <__libc_assert_fail>
 80b17b0:       e8 5b b2 fa ff          call   805ca10 <__stack_chk_fail>
 80b17b5:       8b 74 24 14             mov    0x14(%esp),%esi
 80b17b9:       8d 86 a8 11 fd ff       lea    -0x2ee58(%esi),%eax
 80b17bf:       50                      push   %eax
 80b17c0:       8d 86 a7 d5 fc ff       lea    -0x32a59(%esi),%eax
 80b17c6:       68 bd 01 00 00          push   $0x1bd
 80b17cb:       50                      push   %eax
 80b17cc:       8d 86 6c f0 fc ff       lea    -0x30f94(%esi),%eax
 80b17d2:       50                      push   %eax
 80b17d3:       e8 48 a9 f9 ff          call   804c120 <__libc_assert_fail>
 80b17d8:       83 c0 0b                add    $0xb,%eax
 80b17db:       ba 81 05 00 00          mov    $0x581,%edx
 80b17e0:       0f a3 c2                bt     %eax,%edx
 80b17e3:       0f 83 70 fd ff ff       jae    80b1559 <__pthread_mutex_cond_lock_full+0x3c9>
 80b17e9:       e9 91 fe ff ff          jmp    80b167f <__pthread_mutex_cond_lock_full+0x4ef>
 80b17ee:       66 90                   xchg   %ax,%ax

080b17f0 <__pthread_mutex_cond_lock>:
 80b17f0:       57                      push   %edi
 80b17f1:       56                      push   %esi
 80b17f2:       53                      push   %ebx
 80b17f3:       8b 74 24 10             mov    0x10(%esp),%esi
 80b17f7:       e8 e4 7f f9 ff          call   80497e0 <__x86.get_pc_thunk.bx>
 80b17fc:       81 c3 f8 57 03 00       add    $0x357f8,%ebx
 80b1802:       8b 46 0c                mov    0xc(%esi),%eax
 80b1805:       89 c2                   mov    %eax,%edx
 80b1807:       81 e2 7f 01 00 00       and    $0x17f,%edx
 80b180d:       83 e0 7c                and    $0x7c,%eax
 80b1810:       75 76                   jne    80b1888 <__pthread_mutex_cond_lock+0x98>
 80b1812:       85 d2                   test   %edx,%edx
 80b1814:       0f 85 7e 00 00 00       jne    80b1898 <__pthread_mutex_cond_lock+0xa8>
 80b181a:       c7 c0 2c 86 0e 08       mov    $0x80e862c,%eax
 80b1820:       8b 00                   mov    (%eax),%eax
 80b1822:       85 c0                   test   %eax,%eax
 80b1824:       75 2a                   jne    80b1850 <__pthread_mutex_cond_lock+0x60>
 80b1826:       b8 02 00 00 00          mov    $0x2,%eax
 80b182b:       87 06                   xchg   %eax,(%esi)
 80b182d:       85 c0                   test   %eax,%eax
 80b182f:       0f 85 9b 00 00 00       jne    80b18d0 <__pthread_mutex_cond_lock+0xe0>
 80b1835:       8b 46 08                mov    0x8(%esi),%eax
 80b1838:       85 c0                   test   %eax,%eax
 80b183a:       0f 85 40 01 00 00       jne    80b1980 <__pthread_mutex_cond_lock+0x190>
 80b1840:       65 a1 68 00 00 00       mov    %gs:0x68,%eax
 80b1846:       89 46 08                mov    %eax,0x8(%esi)
 80b1849:       31 c0                   xor    %eax,%eax
 80b184b:       5b                      pop    %ebx
 80b184c:       5e                      pop    %esi
 80b184d:       5f                      pop    %edi
 80b184e:       c3                      ret
 80b184f:       90                      nop
 80b1850:       8b 46 0c                mov    0xc(%esi),%eax
 80b1853:       f6 c4 03                test   $0x3,%ah
 80b1856:       0f 84 94 00 00 00       je     80b18f0 <__pthread_mutex_cond_lock+0x100>
 80b185c:       f6 c4 01                test   $0x1,%ah
 80b185f:       74 c5                   je     80b1826 <__pthread_mutex_cond_lock+0x36>
 80b1861:       b8 02 00 00 00          mov    $0x2,%eax
 80b1866:       87 06                   xchg   %eax,(%esi)
 80b1868:       85 c0                   test   %eax,%eax
 80b186a:       74 dd                   je     80b1849 <__pthread_mutex_cond_lock+0x59>
 80b186c:       8b 46 0c                mov    0xc(%esi),%eax
 80b186f:       83 ec 08                sub    $0x8,%esp
 80b1872:       25 80 00 00 00          and    $0x80,%eax
 80b1877:       50                      push   %eax
 80b1878:       56                      push   %esi
 80b1879:       e8 c2 13 fa ff          call   8052c40 <__lll_lock_wait>
 80b187e:       83 c4 10                add    $0x10,%esp
 80b1881:       eb c6                   jmp    80b1849 <__pthread_mutex_cond_lock+0x59>
 80b1883:       2e 8d 74 26 00          lea    %cs:0x0(%esi,%eiz,1),%esi
 80b1888:       5b                      pop    %ebx
 80b1889:       89 f0                   mov    %esi,%eax
 80b188b:       5e                      pop    %esi
 80b188c:       5f                      pop    %edi
 80b188d:       e9 fe f8 ff ff          jmp    80b1190 <__pthread_mutex_cond_lock_full>
 80b1892:       8d b6 00 00 00 00       lea    0x0(%esi),%esi
 80b1898:       81 fa 00 01 00 00       cmp    $0x100,%edx
 80b189e:       74 c1                   je     80b1861 <__pthread_mutex_cond_lock+0x71>
 80b18a0:       8b 56 0c                mov    0xc(%esi),%edx
 80b18a3:       83 e2 7f                and    $0x7f,%edx
 80b18a6:       83 fa 01                cmp    $0x1,%edx
 80b18a9:       75 72                   jne    80b191d <__pthread_mutex_cond_lock+0x12d>
 80b18ab:       65 a1 68 00 00 00       mov    %gs:0x68,%eax
 80b18b1:       39 46 08                cmp    %eax,0x8(%esi)
 80b18b4:       75 45                   jne    80b18fb <__pthread_mutex_cond_lock+0x10b>
 80b18b6:       8b 46 04                mov    0x4(%esi),%eax
 80b18b9:       83 f8 ff                cmp    $0xffffffff,%eax
 80b18bc:       0f 84 b4 00 00 00       je     80b1976 <__pthread_mutex_cond_lock+0x186>
 80b18c2:       83 c0 01                add    $0x1,%eax
 80b18c5:       89 46 04                mov    %eax,0x4(%esi)
 80b18c8:       e9 7c ff ff ff          jmp    80b1849 <__pthread_mutex_cond_lock+0x59>
 80b18cd:       8d 76 00                lea    0x0(%esi),%esi
 80b18d0:       8b 46 0c                mov    0xc(%esi),%eax
 80b18d3:       83 ec 08                sub    $0x8,%esp
 80b18d6:       25 80 00 00 00          and    $0x80,%eax
 80b18db:       50                      push   %eax
 80b18dc:       56                      push   %esi
 80b18dd:       e8 5e 13 fa ff          call   8052c40 <__lll_lock_wait>
 80b18e2:       83 c4 10                add    $0x10,%esp
 80b18e5:       e9 4b ff ff ff          jmp    80b1835 <__pthread_mutex_cond_lock+0x45>
 80b18ea:       8d b6 00 00 00 00       lea    0x0(%esi),%esi
 80b18f0:       80 cc 01                or     $0x1,%ah
 80b18f3:       89 46 0c                mov    %eax,0xc(%esi)
 80b18f6:       e9 66 ff ff ff          jmp    80b1861 <__pthread_mutex_cond_lock+0x71>
 80b18fb:       b8 02 00 00 00          mov    $0x2,%eax
 80b1900:       87 06                   xchg   %eax,(%esi)
 80b1902:       85 c0                   test   %eax,%eax
 80b1904:       75 5a                   jne    80b1960 <__pthread_mutex_cond_lock+0x170>
 80b1906:       8b 4e 08                mov    0x8(%esi),%ecx
 80b1909:       85 c9                   test   %ecx,%ecx
 80b190b:       0f 85 10 01 00 00       jne    80b1a21 <__pthread_mutex_cond_lock+0x231>
 80b1911:       c7 46 04 01 00 00 00    movl   $0x1,0x4(%esi)
 80b1918:       e9 23 ff ff ff          jmp    80b1840 <__pthread_mutex_cond_lock+0x50>
 80b191d:       8b 56 0c                mov    0xc(%esi),%edx
 80b1920:       83 e2 7f                and    $0x7f,%edx
 80b1923:       83 fa 03                cmp    $0x3,%edx
 80b1926:       0f 85 af 00 00 00       jne    80b19db <__pthread_mutex_cond_lock+0x1eb>
 80b192c:       ba 02 00 00 00          mov    $0x2,%edx
 80b1931:       f0 0f b1 16             lock cmpxchg %edx,(%esi)
 80b1935:       75 54                   jne    80b198b <__pthread_mutex_cond_lock+0x19b>
 80b1937:       83 7e 08 00             cmpl   $0x0,0x8(%esi)
 80b193b:       0f 84 ff fe ff ff       je     80b1840 <__pthread_mutex_cond_lock+0x50>
 80b1941:       8d 83 c8 11 fd ff       lea    -0x2ee38(%ebx),%eax
 80b1947:       50                      push   %eax
 80b1948:       68 a7 00 00 00          push   $0xa7
 80b194d:       8d 83 a7 d5 fc ff       lea    -0x32a59(%ebx),%eax
 80b1953:       50                      push   %eax
 80b1954:       8d 83 f8 be fc ff       lea    -0x34108(%ebx),%eax
 80b195a:       50                      push   %eax
 80b195b:       e8 c0 a7 f9 ff          call   804c120 <__libc_assert_fail>
 80b1960:       8b 46 0c                mov    0xc(%esi),%eax
 80b1963:       57                      push   %edi
 80b1964:       57                      push   %edi
 80b1965:       25 80 00 00 00          and    $0x80,%eax
 80b196a:       50                      push   %eax
 80b196b:       56                      push   %esi
 80b196c:       e8 cf 12 fa ff          call   8052c40 <__lll_lock_wait>
 80b1971:       83 c4 10                add    $0x10,%esp
 80b1974:       eb 90                   jmp    80b1906 <__pthread_mutex_cond_lock+0x116>
 80b1976:       b8 0b 00 00 00          mov    $0xb,%eax
 80b197b:       e9 cb fe ff ff          jmp    80b184b <__pthread_mutex_cond_lock+0x5b>
 80b1980:       8d 83 c8 11 fd ff       lea    -0x2ee38(%ebx),%eax
 80b1986:       50                      push   %eax
 80b1987:       6a 5e                   push   $0x5e
 80b1989:       eb c2                   jmp    80b194d <__pthread_mutex_cond_lock+0x15d>
 80b198b:       0f bf 46 14             movswl 0x14(%esi),%eax
 80b198f:       c7 c2 e4 7e 0e 08       mov    $0x80e7ee4,%edx
 80b1995:       83 c0 05                add    $0x5,%eax
 80b1998:       0f bf 12                movswl (%edx),%edx
 80b199b:       01 c0                   add    %eax,%eax
 80b199d:       39 d0                   cmp    %edx,%eax
 80b199f:       0f 4e d0                cmovle %eax,%edx
 80b19a2:       31 ff                   xor    %edi,%edi
 80b19a4:       83 c7 01                add    $0x1,%edi
 80b19a7:       39 fa                   cmp    %edi,%edx
 80b19a9:       7e 55                   jle    80b1a00 <__pthread_mutex_cond_lock+0x210>
 80b19ab:       f3 90                   pause
 80b19ad:       8b 06                   mov    (%esi),%eax
 80b19af:       85 c0                   test   %eax,%eax
 80b19b1:       75 f1                   jne    80b19a4 <__pthread_mutex_cond_lock+0x1b4>
 80b19b3:       b9 02 00 00 00          mov    $0x2,%ecx
 80b19b8:       f0 0f b1 0e             lock cmpxchg %ecx,(%esi)
 80b19bc:       75 e6                   jne    80b19a4 <__pthread_mutex_cond_lock+0x1b4>
 80b19be:       0f bf 56 14             movswl 0x14(%esi),%edx
 80b19c2:       89 f8                   mov    %edi,%eax
 80b19c4:       bf 08 00 00 00          mov    $0x8,%edi
 80b19c9:       29 d0                   sub    %edx,%eax
 80b19cb:       89 d1                   mov    %edx,%ecx
 80b19cd:       99                      cltd
 80b19ce:       f7 ff                   idiv   %edi
 80b19d0:       01 c1                   add    %eax,%ecx
 80b19d2:       66 89 4e 14             mov    %cx,0x14(%esi)
 80b19d6:       e9 5c ff ff ff          jmp    80b1937 <__pthread_mutex_cond_lock+0x147>
 80b19db:       65 8b 15 68 00 00 00    mov    %gs:0x68,%edx
 80b19e2:       8b 46 0c                mov    0xc(%esi),%eax
 80b19e5:       83 e0 7f                and    $0x7f,%eax
 80b19e8:       83 f8 02                cmp    $0x2,%eax
 80b19eb:       75 45                   jne    80b1a32 <__pthread_mutex_cond_lock+0x242>
 80b19ed:       39 56 08                cmp    %edx,0x8(%esi)
 80b19f0:       0f 85 30 fe ff ff       jne    80b1826 <__pthread_mutex_cond_lock+0x36>
 80b19f6:       b8 23 00 00 00          mov    $0x23,%eax
 80b19fb:       e9 4b fe ff ff          jmp    80b184b <__pthread_mutex_cond_lock+0x5b>
 80b1a00:       b8 02 00 00 00          mov    $0x2,%eax
 80b1a05:       87 06                   xchg   %eax,(%esi)
 80b1a07:       85 c0                   test   %eax,%eax
 80b1a09:       74 b3                   je     80b19be <__pthread_mutex_cond_lock+0x1ce>
 80b1a0b:       8b 46 0c                mov    0xc(%esi),%eax
 80b1a0e:       52                      push   %edx
 80b1a0f:       52                      push   %edx
 80b1a10:       25 80 00 00 00          and    $0x80,%eax
 80b1a15:       50                      push   %eax
 80b1a16:       56                      push   %esi
 80b1a17:       e8 24 12 fa ff          call   8052c40 <__lll_lock_wait>
 80b1a1c:       83 c4 10                add    $0x10,%esp
 80b1a1f:       eb 9d                   jmp    80b19be <__pthread_mutex_cond_lock+0x1ce>
 80b1a21:       8d 83 c8 11 fd ff       lea    -0x2ee38(%ebx),%eax
 80b1a27:       50                      push   %eax
 80b1a28:       68 82 00 00 00          push   $0x82
 80b1a2d:       e9 1b ff ff ff          jmp    80b194d <__pthread_mutex_cond_lock+0x15d>
 80b1a32:       8d 83 c8 11 fd ff       lea    -0x2ee38(%ebx),%eax
 80b1a38:       50                      push   %eax
 80b1a39:       8d 83 a7 d5 fc ff       lea    -0x32a59(%ebx),%eax
 80b1a3f:       68 ac 00 00 00          push   $0xac
 80b1a44:       50                      push   %eax
 80b1a45:       8d 83 f4 f0 fc ff       lea    -0x30f0c(%ebx),%eax
 80b1a4b:       50                      push   %eax
 80b1a4c:       e8 cf a6 f9 ff          call   804c120 <__libc_assert_fail>
 80b1a51:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80b1a58:       00
 80b1a59:       8d b4 26 00 00 00 00    lea    0x0(%esi,%eiz,1),%esi

080b1a60 <__pthread_mutex_cond_lock_adjust>:
 80b1a60:       e8 32 de f9 ff          call   804f897 <__x86.get_pc_thunk.cx>
 80b1a65:       81 c1 8f 55 03 00       add    $0x3558f,%ecx
 80b1a6b:       83 ec 0c                sub    $0xc,%esp
 80b1a6e:       8b 54 24 10             mov    0x10(%esp),%edx
 80b1a72:       8b 42 0c                mov    0xc(%edx),%eax
 80b1a75:       a8 20                   test   $0x20,%al
 80b1a77:       74 5d                   je     80b1ad6 <__pthread_mutex_cond_lock_adjust+0x76>
 80b1a79:       a8 10                   test   $0x10,%al
 80b1a7b:       75 3a                   jne    80b1ab7 <__pthread_mutex_cond_lock_adjust+0x57>
 80b1a7d:       a8 80                   test   $0x80,%al
 80b1a7f:       75 17                   jne    80b1a98 <__pthread_mutex_cond_lock_adjust+0x38>
 80b1a81:       65 8b 0d 68 00 00 00    mov    %gs:0x68,%ecx
 80b1a88:       89 4a 08                mov    %ecx,0x8(%edx)
 80b1a8b:       83 f8 21                cmp    $0x21,%eax
 80b1a8e:       75 04                   jne    80b1a94 <__pthread_mutex_cond_lock_adjust+0x34>
 80b1a90:       83 42 04 01             addl   $0x1,0x4(%edx)
 80b1a94:       83 c4 0c                add    $0xc,%esp
 80b1a97:       c3                      ret
 80b1a98:       8d 81 4c 17 fd ff       lea    -0x2e8b4(%ecx),%eax
 80b1a9e:       50                      push   %eax
 80b1a9f:       8d 81 a7 d5 fc ff       lea    -0x32a59(%ecx),%eax
 80b1aa5:       68 8c 02 00 00          push   $0x28c
 80b1aaa:       50                      push   %eax
 80b1aab:       8d 81 78 11 fd ff       lea    -0x2ee88(%ecx),%eax
 80b1ab1:       50                      push   %eax
 80b1ab2:       e8 69 a6 f9 ff          call   804c120 <__libc_assert_fail>
 80b1ab7:       8d 81 4c 17 fd ff       lea    -0x2e8b4(%ecx),%eax
 80b1abd:       50                      push   %eax
 80b1abe:       8d 81 a7 d5 fc ff       lea    -0x32a59(%ecx),%eax
 80b1ac4:       68 8b 02 00 00          push   $0x28b
 80b1ac9:       50                      push   %eax
 80b1aca:       8d 81 44 11 fd ff       lea    -0x2eebc(%ecx),%eax
 80b1ad0:       50                      push   %eax
 80b1ad1:       e8 4a a6 f9 ff          call   804c120 <__libc_assert_fail>
 80b1ad6:       8d 81 4c 17 fd ff       lea    -0x2e8b4(%ecx),%eax
 80b1adc:       50                      push   %eax
 80b1add:       8d 81 a7 d5 fc ff       lea    -0x32a59(%ecx),%eax
 80b1ae3:       68 8a 02 00 00          push   $0x28a
 80b1ae8:       50                      push   %eax
 80b1ae9:       8d 81 10 11 fd ff       lea    -0x2eef0(%ecx),%eax
 80b1aef:       50                      push   %eax
 80b1af0:       e8 2b a6 f9 ff          call   804c120 <__libc_assert_fail>
 80b1af5:       66 90                   xchg   %ax,%ax
 80b1af7:       66 90                   xchg   %ax,%ax
 80b1af9:       66 90                   xchg   %ax,%ax
 80b1afb:       66 90                   xchg   %ax,%ax
 80b1afd:       66 90                   xchg   %ax,%ax
 80b1aff:       90                      nop

080b1b00 <___pthread_cond_signal>:
 80b1b00:       e8 3c 7e f9 ff          call   8049941 <__x86.get_pc_thunk.ax>
 80b1b05:       05 ef 54 03 00          add    $0x354ef,%eax
 80b1b0a:       55                      push   %ebp
 80b1b0b:       57                      push   %edi
 80b1b0c:       56                      push   %esi
 80b1b0d:       53                      push   %ebx
 80b1b0e:       83 ec 3c                sub    $0x3c,%esp
 80b1b11:       8b 7c 24 50             mov    0x50(%esp),%edi
 80b1b15:       89 44 24 18             mov    %eax,0x18(%esp)
 80b1b19:       8b 47 24                mov    0x24(%edi),%eax
 80b1b1c:       89 c6                   mov    %eax,%esi
 80b1b1e:       c1 ee 03                shr    $0x3,%esi
 80b1b21:       0f 84 ae 00 00 00       je     80b1bd5 <___pthread_cond_signal+0xd5>
 80b1b27:       c1 e0 07                shl    $0x7,%eax
 80b1b2a:       8d 6f 20                lea    0x20(%edi),%ebp
 80b1b2d:       0f b6 c0                movzbl %al,%eax
 80b1b30:       89 44 24 14             mov    %eax,0x14(%esp)
 80b1b34:       8b 47 20                mov    0x20(%edi),%eax
 80b1b37:       a8 03                   test   $0x3,%al
 80b1b39:       0f 85 f1 02 00 00       jne    80b1e30 <___pthread_cond_signal+0x330>
 80b1b3f:       89 c2                   mov    %eax,%edx
 80b1b41:       83 ca 01                or     $0x1,%edx
 80b1b44:       f0 0f b1 55 00          lock cmpxchg %edx,0x0(%ebp)
 80b1b49:       75 ec                   jne    80b1b37 <___pthread_cond_signal+0x37>
 80b1b4b:       83 ec 0c                sub    $0xc,%esp
 80b1b4e:       57                      push   %edi
 80b1b4f:       e8 ac 50 ff ff          call   80a6c00 <__atomic_wide_counter_load_relaxed>
 80b1b54:       89 44 24 18             mov    %eax,0x18(%esp)
 80b1b58:       8b 74 24 18             mov    0x18(%esp),%esi
 80b1b5c:       89 54 24 1c             mov    %edx,0x1c(%esp)
 80b1b60:       83 c4 10                add    $0x10,%esp
 80b1b63:       89 f3                   mov    %esi,%ebx
 80b1b65:       f7 d3                   not    %ebx
 80b1b67:       83 e3 01                and    $0x1,%ebx
 80b1b6a:       8b 54 9f 18             mov    0x18(%edi,%ebx,4),%edx
 80b1b6e:       8d 43 04                lea    0x4(%ebx),%eax
 80b1b71:       85 d2                   test   %edx,%edx
 80b1b73:       74 6b                   je     80b1be0 <___pthread_cond_signal+0xe0>
 80b1b75:       8d 34 9d 00 00 00 00    lea    0x0(,%ebx,4),%esi
 80b1b7c:       89 74 24 08             mov    %esi,0x8(%esp)
 80b1b80:       f0 83 44 37 28 02       lock addl $0x2,0x28(%edi,%esi,1)
 80b1b86:       83 6c 87 08 01          subl   $0x1,0x8(%edi,%eax,4)
 80b1b8b:       8b 47 20                mov    0x20(%edi),%eax
 80b1b8e:       89 c1                   mov    %eax,%ecx
 80b1b90:       89 c2                   mov    %eax,%edx
 80b1b92:       83 e1 fc                and    $0xfffffffc,%ecx
 80b1b95:       f0 0f b1 4d 00          lock cmpxchg %ecx,0x0(%ebp)
 80b1b9a:       75 f2                   jne    80b1b8e <___pthread_cond_signal+0x8e>
 80b1b9c:       8b 4c 24 14             mov    0x14(%esp),%ecx
 80b1ba0:       83 e2 03                and    $0x3,%edx
 80b1ba3:       80 f1 81                xor    $0x81,%cl
 80b1ba6:       83 fa 02                cmp    $0x2,%edx
 80b1ba9:       0f 84 29 02 00 00       je     80b1dd8 <___pthread_cond_signal+0x2d8>
 80b1baf:       8b 44 24 08             mov    0x8(%esp),%eax
 80b1bb3:       ba 01 00 00 00          mov    $0x1,%edx
 80b1bb8:       31 f6                   xor    %esi,%esi
 80b1bba:       8d 5c 07 28             lea    0x28(%edi,%eax,1),%ebx
 80b1bbe:       b8 f0 00 00 00          mov    $0xf0,%eax
 80b1bc3:       65 ff 15 10 00 00 00    call   *%gs:0x10
 80b1bca:       3d 00 f0 ff ff          cmp    $0xfffff000,%eax
 80b1bcf:       0f 87 3b 02 00 00       ja     80b1e10 <___pthread_cond_signal+0x310>
 80b1bd5:       83 c4 3c                add    $0x3c,%esp
 80b1bd8:       31 c0                   xor    %eax,%eax
 80b1bda:       5b                      pop    %ebx
 80b1bdb:       5e                      pop    %esi
 80b1bdc:       5f                      pop    %edi
 80b1bdd:       5d                      pop    %ebp
 80b1bde:       c3                      ret
 80b1bdf:       90                      nop
 80b1be0:       8b 4f 20                mov    0x20(%edi),%ecx
 80b1be3:       8d 57 08                lea    0x8(%edi),%edx
 80b1be6:       83 ec 0c                sub    $0xc,%esp
 80b1be9:       83 e6 01                and    $0x1,%esi
 80b1bec:       89 54 24 34             mov    %edx,0x34(%esp)
 80b1bf0:       c1 e9 02                shr    $0x2,%ecx
 80b1bf3:       89 4c 24 28             mov    %ecx,0x28(%esp)
 80b1bf7:       52                      push   %edx
 80b1bf8:       e8 03 50 ff ff          call   80a6c00 <__atomic_wide_counter_load_relaxed>
 80b1bfd:       89 74 24 34             mov    %esi,0x34(%esp)
 80b1c01:       8d 34 b7                lea    (%edi,%esi,4),%esi
 80b1c04:       89 c1                   mov    %eax,%ecx
 80b1c06:       8b 44 24 18             mov    0x18(%esp),%eax
 80b1c0a:       0f ac d1 01             shrd   $0x1,%edx,%ecx
 80b1c0e:       8b 54 24 1c             mov    0x1c(%esp),%edx
 80b1c12:       89 4c 24 30             mov    %ecx,0x30(%esp)
 80b1c16:       8b 4c 24 2c             mov    0x2c(%esp),%ecx
 80b1c1a:       83 c4 10                add    $0x10,%esp
 80b1c1d:       0f ac d0 01             shrd   $0x1,%edx,%eax
 80b1c21:       03 46 18                add    0x18(%esi),%eax
 80b1c24:       29 c8                   sub    %ecx,%eax
 80b1c26:       39 44 24 20             cmp    %eax,0x20(%esp)
 80b1c2a:       75 54                   jne    80b1c80 <___pthread_cond_signal+0x180>
 80b1c2c:       8b 47 20                mov    0x20(%edi),%eax
 80b1c2f:       89 c1                   mov    %eax,%ecx
 80b1c31:       89 c2                   mov    %eax,%edx
 80b1c33:       83 e1 fc                and    $0xfffffffc,%ecx
 80b1c36:       f0 0f b1 4d 00          lock cmpxchg %ecx,0x0(%ebp)
 80b1c3b:       75 f2                   jne    80b1c2f <___pthread_cond_signal+0x12f>
 80b1c3d:       83 e2 03                and    $0x3,%edx
 80b1c40:       83 fa 02                cmp    $0x2,%edx
 80b1c43:       75 90                   jne    80b1bd5 <___pthread_cond_signal+0xd5>
 80b1c45:       8b 4c 24 14             mov    0x14(%esp),%ecx
 80b1c49:       b8 f0 00 00 00          mov    $0xf0,%eax
 80b1c4e:       ba 01 00 00 00          mov    $0x1,%edx
 80b1c53:       31 f6                   xor    %esi,%esi
 80b1c55:       89 eb                   mov    %ebp,%ebx
 80b1c57:       80 f1 81                xor    $0x81,%cl
 80b1c5a:       65 ff 15 10 00 00 00    call   *%gs:0x10
 80b1c61:       3d 00 f0 ff ff          cmp    $0xfffff000,%eax
 80b1c66:       0f 86 69 ff ff ff       jbe    80b1bd5 <___pthread_cond_signal+0xd5>
 80b1c6c:       83 c0 16                add    $0x16,%eax
 80b1c6f:       83 e0 f7                and    $0xfffffff7,%eax
 80b1c72:       0f 84 5d ff ff ff       je     80b1bd5 <___pthread_cond_signal+0xd5>
 80b1c78:       e9 9f 01 00 00          jmp    80b1e1c <___pthread_cond_signal+0x31c>
 80b1c7d:       8d 76 00                lea    0x0(%esi),%esi
 80b1c80:       c1 e3 02                shl    $0x2,%ebx
 80b1c83:       8d 44 1f 28             lea    0x28(%edi,%ebx,1),%eax
 80b1c87:       89 44 24 08             mov    %eax,0x8(%esp)
 80b1c8b:       f0 83 08 01             lock orl $0x1,(%eax)
 80b1c8f:       8d 5c 1f 10             lea    0x10(%edi,%ebx,1),%ebx
 80b1c93:       8b 03                   mov    (%ebx),%eax
 80b1c95:       89 c2                   mov    %eax,%edx
 80b1c97:       f0 0f b1 03             lock cmpxchg %eax,(%ebx)
 80b1c9b:       75 f8                   jne    80b1c95 <___pthread_cond_signal+0x195>
 80b1c9d:       8b 4c 24 14             mov    0x14(%esp),%ecx
 80b1ca1:       80 f1 80                xor    $0x80,%cl
 80b1ca4:       d1 ea                   shr    $1,%edx
 80b1ca6:       74 40                   je     80b1ce8 <___pthread_cond_signal+0x1e8>
 80b1ca8:       89 6c 24 2c             mov    %ebp,0x2c(%esp)
 80b1cac:       89 f5                   mov    %esi,%ebp
 80b1cae:       66 90                   xchg   %ax,%ax
 80b1cb0:       8b 03                   mov    (%ebx),%eax
 80b1cb2:       89 c2                   mov    %eax,%edx
 80b1cb4:       83 ca 01                or     $0x1,%edx
 80b1cb7:       f0 0f b1 13             lock cmpxchg %edx,(%ebx)
 80b1cbb:       75 f5                   jne    80b1cb2 <___pthread_cond_signal+0x1b2>
 80b1cbd:       89 d0                   mov    %edx,%eax
 80b1cbf:       d1 e8                   shr    $1,%eax
 80b1cc1:       74 19                   je     80b1cdc <___pthread_cond_signal+0x1dc>
 80b1cc3:       31 f6                   xor    %esi,%esi
 80b1cc5:       b8 f0 00 00 00          mov    $0xf0,%eax
 80b1cca:       65 ff 15 10 00 00 00    call   *%gs:0x10
 80b1cd1:       3d 00 f0 ff ff          cmp    $0xfffff000,%eax
 80b1cd6:       0f 87 e4 00 00 00       ja     80b1dc0 <___pthread_cond_signal+0x2c0>
 80b1cdc:       8b 03                   mov    (%ebx),%eax
 80b1cde:       d1 e8                   shr    $1,%eax
 80b1ce0:       75 ce                   jne    80b1cb0 <___pthread_cond_signal+0x1b0>
 80b1ce2:       89 ee                   mov    %ebp,%esi
 80b1ce4:       8b 6c 24 2c             mov    0x2c(%esp),%ebp
 80b1ce8:       8b 44 24 24             mov    0x24(%esp),%eax
 80b1cec:       f7 d8                   neg    %eax
 80b1cee:       19 c0                   sbb    %eax,%eax
 80b1cf0:       83 ec 08                sub    $0x8,%esp
 80b1cf3:       8b 4c 24 24             mov    0x24(%esp),%ecx
 80b1cf7:       83 c8 01                or     $0x1,%eax
 80b1cfa:       8d 04 48                lea    (%eax,%ecx,2),%eax
 80b1cfd:       50                      push   %eax
 80b1cfe:       ff 74 24 34             push   0x34(%esp)
 80b1d02:       e8 a9 4e ff ff          call   80a6bb0 <__atomic_wide_counter_fetch_add_relaxed>
 80b1d07:       8b 44 24 18             mov    0x18(%esp),%eax
 80b1d0b:       83 c4 10                add    $0x10,%esp
 80b1d0e:       c7 00 00 00 00 00       movl   $0x0,(%eax)
 80b1d14:       8d 47 04                lea    0x4(%edi),%eax
 80b1d17:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80b1d1e:       00
 80b1d1f:       90                      nop
 80b1d20:       8b 08                   mov    (%eax),%ecx
 80b1d22:       8b 17                   mov    (%edi),%edx
 80b1d24:       8b 18                   mov    (%eax),%ebx
 80b1d26:       39 d9                   cmp    %ebx,%ecx
 80b1d28:       75 f6                   jne    80b1d20 <___pthread_cond_signal+0x220>
 80b1d2a:       89 c8                   mov    %ecx,%eax
 80b1d2c:       f7 d0                   not    %eax
 80b1d2e:       21 d0                   and    %edx,%eax
 80b1d30:       c1 e8 1f                shr    $0x1f,%eax
 80b1d33:       01 c8                   add    %ecx,%eax
 80b1d35:       25 ff ff ff 7f          and    $0x7fffffff,%eax
 80b1d3a:       89 c3                   mov    %eax,%ebx
 80b1d3c:       8b 07                   mov    (%edi),%eax
 80b1d3e:       89 54 24 08             mov    %edx,0x8(%esp)
 80b1d42:       89 c2                   mov    %eax,%edx
 80b1d44:       89 c1                   mov    %eax,%ecx
 80b1d46:       83 f2 01                xor    $0x1,%edx
 80b1d49:       f0 0f b1 17             lock cmpxchg %edx,(%edi)
 80b1d4d:       75 f3                   jne    80b1d42 <___pthread_cond_signal+0x242>
 80b1d4f:       8b 54 24 08             mov    0x8(%esp),%edx
 80b1d53:       81 e1 ff ff ff 7f       and    $0x7fffffff,%ecx
 80b1d59:       81 e2 ff ff ff 7f       and    $0x7fffffff,%edx
 80b1d5f:       39 d1                   cmp    %edx,%ecx
 80b1d61:       83 d3 00                adc    $0x0,%ebx
 80b1d64:       31 d2                   xor    %edx,%edx
 80b1d66:       89 d8                   mov    %ebx,%eax
 80b1d68:       0f a4 da 1f             shld   $0x1f,%ebx,%edx
 80b1d6c:       c1 e0 1f                shl    $0x1f,%eax
 80b1d6f:       01 c8                   add    %ecx,%eax
 80b1d71:       83 d2 00                adc    $0x0,%edx
 80b1d74:       0f ac d0 01             shrd   $0x1,%edx,%eax
 80b1d78:       8b 54 24 1c             mov    0x1c(%esp),%edx
 80b1d7c:       89 c1                   mov    %eax,%ecx
 80b1d7e:       8b 44 24 20             mov    0x20(%esp),%eax
 80b1d82:       01 c2                   add    %eax,%edx
 80b1d84:       89 c8                   mov    %ecx,%eax
 80b1d86:       29 d0                   sub    %edx,%eax
 80b1d88:       8b 57 20                mov    0x20(%edi),%edx
 80b1d8b:       8d 0c 85 00 00 00 00    lea    0x0(,%eax,4),%ecx
 80b1d92:       83 e2 03                and    $0x3,%edx
 80b1d95:       09 ca                   or     %ecx,%edx
 80b1d97:       89 d3                   mov    %edx,%ebx
 80b1d99:       87 5d 00                xchg   %ebx,0x0(%ebp)
 80b1d9c:       31 da                   xor    %ebx,%edx
 80b1d9e:       83 e2 03                and    $0x3,%edx
 80b1da1:       75 65                   jne    80b1e08 <___pthread_cond_signal+0x308>
 80b1da3:       01 46 18                add    %eax,0x18(%esi)
 80b1da6:       0f 84 80 fe ff ff       je     80b1c2c <___pthread_cond_signal+0x12c>
 80b1dac:       8b 5c 24 24             mov    0x24(%esp),%ebx
 80b1db0:       8d 43 04                lea    0x4(%ebx),%eax
 80b1db3:       e9 bd fd ff ff          jmp    80b1b75 <___pthread_cond_signal+0x75>
 80b1db8:       2e 8d b4 26 00 00 00    lea    %cs:0x0(%esi,%eiz,1),%esi
 80b1dbf:       00
 80b1dc0:       83 f8 f5                cmp    $0xfffffff5,%eax
 80b1dc3:       0f 84 13 ff ff ff       je     80b1cdc <___pthread_cond_signal+0x1dc>
 80b1dc9:       83 f8 fc                cmp    $0xfffffffc,%eax
 80b1dcc:       0f 84 0a ff ff ff       je     80b1cdc <___pthread_cond_signal+0x1dc>
 80b1dd2:       eb 48                   jmp    80b1e1c <___pthread_cond_signal+0x31c>
 80b1dd4:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi
 80b1dd8:       b8 f0 00 00 00          mov    $0xf0,%eax
 80b1ddd:       ba 01 00 00 00          mov    $0x1,%edx
 80b1de2:       31 f6                   xor    %esi,%esi
 80b1de4:       89 eb                   mov    %ebp,%ebx
 80b1de6:       65 ff 15 10 00 00 00    call   *%gs:0x10
 80b1ded:       3d 00 f0 ff ff          cmp    $0xfffff000,%eax
 80b1df2:       0f 86 b7 fd ff ff       jbe    80b1baf <___pthread_cond_signal+0xaf>
 80b1df8:       83 c0 16                add    $0x16,%eax
 80b1dfb:       83 e0 f7                and    $0xfffffff7,%eax
 80b1dfe:       75 1c                   jne    80b1e1c <___pthread_cond_signal+0x31c>
 80b1e00:       e9 aa fd ff ff          jmp    80b1baf <___pthread_cond_signal+0xaf>
 80b1e05:       8d 76 00                lea    0x0(%esi),%esi
 80b1e08:       83 c9 02                or     $0x2,%ecx
 80b1e0b:       89 4f 20                mov    %ecx,0x20(%edi)
 80b1e0e:       eb 93                   jmp    80b1da3 <___pthread_cond_signal+0x2a3>
 80b1e10:       83 c0 16                add    $0x16,%eax
 80b1e13:       83 e0 f7                and    $0xfffffff7,%eax
 80b1e16:       0f 84 b9 fd ff ff       je     80b1bd5 <___pthread_cond_signal+0xd5>
 80b1e1c:       83 ec 0c                sub    $0xc,%esp
 80b1e1f:       8b 5c 24 24             mov    0x24(%esp),%ebx
 80b1e23:       8d 83 ec d6 fc ff       lea    -0x32914(%ebx),%eax
 80b1e29:       50                      push   %eax
 80b1e2a:       e8 31 b1 f9 ff          call   804cf60 <__libc_fatal>
 80b1e2f:       90                      nop
 80b1e30:       8b 4c 24 14             mov    0x14(%esp),%ecx
 80b1e34:       80 f1 80                xor    $0x80,%cl
 80b1e37:       eb 0a                   jmp    80b1e43 <___pthread_cond_signal+0x343>
 80b1e39:       8d b4 26 00 00 00 00    lea    0x0(%esi,%eiz,1),%esi
 80b1e40:       8b 45 00                mov    0x0(%ebp),%eax
 80b1e43:       89 c2                   mov    %eax,%edx
 80b1e45:       83 e2 03                and    $0x3,%edx
 80b1e48:       83 fa 02                cmp    $0x2,%edx
 80b1e4b:       74 24                   je     80b1e71 <___pthread_cond_signal+0x371>
 80b1e4d:       89 c2                   mov    %eax,%edx
 80b1e4f:       83 e2 fc                and    $0xfffffffc,%edx
 80b1e52:       83 ca 02                or     $0x2,%edx
 80b1e55:       f0 0f b1 55 00          lock cmpxchg %edx,0x0(%ebp)
 80b1e5a:       0f 94 c3                sete   %bl
 80b1e5d:       89 c2                   mov    %eax,%edx
 80b1e5f:       0f b6 db                movzbl %bl,%ebx
 80b1e62:       83 e2 03                and    $0x3,%edx
 80b1e65:       85 db                   test   %ebx,%ebx
 80b1e67:       74 da                   je     80b1e43 <___pthread_cond_signal+0x343>
 80b1e69:       85 d2                   test   %edx,%edx
 80b1e6b:       0f 84 da fc ff ff       je     80b1b4b <___pthread_cond_signal+0x4b>
 80b1e71:       83 e0 fc                and    $0xfffffffc,%eax
 80b1e74:       31 f6                   xor    %esi,%esi
 80b1e76:       89 eb                   mov    %ebp,%ebx
 80b1e78:       89 c2                   mov    %eax,%edx
 80b1e7a:       b8 f0 00 00 00          mov    $0xf0,%eax
 80b1e7f:       83 ca 02                or     $0x2,%edx
 80b1e82:       65 ff 15 10 00 00 00    call   *%gs:0x10
 80b1e89:       3d 00 f0 ff ff          cmp    $0xfffff000,%eax
 80b1e8e:       76 b0                   jbe    80b1e40 <___pthread_cond_signal+0x340>
 80b1e90:       83 f8 f5                cmp    $0xfffffff5,%eax
 80b1e93:       74 ab                   je     80b1e40 <___pthread_cond_signal+0x340>
 80b1e95:       83 f8 fc                cmp    $0xfffffffc,%eax
 80b1e98:       74 a6                   je     80b1e40 <___pthread_cond_signal+0x340>
 80b1e9a:       eb 80                   jmp    80b1e1c <___pthread_cond_signal+0x31c>

Disassembly of section .fini:

080b1e9c <_fini>:
 80b1e9c:       53                      push   %ebx
 80b1e9d:       83 ec 08                sub    $0x8,%esp
 80b1ea0:       e8 3b 79 f9 ff          call   80497e0 <__x86.get_pc_thunk.bx>
 80b1ea5:       81 c3 4f 51 03 00       add    $0x3514f,%ebx
 80b1eab:       83 c4 08                add    $0x8,%esp
 80b1eae:       5b                      pop    %ebx
 80b1eaf:       c3                      ret
```
``` 
```sh
nathan@ServeurDebian:~/work$ objdump -d hello4

hello4:     file format elf64-littleaarch64


Disassembly of section .init:

0000000000000630 <_init>:
 630:   d503233f        paciasp
 634:   a9bf7bfd        stp     x29, x30, [sp, #-16]!
 638:   910003fd        mov     x29, sp
 63c:   9400002e        bl      6f4 <call_weak_fn>
 640:   a8c17bfd        ldp     x29, x30, [sp], #16
 644:   d50323bf        autiasp
 648:   d65f03c0        ret

Disassembly of section .plt:

0000000000000650 <.plt>:
 650:   a9bf7bf0        stp     x16, x30, [sp, #-16]!
 654:   f00000f0        adrp    x16, 1f000 <__FRAME_END__+0x1e6d8>
 658:   f947fe11        ldr     x17, [x16, #4088]
 65c:   913fe210        add     x16, x16, #0xff8
 660:   d61f0220        br      x17
 664:   d503201f        nop
 668:   d503201f        nop
 66c:   d503201f        nop

0000000000000670 <__libc_start_main@plt>:
 670:   90000110        adrp    x16, 20000 <__libc_start_main@GLIBC_2.34>
 674:   f9400211        ldr     x17, [x16]
 678:   91000210        add     x16, x16, #0x0
 67c:   d61f0220        br      x17

0000000000000680 <__cxa_finalize@plt>:
 680:   90000110        adrp    x16, 20000 <__libc_start_main@GLIBC_2.34>
 684:   f9400611        ldr     x17, [x16, #8]
 688:   91002210        add     x16, x16, #0x8
 68c:   d61f0220        br      x17

0000000000000690 <__gmon_start__@plt>:
 690:   90000110        adrp    x16, 20000 <__libc_start_main@GLIBC_2.34>
 694:   f9400a11        ldr     x17, [x16, #16]
 698:   91004210        add     x16, x16, #0x10
 69c:   d61f0220        br      x17

00000000000006a0 <abort@plt>:
 6a0:   90000110        adrp    x16, 20000 <__libc_start_main@GLIBC_2.34>
 6a4:   f9400e11        ldr     x17, [x16, #24]
 6a8:   91006210        add     x16, x16, #0x18
 6ac:   d61f0220        br      x17

00000000000006b0 <puts@plt>:
 6b0:   90000110        adrp    x16, 20000 <__libc_start_main@GLIBC_2.34>
 6b4:   f9401211        ldr     x17, [x16, #32]
 6b8:   91008210        add     x16, x16, #0x20
 6bc:   d61f0220        br      x17

Disassembly of section .text:

00000000000006c0 <_start>:
 6c0:   d503245f        bti     c
 6c4:   d280001d        mov     x29, #0x0                       // #0
 6c8:   d280001e        mov     x30, #0x0                       // #0
 6cc:   aa0003e5        mov     x5, x0
 6d0:   f94003e1        ldr     x1, [sp]
 6d4:   910023e2        add     x2, sp, #0x8
 6d8:   910003e6        mov     x6, sp
 6dc:   f00000e0        adrp    x0, 1f000 <__FRAME_END__+0x1e6d8>
 6e0:   f947ec00        ldr     x0, [x0, #4056]
 6e4:   d2800003        mov     x3, #0x0                        // #0
 6e8:   d2800004        mov     x4, #0x0                        // #0
 6ec:   97ffffe1        bl      670 <__libc_start_main@plt>
 6f0:   97ffffec        bl      6a0 <abort@plt>

00000000000006f4 <call_weak_fn>:
 6f4:   f00000e0        adrp    x0, 1f000 <__FRAME_END__+0x1e6d8>
 6f8:   f947e800        ldr     x0, [x0, #4048]
 6fc:   b4000040        cbz     x0, 704 <call_weak_fn+0x10>
 700:   17ffffe4        b       690 <__gmon_start__@plt>
 704:   d65f03c0        ret
 708:   d503201f        nop
 70c:   d503201f        nop
 710:   d503201f        nop
 714:   d503201f        nop
 718:   d503201f        nop
 71c:   d503201f        nop

0000000000000720 <deregister_tm_clones>:
 720:   90000100        adrp    x0, 20000 <__libc_start_main@GLIBC_2.34>
 724:   9100e000        add     x0, x0, #0x38
 728:   90000101        adrp    x1, 20000 <__libc_start_main@GLIBC_2.34>
 72c:   9100e021        add     x1, x1, #0x38
 730:   eb00003f        cmp     x1, x0
 734:   540000c0        b.eq    74c <deregister_tm_clones+0x2c>  // b.none
 738:   f00000e1        adrp    x1, 1f000 <__FRAME_END__+0x1e6d8>
 73c:   f947e021        ldr     x1, [x1, #4032]
 740:   b4000061        cbz     x1, 74c <deregister_tm_clones+0x2c>
 744:   aa0103f0        mov     x16, x1
 748:   d61f0200        br      x16
 74c:   d65f03c0        ret

0000000000000750 <register_tm_clones>:
 750:   90000100        adrp    x0, 20000 <__libc_start_main@GLIBC_2.34>
 754:   9100e000        add     x0, x0, #0x38
 758:   90000101        adrp    x1, 20000 <__libc_start_main@GLIBC_2.34>
 75c:   9100e021        add     x1, x1, #0x38
 760:   cb000021        sub     x1, x1, x0
 764:   d37ffc22        lsr     x2, x1, #63
 768:   8b810c41        add     x1, x2, x1, asr #3
 76c:   9341fc21        asr     x1, x1, #1
 770:   b40000c1        cbz     x1, 788 <register_tm_clones+0x38>
 774:   f00000e2        adrp    x2, 1f000 <__FRAME_END__+0x1e6d8>
 778:   f947f042        ldr     x2, [x2, #4064]
 77c:   b4000062        cbz     x2, 788 <register_tm_clones+0x38>
 780:   aa0203f0        mov     x16, x2
 784:   d61f0200        br      x16
 788:   d65f03c0        ret

000000000000078c <__do_global_dtors_aux>:
 78c:   d503233f        paciasp
 790:   a9be7bfd        stp     x29, x30, [sp, #-32]!
 794:   910003fd        mov     x29, sp
 798:   f9000bf3        str     x19, [sp, #16]
 79c:   90000113        adrp    x19, 20000 <__libc_start_main@GLIBC_2.34>
 7a0:   3940e260        ldrb    w0, [x19, #56]
 7a4:   37000140        tbnz    w0, #0, 7cc <__do_global_dtors_aux+0x40>
 7a8:   f00000e0        adrp    x0, 1f000 <__FRAME_END__+0x1e6d8>
 7ac:   f947e400        ldr     x0, [x0, #4040]
 7b0:   b4000080        cbz     x0, 7c0 <__do_global_dtors_aux+0x34>
 7b4:   90000100        adrp    x0, 20000 <__libc_start_main@GLIBC_2.34>
 7b8:   f9401800        ldr     x0, [x0, #48]
 7bc:   97ffffb1        bl      680 <__cxa_finalize@plt>
 7c0:   97ffffd8        bl      720 <deregister_tm_clones>
 7c4:   52800020        mov     w0, #0x1                        // #1
 7c8:   3900e260        strb    w0, [x19, #56]
 7cc:   f9400bf3        ldr     x19, [sp, #16]
 7d0:   a8c27bfd        ldp     x29, x30, [sp], #32
 7d4:   d50323bf        autiasp
 7d8:   d65f03c0        ret
 7dc:   d503201f        nop

00000000000007e0 <frame_dummy>:
 7e0:   d503245f        bti     c
 7e4:   17ffffdb        b       750 <register_tm_clones>

00000000000007e8 <main>:
 7e8:   a9bf7bfd        stp     x29, x30, [sp, #-16]!
 7ec:   910003fd        mov     x29, sp
 7f0:   90000000        adrp    x0, 0 <__abi_tag-0x2ec>
 7f4:   9120a000        add     x0, x0, #0x828
 7f8:   97ffffae        bl      6b0 <puts@plt>
 7fc:   52800000        mov     w0, #0x0                        // #0
 800:   a8c17bfd        ldp     x29, x30, [sp], #16
 804:   d65f03c0        ret

Disassembly of section .fini:

0000000000000808 <_fini>:
 808:   d503233f        paciasp
 80c:   a9bf7bfd        stp     x29, x30, [sp, #-16]!
 810:   910003fd        mov     x29, sp
 814:   a8c17bfd        ldp     x29, x30, [sp], #16
 818:   d50323bf        autiasp
 81c:   d65f03c0        ret
```
🌞 Essayez d'exécuter le programme hello4
```sh
nathan@ServeurDebian:~/work$ ./hello4
-bash: ./hello4: cannot execute binary file: Exec format error
```


## II. Ptits challenges de cracking
🌞 Avec une commande apt search, déterminez si le paquet ghidra est disponible
```sh
nathan@ServeurDebian:~/work$ apt search ghidra
Sorting... Done
Full Text Search... Done
```
🌞 Ajouter l'URL des dépôts Kali à vos dépôts existants

```sh
nathan@ServeurDebian:~$ sudo nano /etc/apt/sources.list

#deb cdrom:[Debian GNU/Linux 12.7.0 _Bookworm_ - Official amd64 NETINST with firmware 20240831-10:38]/ bookworm contrib main non-free-firmware

deb http://deb.debian.org/debian/ bookworm main non-free-firmware
deb-src http://deb.debian.org/debian/ bookworm main non-free-firmware

deb http://security.debian.org/debian-security bookworm-security main non-free-firmware
deb-src http://security.debian.org/debian-security bookworm-security main non-free-firmware

# bookworm-updates, to get updates before a point release is made;
# see https://www.debian.org/doc/manuals/debian-reference/ch02.en.html#_updates_and_backports
deb http://deb.debian.org/debian/ bookworm-updates main non-free-firmware
deb-src http://deb.debian.org/debian/ bookworm-updates main non-free-firmware
deb http://http.kali.org/kali kali-rolling main contrib non-free non-free-firmware

# This system was installed using small removable media
# (e.g. netinst, live or single CD). The matching "deb cdrom"
# entries were disabled at the end of the installation process.
# For information about how to configure apt package sources,
# see the sources.list(5) manual.


















                                                                     [ Read 19 lines ]
^G Help          ^O Write Out     ^W Where Is      ^K Cut           ^T Execute       ^C Location      M-U Undo         M-A Set Mark     M-] To Bracket
^X Exit          ^R Read File     ^\ Replace       ^U Paste         ^J Justify       ^/ Go To Line    M-E Redo         M-6 Copy         ^Q Where Was        
```
🌞 Ajoutez la clé publique des gars de chez Kali
```sh
nathan@ServeurDebian:~/work$ sudo wget https://archive.kali.org/archive-key.asc -O /etc/apt/trusted.gpg.d/kali-archive-keyring.asc
--2024-11-27 14:11:18--  https://archive.kali.org/archive-key.asc
Resolving archive.kali.org (archive.kali.org)... 192.99.45.140, 2607:5300:60:508c::
Connecting to archive.kali.org (archive.kali.org)|192.99.45.140|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3155 (3.1K) [application/octet-stream]
Saving to: ‘/etc/apt/trusted.gpg.d/kali-archive-keyring.asc’

/etc/apt/trusted.gpg.d/kali-archive-ke 100%[============================================================================>]   3.08K  --.-KB/s    in 0s

2024-11-27 14:11:18 (111 MB/s) - ‘/etc/apt/trusted.gpg.d/kali-archive-keyring.asc’ saved [3155/3155]
```

🌞 Mettez à jour la liste des dépôts que votre OS connaît actuellement
```sh
nathan@ServeurDebian:~/work$ sudo apt update -y
Hit:1 https://packages.mozilla.org/apt mozilla InRelease
Hit:2 http://deb.debian.org/debian bookworm InRelease
Hit:3 http://deb.debian.org/debian bookworm-updates InRelease
Hit:4 http://security.debian.org/debian-security bookworm-security InRelease
Get:5 http://kali.download/kali kali-rolling InRelease [41.5 kB]
Get:6 http://kali.download/kali kali-rolling/main amd64 Packages [20.3 MB]
Get:7 http://kali.download/kali kali-rolling/contrib amd64 Packages [102 kB]
Get:8 http://kali.download/kali kali-rolling/non-free amd64 Packages [196 kB]
Get:9 http://kali.download/kali kali-rolling/non-free-firmware amd64 Packages [10.6 kB]
Fetched 20.6 MB in 16s (1,301 kB/s)
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
961 packages can be upgraded. Run 'apt list --upgradable' to see them.
```
🌞 Avec une commande apt search, déterminez si le paquet ghidra est disponible
```sh
nathan@ServeurDebian:~/work$ apt search ghidra
Sorting... Done
Full Text Search... Done
ghidra/kali-rolling 11.0+ds-0kali1 amd64
  Software Reverse Engineering Framework

ghidra-data/kali-rolling 10.5-0kali1 all
  FID databases for Ghidra

ghidra-dbgsym/kali-rolling 11.0+ds-0kali1 amd64
  debug symbols for ghidra

quark-engine/kali-rolling 23.9.1-0kali2 all
  Android Malware (Analysis | Scoring System)

rz-ghidra/kali-rolling 0.7.0-0kali1+b1 amd64
  ghidra decompiler and sleigh disassembler for rizin

rz-ghidra-dbgsym/kali-rolling 0.7.0-0kali1+b1 amd64
  debug symbols for rz-ghidra
```
🌞 Installez le paquet ghidra
```sh
nathan@ServeurDebian:~/work$ sudo apt-get install ghidra
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  bsdextrautils bsdutils eject fdisk ghidra-data libblkid1 libfdisk1 libice-dev libmount1 libsm-dev libsmartcols1 libuuid1 libx11-dev libxau-dev
  libxcb1-dev libxdmcp-dev libxt-dev login mount openjdk-17-jdk openjdk-17-jdk-headless util-linux util-linux-extra util-linux-locales uuid-dev
  x11proto-dev xorg-sgml-doctools xtrans-dev
Suggested packages:
  libice-doc cryptsetup-bin libsm-doc uuid-runtime libx11-doc libxcb-doc libxt-doc nfs-common openjdk-17-demo openjdk-17-source visualvm wtmpdb
The following NEW packages will be installed:
  ghidra ghidra-data libice-dev libsm-dev libx11-dev libxau-dev libxcb1-dev libxdmcp-dev libxt-dev openjdk-17-jdk openjdk-17-jdk-headless uuid-dev
  x11proto-dev xorg-sgml-doctools xtrans-dev
The following packages will be upgraded:
  bsdextrautils bsdutils eject fdisk libblkid1 libfdisk1 libmount1 libsmartcols1 libuuid1 login mount util-linux util-linux-extra util-linux-locales
14 upgraded, 15 newly installed, 0 to remove and 947 not upgraded.
Need to get 503 MB of archives.
After this operation, 1,251 MB of additional disk space will be used.
Do you want to continue? [Y/n] Y
Get:1 http://kali.download/kali kali-rolling/main amd64 bsdutils amd64 1:2.40.2-11 [105 kB]
Get:2 http://http.kali.org/kali kali-rolling/main amd64 login amd64 1:4.16.0-2+really2.40.2-11 [81.5 kB]
Get:4 http://kali.download/kali kali-rolling/main amd64 bsdextrautils amd64 2.40.2-11 [91.5 kB]
Get:6 http://kali.download/kali kali-rolling/main amd64 fdisk amd64 2.40.2-11 [153 kB]
Get:7 http://kali.download/kali kali-rolling/main amd64 libsmartcols1 amd64 2.40.2-11 [139 kB]
Get:9 http://kali.download/kali kali-rolling/main amd64 libmount1 amd64 2.40.2-11 [199 kB]
Get:15 http://http.kali.org/kali kali-rolling/main amd64 openjdk-17-jdk amd64 17.0.13+11-2 [2,388 kB]
Get:16 http://http.kali.org/kali kali-rolling/main amd64 ghidra amd64 11.0+ds-0kali1 [343 MB]
Get:8 http://archive-4.kali.org/kali kali-rolling/main amd64 libblkid1 amd64 2.40.2-11 [168 kB]
Get:12 http://archive-4.kali.org/kali kali-rolling/main amd64 util-linux amd64 2.40.2-11 [1,213 kB]
Get:23 http://http.kali.org/kali kali-rolling/main amd64 libxdmcp-dev amd64 1:1.1.2-3+b2 [40.8 kB]
Get:3 http://ftp.free.fr/pub/kali kali-rolling/main amd64 eject amd64 2.40.2-11 [59.5 kB]
Get:5 http://ftp.free.fr/pub/kali kali-rolling/main amd64 util-linux-extra amd64 2.40.2-11 [265 kB]
Get:10 http://ftp.free.fr/pub/kali kali-rolling/main amd64 mount amd64 2.40.2-11 [155 kB]
Get:11 http://ftp.free.fr/pub/kali kali-rolling/main amd64 libuuid1 amd64 2.40.2-11 [35.9 kB]
Get:13 http://ftp.free.fr/pub/kali kali-rolling/main amd64 libfdisk1 amd64 2.40.2-11 [210 kB]
Get:14 http://http.kali.org/kali kali-rolling/main amd64 openjdk-17-jdk-headless amd64 17.0.13+11-2 [71.6 MB]
Get:28 http://deb.debian.org/debian bookworm/main amd64 xorg-sgml-doctools all 1:1.11-1.1 [22.1 kB]
Get:29 http://deb.debian.org/debian bookworm/main amd64 xtrans-dev all 1.4.0-1 [98.7 kB]
71% [14 openjdk-17-jdk-headless 34.1 MB/71.6 MB 48%] [16 ghidra 334 MB/343 MB 97%]                                                           8,756 kB/s 14s^Get:17 http://kali.download/kali kali-rolling/main amd64 ghidra-data all 10.5-0kali1 [78.1 MB]
77% [14 openjdk-17-jdk-headless 36.7 MB/71.6 MB 51%] [17 ghidra-data 19.0 MB/78.1 MB 24%]                                                    7,184 kB/s 13s^78% [14 openjdk-17-jdk-headless 37.5 MB/71.6 MB 52%] [17 ghidra-data 26.4 MB/78.1 MB 34%]                                                    7,184 kB/s 12s^Get:19 http://kali.download/kali kali-rolling/main amd64 libice-dev amd64 2:1.1.1-1 [73.8 kB]
Get:21 http://kali.download/kali kali-rolling/main amd64 libsm-dev amd64 2:1.2.4-1 [37.7 kB]
Get:22 http://kali.download/kali kali-rolling/main amd64 libxau-dev amd64 1:1.0.11-1 [23.6 kB]
Get:24 http://http.kali.org/kali kali-rolling/main amd64 libxcb1-dev amd64 1.17.0-2+b1 [181 kB]
Get:25 http://kali.download/kali kali-rolling/main amd64 libx11-dev amd64 2:1.8.10-2 [891 kB]
Get:26 http://http.kali.org/kali kali-rolling/main amd64 libxt-dev amd64 1:1.2.1-1.2+b1 [407 kB]
Get:27 http://kali.download/kali kali-rolling/main amd64 util-linux-locales all 2.40.2-11 [2,905 kB]
Get:18 http://ftp.free.fr/pub/kali kali-rolling/main amd64 x11proto-dev all 2024.1-1 [603 kB]
Get:20 http://ftp.free.fr/pub/kali kali-rolling/main amd64 uuid-dev amd64 2.40.2-11 [47.2 kB]
Fetched 503 MB in 1min 8s (7,386 kB/s)
(Reading database ... 129246 files and directories currently installed.)
Preparing to unpack .../bsdutils_1%3a2.40.2-11_amd64.deb ...
Unpacking bsdutils (1:2.40.2-11) over (1:2.40.2-9) ...
Setting up bsdutils (1:2.40.2-11) ...
(Reading database ... 129246 files and directories currently installed.)
Preparing to unpack .../login_1%3a4.16.0-2+really2.40.2-11_amd64.deb ...
Unpacking login (1:4.16.0-2+really2.40.2-11) over (1:4.16.0-2+really2.40.2-9) ...
Setting up login (1:4.16.0-2+really2.40.2-11) ...
(Reading database ... 129246 files and directories currently installed.)
Preparing to unpack .../eject_2.40.2-11_amd64.deb ...
Unpacking eject (2.40.2-11) over (2.40.2-9) ...
Preparing to unpack .../bsdextrautils_2.40.2-11_amd64.deb ...
Unpacking bsdextrautils (2.40.2-11) over (2.40.2-9) ...
Preparing to unpack .../util-linux-extra_2.40.2-11_amd64.deb ...
Leaving 'diversion of /sbin/ctrlaltdel to /sbin/ctrlaltdel.usr-is-merged by util-linux-extra'
Leaving 'diversion of /sbin/fsck.cramfs to /sbin/fsck.cramfs.usr-is-merged by util-linux-extra'
Leaving 'diversion of /sbin/fsck.minix to /sbin/fsck.minix.usr-is-merged by util-linux-extra'
Leaving 'diversion of /sbin/mkfs.bfs to /sbin/mkfs.bfs.usr-is-merged by util-linux-extra'
Leaving 'diversion of /sbin/mkfs.cramfs to /sbin/mkfs.cramfs.usr-is-merged by util-linux-extra'
Leaving 'diversion of /sbin/mkfs.minix to /sbin/mkfs.minix.usr-is-merged by util-linux-extra'
Unpacking util-linux-extra (2.40.2-11) over (2.40.2-9) ...
Preparing to unpack .../fdisk_2.40.2-11_amd64.deb ...
Unpacking fdisk (2.40.2-11) over (2.40.2-9) ...
Preparing to unpack .../libsmartcols1_2.40.2-11_amd64.deb ...
Unpacking libsmartcols1:amd64 (2.40.2-11) over (2.40.2-9) ...
Setting up libsmartcols1:amd64 (2.40.2-11) ...
(Reading database ... 129242 files and directories currently installed.)
Preparing to unpack .../libblkid1_2.40.2-11_amd64.deb ...
Unpacking libblkid1:amd64 (2.40.2-11) over (2.40.2-9) ...
Setting up libblkid1:amd64 (2.40.2-11) ...
(Reading database ... 129242 files and directories currently installed.)
Preparing to unpack .../libmount1_2.40.2-11_amd64.deb ...
Unpacking libmount1:amd64 (2.40.2-11) over (2.40.2-9) ...
Setting up libmount1:amd64 (2.40.2-11) ...
(Reading database ... 129242 files and directories currently installed.)
Preparing to unpack .../mount_2.40.2-11_amd64.deb ...
Unpacking mount (2.40.2-11) over (2.40.2-9) ...
Preparing to unpack .../libuuid1_2.40.2-11_amd64.deb ...
Unpacking libuuid1:amd64 (2.40.2-11) over (2.40.2-9) ...
Setting up libuuid1:amd64 (2.40.2-11) ...
(Reading database ... 129242 files and directories currently installed.)
Preparing to unpack .../util-linux_2.40.2-11_amd64.deb ...
Unpacking util-linux (2.40.2-11) over (2.40.2-9) ...
Setting up util-linux (2.40.2-11) ...
fstrim.service is a disabled or a static unit not running, not starting it.
(Reading database ... 129239 files and directories currently installed.)
Preparing to unpack .../00-libfdisk1_2.40.2-11_amd64.deb ...
Unpacking libfdisk1:amd64 (2.40.2-11) over (2.40.2-9) ...
Selecting previously unselected package openjdk-17-jdk-headless:amd64.
Preparing to unpack .../01-openjdk-17-jdk-headless_17.0.13+11-2_amd64.deb ...
Unpacking openjdk-17-jdk-headless:amd64 (17.0.13+11-2) ...
Selecting previously unselected package openjdk-17-jdk:amd64.
Preparing to unpack .../02-openjdk-17-jdk_17.0.13+11-2_amd64.deb ...
Unpacking openjdk-17-jdk:amd64 (17.0.13+11-2) ...
Selecting previously unselected package ghidra.
Preparing to unpack .../03-ghidra_11.0+ds-0kali1_amd64.deb ...
Unpacking ghidra (11.0+ds-0kali1) ...
Selecting previously unselected package ghidra-data.
Preparing to unpack .../04-ghidra-data_10.5-0kali1_all.deb ...
Unpacking ghidra-data (10.5-0kali1) ...
Selecting previously unselected package xorg-sgml-doctools.
Preparing to unpack .../05-xorg-sgml-doctools_1%3a1.11-1.1_all.deb ...
Unpacking xorg-sgml-doctools (1:1.11-1.1) ...
Selecting previously unselected package x11proto-dev.
Preparing to unpack .../06-x11proto-dev_2024.1-1_all.deb ...
Unpacking x11proto-dev (2024.1-1) ...
Selecting previously unselected package libice-dev:amd64.
Preparing to unpack .../07-libice-dev_2%3a1.1.1-1_amd64.deb ...
Unpacking libice-dev:amd64 (2:1.1.1-1) ...
Selecting previously unselected package uuid-dev:amd64.
Preparing to unpack .../08-uuid-dev_2.40.2-11_amd64.deb ...
Unpacking uuid-dev:amd64 (2.40.2-11) ...
Selecting previously unselected package libsm-dev:amd64.
Preparing to unpack .../09-libsm-dev_2%3a1.2.4-1_amd64.deb ...
Unpacking libsm-dev:amd64 (2:1.2.4-1) ...
Selecting previously unselected package libxau-dev:amd64.
Preparing to unpack .../10-libxau-dev_1%3a1.0.11-1_amd64.deb ...
Unpacking libxau-dev:amd64 (1:1.0.11-1) ...
Selecting previously unselected package libxdmcp-dev:amd64.
Preparing to unpack .../11-libxdmcp-dev_1%3a1.1.2-3+b2_amd64.deb ...
Unpacking libxdmcp-dev:amd64 (1:1.1.2-3+b2) ...
Selecting previously unselected package xtrans-dev.
Preparing to unpack .../12-xtrans-dev_1.4.0-1_all.deb ...
Unpacking xtrans-dev (1.4.0-1) ...
Selecting previously unselected package libxcb1-dev:amd64.
Preparing to unpack .../13-libxcb1-dev_1.17.0-2+b1_amd64.deb ...
Unpacking libxcb1-dev:amd64 (1.17.0-2+b1) ...
Selecting previously unselected package libx11-dev:amd64.
Preparing to unpack .../14-libx11-dev_2%3a1.8.10-2_amd64.deb ...
Unpacking libx11-dev:amd64 (2:1.8.10-2) ...
Selecting previously unselected package libxt-dev:amd64.
Preparing to unpack .../15-libxt-dev_1%3a1.2.1-1.2+b1_amd64.deb ...
Unpacking libxt-dev:amd64 (1:1.2.1-1.2+b1) ...
Preparing to unpack .../16-util-linux-locales_2.40.2-11_all.deb ...
Unpacking util-linux-locales (2.40.2-11) over (2.40.2-9) ...
Setting up bsdextrautils (2.40.2-11) ...
Setting up eject (2.40.2-11) ...
Setting up xtrans-dev (1.4.0-1) ...
Setting up uuid-dev:amd64 (2.40.2-11) ...
Setting up openjdk-17-jdk-headless:amd64 (17.0.13+11-2) ...
update-alternatives: using /usr/lib/jvm/java-17-openjdk-amd64/bin/jar to provide /usr/bin/jar (jar) in auto mode
update-alternatives: using /usr/lib/jvm/java-17-openjdk-amd64/bin/jarsigner to provide /usr/bin/jarsigner (jarsigner) in auto mode
update-alternatives: using /usr/lib/jvm/java-17-openjdk-amd64/bin/javac to provide /usr/bin/javac (javac) in auto mode
update-alternatives: using /usr/lib/jvm/java-17-openjdk-amd64/bin/javadoc to provide /usr/bin/javadoc (javadoc) in auto mode
update-alternatives: using /usr/lib/jvm/java-17-openjdk-amd64/bin/javap to provide /usr/bin/javap (javap) in auto mode
update-alternatives: using /usr/lib/jvm/java-17-openjdk-amd64/bin/jcmd to provide /usr/bin/jcmd (jcmd) in auto mode
update-alternatives: using /usr/lib/jvm/java-17-openjdk-amd64/bin/jdb to provide /usr/bin/jdb (jdb) in auto mode
update-alternatives: using /usr/lib/jvm/java-17-openjdk-amd64/bin/jdeprscan to provide /usr/bin/jdeprscan (jdeprscan) in auto mode
update-alternatives: using /usr/lib/jvm/java-17-openjdk-amd64/bin/jdeps to provide /usr/bin/jdeps (jdeps) in auto mode
update-alternatives: using /usr/lib/jvm/java-17-openjdk-amd64/bin/jfr to provide /usr/bin/jfr (jfr) in auto mode
update-alternatives: using /usr/lib/jvm/java-17-openjdk-amd64/bin/jimage to provide /usr/bin/jimage (jimage) in auto mode
update-alternatives: using /usr/lib/jvm/java-17-openjdk-amd64/bin/jinfo to provide /usr/bin/jinfo (jinfo) in auto mode
update-alternatives: using /usr/lib/jvm/java-17-openjdk-amd64/bin/jlink to provide /usr/bin/jlink (jlink) in auto mode
update-alternatives: using /usr/lib/jvm/java-17-openjdk-amd64/bin/jmap to provide /usr/bin/jmap (jmap) in auto mode
update-alternatives: using /usr/lib/jvm/java-17-openjdk-amd64/bin/jmod to provide /usr/bin/jmod (jmod) in auto mode
update-alternatives: using /usr/lib/jvm/java-17-openjdk-amd64/bin/jps to provide /usr/bin/jps (jps) in auto mode
update-alternatives: using /usr/lib/jvm/java-17-openjdk-amd64/bin/jrunscript to provide /usr/bin/jrunscript (jrunscript) in auto mode
update-alternatives: using /usr/lib/jvm/java-17-openjdk-amd64/bin/jshell to provide /usr/bin/jshell (jshell) in auto mode
update-alternatives: using /usr/lib/jvm/java-17-openjdk-amd64/bin/jstack to provide /usr/bin/jstack (jstack) in auto mode
update-alternatives: using /usr/lib/jvm/java-17-openjdk-amd64/bin/jstat to provide /usr/bin/jstat (jstat) in auto mode
update-alternatives: using /usr/lib/jvm/java-17-openjdk-amd64/bin/jstatd to provide /usr/bin/jstatd (jstatd) in auto mode
update-alternatives: using /usr/lib/jvm/java-17-openjdk-amd64/bin/serialver to provide /usr/bin/serialver (serialver) in auto mode
update-alternatives: using /usr/lib/jvm/java-17-openjdk-amd64/bin/jhsdb to provide /usr/bin/jhsdb (jhsdb) in auto mode
Setting up ghidra-data (10.5-0kali1) ...
Setting up libfdisk1:amd64 (2.40.2-11) ...
Setting up mount (2.40.2-11) ...
Setting up xorg-sgml-doctools (1:1.11-1.1) ...
Setting up util-linux-locales (2.40.2-11) ...
Setting up util-linux-extra (2.40.2-11) ...
Setting up openjdk-17-jdk:amd64 (17.0.13+11-2) ...
update-alternatives: using /usr/lib/jvm/java-17-openjdk-amd64/bin/jconsole to provide /usr/bin/jconsole (jconsole) in auto mode
Setting up fdisk (2.40.2-11) ...
Setting up ghidra (11.0+ds-0kali1) ...
Processing triggers for libc-bin (2.40-3) ...
Processing triggers for man-db (2.11.2-2) ...
Processing triggers for sgml-base (1.31) ...
Processing triggers for mailcap (3.70+nmu1) ...
Setting up x11proto-dev (2024.1-1) ...
Setting up libxau-dev:amd64 (1:1.0.11-1) ...
Setting up libice-dev:amd64 (2:1.1.1-1) ...
Setting up libsm-dev:amd64 (2:1.2.4-1) ...
Setting up libxdmcp-dev:amd64 (1:1.1.2-3+b2) ...
Setting up libxcb1-dev:amd64 (1.17.0-2+b1) ...
Setting up libx11-dev:amd64 (2:1.8.10-2) ...
Setting up libxt-dev:amd64 (1:1.2.1-1.2+b1) ...
```

## 2. Patch manuel programme simple
🌞 Récupérez le code de password_2.c sur la machine Linux et compilez-le
```sh
nathan@ServeurDebian:~/work$ cat > password_2.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <openssl/md5.h>
#include <unistd.h>

#define HASH_SIZE 16

const unsigned char correct_hash[HASH_SIZE] = {
    0x88, 0x1e, 0xbd, 0xd1, 0x5d, 0x8c, 0x07, 0xd9, 0xc3, 0xff, 0xbb, 0x24, 0xfa, 0xc4, 0x3b, 0x1f
};

void spawn_shell() {
    printf("Access granted! Spawning shell...\n");
    execve("/bin/sh", NULL, NULL);
}

void compute_md5(const char *str, unsigned char *result) {

    MD5_CTX ctx;
    MD5_Init(&ctx);
    MD5_Update(&ctx, str, strlen(str));
    MD5_Final(result, &ctx);
}

int main(int argc, char *argv[]) {
    if (argc != 2) {
        fprintf(stderr, "Usage: %s <password>\n", argv[0]);
        return 1;
    }

    unsigned char hash[HASH_SIZE];
    compute_md5(argv[1], hash);

    if (memcmp(hash, correct_hash, HASH_SIZE) == 0) {
        spawn_shell();
    } else {
        fprintf(stderr, "Access denied.\n");
    }

    return 0;
}
```


### 4. Ptit chall maison

🌞 Récupérer le flag du programme kaddate_challenge

* téléchargez le programme depuis votre machine Linux (avec une commande wget)
  ```sh
nathan@ServeurDebian:~/work$ wget https://gitlab.com/it4lik/b1-os/-/raw/main/tp/5/kaddate_challenge0
--2024-11-27 20:04:41--  https://gitlab.com/it4lik/b1-os/-/raw/main/tp/5/kaddate_challenge
Resolving gitlab.com (gitlab.com)... 172.65.251.78, 2606:4700:90:0:f22e:fbec:5bed:a9b9
Connecting to gitlab.com (gitlab.com)|172.65.251.78|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 16208 (16K) [application/octet-stream]
Saving to: ‘kaddate_challenge.2’

kaddate_challenge.2           100%[=================================================>]  15.83K  --.-KB/s    in 0.007s

2024-11-27 20:04:42 (2.14 MB/s) - ‘kaddate_challenge.2’ saved [16208/16208]
``` 

* Infos
```sh
nathan@ServeurDebian:~$ readelf -h kaddate_challenge
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              DYN (Position-Independent Executable file)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x10c0
  Start of program headers:          64 (bytes into file)
  Start of section headers:          14288 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         13
  Size of section headers:           64 (bytes)
  Number of section headers:         30
  Section header string table index: 29
```

 * get the flag !
  ```sh
nathan@ServeurDebian:~/work$ ./kaddate_challenge
Can you be the first to send us a Message ?
Enter you message:
ougcfgnc bvnkiugcf
Congrats ougcfgnc bvnkiugcf, you are the first user (count=5).
Flag: b1-os{PetitB1deviendraGrandPetitB1}
```










