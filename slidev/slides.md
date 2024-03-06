---
class: text-center
highlighter: shiki
lineNumbers: false
drawings:
  persist: false
transition: slide-left
title: CS130 Operating System Tutorial 1
mdc: true
monaco: true
---

# CS130 Operating System Tutorial 1

---

<Toc></Toc>

---

## Pintos Introduction

Pintos is a simple operating system framework for the 80x86 architecture.

It supports

- kernel threads (project 1)
- loading and running user programs (project 2)
- virtual memory (project 3)
- file system (project 4)

, but it implements all of these in a very simple way. 

---

### Tasks

Deadline Sunday Mar 24, 23:59 CST (not 23:59:59!)

All on GitHub. use the classroom link on Piazza

Document: https://web.stanford.edu/class/cs140/projects/pintos/pintos.html

You need to

1. Pass as much tests as you can
2. Answer design questions. The question is in `doc/threads.tmpl`.
  - Add the answer file in the repository.
  - Format: a plain text, a markdown file, or a PDF file
3. Participate the onsite TA check
  - Monday Mar 25, 7:30pm-8:00pm, SIST 1D-107

If you have any question, please ask on Piazza.

---

### Grading

1. Tests (60%)
  - 10% for each test
2. Design Questions (20%)
3. TA check (20%)

You need to pass at least 40% of the tests to pass the project

---

### Pintos Project 1 Files Overview

```
├── src
│   ├── devices (disk, keyboard, ...)
│   ├── examples (example functions)
│   ├── filesys (project 4)
│   ├── lib (helper functions)
│   ├── threads (project 1)
│   │   ├── init.c <-- 3rd executes (main)
│   │   ├── interrupt.c
│   │   ├── intr-stubs.S
│   │   ├── io.h
│   │   ├── kernel.lds.S
│   │   ├── loader.S <-- 1st executes
│   │   ├── malloc.c
│   │   ├── palloc.c
│   │   ├── pte.h
│   │   ├── start.S <-- 2nd exectues
│   │   ├── switch.S
│   │   ├── synch.c
│   │   ├── thread.c
│   │   └── vaddr.h
│   ├── userprog (project 2)
│   ├── utils (utilities)
│   └── vm (project 1)
└── tests
```

---

### How Pintos boots

1. BIOS finds the bootloader on the disk, which is `loader.S`, compiled into `loader.bin`
2. `loader.bin` runs on CPU, and load the kernel to memory
3. `loader.bin` jumps to `start.S`
4. `start.S` calls `int main(void)` in `init.c`
5. `thread_init`, ...
6. `printf ("Boot complete.\n");`
7. `run_actions (argv);`
8. `shutdown ()`, `thread_exit ()`

---

### The difference between OS and average user programs

1. OS can execute privillege instructions (ring 0)
  - Enable, disable interrupts
2. Access all memory, control page table
3. Interact with other devices

---

#### Interrupt

Interrupts are signals from a device, such as a keyboard or a hard drive, to the CPU, telling it to immediately stop whatever it is currently doing and do something else. 

1. Every $t$ microseconds, a timer will send a interrupt to CPU
2. When other devices receives some data, it will send a interrupt to CPU
3. etc...

CPU sets a list of functions, specify which function to execute when receiving a type of interrupt.

Example:

1. Every time slice (5ms for example), send a time interrupt to CPU. Then CPU switches from user program to interrupt handler. Therefore, kernel takes the control flow. So kernel can do process switching, statistics collecting.
2. When a network interface receive a packet from Ethernet cable, it sends a interrupt to CPU, calling a function in kernel to process the packet.
3. etc...

---

## How to get started

- Environment setup
- Compile and Run
- GDB, clangd, ...

---

### Docker

```bash
docker run -it syrriinge/pintos-sp24:latest
```

#### Devcontainer

Use VSCode to quickly reopen the project in Docker.

<img src="/vsc-devcontainer.png" class="h-50" />

---

### Compile and Run

```bash
# project 1
project1-sp24-ta-team$ cd src/threads/

# compile
project1-sp24-ta-team/src/threads$ make
project1-sp24-ta-team/src/threads$ ls build/kernel.* build/loader.bin
build/kernel.bin  build/kernel.o  build/loader.bin

# run
project1-sp24-ta-team/src/threads$ pintos -v -k -T 60 --bochs -- -q run alarm-single # bochs
project1-sp24-ta-team/src/threads$ pintos -v -k -T 60 --qemu -- -q run alarm-single # qemu

# test, grade
project1-sp24-ta-team/src/threads$ make check
project1-sp24-ta-team/src/threads$ make grade
```

---

### GDB

`make check` prints the `pintos` command

e.g. `pintos -v -k -T 480 --bochs  -- -q -mlfqs run mlfqs-load-60`

Add `--gdb` to run Pintos under the supervision of the GDB debugger.

Then run `pintos-gdb` in another terminal.

```bash
$ pintos --gdb -v -k -T 480 --bochs  -- -q -mlfqs run mlfqs-load-60 # or use --qemu instead of --bochs 
Waiting for gdb connection on port 1234
```

```
# open another terminal
$ pintos-gdb kernel.o
(gdb) target remote localhost:1234
Remote debugging using localhost:1234
0x0000fff0 in ?? ()
(gdb) b main
Breakpoint 1 at 0xc0020225: file ../../threads/init.c, line 78.
(gdb) c
Continuing.
Breakpoint 1, main () at ../../threads/init.c:78
78      {
```

---

### GDB in VSCode

<img src="/vsc-gdb.png"/>

---

### Clangd

> clangd understands your C++ code and adds smart features to your editor: code completion, compile errors, go-to-definition and more.

1. Install `clangd` extension in VSCode
2. Generate `compile_commands.json`

```bash
src/threads$ rm -rf build
src/threads$ bear -- make
src/threads$ mv compile_commands.json build/
```

3. Add `--compile-commands-dir=src/threads/build` in `Clangd: Argument` VSCode setting entry

<img src="/vsc-clangd.png" class="h-50"/>

---


## How to DEBUG

**USE ASSERT TO PREVENT SILLY MISTAKES!!!**

**USE PRINTF!!!**

**USE GDB!!!**

### More about printf

1. `printf` calls `vprintf`
2. `vprintf` calls `acquire_console`
3. `acquire_console` calls `lock_acquire`
4. `lock_acquire` calls `sema_down`
5. ...

`printf` is easy to cause kernel panic if you didn't correctly implement the semaphore, scheduing, ...

Use GDB if `printf` fails to work